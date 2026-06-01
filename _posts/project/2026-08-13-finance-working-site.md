## <span style="color:#802548">_Java and C# : difference of array_</span>
- in C#, array's length represents for initial size.
- so we can calculate actual size in array, using methods.

```C#
string[] array = new string[5];
int actualCount = array.Count(s => s is not null);
```

- in Java, we cannot do this.
- why this kinda things happen?
- because, unlike Java, C# array implements IEnumerable<T>
- so we can use IEnumerable's methods even if being array type

```C#
using System.Linq;

string[] array = new string[5];

int count = array.Count(); // works
```


## <span style="color:#802548">_Java and C# : difference of list_</span>
- then, what about list ?
- in Java, size must be calculated by size()
- but, in C#, thanks to that actual size is public property, we can just get actual size by calling property Count
- and it's time complexity is O(1), which means it is cheaper than calling Counts() 
- but the problem is that, Count includes everything, even null. 

```text
list.Count 
```

- so, if u need to count followed by condition, u should use Count() method, like below.

```C#
int actualCount = array.Count(s => s is not null);
```


## <span style="color:#802548">_Java and C# : nullable dealing_</span>

- in Java, every type could be null in compile time
- in contrast, in C#, without Operator ?, types could not be nullable
- and nullable column and not-nullable property default value would be different 
    - DateOnly? is nullable
        - when no return from database, null return iun C#
    - DateOnly is not-nullable
        - when no return from database. its "0001-01-01" in C#

```C#
DateOnly? date; //null
DateOnly date;  //0001-01-01
```

- type and type? is also different type
- to decimal and decimal?, we cannot operate +, cuz one can be null
    - so type casting is needed
    - or default value setting is needed

```C#
// type casting
decimal result = (decimal)(a + b);

// default value setting
decimal result = (a ?? 0m) + (b ?? 0m);
```



## <span style="color:#802548">_DB : dynamic where condtion cautions_</span>
- for writing dynamic condition for where clause, need to pay attention to unvalid where clause
- let's suppose that in C#, we construct sql like below.

```C#
//SQL mapper form
WHERE c.status = 'ACTIVE'AND c.startDate >= {startDate}


// sQL ORM
var query = db.Contracts
    .Where(c => c.StartDate >= startDate);
```

- when startDate being null, it would produce like below
- then  c.startDate >= NULL is unvalid condition, so where clause overall becomes false
    - therefore, even if c.status is true, this sql always returns empty resultset

```sql
WHERE c.status = 'ACTIVE' AND c.startDate >= NULL
```


## <span style="color:#802548">_DB : insert diff between nullable and not-nullable and default_</span>

- insert for non-nullable column, insert columns must be declared in insert statement
    - if not, exception is thrown.
- insert for nullable column, insert columns dont need to be declared in insert statement
    - then, null is inserted automatically
- insert for nullable default column, insert columns dont need to be declared in insert statement
    - if not declaerd in insert statement, default is inserted
    - if declared in insert statement but no value, null is inserted


## <span style="color:#802548">_C# DBcontext : AsNoTracking()_</span>
- just for getting resultset from db, AsNoTracking is needed.
    - it can perform better
        - these internal checks skips
    - it can save memory usage
        - No snapshot is created so RAM is saved

- if using FromSqlRaw() and just need to fetch rows, then use AsNoTracking()
- FromSQLRaw() tracks entity, so it is not proper choice for just fetching entity
- if u trying to jsut fetch, then use SqlQuery<>(sql)

```C#
using (var context = new AppDbContext())
{
    var users = context.Users
        .FromSqlRaw("SELECT * FROM Users")
        .AsNoTracking()
        .ToList();
}
```

- in case of SqlQuery<>(sql), no need for AsNoTracking()
- cuz it's automatically doesnt track entity
- SqlQuery<>() is designed for DTO and scalar type, so it does not track even if enttiy class is typed parameter
- therefore, AsNoTracking() is not needed here

```C#
var users = dbContext.SqlQuery<Entity>(sql).ToList().First();
```

## <span style="color:#802548">_C# DBcontext : SqlQuery<>()_</span>

- acutally, for SqlQuery<>() method, assigning entity class as a typed parameter is not mandatory
    - assigning DTO class is enough
