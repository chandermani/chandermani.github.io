---
layout: post
title: Unit testing Web API 2 routes made easy with xUnit (Data driven)
author: Chandermani
comments: true
cover: /assets/images/2017-12-08-data-driven-unit-testing-api-routes-xunit.png
tags:
- xunit
- testing
- unit testing
- webapi
- routing
- api
- data driven tests
---
If you are building your API using [ASP.Net Web API 2](https://www.asp.net/web-api) framework, this post will demonstrate you how easy it is to test you API routes. 

## Why test routes?
Route resolution on Web API (as well ASP.Net MVC) is about matching the incoming request route(url) with the routes defined in the *routing table* in a top down fashion and returning the first match. There is always a probability that addition of a new route in the routing table hides/overrides one or more existing routes. This can easily happen for route templates that have placeholders (such as this - `api/{controller}/{action}/{id}`).

Also if you are building an API for your service/product and your API supports versioning (which it should), unit testing routes can come in handy. Imagine your evolving API endpoint such as:
```
http://awesome.api/v1.0/users  
http://awesome.api/v1.1/users  
http://awesome.api/v2.0/users
```

It will be nightmare for the consumers of your API if incorrect controllers/actions get invoked when you release a new API version. All because of incorrect route resolution. Integration test can catch such issues, but a early feedback during unit testing is very valuable. 

This post is about how to implement route tests to making sure correct API endpoints are hit based on the version request and there are no regression issues.

I have also created a sample project to support this post. Download it from my [GitHub repo](https://github.com/chandermani/UnitTestingAPIRoutes).

## Setup
Assuming we already have a API to test, create a new class library project and install **NuGet** packages for **xUnit**, **Moq** and **WebApi**.

*Follow [this post](https://xunit.github.io/docs/getting-started-desktop.html) to setup xUnit as your unit testing framework in Visual Studio if case you are using xUnit for the fist time.*

Write a trivial test (`Fact`) and test it using the **xUnit console runner** or **Visual Studio test runner** to make sure xUnit is working as expected.

Next add your API project reference to the newly created unit test project.

## Begin Testing
Let's look at a sample API controller that we are going to write tests against. The sample application I have shared link above uses this controller (`UsersController`).

```cs
public class UsersController : ApiController
    {
        private List<User> userStore = new List<User>()
        {
            new User() { Id=1, Name="Tom"},
            ...
        };
        
        [Route("v1/users")]
        [Route("v2/users")]
        public IEnumerable<User> Get()
        {
            return userStore;
        }

        [Route("v2/users/special")]
        public IEnumerable<User> GetSpecial()
        {
            return userStore.Take(3);
        }

        [Route("v1/users/{id:int}")]
        [Route("v2/users/{id:int}")]
        public User Get(int id)
        {
            return userStore.FirstOrDefault(u => u.Id == id);
        }

        [Route("v1/users")]
        [Route("v2/users")]
        public void Post([FromBody]User user)
        {
            userStore.Add(user);
        }

        [Route("v1/users/{id:int}")]
        [Route("v2/users/{id:int}")]
        public void Put(int id, [FromBody]User user)
        {
            ...
        }
        ...
    }
```
With *attribute based routing* some of the standard routes for above controller look like:

```
// V1 Routes
http://localhost/api/v1/users
http://localhost/api/v1/users/1

// V2 Routes
http://localhost/api/v2/users
http://localhost/api/v2/users/special   <- Only in v2
http://localhost/api/v2/users/1
```
Let's test some of these routes.

Create a test class for route testing and add a new test:

```cs
[Fact]
public void Should_Resolve_V1_Get_Users_Route()
{
    var configuration = new HttpConfiguration();

    WebApiConfig.Register(configuration);

    var controllerTypeResolver = new Mock<IHttpControllerTypeResolver>();

    var controllerTypes = GetAllControllerTypes();


    controllerTypeResolver
        .Setup(r => r.GetControllerTypes(It.IsAny<IAssembliesResolver>()))
        .Returns(controllerTypes);

    configuration.Services.Replace(typeof(IHttpControllerTypeResolver), 
        controllerTypeResolver.Object);

    configuration.EnsureInitialized();

    var request = new HttpRequestMessage(HttpMethod.Get, "http://localhost/api/v1/users");

    var routeData = configuration.Routes.GetRouteData(request);

    request.SetConfiguration(configuration);
    
    if (routeData != null)
    {
        // For incorrectly formed route url, route data is null. This request may fail later and we can check status code.
        request.SetRouteData(routeData);
    }

    var controllerSelector = configuration.Services.GetHttpControllerSelector();
    var actionSelector = configuration.Services.GetActionSelector();

    //act
    var controllerDescriptor = controllerSelector.SelectController(request);
    var controllerContext = new HttpControllerContext(configuration, routeData, request)
    {
        ControllerDescriptor = controllerDescriptor,
        RequestContext = new HttpRequestContext()
        {
            Configuration = configuration,
            RouteData = routeData
        }
    };

    var actionDescriptor = actionSelector.SelectAction(controllerContext);

    // assert
    controllerDescriptor.ControllerType.Should().Be(typeof(UsersController));

    actionDescriptor.ActionName.Should().Be("Get");

    actionDescriptor.GetParameters().Select(p => p.ParameterName).Should().BeEquivalentTo(new string[] { });
}
```
Phew! Lots of ceremony before the actual route assertions can be made. As it turns out there are several moving pieces (framework) that we need to setup before we can actually test route resolution. Something that we would not want to do for every route we test.

Before we see how to better the implementation, let's quickly see what's the test doing. 

### Test Internals
At a high level this is what happens:
1. A new `HttpConfiguration` object is created and initialized with `WebApiConfig.Register(configuration);`
2. The *Controller type resolver* is setup. This lets us register controllers with the framework.
3. A new `HttpRequestMessage` is created encapsulating the route we want to test. This is the test input.
4. *Route data* is constructed.
5. The configuration build in *Step 2* is linked to the request.
6. `IHttpControllerSelector` and `IHttpActionSelector` are setup. These classes resolve the *controller* and the *action* based on request and route data.
7. The selectors are then used to resolve the controller (`HttpControllerDescriptor`) and the action (`HttpActionDescriptor`). These classes describe the controller and action respectively.
8. A number of asserts are performed on these descriptors.

This test is an excellent exercise in understanding how WebApi route resolution works.

Coming back to the pertinent question. How can we do better? 
For starter look at this library [https://mytestedasp.net/](https://mytestedasp.net/) that does support route testing and much more. 

An alternate approach that I highlight in this post is using **data driven tests** feature of xUnit. You can even marry the capabilities of the above library and approach that I outline below.

## Data driven testing with xUnit
While we can pull out some of the code from the above unit test into helper classes/function there is still a lot of code required to unit test each individual route.

xUnit has a capability of passing data(set) to a unit test. This allows us to run a single unit test with varying data. Here is a sample from xUnit's help pages for data driven tests:

```cs
[Theory]
[InlineData(3)]
[InlineData(5)]
public void MyFirstTheory(int value)
{
    Assert.True(IsOdd(value));
}

bool IsOdd(int value)
{
    return value % 2 == 1;
}
```
The test uses `Theory` attribute instead of `Fact`. The *theory* can have one or more inputs as shown above. 

One great thing about xUnit is that it is very flexible in terms of the *theory* data source. It can take **inline** data(`InlineData` above) or derive it from external data source. The theory input data can be either an array of objects or can be a strongly typed class.

Let's look at how can we use the *theory* construct to improve our tests.

With data driven test we can live with the ceremony code in the test (as show above), instead we call the same test multiple times with different route each time. This requires we define the test input parameter (function parameter), that can help us setup the test and make necessary assertions.

Here is the *test input* class:

```cs
public class RouteTheoryInput
 {
     public RouteTheoryInput()
     {
         this.ParameterNames = new string[] { };
         this.HttpMethod = HttpMethod.Get;
         this.ResponseStatusCode = HttpStatusCode.OK;
     }

     public string Endpoint { get; set; }   // Test Input
     public Type ControllerType { get; set; }   // Thing to assert
     public string ActionName { get; set; }     // Thing to assert
     public string[] ParameterNames { get; set; }   // Thing to Assert. Assert only the parameter name, not types
     public HttpMethod HttpMethod { get; set; } // Test Input
     public HttpStatusCode ResponseStatusCode { get; set; } // Thing to assert
 }
```
The `RouteTheoryInput` class has some properties that serve as the test input and some properties that we can assert on (`ParameterName`, `ResponseStatusCode`, `ControllerType`, `ActionName`).

Here is the updated  `Should_Resolve_V1_Get_Users_Route` test rewritten using the test input class (only differences detailed below):
```cs
[Theory]
[MemberData(...)]   // Detailed later
public void Should_Resolve_V1_Routes_For_Users_API(RouteTheoryInput input) {
    //Arrange
    ...
    var request = new HttpRequestMessage(input.HttpMethod, input.Endpoint);

    ...
    //Assert
    controllerDescriptor.ControllerType.Should().Be(input.ControllerType);

    actionDescriptor.ActionName.Should().Be(input.ActionName);

    actionDescriptor.GetParameters().Select(p => p.ParameterName).Should().BeEquivalentTo(input.ParameterNames);
}
```

The only missing part of the puzzle now is how to construct the input data and pass it to the test. xUnit provides `MemberData` attribute to pass input (object) into a test.

### Using MemberData to pass test inputs
Define a class with a collection of input routes (`RouteTheoryInput` objects) that we want to test:
```cs
public class RouteInputsV1
{
    private const string BASEURL = "http://localhost/api/v1";
    public static TheoryData<RouteTheoryInput> UserEndpoints = new TheoryData<RouteTheoryInput>()
    {
        new RouteTheoryInput() {Endpoint = $"{BASEURL}/users", ControllerType = typeof(UsersController), ActionName = "Get", ParameterNames = new string[] { } },
        new RouteTheoryInput() {Endpoint = $"{BASEURL}/users/1", ControllerType = typeof(UsersController), ActionName = "Get", ParameterNames = new string[] { "id" } },
        new RouteTheoryInput() {Endpoint = $"{BASEURL}/users", ControllerType = typeof(UsersController), ActionName = "Post", ParameterNames = new string[] { "user" }, HttpMethod=HttpMethod.Post },
        ...
    };
}
```
Now reference `RouteInputsV1` in `MemberData` attribute declaration on the test
```cs
[MemberData(nameof(RouteInputsV1.UserEndpoints), MemberType = typeof(RouteInputsV1))]
```
The first parameter to `MemberData` is the name of the public static field on the `RouteInputsV1` with test routes. `MemberType` refers to the class type that holds the input data (routes) `RouteInputsV1`.

We can now run the new data driven test and it will execute the same route test for each of the route defined in the route collection above (`RouteInputsV1.UserEndpoints`).

*That's a data driven test for you!*

A word of caution here. Data driven tests albeit powerful should be used with caution. Structure your test inputs and tests in a manner that does not create confusion. Do not use a single test input collection and a single test to test everything. One input collection should only cater to one scenario. In the example above the input collection only target V1 version of Users API.

While the tests now should run successfully, the execution is a bit slow. There is a need for optimization.
## Optimization
If we look back at the test above, the test setup involves initialization of the `HttpConfiguration` including setting up the controller and routing table every time a test is executed. To save some time this activity can be done once. With xUnit we can do it using the concept of **Class Fixture**.   
But why not constructor or some *setup attribute* like other frameworks? Because

*xUnit.net creates a new instance of the test class for every test that is run*

### Class Fixture to share context
From the xUnit documentation:
> Sometimes test context creation and cleanup can be very expensive. If you were to run the creation and cleanup code during every test, it might make the tests slower than you want. You can use the class fixture feature of xUnit.net to share a single object instance among all tests in a test class. 

Add a new fixture to initialize the `HttpConfiguration`

```cs
 public class RouteFixture
    {
        public HttpConfiguration Configuration { get; set; }

        public RouteFixture()
        {
            this.Configuration = new HttpConfiguration();
            WebApiConfig.Register(this.Configuration);

            var controllerTypeResolver = new Mock<IHttpControllerTypeResolver>();
            var controllerTypes = GetAllControllerTypes();


            controllerTypeResolver.Setup(r => r.GetControllerTypes(It.IsAny<IAssembliesResolver>())).Returns(controllerTypes);
            this.Configuration.Services.Replace(typeof(IHttpControllerTypeResolver), controllerTypeResolver.Object);
            this.Configuration.EnsureInitialized();
        }


        private Collection<Type> GetAllControllerTypes()
        {
            Collection<Type> controllerTypes = new Collection<Type>();
            var baseControllerType = typeof(ApiController);
            Assembly.GetAssembly(typeof(MvcApplication))
                .GetTypes().Where(t => baseControllerType.IsAssignableFrom(t)).ToList().ForEach(t => controllerTypes.Add(t));

            return controllerTypes;
        }
    }
```
The fixture implementation above does the necessary WebApi initialization.

To use the class fixture in the test class change the class definition to:
```cs
 public class APIRouteTests : IClassFixture<RouteFixture>
    {
        private readonly RouteFixture routeFixture;

        public APIRouteTests(RouteFixture routeFixture)
        {
            this.routeFixture = routeFixture;
        }
    }
```

xUnit will inject the same `RouteFixture` instance into the test every time a test is executed. Now it's just a matter of using the fixture in your test:

```cs
 // Arrange
var configuration = routeFixture.Configuration;

// Removed code to setup routes and ControllerTypeResolver.
var request = new HttpRequestMessage(input.HttpMethod, input.Endpoint);
var routeData = configuration.Routes.GetRouteData(request);
```

And we have a much faster and cleaner test!

Look at the sample repository I have created [here](https://github.com/chandermani/UnitTestingAPIRoutes) to see working version of the tests with some more refactoring.