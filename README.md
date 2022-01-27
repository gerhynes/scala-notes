# Scala Notes

Scala is a statically-typed general purpose programming language which supports both object-oriented and functional programming paradigms. It was designed to be syntactically concise and address some of the criticisms of Java.

Scala source code compiles to Java bytecode and can run on a Java Virtual Machine (JVM), making it compatible with libraries from the Java ecosystem.

## Basics

A Scala project has a similar structure to a Java project.

By making an object extend `App`, it becomes executable as a standalone application.

```Scala
object Basics extends App { ... }
```

### Values and Variables

When defining a value, you use the `val` keyword for immutable values, then the name of the value and its type, before assigning it.

```Scala
val meaningOfLife: Int = 42
```

A value is comparable to a constant in Java. Reassigning to a `val` is illegal because `val`s are immutable.

You don't always need to specify the type as the compiler can often infer the type.

```Scala
val aBoolean = false
```

Frequently used types are Int, Boolean, Char, Float and String (which has its own peculiarities).

```Scala
val aString = "Scala isn't so scary"
```

Strings can be concatenated using the + operator.

```Scala
val anotherString = "Maybe" + " I'll " + " like " + " Scala"
```

Strings can also be interpolated using `s""` and `$`.

```Scala
val interpolatedString = s"The meaning of life is $meaningOfLife"
```

Variables, defined with `var`, are used for side effects in Scala. Variables are mutable. Scala prefers vals over vars.

```Scala
var aVariable: Int = 4  
  
aVariable = 5 // side effects
```

The `+= -= *= /=` operators only work with variables.

```Scala 
var aVariable = 2  
aVariable += 3 // also works with -= *= /= ..... side effects
```

### Expressions

With Scala, you need to think in terms of values and expressions, rather than instructions. 

An instruction is something you tell the computer to do.

Expressions are structures that can be reduced to a value. Values are composed to obtain new values. 

Instructions are executed (Java), expressions are evaluated (Scala).

In Scala, everything is an expression that can be reduced to a value.

An if statement is better thought of as an if-expression.

```Scala
val ifExpression = if (meaningOfLife > 43) 56 else 999
```

You can chain if-expressions

```Scala
val chainedExpression = 
	if (meaningOfLife > 43) 56
	else if (meaningOfLife < 0) -2
	else if (meaningOfLife < 999) 78
	else 0
```

A code block is an expression defined by curly braces. 

These can contain local values and nested code blocks but must eventually return a value. 

The value of a code block is the value of its last constituent expression.

The compiler will assign a type to the code block.

Code blocks have their own scope and can have values only available inside them.

```Scala
val aCodeBlock = {
	val aLocalValue = 67

	// value of a block is the value of the last expression
	aLocalValue + 3
}
```

### Functions

Functions are defined with the keyword `def`, take any arguments, specify a return type, and contain a single expression, which is the return value of the function.

```Scala
def myFunction(x: Int, y: String): String = y + " " + x
```

For longer functions, use a code block (which is itself an expression).

```Scala
def myFunction(x: Int, y: String): String = {
	y + " " + x
}
```

Functions are often recursive in practice, often replacing loops or iteration (which are discouraged in Scala).

It's good practice to specify the return types of functions (even when the compiler can infer them). Recursive functions need to have an explicit return type.

```Scala
def factorial(n: Int): Int = 
	if (n <= 1) 1
	else n * factorial(n - 1)

def aRepeatedFunction(aString: String, n: Int): String = {  
	if (n == 1) aString  
	else aString + aRepeatedFunction(aString, n-1)  
}
```

You can define auxiliary functions inside functions.

```Scala
def aBigFunction(n: Int): Int = {  
	def aSmallerFunction(a: Int, b: Int): Int = a + b  
  
	aSmallerFunction(n, n-1)  
}
```

The `Unit` type is used when your function returns no meaningful value, equivalent to using `void` in other languages. 

The `Unit` type is the type of **side effects**, operations which have nothing to do with computing meaningful information, for example printing something on screen, sending something to a socket or server, storing something in a file.

```Scala 
// returns type Unit
println("This doesn't return anything meaningful")

// A function can explicitly return Unit
def myUnitReturningFunction(): Unit = {
	println("You can explicitly return Unit")
}

// Unit can be represented by ()
val theUnit = ()
```