- even if sql select columns are *, no needs for writing all property
    - just picking up some needed column for property is enoguh
    - then automatically mapping is occured

```C#
var users = dbContext.SqlQuery<UserDTO>(sql).ToList().First();

public class UserDTO
{
    public int UserId { get; set; }        // Primary Key (by convention)
    public string TraceOrder { get; set; }
    public string UserEmail { get; set; }

    public DateTime CreatedAt { get; set; }
}
```

- so, when there is a needs for tracking, use FromSqlRaw() or LINQ 
- no needs for tracking, use SqlQuery<>() or AsNoTracking()

```text
| SqlQuery<>()                  | FromSqlRaw() 
| Entity not mandatory          | Entity class mandatory
| untrackedd automatically      | tracked automatically
| no need for AsNoTracking()    | need for AsNoTracking()
```


## <span style="color:#802548">_C# DBcontext : FromSQL<>()_</span>

- u can use dbcontext and SQL with FROMSQL
- but it doesnt support projection


## <span style="color:#802548">_C# DBcontext : no ToList() for 1 row_</span>

- ToList() is also not needed for getting just 1 row
- ToList() pulls every row from your SQL result into your application's RAM.
- if u want just 1 rows, then use just First() or Single() alone

```C#
var users = dbContext.SqlQuery<UserDTO>(sql).First();
```

## <span style="color:#802548">_C# DBcontext : FirstOrDefault()_</span>
- we can use each nullable and non-nullable type for each method

```C#
dbContext.SqlQuery<>(sql).First();          // not nullable type
dbContext.SqlQuery<>(sql).FirstOrDefault(); //nullabla type
```

- making intention clear is also right choice 
- using both is best

```sql
SELECT TOP 1
```


## <span style="color:#802548">_C# DBcontext : async_</span>
- for controlelr -> service -> repositroy, u better use async style
    - it reduces thread running out
    - I/O bottleneck is removed

```text
**Sync (FirstOrDefault)
Thread A:
[Start] ── DB call ── waiting... ── waiting... ── [Result]
          (thread is stuck here)


**Async (FirstOrDefaultAsync)
Thread A:
[Start] ── DB call ──► (released)

Thread B (later):
             ◄── DB result arrives ── [continue]
```

- so, for sync per 100 concurrent requests needs 100 threads
- but for async only needs 10~20 threads
- for that, u should use Task class for getting async result

```C#
public interface IUserRepository
{
    Task<User?> GetByIdAsync(int id);
}
```

- should use Async method for query

```C#
public class UserRepository : IUserRepository
{
    public async Task<User?> GetByIdAsync(int id)
    {
        return await _dbContext.Users
            .Where(u => u.Id == id)
            .FirstOrDefaultAsync();
    }
}
```

- needs to use await for async method
- needs to wrap DTO class into Task class
- return type of repo would be DTO? class, not Task<User?> class

```C#
public class UserService : IUserService
{
    public async Task<UserDto?> GetUserAsync(int id)
    {
        User? user = await _repository.GetByIdAsync(id);

        if (user == null)
            return null;

        // business logic / mapping (sync work)
        return new UserDto
        {
            Id = user.Id,
            Name = user.Name
        };
    }
}
```

- Controller has also same logic
- declaration type is Task<>, but return is acutal typed parameter class

```C#
[HttpGet("{id}")]
    public async Task<IActionResult> GetUser(int id)
    {
        User? user = await _service.GetUserAsync(id);

        if (user == null)
            return NotFound();

        return Ok(user);
    }
```

## <span style="color:#802548">_C# DBcontext : where in clause() in LINQ_</span>
- how to use in condition in where clause

```C#
public Models.Entity FindOneWithCOndition(long No, DateOnly givenDate)
{
    var list = new[] 
        {
            CodeConstants.ApprovalType.APPROVED,
            CodeConstants.ApprovalType.NOTAPPROVED,
            CodeConstants.ApprovalType.CANCELD,
        }
    
    return _dbContext.Enttiy.AsNoTracking().SingleOrDefault(x => x.SequenceNo == SequenceNo
                                                            && x.ApplicationStartDate <= givenDate
                                                            && x.ApplicationEndDate >= givenDate
                                                            && list.Contains(x.ApprovalStatusDivision) /* in clauses*/)
}
```


## <span style="color:#802548">_C# DBcontext : partial update_</span>

