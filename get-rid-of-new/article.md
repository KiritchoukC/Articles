---
title: How to get rid of the `new` C# keyword
published: true
description: How to make your c# code `new`less ad make your life better
tags: csharp, dotnet, ddd
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/6432nntahmac3hcx1tq8.png
---
## Why would I do that?

I recenlty started a new personal project following the DDD approach.

If you striclty follow the DDD rules, you need to create a new class for each property.
And initializing those classes can quickly become a burden.

Example:
```csharp
var person = new Person(
  new PersonId(1), 
  new PersonFirstName("Clément"), 
  new PersonLastName("Kiritchouk"),
  new PersonBirthYear(1994))
```

There's a way to save a few keystrokes, increase readability and make chaining easier.

## Static constructors obviously

_Disclaimer: This is my way of doing this, feel free to adapt or improvre_

First, I create a static partial Constructors class like this:
```csharp
// Constructors.cs
public static partial class Constructors
{

}
```

I use this class for System classes (DateTime, List, ...)
I'll add some interesting ones later on...

Then I Create another static partial Constructors class where I put all my Person constructors.

```csharp
// Constructors.Person.cs
public static partial class Constructors
{
  public static PersonId PersonId(int id) => new PersonId(id);
  public static PersonFirstName PersonFirstName(string firstName) => new PersonFirstName(firstName);
  public static PersonLastName PersonLastName(string lastName) => new PersonLastName(lastName);
  public static PersonBirthYear PersonBirthYear(int year) => new PersonBirthYear(year);
  
  public static Person Person(PersonId id, PersonFirstName firstName, PersonLastName lastName, PersonBirthYear year)
    => new Person(PersonId(id), PersonFirstName(firstName), PersonLastName(lastName), PersonBirthYear(year));
}
```

And Done!

## How to use it

It's refactoring time. Now we can initialize a Person class like this.
Actually, it's easy... Just drop the `new` keyword.

**Don't forget the static using!**

```csharp
using static MyAwesomeProject.Constructors

var person = Person(
  PersonId(1), 
  PersonFirstName("Clément"), 
  PersonLastName("Kiritchouk"),
  PersonBirthYear(1994))
```

## Bonus

Ok maybe you're not convinced yet.
Let's create a list of persons

```csharp
var persons = new List<Person>{
  person1,
  person2,
  person3
};
```
Why not creating a constructor for initializing List ?
Update your Constructors.cs file like this:

```csharp
// Constructors.cs
public static partial class Constructors
{
  public static List<T> List<T>(params T[] items) => new List<T>(items);
}
```

Now you can do

```csharp
var persons = List(person1,person2,person3);
```

You don't have to specify the type anymore because the type system infers it.

## Cons

Well, there is some cons using this technique.
- You have to create a constructor for every type while `new` is built-in.
- It's unusual, other developers might be confused looking at your code.

---

Anyway, I like it and I had to share it with you. Please tell me what are your thoughts on this. 😏
