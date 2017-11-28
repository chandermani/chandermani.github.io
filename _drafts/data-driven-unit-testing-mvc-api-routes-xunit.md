If you are building your API using [ASP.Net Web API 2](https://www.asp.net/web-api) framework, this post will show you how easy it is to test you API routes. 

Why is route testing required
Route resolution on Web API (as well ASP.Net MVC) is about matching a routing table in a top down fashion and returning the first match. There is always a probability that addition of a new route in the routing table hides/overrides one or more existing routes. This can easily happen for route templates that have placeholders (such as this `api/{controller}/{action}/{id}`).

Also if you are building an API for your service/product and your API supports versioning (which it should), unit testing routes can come in handy. Imagine your evolving API endpoint such as:
http://awesome.api/v1.0/users
http://awesome.api/v1.1/users
http://awesome.api/v2.0/users

It will be nightmare for the consumers of your API if incorrect controllers/actions get invoked when you release a new API version. All because of incorrect route resolution. Integration test can catch this issue, but a early feedback with unit testing can be valuable. 

This post is not about how to do effective API versioning but making sure correct API endpoints are hit based on the version request and there are no regression issues.

A sample project highlighting how unit testing can be done for API routes can be download from my GitHub repo.

## Setup
Assuming we already have a API to test, create a new class library project and install nuget packages for xUnit, Moq and WebAPI. Follow [this post](https://xunit.github.io/docs/getting-started-desktop.html) to setup xUnit as your unit testing framework in Visual Studio. Write a trivial test (Fact) and test it using the xUnit console runner or Visual Studio test runner to make sure xUnit is working as expected.

Add your API project reference to the newly created unit test project.

## Begin Testing

Let's look at a sample API controller that we are going to write tests against. The sample application on GitHub uses the controller (UsersController).

```
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
The controller above uses attribute based routing and some of the standard routes for above controller look like:

```
// V1 Routes
http://localhost/api/v1/users
http://localhost/api/v1/users/1

// V2 Routes
http://localhost/api/v2/users
http://localhost/api/v2/users/special
http://localhost/api/v2/users/1
```
Let's test these routes.

Create a test class for route testing and add a new test:

```
[Fact]
public void Should_Resolve_V1_Get_Users_Route()
{
    var configuration = new HttpConfiguration();

    WebApiConfig.Register(configuration);

    var controllerTypeResolver = new Mock<IHttpControllerTypeResolver>();

    var controllerTypes = GetAllControllerTypes();


    controllerTypeResolver.Setup(r => r.GetControllerTypes(It.IsAny<IAssembliesResolver>())).Returns(controllerTypes);
    configuration.Services.Replace(typeof(IHttpControllerTypeResolver), controllerTypeResolver.Object);
    configuration.EnsureInitialized();

    var request = new HttpRequestMessage(HttpMethod.Get, "http://localhost/api/v1/users");

    var routeData = configuration.Routes.GetRouteData(request);

    request.SetConfiguration(configuration);
    if (routeData != null)
    {
        // For incorrectly formed route url,route data is null. This request may fail later and we can check status code.
        request.SetRouteData(routeData);
    }

    var controllerSelector = configuration.Services.GetHttpControllerSelector();
    var actionSelector = configuration.Services.GetActionSelector();

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

    //act
    var controller = controllerSelector.SelectController(request);
    var action = actionSelector.SelectAction(controllerContext);

    // assert
    controller.ControllerType.Should().Be(typeof(UsersController));

    action.ActionName.Should().Be("Get");

    action.GetParameters().Select(p => p.ParameterName).Should().BeEquivalentTo(new string[] { });
}
```
As we can see there is a lot of ceremony before the actual route assertions can be made. We do not want to do this for every route we test.

Data driven testing with xUnit
While we can pull out some of the code above int helper classes/function there is still a lot of code required to unit test each individual route.

xUnit has a capability of passing data(set) to a unit test. This allows us to run a single unit test with varying data. Here is a sample from xUnit's help pages

```
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
Instead of `Fact` we now use `Theory`. The theory can have one or more inputs as shown above. 

The great thing about xUnit is that it is very flexible in terms of the Theory data source. It can take inline (as above) or derive it from another data source. The theory input data can be just a array of objects or can be a strongly typed class.

To reduce the boilerplate and test multiple routes using a single unit test we can define our own data source as a collection of strongly type objects. This is how the test input object in the collection looks like:

```
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
The `RouteTheoryInput` input class has some properties that serve as the test input and some properties that we can assert on (`ParameterName`, `ResponseStatusCode`, `ControllerType`, `ActionName`).

Here is the `Should_Resolve_V1_Get_Users_Route` test (see above) rewritten using test inputs:
```
[Theory]
[MemberData(...)]   // Detailed later
public void Should_Resolve_V1_Routes_For_Users_API(RouteTheoryInput input) {
    //Arrange
    ...
    var request = new HttpRequestMessage(input.HttpMethod, input.Endpoint);

    ...
    //Assert
    controller.ControllerType.Should().Be(input.ControllerType);

    action.ActionName.Should().Be(input.ActionName);

    action.GetParameters().Select(p => p.ParameterName).Should().BeEquivalentTo(input.ParameterNames);
}
```

The only missing part of the puzzle now is how to construct the input data and pass it the test. xUnit provides MemberData attribute to pass input (object) into a test.

Using MemberData to pass test inputs
To illustrate how to pass `RouteTheoryInput` to the test above. Lets define a class will a collection on inputs (RouteTheoryInput objects) that we want to test:
```
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
Now reference these inputs in the MemberData attribute declaration on the test
```
[MemberData(nameof(RouteInputsV1.UserEndpoints), MemberType = typeof(RouteInputsV1))]
```
The first parameter to MemeberData is the name of the public static member on the test input class that will provide the test data. `MemberType` refers to the class that holds the input data.

*That's a data driven test for you!*

A word of caution. Data driven tests are really powerful, but be careful. Structure your test inputs and tests in a manner that does not create confusion. Do not use a single test input collection and a single test to test everything. The input collection should on cater to one scenario. In the example above the input collection only target V1 version of Users API.

There is still some scope of optimizations
Optimization