### Type Inference
When you don't specify a type for a value, the compiler will infer the type and add it behind the scenes.

```Scala
val message = "hello world" // String

val x = 2 // Int

val y = x + "items" // String
```

The compiler can also infer the return type for functions and add it.

If you do specify a type, make sure you comply with it or you'll get a type mismatch error.

```Scala
def success(x: Int) = x + 1 // def success(x: Int): Int = x + 1
```

The compiler can't infer the type of a recursive functions and so this must be made explicit.

```Scala
def factorial(n: Int): Int =  
	if (n <= 0) 1  
	else n * factorial(n-1)
```

### Recursion
In order to run a recursive function, the JVM uses a call stack to keep partial results. Each call uses a stack frame. 

The JVMs internal stack has limited memory and a StackOverflowError will happen when the recursive depth is exceeded.

```Scala
def factorial(n: Int): Int =  
 if (n <= 1) 1  
 else {  
    println("Computing factorial of " + n + " - I first need factorial of " + (n-1))  
    val result = n * factorial(n-1)  
    println("Computed factorial of " + n)  
  
    result  
 }
```

You can avoid a StackOverFlowError using tail recursion. Scala doesn't need to save intermediate results for use later, it uses an accumulator. This lets Scala preserve the same stack frame and not use additional stack frames for recursive calls.

You can annotate functions with `@tailrec` and the compiler will complain if they don't use tail recursion.

```Scala
def anotherFactorial(n: Int): BigInt = {  
	@tailrec  
	def factHelper(x: Int, accumulator: BigInt): BigInt =  
		if (x <= 1) accumulator  
		else factHelper(x - 1, x * accumulator) // TAIL RECURSION = use recursive call as the LAST expression  
  
	factHelper(n, 1)  
}
```

When you need loops, use tail recursion.

### Call By Name and Call By Value

In call by value, the exact value of a parameter is computed before the function is evaluated and the same value is used in the function definition no matter how many times the function is called.

By contrast, in call by name, the parameter is passed literally as is and the expression is evaluated every time it is used when the function runs.

To call a parameter by name, use `=>`. This is useful in lazy streams and things that might fail.

```Scala
def calledByValue(x: Long): Unit = {  
  println("by value: " + x)  // println("by value: " + 1257387745764245L)
  println("by value: " + x)  // println("by value: " + 1257387745764245L)
}  
  
def calledByName(x: => Long): Unit = {  
  println("by name: " + x)  // println("by name: " + System.nanoTime())
  println("by name: " + x)  // println("by name: " + System.nanoTime())
}

calledByValue(System.nanoTime())  
calledByName(System.nanoTime())
```

### Default and Named Arguments
In some cases you will write functions that will be called with the same parameters almost all the time.

Scala lets you set default values for parameters that you don't want to pass every time.

```Scala
def trFact(n: Int, acc: Int = 1): Int =  
 if (n <= 1) acc  
 else trFact(n-1, n*acc)  
  
val fact10 = trFact(10, 2) // you can override default values
```

Leading values with default parameters can't be ommitted. It will confuse the compiler.

Either:

1. Pass in every leading argument.
2. Name the arguments.

```Scala
def savePicture(format: String = "jpg", width: Int = 1920, height: Int = 1080): Unit = println("saving picture")

savePicture(800) // which paramter are you referring to?
savePicture("png") // this works
savePicture(width = 800) // this works
```

If you name arguments, you can pass them in in a different order.

```Scala
savePicture(height = 600, width = 800, format = "png")
```

### String Operations

Scala has access to the Java String class and its methods, such as  `charAt`, `split`, `replace`. 

Besides this, Scala has its own utilities.

You can convert a string to an integer using `toInt`.

You can prepend to a string with `+:` and append with `:+`.

`take` lets you acces a certain number of characters from a string.

```Scala
val aNumberString = "2"  
val aNumber = aNumberString.toInt  
println('a' +: aNumberString :+ 'z')  
println(str.reverse)  
println(str.take(2))
```

You can interpolate a value into a string using `s"$valueName"` and an expression using `s"${}"`.

