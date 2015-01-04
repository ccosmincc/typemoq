TypeMoq ![build badge](https://travis-ci.org/florinn/typemoq.svg?branch=master)
===================

Simple mocking library for JavaScript targeting [TypeScript](http://www.typescriptlang.org/) development. If you have used before a library like [Moq](https://github.com/Moq/moq4) then the syntax should look familiar, otherwise the examples below should hopefully provide enough information to get you started quickly.

----------


Features
-------------
* Strongly typed
* Auto complete/intellisense support
* Control over mock behavior
* Mock both classes (with arguments) and objects
* Auto sandboxing for global classes and objects


Installing
-------------
```
npm install typemoq
```


Usage
-------------

### Create mocks

Mocks can be created either from class types and constructor arguments or from existing objects, including function objects.

###### Using class types and constructor arguments

```typescript
// Using class as constructor parameter
var mock: Mock<Bar> = Mock.ofType(Bar);

// Using interface as type variable and class as constructor parameter
var mock: Mock<IBar> = Mock.ofType<IBar>(Bar);

// Using class as constructor parameter and args
var bar = new Bar();
var mock: Mock<Foo> = Mock.ofType(Foo, MockBehavior.Loose, bar);

// Using a generic class as constructor parameter and args
var mock: Mock<GenericFoo<Bar>> = Mock.ofType(GenericFoo, MockBehavior.Loose, Bar);
```

>**Note:** `MockBehavior` is used to specify how the mock should act when expectations are not defined, more details in the [Control mock behavior](#mock_behavior) section

###### Using existing objects, including function objects

```typescript
// From an existing object
var bar = new Bar();
var mock: Mock<Bar> = Mock.ofInstance(bar);

// Or from function objects
var mock1: Mock<() => string> = Mock.ofInstance(someFunc);
var mock2: Mock<(a: any, b: any, c: any)=>string> = Mock.ofInstance(someFuncWithArgs);
```

----------

**Note:** Mocks (created in any of the ways listed above) expose the actual mock object through the `.object` property (that has the same type as the class or object being mocked).


### Setup mocks

Mocks allow to match functions, methods and properties and setup return callbacks or exceptions to throw.

###### Matching functions

```typescript
// Match a no args function
var mock: Mock<() => string> = Mock.ofInstance(someFunc);
mock.setup(x => x()).returns(() => "At vero eos et accusamus");

// Match a function with args
var mock: Mock<(a: any, b: any, c: any) => string> = Mock.ofInstance(someFuncWithArgs);
mock.setup(x => x(It.isAny(), It.isAny(), It.isAny())).returns(() => "At vero eos et accusamus");
```

###### Matching methods

```typescript
var mock = Mock.ofType(Doer);

// Match a no args method
mock.setup(x => x.doNumber());

// Match a method with explicit number value params
mock.setup(x => x.doNumber(It.isValue(321)));

// Match a method with implicit number value params
mock.setup(x => x.doNumber(321));

// Match a method with explicit string value params
mock.setup(x => x.doString(It.isValue("abc")));

// Match a method with implicit string value params
mock.setup(x => x.doString("abc"));

// Match a method with object value params
var bar = new Bar();
mock.setup(x => x.doObject(It.isAnyObject(Bar)));

// Match a method with any string params
mock.setup(x => x.doString(It.isAnyString()));

// Match a method with any number params
mock.setup(x => x.doNumber(It.isAnyNumber()));

// Match a method with any interface/class params
var bar1 = new Bar();
var bar2 = new Bar();
mock.setup(x => x.doBar(It.isAnyObject(Bar)));
```

###### Matching properties

```typescript
// match a property getter
var mock = Mock.ofType(FooWithPublicGetterAndSetter);
mock.setup(x => x.foo);
```

###### Attaching return callbacks

The callback attached to `.returns` has the same signature as the matching function/method.
Also the callback gets called with the arguments passed to the matching function/method and it must have the same return type, making possible the following:

```typescript
mock.setup(x => x.doString("abc")).returns((s: string) => s.toUpperCase());
```

###### Attaching exceptions to throw

```typescript
mock.setup(...).throws(new CustomException());
```


### Attach callbacks

Attached callbacks are called before the `.returns` callback or `.throws` get called, and they have similar signature and behavior to `.returns` callbacks.

```typescript
var mock = Mock.ofType(Doer);
var called1, called2 = false;
var numberArg: number;

mock.setup(x => x.doString(It.isAnyString())).callback(() => called1 = true).returns(s => s.toUpperCase());
mock.setup(x => x.doNumber(It.isAnyNumber())).callback(n => { numberArg = n; called2 = true; }).returns(n => n + 1);
```


###<a name="mock_behavior"></a>Control mock behavior

###### Using MockBehavior

When creating a mock you may specify a behavior value such as:

* `MockBehavior.Loose` (default) - never throws and returns default values
* `MockBehavior.Strict` - raises exceptions for anything that doesn't have a corresponding expectation

```typescript
var mock = Mock.ofType(Doer, MockBehavior.Strict);
```

###### Enable calling object being mocked

When mock property `callBase` is `true`, base class implementation gets invoked if no expectation overrides the member.
Default for `callBase` is `false`.

```typescript
mock.callBase = true;
```


### Verify expectations

Expectations can be verified either one by one or all at once by marking matchers as verifiable.

###### One by one

```typescript
// Verify that a no args function was called at least once
var mock: Mock<() => string> = Mock.ofInstance(someFunc);
mock.object();
mock.verify(x => x(), Times.atLeastOnce());

// Verify that a function with args was called at least once
var mock: Mock<(a: any, b: any, c: any) => string> = Mock.ofInstance(someFuncWithArgs);
mock.object(1, 2, 3);
mock.verify(x => x(It.isAnyNumber(), It.isAnyNumber(), It.isAnyNumber()), Times.atLeastOnce());

// Verify that no args method was called at least once
var mock = Mock.ofType(Doer);
mock.object.doVoid();
mock.verify(x => x.doVoid(), Times.atLeastOnce());

// Verify that method with params was called at least once
var mock = Mock.ofType(Doer);
mock.object.doString("Lorem ipsum dolor sit amet");
mock.verify(x => x.doString(It.isValue("Lorem ipsum dolor sit amet")), Times.atLeastOnce());

// Verify that value getter was called at least once
var mock = Mock.ofType(Bar);
mock.object.value;
mock.verify(x => x.value, Times.atLeastOnce());

// Verify that value setter was called at least once
var mock = Mock.ofType(Bar);
mock.object.value = "Lorem ipsum dolor sit amet";
mock.verify(x => x.value = It.isValue("Lorem ipsum dolor sit amet"), Times.atLeastOnce());
```

Varius expectation could be specified by using `Times` constructor methods.

**Note:** When constructing a mock it is allowed to pass mock objects as arguments and later verify expectations on them. E.g.: 

```typescript
var mockBar = Mock.ofType(Bar);
var mockFoo = Mock.ofType(Foo, MockBehavior.Loose, mockBar.object);
mockFoo.callBase = true;

mockFoo.object.setBar("Lorem ipsum dolor sit amet");

mockBar.verify(x => x.value = It.isValue("Lorem ipsum dolor sit amet"), Times.atLeastOnce());
```

###### All at once

```typescript
var mock = Mock.ofType(Doer);

mock.setup(x => x.doNumber(999)).verifiable();
mock.setup(x => x.doString(It.isAny())).verifiable();
mock.setup(x => x.doVoid()).verifiable();

mock.object.doVoid();
mock.object.doString("Lorem ipsum dolor sit amet");
mock.object.doNumber(999);

mock.verifyAll();
```


### Create global mocks

Global mocks are created by specifying a class type or an existing object, similar to regular mocks, and a container object of the type/object being mocked. 

###### Using class types

```typescript
// Create an instance using class as ctor parameter
var mock: GlobalMock<GlobalBar> = GlobalMock.ofType(GlobalBar);

// Create an instance using interface as type variable and class as ctor parameter
var mock: GlobalMock<IGlobalBar> = GlobalMock.ofType<IGlobalBar>(GlobalBar);
```

###### Using existing objects, including function objects

```typescript
// Create an instance using class as ctor parameter and ctor args
var bar = new Bar();
var foo = new Foo(bar);
var mock: GlobalMock<Foo> = GlobalMock.ofInstance(foo);

// Create an instance using a generic class as ctor parameter and ctor args
var foo = new GenericFoo(Bar);
var mock: GlobalMock<GenericFoo<Bar>> = GlobalMock.ofInstance(foo);

// Create an instance from an existing object
var bar = new GlobalBar();
var mock: GlobalMock<GlobalBar> = GlobalMock.ofInstance(bar);

// Create an instance from a function object
var mock1: GlobalMock<() => string> = GlobalMock.ofInstance(someGlobalFunc);
var mock2: GlobalMock<(a: any, b: any, c: any) => string> = GlobalMock.ofInstance(someGlobalFuncWithArgs);
```

**Note:**

- Default `container` is considered to be the `window` object
- Due to browser security limitations, global mocks created by specifying class type cannot have constructor arguments


### Auto sandbox global mocks

Replacing and restoring global class types and objects is done automagically by combining global mocks with mock scopes.

```typescript
// Global no args function is auto sandboxed
var mock = GlobalMock.ofInstance(someGlobalFunc);
Scope.using(mock).with(() => {
    someGlobalFunc();
    someGlobalFunc();
});

// Global function with args is auto sandboxed
var mock = GlobalMock.ofInstance(someGlobalFuncWithArgs);
Scope.using(mock).with(() => {
    someGlobalFuncWithArgs(1,2,3);
    someGlobalFuncWithArgs("1","2","3");
    someGlobalFuncWithArgs(1, 2, 3);
);

// Global object is auto sandboxed
var mock = GlobalMock.ofType(GlobalBar);
Scope.using(mock).with(() => {
    var bar1 = new GlobalBar();
    bar1.value;
    bar1.value;
});
```

**Note:** Within a mock scope when constructing objects from global functions/class types being replaced by mocks, the constructor always returns the mocked object of the corresponding type passed in as argument to the `using` function