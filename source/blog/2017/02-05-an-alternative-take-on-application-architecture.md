@{
    Layout = "post";
    Title = "An alternative take on application architecture";
    Date = "2017-02-05T23:00:00+00:00";
    Tags = "programming,architecture,design,csharp,fsharp,erorr handling,null,exceptions";
    Description = "";
}

Do you remember when you have started writing your first programs? It's very likely that the code you've written was very procedural and your carefully crafted instructions were then somehow executed by this magical machine called the computer. Wasn't it fun? It was a lot of fun for me, and if you're reading this then I can assume that it was the same for you. Yet many years later we look at our code and… yeah, we have to look deeper. And deeper. And even deeper. Go to definition. Oh, not that one, the other one. Oh, a little bit deeper. Yes! So here is the bug! Sounds familiar?
<!--more-->

The architecture of modern object-oriented applications is quickly becoming much of a pain. Dependency count is growing dramatically, followed by the complexity of those dependencies. Just take a look at the dependency graph of your application and answer honestly - how many surprises can you find there? I'm pretty sure that there will be a lot, especially if you're not the only author of that code.

In the team I'm working with right now this issue became a real pain. Making even simple changes in code required digging through tens of classes, and making a decision where to put new things was getting harder and harder. Fixing bugs was like fighting with Hydra - each fix introduced another set of bugs. And the problem was not only in the way the code was written but mainly in the overall architecture of the solution. The architecture which I've seen (and created!) multiple times, as it's almost exactly what *"good programming books" * recommend.

## The typical approach

So how typical application architecture looks like? My experience shows that usually we need to deal with multi-layer structure, where (in theory) each layer is responsible for different part of the job to be done. In MVC (or MVVM) applications it very often looks like this:

 * Controller captures request (or ViewModel captures user input) and based on it executes service method responsible for given task, transforming the input data into parameters acceptable by the service.
 * Service encapsulates business logic, using other services, providers, repositories, (…name a pattern here…) to get necessary data or to delegate some of the operations. After its task is done, it usually persists the effect using the repository, and/or returns the result back to the controller.
 * Controller (or ViewModel) transforms the result into output format and returns it to the caller (user).

Of course, this is a very simplified example, as most applications introduce more complicated logic into the process, including:

 * Input data validation,
 * User authorization,
 * Caching and other optimisations,
 * Logging,
 * External dependencies (especially in distributed architecture).

Problems usually appear quite early when you introduce any new thing into the process. The following questions may appear:

 * Should it be a dependency or another layer?
 * If it's a dependency, on which level should I put it?
 * Can I trust the results of the calls to dependencies?
 * Does the dependency throw exceptions if something goes wrong?

This is just tip of an iceberg and it only gets harder and harder as the application starts to grow. Of course, there are patterns and practices to be used which can mitigate those problems, but as the application grows it becomes more and more a mix of approaches and you can never be sure what to expect when you add something new.

## Flow-based approach

My team was facing those issues so often that we decided to sit down and try to take some different approach. We've taken pieces and blocks from different approaches out there, trying to create architecture that would be:

 * Easier to understand and follow;
 * Easier to extend with new processes (or flows, as we call them);
 * Focusing on composability;
 * Designed for failure, yet keeping error handling simple.

We took some lessons from domain driven design, encapsulating the internal state and exposing it by a reaction to external events (so keeping the service as the bounded context). Additionally, the limited trust approach when it comes to any external sources (and data coming from them) helped us to keep the service stable within the constantly changing environment. And we used a handful of techniques from the functional programming to make everything more concise and composable.

## How it looks like

The beginning is still the same - we get some external request that triggers the flow. Each external endpoint corresponds to one flow, helping to keep track of what's implemented where. Once you reach the flow, the magic happens.