```Scala
val greeting = s"Hello, my name is $name and I am $age years old"val anotherGreeting = s"Hello, my name is $name and I will be turning ${age + 1} years old."
```

F-interpolators are for formatted strings, similar to `printf`. For example `%2.2f` to show at least 2 characters and to 2 decimals precision.

```Scala
val speed = 1.2f  
val myth = f"$name can eat $speed%2.2f burgers per minute"  
println(myth)
```

Raw interpolated strings ignore escaped characters inside raw strings but injected variables do get escaped.

```Scala
println(raw"This is a \n newline")  // This is a \n newline

val escaped = "This is a \n newline"
println(raw"$escaped") // This is a 
											 // newline
```

## Object-Oriented Programming
Scala is an object-oriented language with classes that can be instantiated.

Classes can be extended through inheritance. The subclass inherits all the members and methods of the superclass. 

A class doesn't have to have a body.

You can pass arguments to a class and they will be passed to its constructor.

Constructor arguments (class parameters) are not fields and are not available outside the class definition. Constructor arguments can be converted to fields using `val`.

At every instantiation of a class, its entire code block will be evaluated. Every expression within and all side effects will be executed.

```Scala
class Animal {
	val age: Int = 0
	def eat() = println("I'm eating")
}

val anAnimal = new Animal

class Dog(name: String) extends Animal
val aDog = new Dog("Lassie")
aDog.name // won't compile

class Cat(val name: String) extends Animal
val aCat = new Cat("Minxie")
aCat.name // will compile
```

The `this` keyword refers to the particular instance. Any time you reference an object's field, `this` is implied.

If you have a method parameter with the same value as a field, you'll need to use `this` to distinguish them.

```Scala
def greet(): Unit = println(s"Hi, I'm $name") // println(s"Hi, I'm $this.name")

def greet(name: String): Unit = println(s"this.name says hi, $name")

val person = new Person("John")
person.greet("Daniel")
```

Scala supports multiple constructors. The only thing an auxilary constructor can do is to call another constructor (primary or secondary), so they can essentially be avoided by providing default parameters.

```Scala
class Person(Name: String, age: Int) {

// auxiliary constructor - calls primary constructor
def this(name: String) = this(name, 0)
}
```

### Objects

Scala does not have class-level functionality. It doesn't know the concept of "static". A Scala object can, however, have static-like functionality.

Objects are defined similarly to classes but do not receive parameters. An object can have values and method definitions.

A Scala object is a singleton instance. When you define an object, you define its type and its only instance.

In practice, you will write objects and classes with the same name in the same scope: this is to seperate instance functionality and class-level functionality. 

You will always be accessing code from an instance, either a regular instance or the singleton instance.

In objects, you will often have factory methods to build these objects.

```Scala
object Person { // type + its only instance  
 // "static"/"class" - level functionality 
	val N_EYES = 2  
	def canFly: Boolean = false  
  
	// factory method  
	def apply(mother: Person, father: Person): Person = new Person("Bobbie")  
}

val mary = new Person("Mary")
val john = new Person("John")

val bobbie = Person(mary, john)
```

Since a Scala application has to run on the JVM, a Scala application is a Scala object that either:
- has a main method: `def main(args: Array[String]): Unit`
- extends App (which already has a main method): `object Person extends App {...}`

### Inheritance

Scala has single class inheritance. 

The subclass only inherits non-private members of the superclass.

```Scala
class Animal {  
	val creatureType = "wild"  
	def eat = println("nomnom")  
}  
  
class Cat extends Animal {  
	def crunch = {  
	  eat  
		println("crunch crunch")  
  }  
}
```

### Constructors

When you instantiate a subclass, the JVM first calls the constructor of its parent class.

The Scala compiler forces you to guarantee that there is a correct super constructor to call when using a derived class.

If you use an auxiliary constructor, you can also specify that in the extends clause.

```Scala
class Person(name: String, age: Int) {  
	def this(name: String) = this(name, 0) // auxiliary constructor
}  
class Adult(name: String, age: Int, idCard: String) extends Person(name)
```

### Overriding

Overriding works for both methods and vals. You can override fields in the class body or directly in the constructor arguments.

All instances of derived classes will use the overriden member whenever possible.

You can use `super` to reference a method or field from a parent class.

