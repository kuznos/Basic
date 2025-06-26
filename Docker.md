### Build the Docker Image
```
docker build -t myapi:latest .
```

### Run the Container
```
docker run -d -p 5000:8080 --name myapi-container myapi:latest
// -p <host_port>:<container_port>
// 5000 is the port on your host machine (your laptop or server)
// 8080 is the port your app listens on inside the container
```

### Stop and Remove the Container
```
docker stop myapi-container
docker rm myapi-container
```

### Image to Docker Hub
```
docker login
docker tag myapi:latest yourusername/myapi:latest
docker push yourusername/myapi:latest
```

### Pull Docker Image
```
docker pull yourusername/myapi:latest
```

## Using .env file for Configuration

### .env File (in project root)
```
ASPNETCORE_ENVIRONMENT=Production
MySecretKey=abc123
ConnectionStrings__Default=Server=localhost;Database=MyDb;Trusted_Connection=True;
// Use __ (double underscores) to represent nested configuration in .NET (e.g., ConnectionStrings:Default)
```

// Reference It When Running Docker
```
docker run -d \
  -p 5000:8080 \
  --env-file .env \
  --name myapi-container \
  yourusername/myapi:latest
```

### Access Environment Variables in .NET
```
var secret = builder.Configuration["MySecretKey"];
var connectionString = builder.Configuration.GetConnectionString("Default");
```

