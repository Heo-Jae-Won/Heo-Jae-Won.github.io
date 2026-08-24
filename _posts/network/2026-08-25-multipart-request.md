---
title: why multipart-request disappear on HTTP 2/3
published: true
categories: [network]
---


## <span style="color:#802548">_file multipart request in HTTP 1.1_</span>
- text + binary file request의 기본적인 작동은 text request + binary file request 따로하는 방식이다.
- 즉 binary file 3개를 포함한 request라고 한다면, 1개의 text request와 3개의 binary file request로 구성되는 것이다.
    - HTTP 1.1 이하의 시대에서는 TCP connection이 browser tab 당 6개만 허용된 데 비해, 보내고자 하는 image가 6개를 넘을 수 있다.
        - 이 경우 얄짤없이 HOL 문제에 시달리게 된다. 즉, 이전 binary file request가 processed 되기 전까지 browser가 freeze 된다.
    - 다른 문제는 form에 넣어서 image를 보내는 경우, application/x-www-form-urlencoded로 지정하는 경우가 많았다는 점이다.
        - 이 경우, base64 encoding이 일어나면서 파일 크기도 늘어나고, encode와 decode에 CPU를 massive하게 consume하게 된다. 
- 이러한 경위에 따라, binary file multipart request가 등장했다.
    - binary file multipart request는 file과 text를 전부 1개의 request로 처리함으로써 위의 내용을 모두 bypass할 수 있었다.


    ## <span style="color:#802548">_file multipart request lose power_</span>
- 하지만, HTTP 2/3으로 진화하면서 위와 같은 multipart request는 lose power.
    - borwser tab per HTTP connection의 limit가 Multiplexing이 emerge하면서 disappear했기 때문이다.
    - from now on, HTTP2/3은 domain에 대해서 6개가 아닌 only 1 TCP/QUIC connection을 사용한다. 대신 multiplexing으로 overcome한다
- since HTTP2/3 이 방식은 multipart request보다 장점이 더 컸다
    - Error Isolation: multipart request는 file2가 에러가 happens 하면 전체 request가 fail
    - Caching Optimization: multipart request는 uncacheable blob이었다
    - Network Efficiency: parallel streaming이 가능해졌다
    - cloud-native: can utilize S3 storages, which means no WAS usage

## <span style="color:#802548">_file multipart request caution_</span>
- 다만 문제는 1개의 request로 handle 하던 때와 달리, request가 나눠졌으니 api 별로 logic이 scattered 된다
- 이를 integrate 하는 작업이 by hand였다는 점이다
- post를 예로 들자면 request 1개에 모든 과정이 finished인 것이, 전부 divided된 것이다.
    - API 1 (Create Post): The post ID Handshake for images folder
    - API 2 (Upload Images): image upload to exact postId folder 
- uplaod image는 post에 대해선 sync지만, respective binary file request에 대해서는 simultaneously하게 proceed

```js
// Step 1: Fast text save to get the record ID
const textResponse = await fetch('/api/board/posts', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ title: "My Travel Post", content: "Check out these photos!" })
});
const { postId } = await textResponse.json(); 

// Step 2: Fire all 3 file uploads ASYNCHRONOUSLY and SIMULTANEOUSLY
const imageFiles = [file1, file2, file3];

const uploadPromises = imageFiles.map((file, index) => {
    return fetch(`/api/board/posts/${postId}/images`, {
        method: 'POST',
        headers: { 
            'Content-Type': file.type,
            'X-Image-Index': index // Helps backend track the path/order if needed
        },
        body: file // Raw file binary streaming simultaneously
    });
});

// Wait for all simultaneous uploads to finish
const results = await Promise.all(uploadPromises);
console.log("All files uploaded concurrently!");
```

- server는 separate request를 prepare
- multipart request를 쓰지 않으므로, request에서 직접 꺼내와서 encoding, decoding하고 stream으로 직접 webserver에 파일을 create한다

```java
@RestController
@RequestMapping("/api/board/posts/{postId}/images")
public class ImageUploadController {

    @PostMapping
    public ResponseEntity<?> uploadRawImage(
            @PathVariable Long postId,
            HttpServletRequest request) {
        
        try {
            // 1. Extract metadata from custom headers
            String encodedFileName = request.getHeader("X-File-Name");
            if (encodedFileName == null || encodedFileName.isEmpty()) {
                return ResponseEntity.badRequest().body("Missing X-File-Name header.");
            }
            String fileName = URLDecoder.decode(encodedFileName, StandardCharsets.UTF_8);

            // 2. Define the target storage path using NIO.2 Path
            Path uploadDir = Paths.get("C:/uploadPath1", postId.toString());
            Files.createDirectories(uploadDir); // Ensure directory exists
            Path targetFilePath = uploadDir.resolve(fileName);

            // 3. Stream the raw binary body directly to the destination path
            try (InputStream inputStream = request.getInputStream()) {
                Files.copy(inputStream, targetFilePath, StandardCopyOption.REPLACE_EXISTING);
            }

            return ResponseEntity.ok(Map.of(
                "message", "File uploaded successfully",
                "path", targetFilePath.toString()
            ));

        } catch (Exception e) {
            e.printStackTrace();
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body("Upload failed: " + e.getMessage());
        }
    }
}
```

## <span style="color:#802548">_why Grouped Folder Way is needed_</span>

- 참고로 파일은 unified folder에 dump할 게 아니라, post에 대해 respective 한 folder를 만들어줘야 한다
    - file이 10,000를 pass over하면 OS file search가 slow down
        - folder는 해당 folder inside file들에 대해 index of pointers를 갖는다.
        - 파일이 많아지면 index가 CPU L1/L2/L3 cache utilize가 impossible, 더 많아지면 RAM cache도 impossible
        - eventually file search에 disk I/O가 work, so-called Disk Thrashing. hunting for file paths in the index하는데 시간을 쓴다. actually reading the image bytes가 아니라
    - file을 read할 때 OS level의 competition이 일어난다
        - file read 시 system call하는데, OS kernel이 directory-level lock을 걸고, concurrent user들은 서로 corresponding lock을 기다리는 데 시간을 consume
    - file 이름이 identical해서 collision 혹은 overwrite
    - S3 storage같은 cloud-native에 freiendly
        - separate prefixes기에 Throttling request에도 no problem
- 이러한 결론에 따라서 Grouped Folder Way를 modern web은 prefer한다.



```text
/storage/uploads/
  └── 2026/
      └── 08/
          └── 25/
              └── post-992/
                  ├── photo1.jpg
                  └── photo2.jpg

```
