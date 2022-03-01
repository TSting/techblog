---
title: "Newspeak and Domain Modeling"
date: 2022-03-01T09:48:22+01:00
author: kaykreuning
draft: false
tags: [ "domain modeling" ]
---

_this article was first published at [www.kkreuning.nl](https://www.kkreuning.nl)_

Newspeak is a fictional language spoken by the people of Oceania in George Orwell's novel Nineteen Eighty-Four. 

>“‘Don’t you see that the whole aim of Newspeak is to narrow the range of thought? In the end we shall make thoughtcrime literally impossible, because there will be no words in which to express it.’”
- Syme, from "Nineteen Eighty-Four" by George Orwell

Newspeak sucks for the people of Oceania but we developers can learn a thing or two from Ingsoc.

## What about domain primitives

As an illustration let us come up with a simple example method signature in Java:

```java 
String foo(String str);
```

What does this code do? The signature only tells us it’s a method from `String` to `String` so clearly it does something with the input string, or it might not be able to do something and return `null`, or mutate some state, or perform other side effects, or throw a `Throwable` (this is Java after all). 

Now let us take a look at an equivalent signature, in Scala this time:

```scala
sealed trait PostalCode
case class Zipcode(i: Int) extends PostalCode 
case class Postleitzahl(i: Int) extends PostalCode 

def foo(str: String): Option[PostalCode]
```

Better, this code turns a `String` into one of two possible `PostalCodes` (either US or German) or fails to do so, returning a `None`. An estimated guess would be that this function tries to create a valid postal code from a string. We just had to specify our domain with a few types and now the intent of this piece of code is clear. 

Borrowing a term from Domain Driven Design, we call domain stuff as a dedicated to construct a **domain primitive**. In the way that a language primitive (`String`, `boolean`, `int`, `float`, etc.) is a building block of programming code, a domain primitive is a building block for the domain model and business logic (`ShoppingBag`, `Order`, `Quantity`, etc. if you’re in e-commerce for example).

A domain primitive is precise in its definition and impossible to be illegal, its existence implies its validity.

The true power of modeling in this way is revealed when using these domain primitives in the actual business logic:

```scala
def x(p: PostalCode, n: HouseNumber): Future[Option[Address]]
```

Looking at types, there is only one thing this code could be doing: asynchronously returning an address or no result based on a postal code and a house number. This functional will work with any `PostalCode` so we can infer that both US and German addresses are supported per our domain. 
Good luck figuring all of that out from the following signature:

```java
CompletionStage<String> x(String s, int i);
```

Lets see if proper naming might help:

```java
CompletionStage<String> lookupAddress(String postalCode, int houseNumber);
```

We are still not sure what valid a postal code even is (US, German, French, etc.) and the house number can still be zero or negative. Also what happens when non-sensical parameters are passed? Do we return `null` or let the `CompletionStage` fail? Both are not ideal. 

## Language primitives and bomb disposal

Let us investigate the potential problems of using  language primitives in domain logic with another example, in which we model a method to transfer money from one account to another account:

```java
boolean transfer(String fromAccountNumber, String toAccountNumber, BigDecimal amount);
```

Now let’s count the ways in which this method can be called incorrectly:

1. any argument could be `null`. 
2. `amount` can be zero, which doesn’t make sense when transferring money. 
3. `amount` can be negative, which also doesn’t make sense. 
4. `fromAccountNumber` or `toAccountNumber` might not be actual account numbers. 
5. `fromAccountNumber` and `toAccountNumber` can be the same account number, which again doesn’t make sense. 
6. `fromAccountNumber` and `toAccountNumber` can be mixed up, transferring money in the wrong direction. 

That’s 9 potential mistakes for this one method and the compiler won’t help us with any of them, it comes down to human disciple and diligence to do the right thing.

> Issues 1, 2, 3, 4, and 5 can be caught if we add the proper checks ourselves, that way this method will just blow up at runtime, which is unfortunate but whatever. Issue 6 however cannot be checked at all. If the caller mixes up the two account numbers we won’t find out until angry customers start calling!

# Type aliases

In contrast, consider the following Scala code using the excellent [scala-newtype](https://github.com/estatico/scala-newtype) library:

```scala
@newtype case class FromAccount(a: Account)
@newtype case class ToAccount(a: Account)

def transfer(from: FromAccount, to: ToAccount, amount: Money): Boolean
```

We added extra type information, like before, and  now if the caller mixes up the arguments our code won’t compile. We moved all the way up the defect hierarchy, nice!

> It is still possible to mix up `FromAccount` and `ToAccount` in the example above, but at least that will be more obvious now.
We won’t go into it deeper but if we _really_ want to constrain the code all the way up to 11, we can use something like [ArchUnit](https://www.archunit.org) to enforce that a `FromAccount` can only be derived from the logged in user, ensuring its impossible to create otherwise. 

The code above is more verbose but who cares? The code is type safe and its intent is more explicit.

A lot of modern, high level languages support some notion of type aliasing and/or newtypes:

- Scala 2 (as mentioned) has [scala-newtype](https://github.com/estatico/scala-newtype)
- Scala 3 has [Opaque Type Aliases](http://dotty.epfl.ch/docs/reference/other-new-features/opaques.html)
- Haxe supports [Abstracting Primitives](https://haxe.org/blog/abstracting-primitives/)
- Haskell has [newtype](https://wiki.haskell.org/Newtype) built in
- Kotlin has [Type aliases](https://kotlinlang.org/docs/type-aliases.html)
- Swift supports [Type Alias Declaration](https://docs.swift.org/swift-book/ReferenceManual/Declarations.html#grammar_typealias-declaration)
- OCaml has [Custom Data Types](https://ocaml.org/learn/tutorials/data_types_and_matching.html)

Java has... nothing, so we must create wrappers ourselves. 

> Wrapping in any language as opposed to type aliasing can have an impact on performance because we are basically boxing and unboxing the inner value of the domain primitive. Still the trade-off between performance and type safety is definitely worth it.

## How to spot weak models

In your current project right now, how many calls to `require` and `assert` and their relatives can you count? All of them are like booby traps in the code base and when a caller trips up they will explode all over the runtime.

Here is an incomplete list of things to watch for that might need some bomb disposal:

- checking for `null`s, absence should be modeled with an `Option` type. 
- functions with two or more arguments of the same type (unless your function is [commutative](https://en.m.wikipedia.org/wiki/Commutative_property)). 
- using a `List` where a `Set` or `OrderedSet` is more appropriate, which is almost always.  Even if you don’t care, a `Set` communicates intent better. 
- language primitives as method inputs for business logic. 
- `UUID`s as method argument, a customer uuid is not the same as an order uuid. 
- using a `List` instead of a `NonEmptyList` when at least one element is expected.

## Dealing with garbage

So far we didn’t had to deal with the external world yet. Real programs have to watch out for garbage data all the time, be it from database responses, user input, third party services, configuration files, etc. 

Let us come up with some JSON response from an API representing a list of users:

```json
[
	{
		"id": "B4AE6A1B-EE07-4FBE-9588-11F451980A07",
		"name": "John",
		"age": 32,
	},
	{
		"id": "2711EBA4-098C-44E9-A8A2-518FF07969BF",
		"name": "Mary",
		"age": 31
	}
]
```

A naive Scala representation of this data and a parse function look like this:

```scala
final case class User(val id: String, val name: String, val age: Int)

object User {
  def fromJson(input: String): List[User] = ???
}
```

The `User` object can be decoded from a JSON string and passed to the rest of the program. But what about the unhappy path?

```json
[
	{
		"id": “foo”,
		"name": "John",
		"age": -1,
	}
]
```

Is perfectly legal from a type standpoint but domain wise not what we want. If such a response is carelessly passed into our business logic it will become ticking time bomb. A better encoding would be, using [scala-newtype](https://github.com/estatico/scala-newtype) and [refined](https://github.com/fthomas/refined):

```scala
@newtype
final case class UserId(value: UUID)

@newtype
final case class Name(value: String Refined NonEmpty)

@newtype
final case class Age(value: Int Refined Positive)

type Users = NonEmptyList[User]

final case class User(id: UserId, name: Name, age: Age)

object User {
  def fromJson(input: String): Either[DecodeException, Users] = ???
}
```

Now it is **impossible** to create illegal state since we would not be able to satisfy the compiler, illegal state has become inexpressible. We are forced instead to handle invalid input at the most outer layer or “edge” of our program where the raw data comes in. 

**tl; dr**: always use domain primitives and never language primitives in your business logic.  

## Bonus round
We won’t talk about them now, but domain modeling has additional advantages:

- reduced cognitive overhead for developers. 
- better auto-complete suggestions (where supported by your editor or IDE). 
- no ambiguity means less room for bugs which leads to fewer defects. 
- reduced API surface decreases the chance of vulnerabilities like SQL injections and XSS.

## Conclusion
A proper domain model built from domain primitives forces us, developers, to do the right thing.

In the end we shall make ~~thoughtcrime~~ illegal state literally impossible!