- using state.Modified is dangerous

```C#
var user = new User
{
    Id = 1,
    Name = "Mike"
};

dbContext.Users.Attach(user);
dbContext.Entry(user).State = EntityState.Modified;

dbContext.SaveChanges();
```

- update sql would be below
- EntityState.Modified trigger all column update
- so, when no value is set, default value is set to column value like 0, null

```sql
UPDATE Users
SET Name = 'Mike',
    Email = NULL,
    Age = 0,
    City = NULL
WHERE Id = 1
```

- if u need partial update and in legacy C#, use .Property().IsModified
- this one is based on C# object
- Attach → Entry → IsModified → SaveChanges is serial process
    - Attach : it allows us to track entity object. it's like base for Entry()
    - Entry : it allows us to change state, like delete or update

```C#
var user = new User
{
    Id = 1,
    Name = "New Name",
    Email = "new@email.com",
    Age = 30,
    City = "Tokyo"
};

dbContext.Users.Attach(user);
var entry = dbContext.Entry(user);

foreach (var prop in new[] { "Name", "Email", "Age", "City" })
{
    entry.Property(prop).IsModified = true;
}

dbContext.SaveChanges();
```


- this one does not care about object
- this is for updating by bypassing efcore

```C#
dbContext.Users
    .Where(u => u.Age > 20)
    .ExecuteUpdate(setters => setters
        .SetProperty(u => u.Name, u => "Updated")
        .SetProperty(u => u.Email, u => "updated@email.com")
        .SetProperty(u => u.Age, u => 30)
        .SetProperty(u => u.City, u => "Tokyo")
        .SetProperty(u => u.IsActive, u => true)
    );
```

- if updated user is needed, then need to reload Context
- context reload trigger select query, so it update context entity if u had user

```C#
dbContext.Entry(user).Reload();
```

- in C#, they have also dirty checking style like below
- but this triggers select SQL for loading entity

```C#
var user = await dbContext.Users.FindAsync(1);

user.Name = "New Name";
user.Age = 30;
user.Email = "New@email.com";

await dbContext.SaveChangesAsync();
```


## <span style="color:#802548">_C# DBcontext : SQL injection prevention_</span>
- using SqlBuilder class with ExecuteSQL is dangerous

```C#
StringBuilder sqlBuilder = new(); 
sqlBuidler.append( $"SELECT * FROM Users WHERE Name = {userInput}"); 
FormattableString sqlQuery = FormattableStringFactory.Create(sqlBuilder .ToSTring()); 
dbContext.Database.ExecuteSql(sqlQuery);
```

- userInput is John 
    - the StringBuilder = SELECT * FROM Users WHERE Name = JohnIf
- userInput is ' OR 1=1 --
    - the StringBuilder = SELECT * FROM Users WHERE Name = '' OR 1=1 --
- so, we need to do it like below

```C#
FormattableString sql =
    $"SELECT * FROM Users WHERE Name = {userInput}";

dbContext.executeSQL(sql);
```

- userInput is ' OR 1=1 -- 
    - SQL would be SELECT * FROM Users WHERE Name = "' OR 1=1 --"
    - so condition is not satisfied

- if need column branching,

```C#
public User GetUserByCondition(string userSortColumn) {
    // SAFE: Map the input to a hard-coded string
    string safeColumn = userSortColumn switch
    {
        "name" => "Name",
        "price" => "Price",
        "date" => "CreatedAt",
        _ => "Id" // Default fallback
    };


    FormattableString sql =
         $"SELECT * FROM Users WHERE Name = {userInput} ORDER BY {safeColumn}"


    // Now you can safely use it in an interpolated string because YOU defined 'safeColumn'
    dbContext.ExecuteSql(sql);
}
```

- if need simple condition branching, below is right answer


```C#
public void GetUserByCondition(string userSortColumn) {
    // SAFE: Map the input to a hard-coded string
    string safeColumn = userSortColumn switch
    {
        "name" => "Name",
        "price" => "Price",
        "date" => "CreatedAt",
        _ => "Id" // Default fallback
    };


// Use explicit casting to FormattableString inside the ternary
FormattableString sql = updateDate is not null
    ? (FormattableString)$@"SELECT * FROM Users 
                            WHERE Name = {userInput} 
                            AND changeDate < {changeDate} 
                            AND updateDate > {DateTime.Now} 
                            ORDER BY {safeColumn}"
    : (FormattableString)$@"SELECT * FROM Users 
                            WHERE Name = {userInput} 
                            AND changeDate < {changeDate} 
                            ORDER BY {safeColumn}";
}
```

