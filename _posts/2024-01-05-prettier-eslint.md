---
title: prettier & esLint 설정법
published: true
categories:
  - Eslint
tags:
  - Eslint

toc: true
toc_sticky: true
 
date: 2024-01-05
last_modified_at: 2024-01-05
---

- prettier를 설정한 이후 오류가 났다.
  - 자꾸 const가 initialize가 안된다.
  - declare module에서 semi colon을 찾는 오류가 난다.
-  이렇게 시간을 날려버렸다.
-  그래서 어떻게 적용하는 지 간략하게 정리했다. 

## <span style="color:#802548">_1. eslint 적용_</span>
- 먼저 eslint는 프로젝트마다 다를 수 있다.
- extension이 아니라 dependency로 깔게 된 이유다.
  - 전역으로 깔면 안되고, 해당 프로젝트에만 적용하게 깐다. 
  - --save-dev는 -D와 동일하며, 개발에서만 쓰인다는 의미다.
```
npm install --save-dev eslint
```
- 그 뒤에 eslint를 npx로 실행한다. 
```
npx eslint --init
```
- 답변을 다 하게 되면 .eslintrc.json이라는 파일이 생성된다. 
  - 맨 마지막 추가 plugin도 깔아주자.
  -  충돌이 잘 나는 아래 plugin은 지워주자. 이상하게 저것만 prettier하고 자꾸 충돌이 난다.
```
npm remove @vue/eslint-config-typescript
```
- .eslintrc파일에 내용을 추가해야 한다.
  - vue와 관련된 내용은 JS 기본이 아니기에, 인식시키려면 parser가 필요하다.
  - 그러려면 vue parser를 깔아주자. 
```
npm i -D vue-eslint-parser
```
- npm i -D로 깔면 개발용, npm i -S(--save)로 깔면 배포용이다. 
- 그런데 npm 5버전부터는 npm i가 곧 npm i -S다.
  - 개발용으로만 깔 게 아니면 npm i로 그냥 깔면 된다. 
  - 아래 사이트를 보면 npm install의 여러 옵션이 설명되어 있다.

https://docs.npmjs.com/cli/v9/commands/npm-install

- .eslintrc 내용은 아래와 같이 구성했다.
  - async가 있는데 await가 없으면 에러를 낸다.
  - ;를 두개이상 붙이면 error를 낸다.
  - 사용되지 않는 것들을 놔두면 에러가 나는 형식이다. 
```js
{
  "env": {
    "browser": true,
    "es2021": true
  },
  "extends": ["eslint:recommended", "plugin:vue/essential", "plugin:@typescript-eslint/recommended"],
  "parserOptions": {
    "ecmaVersion": 12,
    "parser": "@typescript-eslint/parser",
    "sourceType": "module"
  },
  "plugins": ["vue", "@typescript-eslint"],
  "rules": {
    "no-extra-semi": "error",
    "require-await": "error",
    "no-unused-expressions": [
      "error",
      {
        "allowTernary": true, // a || b
        "allowShortCircuit": true, // a ? b : 0
        "allowTaggedTemplates": true
      }
    ]
  },
  "parser": "vue-eslint-parser"
}
```
- eslint에 관한 구성은 아래 사이트에 잘 설명되어있다.

https://velog.io/@kyusung/eslint-config-2
 
rule 옵션은 아래 공식홈페이지를 참조하자.

https://eslint.org/docs/latest/rules/

- 참고로 vue 프로젝트기 때문에 vue parser를 넣은 것이다. 
- parser가 없어서 vue파일 인지 인식불가능하다.
- 결국 아래 같은 오류가 난다. 
```
 Eslint Vue 3 Parsing error: '>' expected.eslint 
```

- 따라서 parser 설정은 필수다. 
- 그래서 위에서 해당 moduel을 npm에서 받은 것이다.
```
"parser": "vue-eslint-parser"
 ```
- 그러고 나서 lint 명령어를 실행하면 터미널에 error와 warn이 뜬다. 
- 참고로 그냥 소스코드 쓸 때도 rule에 따라 error 표시를 해준다. 

- eslint가 무시했으면 하는 설정파일은 .eslintignore에 적어둔다. 
- 참고로 js파일이면 .js라는 확장자까지 모두 적어줘야 한다. 
- node_modules는 필수로 들어가야 한다. 
  - 아니면 node_modules도 검사하기 때문에 에러문구가 너무 많이 뜬다.


## <span style="color:#802548">_2. prettier 설정_</span>
- prettier는 코딩 스타일에 관한 것이라 전역적으로 사용하려고 그냥 extension을 깔았다.  

- prettier를 깔았다면 설정파일을 만들어주자. 
- 설정파일 이름은 .prettierrc.json으로 만들었다.
- prettierrc로도 만들수있다. eslint+rc, prettier+rc로 뒤에 rc가 붙는데  자동 실행될 스크립트인 runcom file에서 유래됐다고 한다.

- 한마디로 run commands 의 약자인데, 여기서는 runtime configuration이라고 생각된다. 
```
{
  "singleQuote": true,
  "bracketSpacing": true,
  "bracketSameLine": true,
  "arrowParens": "avoid",
  "printWidth": 120,
  "tabWidth": 2
}
```

