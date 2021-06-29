# Variance Basics in Java and Scala

Copied for reading [from](https://oldfashionedsoftware.com/2008/08/26/variance-basics-in-java-and-scala/) Matt Malone's [blog](https://oldfashionedsoftware.com)


## Variance in Java

If you're a Java developer then you probably know a thing or two about subtyping. `B` is a subtype of `A` if `B` extends or implements `A` (I'll use this convention throughout this post). `A` is the supertype, `B` is the subtype. But what about arrays? Or generic collections? Is an array of `B` a subtype of an array of `A`? Is `List<B>` a subtype of List? Let's do some experiments:


```java
String[] arrayB = { "a", "b", "c" };
Object[] arrayA = arrayB;
Object testObj = arrayA[0];

List<String> listB = new ArrayList<String>();
listB.add("123");

List<Object> listA = (List)listB;
testObj = listA.get(0);

List<Double> listC = new ArrayList<Double>();
listC.add(new Double(10.0));
listB = (List)listC;

System.out.println(listB.get(0));
```

I've left out the boilerplate for brevity. This code compiles and it runs fine, mostly. This tells us three things. First, Java treats an array of `B` as a subtype of an array of `A`. We know this because we can use a reference to an array of `Object`s to refer to an array of `String`s. This means an array of `String`s "is-a" array of `Object`s.

Second, we know that an `ArrayList` is a subtype of a `List`. This makes sense, because `ArrayList` implements interface `List`. An ArrayList "is-a" List. We can even finagle the `List` into a `List`, but we have to make an explicit cast.

Third, we see a `ClassCastException` during the call to `listA.get(0)`. We use the explicit cast again to assign a List to a reference to `List`. The compiler allows this! This kind of sloppy typing is one of the main complaints of Java generics' detractors (this link includes an example of an erroneous assignment that doesn't even require an explicit cast!). Now, let's look at some of the things Java doesn't allow.


```java
Object[] arrayA = { new Object() };
String[] arrayB = arrayA;
List<String> listB = new ArrayList<String>();
List<Object> listA = listB;
```

This causes two compiler errors. The first occurs when we try to assign an array of `Object` to a reference to an array of `String`. This is sensible. An array of `Object`s could contain any object: `String`s, `Double`s, `Thread`s, anything. We can't treat such an array as an array of `String`s.

The second error occurs when we try to assign a list of `String`s to a reference to a `List` of `Object`s. This is the same code as in the previous snippet, but without the explicit cast. The compiler doesn't allow this. Since lists are mutable (we can add and remove items) we could add non-`String` members to listA, which means those non-`String` members would also be in `listB`. But if we don't add any items to `listA`, the assignment is perfectly safe. Java can't figure out in all cases when it's safe to do an assignment and when it is not.

Why do Java generics work the way they do? I think it's mainly due to two factors: backward compatibility, promoting adoption of the feature. When generics were introduced, there were already millions of lines of code out there that depend on regular, non-generic, mutable collections. To make the new code compatible with legacy code, the type parameters are erased during compilation, and allowing those dangerous casts lets developers work around the fact that `List<B>` is not a subtype of `List`. To make it easier to use generics in new code and convert to non-generics for integration with old code, the compiler rules were made fairly permissive.

## Variance Terminology

If you're not familiar with the term ***variance***, here's what it means with respect to Java. Java arrays are *covariant*. That means that an array of `B` is a subtype of an array of `A`, provided that `B` is a subtype of `A`. The type-subtype relationship of the arrays follows the relationship of the contents. Lists in Java are invariant (some say nonvariant). A `List` of` String`s has no relationship to a List of `Object`s. You can explicitly cast a `List` of `String`s to a `List` of` Object`s but you can force the conversion the opposite direction, too, (Yuck.) so that doesn't count. Now consider some hypothetical generic class `X` such that `X<A>` is a subtype of `X<B>` if B is a subtype of `A`. That's the opposite of the way arrays work. The hypothetical generic `X` is contravariant.

One more detail of terminology: A type class could theoretically be covariant with respect to one type parameter, and contravariant with respect to another (not in Java, just theoretically). So, say you have a generic class `X` that is covariant with respect to its first type parameter and contravariant with respect to its second. So `X<B, I>` is a subtype of `X` only if `B` is a subtype of `A` and `J` is a subtype of `I`. Weird, huh? This actually happens in Scala.

