# Scala Notes

Scala is a statically-typed general purpose programming language which supports both object-oriented and functional programming paradigms. It was designed to be syntactically concise and address some of the criticisms of Java.

Scala source code compiles to Java bytecode and can run on a Java Virtual Machine (JVM), making it compatible with libraries from the Java ecosystem.

## Basics

A Scala project has a similar structure to a Java project.

By making an object extend `App`, it becomes executable as a standalone application.

```Scala
object Basics extends App { ... }
```

When defining a value, you use the `val` keyword for immutable values, then the name of the value and its type, before assigning it.

```Scala
val meaningOfLife: Int = 42
```

A value is comparable to a constant in Java. Reassigning to a `val` is illegal.

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

With Scala, you need to think in terms of values and expressions, rather than instructions. 

Expressions are structures that can be reduced to a value. Values are composed to obtain new values. 

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

A code block is defined by curly braces. These can contain local values and nested code blocks but must eventually return a value (the last expression of the code block). The compiler will assign a type to the code block.

```Scala
val aCodeBlock = {
	val aLocalValue = 67

	// value of a block is the value of the last expression
	aLocalValue + 3
}
```

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

Functions are often recursive in practice, often replacing loops or iteration (which are discouraged).

```Scala
def factorial(n: Int): Int = 
	if (n <= 1) 1
	else n * factorial(n - 1)
```

The `Unit` return type is used when you have no meaningful value, equivalent to using `void` in other languages. The `Unit` type is the type of **side effects**, operations whihc have nothing to do with computing meaningful information, for example printing something on screen, sending something to a socket or server, storing something in a file.

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

## Object-Oriented Programming
Scala is an object-oriented language with classes that can be instantiated.

Classes can be extended through inheritance. The subclass inherits all the members and methods of the superclass. 

A class doesn't have to have a body.

You can pass arguments to a class and they will be passed to its constructor.

Constructor arguments are not fields and are not available outside the class definition.

Constructor arguments can become fields using `val`.

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
val aCat = new Dog("Minxie")
aCat.name // will compile

```

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

### Methods and Method Naming

Methods that have a single argument can be written using infix notation: `object method argument`.

```Scala
val aCroc = new Crocodile
aCroc.eat(aDog)
aCroc eat aDog // infix notation
```

Scala is quite permissive with method naming, for example, `?!` is a valid method name. `?` and `!` are often used in Akka.

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