```Scala
class Dog(override val creatureType: String) extends Animal {  
  // override val creatureType = "domestic" // can override in the body or directly in the constructor arguments  
	override def eat = {  
	  super.eat  
		println("crunch, crunch")  
  }  
}

val dog = new Dog("domestic") // overrides creatureType
dog.eat  
println(dog.creatureType)
```

There are cases when you want to limit overriding of fields/methods:
1. use `final` on a member  `final def eat = println("nomnom")`
2. use `final` on the entire class `final class Animal {...}`
3. seal the class. You can extend classes in this file, but it prevents extension in other files `sealed class Animal {...}`

### Subtype Polymorphism

Scala has subtype polymorphism.

In the example below, at compile time, the compiler only knows you're calling the method from an animal object. At runtime it will be called from a dog object if the Dog class overrides this method inherited from Animal.

The most derived method will be called at runtime.

```Scala
val aDeclaredAnimal: Animal = new Dog("Hachi")
aDeclaredAnimal.eat() // the most derived method will be called at runtime
```

### Abstract Classes

Scala has abstract classes, where all fields and methods don't necessarily need to have implementation. Whatever class implements the abstract class will ned to provide an implementation of the methods.

```Scala
abstract class WalkingAnimal {
	protected val hasLegs = true // by default public
	def walk(): Unit
}
```

### Access Modifiers

All fields and methods are by default public. You can restrict them using the `private` or `protected` keywords.

A `private` member or method is only accessible by the class. `Protected` means that a class and all its descendents have access to the field or method.

### Traits (Interfaces)

Scala also has interfaces, traits, for the ultimate abstract type, where everyhting is unimplemented. 

You **can** provide implementation in traits but they are usually used to denote characteristics of objects that can later be implemented in concrete classes.

```Scala
trait Carnivore {
	def eat(animal: Animal): Unit
}
```

Scala offers single-class inheritance and multi-trait inheritance. 

If a concrete class implements a trait with abstract methods, those methods need to be implemented or the subclass needs to be declared abstract itself.

When you implement a method that's also present in a supertpe, it's an `override`.

```Scala
// single-class inheritance and multi-trait "mixing"
class Crocodile extends Animal with Carnivore {
	override def eat(animal: Animal): Unit = println("Eating an animal")
}
```

### Method Notation and Method Naming

Method notation is a form of syntactic sugar to make Scala more closely resemble natural language.

Methods that have a single parameter can be called using infix notation / operator notation: `object method argument`.

```Scala
val aCroc = new Crocodile
aCroc.eat(aDog)
aCroc eat aDog // infix notation
```

Scala is quite permissive with method naming. For example, `+` or `?!` are valid method names. `?` and `!` are often used in Akka.

```Scala
trait Philosopher {
	def ?!(thought: String): Unit
}
```

Operators in Scala are actually methods. For example, the `+` operator is actually a mthod belonging to the Int type and equivant to `.+()`.

```Scala
val basicMath = 1 + 2
val moreBasicMath = 1.+(2)
```

Scala also has prefix notation. Unary operators are actually methods with `unary_` prefixed to them.

The `unary_` prefix only works with `-` , `+`, `~` and `!`.

```Scala
val x = -1 // equivalent to 1.unary_-
val y = 1.unary_-
```

Methods that don't receive any parameters can be used with postfix notation. This can, however, introduce ambiguities when reading the code.

```Scala
println(Mary.isAlive)
println(Mary isAlive)
```

`apply` as a method name has special properties in Scala. Whenever the compiler sees an object being called like a function, it looks for a definition of `apply` in that class.

```Scala
mary.apply() 
mary() // these are equivalent
```

### Anonymous Classes

Scala has anonymous classes. You can implement a trait as long as you immediately implement its abstract methods.

```Scala
trait Carnivore {
	def eat(animal: Animal): Unit
}

val dinosaur = new Carnivore {
	override def eat(animal: Animal): Unit =  "Carnivorous dinosaurs can eat pretty much whatever they want"
}
```

You're essentially telling the compiler:

```Scala
class Carnivore_Anonymous_35728 extends Carnivore {

override def eat(animal: Animal): Unit = println("Carnivorous dinosaurs can eat pretty much whatever they want")
}

val dinosaur = new Carnivore_Anonymous_35728
```

