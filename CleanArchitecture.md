
```mermaid
graph TD
A[Solution] --> B[src]
A --> C[tests]

B --> B1[API]
B --> B2[Core]
B --> B3[Infrastructure]
B --> B4[UI]

B2 --> B2a[Domain]
B2 --> B2b[Application]

B3 --> B3a[Persistence]
B3 --> B3b[Infrastructure]
```

# References

### Application
* **Domain Project**
> * AutoMapper
> * FluentValidation
> * FluentValidation.DependencyInjectionExtensions
> * MediatR
> * Microsoft.Extensions.Logging.Abstraction
  
### Persistence
* **Application Project**
> * Microsoft.EntityFrameworkCore.SqlServer
> * Microsoft.Extensions.Options.ConfigurationExtensions

### Infrastructure
* **Application Project**
> * Microsoft.Extensions.Http
> * Microsoft.Extensions.Options.ConfigurationExtensions
> * System.ServiceModel.Http
  
### API
* **Application Project**
* **Persistence Project**
* **Infrastructure Project**
> * Asp.Versioning.Mvc.ApiExplorer
> * MediatR
> * Microsoft.Data.SqlClient
> * Microsoft.EntityFrameworkCore.Tools
> * Serilog
