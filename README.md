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

### Access Modifiers
All fields and methods are by default public. You can restrict them using the `private` or `protected` keywords.

A `private` member or method is only accessible by the class. `Protected` means that a class and all its descendents have access to the field or method.

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
Scala has abstract classes, where all fields and methods don't necessarily need to have implementation. Abstract classes can have both sbstract and non-abstract members.

Whatever class implements the abstract class will need to provide an implementation of the abstract methods.

Abstract classes cannot be instantiated, they are made to be extended later.

```Scala
abstract class WalkingAnimal {
	protected val hasLegs = true // by default public
	def walk(): Unit // to be implemented in concrete class
}
```

### Traits (Interfaces)
Scala also has traits, for the ultimate abstract type, where everything is unimplemented. 

Traits can have both abstract and non-abstract members. You **can** provide implementation in traits but they are usually used to denote characteristics of objects that can later be implemented in concrete classes.

```Scala
trait Carnivore {
	def eat(animal: Animal): Unit
}
```

Scala offers single-class inheritance but multi-trait inheritance. 

If a concrete class implements a trait with abstract methods, those methods need to be implemented or the subclass needs to be declared abstract itself.

When you implement a method that's also present in a supertype, it's an `override`.

```Scala
// single-class inheritance and multi-trait "mixing"
class Crocodile extends Animal with Carnivore {
	override def eat(animal: Animal): Unit = println("Eating an animal")
}
```

Traits differ from abstract classes in the following ways:
1. traits do not have constructor parameters (in Scala 2)  
2. multiple traits may be inherited by the same class  
3. traits describe "behavior", abstract classes describe a "thing"

### Scala's Type Hierarchy
Scala's type hierarchy starts with `scala.Any`, then `scala.AnyRef` (mapped to java.lang.Object). All classes (for example, String, List, Set) will derive from `AnyRef` unless you explicitly say they extend some other class. Derived from all of these is `scala.Null` (its only instance is the null reference).

`scala.AnyVal` derives from `scala.Any` and contain all the primitive values (Int, Unit, Boolean, Float).

Derived from all of them is `scala.Nothing`, in the sense that nothing is a subtype of every single thing in Scala and can replace everything.


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
Scala has anonymous classes, which let you instantiate types and override fields and methods on the spot.

They also effectively let you instantiate abstract classes. 

You can implement a trait as long as you immediately implement its abstract fields and methods. 

If your anonymous class is extending a class, you need to provide the proper constructor arguments (if needed).

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
Case classes are lighweight data structures with little boilerplate.

They let you define a class, a companion object and a lot of sensible default in one go.

When you define a case class, the compiler automatically generates: 
- a sensible equals and hashcode (for inclusion into various collections that rely on equality).
- sensible and quick serialization
- a companion object with `apply`
- pattern matching

With case classes:
- class parameters are promoted to fields
- a sensible `toString` method is provided
- printing an instance automatically delegates to `toString` 
- the `copy` method (which can take named parameters) creates a new instance

Case classes are serializable which is useful when dealing with distributed systems. This is heavily used by Akka.

Case classes have extractor patterns and can be used in pattern matching.

```Scala
case class Person(name: String, age: Int)

// new keyword can be ommitted because of companion
val bob = Person("Bob", 54) // equivalent to Person.apply("Bob, 54")
println(bob.name)
println(bob) // Person(Bob, 54)
```

There are also case objects, which have the same properties as case classes except they don't get companion objects (they are their own companion object).

### Enums (Scala 3)
Enums are a data type you can define much like a class, where you can enumerate all the possible caes (constants) of that type. 

An enum is a sealed data type that cannot be extended.

Enums can have fields and methods, can take constructor arguments, and can have companion objects.

You can use an enum's companion object as a source for factory methods.

```Scala
enum Permissions {
	case RAED, WRITE, EXECUTE, NONE

	def openDocument(): Unit = 
		if (this == READ) println("Opening document...")
		else println("Reading not allowed")
}

val somePermissions: Permissions = Permissions.READ

enum PermissionsWithBits(bits: Int) {
	case READ extends PermissionsWithBits(4) // 100
	case WRITE extends PermissionsWithBits(2) // 010
	case EXECUTE extends PermissionsWithBits(1) // 001
	case NONE extends PermissionsWithBits(0) // 000
}

object PermissionsWithBits {
	def fromBits(bits: Int): PermissionsWithBits = ???
	PermissionsWithBits.NONE
}

def main(args: Array[String]): Unit = {
	somePermissions.openDocument()
}
```