### Singleton Objects
A singleton object is a class that has exactly one instance.

Scala has an `apply` method that can be present in any object. The presence of the apply method in a class allows instances of that class to be invoked like functions.

You can call the apply method directly, with its arguments, or call it using the name of the singleton object. This can be useful in functional programming.

```Scala
object MySingleton { // the only instance of the mySingleton type
	val mySpecialValue = 53278
	def mySpecialMethod(): Int = 5327
	def apply(x: Int): Int x + 1
}

MySingleton.mySpecialMethod()
MySingleton.apply(65)
MySingleton(65) // equivalent to MySingleton.apply(65)
```

### Companions

You can define a class (or trait) and a singleton object in the same file with the same name. In this instance, the class and object are called companions.

Companions can access each other's private fields and methods. 

But the Animal singleton object and instances of the Animal type are different things.

```Scala
class Animal {
	val age: Int = 0
	def eat() = println("I'm eating")
}

object Animal {
	val canLiveIndefinitely = false
}

val animalsCanLiveForever = Animal.canLiveIndefinitely // similar to static fields/methods
```

### Case Classes
Case classes are lighweight data structures with some boilerplate.

When you define a case class, the compiler automatically generates 
- a sensible equals and hashcode (for inclusion into various collections that rely on equality).
- sensible and quick serialization
- companion with apply
- pattern matching

```Scala
case class Person(name: String, age: Int)

// new keyword can be ommitted because of companion
val bob = Person("Bob", 54) // equivalent to Person.apply("Bob, 54")
```

### Exceptions

Exceptions are treated as special ojects by the JVM and are handled using a try catch block, with an optional finally clause that will execute no matter what. 

The finally clause is useful for closing connections or releasing resources that would otherwise be dangerous to leave open.

```Scala
try {
	// code that can throw
	val x: String = null
	x.length
} catch {
	case e: Exception => "Some error message"
} finally {
	// Executes no matter what
}
```

### Generics
Generics let you pass a type (denoted by a single letter) to a class or trait. Inside you can have definitions that depend on this type.

When you later use the class, the type becomes concrete.

For example, in the Scala standard library `List` is generic and can be told to hold specific types, such as Ints or Strings.

```Scala
abstract class MyList[T] {
	def head: T
	def tail: MyList[T]
}

// using a generic with a concrete type
val aList: List[Int] = List(1,2,3) // List.apply(1,2,3)
val first = aList.head // Int
val rest = aList.tail

val aStringList = List("Hello", "world")
val firstString = aStringList.head // String
```

### Immutability
Scala operates with immutable values/objects. Any modification to an instance of a class should result in a new instance.

This has two benefits:
- It works great in multithreaded/distributed environments.
- It helps you reason about the code

```Scala
val aReversedList = aList.reverse // returns a NEW list
```

### Object-Oriented vs Functional

Scala is marketed as a mix of object-oriented and functional programming. It is actually closer to object-oriented. All the code and values you operate with are inside an instance of some type. There are no methods outside a class/object.

### Extending App with Objects

When you define an object that extends `App` you are inheriting from the App type's ``main`` method.

Methods on objects are similar to Java static methods. So you already have a static main method implemented by extending App. This is what makes them runnable.

```Scala
object ObjectOrientation extends App {...}
```

## Functional Programming
Scala is an object-oriented language and runs on the JVM. The JVM knows what an object is but doesn't know what a function is as a first class citizen.

In functional programming the goal is to work with functions as with any other kind of value:
- compose functions
- pass functions as arguments
- return functions as results

To gte the JVM to work with functional programming, Scala's creators created the concept FunctionX.

When you create an object that can be invoked like a function (because it has an apply method), and the only thing it supports is to be invoked like a function, you've essentially defined a function. It acts like a function and the only thing it can do is act like a function.

All Scala functions are instances of these FunctionX types.

FunctionX = Function1, Function2 ... Function22. 

22 is the maximum number of arguments you can pass to a function. The last argument passed to it is the return type.