## Variance in Scala

In Scala, variance is not left to chance. There are very strict rules. Variance with respect to type parameters is spelled out explicitly for each class (or trait). The same conventions are used for `Array`s, `List`s, or any generic class! The variance system (indeed, the whole type system) in Scala is more complicated and has a steeper learning curve than in Java, but it affords you the ability to write very expressive code that behaves in a more intuitive fashion. Here are three generic classes that use each of the variance types.


```scala
class InVar[T]     { override def toString = "InVar" }
class CoVar[+T]     { override def toString = "CoVar" }
class ContraVar[-T] { override def toString = "ContraVar" }

/************ Regular Assignment ************/
val test1: InVar[String] = new InVar[String]
val test2: CoVar[String] = new CoVar[String]
val test3: ContraVar[String] = new ContraVar[String]
```

The `+` denotes covariance with respect to the type parameter, and `-` denotes contravariance. The class is invariant with respect to type parameters without a plus or minus. If you run this code you can see that when the type parameters are the same on both sides, the assignments work fine. Now, let's see what happens when we test assignment for different type parameters.

```scala
scala> /************ Invariant Subtyping ************/
scala> val test1: InVar[String] = new InVar[AnyRef]
<console>:5: error: type mismatch;
found   : InVar[AnyRef]
required: InVar[String]

val test1: InVar[String] = new InVar[AnyRef]
                           ^
scala> val test2: InVar[AnyRef] = new InVar[String]
<console>:5: error: type mismatch;
found   : InVar[String]
required: InVar[AnyRef]
val test2: InVar[AnyRef] = new InVar[String]
                           ^

scala> /************ Covariant Subtyping ************/
scala> val test3: CoVar[String] = new CoVar[AnyRef]
<console>:5: error: type mismatch;
found   : CoVar[AnyRef]
required: CoVar[String]
val test3: CoVar[String] = new CoVar[AnyRef]
                           ^
scala> val test4: CoVar[AnyRef] = new CoVar[String]
test4: CoVar[AnyRef] = CoVar

scala> /************ Contravariant Subtyping ************/
scala> val test5: ContraVar[String] = new ContraVar[AnyRef]
test5: ContraVar[String] = ContraVar
scala> val test6: ContraVar[AnyRef] = new ContraVar[String]
<console>:5: error: type mismatch;
found   : ContraVar[String]
required: ContraVar[AnyRef]
val test6: ContraVar[AnyRef] = new ContraVar[String]
                               ^
```

Now you can see the difference in the three classes. The invariant class doesn't allow assignment in either direction, regardless of whether their type parameters have a subtype relationship. The covariant class allows an assignment from subtype to supertype. `String` is a subtype of `AnyRef`, so `CoVar[String]` is a subtype of `CoVar[AnyRef]`. The contravariant class allows an assignment from supertype to subtype. `String`, again, is a subtype of `AnyRef`, so `ContraVar[AnyRef]` is a subtype of `ContraVar[String]`.

So, it's as simple as that, right? Sorry. There's a little more to it. Once you've declared a type parameter as covariant or contravariant there are some restrictions on where this type can be used. Why? Scala is not a purely functional langage in that it allows objects to alter their internal state. It allows mutability. Mutability throws a monkey wrench into variance. Say, for example you had a Scala implementation of a linked list like so:

```scala
class LinkedList[+A] {
  private var next: LinkedList[A] = null
  def add(item: A): Unit = { ... }
  def get(index: Int): A = { ... }
}

val strList = new LinkedList[String]
strList.add("str1")

val anyList: LinkedList[Any] = strList
anyList.add(new Double(1.0))

val str: String = strList.get(1)
```

This code won't compile. Do you see the problem? This code, if it worked, would allow us to create a list of `String`s and then add a `Double` to that list! If we allow that then there's a disaster when we get to the last line. A `LinkedList` of `String`s returns a `Double`. That's no good. That's the same problem as we saw in Java. Scala nips this sort of code in the bud by disallowing covariant types in certain places including member function parameter types. Places where covariant types are allowed and contravariant types are forbidden are called covariant positions. And the reverse is true, too. If contravariant types are allowed and covariant types are forbidden in some position, this is called a *contravariant* position.