### Exceptions
In Scala throwing an exception is an expression which returns `Nothing`.

Exceptions are instances of classes. For example, when you throw a `NullPointerException` you are instantiating an instance of `NullPointerException` and then throwing it. 

Exceptions (and Errors) are throwable beacuse they extend the `Throwable` class.

An Exception denotes something that went wrong with the program, for example `NullPointerException`. Errors denote something that went wrong with the system, for example a `StackOverflow` error.

```Scala
val aWeirdValue = throw new NullPointerException
```

Exceptions are treated as special ojects by the JVM and are handled using a try catch block, with an optional finally clause that will execute no matter what. 

The finally clause is used for side effects, for example closing connections or releasing resources that would otherwise be dangerous to leave open.

Like everything in Scala, a try catch finally block is an expression. The compiler will attempt to unify the types of values returned and will use  `AnyVal` if the values returned from the try and catch blocks are different.

```Scala
try {
	// code that can throw an exception
	val x: String = null
	x.length
} catch {
	case e: Exception => println("Some error message")
} finally {
	// Executes no matter what
}
```

Exceptions in Scala come from Java and are specific to the JVM, not to Scala.

Custom exceptions can be treated like other classes, they can have class parameters, fields and methods. Usually all will need is a name and a `printStackTrace` utility method.

```Scala
class MyException extends Exception
```

### Packaging and Imports
A package is a number of definitions grouped under the same name. This almost always matches the directory structure.

Members within a package are visible using their simple name.

If accessing a member from a different package, you need to either import the package or use the full qualified name.

```Scala
import playground.Cinderella

val princess = new Cinderella

val princess = new playground.Cinderella // fully qualified class name
```

Packages are ordered hierarchically and this maps the folder structure in the filesystem: `lectures.part2oop.OOBasics`.

### Package Objects
A package object originated form the desire to write methods or constants outside of everything else, for example universal utility methods or constants.

There can only be one package object per package and its name is the same as the package. The filename is `package.scala`.

Methods and constants defined in the package object can be used by their simple name in the rest of the package.

Package object are relatively rarely used but can be quite handy.

```Scala
package object part2oop {
	def sayHello: Unit = println("Hello world")

	val SPEED_OF_LIGHT = 299792458
}
```

### Imports
Your IDE will group imports together. If this gets too long, you can use `packageName._` to import everything from that package.

You can do name aliasing during imports, which can be useful if you're importing more than one class with the same name from different packages.

If two import have the same name, the compiler will assume that you are referencing the first import.

```Scala
import playground.{PrinceCharming, Cinderella => Princess}
// or
import playground._
```

Default imports are packages that are imported without any intentional import by you.

For example: 
- java.lang - String, Object, Exception  
- scala - Int, Nothing, Function  
- scala.Predef - println, ???

### Generics
Generics are a method to use to same code to handle many (potentially unrelated) types.

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

You can have multiple type parameters, for example `class MyMap[Key, Value]`.

Traits can also be type paramaterized but objects cannot.

### Generic Methods
Methods can take a type paramater and then a concrete argument when called.

```Scala
object MyList {
	def empty[A]: MyList[A] = ???
}

val emptyInteherList = MyList.empty[Int]
```

### Variance Problem
Say you have an animal clas that is extended by Cat and Dog classes.

Does a list of Cat also extend a List of Animal?

There are 3 possible answers:
- yes - covariance (use `+` before the type parameter)
- no - invariance (use no operator with the type parameter)
- hell no - contravariance (use `-` before the type parameter)

```Scala
class Animal  
class Cat extends Animal  
class Dog extends Animal  
  
// 1. yes, List[Cat] extends List[Animal] = COVARIANCE  
class CovariantList[+A]  
val animal: Animal = new Cat  
val animalList: CovariantList[Animal] = new CovariantList[Cat]  
// animalList.add(new Dog) ??? HARD QUESTION => we return a list of Animals  
  
// 2. NO = INVARIANCE  
class InvariantList[A]  
val invariantAnimalList: InvariantList[Animal] = new InvariantList[Animal]  
  
// 3. Hell, no! CONTRAVARIANCE  
class Trainer[-A]  
val trainer: Trainer[Cat] = new Trainer[Animal]
```