```Scala
val simpleIncrementor = new Function1[Int, Int] {
	override def apply(arg: Int): Int = arg + 1
}

simpleIncrementor.apply(23) // 24
simpleIncrementor(23) // 24 - same as calling apply

// function with 2 arguments and String return type
val stringConcatenator = new Function2[String, String, String] {
	override def apply(arg1: String, arg2: String): String = arg1 + arg2
}

stringConcatenator("Woo ", "Scala") // Woo Scala
```

### Syntactical Sugar
Syntactical sugar is alternative syntax that replaces more verbose boilerplate code.

For example, a function that takes in an int and doubles it can be written as:

```Scala
val doubler: Int => Int = (x: Int) => 2 * x
// or
val doubler = (x: Int) => 2 * x // compiler infers type

doubler(4) // 8
```

This is equivalent to:

```Scala
val doubler: Function1[Int, Int] = new Function1[Int, Int] {
	override def apply(x: Int) = 2 * x
}
```

### Higher-Order Functions
Methods that take functions as arguments or return functions as results are called higher-order functions.

For example, when working with Lists, the `map` method takes an anonymous function (lambda) as its argument. The function is applied to every element in the List.

The `flatMap` method concatenates any Lists produced by the anonymous function and raturns a bigger list.

The `filter` method takes an anonymous function and returns only those elements for which the condition evaluates as true. 

You can chain, map, flatMap and filter.

These methods let you iterate through collections without using loops or iterators.

```Scala
// map
val aMappedList: List[Int] = List(1,2,3).map(x => x + 1) // List(2,3,4)

// flatMap
val aFlatMappedList = List(1,2,3).flatMap(x => List(x, 2 * x)) // List(1,2,2,4,3,6)

// Alternative syntax
val aFlatMappedList = List(1,2,3).flatMap {x => 
	List(x, 2 * x)
} // List(1,2,2,4,3,6)

// filter
val aFilteredList = List(1,2,3,4,5).filter(x => x <= 3) // List(1,2,3)

// Alternative syntax
val aFilteredList = List(1,2,3,4,5).filter(_ <= 3) // List(1,2,3)

// All pairs between numbers 1,2,3 and letters 'a', 'b', 'c'
val allPairs = List(1,2,3).flatMap(number => List('a','b','c').map(letter => s"$number-$letter")) // List(1-a, 1-b, 1-c, 2-a, 2-b, 2-c, 3-a, 3-b, 3-c)

```

### For Comprehensions

Instead of using chains of higher-order functions, you can use a for comprehension, which can be reduced to a single value. The compiler will interpret it as a chain.

```Scala
// chain
val allPairs = List(1,2,3).flatMap(number => List('a','b','c').map(letter => s"$number-$letter"))

// for comprehension
val alternativePairs = for {
	number <- List(1,2,3)
	letter <- List('a', 'b', 'c')
}   yield s"$number-$letter" 
```

## Collections
### Lists

Lists are the fundamental collection in functional programming.

A list has a head (the first element) and a tail (the remainder of the list).

Lists can be prepended and appended with elements.

`+:` prepends an element
`:+` appends an element

```Scala
val aList = List(1,2,3,4,5)
val firstElement = aList.head
val rest = aList.tail
val aPrependedList = 0 :: aList
val anExtendedList = 0 +: aList :+ 6
```

### Sequences

A sequence is a collection where you can access an element at a given index.

`Seq` is a Trait

```Scala
val aSequence = Seq[Int] = Seq(1,2,3) // Seq.apply(1,2,3)
val accessedElement = aSequence(1) // the element at that index - 2
```

### Vectors
Vectors are indexed, immutable sequences with very fast access times and the same methods as Lists and Sequences.

```Scala
val aVector = Vector(1,2,3,4,5)
```

### Sets
Sets are collections with no duplicates. The main use of a Set is to test whether an element is contained in the Set, using the `contains` method. The order of the elements is not important.

You can add or remove elements with the `plus` and `minus` methods.

```Scala
val aSet = Set(1,2,3,4,1,2,3) // Set(1,2,3,4)
val setHas5 = aSet.contains(5) // false
val anAddedSet = aSet + 5 // Set(1,2,3,4,5)
val aRemovedSet = aSet - 3 // Set(1,2,4,5)
```

### Ranges
Ranges are used for iteration. For example, the following range doesn't contain all the nunbers from 1 to 1000 but it can be processed as if it did.

