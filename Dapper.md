# DAPPER
--------------------------------------

# SELECT

## Select All
```
private const string connectionString = $@"Server=;Database=;User Id=;Password=;TrustServerCertificate=True";
public static IEnumerable<Student> GetAllStudents()
{
    List<Student>? students;
    var connection = new SqlConnection(connectionString);
    var sql = "SELECT Id, Name, Birthdate as BirthdateX FROM Persons.dbo.Student WITH (nolock)";
    students = connection.Query<Student>(sql).ToList();
    return students;
}
```
## Select by Id
```
public static IEnumerable<Student> GetStudentById(int id)
{
    List<Student>? students;
    var connection = new SqlConnection(connectionString);
    var sql = "SELECT Id, Name, Birthdate as BirthdateX FROM Persons.dbo.Student WITH (nolock) WHERE StudentId = @Id";
    students = connection.Query<Student>(sql, new {StudentId = id}).ToList();
    return students;
}
```

## QueryMultipleAsync
```
var connection = new SqlConnection(connectionString);
string sql = @"
SELECT * FROM Persons.dbo.Student WHERE ID = @StudentId;
SELECT * FROM Persons.dbo.StudentClass WHERE StudentId = @StudentId;
";
using (var multi = await connection.QueryMultipleAsync(sql, new {orderID = 1}))
{
   var student = multi.ReadFirst<Student>();
   var StudentClasses = multi.Read<StudentClass>().ToList();
}
```

## Fix Datetime to Dateonly using DateOnlyHandler
```
public class DateOnlyHandler : SqlMapper.TypeHandler<DateOnly>
{
    public override void SetValue(IDbDataParameter parameter, DateOnly value)
        => parameter.Value = value.ToDateTime(TimeOnly.MinValue);

    public override DateOnly Parse(object value)
        => DateOnly.FromDateTime((DateTime)value);
}
```

### Register Dapper type handler
```
SqlMapper.AddTypeHandler(new DateOnlyHandler());
```

-------------------------------------------

# STORED PROCEDURES
```
using (var connection = new SqlConnection("connectionString"))
{
    connection.Open();
    using (var results = connection.QueryMultiple("MyStoredProcedure", commandType: CommandType.StoredProcedure))
    {	
        var resultSet2 = results.Read<Student>().ToList();
    }
}
```
### Parameters
```
using(var connection = new SqlConnection(connectionString))
{
    connection.Open();
    //Set up DynamicParameters object to pass parameters  
    DynamicParameters parameters = new DynamicParameters();   
    parameters.Add("Id", 1);  
	
    //Execute stored procedure and map the returned result to a Customer object  
    var student = conn.QuerySingleOrDefault<Student>("GetStudentById", parameters, commandType: CommandType.StoredProcedure);
}

// or with values for EXEC GetStudentsByYear @BeginningDate, @EndingDate

var storedProcedureName = "GetStudentsByYear";
var values = new { BeginningDate = "2017-01-01", EndingDate = "2019-12-31" };
var results = connection.Query(storedProcedureName, values, commandType: CommandType.StoredProcedure).ToList();
results.ForEach(r => Console.WriteLine($"{r.Id} {r.Name}"));
```

--------------------------------------

# INSERT

### Executing Non-Query
```
using(var connection = new SqlConnection(connectionString))
{
    connection.Open();
    string sql1 = "INSERT INTO Student (Name, Birthdate) VALUES (@Name, @Birthdate)";		
    await connection.ExecuteAsync(sql1, Student);
}
```
### Executing Scalar
```
using(var connection = new SqlConnection(connectionString))
{
    connection.Open();
    string sql1 = "INSERT INTO Student (Name, Birthdate) VALUES (@Name, @Birthdate)";		
    await connection.ExecuteScalarAsync(sql1, Student);
}
```

# UPDATE
```
using(var connection = new SqlConnection(connectionString))
{
    connection.Open();
    string sql1 = "UPDATE Student SET Name = Name + @suffix WHERE StudentId = @Id";		
    connection.ExecuteAsync(sql1, new { @Id = 1, suffix = " Test" });
}
```

# DELETE
```
using(var connection = new SqlConnection(connectionString))
{
    connection.Open();
    string sql1 = "DELETE FROM Student WHERE StudentId = @Id";		
    connection.ExecuteAsync(sql1, new { @Id = 1 });
}
```

----------------------------------------------------------------------
# Dapper with the Repository Pattern
----------------------------------------------------------------------

### Entity
```
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

### Repository Interface
```
public interface IStudentRepository
{
    Task<IEnumerable<Student>> GetAllAsync();
    Task<Student?> GetByIdAsync(int id);
    Task AddAsync(Student student);
}
```

### Implement the Repository with Dapper
```
public class StudentRepository : IStudentRepository
{
    private readonly IDbConnection _db;

    public StudentRepository(IConfiguration config)
    {
        _db = new SqlConnection(config.GetConnectionString("DefaultConnection"));
    }

    public async Task<IEnumerable<Student>> GetAllAsync()
    {
        var sql = "SELECT * FROM Students";
        return await _db.QueryAsync<Student>(sql);
    }

    public async Task<Student?> GetByIdAsync(int id)
    {
        var sql = "SELECT * FROM Students WHERE Id = @Id";
        return await _db.QueryFirstOrDefaultAsync<Student>(sql, new { Id = id });
    }

    public async Task AddAsync(Student student)
    {
        var sql = "INSERT INTO Students (Name) VALUES (@Name)";
        await _db.ExecuteAsync(sql, student);
    }
}
```

### Register in Program.cs
```
builder.Services.AddScoped<IStudentRepository, StudentRepository>();
```

### Controller
```
[ApiController]
[Route("api/[controller]")]
public class StudentsController : ControllerBase
{
    private readonly IStudentRepository _repo;

    public StudentsController(IStudentRepository repo)
    {
        _repo = repo;
    }

    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        var students = await _repo.GetAllAsync();
        return Ok(students);
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> Get(int id)
    {
        var student = await _repo.GetByIdAsync(id);
        return student is null ? NotFound() : Ok(student);
    }

    [HttpPost]
    public async Task<IActionResult> Create(Student student)
    {
        await _repo.AddAsync(student);
        return CreatedAtAction(nameof(Get), new { id = student.Id }, student);
    }
}
```

## Full example 
[Clean architecture example with Dapper](https://github.com/sandeepkumar17/CleanArch)


