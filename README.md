# BLAST
> Language specification.
## **Please give me a ⭐ if you think my work is worthy**

**BLAST is a left-typed functional language**.

1. No imports
2. Less brackets
3. Less code
4. Readable code
5. More efficiency

# Table of Contents
0. [Hello World in BLAST](#hello-world-in-blast)
1. [Creating primitives](#1-creating-primitives)
2. [Creating objects](#2-creating-objects)
3. [Local Enums](#3-local-enums)
4. [Defining methods](#4-defining-methods)
    1. [Default arguments](#4-1-default-arguments)
    2. [Calling methods](#4-2-calling-methods)
    3. [Cascade notation](#4-3-cascade-notation)
    4. [Async methods](#4-4-async-methods)
    5. [Varargs](#4-5-Varargs)
5. [Optional unlocking](#5-optional-unlocking)
6. [Pipe operator](#6-pipe-operator)
7. [Callbacks](#7-callbacks)
8. [Defining classes](#8-defining-classes)
    1. [Simple classes](#8-1-simple-classes)
    2. [Instantiation](#8-2-instantiation)
    3. [Abstract classes](#8-3-abstract-classes)
    4. [Interface classes](#8-4-interface-classes)
    5. [Enum classes](#8-5-enum-classes)
    6. [Model classes](#8-6-model-classes)
    7. [Adapter classes](#8-7-adapter-classes)
9. [Type casting](#9-type-casting)
10. [Control structures](#10-control-structures)
    1. [If statement](#10-1-if-statement)
11. [Loops](#11-loops)
    1. [For loop](#11-1-for-loop)
12. [Generators](#12-generators)
13. [Namespaces](#12-namespaces)
    1. [Flat-namespaces](#13-1-flat-namespaces) 
14. [Java interoperability](#14-java-interoperability)
15. [Exception handling](#15-exception-handling)
    1.[Exception ignoring](#15-1-exception-ignoring)
16. [Object Desctruction](#16-object-destruction)
    1. [Object destruction as statement](#16-1-object-destruction-as-statement)
    2. [Object destruction as expression](#16-2-object-destruction-as-expression)
17. [Events](#17-Events)
    1. [Registering events](#17-1-registering_events)
    2. [Firing events](#17-2-firing-events)
18. [Play](#18-play)

# Hello World in BLAST

```scala
Console.log@ "Hello World";
// or
Console.log("Hello World");
// or
"Hello World" | Console.log@;
// or
"Hello World" | Console.log();
```

# 1. Creating primitives
```java
int a = 3;
int[] b = [1, 2, 3];
int c = {
    int b = 4;
    return 3 + b;
}; // resources are closed and memory freed after the block initialization
any[] d = ["a", 1, false];

int itsFour = if (
  a == 3 ? 4
) // 4

int itsZero = a match (
   /^abc/ ? 4
   5 ? 6
) // defaults to 0

boolean itsFalse = a match /^abc/ // false
```

## 1.1 Block initialization

Variables can be initialized with the block `type varName ::= {}` initializer which means everything declared inside this block will be freed outside.
An efficient way to create heavy objects on-the-fly.

```scala
int c = {
    int b = 4;
    return 3 + b;
}; // resources are closed and memory freed after the block initialization
```

# 2. Creating objects
```scala
ArgumentOption option = {};

ArgumentOption[] options = [
     {
        option: 'i',
        longOption: 'input',
        hasArg: true,
        description: 'input'
     }
];
```

## 2.1 Constructor parameter unlocking

BLAST knows exactly what you want. Therefore the compiler can infer the nested constructor types 

### Example
Lets assume we have an `A(B, C)` class with `B(String param)`, `C(D param)` and `D(Enum param1, Integer param2, Boolean param3)`.
We would normally instantiate this object by: `new A(new B("Hello World"), new C(new D(Enum.Example, 0, false)))`
BLAST gives us a simpler solution which is the below:

```scala
A object = { "Hello World", { Enum.example, 0, false } }
```

The unlocking option above only works if there is no `(String, Enum, D param)` constructor defined which is ambigious. In that case, compile error thrown.

### Single parameter example

If the class has only 1 parameter in its constructor then the brackets are negligible.

```scala
// constructor: A(String example)
A object = "Hello World"

// equivalent to: A object = { "Hello World" }
// and equivalent to: A object = { example: "Hello World" }
```

### No arg example

Without args the brackets are mandatory.

```scala
// constructor: A()
A object = {}
```


### Polymorphism

```scala
Interface object = <ActualType> "Hello World";
```

# 3. Local Enums

So simple and easy

```java
Enum localEnum = enum(String name, int number, boolean condition) {
   X("X", 1, false),
   A("A", 1, false),
   B("B", 1, false),
   C("C", 1, false);
   
   void someMethod() {
   	// do something
   }
}

String name = localEnum.X.name
localEnum.X.someMethod@
```

# 4. Defining methods
```csharp
int syncMethod() {
   return 3;
}

// call it
syncMethod@; // or sychMethod(); also accepted

int inline(int a, int b): a + b;

// call it
inline(1, 2);

Callback<int> inlineCallback(int a, int b): -> a + b;

// Notice we call inlineCallback first with parameters 1, 2 then we call the callback which is returned from inline.
// This is the same as: inlineCallback(1, 2)();
inlineCallback(1, 2)@;

async int asyncMethod() {
   return 3;
}

// call it
asyncMethod@;
// or await it
await asyncMethod@;
```

## 4.1. Default arguments
Default arguments must be the last in the argument list.

```java
void method(int a, int b = 0, boolean b = false) {
    
}

method@ 0;
```

**!!!Pipe parameters cannot have default values**

## 4.2. Calling methods
There are 3 ways of calling methods in BLAST.
1. Using the `@` operator: method@;
2. Using the regular `(args)` operator: method(args);
3. Using the `|` (pipe) operator: args | method;

**Why is there 3 different ways?**
Its simply to be able to write more readable code. 
Imagine the following example. Lets define a method called `sum` which adds two numbers.
You want to add the numbers: 1, 2, 3, 4

**In regular way you would call this method like this**
```scala
sum(1, sum(2, sum(3, 4))) // = 10
```
**However in BLAST you can resolve these nested calls with the following**
```scala
1 | sum@ 2 | sum@ 3 | sum@ 4 // = 10
```
Much readable, ey?

**Okay okay, but what about the `()` operator? Why is it needed then?**
So the `@` operator is for simplification in cases where you can use it, sometimes it can be confusing for the compiler.

**Example**
```scala
methodThatReturnsComplexObject@ 1, 2.complexObjectParam;
```
The above we attempt to reach the `complexObjectParam` parameter which is on the object returned by the `methodThatReturnsComplexObject`

**How does it looks like to the compiler? (Apart from the compile error, lets assume there is no error at all)**
The compiler thinks we want the member of `2` which clearly not what we wanted:
methodThatReturnsComplexObject@ 1, `2.complexObjectParam`;

**This problem is fixed with the `()` operator:**
```scala
methodThatReturnsComplexObject(1, 2).complexObjectParam;
```
I mean.. it's up to you.

## 4.3. Cascde notation

The `cascade notation operator` helps overcome the above issue.

To call multiple methods with `@` operator:

```scala
Object.method@ arg
      ..method2@ arg
      ..method3@ arg
```

## 4.4. Async methods
Async methods declared using the `async` keyword or setting the return value to `Future`

```csharp
async int asyncMethod() {
    return 3;
}
```

There are multiple ways to handle these kind of methods.

### Awaiting
Unlike in c# or javascript the await isn't limited to the scope of async methods. We can await the result at any time.
The only restriction that the awaitable object's or method's type has to be `Future` or any subtype of `Future`.

**We can wait the result using the `await` keyword**
```csharp
int result = await asyncMethod@;
```
**Or do something with the resulting Future**
```csharp
Future future = asyncMethod@;
future.supplyAsync(...)
// ... do anything else
await future; 

// Create a future object on-the-fly and await it
// yes, as the above says we can await future objects as well
Future anyFuture = Future.of(someHeavyMethod@);
// ... bunch of other code
await anyFuture;
```

### Extra example

Thanks to the `pipe operator` we can chain methods more easily with BLAST functional way.

**Instead of this**
```scala
Future awaitable = asyncMethod@
		.thenApply(number -> number * number)
		.thenApply(square -> square * 2)
		.whenComplete(..);
```

**Write this**
```scala
Future awaitable = asyncMethod@
        | apply@ number -> number * number 
        | apply@ square -> square * 2
        | onComplete@ ...;
```

**Then await**
```csharp
await awaitable;
```

# 4.5. Varargs

BLAST, Just like in Java can handle varargs. See the example below:
```scala
int varargMethod(int a, int b, int... c) {}
varargMethod@ 1, 2, 3, 4, 'c', "s"
```

# 5. Optional unlocking

In BLAST **null value is eliminated**. Welcome the world of optionals.
The keyword `empty` replaces `null` in a way that we can define empty values (**but empty != null**).

```java
Optional optionalEmpty = empty; // Optional.empty()
Optional<char> optionalChar = "c"; // Optional.of('c')
Optional<String> optionalString = "string"; // Optional.of("string")
Optional<String> optionalOptional = optionalString; // Optional.of("string")
Optional<int> optionalInt = 3; // Optional.of(3)

// You can't resolve empty into other values
int emptyNotAllowed = optionalEmpty // Runtime error ❌

int optionalResolved = optionalInt; // optionalInt.ElseThrow(runtime error);
int optionalResolvedSafely = optionalInt??; // optionalInt.orElse(0);
```

You may have noticed in the last line that usually the objects do not have a default value.
In BLAST its not true, every object has a default value which is the zero arg constructor and this default value can be overridden.

This might be interesting as well: [Java interopability](#java-interoperability)

## Be careful

Or don't, because BLAST takes care of you. If you want to pass empty back from a method, you have to set it's return value to Optional

```java
int method() {
    return empty
} // Not allowed ❌

Optional method() {
    return empty
} // Allowed ✔️
```

## Okay, be careful though

Optional without template parameters stands for `Optional<any>` which means you can assign different types to it.

```java
Optional method() {
    return if (
        a == 3 ? 0;
        true ? empty; // this branch is negligible because Optional's default value is empty, but we put here for clarification
    )
}

int a = method@ // What happens?
```

In the code above if `a == 3` is true the program continues to run without any problem.
What if isn't true? Well, runtime error happens, duh.

### How to overcome the issue above?

You have two options. 
- One is to check with if statement if the optional is empty.
- Second is to use the `nullish coalescing operator (??)`

```java
// First option
Optional out = method@
int a = if out ? out : 0;
// Second option
int a = out ??; // Here if out is empty the default value is passed. If we want a different value, we have to write: 'out ?? otherValue;'
```

## 5.1. Optional unlocking with complex objects

So now you have learned how you can use optionals with primitives. What about objects?

```scala
ComplexClass complexObject = {};
Optional<ComplexClass> complexObjectOptional = complexObject;
```

### Call method on complexObject

To use the code above and call the method without extracting the optional first lets use the `!` operator.

```scala
ComplexClass complexObject = {};
Optional<ComplexClass> complexObjectOptional = complexObject;

complexObjectOptional!.someMethod@; // use this if you are 100% sure that complexObjectOptional never empty
```

### What if complexObjectOptional is empty?

**Runtime error thrown.**
To overcome this issue use the `nullish coalescing operator (??)`

```scala
ComplexClass complexObject = {};
Optional<ComplexClass> complexObjectOptional = complexObject;

complexObjectOptional??.someMethod@; // which will use the default value of that class
// use this if the default value is good enough for you, otherwise you can use the below
(complexObjectOptional ?? otherValue).someMethod@;
```

# 6. Pipe operator
```csharp
int method(int a..., int b...) pipe(string c) {
  Console.log@ c;
  return 0;
}

// call it
"hello" | method@ 1, 2
```

If you define `pipe` on method you ***must*** call the parameters inside with the `pipe operator` however if you don't define it,
you have a choice to use the `pipe operator` on the normal method parameters.

***example***

```csharp
int method(int a..., int b...) {
  Console.log@ a + b;
  return 0;
}

// call it
1, 2 | method@
```

***what about async methods?***

```csharp
async int method(int a..., int b...) {
  Console.log@ a + b;
  return 0;
}

// fire it ;)
await 1, 2 | method@
```

# 7. Callbacks

***Callbacks doesn't support pipe operator yet.***

```scala
Callback asyncCallback = async int a, int b, int c -> "Hello World"

// call it
asyncCallback@
// or await it
await asyncCallback@
```

# 8. Defining classes
Classes in BLAST determined by file naming postfixes and prefixes. A simple class doesn't include any prebuilt postfix or prefix.
There are other class types like:
* Non-dependency injection:
    - Abstract eg AbstractXY.bl
    - Interfaces: XYInterface.bl
    - Enums: XYEnum.bl
    - Model classes: XYModel.bl
    - DTO classes: XYDTO.bl
    - Adapter classes: XYAdapter.bl
    - Factory classes: XYFactory.bl
* Dependency injection:
    - Components: XYComponent.bl
    - Services: XYService.bl
    - Repositories: XYRepository.bl

Of course some can be combined for example:
- XYModelAdapter
- XYModelFactory
- XYModelAdapterFactory
- XYFactoryFactory
- XYComponentAdapterFactoryFactory
- etc.

## Simple classes

```scala
// file: A.bl
// every class starts with a header definition
namespace A.b.c; // usually in blast you don't write imports because the compiler smart enough.
// However sometimes you get some module name collisions from different packages,
// there you use the namespace definition to resolve ambiguity. For details check the namespaces section

extends Console // to extend other classes
implements ConsoleInterface // to implement interfaces
use virtual // every class static by default, use virtual to make it non-static
use <K> // to give this class a template parameter
use @RequiredArgsConstructor, @NoArgsConstructor // to add annotations

// program starts here..
```
The above definition in a file called A.bl will result in A.class so in blast you don't have to write the class definition in the file. ***Naming matters***.

## 8.1. Instantiation

```csharp
SomeClass instance1 = {};
SomeClass instance1 = {
    prop1: value1,
    prop2: value2,
    ...
    propN: valueN
};
```

## 8.2. Abstract classes
Just like you used to in other languages. :) 

```scala
project
│   src
└───  AbstractAbc.bl
```

## 8.3. Interface classes
Just like you used to in other languages. :) 

```scala
project
│   src
└───  AbcInterface.bl
```

## 8.4. Enum classes

Enum classes are allowing methods, parameter declarations and enum definitions only.

```scala
project
│   src
└───  AbcEnum.bl

String name;
int number;
boolean condition;

X("X", 1, false),
A("A", 1, false),
B("B", 1, false),
C("C", 1, false)
```

## 8.5. Model classes

Model classes are describing the properties of some entity and nothing else.
Contains member declarations only. ***Functions not allowed***.

```scala
project
│   src
└───  AbcModel.bl

String name = "Hello World"; // default value
int number;
boolean condition;
```

## 8.6. DTO classes

The purpose of these classes is to transfer data (usually over HTTP).
The only difference between DTO and Model is that **DTO serializable by default**.

```scala
project
│   src
└───  AbcDTO.bl

String name = "Hello World"; // default value
int number;
boolean condition;
```

## 8.7. Adapter classes

Adapters used to add functionality to model classes.
***Adapters are virtual by default***

```scala
project
│   src
└───  AbcModelAdapter.bl

int someFunctionality() {...}
```

# 9. Type casting

BLAST is smart enough most of the time. However sometimes type casting may be useful.

### Type cast example
```scala
A example = <A> methodThatReturnsB@;

A example2 = <A> <C> methodThatReturnsB@.memberSubtypeOfA;
// Here the first (<C>) cast affects the 'methodThatReturnsB@' call while the second (<A>) takes effect on 'memberSubtypeOfA'
```

## When type casting isn't needed

One of the benefits of this left-typed language is that it can easily infer types.

#### Example: passing List of values to a supertype ArrayList

```scala
List<ComplexObjectInterface> a = [o1, o2, o3,...];
ArrayList<ComplexObject> b = a; // BLAST can handle it
```
## When type casting is needed

If you don't pass the value to other type but you want to use the variable on-the-fly.

#### Example: use the variable on-the-fly as a supertype

```scala
ComplexObjectInterface complexObject = <ComplexObject> {};

// Now you want to call someAction which only available in ComplexObject
<ComplexObject>complexObject.someAction@;
```

# 10. Control structures

Control structures are a bit different than in other languages to make the code more readable and simple.

## 10.1. if statement

The structure of an if statement is the following: 

```csharp
if (
    condition1 ? action;
    condition2 ? action;
    ...
    conditionN ? action;
    true ? action; // this is the else branch
)
```

**The action's return value must be compatible with the variable's or method's return value if used that way, otherwise compiler error thrown**

```csharp
// using as value of a variable
int var = if (
    otherVar == "Hello World" => 1
); // if the condition is false the default value returned is 0

// one line if
int var2 = if var == 1 ? 0 : 2;

// Of course you can define if without returning the value to something, this is the void if
if (
  var == 2 ? someAction(); // where someAction can return anything the value dismissed from stack  
);

// using in method
int method() {
    return if (
        otherVar == "Hello World" ? 1;
        var == 1 ? 2;
        true ? 36;
    ); // using true to specify default value instead of 0
}

// using in a mapping function
intList | map@ elem -> if (
    elem == 0 ? elem*2;
    true ? elem^2;
); // this is where it comes very handy
```
# 11. Loops
## 11.1. For loop
**Syntax:** `for([(expr | variable) (-> | <-)] expr [: expr]) {}?`

### Loop as statement
```scala
// Simplified version
for(int i = 0 -> 10) {

}

// Simplified version different increment
for(int i = 0 -> 10 : 2) {
    
}

// Simplified version for characters
for(char c = "a" -> "z") {
   // j will be the characters from "a" to "z"
}

```

### Loop as expression

```scala
for(ComplexObject o : complexObjectList) {...}
ComplexObject[] shallowCopyList = for(complexObjectList);

// Splitting string into char array, the for loop way
char[] chars = for("Hello World");

// Create an array from an int (from 0 -> 100)
int[] integers = for(100);

// Create an array from an int (from 0 -> 100) with increment of 2
int[] integers = for(100 : 2);

// Create an array from an int (from 10 -> 100) with increment of 2
int[] integers = for(10 -> 100 : 2);

// To define body
int[] integers = for(int i = 10 -> 100 : 2) {
    yield i + 2;
}
```

### Reversed loops

The reversed loops are using the reversed arrow `<-` syntax.

```scala
// int list from 0 to -10
int[] ints7_reverse = for(0 <- -10 : 2)
```

Reversed loops prohibited from non-numerical use.

### Indexed loops

For loops can be used to iterate lists with index as well.

```java
// indexed string loop
for(int i -> "Hello World") {
    Console.log@ i
}

// indexed string loop starting from 2nd index, get every 2nd character's position
for(int i = 2 -> "Hello World" : 2) {
    Console.log@ i
}
```

### Expressional for loops can be used for mappings as well

**Example**
```scala
String[] stringArray = ["Adam", "Armin", "Joseph", "Kate", "Rose"];
String[] mapped = for(String name = stringArray): "Hello $name";
// ["Hello Adam", "Hello Armin", "Hello Joseph", "Hello Kate", "Hello Rose"]
```

**Of course you can use it with filter**

```scala
String[] stringArray = ["Adam", "Armin", "Joseph", "Kate", "Rose"];
String[] mapped = for(String name = stringArray): "Hello $name" | filter@ greeting -> greeting == "Hello Kate";

// Or to filter first
String[] mapped = for(stringArray | filter@ name  -> name  == "Kate"): "Hello $name";

// Which obviously less readable than using the map method, but this is just an example :)
// Use this instead
String[] mapped = stringArray | filter@ name -> name == "Kate" | map@ name -> "Hello $name";
```

## 11.2. Sequences
For loops when used as expression can be used to create sequences easily.

```scala
int[] int_sequence = for(int i = 2 -> 10 : 2) {
  yield i + 1
  yield i
} // [3, 2, 5, 4, 7, 6, 9, 8, 11, 10]
```

# 12. Generators

For memory efficiency sometimes we don't want to store every value in memory at a time. Multiple generators exists:
* IndexedGenerator
* ElementWiseGenerator
* ListGenerator
* InfiniteGenerator

**We can define generators by using the same syntax like for-loops.**

**Syntax:** `for*(<Loop declaration>) {}?`

## Indexed generator example
This example will generate numbers from 0 to 100.
```scala
Generator<int> intGeneratorMethod() {
    return for*(100)
}
```
### Define a body
We can of course define a more complex generator like the following:
```scala
Generator<Something> somethingGeneratorMethod() {
    return for*(int i = 2 -> 100 : 2) {
      Something something = i // if you  find this interesting check the instantiation part
      yield something
    }
}
```

## Infinite generator example
The other interesting generator we should know is the infite generator,
which, as its name implies, will generate an infinite sequence.
To create one simply create a condition in for loop that never ends.

### Example
```csharp
Generator<int> infiniteNumberGenerator () {
    int number = 0
    return for*(true) {
        yield number++
    }
}
```

## Iterate over a Generator
We can use `enhanced for-loops` for this purpose

```csharp
Generator<int> intGenerator = for*(100)
for (int i -> intGenerator) {
    // do smth
}
```

## Save iterator into variable

Saving iterators output into a variable is a bit tricky because not every iterator's length known exactly.
However with IndexedGenerator it is still easy, since its iterator's the length is known.

```scala
Generator<int> intGenerator = for*(100)
int[] ints = for(intGenerator)
```

The hard part comes when we want to iterate over something which's length not known (e.g. InfiniteGenerator). In that case `compile error` thrown. 

# 13. Namespaces
In BLAST we don't write import statements as the compiler smart enough to know which dependency you want to use.
However, sometimes, just like in other languages, there are name conflicts. That is what `namespace` keyword is used for.
Namespace differs from import in that it specifies a `space` for the search instead of the `module` to be imported.

### Example
Let's say we have a `Console` module in `somepackage` and `somepackage2` and we want to use the one lays in `somepackage`.

**First option**
```scala
somepackage.Console.log@ "hello";
```

**Second option**
```csharp
namespace somepackage;

Console.log@ "hello";
```

## 13.1. Flat namespaces
Sometimes you don't want to write out the module's name everytime you call a function from it. eg. `Module.function@ input`
In this case, flat namespaces help.

```csharp
flat namespace somepackage.Console;

log@ "hello";

// But we can still call via the normal way as well.
Console.log@ "hello" 
```

# 14. Java interoperability

In BLAST there is no way to define null value. However, in vanilla java it still exists, in fact its a problem which should be handled by the language.

This problem is solved by the optional unlocking.
Optional unlocking not only handling `optionals` but `nulls` as well.

However, nulls are not resolved automatically, as are optionals, so if `null` is not handled by the unlock operator, a runtime error will still occur.

### Example

**Third party java code**
```java
package org.example.java

class NullObjectClass {
    public static Object getNullObject() {
        retrun null;
    }
}
```

**BLAST code**
```scala
Object nullObject = NullObjectClass.getNullObject@;
nullObject.someMethod@ ❌ // Runtime error

nullObject??.someMethod@ ✔️ // No problem
```

**The conclusion is that developer must take care when using third party java libraries**

# 15. Exception handling

The exception handling much cleaner and simpler than in other languages. There is no try-catch, only ***handle(..) {}***

```scala
methodThatThrowsCustomException();
methodThatThrowsCustomException2();

handle(CustomException | CustomException2 e) {
    // do ex handling here..
}
```

**Or we can do it separately:**

```scala
handle(CustomException e) {
    // do ex handling here..
}

handle(CustomException2 e) {
    // do ex handling here..
}
```

The idea is that handle(..) catches the actual scope's thrown exceptions. With this approach, you won't be littering your code all over the place with try-catch.
**handle** could always go to the end of each method.

## 15.1. Exception ignoring

Sometimes we don't want to deal with exceptions. Check the next example how to use handle in those cases.

```java
int exceptionalCall() {
    throw <RuntimeException>{};
}

Optional<int> integer = handle(exceptionalCall@, ExceptionType1 | ExceptionType2 | ...);

// To give custom default value
Optional<int> integer = handle(exceptionalCall@, ExceptionType1 | ExceptionType2 | ...): 13;
```

In this example handle returns the called method's value, otherwise empty optional, if default value not specified.

# 16. Object destruction

Object destruction is a very handy way to extract parameters from instances.
**Syntax:** `instance(<var decl>, ...) [block]`

## 16.1. Object destruction as statement
In the next example we have a class called `Something somethingInstance = Something(int a, OtherObject b, String c)` and we want to get the `int` and `String` parameters only
```java
somethingInstance(int customIntVariableName, *, String customStringVariableName)

// use customIntVariableName and customStringVariableName here
```

**Use block to decrease the lifetime of variables**

```java
somethingInstance(int customIntVariableName, *, String customStringVariableName) {
    // use customIntVariableName and customStringVariableName here
}

// Cannot use customIntVariableName and customStringVariableName here
```

### Chained object destruction
We can define multiple object destruction expressions in the same statement each followed by comma
```java
somethingInstance(int a, OtherObject otherObject, String c), otherObject(Type d, Type e)  {
    // a, b, c, d, e available in this scope
}
```

Note above that we destruct the `otherObject` which is extracted from `somethingInstance`

## 16.2. Object destruction as expression
We will use the same object as example

```java
// Assign to variable
String out = somethingInstance(int customIntVariableName, *, String customStringVariableName): "${customIntVariableName}${customStringVariableName}"

// Use in method call
method@ somethingInstance(int customIntVariableName, *, String customStringVariableName): "${customIntVariableName}${customStringVariableName}"

// Use in pipe expression
somethingInstance(int customIntVariableName, *, String customStringVariableName): "${customIntVariableName}${customStringVariableName}" | method@

// etc..
```

## 17. Events
There is a built-in functionality to register and fire events with ease.
Events in blast are special exceptions that can be raised with the `fire` keyword.

### 17.1. Registering events
**Syntax**: `on((<event_type> <event_variable>)(',' <event_type> <event_variable>)*){<event_block>}` 
Here is an example how to register an event:

```scala
on(BlastEvent e) {
  Console.log@ "Event run {{e}}"
}
```
Or run the same logic with multiple events:
```scala
on(CustomBlastEvent1 | CustomBlastEvent2 e) {
    Console.log@ "Event run {{e}}"
}
```

### 17.2. Firing events
```java
fire BlastEvent:::{
    prop1: "event_property1",
    prop2: "event_property2",
    ...
}
```

# 18. Conditional play statement

Conditional play statement is an easy way to spawn a separate thread for a while.

**sytax:** `play {<statements>} while <condition>`

As the name suggests this statement will play its body in a separate thread until the condition is false.

```scala
int frame = 0
play {
    frame++
} until frame < 60
```

Play will fire a PlayFinishEvent after completion.
```scala
PlayFinishEvent animationFinishedEvent = play {
    Animation.playNextFrame@
} while Animation.hasFrame

on(animationFinishedEvent) {
    // do something with animation
}
```

It is possible to create infinite loops as well.
**syntax:** `play {<statements>}`
In this case no events getting spawned.
```scala
play {
   GameEngine.update@
}
```
It is possible to play something until an event happens
**syntax:** `play {<statements>} until <event> [| <event>]*`
```scala
play {
   GameEngine.update@
} until GameStoppedEvent
```

# TO BE CONTINUED..