- if needs complex condition branching, below is good when cannot use LINQ

```C#
var parameters = new List<object>();
var sql = "SELECT * FROM Users WHERE Name = {0} AND changeDate < {1}";
parameters.Add(userInput);
parameters.Add(changeDate);

if (updateDate is not null)
{
    // Append to string and add to parameter list
    sql += " AND updateDate > {2}";
    parameters.Add(DateTime.Now);
}

sql += $" ORDER BY {safeColumn}"; // safeColumn must still be whitelisted!

// In EF Core:
var results = _context.Users.FromSqlRaw(sql, parameters.ToArray()).AsNoTracking().ToListAsync();
```

- if needs complex condition branching, using LINQ would be very convineint choice

```C#
var query = _context.Users
    .Where(u => u.Name == userInput && u.ChangeDate < changeDate);

if (updateDate is not null)
{
    query = query.Where(u => u.UpdateDate < updateDate && u.UpdateDate > DateTime.Now );
}
else {
    query = query.Where(u => u.UpdateDate > DateTime.Now);
}

// Use a library like System.Linq.Dynamic.Core for string-based ordering
// or a simple switch for typed ordering:
query = safeColumn switch {
    "Name" => query.OrderBy(u => u.Name),
    "Price" => query.OrderBy(u => u.Price),
    _ => query.OrderBy(u => u.Id)
};

var results = await query.ToListAsync();
```

## <span style="color:#802548">_DB : count(*) window function caution_</span>

- count() function needs group by
- but, using winodw function like count() over, we can skip group by

```sql
SELECT 
    empno,
    sal,
    COUNT(*) OVER (ORDER BY sal) AS running_count
FROM emp;
```

- but the problem is that, if duplicate happens, duplicates would be counted

```text
| empno | sal | running_count |
| ----- | --- | ------------- |
| 1     | 100 | 2             |
| 2     | 100 | 2             |
| 3     | 200 | 3             |
| 4     | 300 | 4             |
```

- actually it could cause problem

```sql
SELECT *
FROM (
    SELECT empno, sal,
           COUNT(*) OVER (ORDER BY sal) AS cnt
    FROM emp
) t
WHERE cnt = 2;
```

- each row has 1 count actually
- but it seems they have 2 count for each

```text
| sal | ROWS | CNT |
| --- | ---- | ----- |
| 100 | 1    | 2     |
| 100 | 2    | 2     |
```


- in order avoid this, needs to set rows as a criteria 

```sql
SELECT 
    empno,
    sal,
    COUNT(*) OVER (
        ORDER BY sal 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_count
FROM emp;
```

- then, different row is dealt as different count

```text
| empno | sal | running_count |
| ----- | --- | ------------- |
| 2     | 100 | 1             |
| 4     | 100 | 2             |
| 3     | 200 | 3             |
| 1     | 300 | 4             |
```

- want to see change of accumulated count, need to use ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
- dont want, dont use ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
- just dont set option

```sql
SELECT 
    empno,
    sal,
    COUNT(*) OVER (
        partition BY sal 
    ) AS running_count
FROM emp;
```



## <span style="color:#802548">_DB : count(*) window function vs count(*) group by_</span>

- group by is splits rows → aggregates → returns fewer rows
- windows keeps rows keeps rows intact, sustaining
- let's suppose this kinda row

```text
| Sale_ID | Salesperson | Region |
| ----- | --- | ------------- |
| 1     | Alice | North            |
| 2     | Bob | North             |
| 3     | Charlie | South             |
```

- if using group by

```sql
SELECT Region, COUNT(*) as Total
FROM Sales
GROUP BY Region;
```

- result is below

```text
| Sale_ID | Total |
| ----- | --- 
| North    | 2
| South     | 1
```


- if using group by

```sql
SELECT Sale_ID, Salesperson, Region,
       COUNT(*) OVER(PARTITION BY Region) as Region_Count
FROM Sales;

```

- result is below
- count window function can utilize all other columns

