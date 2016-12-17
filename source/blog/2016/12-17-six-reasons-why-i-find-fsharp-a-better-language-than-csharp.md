@{
    Layout = "post";
    Title = "Six reasons why I find F# a better language than C#";
    Date = "2016-12-17T14:20:00+00:00";
    Tags = "programming,language,csharp,fsharp,erorr handling,null";
    Description = "";
}

I was thinking about this post for a while. The more I use F# and speak about it to others, the more often I am asked the same question: what are the advantages of using F#? It's not so hard to find those, but I wanted to have a list that is more like fact list and less like personal preferences. Let's see what I came up with!
<!--more-->

*First of all, a disclaimer. This is my personal opinion based on my experiences with both languages. I'm not trying to start any flame wars - I'm looking for strong facts, but they may not be same strong for everyone.*

## 1. Record types
I think there is more or less agreed among developers now that immutability is one of the desired properties for data structures. Sure, there are some cases where mutable types have some advantages (like performance critical code), but those cases are quite rare among the vast amount of code that we write every day. Additionally, there are lots of optimisations made on compile time and in the runtime that make performance impact of immutability almost neglectable, and even in some cases immutable data structures can be more performant (e.g. when doing comparisons between instances).
But having immutability alone is not enough to be able to work in an effective way. We need to have additional tools to work with immutable types. One of those is the ability to easily create clones with one or more properties changed. This is where F# record types really shine. Let's look at this example:

    [lang=fsharp]
    type Person = { Name: string; Age: int }
    let john = { Name = "John"; Age = 27 }
    let olderJohn = { john with Age = john.Age + 1 }

That's all you need to define the type, create an instance and clone it when using F#. Now let's try to write equivalent C# code:

    [lang=csharp]
    class Person
    {
      public Person(string name, int age)
      {
        Name = name;
        Age = age;
      }
      public string Name { get; }
      public int Age { get; }
    }

That's just immutability, and we already have much more code. How to add cloning and updating values? Well, cloning is pretty easy, we can for example use `MemberwiseClone` method from System.Object, but we would need to implement it in every type we want to have this. The much harder part is updating the values. In this extremely simple case we can just create new object and copy properties from one to another, but what if we had not 2, but 10, 20 or more properties? Each time we would need to update even one property we would end up with a huge amount of code. I can't really think of an easy way to do it in the current version of C# (apart from some utils using serialization or reflection but that would be ugly and not really performant approach). There are plans to fix it in newer versions of C#, but until it's live we're stuck.

## 2. Collections
Immutable records are not the only thing we want to have - we also need to have immutable collections to be able to have fully immutable flow. And while creating an immutable version of existing collection is relatively simple in C# (having things like `ReadOnlyCollection` and `AsReadOnly()` LINQ extension), getting updated version with something modified is much harder. In F# you can write:

    [lang=fsharp]
    let items1 = [0;1;2;3;4;5]
    let head::tail = items1
    let items2 = head::tail

Extracting head from the list (or array) is extremely easy, the same is for prepending it to the list. Of course, more complicated cases require the use of other tools like `List` or `Seq` modules, but it's there and it's easy to use. There are libraries for C# that make it possible as well - but it's another dependency you need to have in your project, and they are not so deeply integrated into the language.

## 3. Discriminated unions
While records are great for keeping data that belong together, it's not the only case we can encounter when implementing some domain logic. I very often find myself in a situation where I want to have a data structure which will contain a different set of information for different cases. Classic object-oriented approach for this would be hiding the implementation behind a common interface and then using a plethora of design patterns to get the right information from the object.
Let's see how it looks based on a common example: validation. We are getting some data structure which we need to validate according to the set of rules. What we need as an output is some data structure that will describe us what is the result of the validation. This is an example implementation of this in F#:

    [lang=fsharp]
    type ValidationResult =
      | Passed
      | UnrecognizedProduct of Product
      | IncorrectPrice of Price

For simplicty I'll just add few results, but the list can be longer. Let's see how this can be done in C#:

    [lang=csharp]
    interface IValidationResult
    {
        bool IsPassed { get; }
    }
    class Passed : IValidationResult
    {
        public bool IsPassed { get; } = true;
    }
    class UnrecognizedProduct : IValidationResult
    {
        public bool IsPassed { get; } = false;
        public Product InvalidProduct { get; }
    }
    class IncorrectPrice : IValidationResult
    {
        public bool IsPassed { get; } = false;
        public Price InvalidPrice { get; }
    }

I kept formatting brief, but in real code this will be many more lines and potentially even files. Can you see a problem here? We don't have any common structure for all the cases, as they may contain different set of data depending on the kind of the error. But that's not the only problem we have - let's try to translate this result to user friendly message.

