# Dapper

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

## Stored Pricedures
using (var connection = new SqlConnection("connectionString"))
{
    connection.Open();
    using (var results = connection.QueryMultiple("MyStoredProcedure", commandType: CommandType.StoredProcedure))
    {	
        var resultSet2 = results.Read<Student>().ToList();
    }
}