```text
| Sale_ID | Salesperson | Region |  Region_Count
| ----- | --- | ------------- |
| 1     | Alice | North            |    2
| 2     | Bob | North             |     2
| 3     | Charlie | South             | 1
```


- GROUP BY is often faster for large datasets with few unique groups
    - because it reduces the amount of data it has to process and move early on.
- Window Functions are better when you have a high number of unique groups (high cardinality) and need to see detail
    -  as they avoid the "summarize and join back" steps often needed to get row-level context.


## <span style="color:#802548">_C# constant grouping_</span>

- when u want to grouping constants
- partial is for seperating class files

```C#
public static partial class CdMstConstants
{

    public sealed partial class Code_TYPE_01
    {
        public const string DIVISION_CODE = "C0001"
    }

    public sealed partial class Code_TYPE_02
    {
        public const string DIVISION_CODE2 = "C0002"
    }
}
```

- when u want to include method in constant like Java enum

```C#
public class Status
{
    public string Name { get; }

    private Status(string name)
    {
        Name = name;
    }

    public static readonly Status Active = new("ACTIVE");
    public static readonly Status Inactive = new("INACTIVE");

    public bool IsActive()
    {
        return this == Active;
    }
}
```

## <span style="color:#802548">_C# DBConetxt: configuration_</span>
- after configuration Entity, then it's better making a GetContext util method
- this is version of localhost

```C#
public class TestUtil {

    public static MyDbContext GetContext() 
    {
        var options = new DbContextOptionsBuilder<MyDbContext>()
            .UseSqlServer(@"Data Source=localhost;Initial Catalog=my;Intergrated Security=True;TrustServerCertificate=True")
            .Options;

        return new MyDBContext(options);
    }
}

```

## <span style="color:#802548">_C# DBConetxt: executeSQLRaw_</span>

```C#
string sql = @"
    INSERT INTO TRN_SLIP_JOURNAL (
        FUND_CODE
    )
    VALUES (
        @FunDCode,
    )
";

foreach (var row in rows) {;
    _dbContext.Database.ExecuteSqlRaw(
        sql,
        new SqlParameter("@FundCode", row.fundCode);
        new SqlParameter("@ProgramId", row.programID == null ? DBNull.Value : row.programId);
    )
}
```







## <span style="color:#802548">_C# : Xunit_</span>
- this is AutoFac way in C# .net
    - at first, configuration needs to be implemented
    - at then, Servcie and Repo and DBContext
    - if first put DBcontext, then no repo and service DI is completed, which means no parameter of constructor of DbContext
- after regsitering service, we can change one service to mock service
    - mock service must be declared in that test configuration outside Test class as a internal class
- finally, build container

```C#
public class AServiceTest {
    private readonly IContainer _container;

    private readonly ICOnfiguration _configuration;

    private readonly SeviceInfo _serviceInfo;

    private readonly UserInfo _userInfo;

    public AServiceTest()
    {
        _configuration = new ConfigurationBuilder().SetBasePath(Directory.GetCurrentDirectory()).AddJsonFile(Path.Combine(projectRootDir, "appsettings.json"), false, false).Build();
        _userInfo = new UserInfo()
        {
            UserId = "101",
            DeptCode = "999",
            Code = "111",
        };
        _serviceInfo = new ServiceInfo() 
        {
            ServiceDetailId = "RegisterService"
        };
        var builder = new ContainerBuilder();

        builder.RegisterModule<ServiceModule>();
        builder.RegisterModule<ExtensionRepositoryModule>();
        builder.RegisterModule<AutomatedRepositoryModule>();
        buidler.RegisterInstance(_configuration);
        buidler.RegisterInstance(_userInfo)
                .As<UserInfo>();
        buidler.RegisterInstance(_serviceInfo)
                .As<ServiceInfo>();
        builder.Register(C => TestUtil.GetContext())
                .As<MyDbContext>()
                .InstancePerLifetimeScope();

        builder.RegisterType<BuisnessDateMockSerivce>()
                .As<IBusinessDateService>()
                .SingleInstance();

        _container = builder.build();
    }
}

#pragma warning disable SA1402
internal class BuisnessDateMockSerivce : IBusinessDateSharedService
#pragma warning disable SA1402
{
    public string GetBusinessDate(string bankCode, string businessDivison)
    {
        return "2026-05-14";
    }

    public string getBusinessDateTime(string bankCode, string businessDivision)
    {
        return DateTime.Parse("2026-05-14 08:13:13");
    }
}
```

