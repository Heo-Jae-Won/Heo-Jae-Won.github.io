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
- FromSQl's parameter must be a DbSet object
- but it does support db level of projection like specifying selected columns
- but it cannot be used as a DTO projection like SqlQuery


## <span style="color:#802548">_C# module resistration_</span>
- registration order is always like below
- it means, if someone requests Interface, create and provide ImplClass

```C#
builder.RegisterType<-----ImplClass>().As<------InterfaceClass>
```


## <span style="color:#802548">_C# ref_</span>

- in C#, u should try to avoid ref keyword
- in almost cases, ref doesnt be needed
- setter doesnt need ref keyword, cuz copy is passed by parameter

```C#
private void setResponseData(
    ref ResponseModel responseModel,
    CancelInfo cancelInfo
)

setResponseData(ResponseModel responseModel) {
    responseModel.a = a;
    responseModel.b = b;
    responseModel.c = c;
    responseModel.d = d;
}
```

- it can have a meaning if u need to do a new initilazation in one method and after returning, u want to continue logic with that model
- but just setter logic already passes address as a parameter, so it is no effect

```C#
void SetResponseData(ResponseModel respponseModel) {
    respponseModel = new ResponseModel();
}

public ServiceLogic() {
    ResponseModel responsModel = new ResponseModel();
    setResponseData(responseModel);

    repoCall(responseModel);
}
```

- and it's better to return a new initialized model rather than ref

```C#
private ResponseModel setResponseDataAndReturnModel(ResponseModel responseModel) {
    responseModel = new ResponseModel();
    responseModel.a = a;
    ...

    return responseModel
}

public ServiceLogic() {
    ResponseModel responsModel = new ResponseModel();
    responsModel = setResponseDataAndReturnModel(responseModel);

    repoCall(responseModel);
}
```

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












## <span style="color:#802548">_C# dbContext : rowversion_</span>

- for autogenerating some rows in efcore dbcontext, dbcontext must have modelBuilder, which means model configuration

```C#
modelBuilder.Entity<Enttiy>(enttiy => {
    .
    .

})
```

- and for being possible to autogenerate, .IsRowVersion() is nessacary 
- .IsConcureencyToken() is for optimistic concurrency check

```C#
entity.Property(e => e.RowVersion
                        .IsRowVersion()
                        .IsConcurrencyToekn()
                        .HasColumnName("ROW_VERSION"));
```

- then, after craeting entity, entity object returns new autogenerated value from DB

```C#
Entity entity = _repository.CreateEntity();

_.update(entity.RowVersion);
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

## <span style="color:#802548">_C# dbcontext : scoped_</span>
- DBcontext is basically scoped
    - scoped means new instance for every request.0
- if not scoped, there would be some serious problems
- when dbContext is not scoped, they share same dbcontext
- and it will cause problems even repoo doesnt touch shared object
- cuz DbContext is not thread-safe, so state in dbContext would be corrupted
    - both attempts tracker and identity map dictionary 

```C#
await Task.WhenAll(
    repo.GetUserAsync(1),
    repo.GetUserAsync(2),
)
```

- buf even if scoped, above code is dangerous



- right form of pararellism is like below
- not dbContext, but IDbContextFactory in repo

```C#
public class UserRepository : IUserRepository
{
    private readonly IDbContextFactory<AppDbContext> _contextFactory;

    public UserRepository(IDbContextFactory<AppDbContext> contextFactory)
    {
        _contextFactory = contextFactory;
    }

