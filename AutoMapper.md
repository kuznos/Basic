## Install Nuget Package
```
dotnet add package AutoMapper
```

## Entities
```
public class Employee {
    public string Name { get; set; }
    public int Salary { get; set; }
    public string Department { get; set; }
}

public class EmployeeDTO {
    public string Name { get; set; }
    public int Salary { get; set; }
}
```

## Create a Mapping Profile
```
using AutoMapper;
public class MappingProfile : Profile {
    public MappingProfile() {
        CreateMap<Employee, EmployeeDTO>();
    }
}
```

## Register AutoMapper in Program.cs
```
builder.Services.AddAutoMapper(typeof(Program));
```

## Use AutoMapper in Your Code
```
public class EmployeeService {
    private readonly IMapper _mapper;

    public EmployeeService(IMapper mapper) {
        _mapper = mapper;
    }

    public EmployeeDTO GetEmployeeDto(Employee employee) {
        return _mapper.Map<EmployeeDTO>(employee);
    }
}
```

-----------------------------------------------------

## Using Config

```
//Create and Populate Employee Object i.e. Source Object
Employee emp = new Employee
{
    Name = "James",
    Salary = 20000,
    Address = "London",
    Department = "IT"
};
            
// Initializing AutoMapper
var mapper = MapperConfig.InitializeAutomapper();

//Way1: Specify the Destination Type and to The Map Method pass the Source Object
//Now, empDTO1 object will having the same values as emp object
var empDTO1 = mapper.Map<EmployeeDTO>(emp);

//Way2: Specify the both Source and Destination Type 
//and to The Map Method pass the Source Object
//Now, empDTO2 object will having the same values as emp object
var empDTO2 = mapper.Map<Employee, EmployeeDTO>(emp);
```

## When Property Names are Different

```
using AutoMapper;
namespace AutoMapperDemo
{
    public class MapperConfig
    {
        public static Mapper InitializeAutomapper()
        {
            //Provide all the Mapping Configuration
            var config = new MapperConfiguration(cfg =>
            {
                //Configuring Employee and EmployeeDTO
                cfg.CreateMap<Employee, EmployeeDTO>()

                //Provide Mapping Configuration of FullName and Name Property
                .ForMember(dest => dest.FullName, act => act.MapFrom(src => src.Name))
                
                //Provide Mapping Dept of FullName and Department Property
                .ForMember(dest => dest.Dept, act => act.MapFrom(src => src.Department));

                //Any Other Mapping Configuration ....
            });

            //Create an Instance of Mapper and return that Instance
            var mapper = new Mapper(config);
            return mapper;
        }
    }
}
```