```Scala
val aRange = 1 to 1000

// will generate a list of all even numbers between 2 and 2000
val twoByTwo = aRange.map(x => 2 * x).toList
```

You can use the `toList`, `toSet`, `toSeq` methods to convert between these collections.

### Tuples
Tuples are a simple container for disparate items.

```Scala
val aTuple = ("Bon Jovi", "Rock", 1982)
```

### Maps
A Map is an iterable sequence that consists of pairs of keys and values.

```Scala
val aPhoneBook: Map[String, Int] = Map(
	("Daniel", 034937846),
	"Jane" -> 187327063 // equivalent to ("Jane", 187327063)
)
```

## Pattern Matching
Pattern matching is a mechanism for checking a value against a pattern. It's comparable to a switch statement in Java and can be used instead of a series of if/else statements. Think of it as a switch expression.

Pattern matching will try all cases in sequence.

`_` serves as the catch all case. Cases are also called alternatives.

```Scala
val anInteger = 55

val order = anInteger match {
	case 1 => "first"  
	case 2 => "second"  
	case 3 => "third"  
	case _ => anInteger + "th"
}
```

Pattern matching can also deconstruct data structures into their constituent parts.

One of the benefits of case classes is being able to deconstruct them in pattern matching. Pattern matching can also be used for normal classes but requires much more work.

```Scala
case class Person(name: String, age: Int)
val bob = Person("Bob", 43) // equivalent to Person.apply("Bob", 43)

val personGreeting = bob match {  
	case Person(n, a) => s"Hi, my name is $n and I am $a years old." 
	case _ => "Something else"  
}
```

Pattern matching can also deconstruct tuples.

```Scala
val aTuple = ("Bon Jovi", "Rock")  
val bandDescription = aTuple match {  
	case (band, genre) => s"$band belongs to the genre $genre"  
	case _ => "I don't know what you're talking about"  
}
```

You can also decompose lists using pattern matching.

```Scala
val aList = List(1,2,3)  
val listDescription = aList match {  
	case List(_, 2, _) => "List containing 2 in index 1"  
	case _ => "unknown list"  
}
```

If a pattern match doesn't match anything it'll thorw a MatchError so it's good practice to include a catch all case. 

## More Advanced Features
### Lazy Evaluation
Lazy evaluation means an expression is not evaluated until it's first used. It's useful ininfinite collections.

```Scala
lazy val aLazyValue = 2  
lazy val lazyValueWithSideEffect = {  
  println("I am so very lazy!")  
  43  
}

val eagerValue = lazyValueWithSideEffect + 1 // lazyValueWithSideEffect used, code block triggered
```

### "Pseudo Collections"

`Option` and `Try` are useful when you have unsafe methods, for example saving you from having to check for the method returning a Null value.

An Option is like a "collection" containing at most one element. 

For example, you can pass a method which can return Null to Option. If the method returns a valid value, then the "collection" contains that value. It'll return ``Some``, a subtype of Option.

If the method returns Null, the value will be ``None`` (a singleton object and still a valid value).

You can used Options with pattern matching and operate on them with map, flatMap and filter.

```Scala
def methodWhichCanReturnNull(): String = "Hello, Scala"  
val anOption = Option(methodWhichCanReturnNull()) // Some("Hello, Scala")  
// option = "collection" which contains at most one element: Some(value) or None

val stringProcessing = anOption match {  
	case Some(string) => s"I have obtained a valid string: $string"  
	case None => "I obtained nothing"  
}
```

A Try guards against methods which can throw exceptions, avoiding having to write defensive try/catch blocks. 

It's essentially a "collection" with either a value if the code went well, or an exception if the code threw one. The two subtypes of Try are `Success` and `Failure`.

The Try object can also be processed with map, flatMap and filter.

```Scala
def methodWhichCanThrowException(): String = throw new RuntimeException  
val aTry = Try(methodWhichCanThrowException())  

val anotherStringProcessing = aTry match {  
	case Success(validValue) => s"I have obtained a valid string: $validValue"  
	case Failure(ex) => s"I have obtained an exception: $ex"  
}
```


### Asynchronous Programming
asynchronous programming in Scala is done with another pseudo-collection, `Future`. 

