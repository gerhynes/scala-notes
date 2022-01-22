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
