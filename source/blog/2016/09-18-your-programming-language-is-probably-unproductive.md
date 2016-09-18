@{
    Layout = "post";
    Title = "Your programming language is probably unproductive";
    Date = "2016-09-18T21:20:00+00:00";
    Tags = "programming,language,csharp,java,c,fsharp,ml,ocaml,elm";
    Description = "";
}

It's really hard to create a good programming language. There are a number of aspects you need to think about, and you'll probably never reach the perfection. Still, there are some things you can take into account to make the code maintainable and easy to use and understand by humans. And similarly, there are things that make the code much harder to read and use effectively. Let's get through some of them.
<!--more-->

## Braces
There is one thing that is common to most of the programming languages people currently use. They all take from the same design that was introduced a number of years ago, a design that allowed computers to easily understand more abstract instructions made by humans. The design which was created to easily create compilers for basically any platform out there. That design was implemented by a structural programming language called C.
One of the key aspects of that design was an intensive use of all kinds of braces. They allowed to easily create starting and terminating points for all kinds of artefacts appearing in code, like blocks or function parameters. The idea was brilliant and it works really well even today.
But what's good for computers is not always the best thing for human beings. Bytecode anyone? The introduction of braces in the code was a trigger for a number of religious wars, and it made us, developers, focus on the design of the language instead of focusing on the most important thing in our work - solving real problems.

Of course, there is a number of fields on which developers fight, some of them being fundamental, but vast of them are purely religious. While dynamic versus static typing can have a severe impact on how our programs will actually work, the location of an opening brace is definitely not that important. Moreover, the sole existence of braces as block delimiters makes even more harm. We are used to reading texts in our natural languages, and in those texts, we barely use any kind of braces. Additionally, they make some great capabilities of "braceless" languages like partial application much harder to implement. So, what's the best way to separate one block of content from another?

## White space
You know that. You see that every day in your code. People putting empty lines just to make one part of code distinguish from another. We all use indentation to mark blocks of code readable. Yet most of the languages force us to use braces almost everywhere. And while some of them allow omitting braces in some cases, many devs are so used to them that they put them basically everywhere. It ends up with code which has a ratio of meaningful code lower than 50%. Rest of it is pure boilerplate and don't give any value at all. Just look at the simple example:

    [lang=csharp]
    Product GetProductById(int id)
    {
        if (id <= 0)
        {
            return null;
        }

        return _productRepository.Get(id);
    }

And compare it with:

    [lang=fsharp]
    let getProductById repository id : Product option =
        if (id <= 0) then None
        else repository.Get id

It's a kind of code you'll probably find in any LOB application. Forget for a moment that it's an actual piece of software. Think about it as another paragraph of this post. Now try to read the body of both functions again. Which of them feels more natural?

The result may be a bit skewed because we are taught the first approach from the first days we start to learn programming. For most of the developers, the very first languages they interact with are ones like C, C++, Java, C# or JavaScript. The most popular, the most mainstream ones. Yet those are ones which have the most flaws in their design. And it's not only about braces or whitespace usage.

## Verbosity
Initially, code written in C tended to be quite concise. Small screens with small resolutions and small storage capacity forced developers to be very scarce on the amount of code they write. They used short variable names, acronyms and were trying to reuse what they had as much as possible. But as programs started to become bigger and bigger, this approach made them less and less readable, and eventually people started to be more elaborate in their code. The evolution of computers made it even more feasible, and people started to abuse the new capabilities. Languages like Java and C# made it to the almost ridiculous point where not only we have to be very explicit about every aspect of our code, but also made us split this code into an endless amount of files. And if we add to it the rise of design patterns and multi-layered code, we end up with code with basically mirrors the structure of the organisations we work for - corporations. We reached the point where form has overtaken content, where processes are more important than the actual purpose. We made the same mistake again.

## Getting back to basics
For some time I got really fascinated with that beautiful, elaborated form. I enjoyed order, explicitness, curls of the block brackets. It's really tempting. But it also points towards nonsense. We start to treat the code as a piece of art, not a piece of engineering. And while a good design and well thought-through form help us to engineer better and more maintainable solutions, it cannot be our main focus. As in school of Bauhaus, the form needs to serve human, not the opposite. And if you look around you'll find out that it's a key to the success of many companies - the right amount of design backed by really clever engineering.

## Even more improvements
So how can we make our code better and effectively make our software better? Think about using right tools. As programming languages are the basic tools in developers toolbox, think what you put in yours. Let's take a look at what people who are not software engineers use. There are languages which are very domain specific, like R, which was created by statisticians, for statisticians. In many aspects, it's totally different from the languages software developers use. Even with some real flaws, it's extremely successful in its field.

Then there are two more general languages that didn't go the C way, and in effect are very popular among people who aren't really software engineers. First one is Python, which is used very heavily in data science, but it's also very popular in web and game development and IoT areas. The second is Ruby, which was first programming language for many developers with a non-technical background. Both languages are very successfully used by initiatives like Django Carrots or Rails Girls and turn out to be very effective tools for learning programming. Their simple syntax and dynamic nature help people to focus on their intentions instead of on the form.

## Going functional
While both Python and Ruby are great tools for relatively simple solutions, their object-oriented nature has some serious drawbacks when it comes to more complicated solutions. That's where functional programming comes into the way. There is a whole family of languages which have a really effective design, and they are all functional languages. It's the ML family, which started with (surprisingly) ML language, whose most recent and actively developed ancestor is Ocaml. There are also languages which more or less take inspiration from the family, and you'll find Haskell, Elm and F# among them.

The first one is one of the purest functional languages, sitting on the top of the Ivory Tower of functional programming. And while it's great for learning functional programming, its applications are also quite limited.

Then we have Elm, which is gaining some big momentum right now, and there are reasons for that. It compiles to JavaScript and allows you to write purely functional applications that will work in a web browser. It's famous for its reliability and performance, hence if you're into web development you should definitely try it.

And then we have F#, which is general-purpose, functional first and strongly typed language that is primarily running on .NET platform. It can be also used to generate web applications in HTML and JavaScript, using tools like WebSharper or Fable.

All of those languages are quite easy to learn and help developers write concise and effective code with high maintainability and readability scores. Their simplicity end effectiveness is an inspiration for other language designers, and we can see concepts from those languages slowly appearing in other languages like C# or Java.

## Summary
Language design and form of the code shouldn't be the things developers put their focus on. Languages that have a simpler syntax and are primarily designed for humans allow us to focus on the purpose of what we write. Getting rid of unnecessary things from the code, along with proper use of whitespace, makes the code more concise and readable, and thus more maintainable. And the fewer things are out there, the less time people will spend on discussions on form, and the more they will be focused on their main goal - solving other people's problems.