A Future represents a value which may or may not _currently_ be available, but will be available at some point, or an exception if that value could not be made available.

```Scala
val aFuture = Future { // equivalent to Future.apply()
  println("Loading...")  
  Thread.sleep(1000)  
  println("I have computed a value.")  
  67  
}
```

In order to run a future, you need to import an execution context, for example `global`. The global value is a collection of threads on which you can schedule the evaluation of an expression.

A Future is a "collection" which contains a value when it's evaluated.

A Future is composable with map, flatMap and filter simialr to other pseudo-collections.

Future, Try and Option are "Monads" in functional programming.

### Implicits
Implicits are used for 
- implicit arguments
- implicit conversions

If you define a method with implicit arguments you can call it without passing any arguments. The compiler understands that the method takes an implicit argument and looks for implicit values.

```Scala
def aMethodWithImplicitArgs(implicit arg: Int) = arg + 1  
implicit val myImplicitInt = 46  
println(aMethodWithImplicitArgs)  // aMethodWithImplicitArgs(myImplicitInt)
```

You use implicit conversions to add methods to existing types over which you don't have control.

For example, you can give an Int an isEven method using implicit conversion.

This is dangerous. Use it carefully.

```Scala
implicit class MyRichInteger(n: Int) {  
	def isEven() = n % 2 == 0  
}  
  
println(23.isEven()) // equivalent to new MyRichInteger(23).isEven()
```

## Contextual Abstractions (Scala 3)
### Context Parameters/Arguments
Scala's compiler sorts a list by accessing the built-in value of the trait  `Ordering[Int]`.

The `sorted` method takes an argument list but that argument list contains an element of type `Ordering[Int]`. This element is contextual and depends on where the method is being called.

For every method that takes a "magical" argument of type `Ordering[Int]`, the compiler looks for a `given` instance.

Depending on the `given` instances that you define in the scope of a method with these kinds of arguments, you can change the built-in behaviour of the method.

```Scala
val aList = List(2,1,4,3)

// customize sorted to be descending
val anOrderedList = aList.sorted // contextual argument: (descendingOrdering)

// Ordering
given descendingOrdering: Ordering[Int] = Ordering.fromLessThan(_ > _) // (a,b) => a > b
```

A given instance is analogous to an implicit val.

**NB** Implicits are going to be deprecated in future versions of Scala 3.

A contextual argument can be specified using the `using` keyword. The compiler will look for a given instance.

```Scala
trait Combinator[A] { // monoid
	def combine(x: A, y: A): A
}

def combineAll[A](list: List[A])(using combinator: Combinator[A]): A = list.reduce((a,b) => combinator.combine(a,b))

given intCombinator: Combinator[Int] = new Combinator[Int] {

override def combine(x: Int, y: Int) = x + y

}

val theSum = combineAll(aList) // (intCombinator)
```

The compiler looks for given instances in:
- the local scope
- the imported scope
- the companions of all the types involved in the method call

In this instance the compiler will look in the companion of List and the companion of Int.

### Context Bounds
There are shorter syntaxes available for contextual arguments.

If you're not using the instance directly but are calling other methods that use it, you can use `using` and you don't need to name the particular instance.

You can alternatively specify a type restriction. This alerts the compiler that you need an instance in scope where this method is going to be called.

```Scala
def combineAll_v2[A](list: List[A])(using Combinator[A]): A = ???

def combineAll_v3[A : Combinator](list: List[A]): A = ???
```

Context arguments are useful for:
- type classes
- dependency injection
- context-dependent functionality
- type-level programming

### Extension Methods

You can add additional methods to a type after it was defined even if you have no control over the source of that type.

Extension methods are heavily used in functional programming libraries, such as Cats.

You can use the `extension` keyword to add extensions to a type.

```Scala
case class Person(name: String) {
	def greet(): String = s"Hi, my name is $name, I love Scala!"
}

extension (string: String)
	def greet(): String = new Person(string).greet()

val danielsGreeting = "Daniel".greet() // "type enrichment"
```

Extension methods can be combined with context parameters.

```Scala
extension [A] (list: List[A])
	def combineAllValues(using combinator: Combinator[A]): A = list.reduce(combinator.combine)

val theSum_v2 = aList.combineAllValues
```