### Bounded Types
Bounded types let you use your generic classes only for certain types that are either a subclass of a different type, using `<:`, or a superclass of a different type, using `>:`.

```Scala
class Cage[A <: Animal](animal: A)  
val cage = new Cage(new Dog)  
  
class Car  
// generic type needs proper bounded type - Car not subclass of Animal
//  val newCage = new Cage(new Car)
```

Bounded types let you solve one variance problem. 

You can specify that if a supertype is added to a list of subtypes, the entire list becomes a list of supertypes.

```Scala
class MyList[+A] {  
  // use the type A  
	def add[B >: A](element: B): MyList[B] = ???  
	/*  
	A = Cat 
	B = Animal 
	*/
}
```

Similarly, `Nothing` is a proper substitute for any type and so can be used for an empty collection.

```Scala
object Empty extends MyList[Nothing] {
	def head: Nothing = throw new NoSuchElementException  
	def tail: MyList[Nothing] = throw new NoSuchElementException  
	def isEmpty: Boolean = true  
	def add[B >: Nothing](element: B): MyList[B] = new Cons(element, Empty)  
	def printElements: String = ""
}
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

To get the JVM to work with functional programming, Scala's creators created the concept FunctionX.

All Scala functions are objects.

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

### Function Types
Scala supports these function types out of the box.

For example, a function that takes in an int and doubles it can be written as:

```Scala
val doubler: Int => Int = (x: Int) => 2 * x // specifying function type
// or
val doubler = (x: Int) => 2 * x // compiler infers function type

doubler(4) // 8
```

This is equivalent to:

```Scala
val doubler: Function1[Int, Int] = new Function1[Int, Int] {
	override def apply(x: Int) = 2 * x
}
```

A function that takes in two arguments of type String and concatenates them to return a value of type String (that is, an example of Function2), could be written as:

```Scala
def concatenator: (String, String) => String = new Function2[String, String, String] {  
  override def apply(a: String, b: String): String = a + b  
}  
println(concatenator("Hello ", "Scala"))
```

### Anonymous Functions
Instantiating a function is still tied to object-oriented programming. The syntax for anonymous functions (lambdas) lets you be more concise and functional.

Anonymous functions consist of `paramaters => expression`, the return type is always inferred. 

If you specify the funtion type, you don't need to specify the parameter and return types (since you have already).

```Scala
val doubler = new Function1[Int, Int] {
	override def apply(x: Int) = x * 2
}

// can be written as 
val doubler = (x: Int) => Int = x * 2

// If you specify the function type, you don't need to specify the parameter and return types
val doubler: Int => Int = x => x * 2
```

If you have multiple parameters in a lambda, you have to put them between parentheses.

```Scala
val adder = (a: Int, b: Int) => a + b

val adder: (Int, Int) => Int = (a: Int, b: Int) => a + b
```

If you don't have any parameters, you can just use `() =>`.

```Scala
val justDoSomething: () => Int = () => 3

println(justDoSomething) // prints the function instance
println(justDoSomething()) // calls the function and prints the return value
```

You can write lambdas with `{}`

```Scala
val stringToInt = { (str: String) =>
	str.toInt
}
```

You can use `_` to stand in for a parameter. This can be very useful when you're chaining higher order function calls.

Each undescore stands for a different parameter so you can't use an underscore for the same parameter multiple times.

The `_` is extremely contextual and you will need to tell the compiler which type to expect.

```Scala
val niceIncrementer = Int => Int = _ + 1 // equivalent to x => x + 1

val niceAdder: (Int, Int) => Int = _ + _ // equivalent to (a,b) => a + b

