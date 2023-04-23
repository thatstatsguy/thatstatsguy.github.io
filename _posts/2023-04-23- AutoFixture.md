---
layout: post
title:  Unit Testing with Autofixture
date:   2023-04-23
description: Making test unit testing faster, cleaner and less brittle with one library
tags: AutoFixture AutoMocking C#
categories: Testing
---
Today we're discussing how to make your unit testing experience 100% better with [AutoFixture](https://github.com/AutoFixture/AutoFixture). When I'm developing a solution, there's that inevitable moment where you make a change to a piece of code which requires refactoring 37 unit tests in order to get the code to compile - no my favourite part of developing a solution.

Autofixture comes in super handy to prevent this by (a) supplying test fixture values for non-important elements of your unit tests and (b) making your code base more maintainable by making unit tests less brittle.

## Supplying test values for non-essential elements of unit tests

As an example, suppose we're testing logic for an addition method.

```
public int Add (int a, int b, string validationError)
{
    if (a < 0 || b < 0)
    {
        throw new Exception(validationError);
    }
    else
    {
        return a + b;
    }
}
```

In the example above, we throw an exception `a` or `b` are less than 0. When testing this method to ensure that the addition is correct **when the inputs are valid** the validation error is not used in the test. Obviously you could specifying the function as `public int Add (int a, int b, string validationError = "")` to avoid having to specify the message, but suppose that this always had to be specified.

To test the above method one could set up the following test
```
[Fact(DisplayName = "Test Addition without AutoFixture")]
public void TestAddition1()
{
    //arrange
    var input1 = 1;
    var input2 = 2;
    var expectedResult = input1 + input2;
    //act
    var actual = Add(input1, input2, "");
    //assert
    Assert.Equal(expectedResult, actual);
}
```
The test does the job, but we really don't care about the validation message as we're not testing the scenario in which the exception will be thrown. Trying again with AutoFixture, we create the message on the fly in the test.

```
[Fact(DisplayName = "Test Addition with AutoFixture")]
    public void TestAddition2()
    {
        //arrange
        var input1 = 1;
        var input2 = 2;
        var expectedResult = input1 + input2;
        var fixture = new Fixture();

        //act
        var actual = Add(input1, input2, fixture.Create<string>());
        //assert
        Assert.Equal(expectedResult, actual);
    }
```

Great, we've generated some test string value. If you're having to do this multiple times within a test this can be a real time saver. Also, if you're generating records on the fly this can assist in making your tests less brittle as any new parameter is automagically filled in by AutoFixture. Keep in mind doing this only makes sense if the data isn't relevant to the test scenario you're performing.

I'm lazy and want to saved myself a few extra keystrokes - we can use the AutoData attribute to pass in values into our test as if we were inlining data using `Theory` instead of `Fact`

```
[Theory(DisplayName = "Test Addition with AutoFixture AutoData Part 1")]
[AutoData]
public void TestAddition3(string validationError)
{
    //arrange
    var input1 = 1;
    var input2 = 2;
    var expectedResult = input1 + input2;

    //act
    var actual = Add(input1, input2, validationError);
    //assert
    Assert.Equal(expectedResult, actual);
}
```

Okay we're getting somewhere now, the validation message fixture is passed in via the method parameters. In simple scenarios like addition we may even want to fixture the inputs to our function because as long as it's a int greater than 0 it should work. It just so happens that AutoFixture integers are always generated with values greater than 0 so we can adjust our test to be

```
[Theory(DisplayName = "Test Addition with AutoFixture AutoData Part 2")]
[AutoData]
public void TestAddition4(int input1, int input2, string validationError)
{
    //arrange
    var expectedResult = input1 + input2;

    //act
    var actual = Add(input1, input2, validationError);
    //assert
    Assert.Equal(expectedResult, actual);
}
```
Nice! We can clearly see what's being tested here.

## Using AutoFixture for Automatic Mocking

I've found that the real gem in AutoFixture is it's ability to help automatically mock dependencies for testing classes. Take the example of `MyService`.

