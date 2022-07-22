# BLAST

This is the public repository for overview of current progress.
## **Please give me a ⭐ if you think my work is worthy**

BLAST is a left typed language, which means **you only write types to the left side of each declaration and won't see types on the right side**.

1. No imports
2. Less brackets
3. Less code
4. Readable code
5. More efficiency

# Table of Contents
1. [Creating primitives](#creating-primitives)
2. [Creating objects](#creating-objects)
3. [Local Enums](#local-enums)
4. [Defining methods](#defining-methods)
5. [Optional resolution](#optional-resolution)
6. [Pipe operator](#pipe-operator)
7. [Callbacks](#callbacks)
8. [Defining classes](#defining-classes)
    1. [Simple classes](#simple-classes)
    2. [Instantiation](#instantiation)
    3. [Abstract classes](#abstract-classes)
    4. [Interface classes](#interface-classes)
    5. [Enum classes](#enum-classes)
    6. [Model classes](#model-classes)
    7. [Adapter classes](#adapter-classes)
9. [Continuity operator](#continuity-operator)
10. [Control structures](#control-structures)
    1. [If statement](#if-statement)
11. [Loops](#loops)
    1. [For loop](#for-loop)

# Creating primitives
```java
int a = 3;
int[] b = [1, 2, 3];
int c = {
    int b = 4;
    return 3 + b;
};
any[] d = ["a", 1, false];

int itsFour = if (
  a == 3 ? 4
) // 4

int itsZero = match (a) (
   /^abc/ ? 4
   5 ? 6
) // defaults to 0

boolean itsFalse = match(a) /^abc/ // false
```

# Creating objects
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

# Local Enums

So simple and easy

```java
Enum localEnum = enum(String name, int number, boolean condition) {
   X("X", 1, false),
   A("A", 1, false),
   B("B", 1, false),
   C("C", 1, false)
}

String name = localEnum.X.name
```

# Defining methods
```csharp
int syncMethod() {
   return 3;
}

// call it
syncMethod@

int inline(int a, int b) -> a + b;

// call it
inline@

async int asyncMethod() {
   return 3;
}

// call it
asyncMethod@
// or await it
await asyncMethod@

```

# Optional resolution

In BLAST **null value is eliminated**. Welcome the world of optionals.
The keyword `empty` replaces `null` in a way that we can define empty values (**but empty != null**).

```java
Optional optionalEmpty = empty; // Optional.empty()
Optional optionalChar = "b"; // Optional.of('b')
Optional optionalString = "asd"; // Optional.of("asd")
Optional optionalOptional = optionalString; // Optional.of("asd")
Optional optionalInt = 3; // Optional.of(3)

// You can't resolve empty into other values
int emptyNotAllowed = optionalEmpty // Compile error ❌

int optionalResolved = optionalInt; // optionalInt.getOrElse(0);
```

You may have noticed in the last line that usually the objects do not have a default value.
In BLAST its not true, every object has a default value which is the zero arg constructor and this default value can be overridden.

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

Optional without template parameters stands for Optional\<any\> which means you can assign different types to it.

```java
Optional method() {
    return if (
        a == 3 ? 0;
        true ? empty; // this branch wouldn't need because Optional's default value is empty, but we put here for clarification
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
int a = out ??; // Here if out is empty the default value is passed. If we want a different value, we have to write out ?? otherValue;
```

## Optional resolution with complex objects

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

What happen here is that BLAST tries to extract the optional value and call the method on the object.

### What if complexObjectOptional is empty?

**Compile error thrown.**
To overcome this issue use the `nullish coalescing operator (??)`

```scala
ComplexClass complexObject = {};
Optional<ComplexClass> complexObjectOptional = complexObject;

complexObjectOptional??.someMethod@; // which will use the default value of that class
// use this if the default value is good enough for you, otherwise you can use the below
complexObjectOptional ?? otherValue _. someMethod@;
```

In the last line we used the continuity `_.` operator. Check out how it works: [Continuity operator](#continuity-operator)


# Pipe operator
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

# Callbacks

***Callbacks doesn't support pipe operator yet.***

```scala
Callback asyncCallback = async int a, int b, int c -> "Hello World"

// call it
asyncCallback@
// or await it
await asyncCallback@
```

# Defining classes
Classes in BLAST determined by file naming postfixes and prefixes. A simple class doesn't include any prebuilt postfix or prefix.
There are other class types like:
* Non-dependency injection:
    - Abstract eg AbstractXY.bl
    - Interfaces: XYInterface.bl
    - Enums: XYEnum.bl
    - Model classes: XYModel.bl
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

## Instantiation

```csharp
SomeClass instance1 = {};
SomeClass instance1 = {
    prop1: value1,
    prop2: value2,
    ...
    propN: valueN
};
```

## Abstract classes
Just like you used to in other languages. :) 

```scala
// AbstractAbc.bl
```

## Interface classes
Just like you used to in other languages. :) 

```scala
// AbcInterface.bl
```

## Enum classes

Enum classes only allows methods, parameter declarations and enum definitions

```scala
// AbcEnum.bl
String name;
int number;
boolean condition;

X("X", 1, false),
A("A", 1, false),
B("B", 1, false),
C("C", 1, false)
```

## Model classes

Model classes are describing the properties of some entity and nothing else.
Contains member declarations only. ***Functions not allowed***.

```scala
// AbcModel.bl
String name = "Hello World"; // default value
int number;
boolean condition;
```

## Adapter classes

Adapters used to add functionality to model classes.
***Adapters are virtual by default***

```scala
// AbcModelAdapter.bl
int someFunctionality() {...}
```

# Continuity operator

In order to eliminate nested evaluations inside parenthesis we introduced the `_.` continuity operator.

# Type casts

BLAST is smart enough most of the time. However sometimes type casting may be useful.

## When typecasting not needed

One of the benefits of this left-typed language is that it can easily infer types.

#### Example: passing List of values to a supertype ArrayList

```scala
List<ComplexObjectInterface> a = [o1, o2, o3,...];
ArrayList<ComplexObject> b = a; // BLAST can handle it
```
## When typecasting is needed

If you don't pass the value to other type but you want to use the variable on-the-fly.

#### Example: use the variable on-the-fly as a supertype

```scala
// If only the ComplexObject implements ComplexObjectInterface 
// or the appropriate constructor is invoked (and not ambiguous) the typecast here is negligible.
// Otherwise compile error thrown.
ComplexObjectInterface a = {} as ComplexObject;

// Now you want to call someAction which only available in ComplexObject
a as ComplexObject _. someAction@;
```

## Usages

Some usages:
* Type casts
* coalescing operator 

```java
// Type cast example
// instead of writing ([1, 2, 3] as XYArrayList). slice@ 1, 2; write the following:
[1, 2, 3] as XYArrayList _. slice@ 1, 2
// coalescing operator example
// instead of writing (complexObjectOptional ?? otherValue).someMethod@; write the following:
complexObjectOptional ?? otherValue _. someMethod@;
```

BLAST stays true to itself :)

# Control structures

Control structures are a bit different than in other languages to make the code more readable and simple.

## if statement

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
# Loops
## For loop
In for loops the default increment is 1. You only have to specify increment if other needed.

```java
for int j = 0; j < 10 {
   // code 
}

// Different increment
for int j = 0; j < 10; j += 2 {
   // code 
}
```

# TO BE CONTINUED..
