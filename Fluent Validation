## Fluent Validation
### dotnet add package FluentValidation

```
public class Library
{
    ...
    public List<Book> Books { get; set; }
}
```

## Examples

```
public class AddressValidator : AbstractValidator<Address>
{
    public AddressValidator()
    {
        RuleFor(address => address.Street)
          .NotEmpty()
          .WithMessage("Street is required.");

        RuleFor(address => address.ZipCode)
          .NotEmpty()
          .Matches(@"^\d{5}(-\d{4})?$")
          .WithMessage("Invalid ZIP code.");
        // 5-digit number, optionally followed by a hyphen and another 4 digits (common US ZIP code format)
    }
}
```
```
public class LibraryValidator : AbstractValidator<Library>
{
    public LibraryValidator()
    {
        RuleFor(library => library.Address)
          .SetValidator(new AddressValidator());

        RuleFor(library => library.PhoneNumber)
          .NotEmpty()
          .Matches(@"^\d{10}$")
          .WithMessage("Invalid phone number.");
        // phone number matches a specific format, specifically 10 digits long

        RuleFor(library => library.Email).EmailAddress();

        RuleFor(library => library.Books)
          .NotEmpty()
          .Must(b => b.Count >= 5)
          .WithMessage("At least 5 books are required.");

    }
}

```
public class BookValidator : AbstractValidator<Book>
{
    public BookValidator()
    {
        RuleFor(book => book.BookId)
          .NotEmpty()
          .Must(BeAValidBookId)
          .WithMessage("Book ID must be in the format 'Letter-Numbers'.");

        RuleFor(book => book.Title)
          .NotEmpty()
          .Length(2, 100);

        // Conditional validation
        When(book => book.Title.Length > 50, () =>
        {
            RuleFor(book => book.Author)
              .NotEmpty()
              .WithMessage("Author is required for long titles.");
        });
    }
    ### Custom Validator
    private bool BeAValidBookId(string bookId)
    {
        if (string.IsNullOrWhiteSpace(bookId))
        {
            return false;
        }

        // Regular expression to match the format "Letter-Numbers"
        var regex = new Regex(@"^[A-Za-z]-\d+$");

        return regex.IsMatch(bookId);
    }
}
```
## Using the Validators
public void ValidateAndCreateLibrary(Library library)
{
    var validator = new LibraryValidator();
    var results = validator.Validate(library);

    if (!results.IsValid)
    {
        foreach (var failure in results.Errors)
        {
            Console.WriteLine($"Error: {failure.ErrorMessage}");
        }
        throw new ValidationException("Validation failed for the library object.");
    }

    // Further processing
}