// can't replace this with underscores
// elem used more than once
listOfIntegers.flatMap(elem => new Cons(elem, new Cons(elem + 1, Empty))).toString
```

### Higher-Order Functions
Functions that either take functions as parameters or return functions as results are called higher-order functions (HOFs).

```Scala
val superFunction: (Int, (String, (Int => Boolean)) => Int) => (Int => Int) = ???
```

This `superAdder` function is a curried function, meaning it can be called with multiple parameter lists. This is because it receives a parameter and returns another function which also receives a parameter. 

```Scala
// what is actually going on
val superAdder: Function1[Int, Function1[Int, Int]] = new Function1[Int, Function1[Int, Int]] {  
  override def apply(x: Int): Function1[Int, Int] = new Function1[Int, Int] {  
    override def apply(y: Int): Int = x + y  
 }  
}

// written functionally
val superAdder: Int => (Int => Int) = (x: Int) => (y: Int) => x + y

println(superAdder(3)(4)) // curried function
```

You can define a function with multiple parameter lists to act like curried functions.

You need to specify their function type.

```Scala
def curriedFormatter(c: String)(x: Double): String = c.format(x)

val standardFormat: (Double => String) = curriedFormatter("%4.2f")
val preciseFormat: (Double => String) = curriedFormatter("%10.0f")

println(standardFormat(Math.PI)) // 3.14
println(preciseFormat(Math.PI)) // 3.14159265
```

### map, flatMap and filter
Scala Lists have several useful methods, including: `map`, `flatMap` and `filter`.

The `map` method takes an anonymous function (lambda) as its argument and this function is applied to every element in the List.

The `flatMap` method concatenates any Lists produced by the anonymous function and raturns a bigger list.

The `filter` method takes an anonymous function and returns only those elements for which the condition evaluates as true. 

You can chain, ``map``, ``flatMap`` and ``filter``.

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

You may also see this syntax:

```Scala
val list = List(1,2,3)

list.map { x =>
	x * 2
}
```

`foreach` is similar to `map` except that it receives a function returning Unit.

```Scala
val list = List(1,2,3)

list.foreach(println)
```

### For Comprehensions
Instead of using chains of higher-order functions, you can use a for comprehension, which can be reduced to a single value. The compiler will interpret it as a chain.

For comprehensions are more readable and are preferred in practice.

```Scala
// chain
val allPairs = List(1,2,3).flatMap(number => List('a','b','c').map(letter => s"$number-$letter"))

