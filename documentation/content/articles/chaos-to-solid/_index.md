+++
title = "Chaos to SOLID"
description = ""
+++

This article is a journey on the path from chaos to SOLID.

Code can be found on Github at https://github.com/codenesium/ASP.NET-Core-Examples/tree/master/ChaosToSOLID

### What is chaos?

* Chaos is a project that everyone is scared to touch and requires a week to add a field.  
* It's no tests or even worse bad tests that frustrate developers, slow the development process and don't help quality.
* It's late nights fixing bugs that should have been caught with tests.
* It's fixing the same bug over and over because it keeps getting reintroduced. 
* It's a unit test suite that takes hours to run and is constantly broken by changes in other parts of the system.
* It's the wall on a new project that you hit when the quality issues start dragging you down. 


We're going to take a set of requirements and implement the project multiple times as we move from a naive attempt to a complete solution. Along the way we will
explore how to unit test and various tools that make testing easier. Our stack is ASP.NET Core. 


### What is SOLID?

This is straight from [Wikipedia](https://en.wikipedia.org/wiki/SOLID)

##### Single responsibility principle
a class should have only a single responsibility (i.e. changes to only one part of the software's specification should be able to affect the specification of the class).
##### Open/closed principle
"software entities … should be open for extension, but closed for modification."
##### Liskov substitution principle
"objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program." See also design by contract.
##### Interface segregation principle
"many client-specific interfaces are better than one general-purpose interface."[4]
##### Dependency inversion principle
one should "depend upon abstractions, [not] concretions."

But what do these things mean in practice?

What happens when you don't write SOLID code?

"a class should have only a single responsibility" sounds simple but what is a responsibility? 

Responsibility could mean interface with the database. Does that mean you're SOLID if you have a repository class with 500 methods?
By applying SOLID principles to your software you can accommodate change and be confident that the software you write today will work in the future.



#### The task
You are a software engineer at a bank and you have been tasked with building a service that runs on the backend.

The web service that runs the bank website will call your web service with details to create the account in the database. There are
two methods you need to implement. They need a method to create an account and a method to retrieve it by id. As you can already guess
these are just methods you would see on a REST API. We're going to focus on these two methods for the duration of the project.


#### Iteration 1
```
[Route("api/[controller]")]
[ApiController]
public class AccountControllerV1 : ControllerBase
{
	/// <summary>
	/// This system runs as a service in a bank. The website backend that our customers use enter
	/// their account information and then that service calls our service to actually create the accounts for the customers.
	/// Create a method that takes an account name and customer id and creates an account for them.
	/// </summary>
	/// <param name="account"></param>
	/// <returns></returns>
	[HttpPost]
	public IActionResult Create(Account account)
	{
		if(string.IsNullOrWhiteSpace(account.Name))
		{
			return new StatusCodeResult((int)System.Net.HttpStatusCode.UnprocessableEntity);
		}

		if (string.IsNullOrWhiteSpace(account.Name))
		{
			return new StatusCodeResult((int)System.Net.HttpStatusCode.UnprocessableEntity);
		}

		if (string.IsNullOrWhiteSpace(account.SSN))
		{
			return new StatusCodeResult((int)System.Net.HttpStatusCode.UnprocessableEntity);
		}

		using (var context = new ApplicationDbContext())
		{
			if (context.Accounts.Any(a => a.Name == account.Name))
			{
				return new StatusCodeResult((int)System.Net.HttpStatusCode.UnprocessableEntity);
			}
			account.DateCreated = DateTime.Now;
			context.Accounts.Add(account);
			context.SaveChanges();
			return Ok(account);
		}
	}
}
```

And the unit tests too because someone read on a blog that unit tests are good. 

```
[Fact]
public void Test_Account_Create()
{
	AccountControllerV1 controller = new AccountControllerV1();

	IActionResult result = controller.Create(new Account() { Id = 1, Name = "Test", SSN="078-05-1120" });
	Assert.True(result is OkObjectResult);
	var response = (result as OkObjectResult);
	var responseItem = response.Value as Account;
	Assert.True(responseItem.Id == 1);
	Assert.True(responseItem.Name == "Test");
	Assert.True(responseItem.SSN == "078-05-1120");
	Assert.True(responseItem.DateCreated != null);
}

```

Technically this satisfies our requirements. We have 100% test coverage. We're even validating the input because we're good developers.

I want to express to you that this will lead to chaos...let me explain why.

This code is not testable. It will never be testable written like this and it will devolve into a mess over time as requirements change. 

This sounds like a contradiction because we clearly have a test.

1. Our method is validating and interacting with the database. It's easy to understand now. Imagine that this table grows to require 20 fields. That's not a unrealistic number. Now imagine how much validation is going to be required on those 20 fields. At least 20 field type checks and then any database validation required for your business rules. How many lines can be put in this method? 100? 500? At that point how easy would it be to go make a change in the method and be very confident you didn't break anything?

2. We're not testing the individual things inside the create method. We're testing all of it. Because we don't really have a choice. A failure in validation or in the database is all the same to this test. What's the difference between an empty name error and a record already exists error? How would we even set those tests up? We would have to create a model for each test while having to think about the various rules that have to pass to test the one thing we want to test. We can mock the Entity Framework context because .NET core is awesome but that's not a unit test. Do we really want to deal with EntityFramework every time we need to test this method?

3. This test is testing that all of the fields are correct. That means when we change or add a field we're going to have to update every one of these tests. There could be lots and lots of tests. Good luck with that. This is a violation of [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 

3. We're creating our Account model with inline object initialization. This leads to bugs because when we add a field to this class we won't get a compile time error that we're not setting the new fields. This is a violation of the open/closed principle.  

4. We're relying on DateTime.Now to set the dateCreated field. How can we test that?


#### Iteration 2

In this iteration we're going to change our controller to use dependency injection. This is application of the D in SOLID. Classes should have their dependencies injected so that the functionality of the class can be verified with mocks. 

```
/// <summary>
/// We've changed our account controller to use an injected database context.
/// </summary>
[Route("api/[controller]")]
[ApiController]
public class AccountControllerV2 : ControllerBase
{
	private ApplicationDbContext context;
	public AccountControllerV2(ApplicationDbContext context)
	{
		this.context = context;
	}
	[HttpPost]
	public IActionResult Create(Account account)
	{
		if (string.IsNullOrWhiteSpace(account.Name))
		{
			return new StatusCodeResult((int)System.Net.HttpStatusCode.UnprocessableEntity);
		}

		if (context.Accounts.Any(a => a.Name == account.Name))
		{
			return new StatusCodeResult((int)System.Net.HttpStatusCode.UnprocessableEntity);
		}

		if (string.IsNullOrWhiteSpace(account.SSN))
		{
			return new StatusCodeResult((int)System.Net.HttpStatusCode.UnprocessableEntity);
		}

		account.DateCreated = DateTime.Now;
		this.context.Accounts.Add(account);
		this.context.SaveChanges();
		return Ok(account);
	}
}
```

```
[Fact]
public void Test_Account_Create()
{

	DbContextOptionsBuilder optionsBuilder = new DbContextOptionsBuilder();
	optionsBuilder.UseInMemoryDatabase("test1");
	var context = new ApplicationDbContext(optionsBuilder.Options);
	AccountControllerV2 controller = new AccountControllerV2(context);
	IActionResult result = controller.Create(new Account() { Id = 1, Name = "Checking", SSN = "078-05-1120" });
	Assert.True(result != null);
	Assert.True(context.Accounts.Count() == 1);
	Assert.True(context.Accounts.First().Id == 1);
	Assert.True(context.Accounts.First().Name == "Checking");
	Assert.True(context.Accounts.First().DateCreated != null);
	Assert.True(context.Accounts.First().SSN == "078-05-1120");
}

[Fact]
public void Test_Account_Create_With_Validation()
{
	//setup
	DbContextOptionsBuilder optionsBuilder = new DbContextOptionsBuilder();
	optionsBuilder.UseInMemoryDatabase("test2");
	var context = new ApplicationDbContext(optionsBuilder.Options);
	AccountControllerV2 controller = new AccountControllerV2(context);

	//act
	IActionResult result = controller.Create(new Account() { Id = 1, Name = "Checking", SSN="078-05-1120" });
	Assert.True(result != null); // the result isn't null
	Assert.True(context.Accounts.Count() == 1); // the result has the count we expect
	Assert.True(context.Accounts.First().Id == 1); // the record has the id we expect
	Assert.True(context.Accounts.First().Name == "Checking"); // it has the right name
	Assert.True(context.Accounts.First().DateCreated != null); // we saved a date
	Assert.True(context.Accounts.First().SSN == "078-05-1120");
	
	// test that creating a blank name returns a 422 code
	IActionResult result2 = controller.Create(new Account() { Id = 1, Name = "" });
	Assert.True((result2 as StatusCodeResult).StatusCode == 422);

	// test that creating duplicate names returns 4222
	IActionResult resultUnique = controller.Create(new Account() { Id = 2, Name = "Checking1" });
	IActionResult resultUniqueTest = controller.Create(new Account() { Id = 2, Name = "Checking1" });
	Assert.True((result2 as StatusCodeResult).StatusCode == 422);

}
```


In Iteration 2 we changed our controller to take the Entity Framework context as a constructor argument. This is the D in SOLID. Dependency inversion
means classes that depend on other classes to operate have those classes injected instead of creating them. There are inversion of control systems that handle
creating whatever instance a class required. This project uses Autofac but there are several IOC libraries available. By injecting the Entity Framework context
we can verify things happened to the context like records were created. 

In Test_Account_Create we are now testing that a record actually got added to the context after calling create on the controller.

In Test_Account_Create_With_Validation we test that creating an account with a blank name returns a 422(Unprocessable Entity) code and then we test that creating an account with the same name as 
an existing return a 422 error. 

We have 100% test coverage. What are some issues with this code?

Our tests are growing! How big will they go? They're complicated too. 

1. Test_Account_Create_With_Validation is testing lots of stuff. A failure anywhere in 
this test will require some digging to understand what the problem really is. 

2. We're still using Entity Framework which is slow in a test context. Unit tests should be crazy fast so you get instant feedback when you're working in the code.

3. We're still checking every field which is brittle as we discussed.

4. When we add 20 more fields to this table how long will this test be? 1k lines?

5. We're still relying on DateTime.Now. 


#### Iteration 3

###### Requirement change
A new requirement came in and we have to change this controller to call an external service to validate this customer's SSN with the FBI counter-terrorism database. We can't
let people on the terror list open bank accounts so we are going to call the FBI service with the user's social security number to determine if we should let them open an account. 

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;

namespace Chaos.Controllers
{
    /// <summary>
    /// A new requirement came in and we have to change this controller to use an external service to validate this customer with the FBI counter-terrorism database.
    /// </summary>
    [Route("api/[controller]")]
    [ApiController]
    public class AccountControllerV3 : ControllerBase
    {
        private ApplicationDbContext context;
        public AccountControllerV3(ApplicationDbContext context)
        {
            this.context = context;
        }
        [HttpPost]
        public async Task<IActionResult> Create(Account account)
        {
            if (string.IsNullOrWhiteSpace(account.Name))
            {
                return new StatusCodeResult((int)System.Net.HttpStatusCode.UnprocessableEntity);
            }

            if (context.Accounts.Any(a => a.Name == account.Name))
            {
                return new StatusCodeResult((int)System.Net.HttpStatusCode.UnprocessableEntity);
            }

            if (string.IsNullOrWhiteSpace(account.SSN))
            {
                return new StatusCodeResult((int)System.Net.HttpStatusCode.UnprocessableEntity);
            }

            if(!(await this.VerifyWithFBI(account.SSN)))
            {
                return new StatusCodeResult((int)System.Net.HttpStatusCode.BadRequest);
            }

            context.Accounts.Add(account);
            context.SaveChanges();
            return Ok(account);
        }

		
        private async Task<bool> VerifyWithFBI(string ssn)
        {
            HttpClient client = new HttpClient();
            string response = await client.GetStringAsync("https://jsonplaceholder.typicode.com/posts/1");
            if(!string.IsNullOrWhiteSpace(response))
            {
                return true;
            }
            else
            {
                return false;
            }
        }
    }
}
```

```
 [Fact]
public async void Test_Account_Create_With_Validation_Cant_Test_FBI_Service()
{
	//setup
	DbContextOptionsBuilder optionsBuilder = new DbContextOptionsBuilder();
	optionsBuilder.UseInMemoryDatabase("test2");
	var context = new ApplicationDbContext(optionsBuilder.Options);
	AccountControllerV3 controller = new AccountControllerV3(context);

	//act
	IActionResult result = await controller.Create(new Account() { Id = 1, Name = "Checking", SSN= "000-05-1120" });
	Assert.True(result != null); // the result isn't null
	Assert.True(context.Accounts.Count() == 1); // the result has the count we expect
	Assert.True(context.Accounts.First().Id == 1); // the record has the id we expect
	Assert.True(context.Accounts.First().Name == "Checking"); // it has the right name
	Assert.True(context.Accounts.First().DateCreated != null); // we saved a date
	Assert.True(context.Accounts.First().SSN == "000-05-1120");
	// test that creating a blank name returns a 422 code
	IActionResult result2 = await controller.Create(new Account() { Id = 1, Name = "" });
	Assert.True((result2 as StatusCodeResult).StatusCode == 422);

	// test that creating duplicate names returns 4222
	IActionResult resultUnique = await controller.Create(new Account() { Id = 2, Name = "Checking1" });
	IActionResult resultUniqueTest = await controller.Create(new Account() { Id = 2, Name = "Checking1" });
	Assert.True((result2 as StatusCodeResult).StatusCode == 422);


	//How can we test that our FBI service worked or failed? We really can't.
}
```

We've added the FBI service method. Ignoring the fact this is just a test json endpoint what's wrong with this method?

1. Our endpoint is hard-coded.
2. Is there any way to mock this method in tests for our controller? Probably not because it doesn't use an interface and the method isn't virtual. 
3. This is just an FYI but HttpClient should be used as a singleton in .NET core. It will use up all of the sockets on a machine used like this. You should 
not use using statements with it either. 


#### Iteration 4
In this iteration we're going to make our FBI service testable in our controller so that if it returns false we can test our controller returns the right status code. 

```
[Route("api/[controller]")]
[ApiController]
public class AccountControllerV4 : ControllerBase
{
	private ApplicationDbContext context;
	private IFBIService fbiService;
	public AccountControllerV4(ApplicationDbContext context, IFBIService fbiService)
	{
		this.context = context;
		this.fbiService = fbiService;
	}
	[HttpPost]
	public async Task<IActionResult> Create(Account account)
	{
		if (string.IsNullOrWhiteSpace(account.Name))
		{
			return new StatusCodeResult((int)System.Net.HttpStatusCode.UnprocessableEntity);
		}

		if (context.Accounts.Any(a => a.Name == account.Name))
		{
			return new StatusCodeResult((int)System.Net.HttpStatusCode.UnprocessableEntity);
		}

		if (string.IsNullOrWhiteSpace(account.SSN))
		{
			return new StatusCodeResult((int)System.Net.HttpStatusCode.UnprocessableEntity);
		}

		if (!(await this.fbiService.VerifyWithFBI(account.SSN)))
		{
			return new StatusCodeResult((int)System.Net.HttpStatusCode.BadRequest);
		}

		context.Accounts.Add(account);
		context.SaveChanges();
		return Ok(account);
	}
}

public interface IFBIService
{
	Task<bool> VerifyWithFBI(string ssn);
}

public class FBIService : IFBIService
{
	public async Task<bool> VerifyWithFBI(string ssn)
	{
		HttpClient client = new HttpClient();
		string response = await client.GetStringAsync("https://jsonplaceholder.typicode.com/posts/1");
		if (!string.IsNullOrWhiteSpace(response))
		{
			return true;
		}
		else
		{
			return false;
		}
	}
}
```


```
public async void Test_Account_Create_With_Validation_With_FBIService_mock()
{
	//setup
	DbContextOptionsBuilder optionsBuilder = new DbContextOptionsBuilder();
	optionsBuilder.UseInMemoryDatabase("test2");
	var context = new ApplicationDbContext(optionsBuilder.Options);
	FBIService fbiService = new FBIService();

	Mock<IFBIService> fbiServiceMock = new Mock<IFBIService>();
	fbiServiceMock.Setup(x => x.VerifyWithFBI(It.IsAny<string>())).Returns(Task.FromResult<bool>(true));
	AccountControllerV4 controller = new AccountControllerV4(context, fbiService);

	//act
	IActionResult result = await controller.Create(new Account() { Id = 1, Name = "Checking", SSN = "000-05-1120" });
	Assert.True(result != null); // the result isn't null
	Assert.True(context.Accounts.Count() == 1); // the result has the count we expect
	Assert.True(context.Accounts.First().Id == 1); // the record has the id we expect
	Assert.True(context.Accounts.First().Name == "Checking"); // it has the right name
	Assert.True(context.Accounts.First().DateCreated != null); // we saved a date
	Assert.True(context.Accounts.First().SSN == "000-05-1120");
	// test that creating a blank name returns a 422 code
	IActionResult result2 = await controller.Create(new Account() { Id = 1, Name = "", SSN = "000-05-1120" });
	Assert.True((result2 as StatusCodeResult).StatusCode == 422);

	// test that creating duplicate names returns 422
	IActionResult resultUnique = await controller.Create(new Account() { Id = 2, Name = "Checking1", SSN = "000-05-1120" });
	IActionResult resultUniqueTest = await controller.Create(new Account() { Id = 3, Name = "Checking1", SSN = "000-05-1120" });
	Assert.True((result2 as StatusCodeResult).StatusCode == 422);


	// we're now able to test that if the fbi service returns false we handle it with a 400 response
	Mock<IFBIService> fbiServiceMock2 = new Mock<IFBIService>();
	fbiServiceMock2.Setup(x => x.VerifyWithFBI(It.IsAny<string>())).Returns(Task.FromResult<bool>(false));
	AccountControllerV4 controller2 = new AccountControllerV4(context, fbiServiceMock2.Object);
	IActionResult result3 = await controller2.Create(new Account() { Id = 4, Name = "CheckingNew", SSN = "000-05-1120" });
	Assert.True((result3 as StatusCodeResult).StatusCode == 400);

}
```

In this iteration we changed the FBI service to use an interface and be injected into the controller. This will allow us to test the FBI
service in isolation and it will allow us to mock the service in our controller. Our methods continue to grow. How can we improve this? Our controller is doing a lot of stuff. How could we move this application to a service bus? That would be difficult because our app is deeply coupled to ASP.NET. Our validation is still in our controller method and should be in it's own class. 



#### Iteration 5

This one is a major refactor. We're going to break everything into SOLID pieces and add a repository layer over the Entity Framework context. This will let us mock
our data access layer without needing to use Entity Framework. We're moving validation to it's own set of classes and we're injecting everything.

```
 public class AccountControllerV5 : ControllerBase
{
	private IAccountRepositoryV5 repository;
	private IFBIServiceV5 fbiService;
	private IAccountModelValiatorV5 modelValidator;

	public AccountControllerV5(IAccountRepositoryV5 repository, IAccountModelValiatorV5 modelValidator, IFBIServiceV5 fbiService)
	{
		this.repository = repository;
		this.fbiService = fbiService;
		this.modelValidator = modelValidator;
	}

	[HttpPost]
	public async Task<IActionResult> Create(Account account)
	{
		ValidationResultV5 result = this.modelValidator.Validate(account);

		if (result.Success)
		{
			if (!await this.fbiService.VerifyWithFBI(account.SSN))
			{
				return new StatusCodeResult((int)System.Net.HttpStatusCode.BadRequest);
			}

			this.repository.Create(account);

			return this.Ok(account);
		}
		else
		{
			return this.StatusCode(422, result);
		}
	}
}

#region validation
public class ValidationResultV5
{
	public bool Success { get; private set; } = true;
	public string Message { get; private set; } = "";

	public ValidationResultV5(bool success, string message)
	{
		this.Success = success;
		this.Message = message;
	}

	public ValidationResultV5()
	{
	}
}


public interface IAccountModelValiatorV5
{
	ValidationResultV5 Validate(Account account);
}

public class AccountModelValiatorV5
{
	private IAccountRepositoryV5 repository;
	public AccountModelValiatorV5(IAccountRepositoryV5 repository)
	{
		this.repository = repository;
	}

	public ValidationResultV5 Validate(Account account)
	{
		if (string.IsNullOrWhiteSpace(account.Name))
		{
			return new ValidationResultV5(false, "Account name cannot be empty");
		}

		if (this.repository.Exists(account.Name))
		{
			return new ValidationResultV5(false, "Account name already exists");
		}

		if (string.IsNullOrWhiteSpace(account.SSN))
		{
			return new ValidationResultV5(false, "Customer id cannot be empty");
		}

		return new ValidationResultV5();
	}
}
#endregion

#region repository
public interface IAccountRepositoryV5
{
	bool Exists(string name);

	void Create(Account account);
}

public class AccountRepositoryV5 : IAccountRepositoryV5
{
	private ApplicationDbContext context;

	public AccountRepositoryV5(ApplicationDbContext context)
	{
		this.context = context;
	}

	public bool Exists(string name)
	{
		return this.context.Accounts.Any(x => x.Name == name);
	}
	public void Create(Account account)
	{
		this.context.Accounts.Add(account);
		context.SaveChanges();
	}
}
#endregion

#region fbiService
public interface IFBIServiceV5
{
	Task<bool> VerifyWithFBI(string ssn);
}

public class FBIServiceV5 : IFBIServiceV5
{
	public FBIServiceV5()
	{
	}

	/// <summary>
	/// This can only be tested in an integration test.
	/// </summary>
	/// <param name="ssn"></param>
	/// <returns></returns>
	public async Task<bool> VerifyWithFBI(string ssn)
	{
		var client = new HttpClient();
		string response = await client.GetStringAsync("https://jsonplaceholder.typicode.com/posts/1");
		if (!string.IsNullOrWhiteSpace(response))
		{
			return true;
		}
		else
		{
			return false;
		}
	}
}
```
```
[Fact]
public async void Test_Account_Create_Happy_Path()
{
	//setup
	Mock<IAccountRepositoryV5> repositoryMock = new Mock<IAccountRepositoryV5>();
	Mock<IAccountModelValiatorV5> modelValidatorMock = new Mock<IAccountModelValiatorV5>();
	Mock<IFBIServiceV5> fbiServiceMock = new Mock<IFBIServiceV5>();

	// all of our mocks are returning what they should for a successful request
	fbiServiceMock.Setup(x => x.VerifyWithFBI(It.IsAny<string>())).Returns(Task.FromResult<bool>(true));
	modelValidatorMock.Setup(x => x.Validate(It.IsAny<Account>())).Returns(new ValidationResultV5());

	//act
	AccountControllerV5 controller = new AccountControllerV5(repositoryMock.Object, modelValidatorMock.Object, fbiServiceMock.Object);
	IActionResult result = await controller.Create(new Account());

	//verify
	Assert.True((result as OkObjectResult).StatusCode == 200); // ok result
	modelValidatorMock.Verify(x => x.Validate(It.IsAny<Account>())); // verify the methods on our objects are actually called
	fbiServiceMock.Verify(x => x.VerifyWithFBI(It.IsAny<string>()));
}


[Fact]
public async void Test_Account_Create_Validation_Failed()
{
	//setup
	Mock<IAccountRepositoryV5> repositoryMock = new Mock<IAccountRepositoryV5>();
	Mock<IAccountModelValiatorV5> modelValidatorMock = new Mock<IAccountModelValiatorV5>();
	Mock<IFBIServiceV5> fbiServiceMock = new Mock<IFBIServiceV5>();

	// all of our mocks are returning what they should for a successful request
	fbiServiceMock.Setup(x => x.VerifyWithFBI(It.IsAny<string>())).Returns(Task.FromResult<bool>(true));
	repositoryMock.Setup(x => x.Create(It.IsAny<Account>()));
	modelValidatorMock.Setup(x => x.Validate(It.IsAny<Account>())).Returns(new ValidationResultV5(false, "error"));

	//act
	AccountControllerV5 controller = new AccountControllerV5(repositoryMock.Object, modelValidatorMock.Object, fbiServiceMock.Object);
	IActionResult result = await controller.Create(new Account());

	//verify
	Assert.True((result as ObjectResult).StatusCode == 422);
}


[Fact]
public async void Test_Account_Create_FBIService_Failed()
{
	//setup
	Mock<IAccountRepositoryV5> repositoryMock = new Mock<IAccountRepositoryV5>();
	Mock<IAccountModelValiatorV5> modelValidatorMock = new Mock<IAccountModelValiatorV5>();
	Mock<IFBIServiceV5> fbiServiceMock = new Mock<IFBIServiceV5>();

	// all of our mocks are returning what they should for a successful request
	fbiServiceMock.Setup(x => x.VerifyWithFBI(It.IsAny<string>())).Returns(Task.FromResult<bool>(false));
	repositoryMock.Setup(x => x.Create(It.IsAny<Account>()));
	modelValidatorMock.Setup(x => x.Validate(It.IsAny<Account>())).Returns(new ValidationResultV5());

	//act
	AccountControllerV5 controller = new AccountControllerV5(repositoryMock.Object, modelValidatorMock.Object, fbiServiceMock.Object);
	IActionResult result = await controller.Create(new Account());

	//verify
	Assert.True((result as StatusCodeResult).StatusCode == 400);
}


[Fact]
public void Test_Account_Model_Validator_Empty_Account_Name()
{
	Mock<IAccountRepositoryV5> repositoryMock = new Mock<IAccountRepositoryV5>();

	AccountModelValiatorV5 valiator = new AccountModelValiatorV5(repositoryMock.Object);
	Account account = new Account();
	account.DateCreated = DateTime.Now;
	account.SSN = "000-05-1120";
	account.Name = string.Empty;
	ValidationResultV5 result = valiator.Validate(account);

	Assert.False(result.Success);
	Assert.Equal("Account name cannot be empty", result.Message);
}

[Fact]
public void Test_Account_Model_Validator_Valid_Account_Name()
{
	Mock<IAccountRepositoryV5> repositoryMock = new Mock<IAccountRepositoryV5>();

	AccountModelValiatorV5 valiator = new AccountModelValiatorV5(repositoryMock.Object);
	Account account = new Account();
	account.DateCreated = DateTime.Now;
	account.SSN = "000-05-1120";
	account.Name = "Test";
	ValidationResultV5 result = valiator.Validate(account);

	Assert.True(result.Success);
	Assert.True(result.Message == string.Empty);
}

[Fact]
public void Test_Account_Model_Validator_Empty_SSN()
{
	Mock<IAccountRepositoryV5> repositoryMock = new Mock<IAccountRepositoryV5>();

	AccountModelValiatorV5 valiator = new AccountModelValiatorV5(repositoryMock.Object);
	Account account = new Account();
	account.DateCreated = DateTime.Now;
	account.SSN = default(string);
	account.Name = "test";
	ValidationResultV5 result = valiator.Validate(account);

	Assert.False(result.Success);
	Assert.Equal("Customer id cannot be empty", result.Message);
}

[Fact]
public void Test_Account_Model_Validator_Valid_SSN()
{
	Mock<IAccountRepositoryV5> repositoryMock = new Mock<IAccountRepositoryV5>();

	AccountModelValiatorV5 valiator = new AccountModelValiatorV5(repositoryMock.Object);
	Account account = new Account();
	account.DateCreated = DateTime.Now;
	account.SSN = "000-05-1120";
	account.Name = "test";
	ValidationResultV5 result = valiator.Validate(account);

	Assert.True(result.Success);
	Assert.Equal(string.Empty, result.Message);
}

[Fact]
public void Test_Account_Model_Validator_Account_Exists()
{
	Mock<IAccountRepositoryV5> repositoryMock = new Mock<IAccountRepositoryV5>();
	repositoryMock.Setup(x => x.Exists(It.IsAny<string>())).Returns(true);
	AccountModelValiatorV5 valiator = new AccountModelValiatorV5(repositoryMock.Object);
	Account account = new Account();
	account.DateCreated = DateTime.Now;
	account.SSN = "000-05-1120";
	account.Name = "test";
	ValidationResultV5 result = valiator.Validate(account);

	Assert.False(result.Success);
	Assert.Equal("Account name already exists", result.Message);
}

[Fact]
public void Test_Account_Model_Validator_Account_Not_Exists()
{
	Mock<IAccountRepositoryV5> repositoryMock = new Mock<IAccountRepositoryV5>();
	repositoryMock.Setup(x => x.Exists(It.IsAny<string>())).Returns(false);
	AccountModelValiatorV5 valiator = new AccountModelValiatorV5(repositoryMock.Object);
	Account account = new Account();
	account.DateCreated = DateTime.Now;
	account.SSN = "000-05-1120";
	account.Name = "test";
	ValidationResultV5 result = valiator.Validate(account);

	Assert.True(result.Success);
	Assert.Equal(string.Empty, result.Message);
}


[Fact]
public void Test_Account_Repository_Create()
{ 
	DbContextOptionsBuilder optionsBuilder = new DbContextOptionsBuilder();
	optionsBuilder.UseInMemoryDatabase("test1");
	var context = new ApplicationDbContext(optionsBuilder.Options);
	IAccountRepositoryV5 repository = new AccountRepositoryV5(context);
	repository.Create(new Account());
	Assert.True(context.Accounts.Count() == 1);
}


[Fact]
public void Test_Account_Repository_Exists_Record_Exists()
{
	DbContextOptionsBuilder optionsBuilder = new DbContextOptionsBuilder();
	optionsBuilder.UseInMemoryDatabase("test1");
	var context = new ApplicationDbContext(optionsBuilder.Options);
	IAccountRepositoryV5 repository = new AccountRepositoryV5(context);

	var account = new Account();
	account.Name = "testing";
	context.Accounts.Add(account);
	context.SaveChanges();

	bool exists = repository.Exists("testing");
	Assert.True(exists);
}

[Fact]
public void Test_Account_Repository_Exists_Record_Not_Exists()
{
	DbContextOptionsBuilder optionsBuilder = new DbContextOptionsBuilder();
	optionsBuilder.UseInMemoryDatabase("test1");
	var context = new ApplicationDbContext(optionsBuilder.Options);
	IAccountRepositoryV5 repository = new AccountRepositoryV5(context);

	bool exists = repository.Exists("testing");
	Assert.False(exists);
}
```

That's a lot of change. Our validators, repository and FBI service are now in their own classes and are injected into our controller. This is looking better!
We've added lots of unit tests that actually test a unit. I won't go into detail on what these tests do. They should be self explanatory which is how unit tests are supposed to be. The test name should clue you in on what is being tested. The tests should be less than 10 lines long in most cases so there isn't this enormous overhead
to understand what a test does and changing tests should be easier that our first tests we wrote that were testing everything. Our repository tests are technically integration tests but that's ok there isn't any other way to test it. 



#### Iteration 6

##### Requirement change
There has been a requirement change and we need to receive a token from the client web service when accounts are created but we can't return the token in any account responses because it should be private.



We also need to add a Get method to retrieve an account by id. 

We need to accept a token in the request but not return it in the response. 

**There are no changes in this iteration's code. Look at it and think about how you would make the request and response differ.** 

```
 public class AccountControllerV6 : ControllerBase
{
	private IAccountRepositoryV6 repository;
	private IFBIServiceV6 fbiService;
	private IAccountModelValiatorV6 modelValidator;

	public AccountControllerV6(IAccountRepositoryV6 repository, IAccountModelValiatorV6 modelValidator, IFBIServiceV6 fbiService)
	{
		this.repository = repository;
		this.fbiService = fbiService;
		this.modelValidator = modelValidator;
	}

	[HttpPost]
	public async Task<IActionResult> Create(Account account)
	{
		ValidationResultV6 result = this.modelValidator.Validate(account);

		if (result.Success)
		{
			if (!await this.fbiService.VerifyWithFBI(account.SSN))
			{
				return new StatusCodeResult((int)System.Net.HttpStatusCode.BadRequest);
			}

			this.repository.Create(account);

			return this.Ok(account);
		}
		else
		{
			return this.StatusCode(422, result);
		}
	}

	[HttpGet]
	public IActionResult Get(int id)
	{
		Account record = this.repository.Find(id);

		if (record == null)
		{
			return this.StatusCode(404);
		}
		else
		{
			return this.Ok(record);
		}
	}
}

#region validation
public class ValidationResultV6
{
	public bool Success { get; private set; } = true;
	public string Message { get; private set; } = "";

	public ValidationResultV6(bool success, string message)
	{
		this.Success = success;
		this.Message = message;
	}

	public ValidationResultV6()
	{
	}
}


public interface IAccountModelValiatorV6
{
	ValidationResultV6 Validate(Account account);
}

public class AccountModelValiatorV6
{
	private IAccountRepositoryV6 repository;
	public AccountModelValiatorV6(IAccountRepositoryV6 repository)
	{
		this.repository = repository;
	}

	public ValidationResultV6 Validate(Account account)
	{
		if (string.IsNullOrWhiteSpace(account.Name))
		{
			return new ValidationResultV6(false, "Account name cannot be empty");
		}

		if (this.repository.Exists(account.Name))
		{
			return new ValidationResultV6(false, "Account name already exists");
		}

		if (string.IsNullOrWhiteSpace(account.SSN))
		{
			return new ValidationResultV6(false, "Customer id cannot be empty");
		}

		return new ValidationResultV6();
	}
}
#endregion

#region  repository
public interface IAccountRepositoryV6
{
	bool Exists(string name);

	void Create(Account account);

	Account Find(int id);
}

public class AccountRepositoryV6 : IAccountRepositoryV6
{
	private ApplicationDbContext context;

	public AccountRepositoryV6(ApplicationDbContext context)
	{
		this.context = context;
	}

	public bool Exists(string name)
	{
		return this.context.Accounts.Any(x => x.Name == name);
	}
	public void Create(Account account)
	{
		this.context.Accounts.Add(account);
		context.SaveChanges();
	}

	public Account Find(int id)
	{
	   return this.context.Accounts.FirstOrDefault(x => x.Id == id);
	}
}
#endregion

#region fbiService
public interface IFBIServiceV6
{
	Task<bool> VerifyWithFBI(string ssn);
}

public class FBIServiceV6 : IFBIServiceV6
{
	HttpClient client;
	public FBIServiceV6(HttpClient client)
	{
		this.client = client;
	}

	public async Task<bool> VerifyWithFBI(string ssn)
	{
		string response = await this.client.GetStringAsync("https://jsonplaceholder.typicode.com/posts/1");
		if (!string.IsNullOrWhiteSpace(response))
		{
			return true;
		}
		else
		{
			return false;
		}
	}
}
```



There isn't a test project for this iteration because I wanted to demonstrate the changing requirements.
V7 will fix the issues and test them.
We've added our method to allow retrieving the account but we're
exposing the secret token we're not supposed to(this is in Account.cs). Our response and request model
have diverged! What do we do?
You may have realized by now that using your database entity as your request model is not a 
good idea. We've discovered that when the model diverges you're in a pickle. You can hack it to make it
work which means having extra fields or you split your model into a request and a response. 
 
Another thing to think about now is how would we dump ASP.NET and move our system to a service bus?
The validation and the FBI service is still in the controller. Would the service bus have to call the controller in code?
Our system is still too coupled. You should be able to replace the controller layer with a winform app or a service bus and there
should be no issues there. The root problem is there is still too much happening in our controller.
We changed the FBI service in this iteration to accept the HttpClient as a parameter. This would let us inject it and test this service later. 






#### Iteration 7

Create separate request and response models so we can satisfy our requirements that a token is in the request but not in the response. 

```
public class AccountControllerV7 : ControllerBase
{
	AccountServiceV7 service;

	public AccountControllerV7(AccountServiceV7 service)
	{
		this.service = service;
	}

	[HttpPost]
	public async Task<IActionResult> Create(AccountRequestModelV7 model)
	{
		ActionResultV7 result = await this.service.Create(model);
		if(result.Success)
		{
			return this.Ok(result.Object);
		}
		else
		{
			return this.StatusCode(400);
		}
	}

	[HttpGet]
	public async Task<IActionResult> Get (int id)
	{
		ActionResultV7 result = await this.service.Get(id);
		if (result.Success)
		{
			return this.Ok(result.Object);
		}
		else
		{
			return this.StatusCode(404);
		}
	}
}


#region models
public class AccountRequestModelV7
{
	public int Id { get; set; } // internal record identifier for this account
	public string Name { get; set; } // checking, saving, investment
	public string SSN { get; set; } // a unique customer id used to globally track a person's bank accounts
	public DateTime DateCreated { get; set; } // when the account was created
	public string Token { get; set; } // secret token. We never want to expose this.
}

public class AccountResponseModelV7
{
	public int Id { get; set; } // internal record identifier for this account
	public string Name { get; set; } // checking, saving, investment
	public string SSN { get; set; } // a unique customer id used to globally track a person's bank accounts
	public DateTime DateCreated { get; set; } // when the account was created

}

#endregion

#region service


public class ActionResultV7
{
	public bool Success { get; private set; }

	public object Object { get; private set; }

	public ActionResultV7(bool success, object @object)
	{
		this.Success = success;
		this.Object = @object;
	}

	public ActionResultV7(ValidationResultV7 result)
	{
		this.Success = result.Success;
		this.Object = result.Message;
	}
}

public class AccountServiceV7
{
	private IAccountRepositoryV7 repository;
	private IFBIServiceV7 fbiService;
	private IAccountModelValiatorV7 modelValidator;
	public AccountServiceV7(IAccountRepositoryV7 repository, IAccountModelValiatorV7 modelValidator, IFBIServiceV7 fbiService)
	{
		this.repository = repository;
		this.fbiService = fbiService;
		this.modelValidator = modelValidator;
	}

	public async Task<ActionResultV7> Create(AccountRequestModelV7 model)
	{
		ValidationResultV7 result = this.modelValidator.Validate(model);

		if (result.Success)
		{
			if (!await this.fbiService.VerifyWithFBI(model.SSN))
			{
				return new ActionResultV7(false, new ValidationResultV7(false, "Unable to validate with FBI"));
			}

			var account = new Account(0, model.Name, model.SSN, DateTime.Now, model.Token);

			this.repository.Create(account);

			return new ActionResultV7(true, account);
		}
		else
		{
			return new ActionResultV7(result);
		}
	}

	public async Task<ActionResultV7> Get(int id)
	{
		Account record = await this.repository.Find(id);
		if(record == null)
		{
			return new ActionResultV7(false, new ValidationResultV7(false, "Record not found"));
		}
		else
		{
			var response = new AccountResponseModelV7()
			{
				DateCreated = record.DateCreated,
				SSN = record.SSN,
				Id = record.Id,
				Name = record.Name
			};
			return new ActionResultV7(true, response);
		}
	}


}

#endregion

#region validation
public class ValidationResultV7
{
	public bool Success { get; private set; } = true;
	public string Message { get; private set; } = "";

	public ValidationResultV7(bool success, string message)
	{
		this.Success = success;
		this.Message = message;
	}

	public ValidationResultV7()
	{
	}
}


public interface IAccountModelValiatorV7
{
	ValidationResultV7 Validate(AccountRequestModelV7 account);
}

public class AccountModelValiatorV7
{
	private IAccountRepositoryV7 repository;
	public AccountModelValiatorV7(IAccountRepositoryV7 repository)
	{
		this.repository = repository;
	}

	public ValidationResultV7 Validate(Account account)
	{
		if (string.IsNullOrWhiteSpace(account.Name))
		{
			return new ValidationResultV7(false, "Account name cannot be empty");
		}

		if (this.repository.Exists(account.Name))
		{
			return new ValidationResultV7(false, "Account name already exists");
		}

		if (string.IsNullOrWhiteSpace(account.SSN))
		{
			return new ValidationResultV7(false, "Customer id cannot be empty");
		}

		return new ValidationResultV7();
	}
}
#endregion

#region repository
public interface IAccountRepositoryV7
{
	bool Exists(string name);

	void Create(Account account);

	Task<Account> Find(int id);
}

public class AccountRepositoryV7 : IAccountRepositoryV7
{
	private ApplicationDbContext context;

	public AccountRepositoryV7(ApplicationDbContext context)
	{
		this.context = context;
	}

	public bool Exists(string name)
	{
		return this.context.Accounts.Any(x => x.Name == name);
	}
	public void Create(Account account)
	{
		this.context.Accounts.Add(account);
		context.SaveChanges();
	}

	public async Task<Account> Find(int id)
	{
	   return await this.context.Accounts.FirstOrDefaultAsync(x => x.Id == id);
	}
}
#endregion

#region fbiService
public interface IFBIServiceV7
{
	Task<bool> VerifyWithFBI(string ssn);
}

public class FBIServiceV7 : IFBIServiceV7
{
	HttpClient client;
	public FBIServiceV7(HttpClient client)
	{
		this.client = client;
	}

	public async Task<bool> VerifyWithFBI(string ssn)
	{
		string response = await this.client.GetStringAsync("https://jsonplaceholder.typicode.com/posts/1");
		if (!string.IsNullOrWhiteSpace(response))
		{
			return true;
		}
		else
		{
			return false;
		}
	}
}
```

In this iteration we added AccountRequestModelV7 and AccountResponseModelV7 and we created another layer which is the service layer. By creating this service layer we can put all of the business logic like calling the FBI service and validating entities in this layer instead of the controller. This will allow us to move to a service bus later which we couldn't do if the validation was done in the controller. Notice how "thin" the controller is. It does very little besides call the service.

1. We're still using inline object initialization.
2. We still can't test the dateCreated logic. 
3. We still can't test individual field level validation our models without testing all of the validation.



#### Iteration 8

We're going to introduce Fluent Validation in this iteration. Fluent Validation will let us test individual validators on fields. 

We're going to move object mapping to it's own class so we only have to update that logic in once place.

We're going to create a date service to facilitate testing. This interface would let us introduce NodaTime or other date libraries to help with time zone issues. 

```
public class AccountControllerV8 : ControllerBase
{
	AccountServiceV8 service;

	public AccountControllerV8(AccountServiceV8 service)
	{
		this.service = service;
	}

	[HttpPost]
	public async Task<IActionResult> Create(AccountRequestModelV8 model)
	{
		ActionResultV8 result = await this.service.Create(model);
		if (result.Success)
		{
			return this.Ok(result.Object);
		}
		else
		{
			return this.StatusCode(400);
		}
	}

	[HttpGet]
	public async Task<IActionResult> Get(int id)
	{
		ActionResultV8 result = await this.service.Get(id);
		if (result.Success)
		{
			return this.Ok(result.Object);
		}
		else
		{
			return this.StatusCode(404);
		}
	}
}

#endregion

#region models
public class AccountRequestModelV8
{
	public int Id { get; set; } // internal record identifier for this account
	public string Name { get; set; } // checking, saving, investment
	public string SSN { get; set; } // a unique customer id used to globally track a person's bank accounts
	public DateTime DateCreated { get; set; } // when the account was created
	public string Token { get; set; } // secret token. We never want to expose this.

	public AccountRequestModelV8()
	{

	}

	public AccountRequestModelV8(int id, string name, string ssn, DateTime dateCreated, string token)
	{
		this.Id = id;
		this.Name = name;
		this.SSN = ssn;
		this.DateCreated = dateCreated;
		this.Token = token;
	}
}

public class AccountResponseModelV8
{
	public int Id { get; set; } // internal record identifier for this account
	public string Name { get; set; } // checking, saving, investment
	public string SSN { get; set; } // a unique customer id used to globally track a person's bank accounts
	public DateTime DateCreated { get; set; } // when the account was created

	public AccountResponseModelV8(int id, string name, string ssn, DateTime dateCreated)
	{
		this.Id = id;
		this.Name = name;
		this.SSN = ssn;
		this.DateCreated = dateCreated;
	}
}

#endregion

#region service

public class ActionResultV8
{
	public bool Success { get; private set; }

	public object Object { get; private set; }

	public ActionResultV8(bool success, object @object)
	{
		this.Success = success;
		this.Object = @object;
	}

	public ActionResultV8(ValidationResultV8 result)
	{
		this.Success = result.Success;
		this.Object = result.Message;
	}

	public ActionResultV8(ValidationResult result)
	{
		this.Success = result.IsValid;
		this.Object = result.Errors.FirstOrDefault()?.ErrorMessage;
	}
}

public class AccountServiceV8
{
	private IAccountRepositoryV8 repository;
	private IFBIServiceV8 fbiService;
	private AccountRequestModelValidatorV8 modelValidator;
	private IDateServiceV8 dateService;
	private IAccountMapperV8 mapper;
	public AccountServiceV8(IAccountRepositoryV8 repository, 
		AccountRequestModelValidatorV8 modelValidator, 
		IFBIServiceV8 fbiService, 
		IDateServiceV8 dateService,
		IAccountMapperV8 mapper)
	{
		this.repository = repository;
		this.fbiService = fbiService;
		this.modelValidator = modelValidator;
		this.dateService = dateService;
		this.mapper = mapper;
	}


	public async Task<ActionResultV8> Create(AccountRequestModelV8 model)
	{
		ValidationResult result = this.modelValidator.Validate(model);

		if (result.IsValid)
		{
			if (!await this.fbiService.VerifyWithFBI(model.SSN))
			{
				return new ActionResultV8(false, new ValidationResultV8(false, "Unable to validate with FBI"));
			}

			Account account = this.mapper.MapRequestToEntity(model);

			this.repository.Create(account);

			return new ActionResultV8(true, account);
		}
		else
		{
			return new ActionResultV8(result);
		}
	}

	public async Task<ActionResultV8> Get(int id)
	{
		Account record = await this.repository.Find(id);
		if (record == null)
		{
			return new ActionResultV8(false, new ValidationResultV8(false, "Record not found"));
		}
		else
		{
			AccountResponseModelV8 response = this.mapper.MapEntityToResponse(record);

			return new ActionResultV8(true, response);
		}
	}


}

#endregion

#region mapper

public interface IAccountMapperV8
{
	Account MapRequestToEntity(AccountRequestModelV8 model);
	AccountResponseModelV8 MapEntityToResponse(Account entity);
}

/// <summary>
/// You could also use AutoMapper here. I prefer just using constructors or methods to set the properties.
/// 
/// </summary>
public class AccountMapperV8 : IAccountMapperV8
{
	public Account MapRequestToEntity(AccountRequestModelV8 model)
	{
		return new Account(model.Id, model.Name, model.SSN, model.DateCreated, model.Token);
	}

	public AccountResponseModelV8 MapEntityToResponse(Account entity)
	{
		return new AccountResponseModelV8(entity.Id,entity.Name, entity.SSN, entity.DateCreated);
	}
}

#endregion

#region validation

public class AccountRequestModelValidatorV8 : AbstractValidator<AccountRequestModelV8>
{
	IAccountRepositoryV8 repository;
	public AccountRequestModelValidatorV8(IAccountRepositoryV8 repository)
	{
		this.repository = repository;
		this.RuleFor(x => x.Name).NotEmpty();
		this.RuleFor(x => x.SSN).NotEmpty();
		this.RuleFor(x => x.Token).NotEmpty();
		this.RuleFor(x => x.Name).Must(this.AccountValidated).WithMessage("Account already exists");
	}

	private bool AccountValidated(string name)
	{
		return !this.repository.Exists(name);
	}

}
public class ValidationResultV8
{
	public bool Success { get; private set; } = true;
	public string Message { get; private set; } = "";

	public ValidationResultV8(bool success, string message)
	{
		this.Success = success;
		this.Message = message;
	}

	public ValidationResultV8()
	{
	}
}
#endregion

#region repository
public interface IAccountRepositoryV8
{
	bool Exists(string name);

	void Create(Account account);

	Task<Account> Find(int id);
}

public class AccountRepositoryV8 : IAccountRepositoryV8
{
	private ApplicationDbContext context;

	public AccountRepositoryV8(ApplicationDbContext context)
	{
		this.context = context;
	}

	public bool Exists(string name)
	{
		return this.context.Accounts.Any(x => x.Name == name);
	}
	public void Create(Account account)
	{
		this.context.Accounts.Add(account);
		context.SaveChanges();
	}

	public async Task<Account> Find(int id)
	{
		return await this.context.Accounts.FirstOrDefaultAsync(x => x.Id == id);
	}
}
#endregion

#region fbiService
public interface IFBIServiceV8
{
	Task<bool> VerifyWithFBI(string ssn);
}

public class FBIServiceV8 : IFBIServiceV8
{
	HttpClient client;
	public FBIServiceV8(HttpClient client)
	{
		this.client = client;
	}

	public async Task<bool> VerifyWithFBI(string ssn)
	{
		string response = await this.client.GetStringAsync("https://jsonplaceholder.typicode.com/posts/1");
		if (!string.IsNullOrWhiteSpace(response))
		{
			return true;
		}
		else
		{
			return false;
		}
	}
}
#endregion

#region dateService
public interface IDateServiceV8
{
	DateTime Now();
}

public class DateServiceV8 : IDateServiceV8
{
	public DateTime Now()
	{
		return DateTime.Now;
	}
}
```



#### Final Iteration 
This iteration is just testing what was in iteration 8.

We're introducing Fluent Assertions which is a great testing library.


```
public class UnitTestAccountV9Controller
{
	[Fact]
	public async void Test_Account_Create_200()
	{
		//setup
		Mock<IAccountServiceV9> serviceMock = new Mock<IAccountServiceV9>();
		serviceMock.Setup(x => x.Create(It.IsAny<AccountRequestModelV9>())).Returns(Task.FromResult(new ActionResultV9(true, default(object))));
		AccountV9Controller controller = new AccountV9Controller(serviceMock.Object);

		//act
		IActionResult result = await controller.Create(new AccountRequestModelV9());

		//verify
		result.Should().BeOfType<OkObjectResult>();
		(result as ObjectResult).StatusCode.Should().Be(200);
		serviceMock.Verify(x => x.Create(It.IsAny<AccountRequestModelV9>()));
	}

	[Fact]
	public async void Test_Account_Create_422()
	{
		//setup
		Mock<IAccountServiceV9> serviceMock = new Mock<IAccountServiceV9>();
		serviceMock.Setup(x => x.Create(It.IsAny<AccountRequestModelV9>())).Returns(Task.FromResult(new ActionResultV9(false, default(object))));
		AccountV9Controller controller = new AccountV9Controller(serviceMock.Object);

		//act
		IActionResult result = await controller.Create(new AccountRequestModelV9());

		//verify
		result.Should().BeOfType<StatusCodeResult>();
		(result as StatusCodeResult).StatusCode.Should().Be(422);
		serviceMock.Verify(x => x.Create(It.IsAny<AccountRequestModelV9>()));
	}


	[Fact]
	public async void Test_Account_Get_200()
	{
		//setup
		Mock<IAccountServiceV9> serviceMock = new Mock<IAccountServiceV9>();
		AccountV9Controller controller = new AccountV9Controller(serviceMock.Object);
		serviceMock.Setup(x => x.Get(It.IsAny<int>())).Returns(Task.FromResult(new ActionResultV9(true, new AccountResponseModelV9())));

		//act
		IActionResult result = await controller.Get(default(int));

		//verify
		result.Should().BeOfType<OkObjectResult>();
		(result as OkObjectResult).StatusCode.Should().Be(200);
		(result as OkObjectResult).Value.Should().NotBeNull();
		serviceMock.Verify(x => x.Get(It.IsAny<int>()));
	}

	[Fact]
	public async void Test_Account_Get_404()
	{
		//setup
		Mock<IAccountServiceV9> serviceMock = new Mock<IAccountServiceV9>();
		AccountV9Controller controller = new AccountV9Controller(serviceMock.Object);
		serviceMock.Setup(x => x.Get(It.IsAny<int>())).Returns(Task.FromResult(new ActionResultV9(false, default(object))));

		//act
		IActionResult result = await controller.Get(default(int));

		//verify
		result.Should().BeOfType<StatusCodeResult>();
		(result as StatusCodeResult).StatusCode.Should().Be(404);
		serviceMock.Verify(x => x.Get(It.IsAny<int>()));
	}

	[Fact]
	public async void Test_AccountService_Create_Success()
	{
		//setup
		Mock<IAccountRepositoryV9> repository = new Mock<IAccountRepositoryV9>();
		Mock<IAccountMapperV9> mapperMock = new Mock<IAccountMapperV9>();
		Mock<IAccountModelValidator> modelValidatorMock = new Mock<IAccountModelValidator>();
		Mock<IDateServiceV9> dateServiceMock = new Mock<IDateServiceV9>();
		Mock<IFBIServiceV9> fbiServiceMock = new Mock<IFBIServiceV9>();

		modelValidatorMock.Setup(x => x.Validate(It.IsAny<AccountRequestModelV9>())).Returns(new FluentValidation.Results.ValidationResult());
		fbiServiceMock.Setup(x => x.VerifyWithFBI(It.IsAny<string>())).Returns(Task.FromResult(true));
		mapperMock.Setup(x => x.MapEntityToResponse(It.IsAny<Account>())).Returns(new AccountResponseModelV9());
		repository.Setup(x => x.Create(It.IsAny<Account>()));

		IAccountServiceV9 service = new AccountServiceV9(repository.Object, modelValidatorMock.Object, fbiServiceMock.Object, dateServiceMock.Object, mapperMock.Object);

		//act
		ActionResultV9 result = await service.Create(new AccountRequestModelV9());

		//verify
		result.Success.Should().BeTrue();
		result.Object.Should().BeOfType<AccountResponseModelV9>();
		modelValidatorMock.Verify(x => x.Validate(It.IsAny<AccountRequestModelV9>()));
		mapperMock.Verify(x => x.MapEntityToResponse(It.IsAny<Account>()));
		repository.Verify(x => x.Create(It.IsAny<Account>()));
		fbiServiceMock.Verify(x => x.VerifyWithFBI(It.IsAny<string>()));
		dateServiceMock.Verify(x => x.Now());
	}

	[Fact]
	public async void Test_AccountService_Create_ValidationFailure()
	{
		//setup
		Mock<IAccountRepositoryV9> repository = new Mock<IAccountRepositoryV9>();
		Mock<IAccountMapperV9> mapperMock = new Mock<IAccountMapperV9>();
		Mock<IAccountModelValidator> modelValidatorMock = new Mock<IAccountModelValidator>();
		Mock<IDateServiceV9> dateServiceMock = new Mock<IDateServiceV9>();
		Mock<IFBIServiceV9> fbiServiceMock = new Mock<IFBIServiceV9>();

		modelValidatorMock.Setup(x => x.Validate(It.IsAny<AccountRequestModelV9>())).Returns(new ValidationResult(new List<ValidationFailure>() { new ValidationFailure(default(string), default(string)) }));
		fbiServiceMock.Setup(x => x.VerifyWithFBI(It.IsAny<string>())).Returns(Task.FromResult(true));
		mapperMock.Setup(x => x.MapEntityToResponse(It.IsAny<Account>())).Returns(new AccountResponseModelV9());
		repository.Setup(x => x.Create(It.IsAny<Account>()));

		IAccountServiceV9 service = new AccountServiceV9(repository.Object, modelValidatorMock.Object, fbiServiceMock.Object, dateServiceMock.Object, mapperMock.Object);

		//act
		ActionResultV9 result = await service.Create(new AccountRequestModelV9());

		//verify
		result.Success.Should().BeFalse();
		result.Object.Should().BeOfType<List<ValidationFailure>>();
		modelValidatorMock.Verify(x => x.Validate(It.IsAny<AccountRequestModelV9>()));
	}

	[Fact]
	public async void Test_AccountService_Create_FBIService_Failure()
	{
		//setup
		Mock<IAccountRepositoryV9> repository = new Mock<IAccountRepositoryV9>();
		Mock<IAccountMapperV9> mapperMock = new Mock<IAccountMapperV9>();
		Mock<IAccountModelValidator> modelValidatorMock = new Mock<IAccountModelValidator>();
		Mock<IDateServiceV9> dateServiceMock = new Mock<IDateServiceV9>();
		Mock<IFBIServiceV9> fbiServiceMock = new Mock<IFBIServiceV9>();

		modelValidatorMock.Setup(x => x.Validate(It.IsAny<AccountRequestModelV9>())).Returns(new FluentValidation.Results.ValidationResult());
		fbiServiceMock.Setup(x => x.VerifyWithFBI(It.IsAny<string>())).Returns(Task.FromResult(false));
		mapperMock.Setup(x => x.MapEntityToResponse(It.IsAny<Account>())).Returns(new AccountResponseModelV9());
		repository.Setup(x => x.Create(It.IsAny<Account>()));

		IAccountServiceV9 service = new AccountServiceV9(repository.Object, modelValidatorMock.Object, fbiServiceMock.Object, dateServiceMock.Object, mapperMock.Object);

		//act
		ActionResultV9 result = await service.Create(new AccountRequestModelV9());

		//verify
		result.Success.Should().BeFalse();
		result.Object.Should().BeOfType<ValidationResultV9>();
		fbiServiceMock.Verify(x => x.VerifyWithFBI(It.IsAny<string>()));
	}

	[Fact]
	public async void Test_AccountService_Get_Found()
	{
		//setup
		Mock<IAccountRepositoryV9> repository = new Mock<IAccountRepositoryV9>();
		Mock<IAccountMapperV9> mapperMock = new Mock<IAccountMapperV9>();
		Mock<IAccountModelValidator> modelValidatorMock = new Mock<IAccountModelValidator>();
		Mock<IDateServiceV9> dateServiceMock = new Mock<IDateServiceV9>();
		Mock<IFBIServiceV9> fbiServiceMock = new Mock<IFBIServiceV9>();

		modelValidatorMock.Setup(x => x.Validate(It.IsAny<AccountRequestModelV9>())).Returns(new ValidationResult());
		fbiServiceMock.Setup(x => x.VerifyWithFBI(It.IsAny<string>())).Returns(Task.FromResult(true));
		mapperMock.Setup(x => x.MapEntityToResponse(It.IsAny<Account>())).Returns(new AccountResponseModelV9());
		repository.Setup(x => x.Find(It.IsAny<int>())).Returns(Task.FromResult<Account>(new Account()));

		IAccountServiceV9 service = new AccountServiceV9(repository.Object, modelValidatorMock.Object, fbiServiceMock.Object, dateServiceMock.Object, mapperMock.Object);

		//act
		ActionResultV9 result = await service.Get(default(int));

		//verify
		result.Success.Should().BeTrue();
		result.Object.Should().BeOfType<AccountResponseModelV9>();
		mapperMock.Verify(x => x.MapEntityToResponse(It.IsAny<Account>()));
		repository.Verify(x => x.Find(It.IsAny<int>()));
	}

	[Fact]
	public async void Test_AccountService_Get_Not_Found()
	{
		//setup
		Mock<IAccountRepositoryV9> repository = new Mock<IAccountRepositoryV9>();
		Mock<IAccountMapperV9> mapperMock = new Mock<IAccountMapperV9>();
		Mock<IAccountModelValidator> modelValidatorMock = new Mock<IAccountModelValidator>();
		Mock<IDateServiceV9> dateServiceMock = new Mock<IDateServiceV9>();
		Mock<IFBIServiceV9> fbiServiceMock = new Mock<IFBIServiceV9>();

		modelValidatorMock.Setup(x => x.Validate(It.IsAny<AccountRequestModelV9>())).Returns(new ValidationResult());
		fbiServiceMock.Setup(x => x.VerifyWithFBI(It.IsAny<string>())).Returns(Task.FromResult(true));
		mapperMock.Setup(x => x.MapEntityToResponse(It.IsAny<Account>())).Returns(new AccountResponseModelV9());
		repository.Setup(x => x.Find(It.IsAny<int>())).Returns(Task.FromResult<Account>(null));

		IAccountServiceV9 service = new AccountServiceV9(repository.Object, modelValidatorMock.Object, fbiServiceMock.Object, dateServiceMock.Object, mapperMock.Object);

		//act
		ActionResultV9 result = await service.Get(default(int));

		//verify
		result.Success.Should().BeFalse();
		repository.Verify(x => x.Find(It.IsAny<int>()));
	}

	[Fact]
	public void Test_Mapper_MapRequestToEntity()
	{
		//setup
		IAccountMapperV9 mapper = new AccountMapperV9();

		var request = new AccountRequestModelV9(1, "A", "000-05-1120", DateTime.Parse("7/3/2018 4:41:42 PM"), "A");

		//act
		Account entity = mapper.MapRequestToEntity(request);

		//verify
		entity.DateCreated.Should().Be(DateTime.Parse("7/3/2018 4:41:42 PM"));
		entity.Id.Should().Be(1);
		entity.Name.Should().Be("A");
		entity.SSN.Should().Be("000-05-1120");
		entity.Token.Should().Be("A");
	}

	[Fact]
	public void Test_Mapper_MapEntityToResponse()
	{
		//setup
		IAccountMapperV9 mapper = new AccountMapperV9();

		var entity = new Account(1, "A", "000-05-1120", DateTime.Parse("7 /3/2018 4:41:42 PM"), "A");

		//act
		AccountResponseModelV9 response = mapper.MapEntityToResponse(entity);

		//verify
		response.DateCreated.Should().Be(DateTime.Parse("7/3/2018 4:41:42 PM"));
		response.Id.Should().Be(1);
		response.Name.Should().Be("A");
		response.SSN.Should().Be("000-05-1120");
	}

	[Fact]
	public void Test_ModelValidator_Name_NotEmpty_Valid()
	{
		//setup
		Mock<IAccountRepositoryV9> repositoryMock = new Mock<IAccountRepositoryV9>();
		repositoryMock.Setup(x => x.Exists(It.IsAny<string>())).Returns(false);
		AccountRequestModelValidatorV9 validator = new AccountRequestModelValidatorV9(repositoryMock.Object);

		//act and verify
		validator.ShouldNotHaveValidationErrorFor(x => x.Name, "A");
	}

	[Fact]
	public void Test_ModelValidator_Name_NotEmpty_Invalid()
	{
		//setup
		Mock<IAccountRepositoryV9> repositoryMock = new Mock<IAccountRepositoryV9>();
		repositoryMock.Setup(x => x.Exists(It.IsAny<string>())).Returns(false);
		AccountRequestModelValidatorV9 validator = new AccountRequestModelValidatorV9(repositoryMock.Object);

		//act and verify
		validator.ShouldHaveValidationErrorFor(x => x.Name, "");
		validator.ShouldHaveValidationErrorFor(x => x.Name, null as string);
	}

	[Fact]
	public void Test_ModelValidator_NameUnique_Valid()
	{
		//setup
		Mock<IAccountRepositoryV9> repositoryMock = new Mock<IAccountRepositoryV9>();
		repositoryMock.Setup(x => x.Exists(It.IsAny<string>())).Returns(false);
		AccountRequestModelValidatorV9 validator = new AccountRequestModelValidatorV9(repositoryMock.Object);

		//act and verify
		validator.ShouldNotHaveValidationErrorFor(x => x.Name, "A");
	}

	[Fact]
	public void Test_ModelValidator_NameUnique_Invalid()
	{
		//setup
		Mock<IAccountRepositoryV9> repositoryMock = new Mock<IAccountRepositoryV9>();
		repositoryMock.Setup(x => x.Exists(It.IsAny<string>())).Returns(true);
		AccountRequestModelValidatorV9 validator = new AccountRequestModelValidatorV9(repositoryMock.Object);

		//act and verify
		validator.ShouldHaveValidationErrorFor(x => x.Name, "A");
	}


	[Fact]
	public void Test_ModelValidator_SSN_NotEmpty_Valid()
	{
		//setup
		Mock<IAccountRepositoryV9> repositoryMock = new Mock<IAccountRepositoryV9>();
		repositoryMock.Setup(x => x.Exists(It.IsAny<string>())).Returns(false);
		AccountRequestModelValidatorV9 validator = new AccountRequestModelValidatorV9(repositoryMock.Object);

		//act and verify
		validator.ShouldNotHaveValidationErrorFor(x => x.SSN, "000-05-1120");
	}

	[Fact]
	public void Test_ModelValidator_SSN_NotEmpty_Invalid()
	{
		//setup
		Mock<IAccountRepositoryV9> repositoryMock = new Mock<IAccountRepositoryV9>();
		repositoryMock.Setup(x => x.Exists(It.IsAny<string>())).Returns(false);
		AccountRequestModelValidatorV9 validator = new AccountRequestModelValidatorV9(repositoryMock.Object);

		//act and verify
		validator.ShouldHaveValidationErrorFor(x => x.SSN, string.Empty);
	}

	[Fact]
	public void Test_ModelValidator_Token_NotEmpty_Valid()
	{
		//setup
		Mock<IAccountRepositoryV9> repositoryMock = new Mock<IAccountRepositoryV9>();
		repositoryMock.Setup(x => x.Exists(It.IsAny<string>())).Returns(false);
		AccountRequestModelValidatorV9 validator = new AccountRequestModelValidatorV9(repositoryMock.Object);

		//act and verify
		validator.ShouldNotHaveValidationErrorFor(x => x.Token, "A");
	}

	[Fact]
	public void Test_ModelValidator_Token_NotEmpty_Invalid()
	{
		//setup
		Mock<IAccountRepositoryV9> repositoryMock = new Mock<IAccountRepositoryV9>();
		repositoryMock.Setup(x => x.Exists(It.IsAny<string>())).Returns(false);
		AccountRequestModelValidatorV9 validator = new AccountRequestModelValidatorV9(repositoryMock.Object);

		//act and verify
		validator.ShouldHaveValidationErrorFor(x => x.Token, "");
		validator.ShouldHaveValidationErrorFor(x => x.Token, null as string);
	}


	[Fact]
	public void Test_Account_Repository_Create()
	{
		//setup
		DbContextOptionsBuilder optionsBuilder = new DbContextOptionsBuilder();
		optionsBuilder.UseInMemoryDatabase("test1");
		var context = new ApplicationDbContext(optionsBuilder.Options);
		IAccountRepositoryV9 repository = new AccountRepositoryV9(context);

		//act
		repository.Create(new Account());

		//verify
		context.Accounts.Count().Should().Be(1);
	}


	[Fact]
	public void Test_Account_Repository_Exists_Record_Exists()
	{
		//setup
		DbContextOptionsBuilder optionsBuilder = new DbContextOptionsBuilder();
		optionsBuilder.UseInMemoryDatabase("test1");
		var context = new ApplicationDbContext(optionsBuilder.Options);
		IAccountRepositoryV9 repository = new AccountRepositoryV9(context);
		context.Accounts.Add(new Account(default(int), "test", default(string), default(DateTime), default(string)));
		context.SaveChanges();

		//act
		bool exists = repository.Exists("test");

		//verify
		exists.Should().BeTrue();
	}

	[Fact]
	public void Test_Account_Repository_Exists_Record_Not_Exists()
	{
		//setup
		DbContextOptionsBuilder optionsBuilder = new DbContextOptionsBuilder();
		optionsBuilder.UseInMemoryDatabase("test1");
		var context = new ApplicationDbContext(optionsBuilder.Options);
		IAccountRepositoryV9 repository = new AccountRepositoryV9(context);

		//act
		bool exists = repository.Exists("testing");
		exists.Should().BeFalse();

	}

	[Fact]
	public async void Test_Account_Repository_Find_Record_Exists()
	{
		//setup
		DbContextOptionsBuilder optionsBuilder = new DbContextOptionsBuilder();
		optionsBuilder.UseInMemoryDatabase("test1");
		var context = new ApplicationDbContext(optionsBuilder.Options);
		IAccountRepositoryV9 repository = new AccountRepositoryV9(context);
		context.Accounts.Add(new Account(1, default(string), default(string), default(DateTime), default(string)));
		context.SaveChanges();

		//act
		Account record = await repository.Find(1);

		//verify
		record.Should().NotBeNull();
	}

	[Fact]
	public async void Test_Account_Repository_Find_Record_Not_Exists()
	{
		//setup
		DbContextOptionsBuilder optionsBuilder = new DbContextOptionsBuilder();
		optionsBuilder.UseInMemoryDatabase("test1");
		var context = new ApplicationDbContext(optionsBuilder.Options);
		IAccountRepositoryV9 repository = new AccountRepositoryV9(context);

		//act
		Account record = await repository.Find(1);

		//verify
		record.Should().BeNull();

	}

	[Fact]
	public void Test_DatetimeService()
	{
		//setup
		IDateServiceV9 service = new DateServiceV9();
		DateTime now = DateTime.Now;

		//act
		DateTime result = service.Now();

		//verify
		result.Should().BeCloseTo(now, 1000);
	}


	[Fact]
	public async void IntegrationTest_FBIService_Success()
	{
		//setup
		IFBIServiceV9 service = new FBIServiceV9("https://jsonplaceholder.typicode.com/posts/1");

		//act
		bool result = await service.VerifyWithFBI(default(string));

		//verify
		result.Should().BeTrue();
	}

	[Fact]
	public async void IntegrationTest_FBIService_Failure()
	{
		//setup
		IFBIServiceV9 service = new FBIServiceV9("https://jsonplaceholder.typicode.com/posts/fail");

		//act
		bool result = await service.VerifyWithFBI(default(string));

		//verify
		result.Should().BeFalse();
	}
```


We made lots of changes to really make our code testable. 
We introduced Fluent Validation to make testing easier for individual fields. 
We wrote all of our tests including an integration test for the FBI service

There is at least one big issue I still see with these tests. There is a lot of 
repetitive setup code. I would take the time to build factories to build your default mocks and
make the code as DRY as possible. 
  
We don't have end to end tests. I'll leave that for another article.   



### Summary

Use interfaces and dependency injection. Never directly depend on the database, file system, date libraries or network in your application. Unit tests 
should be deterministic and you can't do that if you have these types of dependencies.

Only test the actual unit you should be testing not every little detail. Test should not overlap.

Write single responsibility code. 

Write fast tests whenever possible.

If you don't write SOLID code your project will not handle requirement changes and will turn into a ball of mud.
By enforcing SOLID from the beginning of a project, even though it takes more code, it is significantly more likely that your project can evolve.
Methods that do different things based on a parameter or setting should be two functions. 
Classes that handle different things should be split and the functionality combined with a facade or a mediator. "Different things" is subjective and it's 
difficult to give you hard and fast rules. Generally if the code is difficult to work on and test and it's not doing something hard then it should be refactored.

That's the end. I hope you learned something that makes coding easier. 