The above code causes a compiler error for the add method. You can use covariant types in most other places including constructor parameter types, member val types, and method return types. You can also use covariant types as type parameters, but only where covariant types themselves are allowed. This means, in this example, you could add a method that returns a `Set[A]`, but you could not add a method that takes a `Set[A]` as a parameter, because you could use that passed-in `Set[A]` to alter the state.

Contravariant types can be used as constructor parameter types, member function parameters types, and as type parameters in each of those positions.

Here's a fun exercise for the reader. Experiment with using type parameters of the different kinds of variances in different positions. Below is some example code to get you started. For each usage that fails compilation, why is it not allowed? Can you think of a way that such a usage could cause an inconsistency (such as allowing a `Double` in a `String` collection, for example)?

```scala
class InVar[T](param1: T) {
  def method1(param2: T) = { }
  def method2: T = { param1 }
  def method3: List[T] = { List[T](param1) }
  def method4[U >: T]: List[U] = { List[U](param1) }
  val val1: T = method2
  val val2: Any = param1
  var var1: T = method2
  var var2: Any = param1
}

class CoVar[+T](param1: T) {
  def method1(param2: T) = { }
  def method2: T = { param1 }
  def method3: List[T] = { List[T](param1) }
  def method4[U >: T]: List[U] = { List[U](param1) }
  val val1: T = method2
  val val2: Any = param1
  var var1: T = method2
  var var2: Any = param1
}

class ContraVar[-T](param1: T) {
  def method1(param2: T) = { }
  def method2: T = { param1 }
  def method3: List[T] = { List[T](param1) }
  def method4[U >: T]: List[U] = { List[U](param1) }
  val val1: T = method2
  val val2: Any = param1
  var var1: T = method2
  var var2: Any = param1
}
```

Maybe in a future post, I'll do a more thorough analysis of all the places type parameters of the different variances can be used. If you'd be interested in reading such a thing, please do leave a comment.

## Conclusion

What was the point of all that? We still don't get the mutable, covariant collections we were hoping for. But we do get two things we don't get in Java. We get invariant, typesafe, mutable collections that won't wind up holding objects of the wrong type, and we get covariant, typesafe, immutable collections. If you're new to functional programming (like I am), your first thought might be, *What good is an immutable list? Or an immutable array? If it's immutable, I can't add any items to it, right?*

Take the Scala `List` as an example. It's declared as a `class List[+A]` and it has a method for adding new items that's declared `def + B >: A : List[B]`. So List is covariant with respect to its type parameter `A`. That's great! So a `List[BigInt]` "is-a" `List[Number]` and it "is-a" `List[Object]`. To add items to a List use the `+`function. It doesn't change the `List` it was called on, but it does return a new `List`.

Also, you can add anything to a List. What you add affects what gets returned. Look at the function definition again: `def + B >: A : List[B]`. It take a parameter of type `B`, where `B` is any supertype of `A` (or `A` itself), and it returns a `List[B]`. Let's consider the simple case, and then something more complex.

```scala
scala> val strList = List[String]("abc")
strList: List[String] = List(abc)

scala> strList = strList + "xyz"
strList: List[String] = List(abc, xyz)

scala> val objList = strList + new Object()
objList: List[java.lang.Object] = List(abc, xyz, java.lang.Object@156ee8e)

scala> val anyList = strList + 3.1416
anyList: List[Any] = List(abc, xyz, 3.1416)
```

First we create a `List[String]` called `strList` containing one item. Then we add a second String and store the resulting `List` back in the `strList` variable. Then we call the `+` function with an `Object` parameter. This is allowed because the parameter must have type `B` where `B` is a supertype of `A`, and `Object` is indeed a superclass of String. The call to `+` returns a `List[Object]` which contains both the `String`s and the `Object`. Simple.

Then we call the `+` method on testList again. Remember, `strList` still just contains the two `String`s because it's immutable and we didn't assign the last result back to `strList`. We couldn't have. The result was a `List[Object]`, not a `List[String]`, and List is covariant, not contravariant. This time we supply a parameter of type `Double`. But `Double` isn't even a supertype of `String`! That's ok. Scala determines the nearest common ancestor, the type `Any`, and uses that as `B`. The `+` function returms a `List[Any]` that contains the Strings and the `Double`. That's handy. And covariant and typesafe.