// for comprehension
val alternativePairs = for {
	number <- List(1,2,3)
	letter <- List('a', 'b', 'c')
}   yield s"$number-$letter" 
```

If you want to include filtering in a for comprehension, you include a guard.

```Scala
val filteredPairs = for {
	number <- List(1,2,3) if number % 2 == 0
	letter <- List('a', 'b', 'c')
}   yield s"$number-$letter" 
```

You can still perform side effects with for comprehensions.

```Scala
// equivalent to using foreach
for {
	n <- numbers
} println(n)
```

## Collections

Scala offers both mutable and immutable colections. The standard library has type definitions for immutable collections so you are using the immutable versions by default.

### Immutable Collections
Immutable collections are found in the `scala.collections.immutable` package.

They extend in a hierarchy starting with `Traversable`, which offers methods for mapping, converting, getting size information, testing elements, folds, retrieval and string operations.

`Traversable` is extended by `Iterable`, which is extended by the major collections types `Set`, `Seq` and `Map`.

`Set` is extended by `HashSet` and `SortedSet`.

`Map` is extended by `HashMap` and `SortedMap`.

`Seq` is extended by `IndexedSeq` and `LinearSeq`. 

### Mutable Collections
Mutable collections are found in the `scala.collections.mutable` package.

Their hierarchy closely mirrors immutable collections but the implementations of  `Set`, `Seq` and `Map` differ slightly.

`Set` is extended by `HashSet` and `LinkedHashSet`.

`Map` is extended by `HashMap` and `MultiMap`.

`Seq` is extended by `IndexedSeq`, `Buffer` and `LinearSeq`. 

There are also ower level implementations:

`IndexedSeq` is extended by `StringBuilder` and `ArrayBuffer`.

`Buffer` is extended by `ArrayBuffer` and `ListBuffer`.

`LinearSeq` is extended by `LinkedList` and `MutableList`.

### Lists
Lists are the fundamental collection in functional programming. They are immutable and extend `LinearSeq`.

A list has a head (the first element) and a tail (the remainder of the list).

The `head`, `tail` and `isEmpty` methods are fast: O(1). Most other operations (`length`, `reverse`) are O(n).

Lists are sealed and have two subtypes:
- object `Nil` (empty list)
- class `::`

Lists can be prepended and appended with elements:

`+:` prepends an element
`:+` appends an element

```Scala
val aList = List(1,2,3,4,5)
val firstElement = aList.head
val rest = aList.tail
val aPrependedList = 0 :: aList // :: prepends a value
val anExtendedList = 0 +: aList :+ 6 // +: prepends, :+ appends
```

If you want to create a list where every element has the same value, you can use a curried function `fill`: 

```Scala
val fiveApples = List.fill(5)("apple") // list of 5 "apple" strings
```

The `mkString` method lets you pretty print a list. It concatenates all the values and adds a seperator, if specified.

```Scala
println(aList.mkString("-"))
```

### Sequences
Sequences are very general interfaces for data structures which you can traverse in a given order and where you can access an element at a given index.

Sequences support concatenating, appending and prepending.

`Seq` is a Trait. The `Seq` companion object has an `apply` factor method that can construct subclasses of Sequence.

```Scala
val aSequence = Seq(1,2,3,4) // Seq.apply(1,2,3,4)
val accessedElement = aSequence(1) // the element at that index = 2
val aLongerSequence = aSequence ++ Seq(7,6,5) 
val aSortedSequence = aLongerSequence.sorted // List(1,2,3,4,5,6,7)
```

Indexed Sequences have the property that they can be quickly accessed (in constant time): `Vector`, `String`, `Range`.

Linear Sequences don't have constant time access and only guarantee the ordering of elements: `List`, `Stream`, `Stack`, `Queue`.

### Arrays
Arrays in Scala are the equivalent of simple Java arrays.

They can be manually constructed with predefined lengths, they are mutable, have fast indexing and are completely interoperablw with Java's native arrays.

You can create an array without specifying contents using `Array.ofDim[type](numberOfElements)`. The elements will be set to their default values: 0, false, null.

```Scala
val numbers = Array(1,2,3,4)
val threeElements = Array.ofDim[Int](3)
```

Since arrays are mutable, their values can be updated.
```Scala
val numbers = Array(1,2,3,4)
numbers(2) = 0 // syntactical sugar for numbers.update(2,0)
```

Arrays can be converted to sequences using implicit conversion. This is how arrays have access to sequence methods like `mkString`.

```Scala
val numbersSeq: Seq[Int] = numbers
print(numbersSeq) // WrappedArray(1,2,3,4)
```

### Vectors
Vectors are indexed, immutable sequences with very fast access times (effectively constant) and the same methods as Lists and Sequences. They are the default implementation for immutable sequences.

```Scala
val aVector:Vector[Int] = Vector(1,2,3,4,5)
```

Vectors are efficient for large collections.

```Scala
// vectors vs lists
val maxRuns = 1000  
val maxCapacity = 1000000

def getWriteTime(collection: Seq[Int]): Double = {  
  val r = new Random  
 val times = for {  
    it <- 1 to maxRuns  
 } yield {  
    val currentTime = System.nanoTime()  
    collection.updated(r.nextInt(maxCapacity), r.nextInt())  
    System.nanoTime() - currentTime  
 }  
  
  times.sum * 1.0 / maxRuns  
}  
  
val numbersList = (1 to maxCapacity).toList  
val numbersVector = (1 to maxCapacity).toVector

// keeps reference to tail  
// updating an element in the middle takes a long time
println(getWriteTime(numbersList))  
// depth of the tree is small  
// needs to replace an entire 32-element chunk  
println(getWriteTime(numbersVector))
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
Tuples are a simple "list-like" container for grouping disparate items. They are finite and ordered. 

Tuples can at most group 22 elements, since they are used in conjunctiuon with function types.