## 4. Pattern matching
To display the right error message we can use pattern matching in F#:

    [lang=fsharp]
    let getTranslation key = …
    let getErrorMessage = function
      | Passed -> getTranslation "passed"
      | UnrecognizedProduct product -> sprintf (getTranslation "unrecognizedProduct") product.Name
      | IncorrectPrice price -> sprintf (getTranslation "incorrectPrice") price.Gross

In C# I can see two approaches. One could be type-checking the result in method full of if's:


    [lang=csharp]
    public string GetErrorMessage(IValidationResult result)
    {
      if (result == null)
        …handle null param somehow…
      if (result.IsPassed)
        return GetTranslation("passed");
      if (result.GetType() == typeof(UnrecognizedProduct))
        return string.Format(GetTranslation("unrecognizedProduct"), (result as UnrecognizedProduct).InvalidProduct.Name); 
      if (result.GetType() == typeof(IncorrectPrice))
        return string.Format(GetTranslation("incorrectPrice"), (result as IncorrectPrice).InvalidPrice.Gross);
      …
    }

The problem with this approach is visible at the end of the method. What to do for other cases? We don't have them yet, but they may be added in the future. With this code you won't get any warning if new implementation of `IValidationResult` is added to the code, and you need to either ignore it (by returning null, empty string or some generic error message) or throw an exception. Both are not really good ways to keep the code up to date. Can it be done in other way?
We could change the definition of `IValidationResult` to have a method which will format the error message:

    [lang=csharp]
    interface IValidationResult
    {
        bool IsPassed { get; }
        string GetErrorMessage(string translation);
    }

And then the code could get much simpler:

    [lang=csharp]
    public string GetErrorMessage(IValidationResult result)
    {
      if (result == null) { …handle null param somehow… }
      return result.GetErrorMessage(GetTranslation("unrecognizedProduct"));
    }

This is a much safer way to do it as at any time you'll be adding new validation result you'll have to add error formatting logic to it. But there are still some issues with this C# code. One is that we probably need to handle the null result case somehow (which we don't have to do in F#). The other problem is on much higher level - any time you need to check something based on the IValidationResult you would need to extend this class. Imagine you have such logic for a product which has plenty of fields and even more logic to handle different kinds of products etc. You'll end up with huge classes containing both the data and the logic. That would make tracking the logic across the products extremely hard because you need to check it in multiple implementations of the interface. For me, it's very cumbersome way of doing it cause I find code with separation between data structures and logic much easier to understand and track compared to heavy models.

## 5. Error handling
Union types have another advantage when it comes to handling errors. There is a common pattern used in F# (and other languages as well) which can be illustrated with this simple type:

    [lang=fsharp]
    type Result<'TSuccess, 'TFailure> = Success of 'TSuccess | Failure of 'TFailure

Having such simple type that allows you to say whether the action succeeded or failed is a great benefit. Not only your output type is more clearly stating that some operation may not succeed and you need to handle the failure, but also you have a very clear way of saying what exactly went wrong. Combine it with simple `bind` and `map` functions and you are on the right track - you can start doing Railway Oriented Programming. This deserves a post or even series of its own, but believe me - this is a game changer when it comes to error handling and heading towards error-free code.
Of course, you can define the similar type in C# (and I would recommend you to do it and start using it) but without pattern matching, partial application and pipe operator it's much more complicated to use. Having defined bind and map functions will help a bit, but you just can't avoid the amount of code you need to write.

## 6. Safe defaults
Having a nice way of handling errors is a great thing, but it would be even better to prevent them in the first place. That's where design decisions come in - F# defaults are safer than the ones in C#. Immutable records, collections and bindings (aka "variables"), non-nullable types by default and things like `Option` type (Maybe monad) baked into the language and standard libraries give you the power to write code which will be free from `NullReferrenceException` and other C# nightmares. Sure, writing very defensive code in C# can lead you to the same, but it will be at the cost of much more code you need to write.  And the more code you have, the bigger is the chance of having errors. Not to mention that readability and maintainability of the small codebase are much higher. With powerful type system, great type inference and language constructs like pipe and function composition operators, F# offers a lot that C# just cannot give, even in the upcoming (7) version.

## Summary
There are many lessons to be learned from using F#. But the biggest one for me is that C# is no longer the preferred tool when it comes to writing any kind of code. The amount of code you need to write, the unsafe defaults and lack of powerful language constructs makes C# much less appealing than it used to be. The more I experience F#, the more I find it the better choice for general purpose programming. And I would recommend you having your own ride, especially if you're a .NET developer - all the tools you need you probably already have on your machine. And I can guarantee you one thing - it will be cool and safe ride at the same time!