- after building container, we can use that container as a DI tool
    - if we dont provide this kinda container, then we must manually execute DI and it's bordersome
- you must be careful about using direct _container
    - scope is needed, not _container itself
    - because it's unit test, so scope must be in 1 case test
    - so using scope is recommended
    - before executing SQL, truncate all data and add nessasary data

```C#
[Fact]
public void TestCase1004()
{
    using var scope = _container.BeginLifetimeScope();

    var contesxt = scope.Resolve<MyDbContext>;
    var MyService = scope.Resolve<MyService>;

    context.Database.ExecuteSql($"truncate table Table1");
    context.ChangeTracker.Clear();

    context.Table1Entity.AddRange(TestUtil.ReadDataFromCsv<Table1>)("Service/MyService/Data/Table1/Table1_testcase_1004.csv");
    context.SaveChanges();

    MyService.myServiceMethod1("1","2","3");
}
```


## <span style="color:#802548">_Java and C# : UNIT TEST_</span>
- service logic consists of 5 repo and 20 queries with if statement
- but test case of service logic care about just 1 query and doesnt have a concern about service itself
- in other hand, test case is not about service flow so test data not exists
- in this case, Mock must be used 
    - we cannoot implement only used methods
    - so if we dont use it in that case, just thorw exception 

```C#
internal class TestAMockRepository : IARepository
{
    public IList<AEntity> getEnttiy() 
    {
        throw new NotImplementedException();
    }

    public AEntity? getEntityForClosing()
    {
        return new () 
        {
            SequeceNo = 0,
            TransactionNumber = "",
        }
    }
}
```


- as u deciding mock, u must register it in Config
- but in this case, not in constructor Builder, cuz this mockservice is used in specific case

```C#
[Fact]
public void testCase1001()
{
    using var scope = _container.BeginLifetimeScope(builder => {
        builder.RegisterType<TestAMockRepository>()
                .As<IARepository>
                .InstancePerLifetimeScope();
    })
}
```

- if case is for using some mock method and using real repo methods, then below pattern is recommended

```C#
internal class TestCMockRepository : ICUpdateRepository
{
    private readonly CUpdateRepository _realRepository;

    public TestCUpdateMockRepository(
        CUpdateRepository realRepository
    )
    {
        _realRepository = realRepository;
    }

    public void CancelCredit()
    {
        return;
    }

    public bool CancelLoan()
    {
        return false;
    }

    public void UpdateCredit()
    {
        _realRepository.UpdateCredit();
    }
}
```

- then, 2 each service must be registerd in specific test config
- not just Mock, but also real one must be registerd in config
- if not registering, circular dependency injection problem arises

```C#
[Fact]
public void testCase1001()
{
    using var scope = _container.BeginLifetimeScope(builder => {
        builder.RegisterType<TestCMockRepository>()
                .As<ICUpdateRepository>
                .InstancePerLifetimeScope();
        builder.RegisterType<CUpdateRepository>()
                .As<CUpdateRepository>
                .InstancePerLifetimeScope();
    })
}
```




## <span style="color:#802548">_Java and C# : EMPTY CLLECTION_</span>

```C#
Enumerable.Empty<T>();  //Enumerable
ImmutableList<T>.Empty  //List
Array.Empty<T>();       //array
```


## <span style="color:#802548">_Java and C# : MAX() LINQ_</span>

- when there is no matching value and ChangeData entity property is not null, null exception occurs

```C#
DateOnly? maxChangeDate = dbContext.Enttiy.Max(x => x.ChangeDate);
```

- so in order to detour this problem, needs Order by and select first one

```C#
DateOnly? maxChangeDate = dbContext.Enttiy.OrderByDescending(x => x.ChangeDate).Select(x => x.ChangeDate).FirstOrDefault;
```

- and then, u composite that query to another query, which means subquery

```C#
DateOnly? maxChangeDate = dbContext.Enttiy.OrderByDescending(x => x.ChangeDate).Select(x => x.ChangeDate).FirstOrDefault;

var query = dbContext.Enttiy.where(x=> x.ChangeDate == maxChangeDate);
var totalCount = query.Count();
```

