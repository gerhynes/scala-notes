# Scala Notes
Sources:
- [The Scala Documentation](https://docs.scala-lang.org/)
- [Scala & Functional Programming Essentials](https://www.udemy.com/course/rock-the-jvm-scala-for-beginners/)
- [Get Programming with Scala](https://www.manning.com/books/get-programming-with-scala)

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

The JVMs internal stack has limited memory and a `StackOverflowError` will happen when the recursive depth is exceeded.

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

You can avoid a `StackOverFlowError` using tail recursion. Scala doesn't need to save intermediate results for use later, it uses an accumulator. This lets Scala preserve the same stack frame and not use additional stack frames for recursive calls.

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

In the example below, at compile time, the compiler only knows you're calling the method from an animal object. At runtime it will be called from a dog object if the `Dog` class overrides this method inherited from `Animal`.

The most derived method will be called at runtime.

```Scala
val aDeclaredAnimal: Animal = new Dog("Hachi")
aDeclaredAnimal.eat() // the most derived method will be called at runtime
```

### Abstract Classes
Scala has abstract classes, where all fields and methods don't necessarily need to have implementation. Abstract classes can have both abstract and non-abstract members.

Whatever class implements the abstract class will need to provide an implementation of the abstract methods.

Abstract classes cannot be instantiated, they are made to be extended later.

```Scala
abstract class WalkingAnimal {
	protected val hasLegs = true // by default public
	def walk(): Unit // to be implemented in concrete class
}
```

You will use traits more often than abstract classes. Abstract classes are useful if you need a base class that requires constructor arguments or your Scala code will be called from Java code (which doesn't have traits).

### Traits
Scala also has traits, for the ultimate abstract type, where everything is usually unimplemented. 

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

If a trait has only concrete methods, you can mix it into a class on the fly.

```Scala
trait TailWagger {
	def startTail(): Unit = println("tail is wagging")
	def stopTail(): Unit = println("tail is stopped")
}

trait Runner {
	def startRunning(): Unit = println("I'm running")
	def stopRunning(): Unit = println("Stopped running")
}

class Dog(name: String)

val dog = new Dog("Fido") with TailWagger with Runner
```

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

Operators in Scala are actually methods. For example, the `+` operator is actually a method belonging to the Int type and equivant to `.+()`.

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

### Companion Objects
You can define a class (or trait) and a singleton object in the same file with the same name. In this instance, the class and object are called companions.

Companions can access each other's private fields and methods. 

But the `Animal` singleton object and instances of the `Animal` type are different things.

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

The ability to create new class instances without using the `new` keyword comes from the `apply` method in the companion object.

```Scala
class Person {
	var name = ""
}

object Person {
	def apply(name: String): Person = {
		var p = new Person
		p.name = name
		p
	}
}
```

When you use something like `val fred = Person("Fred")`, the compiler sees that there's no `new` keyword and looks for a companion object with an `apply` method. If it finds a companion object, it uses it. Otherwise, it throws a compilation error.

You can give a companion object multiple `apply` methods with different parameters to provide multiple constructors.

```Scala
class Person {
    var name: Option[String] = None
    var age: Option[Int] = None
    override def toString = s"$name, $age"
}

object Person {
    // a one-arg constructor
    def apply(name: Option[String]): Person = {
        var p = new Person
        p.name = name
        p
    }

    // a two-arg constructor
    def apply(name: Option[String], age: Option[Int]): Person = {
        var p = new Person
        p.name = name
        p.age = age
        p
    }
}
```

Just as an `apply` method in a companion object lets you construct new class instances, an `unapply` method (also called an extractor method) lets you deconstruct instances.

```Scala
class Person(var name: String, var age: String)

object Person {
	def unapply(p: Person): String = s"${p.name}, ${p.age}"
}

val p = new Person("Lori", 29)

val details = Person.unapply(p) // "Lori, 29"
```

`unapply` can return any type.

```Scala
object Person {
	def unapply(p: Person): Tuple2[String, Int] = (p.name, p.age)
}

val (name, age) = Person.unapply(p)
```

You get `apply` and `unapply` methods for free when you create case classes rather than regular classes.

### Case Classes
Case classes are lighweight data structures with little boilerplate. They let you define a class, a companion object and a lot of sensible default in one go.

When you define a case class, the compiler automatically generates: 
- sensible `equals` and `hashcode` methods (for inclusion into various collections that rely on equality).
- sensible and quick serialization
- a companion object with `apply`
- pattern matching

With case classes:
- class parameters are promoted to fields
- a sensible `toString` method is provided
- printing an instance automatically delegates to `toString` 
- the `copy` method (which can take named parameters) creates a new instance

Case classes are serializable which is useful when dealing with distributed systems. This is heavily used by Akka.

Case classes have extractor patterns and can be used in pattern matching. The biggest advantage of case classes is that they support pattern matching, since this is a major feature of functional programming.

```Scala
case class Person(name: String, age: Int)

// new keyword can be ommitted because of companion
val bob = Person("Bob", 54) // equivalent to Person.apply("Bob, 54")
println(bob.name)
println(bob) // Person(Bob, 54)
```

Since case class constructor parameters are `val` fields by default, accessor methods are generated but mutator methods are not (since you don't mutate data structures in functional programming).

```Scala
val christina = Person("Christina", "niece")
christina.name // Christina
christina.name = "Tina" // error: reassignment to val
```

Because case classes have a built-in `unapply` method, you can use pattern matching expressions with them. `unapply` returns the case class constructor fields in a tuple wrapped in an `Option`.

```Scala
trait Person {
	def name: String
}

case class Student(name: String, year: Int) extends Person
case class Teacher(name: String, speciality: String) extends Person

def getPrintableString(p: Person): String = match {
	case Student(name, year) => 
		s"$name is a student in $year"
	case Teacher(name, subjectTaught) => 
		s"$name teaches $subjectTaught"
}
```

When pattern matching, you can bind an entire class instance to a value by providing a name and `@`.

```Scala
case class Person(Name: String, age: Int)

def welcome(person: Person): String = person match {
	case p @ Person(_, 18) => s"${p.name}, you look old!"
	case Person(_,_) => "Hey there!"
}
```

The automatically generated `copy` method is helpful when you need to (i) clone an object and (ii) update one or more fields during the cloning process. 

Since you don't mutate data structures in functional programming, this is how you create a new instance of a class from an existing instance, called "update as you copy".

```Scala
case class BaseballTeam(name:String, lastWorldSeriesWin: Int)

val cubs 1908 = BaseballTeam("Chicago Cubs", 1908)

val cubs2016 = cubs1908.copy(lastWorldSeriesWin = 2016)
```

### Case Objects
A case object is like a regular object but with more features.

- it's serializable
- it has a default `hashCode` implementation
- it has an improved `toString` implementation

Case objects don't get companion objects since they are their own companion object.

Case objects are primarily used instead of regular objects 
- when creating enumerations (in Scala 2)
- when creating containers for "messages" that you want to pass between objects (such as with Akka actors)

This is how you create an enumeration with case objects.

```Scala
sealed trait Topping
case object Cheese extends Topping
case object Pepperoni extends Topping
case object Sausage extends Topping
case object Mushrooms extends Topping
case object Onions extends Topping
```

Case objects are also handy when you want to model the concept of a "message", and pass them around in a safe way. For example, with a digital voice assistant: "speak enclosed text", "pause", "resume".

```Scala
case class StartSpeakingMessage(textToSpeak: String)
case object PauseSpeakingMessage
case object ResumeSpeakingMessage
case object StopSpeakingMessage
```

Note that `StartSpeakingMessage` is a case class rather than a case object because a case object doesn't take constructor parameters.

So in an Akka project, a `Speak` class might look like this.

```Scala
class Speak extends Actor {
	def receive = {
		case StartSpeakingMessage(textToSpeak) => 
			// code to speak the text
		case PauseSpeakingMessage =>
			// code to pause speaking
		case ResumeSpeakingMessage =>
			// code to resume speaking
		case StopSpeakingMessage =>
			// code to stop speaking
	}
}
```

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

Exceptions are treated as special ojects by the JVM and are handled using a `try/catch` block, with an optional `finally` clause that will execute no matter what. 

The `finally` clause is used for side effects, for example closing connections or releasing resources that would otherwise be dangerous to leave open.

Like everything in Scala, a `try` `catch` `finally` block is an expression. The compiler will attempt to unify the types of values returned and will use  `AnyVal` if the values returned from the try and catch blocks are different.

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

Custom exceptions can be treated like other classes: they can have class parameters, fields and methods. Usually all they will need is a name and a `printStackTrace` utility method.

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

If two imports have the same name, the compiler will assume that you are referencing the first import.

```Scala
import playground.{PrinceCharming, Cinderella => Princess}
// or
import playground._
```

Default imports are packages that are imported without any intentional import by you.

For example: 
- java.lang - `String`, `Object`, `Exception`  
- scala - `Int`, `Nothing`, `Function`  
- scala.Predef - `println`, `???`

### Generics
Generics are a method to use the same code to handle many (potentially unrelated) types.

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
Say you have an `Animal` class that is extended by `Cat` and `Dog` classes.

Does a list of `Cat` also extend a List of `Animal`?

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
val aReversedList = aList.reverse // returns a new list
```

## Object-Oriented vs Functional Programming
Scala is marketed as a mix of object-oriented and functional programming. It is actually closer to object-oriented. All the code and values you operate with are inside an instance of some type. There are no methods outside a class/object.

### Extending App with Objects
When you define an object that extends `App` you are inheriting from the `App` type's `main` method.

Methods on objects are similar to Java static methods. So you already have a static main method implemented by extending `App`. This is what makes them runnable.

```Scala
object ObjectOrientation extends App {...}
```

## Functional Programming
Functional programming is a style of programming that emphasises writing applications using only pure functions and immutable values, in other words code as maths with the functions as a series of algebraic equations.

A **pure function** has these characteristics:
- the function's output depends only on its input values (it has referential transparency)
- it doesn't mutate any hidden state
- it doesn't have any back doors to read or write data with the outside world

A pure function is total and has no side effects. A function is total if it is well-defined for every input: it must terminate for every parameter value and return an instance that matches its return type. A side effect is an operation that has an observable interaction with elements outside its local scope. It affects, or is affected by, the state of the application by interacting with the outside world. 

Any method that returns `Unit`, such as `foreach`, is going to be an impure function because it is being called for its side effects. Throwing an exception is not a side effect because it doesn't change the state of components external to the function.

An inpure function does one or more of these things:
- reads hidden inputs
- writes hidden outputs
- mutates the parameters its given
- performs I/O with the outside world

Write the "core" of your aplication in pure functions and write impure wrappers around it to interact with the outside world.
 
Scala is an object-oriented language and runs on the JVM. The JVM knows what an object is but doesn't know what a function is as a first class citizen.

In functional programming the goal is to work with functions as with any other kind of value:
- compose functions
- pass functions as arguments
- return functions as results

To get the JVM to work with functional programming, Scala's creators created the concept `FunctionX`.

All Scala functions are objects.

When you create an object that can be invoked like a function (because it has an apply method), and the only thing it supports is to be invoked like a function, you've essentially defined a function. It acts like a function and the only thing it can do is act like a function.

All Scala functions are instances of these `FunctionX` types.

`FunctionX` = `Function1`, `Function2` ... `Function22`. 

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

For example, a function that takes in an Int and doubles it can be written as:

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

A function that takes in two arguments of type String and concatenates them to return a value of type String (that is, an example of `Function2`), could be written as:

```Scala
def concatenator: (String, String) => String = new Function2[String, String, String] {  
  override def apply(a: String, b: String): String = a + b  
}  
println(concatenator("Hello ", "Scala"))
```

### Anonymous Functions
Instantiating a function is still tied to object-oriented programming but the syntax for anonymous functions (lambdas) lets you be more concise and functional.

Anonymous functions consist of `paramaters => expression`, the return type is always inferred. 

If you specify the function type, you don't need to specify the parameter and return types (since you have already).

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

Each underscore stands for a different parameter so you can't use an underscore for the same parameter multiple times.

The `_` is extremely contextual and you will need to tell the compiler which type to expect.

```Scala
val niceIncrementer = Int => Int = _ + 1 // equivalent to x => x + 1

val niceAdder: (Int, Int) => Int = _ + _ // equivalent to (a,b) => a + b

// can't replace this with underscores
// elem used more than once
listOfIntegers.flatMap(elem => new Cons(elem, new Cons(elem + 1, Empty))).toString
```

In other words, all three of these functions do the same thing:

```Scala
val doubledInts = ints.map((i:Int) => i * 2)
val doubledInts = ints.map(i => i * 2)
val doubledInts = ints.map(_ * 2)
```

For example, on a `List[Int` the `map` method expects a function that transforms one Int into another Int. Similarly, when called on a `List[Int`, the `filter` method expects a function that takes an Int and returns a Boolean.

```Scala
def double(i: Int): Int = i * 2
val doubledInts = ints.map(double)

def lessThanFive(i: Int): Boolean = (i < 5)
val filteredInts = ints.filter(lessThanFive) 
// same as int.filter(_ < 5)
```

### Higher-Order Functions
Scala lets you create functions as variables, which means you can pass functions as parameters to other functions. Functions that either take functions as parameters or return functions as results are called higher-order functions (HOFs).

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

Higher-order functions let you abstract functionality and reduce code duplication.

```Scala
def stats(s: String, predicate: Char => Boolean): Int = 			
	s.count(predicate)

def size(s: String): Int = stats(s, _ => true)
def countLetters(s: String): Int = stats(s, _.isLetter)
def countDigits(s: String): Int = stats(s, _.isDigit)

// Create custom statisitcs by calling stats directly
val text = "This is some sample text"
stats(text, _.isUpper) // count uppercase letters
stats(text, _ == 'x') // count instances of character 'x'
```

You can provide default parameters and then call the function without the `predicate` argument.

```Scala
def stats(s: String, predicate: Char => Boolean = { _ => true }): Int = s.count(predicate)

stats(s) // count all characters
```

Higher-order functions can also return functions. For example, you could use a `predicateSelector` function to insert the appropriate `predicate` into `stats`.

```Scala
sealed trait Mode
case object Length extends Mode
case object Letter extends Mode
case object Digits extends Mode

def predicateSelector(mode: Mode): Char => Boolean = mode match {
	case Length => _ => true
	case Letters = _.isLetter
	case Digits = _.isDigit
}

stats(text, predicateSelector(Length)) // count all chars
stats(text, predicateSelector(Letters)) // count all letters
```

### map, flatMap and filter
Scala Lists (and other sequences) have several useful methods, including: `map`, `flatMap` and `filter`.

The `map` method takes an anonymous function (lambda) as its argument and this function is applied to every element in the List.

The `flatMap` method concatenates any Lists produced by the anonymous function and raturns a bigger list.

The `filter` method takes an anonymous function and returns only those elements for which the condition evaluates as true. 

You can chain, `map`, `flatMap` and `filter`.

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

The main Scala collections you'll use on a regular basis are:
- `ArrayBuffer` - an indexed, mutable sequence
- `List` - a linear (linked list), immutable sequence
- `Vector` - an indexed, immutable sequence
- `Map` - the base Map (key/value pairs) class
- `Set` - the base Set (no duplicates) class

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

### ArrayBuffers
An `ArrayBuffer` is similar to a Java `ArrayList`. It's mutable and has methods similar to Java sequences.

```Scala
val nums = ArrayBuffer(1, 2, 3) // ArrayBuffer(1,2,3) 

nums += 4 // ArrayBuffer(1,2,3,4) 

nums ++= List(5,6,7) // ArrayBuffer(1,2,3,4,5,6,7) 

nums -= 7 // ArrayBuffer(1,2,3,4,5,6) 

nums --= Array(5,6) // ArrayBuffer(1,2,3,4)
```

`ArrayBuffer` methods include:
- `append(value)` – add value to end 
- `appendAll(Seq(value, value))` – add sequence of values to end 
- `clear()` – remove all values 
- `insert(index, value)` – add value at index 
- `insertAll(Seq(value1, value2))` - add sequence of values at index 
- `prepend(value)` – add value to start 
- `prependAll(Seq(value1, value2))` - add sequence of values to start 
- `range(value1, value2)` - create `ArrayBuffer` populated with values between two given values 
- `dropInPlace(n)` - remove first n elements 
- `dropRightInPlace(n)` - remove last n elements

### Lists
`List`s are the fundamental collection in functional programming. They are immutable and extend `LinearSeq`.

A `List` has a head (the first element) and a tail (the remainder of the list).

The `head`, `tail` and `isEmpty` methods are fast: O(1). Most other operations (`length`, `reverse`) are O(n).

`List`s aren't ideal for accessing elements by index value, especially when larger. Use `ArrayBuffer` or `Vector` instead.

`List`s are sealed and have two subtypes:
- object `Nil` (empty list)
- class `::`

`List`s can be prepended and appended with elements:
`+:` prepends an element
`:+` appends an element

(`:` always points to the `List`)

```Scala
val aList = List(1,2,3,4,5)
val size = aList.size
val firstElement = aList.head
val rest = aList.tail
val aPrependedList = 0 :: aList // :: prepends a value
val anExtendedList = 0 +: aList :+ 6 // +: prepends, :+ appends
```

Since a `List` is a singly linked list, appending elements is relatively slow and you should prepend if possible.

You can concatenate two `List`s using `++`.

```Scala
val list1 = List(1,2,3)
val list2 = List(4,5,6)
val biggerList = list1 ++ list2 // List(1,2,3,4,5,6)
```

If you want to create a list where every element has the same value, you can use a curried function `fill`: 

```Scala
val fiveApples = List.fill(5)("apple") // List("apple", "apple", "apple", "apple", "apple")
```

The `mkString` method lets you pretty print a list. It concatenates all the values and adds a seperator, if specified.

```Scala
println(aList.mkString("-"))
```

### List Methods
You can check if a `List` is/isn't empty using `isEmpty` or `nonEmpty`.

```Scala
List(1,2).isEmpty // false
List(1.2).nonEmpty // true
```

You can enquire about a `List`'s content using `contains`, `exists` and `count`.

`contains` takes a value and returns true if it equals any of the elements in the sequence.

`exists` takes a function and returns true if any element respects the predicate.

`count` takes a function and returns the number of elements that respect the predicate.

```Scala
List().contains("scala") // false

List(1,2,3).exists(_ > 42) // false

List(1,2,3).count(_ > 1) // 2
```

You can find an element at a given index with `apply` or `headOption`

`apply` returns the element at a given index and throws an exception if the index is invalid. You can also use the syntax `listName(n)`.

`headOption` returns a nullable value containing the first element of the sequence, if any.

```Scala
List(1,2,3).apply(0) // 1
List(1,2,3)(0) // 1

List().headOption // None
list(1,2,3).headOption // Some(1)
```

`find` optionally returns the first element for which the predicate is true, if any.

```Scala
List(1,2,3).find(_ > 1) // Some(2)
List(1,2,3).find(_ > 3) // None
```

You can access the maximum and minimum values in a sequence using `max`, `min`, `maxBy` and `minBy`.

`max` will return the maximum element in a sequence with a given ordering.

`min` will return the minimum element in a sequence with a given ordering.

`maxBy` returns the maximum element according to a custom order criteria.

`minBy` returns the minimum element according to a custom order criteria.

They will all throw an `UnsupportedOperationException` on an empty `List`.

```Scala
List(1,3,2).max // 3
List(2,1,3).min // 1

case class Person(name: String, age: Int)

List(Person("Susan", 16), Person("Peter", 18)).maxBy(_.age) // Person("Peter", 18)

List(Person("Susan", 16), Person("Peter", 18)).minBy(_.name) // Person("Peter", 18)
```

You can take and drop elements from a `List` based off different criteria.

`take` creates a new `List` containing the first n items of the target `List`.

`drop` creates a new `List` without the first n items of the target `List`.

`takeWhile` creates a new `List` by selecting elements from the target `List`'s head until a given predicate is satisfied.

`dropWhile` creates a new `List` by removing elements from the target `List` until a given predicate is satisfied.

```Scala
List(0,1,2).drop(1) // List(1,2)

List(0,1,2).take(1) // List(0)

List(0,1,2).dropWhile(_ < 2) // List(2)

List(0,1,2).takeWhile(_ < 2) // List(0,1)
```

`filter` takes a function and returns a new `List` with all the elements for which the given predicate is true.

`filterNot` takes a function and returns a new `List` containing all the elements for which the given predicate is false.

```Scala
List(1,2,3).filter(_ > 2) // List(3)

List(1,2,3).filterNot(_ > 2) // List(1,2)
```

`distinct` returns a new `List` without any duplicate elements.

```Scala
List(1,2,3,3,3).distinct // List(1,2,3)
```

`sorted` returns a new `List` with the elements sorted acording to their given order.

`sortBy` returns a new `List` with the elements sorted by a specified criteria. You can combine multiple criteria using a tuple. The first criteria has precedence over the second and so on.

`reverse` returns a new `List` with the elements in reverse order.

```Scala
List(0.4, -2, 3).sorted // List(-2.0, 0.4, 3.0)

List(1,0,2).sortBy(_ * -10) // List(2,1,0)

List(1,0,2).reverse // List(2,0,1)
```

`mkString` produces a string representing the `List`, concatenated by a seperator (defaults to empty string). You can also optionally provide prefix and suffix strings for the produced `List`.

```Scala
List(0,1,2).mkString // "012"

List(0,1,2).mkString(",") // "0,1,2"

List(0,1,2).mkString("[", ",", "]") // "[0,1,2]"
```

`sum` sums the elements of a numerical `List`.

```Scala
List(0,1,2) // 3

List(1.4, 2.5, 3.6) // 7.5
```

`groupBy` returns a `Map` containing the `List` elements grouped according to a discriminator function.

```Scala
List("hello", "world", "scala").groupBy(_.contains("a")) // Map(false -> List("hello", "world"), true -> List("scala"))
										
List("hello", "world", "scala").groupBy(_.length)) // Map(5 -> List, "hello", "world", "scala")

List(0,1,2).groupBy(_ % 2) // Map(0 -> List(0,2), 1 -> List(1))
```
 
### Traversing a List
You can use pattern matching to traverse a `List`. Here `getSurnames` returns an empty `List` if there are no contacts. Otherwise, it extracts the surname of the head element and recursively calls itself on the tail.

```Scala
case class Contact(name: String, surname: String, number: String)

def getSurnames(contacts: List[Contact]): List[String] = contacts match {
	case Nil => Nil
	case head :: tail => head.surname :: getSurnames(tail)
}
```

`map` lets you iterate over every element in a `List` and apply a function to it.

```Scala
def getSurnames(contacts: List[Contact]): List[String] =
	contacts.map(contact => contact.surname)
	// alternative syntax
	contacts.map(_.surname)
```

`flatten` lets you combine two `List`s into one to produce a non-nested structure. Here `map` by itself would give you `List[List[ContactNumber]]`.

```Scala
List(List(), List(1,2), Lis(3)).flatten // List(1,2,3)

def getNumbers(contacts: List[Contact]): List[ContactNumber] =
	contacts.map(_.numbers).flatten
```

`flatMap` lets you combine `map` and `flatten`.

```Scala
def getNumbers(contacts: List[Contact]): List[ContactNumber] =
	contacts.flatMap(_.numbers)

// List(1,2,3) -> List(1,1,1,2,2,2,3,3,3)
def triple(nums: List[Int]): List[Int] =  
  nums.flatMap(num => List(num, num, num))
```

for comprehensions can be a more elegent way to access nested properties in `List`s.

```Scala
// with for comprehension
def selectByEmails(contacts: List[Contact], email: List[String]): List[Contact] = 
	for {
		contact <- contacts
		email <- emails
		if contact.email.exists(_.equalsIgnoreCase(email))
	} yield contact

// with flatMap
def selectByEmails(contacts: List[Contact], email: List[String]): List[Contact] = 
contacts.flatMap { contact =>
	emails.flatMap { email =>
		if(contact.email.exists(_.equalsIgnoreCase(email)))
			List(contact)
		else List()
	}
}
```

If you ever need to create an empty `List` of a given type, you can use `List.empty[A]` to specify a type, for example `List.empty[Int]`.

### Sequences
`Sequence`s are very general interfaces for data structures which you can traverse in a given order and where you can access an element at a given index.

`Sequence`s support concatenating, appending and prepending.

`Seq` is a Trait. The `Seq` companion object has an `apply` factor method that can construct subclasses of Sequence.

```Scala
val aSequence = Seq(1,2,3,4) // Seq.apply(1,2,3,4)
val accessedElement = aSequence(1) // the element at that index = 2
val aLongerSequence = aSequence ++ Seq(7,6,5) 
val aSortedSequence = aLongerSequence.sorted // List(1,2,3,4,5,6,7)
```

Indexed `Sequence`s have the property that they can be quickly accessed (in constant time): `Vector`, `String`, `Range`.

Linear `Sequence`s don't have constant time access and only guarantee the ordering of elements: `List`, `Stream`, `Stack`, `Queue`.

### Arrays
`Array`s in Scala are the equivalent of simple Java arrays.

They can be manually constructed with predefined lengths, they are mutable, have fast indexing and are completely interoperable with Java's native arrays.

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

`Array`s can be converted to sequences using implicit conversion. This is how arrays have access to sequence methods like `mkString`.

```Scala
val numbersSeq: Seq[Int] = numbers
print(numbersSeq) // WrappedArray(1,2,3,4)
```

### Vectors
`Vector`s are indexed, immutable sequences with very fast access times (effectively constant) and the same methods as `List`s and `Sequence`s. They are the default implementation for immutable sequences.

```Scala
val aVector:Vector[Int] = Vector(1,2,3,4,5)
```

`Vector`s are efficient for large collections. Since they aren't linked lists, you can append and prepend elements with similar speed.

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
`Set`s are collections with no duplicates. The main use of a `Set` is to test whether an element is contained in the `Set`, using the `contains` method. The order of the elements is not important.

You can add or remove elements with the `plus` and `minus` methods. This happens in constant time.

```Scala
val aSet = Set(1,2,3,4,1,2,3) // Set(1,2,3,4)
val setHas5 = aSet.contains(5) // false
val anAddedSet = aSet + 5 // Set(1,2,3,4,5)
val aRemovedSet = aSet - 3 // Set(1,2,4,5)
```

`Set`s also have `add` and `remove` methods, which return true if an element was successfully added/removed.

### Working with Sets
`map` lets you iterate over a `Set` and apply a function to every element. 

`flatten` lets you simplify nested `Set`s.

`flatMap` combines `map` and `flatten`.

You can use for comprehensions on `Set`s.

```Scala
def getIds(students: Set[Student]): Set[Int] = 
	students.map(_.id)

def getTopics(students: Set[Students]): Set[String] =
	students.map(_.topics).flatten

def getTopics(students: Set[Student]): Set[String] = 
	students.flatMap(_.topics)

deg getTopicsForStudentIIds(students: Set[Student], ids: Set[Int]): Set[String] = 
	for {
		student <- student
		id <- ids
		if student.id == id
		topic <- student.topics
	} yield topic
```

Sets mimic the concept of a mathematical set. You ccan check the similarities and differences of two sets with `union`, `intersect` and `diff`.

`union` (or `++`) takes another Set as its parameter and returns a new Set containing all the values from both Sets.

`intersect` (or `&`) takes another set as its parameter and returns a new Set containing only the values shared by both Sets.

`diff` (or `--`) takes another set as its parameter and returns a new Set containing the elements in the orginal Set that are not in the other Set.

```Scala
Set(1,0,2).union(Set(1,3)) // Set(0,1,2,3)

Set(1,0,2).intersect(Set(1,3)) // Set(1)

Set(1,0,2).diff(Set(1,3)) // Set(1)
```

Since both `List` and `Set` implement the `Iterable` interface, they have access to methods such as `exists`, `filter`, `max`, `min`, `maxBy` and `minBy`.

```Scala
def existsById(sudents: Set[Student], id: Int): Boolean = 
	students.exists(_.id == id)

def filterByTopic(students: Set[Student], topic: String): Set[Student] = 
	students.filter { student =>
		student.topics.conains(topic)
	}
	
def maxByTopic(students: Set[Student]): Student = 
	sudents.maxBy { student =>
		student.topics.size
	}
```

### Ranges
`Range`s are used for iteration. For example, the following range doesn't contain all the nunbers from 1 to 1000 but it can be processed as if it did.

```Scala
val aRange = 1 to 1000

// will generate a list of all even numbers between 2 and 2000
val twoByTwo = aRange.map(x => 2 * x).toList
```

You can use the `toList`, `toSet`, `toSeq` methods to convert between these collections.

### Tuples
`Tuple`s are a simple "list-like" container for temporarily grouping disparate items. They are finite and ordered. 

`Tuple`s can at most group 22 elements, since they are used in conjunctiuon with function types.

You can access specific elements using `_n` (this isn't zero-indexed!).

They are useful when you need to combine a few things together and a class would be too much overhead or feel artificial.

```Scala
val aTuple = (2, "Hello Scala") // equivalent to Tuple2[Int, String]

println(aTuple._1) // access the first element
println(aTuple.copy(_2 = "goodbye Java"))  // replace second element in copy
println(aTuple.swap)  // ("hello, Scala", 2)
```

You can destructure tuples and assign the values to variables. This can be useful where a method return disparate data and a class would feel like overkill. You can also discard values you're not interested in.

```Scala
def getStockInfo = {
    // other code here ...
    ("NFLX", 100.00, 101.00)  // this is a Tuple3
}

val (symbol, currentPrice, bidPrice) = getStockInfo
val (_, currentPrice, bidPrice) = getStockInfo
```

Because `Tuple`s aren't colections, they don't have methods like `map` or `filter`.

Scala automatically decomposes a `Tuple` with only one element. It also has an alternative constructor for `Tuple`'s with two elements, the arrow constructor.

```Scala
(1) // 1
1 -> 2 // (1,2)
```

### Maps
A `Map` is an iterable sequence that consists of pairs of keys and values. 

You populate a map with tuples (pairings), which can also be written as `key -> value`.

Trying to access an element that doesn't exist will crash your programme. You can protect against this either by using the `withDefaultValue` method to set a default value or by throwing an exception.

```Scala
val aPhoneBook: Map[String, Int] = Map(
	("Daniel", 034937846),
	"Jane" -> 187327063 // equivalent to ("Jane", 187327063)
).withDefaultValue(-1)

println(aPhonebook.contains("Jane"))  
println(aPhonebook("Mary")) // not in map
```

`Map`s are immutable, so to add or remove a value you need to create a new map.

```Scala
val aNewPairing = "Mary" -> 123456789
val aNewPhonebook = aPhoneBook + aNewPairing
val aSmallerPhonebook = aPhonebook - aKey
```

A `Map`s keys are unique, so if you add an entry for an existing key it's value will be replaced n the new `Map`.

```Scala
Map(1 -> "hello") + (1 -> "Scala") // Map(1 -> "Scala")
```

You can merge the entries of two `Map`s wih `++`.

```Scala
Map(1 -> "Hello") ++ Map(2 -> "Scala") // Map(1 -> "Hello", 2 -> "Scala")
```

You can remove multiple entries by using `--` and providing their keys in an iterable (usually Set or List). Removing a non-existant key just returns the original Map.

```Scala
Map("Rome" -> "Italy", "London" -> "UK") -- Set("Rome", "Paris") // Map("London" -> "UK")
```

`Map`s have access to `map`, `flatMap` and `filter`. These apply to the Map's tuples. You can't use `flatten.

```Scala
// Swap keys and values
Map("Rome" -> "Italy", "London" -> "UK").map {
	case (capital, country) => country -> capital
}

// If you transform a tuple into a type, you'll get back an iterable of that type, here List[String]
Map("Rome" -> "Italy", "London" -> "UK").map {
	case (capital, country) => country -> s"$capital is the capital of $country"
}
```

When you use `flatMap` on a `Map`, the compiler transforms it into an iterable of tuples. It will return a `Map` if the structure of the returned value allows it to, for example a List of tuples.

```Scala
def filterByStudentId(registrations: map[ExamSession, List[Sudent]], ids: List[Int]): Map[ExamSession, List[Student]] =
	registrations.flatMap { case (examSession, students) =>
		val matches = sudents.filter(student =>
			ids.contains(student.id))
		if(matches.nonEmpty) List(examSession -> matches)
		else List.empty
	}

def filterByStudentId(registrations: map[ExamSession, List[Sudent]], ids: List[Int]): Map[ExamSession, List[Student]] =
	for {
		(examSession, students) <- registrations
		matches = students.filter(student => id.contains(student.id))
		if matches.nonEmpty
	} yield examSession -> matches
```

You can operate on just keys using `filterKeys` or values using `mapValues`.

```Scala
println(phonebook.map(pair => pair._1.toLowerCase -> pair._2))

println(phonebook.view.filterKeys(x => x.startsWith("J")).toMap)

println(phonebook.view.mapValues(number => "0245-" + number).toMap)
```

You can traverse a `Map` with a for loop or using `foreach` and `match`.

```Scala
for((k,v) <- ratings) println(s"key: $k, value: $v")

ratings.foreach {
	case(movie, rating) => println(s"movie: $movie, rating: $rating") 
}
```

### Working with Maps
`get` takes a key as a parameter and returns an Option which resolves to Some if the key exists, None if it doesn't.

`getOrElse` takes a key and an expression. If the key exists, it returns its associated value. If it doesn't, it returns the value provided by the expression.

`apply` takes a key and returns the value associated with that key if present. It throws a NoSuchElement exception otherwise. 

```Scala
Map(1 -> "a", 2 -> "b").get(2) // Some(b)
Map(1 -> "a", 2 -> "b").getOrElse(2, defaultValue) // b
Map(1 -> "a", 2 -> "b").apply(2) // b
```

`keys` returns a Set containing the keys in a Map.

`values` returns an iterable containing the values in a Map.

`size` returns the number of keys in the Map.

`contains` takes a key and returns true if this key exists.

`filter` filters the entries based on some of their characteristics.

`maxBy` takes in a function and returns the first element which yields the largest value measured by the function.

### Common Sequence Methods
Common sequence methods inlcude:
- `map`
- `filter`
- `foreach`
- `head`
- `tail`
- `take`, `takeWhile`
- `drop`, `dropWhile`
- `reduce`

These work on all of the "sequence" collections, such as `Array`, `ArrayBuffer`, `List`, `Vector`. None of these methods mutate the collections they're called on. They return a new collection with the modified results.

`map` steps through each element in a `List`, applies the algorithm you provided to each element in turn, and returns a new `List` with the modified elements.

It's common to use `map` to return a `List` with a different type. For example:

```Scala
vals numsLessThanFive = nums.map(_ < 5) // List(true, true, true, true, false)
``` 

`filter` creates a new `List` with elements that pass the predicate.

```Scala
val shortNames = names.filter(_.length <= 4) 
```

`foreach` is used to loop over a collection and produce side effects, such as printing information. You can chain it with other methods.

```Scala
names.foreach(println)

nums.filter(_ < 4).foreach(println)
```

`head` is used to access the first element from a `List`. Because a String is a sequence of characters, you can use `head` to get the first letter.

```Scala
nums.head // 1
"scala".head // 's'
```

`tail` accesses every element in a List after the head element.

```Scala
nums.tail // List(2,3,4)
"scala".tail // 'cala'
```

`head` and `tail` will throw an exception if called on an empty collection.

`take` and `takeWhile` let you take elements out of a `List` when you want to create a new List. `take` returns the first n elements. `takeWhile` returns the first group of elements that match a predicate.

```Scala
nums.take(2) // List(1,2)
names.take(1) // ("Joel")

nums.takeWhile(_ < 5) // List(1,2,3,4)
names.takeWhile(_.length < 5) // List("Joel", "Ed")
```

`drop` and `dropWhile` are the opposite. `drop` returns the elements after the first n specified. `dropWhile` returns the elements after the first group of elements that satisfy a predicate.

```Scala
nums.drop(1) // List(2,3,4)
names.dropWhile(_ != "Chris") // List("Chris", "Maurice") 
```

`reduce` takes a function and applies it to successive elements in a `List`.

```Scala
val a = List(1,2,3,4)
def add(x: Int, y: Int): Int = x + y
a.reduce(add) // 10

a.reduce(_ + _) // 10
a.reduce(_ * _) // 24
```

### Common Map Methods
Common methods for immutable `Map`s include:

```Scala
// iterate over a Map
for((k,v) <- m) println(s"$k, $v")

// get keys
m.keys

// get values
m.values

// test if a Map contains a key
m.contains(3)

// transform Map values
val upperMap = map.transform((k,v) => v.toUpperCase)

// filter a Map by its keys
val twoAndThree = m.filterKeys(Set(2,3)).toMap

// take the first n elements from a Map (only applicable if the Map is sorted)
val firstTwoElements = m.take(2)
```
 
For mutable `Map`s:

```Scala
// add elements with +=
states += ("AZ" -> "Arizona")
states ++= Map("CO" -> "Colorado", "KY" -> "Kentucky")

// remove elements with -=
states -= "KY"
states --= List("AZ", "CO")

// aupdate elements by reassigning them
states("AK") = "Alaska, the state you've been looking for"

// filter elements
states.filterinPlace((k,v) => k == "AK")
```

### "Pseudo Collections"
Functional programming doesn't use null values, and so Scala's solution is to use constructs like `Option`, `Some` and `None`.

`Option` and `Try` are useful when you have unsafe methods, which might cayse a `NullPointerException`, saving you from having to check for the method returning a null value.

### Option, Some, None
An `Option` is like a "collection" containing at most one element. It acts as a wrapper for a value that might or might not be present.

For example, you can pass a method which can return `null` to `Option`. If the method returns a valid value, then the "collection" contains that value. It'll return `Some`, a subtype of `Option`.

If the method returns `null`, the value will be `None` (a singleton object and still a valid value).

So, `Option` is a container with either zero or one item in it. `Some` is a container with one item in it, while `None` is a container with nothing in it.

Including `Option` in the method signature tells you that the return value is nullable.

```Scala
def toInt(s: String): Option[Int] = {
	try {
		Some(Integer.parseInt(s.trim))
	} catch {
		case e: Exception => None
	}
}

def sqrt(n: Int): Option[Double] =
	if (n >= 0) Some(Math.sqrt(n)) else None
```

Lots of methods on collections are designed to work with `Option`s.

If you are consuming a method that returns an `Option`, you can use a `match` expression or a for expression.

```Scala
// using match
toInt(x) match {
	case Some(i) => println(i)
	case None = println("That didn't work")
}

// using a for expression
val y = for {
	a <- toInt(StringA)
	b <- toInt(StringB)
	c <- toInt(StringC)
} yield a + b + c // Some(6)
```

Safe APIs are designed to return `Option`s so the user of the API doesn't need to wrap the result in an `Option`.

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

`Option`s have methods such as `isEmpty` and `get`.

`get` is an unsafe way to retrieve a value from an `Option`. In general, don't use it. `getOrElse` is safer and will return the `Option`'s value if it's non-empty, or else will return the default provided.

You can used `Option`s with pattern matching and operate on them with `map`, `flatMap`, `filter` and for comprehensions.

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

`map` lets you transform an optional value, `flatten` lets you simplify a nested optional structure, `flatMap` lets you combine optional values in an ordered sequence.

```Scala
case class Car(model: String, owner: Option[Person], registrationPlate: Option[String])

case class Person(name: String, age: Int, drivingLicence: Option[String])

def ownerName(car: Car): Option[String] =
	car.owner.map(p => p.name)
```

When you use `map` on an `Option`, `map` takes a function:
- if the optional value is present, `map` will apply the function to it and return it wrapped as an optional value
- if the optional instance is `None`, `map` will return it without applying the function.

```Scala
def ownerDrivingLicence(car: Car): Option[String] =
	car.owner.map(_.drivingLicence).flatten
```

`flatten` merges two nested optional values into one. If you need to flatten more than two values, you can call it multiple times. 

When you use `flatten` on an instance of `Option[Option[A]]` and it returns an `Option[A]`:
- If the outer optional value is `Some`, it returns the inner instance of `Option`
- If the outer optional valie is `None`, it returns it.

`flatMap` lets you combine the functionality of `map` and `flatten`.

```Scala
def ownerDrivingLicence(car: Car): Option[String] =
	car.owner.flatMap(_.drivingLicence)

```

When used on `Option[A]`, `flatMap` will apply a function to an optional value, it it's present. The function returns an instance of `Option`, so you don't need to wrap the result in `Option`. If the optional instance is `None`, `flatMap` won't apply the function.

You can use a for-comprehension on nested `Option`s. `yield` returns a value you can compute using the extracted values wrapped inside an `Option`. If the for-comprehension finds an absent optional value, it evaluates the entire expression as `None`.

```Scala
def ownerDrivingLicence(optCar: Option[Car]): Option[String] =
	for {
		car <- optCar
		person <- car.owner
		drivingLicence <- person.drivingLicence
	} yield drivingLicence
```

You can add filtering to a for-comprehension.

```Scala
def ownerDrivingLicence(optCar: Option[Car], ownerName: String): Option[String] = 
	for {
		car <- optCar
		person <- car.owner
		if person.name == ownerName // chain will stop if false
		drivingLicence <- person.drivingLicence
	} yield drivingLicence.toUppercase // can modify yield value before returning
```

When using `Option`s, you only need to think about wheter you got a `Some` or `None` when you finally handle the result value in a `match` expression.

```Scala
toInt(x) match {
	case Some(i) => println(i)
	case None => println("That didn't work")
}
```

`Option`s are useful where a value is optional and you want to avoid nulls, such as line2 in an address.

```Scala
class Address (
	var street1: String, 
	var street2: Option[String], 
	var city: String, 
	var state: String, 
	var zip: String
)
```

`Option` lets you redefine partial functions as total functions to make your code safer.

```Scala
// partial function
val log: partialFunction[Int, Double] = { 
	case x if x > 0 => Math.log(x)
}

// total function
def log(x: Int): Option[Double] = x match {
	case x if x > 0 => Some(Math.log(x))
	case _ => None
}
```

### Option methods
`isDefined` returns true if an optional instance has a value, false otherwise.

```Scala
Some(1).isDefined // true
None.isDefined // false
```

`isEmpty` is the opposite of `isDefined`. It returns true if an optional instance is empty, false otherwise.

```Scala
Some(1).isEmpty // false
None.isEmpty // true
```

`getOrElse` returns the optional value if present, otherwise it executes the provided default operation.

```Scala
Some(1).getOrElse(-1) // 1
None.getOrElse(-1) // -1
```

`find` returns an optional value if its element satisfies a given predicate.

```Scala
Some(10).find(_ > 5) // Some(10)
Some(1).find(_ > 5) // None
None.find(_ > 5) // None
```

`exists` combines `find` with `isDefined`. It retuns true if the value is present and satisfies the given predicate, false otherwise. 

```Scala
Some(10).exists(_ > 5) // true
Some(1).exists(_ > 5) // false
None.exists(_ > 5) // false
```

### Try, Success, Failure
A `Try` is a wrapper for a computation that might or might not fail and is useful for functional error handling. 

`Try`, `Success`, `Failure` are commonly used when writing methods that interact with files, databases and networks, since these can easily throw exceptions.

`Try` makes it simple to catch exceptions, `Failure` contains the exception.

A `Try` guards against methods which can throw exceptions, avoiding having to write defensive try/catch blocks and preventing runtime crashes. Multiple nested try/catch blocks make the code hard to follow and you can't chain multiple operations that are prone to failure.

A `Try` is essentially a "collection" with either a value if the code went well, or an exception if the code threw one. The two subtypes of `Try` are `Success` and `Failure`. You need to provide implementations for both, since `Try` is a sealed class.

`Failure`  holds the throwable that would have been thrown. `Success` wraps the value that the successful computation returned.

```Scala
sealed abstract class Try[+T]
case class Failure[+T](t: Throwable) extends Try[T]
case class Success[+T](value: T) extends Try[T]
```

```Scala
def toInt(s: String): Try[Int] = Try(Integer.parseInt.s.trim)

val a = toInt("1") // Success(1)
val b = toInt("boo") // Failure(java.lang.NumberFormatException: For input string: "boo")
```

Common approaches for working with the results of a `Try` use `match` and `for` expressions:

```Scala
toInt(x) match {
	case Success(i) => println(i)
	case Failure(s) => println(s"Failed. Reason: $s")
}

val y = for {
	a <- toInt(stringA)
	b <- toInt(stringB)
	c <- toInt(stringC)
} yield a + b + c // Success(6)
```

When you use a for expression and everything works, it returns the value wrapped in a `Success`. If it fails, it returns a `Failure`.

The `Try` object can also be processed with `map`, `flatten`, `flatMap` and `filter`.

`map` can transform the result of a successful computation. If the result is of type `Failure`, nothing happens. 

`flatten` lets you merge two nested `Try` instances into one.

`flatMap`, as usual, combine `map` and `flatten`.

```Scala
def getRegistrationDate(registration: Try[Registration]): Try[LocalDate] = registration.map(_.localDate)

def registerForNextExamSession(student: Student, topic: String): Try[Registration] = 
	getNextExamSession(topic).map { examSession =>
		register(student, examSession)
	}.flatten

def registerForNextExamSession(student: Student, topic: String): Try[Registration] = 
	getNextExamSession(topic).flatMap { examSesssion =>
		register(student, examSession)
	}
```

You can use a for comprehension to express a chain of `flatMap` and `map` operations to improve ther readability.

```Scala
def registerForNextExamSession(student: Student, topic: String): Try[Registration] = 
	for {
		examSession <- getNextExamSession(topic)
		registration <- register(student, examSession)
	} yield registration
```

`isSucccess` returns true if the instance is of type `Success`.

`isFailure` returns true if the instance is of type `Failure`.

`getOrElse` returns the value if your instance is of type `Success` or evaluates the parameter to produce a result if the instance is of type `Failure`.

```Scala
Try(5/2).getOrElse { println("Side Effect"); 42} // 2
Try(5/0).getOrElse { println("Side Effect"); 42} // 42
```

A `Try` can be used similar to an `Option` to make unsafe methods safer.

If your code might return null, use an `Option`. 

If your code might throw exceptions, use a `Try`.

```Scala
def betterUnsafeMethod(): Try[String] = Failure(new RuntimeException)
def betterBackupMethod(): Try[String] = Success("A valid result")
val betterFallback = betterUnsafeMethod() orElse betterBackupMethod()
```

### Either
`Either` is an immutable disjoint union used to represent a value with one of two possible types. An instance of Either is either `scala.util.Left` or `scala.util.Right`. It's most commonly used for validation. The left value is used for failure and the right value for success.

An instance of `Either[A,B]` can represent a faled outcome by returnng type `A` and a successful outcome by retuning type `B`. Compared to `Option`, `Either` lets you prvide more information when something fails.

```Scala
private def containsOnlyDigits(phoneNumber: String): Boolean = ???
private def hasExpectedSize(phoneNumber: String): Boolean = ???

def validatePhoneNumber(phoneNumber: String): Either[String, String] =
	if(!containsOnlyDiigits(phoneNumber))
		Left("A phone number should only contain dgits")
	else if(!hasExpectedSize(phoneNumber))
		Left("Unexpected number of digts. Expected 10 digits.")
	else Right(phoneNumber)
```

You cannot initialize `Either` directly, as it's asbstract. For an instance of `Either[A,B]`, provide a value of type `A` to create a `Left[A,B]` and a value of type `B` to create a `Right[A,B]`.

You can use pattern matching on `Either`. You need to provide case clauses for both `Left` and `Right` because `Either` is a sealed class. 

```Scala
def toMessage(outcome: Either[String, Pass]): String =
	outcome match {
		case Left(msg) => s"Fail: $msg"
		case Rght(pass) => s"Pass with score ${pass.score}
	}
```

You can use `map` to transform the value wrapped into a `Left` or `Right` instance of `Either`. The `left` and `right` methods let you tell the compiler whether to manipulate the left or right side. From Scala 2.12, it assumes you want to manipulate the right side and so `right` can be ommitted.

When applied on its left projection, it takes a function of type `A => C` to produce a value of type `Either[C,B]`. When working on its right projection, it takes a parameter of type `B => C` to produce a value of type `Either[A,C]`. 

If your instance matches the selected side, it applies the function to its value. If not, it returns the instance wthout perfrming any transformation.

```Scala
case class Pass(score: Int) {
	require(score >= 60 && score <= 100, "Invalid pass: Score must be between 60 and 100")
	def toPercentage: Double = score / 100.0
} 

def toPercentage(outcome: Either[String, Pass]): Either[String, Double] = outcome.right.map(_.toPercentage)
```

`flatMap` produces a new `Either` instance. It will have a different signature depending on which side of `Either` you select to transform with the `left` and `right` methods.

When applied to the left, it takes a function of type `A => Either[C,D]` and returns a value of of type `Either[C, BD]`; the compiler infers `BD` as the type of the common superclass between `B` and `D`.

When transforming the right projection, it takes a function of type `B => Either[C,D]` and returns a value of type `Either[AC, D]`; the compiler infers `AC` as the type of the common superclass between `A` and `C`.

If your instance matches the selected side, it applies the function t its content to produce a new `Either` value. If not, no transformation happens.

```Scala
def combine(outcomeA: Either[String, Pass], outcomeB: Either[String, Pass]): Either[String, Pass] =
	outcomeA.flatMap { passA =>
		outcomeB.map { passB =>
			val averageScore = (passA.score + passB.score) / 2
			Pass(averageScore)
		}
	}
```

You can apply for comprehensions to `Either`.

```Scala
def combine(outcomeA: Either[String, Pass], outcomeB: Either[String, Pass]): Either[String, Pass] =
	for {
		passA <- outcomeA
		passB <- outcomeB
	} yield {
		val averageScore = (passA.score + passB.score) / 2
		Pass(averageScore)
	}
```

### Working with Either
You can retrieve the value of an `Either` using `getOrElse` with the `left` and `right` functions.

`isLeft` returns true if the instance of of type `Left`, false otherwise.

`isRight` returns true if the instance is of type `Right`, false otherwise.

`exists` takes a predicate function of type `A => Boolean` for its left projection and `B => Boolean` for its right projection. It returns true if its instance matches the selected projection and its value respects the predicate, false otherwise.

```Scala
Right(42).getOrElse { println("generating default"); 0} // 42 

Right(42).isLEft // false
Right(42).isRight // true

val e: Either[String, Int] = Left("hello")
e.left.exists(_.startsWith("scala")) // false
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
3. The type of the Pattern Matching expression is the unified type of all the types in all the cases  
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

## Asynchronous Programming
Asynchronous programming in Scala is done with another pseudo-collection, `Future`.

A `Future` represents a value which may or may not currently be available, but will be available at some point, or an exception if that value could not be made available.

Where a task takes an indeterminate amount of time, its return value may or may not be currently available, but will be at some point. `Future`s let you run these potentially long-running tasks off of the main thread. 

Just as `Try` represents synchronous computations that can fail, `Future` represents asynchronous computations that can fail.

A `Future` takes an execution context, such as `global`, as an implicit parameter. This provides information about the resources available to the program, such as the available set of threads to use. 

In larger applications, you should pass your execution context as an implicit parameter to classes and functions rather than directly using the global execution context. This gives you more control over which execution context to use, including custom ones if necessary.

```Scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

val aFuture = Future { // equivalent to Future.apply()
  println("Loading...")  
  Thread.sleep(1000)  
  println("I have computed a value.")  
  67  
}

def aLongRunningTask(): Future[Int] = ???
val x = aLongRunningTask
```

A `Future` is intended to be run only once (handle this slow computation on another thread and report back when you're done). By comparison, Akka actors are intended to run for a long time and respond to many requests during their lifetime.

A `Future` is a "collection" which contains a value when it's evaluated. The value in a `Future` is always an instance of `Success` or `Failure`.

```Scala
val a = Future {
	Thread.sleep(3 * 1000)
	42
}

val b = a.map(_ * 2) // Future(<not completed>)

// later
b // Future(Success(84))

```

`Future` has an `onComplete` method which takes a partial function in which you handle the `Success` and `Failure` cases. It processes the result of a completed `Future` as an instance of `Try`.

`onComplete` lets you proces the `Future`'s result as a side effect. Like using `foreach` on a collection, `onComplete` returns `Unit` and is only used for side effects.

```Scala
a.onComplete {
	case Success(value) if value.quantity <= 0 => println("Value not available")
	case Success(value) => println(s"Got the value: $value")
	case Failure(e) => e.printStackTrace
}
```

A `Future` is also composable with `map`, `flatten`, `flatMap` and `filter` similar to other pseudo-collections.

`map` access the value wrapped in a `Future` instance and applies a function to it. This lets you transform its produced result. If your `Future` instance has completed successfully, `map` will produce another `Future`.

```Scala
Future(12/2).map(_ * 3) // Future(Success(18))
```

`flatten` merges two nested `Future`s into one. For example, if you have an asynchronous operation that also needs to access a database asynchronously.

```Scala
Future(Future(12/2)).flatten // Future(Success(6))
```

`flatMap` combines `map` and `flatten` to chain `Future`s together.  It lets you express an execution dependency, for example checking for availability must complete successfully before placing an order.

```Scala
Future(12/2).flatMap(n => n.toString) // Future(Success(6))
```

`Future`, `Try` and `Option` are "Monads" in functional programming.

### Combining Multiple Futures
You can combine mutliple futures with a for expression to obtain a single result. This will usually be more readable than using chained `flatMap`s.

Here the three operations will run in order. If any fail, the entire chain will result in a failed asynchronous computation.

```Scala
def getOrderDetails(orderId: Int)(using ec: ExecutionContext): Future[OrderDetails] =
	getOrders(orderId).flatMap { order =>
		getUser(order.userId).flatMap { user =>
			getProduct(order.prductId).map { product =>
				OrderDetails(order, user, product)
			}
		}
}

def getOrderDetails(orderId: Int)(using ec: ExecutionContext): Future[OrderDetails] =
	for {
		order <- getOrder(orderId)
		user <- getUser(order.userId)
		product <- getProduct(oorder.productId)
	} yield OrderDetails(order, user, product)
```

If operations are independent, you can execute them in parallel and extract the result of the computation once they complete successfully.

```Scala
def getOrderDetails(orderId: Int)(using ec: ExecutionContext): Future[OrderDetails] =
	for {
		order <- getOrder(orderId)
		futureUser <- getUser(order.userId)
		futureProduct <- getProduct(oorder.productId)
		user <- futureUser
		product <- futureProduct
	} yield OrderDetails(order, user, product)
```

When morking with multiple asynchronous computations that run independently and in parallel, Scala offers several methods for coordinating them.

`firstCompletedOf` takes in a sequence of `Future`s and returns a `Future` that contains the first instance in the sequence to complete, whether successfully or unsuccessfully.

`find` takes in a sequence of `Future`s and a predicate. It returns a `Future` containing the first value produced that respects the predicate.

`sequence` takes in a sequence of `Future`s and returns a new `Future` containing all the results in a sequence in the same order. If any of them fail, it completes the new `Future` as a failure.

```Scala
def futureA = Future { Thread.sleep(42); 42 }
def futureB = Future(123/0)
def futureC = Future(123)

Future.firstCompletedOf(Seq(futureA, futureB)) // Future(Failure(java.lang.ArithmeticException: / by zero))

Future.find(Seq(futureA, futureB))(_ > 10) // Future(Success(42))

Future.sequence(Seq(futureA, futureC)) // Future(Success(List(42,123))
```

```Scala
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future
import scala.util.{Failure, Success}

object MultipleFutures extends App {
    // use this to determine the “delta time” below
    val startTime = currentTime

    // (a) create three futures
    val aaplFuture = getStockPrice("AAPL")
    val amznFuture = getStockPrice("AMZN")
    val googFuture = getStockPrice("GOOG")

    // (b) get a combined result in a for-expression
    val result: Future[(Double, Double, Double)] = for {
        aapl <- aaplFuture
        amzn <- amznFuture
        goog <- googFuture
    } yield (aapl, amzn, goog)

    // (c) do whatever you need to do with the results
    result.onComplete {
        case Success(x) => {
            val totalTime = deltaTime(startTime)
            println(s"In Success case, time delta: ${totalTime}")
            println(s"The stock prices are: $x")
        }
        case Failure(e) => e.printStackTrace
    }

    // important for a short parallel demo: you need to keep
    // the jvm’s main thread alive
    sleep(5000)

    def sleep(time: Long): Unit = Thread.sleep(time)

    // a simulated web service
    def getStockPrice(stockSymbol: String): Future[Double] = Future {
        val r = scala.util.Random
        val randomSleepTime = r.nextInt(3000)
        println(s"For $stockSymbol, sleep time is $randomSleepTime")
        val randomPrice = r.nextDouble * 1000
        sleep(randomSleepTime)
        randomPrice
    }

    def currentTime = System.currentTimeMillis()
    def deltaTime(t0: Long) = currentTime - t0
}
```

A `Future` immediately begins running the block of code inside it (unlike a Java Thread, where you create an instance and later call it with `start`).

The for expression waits for the values to return, combines them into a tuple and assigns them to the `result` variable. `result` is a `Future` that wraps three tuples.

The main thread doesn't stop when `getStockPrice` is called, or when the for expression runs.

When using asynchronous computations, you shouldn't block a thread waiting for a `Future` to complete. But there are cases where you might need to, such as in testing. You can do this using `Await.ready(Future(), atMost = 5 seconds)` or `Await.result(Future(), atMost = 5 seconds)` to throw an exception on failure or timeout.

### Partial Functions
Partial functions are functions that don't provide implementations for every possible input value they can be given. They handle only a subset of of possible data.

Partial functions are based on pattern matching. You can view a partial function as an anonymous function with one or more case classes in its body. If your input doesn't match any of the case clauses, it will throw a `MatchError` exception at runtime.

They can be used to abstract commonalities between functions.

For example, the function below only operates on the values 1, 2 and 5 and will throw an exception for anything else.

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

With function composition, you usually chain two functions by passing the result of the first function as a parameter to the second. You can also use `andThen`. 

```Scala
val f: String => Int = _.size
val g: Int => Boolean = _ > 2

val gof: String => Boolean = { s => g(f(s)) }

val gof: String => Boolean = f.andThen(g)
```

With partial functions, you might want to combine partial functions as fallbacks if the previous partial function couldn't match the given input. You can compose partial functions as fallbacks using `orElse`.

```Scala
val sqrt: PartialFunction[Int, Double] = 
	{ case x if x >= 0 => Math.sqrt(x)}
val zero: PartialFunction[Int, Double] = { case _ => 0 }
val value: PartialFunction[Int, Double] = { case x => x }

// First create a new funtion using orElse
// Then apply the parameter to it
def sqrtOrZero(n: Int): Double = sqrt.orElse(zero)(n)
def sqrtOrValue(n: Int): Double = sqrt.orElse(value)(n)
``` 

Partial functions are also used in exception handling with the try-catch expression.

```Scala
def n(): Int =
try {
	throw new Exception("BOOM!")
	42
} catch {
	case ex: Exception => println(s"Ignoring exception $ex. Returning zero instead.")
	0
}
```

The `catch` keyword follows partial functions that identify which exceptions it should handle: if an exception doesn't match it, it won't intercept it.

## More Advanced Features

### Lazy Evaluation
Lazy evaluation means an expression is not evaluated until it's first used. It's useful in infinite collections.

By default, Scala evaluates a functon parameter by value (eager evaluation). It computes its value and then passes it to the function. There are cases, however, when you may want to only evaluate a parameter when the function invokes it, and not before. This is called parameter evaluation by name (or lazy evaluation). 

`=>` tells the program to evaluate a parameter by name. The compiler transforms it into a function that takes no parameters and returns the evaluated value.   

The `@tailrec` annotation ensures the function is tail recursive.

```Scala
@tailrec
def retry[T](n: Int, operation: => T): Try[T] = {
	val result = Try(operation)
	if(result.isFailure && n > 0) retry(n - 1, operation)
	else result
}
```

You can declare a value to be `lazy` and delay its initialization until it is first invoked. This can help to speed up the initialization of a class instance that relies on the lazy value. 

```Scala
lazy val aLazyValue = 2  
lazy val lazyValueWithSideEffect = {  
  println("Initialized!")  
  42  
}
```

### The IO Type
The `IO` type, which is part of the cats-effect library, lets you represent synchronous and asynchrnous side effects lazily. By seperating the definition of what to execute from its actual execution, it can become easer to maintain and test. Representing side effects with a lazy approach can make your code less prone to errors.

`IO` can be considered the lazy alternative to the eagerly evaluated `Future`. `IO` is also cancellable, while `Future` is not.

`Future` is not referentially transparent. You can't replace its invocation with its returned value and be confident you will obtain the same result. `IO` is however referentially transparent as it clearly seperates description and execution.

```Scala
import cats.effect.IO

def rollDie(): IO[Int] = IO {
	val n = Random.nextInt(6) + 1
	println(s"Rolled $n...")
	if(n < 4) throw new IllegalStateException(s"Fail. Rolled $n")
	else n
}
```

When you call `rollDie`, you describe the side effect without evaluating it. When ready to do so, you can call the `unsafeRunSync` method to execute it synchronously.

```Scala
val myRoll = rollDie()

myRoll.unsafeRunSync()
```

You can use `IO.async` to describe a side effect your program should execute asynchronously. When invoking this method, you need to implement a function that takes a callback as its parameter. You define the operation to run asynchronously by producing a value of type `Either` and finally feed it back to the callback.

```Scala
import cats.effect.IO

class Stats(val playerId: Long) {
	/* Some meaningful stats loaded here */
	println(s"Loading statistics for player $playerId")
	// Simulate slow operation
	Thread.sleep(10000)
}

object Stats {
	def load(playerId: Long): IO[Stats] IO.async { callback =>
		val either: Either[Throwable, Stats] = Right(new Stats(playerId))
		callback(either)
	}
}
```

When evaluating the side effect, you will receive the result that the last callback produced. `unsafeRunAsync` lets you process it on completion of its execution. Use `unsafeRunAsyncAndForget` if you don't wish to process the result. Both of these return `Unit`. They're comparable to `Future.onComplete`.

```Scala
myIO.unsafeRunAsync {
	case Left(ex) => // do something with the error
	case Right(value) => // do something with the value
}

myIO.unsafeRunAsyncAndForget() // run the side effect, discard the value
```

### Working with the IO Type
`map` lets you transform an `IO` value by applying a given function and returning an `IO`.

```Scala
val ioA = IO("Hello World").map(_.length)
ioA.unsafeRunSync() // 12
```

`flatMap` combines two side effects sequentially. It takes a function and returns an `IO`.

```Scala
val ioA = IO("Hello World").flatMap(n =>
	IO(println(n.length)))
								   )
ioA.unsafeRunSync() // 12
```

You can use a for comprehension as an alternative to `map` and `flatMap`.

```Scala
def rollDie: IO[Int] = IO(Random.nextInt(6) + 1)

def rollDieTwice: IO[Int] =
	for {
		n1 <- rollDie
		n2 <- rollDie
	} yield n1 + n2
```

`parSequence` (which requires a `ContextShift[IO]` implicit parameter) lets you execute multiple side effects in parallel, returning a value of type `IO[List[A]]`. It's analogous to `Future.sequence` while `ContextShift[IO]` is equivalent to `ExecutionContext` by definign what resources are available to your program.

```Scala
def rollDie(n: Int): IO[Int] = IO {
	println(s"Rolling d$n")
	Random.nextInt(n) + 1
}

def rollDice(using cs: ContextShift[IO]): I[List[Int]] = {
	List(rollDie(12), rollDie(8), rollDie(8)).parSequence
}

rollDice.unsafeRunSync() // List(11,7,5)
```

### Implicits
Implicits are used for 
- implicit arguments
- implicit conversions

If you define a method with implicit arguments you can call it without passing any arguments. The compiler understands that the method takes an implicit argument and looks for implicit values within the function's scope (this is called "implicit resolution").

Implicit parameters are marked with `implicit` (Scala 2) or `using` (Scala 3). The implicit values themselves are marked with `implicit` (Scala 2) or `given` (Scala 3).

```Scala
def aMethodWithImplicitArgs(implicit arg: Int) = arg + 1  
implicit val myImplicitInt = 46  
println(aMethodWithImplicitArgs)  // aMethodWithImplicitArgs(myImplicitInt)

implicit val timeout = 3000  
def setTimeout(f: () => Unit)(implicit timeout: Int) = f()  
  
setTimeout(() => println("timeout"))// extra parameter list omitted
```

This can be useful if you have a number of related functions that all take the same parameter.

```Scala
// without implicits
for {
_ <- validateAddressWithinDistance(userContext)
_ <- validateSelection(selection)(userContext)
_ <- validateBalance(selection)(userContext)
} yield placeOrder(selection)(userContext)

// with implicits
// Scala 2
implicit val userContext: UserContext = getUserContext(userId)
// Scala 3
given userContext: UserContext = getUserContext(userId)
for {
_ <- validateAddressWithinDistance
_ <- validateSelection(selection)
_ <- validateBalance(selection)
} yield placeOrder(selection)
```

You use implicit conversions to add methods to existing types over which you don't have control.

For example, you can give an `Int` an `isEven` method using implicit conversion.

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

You can decide how the compiler fetches an implicit value for you. The compiler first looks in local scope, then imported scope, then the companion objects of the types included in the call. If it finds 0 matches or 2+ matches, it will throw a compilaton error.

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

For every method that takes a "magical" argument of type `Ordering[Int]`, the compiler looks for an implicit `given` instance.

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

### Type Aliases
A type alias is usually used to simplify declaration for complex types, such as parameterized types or function types.

```Scala
type ReceiveFunction = PartialFunction[Any, Unit]

def receive: ReceiveFunction = {  
  case 1 => println("hello")  
  case _ => println("confused....")  
}
```

## JSON (de)seralization
Scala doesn't offer JSON support natively, so you will need a library such as circe. Converting a Scala instance to/from JSON is called serialization/deserialization.

circe uses the type class pattern to define a conversion to JSON for any instance. You can convert any type to JSON that has an instance of `Encoder[T]`. It has a predefined set of encoders for basic types, such as `List` or `Map`.

```Scala
case class Person(fullName: String, dateOfBirth: LocalDate)

object Person {
	given personEncoder: Encoder[Person] with {
		def apply(p: Person): Json = {
			val age = Period.between(
				p.dateOfBirth, LocalDate.now()
			).getYears
			Json.obj(
			"fullName" -> Json.fromString(p.fullName)
			"age" -> Json.fromInt(age)
			)
		}
	}
}
```

When using case classes, you can request that the coder be generated automatically. The generated encoder analyzes the structure of the case class to define a new encoder.

```Scala
import io.circe.generic.semiauto._

case class Person(fullName: String, dateOfBirth: LocalDate)

object Person {
	given personEncoder: Encode[Person] = deriveEncoder[Person]
}
```

The `Decoder` trait defines how to parse a JSON object into a class instance. Once you have defined a decode you can use the `decode` function from `io.circe.parser._`, which returns ether a class instance or an error.

circe offers predefined decoders for commonly used types. Using `""""""` lets you automatically escape special characters, such as " " or  \.

```Scala
import io.circe._

object Person {
	given personDecoder: Decoder[Person] with {
		def apply(c: Hcursor): Either[DecodingFailure, Person] =
		for {
			fullName <- c.downField("fullName").as[String]
			datefBirth <- c.downField("dateOfBirth").as[LocalDate]
		} yield Person(fullName, dateOfBirth)
	}
}

val goodData: String =
 """ { "fullName": "John Doe", "dateOfBirth": "1987-11-22" } """

decode[Person](goodData) // Right(Person(John Doe, 1987-11-22))

decode[List[String]](""" ["Hello", "World"] """) // Right(List(Hello, World))
```

When using case classes, circe can automatically infer a decoder by analyzing their structure.

```Scala
import io.circe.generic.semiauto._

case class Person(fullName: String, dateOfBirth: LocalDate)

object Person {
	given personDecoder: Decoder[Person] = deriveDecoder[Person] 
}
```

You can define custom encoders and decoders, for example if you need to use a specific date format.

```Scala
import java.time.LocalDate
import io.circe._

given dateEncoder: Encoder[LocalDate] with {
	def apply(date: LocalDate): Json = 
	Json.fromString(formatter.format(date))
}

given dateDecoder: Decoder[LocalDate] with {
	override def apply(c: HCursor): Either[DecodingFailure, LocalDate] =
		c.as[String].map(text => LocalDate.parse(text, formatter))
}

LocalDate.of(1981,7,25).asJson // "25/07/1981"

decode[LocalDate](""" "11/22/1987" """) // Right(1987-11-22)
```

## sbt and ScalaTest
### sbt
While you can use several build tools with Scala (such as Ant, Maven, Gradle), sbt was the first designed specifically for Scala, and is upported by Lightbend.

sbt uses a standard project directory structure.

```
build.sbt
project/
src/
-- main/
   |-- java/
   |-- resources/
   |-- scala/
|-- test/
   |-- java/
   |-- resources/
   |-- scala/
target/
```

You can scaffold a Hello World project with

```
sbt new sbt/scala-seed.g8
```

The `build.sbt` file lets you specify versions and dependencies. 

```
name := "HelloWorld"
version := "1.0"
scalaVersion := "2.13.8"
```

### ScalaTest
ScalaTest is one of the main testing libraries for Scala. Add it to your library dependencies in `build.sbt`. 

```Scala
libraryDependencies += "org.scalatest" %% "scalatest" % "3.2.11" % Test
```

Create test files in the `src/test/scala` directory.

- Your class should extend a testing suite, such as `FunSuite`
- Each test needs to have a unique name
- Each test should asset that a condition has been satisfied

```Scala
package simpletest

import org.scalatest.funsuite.AnyFunSuite

class HelloTests extends AnyFunSuite {
    // test 1
    test("the name is set correctly in constructor") {
        val p = new Person("Barney Rubble")
        assert(p.name == "Barney Rubble")
    }
    // test 2
    test("a Person's name can be changed") {
        val p = new Person("Chad Johnson")
        p.name = "Ochocinco"
        assert(p.name == "Ochocinco")
    }
}
```

You can run tests with `sbt test`.

You can also write tests in a BDD-style (Behaviour Driven Development).

```Scala
package simpletest

import org.scalatest.FunSpec

class MathUtilsSpec extends FunSpec {
    describe("MathUtils::double") {
        it("should handle 0 as input") {
            val result = MathUtils.double(0)
            assert(result == 0)
        }
        it("should handle 1") {
            val result = MathUtils.double(1)
            assert(result == 2)
        }
		// you can mark tests as "pending"
        it("should handle really large integers") (pending)
    }
}
```

With ScalaTest, you need to define a class that extends one of its specification classes, such as `org.scalatest.flatspec.AnyFlatSpec` and then seleect one of its matchers, such as `org.scalatest.matchers.should.Matchers`

For example, to test a program that computes the frequency of characters in a text.

```Scala
// Frequency.scala
class Frequency {
	def count(text: String): Set[(Char, Int)] =
		text.groupBy(char => char).map {
			case (char, occurrences) => char -> occurrences.length
		}.toSet
}
```

```Scala
// FrequencyTest.scala
class FrequencyTest extends AnyFlatSpec with Matchers {
	val frequency = new Frequency
	
	"Frequency" should "count no frequency for an empty string" in {
		val result = frequency.count("")
		result.shouldEqual(Set.empty)
	}
	
	it should "count the frequency of the characters in a text" in {
		val result = frequency.count("test")
		val expcted = Set('t' -> 2, 'e' -> 1, 's' -> 1)
		result.shouldEqual(expected)
	}
}
```

If the same program uses a `Future`, such as reading asynchronously from a file, `AnyFlatSpec` can support this. The assertion is valid only if the `Future` completes successfully and its value respects the given assumption.

```Scala
// Frequency.scala
class Frequency {
	def countFromFile(filename: String)(using ec: ExecutionContext): Future[Set[(Char, Int)]] =
	readFromFile(filename).map(count)
	
	private def readFromFile(filename: String)(using ec: ExecutionContext): Future[String] = 
		val source = Source.fromFile(filename)
		try {
			source.getLines().mkString
		} finally source.close()
	
	def count(text: String): Set[(Char, Int)] =
		text.groupBy(char => char).map {
			case (char, occurrences) => char -> occurrences.length
		}.toSet
}
```

```Scala
// FrequencyTest.scala
class FrequencyTest extends AnyFlatSpec with Matchers {
	val frequency = new Frequency
	
	// ...
	
	it should "asynchronously count characters from a file" in {
		val filename = getClass.getResource("/myFile.txt").getPath
		val result = frequency.coountFromFile(filename)
		val expcted = Set('t' -> 2, 'e' -> 1, 's' -> 1)
		result.map(_.shouldEqual(expected))
	}
}
```


### HTTP APIs
There are multiple external libraries for building HTTP servers with Scala, such as http4s.

With http4s, you link your routes to your business logic through instances of `HttpRoutes`. Each `HttpRoutes` uses partial functions to match an incoming HTTP request and produce a HTTP response together with a side effect, such as an IO read/write or a connection to a third party API. 

A `HttpRoutes[IO]` matches a request and produces a response wrapped in an IO instance to represent possible side effects. Your executable object extends `IOApp` and uses a `BlazeServerBuilder` to bind to a given port and host and to mount multiple instances of `HttpRoutes[IO]` to define the API of a HTTP server using a `fs2.Stream` instance.

`IOApp` is an interface that requires you to define a `run` function.

`BlazeServerBuilder` defines a HTTP server with an address, port, and routes using a stream to process requests. `ExecutionContext.global` provides a thread pool to use while streaming.

`BlazeServerBuilder[IO]` with `bindHttp` provides a port and host. `withHttpApp` attaches a group of routes with a prefix to your server. Finally, `serve` transfors the Blaze definition into a stream that represents your HTTP server.

```Scala
object DemoApp extends IOApp {
	private val httpApp = Router(
		"/" -> PingApi().routes
	).orNotFound
	
	override def run(args: Lst[String]): IO[ExitCode] =
		stream(args).compile.drain.as(ExitCode.Success)
	
	private def stream(args: List[String]): fs2.Stream[IO, ExitCode] =
		BlazeServerBuilder[IO](ExecutionContext.global)
		.bindHttp(8000, "0.0.0.0")
		.withHttpApp(httpApp)
		.serve
}
```

It's a good practice to have a few endpoints to ensure yoour system is responsive and healthy. A `healthCheck` endpoint is useful for monitoring systems, while a `ping` endpoint lets you perform a lightweight request to check that they can successfully communicate with the system. 