The first thing which strikes you when you look at the code is how explicit yet simple it is. You don't have to dig into layers to see the steps needed to successfully execute the flow. In most of the flows one can find steps like:

 * Validate the input;
 * Transform it to internal object capturing the intent;
 * Gather dependencies needed to execute the intent (we're calling it context);
 * Check the validity of the intent given the context;
 * Execute the intent given the context;
 * Run any of the repeatable steps (in our case it's things like price update or summary recalculation);
 * Persist the result into database.

One thing that should strike you is the absolute lack of any direct error handling or even if statements throughout the flow. All of the calls are kept within the `Result` type and glued together using `bind` and `map` operations (oh, it sounds like a monad, scary!). That helped us to keep most of the dependencies afloat, making dependency graph shallow and easy to track (for injected dependencies the max depth is 3, for static dependencies it may go to 4 or 5 in some edge cases).

## Composing flows

Of course, the code very often gets more complicated than that. For example, when we create the context for execution we need to get the data from multiple sources, like database or external services. The nice thing about using composable types is that you can chain multiple calls using `bind` and make the code very brief and readable. For example, when you try to get something from a database and it will fail, getting additional information from other services usually makes no sense. In traditional OO approach, you could solve it by using constructs like

    [lang=csharp]
    if (dbResult == null) return null;

after each call. But it adds lots of clutter to the code and also leaves open questions like *"what should I return now?" * or *"how to name the next call result?"*.

By leveraging things like lambdas, anonymous types and extension methods we achieved a state in which most of the functions are basically expressions (and can be written without curly braces in C# 6!). Additionally, most of them are static, making them independent of the global or local state, only reacting to input with appropriate output. Yes, that's a pure function, and it has several benefits:

 * Not having any external dependencies, just input parameters, makes them easily testable.
 * Not having any side effects makes them easily testable.
 * Not dealing with the class state allows making them public without risks, which makes them easily testable.
 * As they are static and usually don't depend on context-specific types, reusing them (e.g. by extracting to common class) is pretty simple.

You see some pattern here? 

## Handling errors

We represent the result of any potentially failing call as the `Result` type. It's a very simple union type, which can be either a `Success` or a `Failure`. In F# you could write it as: `type Result<'a> = Success of 'a | Failure of Error`, where `Error` is a structure describing what went wrong.

Introducing `Result` type and making it almost a must for any impure function had several consequences. First of all, you can easily see from the function's signature that it may have side effects or inconsistent behaviour, thus it makes you more aware of any potential issues. Then you get into rules of writing functions:

 * If function can fail in any way, it must return `Result` type;
 * If function may encounter any exceptions, they have to be handled internally and wrapped into Result type;
 * Null is not a valid return value in most of the cases, rather use `Option` instead (aka `Maybe`);
 * Mark any external call results unsafe (even just by calling them DTOs) and don't use them directly (map them to internal representation with options instead of nullables).

Unfortunately, those rules cannot be enforced by linting or any other automated tool I'm aware of, so we need to keep it in mind when doing code reviews. This may be a bit of the drawback, but if you're doing in-depth reviews (and I think you should do!) then you need to dig into the logic anyway. It allows you to understand what and why is happening in the code and thus it feels natural to verify such things during the review.

Having such rules makes it easier to write the code, as you know if something may fail or have an unexpected result. As the result, you need to write less code (thanks to a set of helpers around `Result` and `Option` types) but at the same time handle errors gracefully.

## Decisions that we've made

Some of the decisions were almost no-brainers, but still, they are not the defaults for many apps:

 * Grouping classes in the domain (flow) specific namespaces.
 * Only creating reusable blocks when there are many repetitions of the same code and we're sure it should be the same for all of the cases.

Those simple things already helped to make the code more discoverable and easy to understand, but it was only a start. There were some more controversial decisions that we've taken which helped us to achieve the goal:

 * Keeping interfaces within the same file as the implementing class  - fewer files, less noise.
 * Using static dependencies all around (of course for pure functions only) - fewer dependencies, less noise.
 * Using tuples to pass multiple results within the flows - no need to create types for more elaborate results.
 * Using anonymous objects to pass structured data between subsequent steps of the flow - no need to create and name short living types.
 * Leverage the potential of `using static`  - make the code more concise and readable.
 * Use value types (structs) quite a lot, especially for the `Result`, `Option` and `Error` types - fewer nulls, fewer checks in code.

Although some of them were not easy to accept by others, we opted to give them a try. The code we're dealing with now is probably one of the best I've been working with (in an object-oriented language). It's easy to reason about, extend and test. Sure it would be better to use functional-first language and have much fewer problems to deal with, but being stuck with OO language boosted the creativity a bit as well. Hopefully, it will also be a big leap towards more acceptance and understanding of the functional programming in our team.

## Drawbacks

I mentioned few times improved testability as a benefit of this architectural approach, but there are some cases where it comes at some cost. For example, when you have static dependencies (e.g. function that calculates item summary based on a list of items) there is no way to mock them. Thus testing any function that uses static dependencies requires from you more careful approach about input data, because if the data will be in the improper state the call will fail. And because the architecture separates error handling from actual business logic as much as possible, there are functions which need to be executed in a safe context and don't do any validation within. We made it less of the problems by simple test helpers generating valid test data and then just modifying properties important in given test case.

Another drawback may be unfamiliarity of the code for any people outside of the team. Although architecture doesn't use any difficult concepts, it uses a bunch of terms from functional programming which may be a bit overwhelming for some. Still, if they are willing to learn, it should bring a lot of useful tools to their toolbox. It just requires a bit of time to get used to it, and maybe a bit of openness for the new concepts.

## Summary

There is no perfect architecture, but it doesn't mean we should stick to what we've got right now and not try to experiment. By going back to basics and asking the right questions we can iteratively make our solutions better. What are the biggest issues with current approach? How can we mitigate them? Is there a way to avoid some of the mistakes we've made before?

By being open for different approaches and combining it with your experiences you can improve the application architecture step by step. It may not be an easy thing to deal with for everyone and we had to face that issue in our team as well. But so far the overall result is quite promising. We're delivering feature by feature, with good quality code and good test coverage, while keeping on track with a very challenging deadline. The final test - enabling it on production servers - is still to come, but I'm optimistic about the results. The count of bugs discovered during the tests is quite low so far, and working with the code is much more of a pleasure than it is with the old solution. That sole thing is enough to say it was worth to take the risk.