- but the problem is that, when maxChangeDate becomes null, these query object would be null
- so below gonna be exception, cuz there is no 1 record

```C#
var result = query.Select(entity => new Entity(){
    SequecneNo = entity.SequenceNo,
    TotalCount = totalCount
}).First();
```

- in this case, we need to use FirstOrDefault()

```C#
var result = query.Select(entity => new Entity(){
    SequecneNo = entity.SequenceNo,
    TotalCount = totalCount
}).FirstOrDefault() 
```

- and null in object is equlas to no count in DB. so we can translate it into like below using ?? opeartor

```C#
var result = query.Select(entity => new Entity(){
    SequecneNo = entity.SequenceNo,
    TotalCount = totalCount
}).FirstOrDefault() ?? new Entity() {
    TotlaCount = 0
}
```





























## <span style="color:#802548">_C# Stream : reading file data_</span>

- reading csv file data


```C#
public class TestUtil {
    public static IList<T> ReadDataFromCsv<T>(string filepath) 
    {
        string projDir = GetProjectRootDir();

        var config = new CsvConfiguration(new CultureInfo("ja-JP"))
        {
            HeaderValidated = null,
            MissingFieldFound = null,
        };
        var dateTimeType = new TypeConverterOptions()
        {
            Formats = ["yyyy-mm-dd HH:mm:ss.SSS", "yyyy-mm-dd HH:mm:ss", "yyyy/mm/dd HH:mm:ss.SSS", "yyyy/mm/dd HH:mm:ss"],
        }

        StreamReader sr = new StreamRead(Path.Combine(projDir, filepath), Encoding.UTF8);
        CsvReader csvReader = new (sr, config);
        csvReader.Context.TypeConverterOptionsCache.AddOptions<DateTime>(dateTypeType);
        csvReader.Context.TypeConverterOptionsCache.getOptions<DateTime?>().NullValues.Add("<nul>");
        csvReader.Context.TypeConverterOptionsCache.getOptions<DateOnly?>().NullValues.Add("<nul>");
        csvReader.Context.TypeConverterOptionsCache.getOptions<string?>().NullValues.Add("<nul>");
        csvReader.Context.TypeConverterOptionsCache.getOptions<int?>().NullValues.Add("<nul>");
        csvReader.Context.TypeConverterOptionsCache.getOptions<long?>().NullValues.Add("<nul>");
        csvReader.Context.TypeConverterOptionsCache.getOptions<decimal?>().NullValues.Add("<nul>");
        csvReader.Context.TypeConverterOptionsCache.getOptions<bool?>().NullValues.Add("<nul>");

        var data = csvReader.GetRecords<T>().ToList();

        sr.Close();

        return data;
    }
}

```


## <span style="color:#802548">_project_</span>


```
***우선 package 구조부터 살펴보기. 
1. 각 프로젝트 별로 있을 수도 like NTT (배치1, 배치2, 배치3, 배치4....)
2. 기능 별로 패키지 나눌 수도 like 日証金 (service, common service, web, repository, batch)

***DB 연결 방법 살펴보기.
repository는 ORM과 SQL mapper 뭘 쓰나 살펴보기.
ORM과 SQL mapper 중 쓸 수 있는 거 보고, 간편하게 쓸 수 있는 지 찾아보기
create, update 등은 많은 컬럼을 나열하는 개노가다 model class 작성작업을 하지 않게끔 ORM을 쓸 수 있다면 금상첨화

***에러 메시지 처리 방법에 대해 알아보기.
error handling이 자동으로 다뤄지는지 ?
자동이라면 어떻게 자동으로 다뤄지는지 ? 

***로그 찍는 법에 대해 알아보기.
로그를 찍으면 어디에 로그가 저장되는지 ?

***테스트 방법에 대해 알아보기.
테스트는 unit test를 사용하는지?
아니면 그냥 화면단 테스트만 진행하는지?

***git 관리에 대해 알아보기.
내가 작성할 브랜치 이름의 명명규칙이 어떻게 되는지?
어떤 브랜치를 기준으로 분기해야하는지?
언제 기준 브랜치로 merge를 해야하는지?
git PR을 날리는지? 아니면 그냥 merge로 전부 처리하는지?
```