```
public class MyService
{
    private readonly IConfiguration _configuration;
    private readonly IFoo _foo;

    public MyService(IConfiguration configuration, IFoo foo)
    {
        _configuration = configuration;
        _foo = foo;
    }

    //checks if our configuration is correctly setup
    public bool CheckValidationConfiguration()
    {
        var configValue =
            _configuration["ConfigElement"] ?? throw new Exception("Config element not specified!");
        
        return int.TryParse(configValue, out _);;
    }

    //used to represent a second method which requires a dependency on a different interface
    public void TestMethod2()
    {
        _foo.Bar();
    }
}
```

It takes in two dependencies `IConfiguration` and `IFoo` where `IFoo` is a dummy interface defined as

```
public interface IFoo
{
    public void Bar();
}
```

The class `MyService` has two example methods, one to check if the provided is correct and a second method `TestMethod2` which uses `IFoo` to do something else. The `TestMethod2` isn't important to this example other than inducing an extra dependency. Suppose we wanted to test the `CheckValidConfiguration` method. Without AutoFixture it may look something like this.

```
[Fact(DisplayName = "Test Do Something without AutoFixture")]
public void Baseline()
{
    //arrange
    var configurationMock = new Mock<IConfiguration>();
    var fooMock = new Mock<IFoo>();
    configurationMock
        .Setup(x => x[It.IsAny<string>()])
        .Returns("1");
    var service = new MyService(configurationMock.Object, fooMock.Object);

    //act
    var validationConfiguration = service.CheckValidationConfiguration();

    //assert
    Assert.True(validationConfiguration);
}
```
Notice how `IFoo` is boilerplate just to get the code to work and test our method. Our arrange step can get really messy if our constructor has several dependencies. In addition doing tests this way makes our tests very brittle to changes in code. For example, if `MyService` needs a new dependency, there will be no fun had refactoring our tests. 

AutoFixture can help here by automatically mocking out the interfaces on our behalf.

```
[Fact(DisplayName = "Test Do Something with Manually Created Fixture")]
public void ManualAutoMoq()
{
    //arrange
    var fixture = new Fixture().Customize(new AutoMoqCustomization());
    var configurationMock = fixture.Freeze<Mock<IConfiguration>>();
    configurationMock
        .Setup(x => x[It.IsAny<string>()])
        .Returns("1");
    var sut = fixture.Create<MyService>();
    
    //act
    var validationConfiguration = sut.CheckValidationConfiguration();

    Assert.True(validationConfiguration);
}
```

Notice how the freeze method is called - what this does (to my understanding) is forces AutoFixture to return the same object every time the mock is used (as opposed to randomly generating this object 'on the fly'). Notice there isn't any mention of `IFoo` - this is great because we didn't need it to test anyway.

We can level up this test even more by removing boilerplate code required to generate the fixture each time we do a test. This will require us to define a custom attribute, but this is a once off thing. The code is given below.

```
using AutoFixture;
using AutoFixture.AutoMoq;
using AutoFixture.Xunit2;

namespace TestAutoFixture.Tests;

public class AutoMoqDataAttribute : AutoDataAttribute
{
    public AutoMoqDataAttribute()
        : base(() => new Fixture().Customize(new AutoMoqCustomization()))
    {
    }
}
```
and finally the test

```
[Theory]
[AutoMoqData]
public void AutoMoqTest(
    [Frozen] Mock<IConfiguration> configurationMock,
    MyService service)
{
    //arrange
    configurationMock
        .Setup(x => x[It.IsAny<string>()])
        .Returns("1");
    
    //act
    var validationConfiguration = service.CheckValidationConfiguration();
    //assert
    Assert.True(validationConfiguration);
}
```

To me this is amazing code - super lean and you really just focus on what's being tested.

<iframe src="https://giphy.com/embed/26ufdipQqU2lhNA4g" width="480" height="480" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>

## Wrapping up
All code from today's article can be found on my [Github repo](https://github.com/thatstatsguy/UnitTesting/tree/main/Testing%20with%20AutoFixture).

Something to note about the last test is the order in which logic is added. If you do anything in your constructor with the inputs, you may find that the logic for your mocks is only implemented after the constructor is called. I made this mistake in this [question on the AutoFixture community](https://github.com/AutoFixture/AutoFixture/discussions/1389) - learn from my mistakes😂

Until next time :) 

## Further reading
- [AutoFixture quick start guide](https://autofixture.github.io/docs/quick-start/#)