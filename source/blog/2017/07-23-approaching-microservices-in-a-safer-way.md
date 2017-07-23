@{
    Layout = "post";
    Title = "Approaching Microservices in a safer way";
    Date = "2017-07-23T18:00:00+00:00";
    Tags = "programming,architecture,design,microservices,services";
    Description = "";
}

Microservices. You've probably heard about them million times. It's inevitable to avoid this if you're at least a bit interested in software architecture. Actually, you've heard about them so many times that you have to try them for your own! But should you? Is it the right approach for your application? Will it solve any of your issues? Or will it be the source of a terrible headache? Let's try to get some answers to those questions!
<!--more-->

## Mythical Microservice

If you've heard or read about microservices in multiple places, you've probably already noticed one worrying thing - lack of precise definition of microservice. Of course, there are people telling you that microservice can only have up to 100 lines, or needs to be small enough to be rewritten within a scope of one week, sprint, or whatever. None of those "definitions" tell you anything; in some languages and frameworks 100 lines will be enough to get some functionality up and running; in others, it will take more than that to do just initial bootstrap. The same can be said about time scope - it doesn't say anything about team size, it's capabilities, knowledge of the topic etc.

It's basically up to you to decide what a microservice is in your environment. I've seen many examples, from ones as simple as single responsibility mail-sending service to large services capturing quite complex domain logic, exposing multiple endpoints and talking to plenty of other services. In my opinion, it really isn't so much about the size per se, rather about the scope. It really helps if you think about your system through the glasses of domain-driven design and its bounded contexts. Each of those contexts makes a perfect candidate for such a microservice. And even if you're not really focusing on incorporating full DDD approach, it may help a lot when looking through its lenses.

## Start simple

So, let's do the microservice thing! Yeah, it's easier said than done. If you're starting from scratch and don't have a strictly defined scope, you probably should start simple. Go with the monolithic approach, extracting only things which are totally separate from your core problem. Perfect candidates for first microservices would be things like interop modules, handling some legacy apps and data, or communicating with external systems. It should help you to embrace what it feels like to have more distributed architecture and get prepared for more.

## Modular monolith

In most cases, you should start with the monolithic approach. One thing you want to do in your monolith is to start thinking about separation of domains and grouping things by them, not by their technical properties. To give you an example, let's say you need to get some information about the products. Information may come from your database or from external service, doesn't really matter. You'll probably create some entities or DTOs to model what you're getting from an external source. You may have some repository or some network level service that would fetch the information. You also may want to use your application-specific view of that data by creating some domain or value object that will capture that data in a useful structure. Then you may want to create the mapper between your internal data structures and the external ones. Finally, you may want to create some service which will expose the functionality of fetching and mapping product data to other parts of your application. The key thing here is to keep all those things together - within one folder, module, namespace - whatever your language and frameworks support. Avoid grouping things by entities, DTOs, mappers, services and so on. By this approach, you'll get less coupling between unrelated components, and less coupling means fewer problems with moving things outside of your monolith.

For bigger chunks of functionality, it's often quite wise to create separate folders or projects, which will encapsulate all the code related to its domain. Make sure to create some kind of API for it - let it be an interface, for example - and use only it when communicating with that module. Be also very cautious when using data structures defined within that module - you may want to move them out some day, so don't rely too much on what's within that module.

## Feel the pain

While you're still not having microservice architecture, you should already feel the first pain. Keeping things separate means having some duplicated code, an additional mapping between data from different modules, and so on. It's inevitable in microservice world, and it's actually very intentional. The key here is to learn that code reuse isn't the golden principle in software development. Functionality reuse is the other story - and yes, microservice architecture puts focus on reusing bits of functionality, not bits of code. It doesn't mean you cannot reuse code at all; you may still have some shared code, but you'll need to make sure it's actually reusable. You may want to create some shared libraries to achieve it, and they are not much less of a problem than microservices themselves. Always think twice before you reuse anything - sometimes some copy-paste can save you a lot of time in the future.

## Prepare for the future

Another aspect which limiting code reuse affects is refactoring. You may think that it's easier to do refactoring when having more shared code, as you may do it in one round. But this isn't so easy with microservices. They are usually separate projects, and even if you have shared libraries, you may need to synchronize the changes in all of them which are using that library. And one day you may be fed up with the technology you use right now, or it may just become obsolete, and you may want to change it to something else. With limited code reuse and modular (if not microservice) architecture, it's much easier to use new frameworks, languages, or even platforms. You're already prepared for that, technically and mentally. And even if you want to stick with what you have right now, refactoring things within a module is actually much easier when it doesn't share too much code with others.

## Know when to extract

So it's the time. You feel it in your bones. But you need to have some logical reasons. How to tell that it's time to extract something into a microservice? Try to go through the checklist below to find some of the potential reasons:

 * Your codebase has grown so much that maintaining monolith becomes a pain.
 * Part of your application is used more (or less) often than others and you would like to scale it separately from other parts of your app.
 * Some parts of the code need to be deployed more (or less) often than the rest, and the deployment of the whole app is somehow painful.
 * You want to reuse the functionality that some piece of code provides in other applications, but you don't want to reuse the code (e.g. as a shared library).
 * Your team has grown and it's much harder now to work on one codebase without interrupting each other and causing conflicts.

As you can see, being prepared for potential split makes solving scaling problems much easier. And by scaling I don't only mean performance concerns - it may also mean scaling in terms of size, development process, team size or maintanance.

## Know how to extract

If you have already thought about the modularity of your application, it's much easier now to split things into chunks. But there are many concerns left still, just to name some of them:

 * How it should communicate with, or how it should be reached from other applications?
 * How to consume the service from other applications?
 * Where, when and how should it be deployed?
 * Who should be responsible for it?
 * How to handle bits of shared code that candidate for extraction is making use of?
 * Which technologies to choose for it?

There are lots of things to think about when transforming into microservice, so make sure you have thought about them before beginning the process. In a longer term, the microservice architecture may be extremely beneficial for you, but it may also become the source of serious problems, especially when you're not well prepared. It's up to you to decide if it's right or not - and sometimes the only way to find it out is to try it on your own. Just make sure you know what are you getting into.

## Some more concerns

Other things worth considering when moving towards microservices may include:

 * Deployment - how to configure the tooling?
 * Monitoring - how to make sure that the service is working properly?
 * Updates - how to handle service downtimes without breaking the user experience?
 * Backwards compatibility - how to make changes without breaking other parts of the system?
 * Size - how much should be extracted?
 * Growth - how much can be added to the service without causing additional issues?

And the list goes on, and on...

## Summary

Microservices may be the right solution for your system. They can solve some of the issues that are mostly unsolvable in the monolithic architecture. But they also bring their own bag of troubles that you have to deal with instead. I personally really appreciate this approach, but I can also see a lot of reasons to not do it or to do it in a limited scale. Getting my hands dirty with moving things out of monolith into services, creating new services, or even getting rid of some of them added a lot to my toolbox, and I'm really happy to share my experiences and thoughts with you. Hope you can find some value in them as well.

In future posts I would like to address some of the concerns (and some of the benefits) in a bit more detailed way, so stay tuned! If you have any questions or comments, feel free to post them here or on Twitter - it's alway great to hear what you think!
