# xUnit

### Setup
```
dotnet new install xunit.v3.templates
dotnet new xunit3
```

### Fact - Fixed test logic - No Parameters

```
[Fact]
public void AddingTwoNumbers_ShouldReturnCorrectSum()
{
    var result = 2 + 3;
    Assert.Equal(5, result);
}
```

### Theory - Test logic with multiple sets of data

```
[Theory]
[InlineData(2, 3, 5)]
[InlineData(10, 5, 15)]
[InlineData(-1, 1, 0)]
public void AddingNumbers_ShouldReturnExpectedSum(int a, int b, int expected)
{
    var result = a + b;
    Assert.Equal(expected, result);
}

```

-----------------------------------------------

# Mocking Dependencies with Moq

## First Example 

### Repositoty
```
public interface IStudentRepository
{
    Student GetById(int id);
}
```

### Service
```
public class StudentService
{
    private readonly IStudentRepository _repo;

    public StudentService(IStudentRepository repo)
    {
        _repo = repo;
    }

    public Student GetStudent(int id)
    {
        if (id <= 0)
            throw new ArgumentException("Invalid ID");

        return _repo.GetById(id) ?? throw new InvalidOperationException("Student not found");
    }
}
```
### Test

```
public class StudentServiceTests
{
    [Fact]
    public void GetStudent_InvalidId_ThrowsArgumentException()
    {
        var mockRepo = new Mock<IStudentRepository>();
        var service = new StudentService(mockRepo.Object);

        Assert.Throws<ArgumentException>(() => service.GetStudent(0));
    }

    [Fact]
    public void GetStudent_NotFound_ThrowsInvalidOperationException()
    {
        var mockRepo = new Mock<IStudentRepository>();
        mockRepo.Setup(r => r.GetById(1)).Returns((Student?)null);

        var service = new StudentService(mockRepo.Object);

        Assert.Throws<InvalidOperationException>(() => service.GetStudent(1));
    }

    [Fact]
    public void GetStudent_ValidId_ReturnsStudent()
    {
        var mockRepo = new Mock<IStudentRepository>();
        mockRepo.Setup(r => r.GetById(1)).Returns(new Student { Id = 1, Name = "Alice" });

        var service = new StudentService(mockRepo.Object);
        var result = service.GetStudent(1);

        Assert.Equal("Alice", result.Name);
    }
}
```
## Second Example - PaymentService that depends on a IPaymentGateway

```
public interface IPaymentGateway
{
    Task<bool> ProcessPaymentAsync(string cardNumber, decimal amount);
}
```

```
public class PaymentService
{
    private readonly IPaymentGateway _gateway;

    public PaymentService(IPaymentGateway gateway)
    {
        _gateway = gateway;
    }

    public async Task<string> PayAsync(string cardNumber, decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentOutOfRangeException(nameof(amount), "Amount must be greater than zero");

        var success = await _gateway.ProcessPaymentAsync(cardNumber, amount);

        return success ? "Success" : throw new InvalidOperationException("Payment failed");
    }
}
```

### Unit Tests with Moq + xUnit
```
public class PaymentServiceTests
{
    [Fact]
    public async Task PayAsync_AmountLessThanOrEqualToZero_ThrowsException()
    {
        var mockGateway = new Mock<IPaymentGateway>();
        var service = new PaymentService(mockGateway.Object);

        await Assert.ThrowsAsync<ArgumentOutOfRangeException>(() => service.PayAsync("1234", 0));
    }

    [Fact]
    public async Task PayAsync_GatewayFails_ThrowsInvalidOperationException()
    {
        var mockGateway = new Mock<IPaymentGateway>();
        mockGateway.Setup(g => g.ProcessPaymentAsync(It.IsAny<string>(), It.IsAny<decimal>()))
                   .ReturnsAsync(false);

        var service = new PaymentService(mockGateway.Object);

        await Assert.ThrowsAsync<InvalidOperationException>(() => service.PayAsync("1234", 100));
    }

    [Fact]
    public async Task PayAsync_ValidPayment_ReturnsSuccessAndCallsGateway()
    {
        var mockGateway = new Mock<IPaymentGateway>();
        mockGateway.Setup(g => g.ProcessPaymentAsync("1234", 100)).ReturnsAsync(true);

        var service = new PaymentService(mockGateway.Object);

        var result = await service.PayAsync("1234", 100);

        Assert.Equal("Success", result);
        mockGateway.Verify(g => g.ProcessPaymentAsync("1234", 100), Times.Once);
    }
}

```
> ProcessPaymentAsync is just an interface method at this point, which means it has no actual implementation. When you use Moq to create a Mock<IPaymentGateway>, you're saying:
> "Pretend we have this dependency, and I’ll tell you how it should behave during the test."
