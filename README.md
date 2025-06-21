# VS Code

## Create the solution
```
dotnet new sln -n Demo
```
## Add Web Api project to solution
```
dotnet new sln -n Demo
dotnet new webapi --use-controllers -n Demo.Api --framework net8.0
dotnet sln Demo.sln add Demo.Api/Demo.Api.csproj
```
## Add library class project from Core Folder
```
mkdir Core
dotnet new classlib -n Demo.Core --framework net8.0 -o Core/Demo.Core
dotnet sln Demo.sln add Core/Demo.Core/Demo.Core.csproj
```
## Add the Reference
```
dotnet add Demo.Api/Demo.Api.csproj reference Core/Demo.Core/Demo.Core.csproj
```