    public async Task<User> GetUserAsync(int id)
    {
        using var context = await _contextFactory.CreateDbContextAsync();
        return await context.Users.FindAsync(id);
    }
}
```

- this is service logic

```C#
public class UserSerivce 
{
    public async Task<(List<User> Users, List<Order> Orders)> getUserAsync(int id) 
    {
        using var scope1 = scopeFactory.CreateScope();
        using var scope2 = scopeFactory.CreateScope();

        varw db1 = scope1.ServiceProvider.GetSerivce1<AppDbContext>();
        varw db1 = scope2.ServiceProvider.GetSerivce2<AppDbContext>();

        var usersTask = db1.User.toListAsync();
        var ordersTask = db2.Orders.toListAsync();

        await Task.whenAll(usersTask, ordersTask);

        /*var users = usersTask.Result; 
        var orders = ordersTask.Result;

        return (users, orders);*/

        return (await usersTask, await ordersTask)
    }
}
```

- shared ORM session across threads is unsafe



## <span style="color:#802548">_efficient sql usage_</span>

- at first, source code is follwed as below
- it causes so many network call

```C#
string[] codeList = ["001", "002", "003"];
for(string fundCode in codeList) 
{
    select * from enttiy
    where code = fundCode
    and changeDate = (SELECT MAX(Change_date) from entity
    where code = fundCode and change_date <= '2025-07-13')
    and isDelete = false;
}
```

- so i change it to union all
- but it cannot solve so many table scan problem

```sql
select * from enttiy
where code = '001'
and changeDate = (SELECT MAX(Change_date) from entity
where code = '001' and change_date <= '2025-07-13')
and isDelete = false;

union all

select * from enttiy
where code = '002'
and changeDate = (SELECT MAX(Change_date) from entity
where code = '002' and change_date <= '2025-07-13')
and isDelete = false;

union all

select * from enttiy
where code = '003'
and changeDate = (SELECT MAX(Change_date) from entity
where code = '003' and change_date <= '2025-07-13')
and isDelete = false;
```

- if want to keep max(), need group by
- then table scan decreases to 2 time
- but filter logic is in 2 places

```sql
with latest as (
    SELECT 
        code,
        max(Change_date) as max_change_date
        from entity
        where code in ('001','002','003')
        and chnage_date <= '2025-07-13'
        and isDeleted = false
        GROUP BY code
)

select t.*
from entity t
JOIN latest i
on t.code = i.code
and t.change_date = i.max_changed_date
where t.isDelete = false;
```

- we can minimize table scan to 1 time, using window function
- filter logic is only in CTE
- furthermore, window function is Highly Maintainable
    -  If you ever need to fetch the "second latest" or "top 3" rows instead, you simply change rn = 1 to rn <= 3

```sql
WITH ranked AS (
    SELECT 
        t.*,
        ROW_NUMBER() OVER (
            PARTITION BY code
            order by change_date desc
        ) as rn
    FROm entity t
    where code in (
        '001',
        '002',
        '003'
    )
    and change_date <= '2025-07-13'
    and isDeleted = false
)
select *
from ranked
where rn = 1;
```


- but ROW_NUMBER has a serious problem
- when row has value that change_date is same, then only 1 record is retrieved
- so, business logic is broken
- in this case, dense_rank is needed
    - we can prevent only 1 record per code problem
    - we can get all records per code, using dense_rank()

```sql
WITH ranked AS (
    SELECT 
        t.*,
        DENSE_RANK() OVER (
            PARTITION BY code
            ORDER BY change_date DESC
        ) AS rnk
    FROM entity t
    WHERE code IN ('001', '002', '003')
      AND change_date <= '2025-07-13'
      AND isDeleted = false
)
SELECT *
FROM ranked
WHERE rnk = 1;
```

- finally index is set
- for setting index, we must remember "Equality, Sort, Range/Inequality" Rule 
    - = is first, and then desc or asc and then <=, >=
        - equality(=) jumps straight down the B-Tree branches to the exact subset of data matching that value
        - if we make a sortted index, we dont need to operated high-cpu sorting step
        - Continuous Stream(<= , >=) makes less scan
- my query has below condition

```text
code = ... (Equality)
isDeleted = false (Equality)
change_date DESC (Sort)
change_date <= '2025-07-13' (Range / Inequality)
```

- why (code, isDeleted, change_date)'s index is superior than (code, change_date, isDeleted)?
    - Inside the bucket for code = '001', the rows are sorted by change_date
    - let's suppose that we set a index (code, change_date, isDelete)
    - in condition of change_date <= '2025-07-13', db scan across multiple different dates to find all matching entries
    - (code, isDeleted, change_date) make sure efficient operation of index scan

```sql
CREATE INDEX idx_entity_lookup 
ON entity (code, isDeleted, change_date DESC);
```



## <span style="color:#802548">_efficient sql usage2_</span>

- 페이지 IO 차이??

## <span style="color:#802548">_chunk process batch_</span>
- using chunk for batching is needed
- chunk must be processed in each respective transaction
- for utilizing chunk process merit, followed one is bad practice

```C#
using (var scope = new TransactionScope(
    TransactionScopeOption.Required, 
    new TransactionOptions { IsolationLevel = IsolationLevel.ReadCommitted },
    TransactionScopeAsyncFlowOption.Enabled))
{
    foreach (var chunk in chunksOf4900)
    {
        context.MyTable.AddRange(chunk);
        context.SaveChanges(); 
    }
    transaction.Commit(); 
}
```

- this is needed
- transaction must be executed in foreach statement

```C#
foreach (var chunk in chunksOf4900)
{
    using (var scope = new TransactionScope(
        TransactionScopeOption.Required, 
        new TransactionOptions { IsolationLevel = IsolationLevel.ReadCommitted },
        TransactionScopeAsyncFlowOption.Enabled))
    {
        context.MyTable.AddRange(chunk);
        context.SaveChanges(); // Writes to database
        
        transaction.Commit(); // 💥 Instantly frees the 4,900 locks from SQL Server RAM!
    }
    
    // Crucial for EF Core: Clears C# tracking memory so your app doesn't slow down
    context.ChangeTracker.Clear(); 
}
```

- this is more complexed example
    - after inserting, saveChanges()
    - after updating, saveChanges()
- why execute SaveChanges() for each DML?
    - cuz it free early memory taken up by DML

```C#
// Loop through your data list
foreach (var item in List)
{
    // Open a fresh micro-transaction for this specific item/chunk
    using (var scope = new TransactionScope(
        TransactionScopeOption.Required, 
        new TransactionOptions { IsolationLevel = IsolationLevel.ReadCommitted },
        TransactionScopeAsyncFlowOption.Enabled))
    {
        try
        {
            _serviceA.Insert(item);
            context.SaveChanges(); // Pushes insert to DB (holds temporary tiny lock)

            _serviceA.Update(item);
            context.SaveChanges(); 

            transaction.Commit(); // 💥 Instantly frees the locks from SQL Server RAM!
        }
        catch (Exception ex)
        {
            // 4. IF EITHER FAILS, rollback this specific item's insert and update completely
            transaction.Rollback(); 
            
            
            throw;
        }
    }

    context.ChangeTracker.Clear();
}
```

- but problem is that, above one has a serious problem in one or nothing ACID
- so for one or nothing transaction, we need chunk process toward temp table
- and if it fails, delete all recorded
- if all chunk process success, then now insert all things to original table

```C#
Guid currentBatchId = Guid.NewGuid();

