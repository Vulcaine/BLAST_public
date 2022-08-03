# BLAST

This is the public repository for an overview of the current progress.
## **Please give me a ⭐ if you think my work is worthy**

BLAST is a left-typed language, which means **you only write types to the left side of each declaration and won't see types on the right side**.

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
5. [Optional unlocking](#optional-unlocking)
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
9. [Type casting](#type-casting)
10. [Control structures](#control-structures)
    1. [If statement](#if-statement)
11. [Loops](#loops)
    1. [For loop](#for-loop)
12. [Namespaces](#namespaces)
    1. [Flat-namespaces](#flat-namespaces) 
13. [Java interoperability](#java-interoperability)
14. [Exception handling](#exception-handling)

# Creating primitives
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

# Dealing with method argument subsets

In BLAST we can define multiple varargs but **vararg must be followed by vararg**.

**Example: Three input sets**
```java
int varargMethod(int... a, int... b, int... c) {
   return 3;
}

int varargMethod2(int a, int... b, int... c) {
   return 3;
}

int varargMethodMultipleTypeArgs(int a..., String... b, int... c) {
   return 3;
}

int varargMethod4(int... a, int b, int... c) {
   return 3;
} // Not allowed yet ❌
```
The varargMethod above requires zero or 1+ int values which divided into three parts.

**Example: call the varargMethod and varargMethodMultipleTypeArgs methods**
```java
varargMethod@ 1 // where 'a' is going to be [1], 'b' is [], 'c' is []
varargMethod@ 1,2,3 // where 'a' is going to be [1], 'b' is [2], 'c' is [3]
varargMethod@ 1,2,3,4,5,6 // where 'a' is going to be [1, 2], 'b' is [3, 4], 'c' is [5, 6]
varargMethod@ 1,2,3,4 // where 'a' is going to be [1], 'b' is [2], 'c' is [3, 4]

varargMethodMultipleTypeArgs@ 1,2,"a","b","c","d",3,4 // where 'a' is going to be [1,2], 'b' is ["a","b","c","d"], 'c' is [3, 4]
```

# Optional unlocking

In BLAST **null value is eliminated**. Welcome the world of optionals.
The keyword `empty` replaces `null` in a way that we can define empty values (**but empty != null**).

```java
Optional optionalEmpty = empty; // Optional.empty()
Optional<char> optionalChar = "c"; // Optional.of('c')
Optional<String> optionalString = "string"; // Optional.of("string")
Optional<String> optionalOptional = optionalString; // Optional.of("string")
Optional<int> optionalInt = 3; // Optional.of(3)

// You can't resolve empty into other values
int emptyNotAllowed = optionalEmpty // Compile error ❌

int optionalResolved = optionalInt; // optionalInt.getOrElseThrow(runtime error);
int optionalResolvedSafely = optionalInt??; // optionalInt.getOrElse(0);
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
(complexObjectOptional ?? otherValue).someMethod@;
```

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
project
│   src
└───  AbstractAbc.bl
```

## Interface classes
Just like you used to in other languages. :) 

```scala
project
│   src
└───  AbcInterface.bl
```

## Enum classes

Enum classes only allows methods, parameter declarations and enum definitions

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

## Model classes

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

## Adapter classes

Adapters used to add functionality to model classes.
***Adapters are virtual by default***

```scala
project
│   src
└───  AbcModelAdapter.bl

int someFunctionality() {...}
```

# Type casting

BLAST is smart enough most of the time. However sometimes type casting may be useful.

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
// If only the ComplexObject implements ComplexObjectInterface 
// or the appropriate constructor is invoked (and not ambiguous) the typecast here is negligible.
// Otherwise compile error thrown.
ComplexObjectInterface complexObject = <ComplexObject>{};

// Now you want to call someAction which only available in ComplexObject
<ComplexObject>complexObject.someAction@;
```

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

### Regular loops
```java
for int j = 0; j < 10 {
   // code 
}

// Different increment
for int j = 0; j < 10; j += 2 {
   // code 
}
```

### Enhanced loops

```scala
// Simplified version
for(int i = 0:10) {

}

// Simplified version different increment
for(int i = 0:10:2) {
    
}

// Simplified version for characters
for(char c = "a":"z") {
   // j will be the characters from "a" to "z"
}

// Simplified version for strings
for(String s = "abc":"adb") {
   // j will be the strings from "abc" to "adb"
}

// Iterate lists
for(ComplexObject o : complexObjectList) {...}
ComplexObject[] shallowCopyList = for(complexObjectList);

// Splitting string into char array, the for loop way
String string = "Hello world";
char[] chars = for(string);

// Create an array from an int (from 0 -> 100)
int number = 100;
int[] integers = for(number);

// Some other examples
String[] out = for("abc":"adb"); // if no body provided the return values are the values of j
String[] out = for(String s = "abc":"adb"): "hello, $s"; // ["hello, abc", "hello, abd", "hello, abe", ..., "hello, adb"]
```

### As you can see above for loops can be used for mapping as well

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

// Which obviously worse than using the map method, but this is just an example..
// Use this instead
String[] mapped = stringArray | filter@ name -> name == "Kate" | map@ name -> "Hello $name";
```


# Namespaces
In BLAST we don't write import statements as the compiler smart enough to know which dependency you want to use.
However, sometimes, just like in other languages, there are name conflicts. That is what `namespace` keyword is used for.
Namespace differs from import in that it specifies a `space` for the search instead of the `module` to be imported.

### Example
Let's say we have a `Console` module in `A.somepackage` and `A.somepackage2` and we want to use the one lays in `A.somepackage`.

**First option**
```scala
A.somepackage.Console.log@ "hello";
```

**Second option**
```csharp
namespace A.somepackage;

Console.log@ "hello";
```

## Flat namespaces
Sometimes you don't want to write out the module's name everytime you call a function from it. eg. `Module.function@ input`
In this case, flat namespaces help.

```csharp
flat namespace A.somepackage.Console;

log@ "hello";

// But we can still call via the normal way as well.
Console.log@ "hello" 
```

# Java interoperability
# Exception handling

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
**handle** always goes to the end of each method.

# TO BE CONTINUED..