You can access specific elements using `_n` (this isn't zero-indexed).

```Scala
val aTuple = (2, "Hello Scala") // equivalent to Tuple2[Int, String]

println(aTuple._1) // access the first element
println(aTuple.copy(_2 = "goodbye Java"))  // replace second element in copy
println(aTuple.swap)  // ("hello, Scala", 2)
```

### Maps
A Map is an iterable sequence that consists of pairs of keys and values. 

You populate a map with tuples (pairings), which can also be written as `kay -> value`.

Trying to access an element that doesn't exist will crash your programme. You can protect against this either by using the `withDefaultValue` method to set a default value or by throwing an exception.
```Scala
val aPhoneBook: Map[String, Int] = Map(
	("Daniel", 034937846),
	"Jane" -> 187327063 // equivalent to ("Jane", 187327063)
).withDefaultValue(-1)

println(aPhonebook.contains("Jane"))  
println(aPhonebook("Mary")) // not in map
```

Maps are immutable, so to add a value you need to create a new map.

```Scala
val aNewPairing = "Mary" -> 123456789
val aNewPhonebook = aPhoneBook + aNewPairing
```

Maps have access to `map`, `flatMap` and `filter`. These apply to pairings.

You can operate on keys using `filterKeys` or values using `mapValues`.

```Scala
println(phonebook.map(pair => pair._1.toLowerCase -> pair._2))

println(phonebook.view.filterKeys(x => x.startsWith("J")).toMap)

println(phonebook.view.mapValues(number => "0245-" + number).toMap)
```

## Pattern Matching

Pattern matching is a mechanism for checking a value against a pattern. It's comparable to a switch statement in Java and can be used instead of a series of if/else statements. Think of it as a switch expression.

Pattern matching will try all cases in sequence.

`_` serves as the catch all case (wildcard). Cases are also called alternatives.

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

You can add a guard to test the decomposed values against a condition.

```Scala
case class Person(name: String, age: Int)
val bob = Person("Bob", 20) // equivalent to Person.apply("Bob", 43)

val personGreeting = bob match {  
	case Person(n, a) if a < 21 => s"Hi, my name is $n. I can't drink in the US"
	case Person(n, a) => s"Hi, my name is $n and I am $a years old." 
	case _ => "Something else"  
}
```

1. Cases are matched in order  
2. If no cases match, you get a `MatchError`  (so use a wildcard)
3. The type of the PM expression is the unified type of all the types in all the cases  
4. Pattern Matching works really well with case classes*

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

If a pattern match doesn't match anything it'll throw a `MatchError` so it's good practice to include a catch all case. 

Catch blocks and generators are also based on pattern matching.

```Scala
// catches are actually matches
try {  
  // do something 
} catch {  
  case e: RuntimeException => "runtime"  
	case npe: NullPointerException => "npe"  
	case _ => "something else"  
}  

// equivalent to 
try {  
  // do something 
} catch (e) {  
  e match {  
    case e: RuntimeException => "runtime"  
		case npe: NullPointerException => "npe"  
		case _ => "something else"  
	 }  
}

// generators are also based on pattern matching 
val tuples = List((1,2), (3,4))  
val filterTuples = for {  
  (first, second) <- tuples  
} yield first * second
// similaryly, case classes, ::
```

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
`Option` and `Try` are useful when you have unsafe methods, which might cayse a `NullPointerException`, saving you from having to check for the method returning a Null value.

### Option
An Option is like a "collection" containing at most one element. It acts as a wrapper for a value that might or might not be present.

For example, you can pass a method which can return Null to Option. If the method returns a valid value, then the "collection" contains that value. It'll return ``Some``, a subtype of Option.

If the method returns Null, the value will be ``None`` (a singleton object and still a valid value).

Lots of methods on collections are designed to work with Options.

Safe APIs are designed to return Options so the user of the API doesn't need to wrap the result in an Option.

```Scala
def methodWhichCanReturnNull(): String = "Hello, Scala"  
val anOption = Option(methodWhichCanReturnNull()) // Some("Hello, Scala")  

// option = "collection" which contains at most one element: Some(value) or None
val stringProcessing = anOption match {  
	case Some(string) => s"I have obtained a valid string: $string"  
	case None => "I obtained nothing"  
}

// Unsafe API method
def unsafeMethod(): String = null

// Safer way to write API methods
def saferMethod(): Option[String] = None
def backupMethod(): Option[String] = Some("A valid result")

val saferResult = saferMethod() orElse backupMethod()

// collections use Options
val map = Map("key" -> "value")
map.get("key") // Some(value)
map.get("other") // None

val list = List(1,2,3)
list.headOption // Some(1)
list.find(_ % 2 == 0) // Some(2)
```

Options have methods such as `isEmpty` and `get`.

`get` is an unsafe way to retrieve a value from an Option. In general, don't use it.

You can used Options with pattern matching and operate on them with ``map``, ``flatMap``, ``filter`` and for comprehensions.

```Scala
// using chained functions
config.get("host")
	.flatMap(host => config.get("port")
	.flatMap(port => Connection(host, port))
	.map(connection => connection.connect))
	.foreach(println)

// using for comprehension
val forConnectionStatus = for {
	host <- config.get("host")
	port <- config.get("port")
	connection <- Connection(host, port)
} yield connection.connect
forConnectionStatus.foreach.println()
```

### Try
A Try is a wrapper for a computation that might or might not fail.

A Try guards against methods which can throw exceptions, avoiding having to write defensive try/catch blocks and preventing runtime crashes.

Multiple nested try/catch blocks make the code hard to follow and you can't chain multiple operations that are prone to failure.

A Try is essentially a "collection" with either a value if the code went well, or an exception if the code threw one. The two subtypes of Try are `Success` and `Failure`.

`Failure`  holds the throwable that would have been thrown while `Success` wraps the value that the successful computation returned.

```Scala
sealed abstract class Try[+T]
case class Failure[+T](t: Throwable) extends Try[T]
case class Success[+T](value: T) extends Try[T]
```

The Try object can also be processed with map, flatMap and filter.

```Scala
def methodWhichCanThrowException(): String = throw new RuntimeException  
val aTry = Try(methodWhichCanThrowException())  

val anotherStringProcessing = aTry match {  
	case Success(validValue) => s"I have obtained a valid string: $validValue"  
	case Failure(ex) => s"I have obtained an exception: $ex"  
}
```

A Try can be used similar to an Option to make unsafe methods safer.

If your code might return null, use an Option. 

If your code might throw exceptions, use a Try.

```Scala
def betterUnsafeMethod(): Try[String] = Failure(new RuntimeException)
def betterBackupMethod(): Try[String] = Success("A valid result")
val betterFallback = betterUnsafeMethod() orElse betterBackupMethod()
```

You can use ``map``, ``flatMap``, `filter` and for comprehension with a Try.

### Asynchronous Programming
Asynchronous programming in Scala is done with another pseudo-collection, `Future`. 

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

implicit val timeout = 3000  
def setTimeout(f: () => Unit)(implicit timeout: Int) = f()  
  
setTimeout(() => println("timeout"))// extra parameter list omitted
```

You use implicit conversions to add methods to existing types over which you don't have control.

For example, you can give an Int an ``isEven`` method using implicit conversion.

This is dangerous. Use it carefully.

```Scala
// implicit defs  
case class Person(name: String) {  
  def greet = s"Hi, my name is $name"  
}  
  
implicit def fromStringToPerson(string: String): Person = Person(string)  
"Peter".greet  
// fromStringToPerson("Peter").greet - automatically by the compiler

implicit class MyRichInteger(n: Int) {  
	def isEven() = n % 2 == 0  
}  
  
println(23.isEven()) // equivalent to new MyRichInteger(23).isEven()
```

Implicits need to be organized carefully.

You can decide how the compiler fetches an implicit value for you. The compiler first looks in local scope, then imported scope, then the companion objects of the types included in the call.

```Scala
// local scope
// collections sorted method takes an implicit parameter. 
implicit val inverseOrdering: Ordering[Int] = Ordering.fromLessThan(_ > _)  
List(1,2,3).sorted // List(3,2,1)

// imported scope
import scala.concurrent.ExecutionContext.Implicits.global  
val future = Future {  
  println("hello, future")  
}

// companion objects of the types included in the call  
object Person {  
  implicit val personOrdering: Ordering[Person] = Ordering.fromLessThan((a, b) => a.name.compareTo(b.name) < 0)  
}
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

### Braceless Syntax (Scala 3)
Scala 3 introduced an alternative braceless syntax based on significant indentation (similar to Python).

An if expression can now be written as any of the following:

```Scala
// oneliner
val anIfExpression = if (2 > 3) "bigger" else "smaller"  
  
// java-style  
val anIfExpression_v2 =  
	if (2 > 3) {  
    "bigger"  
 } else {  
    "smaller"  
 }  
  
// compact  
val anIfExpression_v3 =  
	if (2 > 3) "bigger"  
	else "smaller"  
  
// scala 3  
val anIfExpression_v4 =  
	if 2 > 3 then  
		"bigger" // higher indentation than the if part  
	else  
		"smaller"  

// scala 3 - more complex
val anIfExpression_v5 =  
	if 2 > 3 then  // anything after then is treated as a code block
		val result = "bigger"  
		result  
	else  
		val result = "smaller"  
		result  
  
// scala 3 one-liner  
val anIfExpression_v6 = if 2 > 3 then "bigger" else "smaller"
```

For comprehensions can be written as:

```Scala
// scala 2 
val aForComprehension = for {  
  n <- List(1,2,3)  
  s <- List("black", "white")  
} yield s"$n$s"  
  
// scala 3  
val aForComprehension_v2 =  
	for  
		n <- List(1,2,3)  
    s <- List("black", "white")  
  yield s"$n$s"
```

Pattern matching can be written as:

```Scala
// scala 2
val meaningOfLife = 42  
val aPatternMatch = meaningOfLife match {  
	case 1 => "the one"  
	case 2 => "double or nothing"  
	case _ => "something else"  
}  
  
// scala 3  
val aPatternMatch_v2 =  
	meaningOfLife match  
		case 1 => "the one"  
		case 2 => "double or nothing"  
		case _ => "something else"
```

You can define methods without braces. Everything at the same indent level will be treated as belonging to the same code block. Be careful with this.

```Scala
def computeMeaningOfLife(arg: Int): Int = // significant indentation starts here - think of it like a phantom code block  
	val partialResult = 40



	partialResult + 2 // sill in the same phantom code block
```

You can also define classes, traits, objects, enums and data types with significant indentation.

You need to add `:` to a class definition to indicate to the compiler that you're going to start an indentation region.

Methods and fields need to start at the same indentation.

To clarify where a class (or method, match, if expression etc.) ends you can add an `end` token.

You should probably add an `end` token to any method with more than 4 lines of code and for a class, trait or object that doesn't fit entirely on the screen.

```Scala
// class definition with significant indentation (same for traits, objects, enums etc)  
class Animal: // compiler expects the body of Animal  
	def eat(): Unit =  
		println("I'm eating")  
  end eat // optional end token for clarity
  
	def grow(): Unit =  
		println("I'm growing")  
  
  // 3000 more lines of code  
end Animal // if, match, for, methods, classes, traits, enums, objects

// anonymous classes  
val aSpecialAnimal = new Animal:  
	override def eat(): Unit = println("I'm special")
```

Don't mix tabs and spaces if using significant indentation.

### Partial Functions
Partial functions are functions that operate only on a subset of the given input domain.

Partial functions are based on pattern matching.

This function only operates on the values 1, 2 and 5 and will throw an exception for anything else.
```Scala
val partialFunction: PartialFunction[Int, Int] = {
	case 1 => 42
	case 2 => 65
	case 5 => 999
}

// equivalent to 
val pf = (x: Int) => x match {
	case 1 => 42
	case 2 => 65
	case 5 => 999
}

val function: (Int => Int) = partialFunction
```

Partial functions can operate on collections as well.

```Scala
val modifiedList = List(1,2,3).map {  
	case 1 => 42  
	case _ => 0  
}
```

Partial functions are useful for lifting.

```Scala
val lifted = partialFunction.lift // total function Int => Option[Int]  
lifted(2) // Some(65)  
lifted(5000) // None
```

`orElse` can be used to chain partial functions.

```Scala
val pfChain = partialFunction.orElse[Int, Int] {  
  case 60 => 9000  
}  
  
pfChain(5) // 999 per original partialFunction  
pfChain(60) // 9000 per additional partial function
pfChain(457) // throws a MatchError
```

### Type Aliases
A type alias is usually used to simplify declaration for complex types, such as parameterized types or function types.

```Scala
type ReceiveFunction = PartialFunction[Any, Unit]

def receive: ReceiveFunction = {  
  case 1 => println("hello")  
  case _ => println("confused....")  
}
```