try
{
    // STAGE 1: Fast, safe, chunked insertion into the Staging Table
    foreach (var chunk in chunksOf4900)
    {
        using (var transaction = context.Database.BeginTransaction())
        {
            foreach(var item in chunk) { item.BatchId = currentBatchId; }
            
            context.StagingItems.AddRange(chunk);
            context.SaveChanges();
            transaction.Commit(); // 💥 Frees locks from RAM instantly!
        }
        context.ChangeTracker.Clear();
    }

    // STAGE 2: Atomic, all-or-nothing move to the Live production table
    using (var liveTransaction = context.Database.BeginTransaction())
    {
        await context.Database.ExecuteSqlRawAsync(@"
            INSERT INTO LiveItems (Name, Status)
            SELECT Name, 'Processed' 
            FROM StagingItems 
            WHERE BatchId = {0}", currentBatchId);

        await context.Database.ExecuteSqlRawAsync(
            "DELETE FROM StagingItems WHERE BatchId = {0}", currentBatchId);

        liveTransaction.Commit();
    }
}
catch (Exception ex)
{
    _logger.LogError(ex, "Processing failed. Cleaning up staging data.");
    
    await context.Database.ExecuteSqlRawAsync(
        "DELETE FROM StagingItems WHERE BatchId = {0}", currentBatchId);
    
    throw;
}
```

- for second thing to consider is transaction scope
- if in that service also has BeginTransaction, then orchestrator pattern is needed

```C#
// The loop manages the single active transaction scope
foreach (var item in List)
{
    using (var transaction = context.Database.BeginTransaction()) // ONE transaction
    {
        try
        {
            _serviceA.Insert(item); // Runs seamlessly inside the transaction
            _serviceA.Update(item); // Runs seamlessly inside the transaction

            transaction.Commit();   // Commits BOTH. Instantly frees RAM locks!
        }
        catch
        {
            transaction.Rollback(); // Rolls back BOTH safely!
            throw;
        }
    }
    context.ChangeTracker.Clear();
}
```

- if u cannot, TransactionScope is needed
- it's like @Transactional in Spring
- Transaction Propagation
    - Spring detects the existing transaction, skips creating a new one, and forces the inner method to seamlessly join the outer one

```C#
using System.Transactions; // Required namespace

foreach (var item in List)
{
    // Outer Scope: Enforces that EVERYTHING inside rolls back together
    using (var outerScope = new TransactionScope(TransactionScopeOption.Required))
    {
        try
        {
            // Even if Insert() has an internal transaction, TransactionScope 
            // intercepts it and suppresses the EF Core nesting error!
            _serviceA.Insert(item); 
            _serviceA.Update(item); 

            outerScope.Complete(); // Commits everything at once and frees RAM
        }
        catch
        {
            // Auto-rolls back both steps if an exception occurs
            throw; 
        }
    }
    context.ChangeTracker.Clear();
}
```

- for async/await, TransactionScope can lose track of the transaction
- pass TransactionScopeAsyncFlowOption.Enabled 

```C#
using (var scope = new TransactionScope(
    TransactionScopeOption.Required, 
    new TransactionOptions { IsolationLevel = IsolationLevel.ReadCommitted },
    TransactionScopeAsyncFlowOption.Enabled))
{
    await _context.SaveChangesAsync();
    scope.Complete();
}
```


## <span style="color:#802548">_efficient sql usage3_</span>

- 

```sql
select U.* from
(
    VALUES(1,20),(3,40)
) V (USER_ID, AGE)
JOIN USERS U 
    ON U.USER_ID = V.USER_ID
    AND U.AGE = V.AGE
```




## <span style="color:#802548">_BeginTransaction problem in C# efcore_</span>


- or Alternative 2: MediatR + Pipeline Behaviors (The Modern C# Architectural Standard)

```C#
public class TransactionBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    private readonly MyDbContext _context;
    public TransactionBehavior(MyDbContext context) => _context = context;

    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken cancellationToken)
    {
        // Automatically injects a transaction wrapper around any command handlers!
        using var transaction = await _context.Database.BeginTransactionAsync();
        try
        {
            var response = await next(); // Executes your actual service logic
            await transaction.CommitAsync();
            return response;
        }
        catch
        {
            await transaction.RollbackAsync();
            throw;
        }
    }
}
```

 [ Web Request ] 
        │
        ▼
 1. CONTROLLER ────────► Simply receives the HTTP request and sends a command packet.
        │
        ▼
 2. PIPELINE BEHAVIOR ─► Intercepts the packet, automatically calls `.BeginTransactionAsync()`.
        │
        ▼
 3. SERVICE HANDLER ───► Executes your Insert() and Update() code (100% clean of transaction logic).
        │
        ▼
 2. PIPELINE BEHAVIOR ─► Detects success, automatically calls `.CommitAsync()`, and frees SQL Server locks!
        │
        ▼
 [ HTTP Response ]


```C#
[ApiController]
[Route("api/items")]
public class ItemsController : ControllerBase
{
    private readonly IMediator _mediator;
    public ItemsController(IMediator mediator) => _mediator = mediator;

    [HttpPost]
    public async Task<IActionResult> Process([FromBody] ProcessItemCommand command)
    {
        // The controller just passes the data to MediatR and forgets about it.
        var result = await _mediator.Send(command);
        return Ok(result);
    }
}
```

```C#
public class ProcessItemCommandHandler : IRequestHandler<ProcessItemCommand, bool>
{
    private readonly ServiceA _serviceA;
    public ProcessItemCommandHandler(ServiceA serviceA) => _serviceA = serviceA;

    public async Task<bool> Handle(ProcessItemCommand request, CancellationToken cancellationToken)
    {
        // 100% focused on business rules. No try/catch loops. No commit/rollback calls.
        // If anything fails here, Layer 2 will catch it and roll back automatically.
        await _serviceA.InsertAsync(request.Item);
        await _serviceA.UpdateAsync(request.Item);
        
        return true; 
    }
}
```

