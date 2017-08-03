@{
    Layout = "post";
    Title = "Not so obvious benefits of Microservices";
    Date = "2017-08-03T23:00:00+00:00";
    Tags = "programming,architecture,design,microservices,services";
    Description = "";
}

Recently I wrote a bit about how to approach microservice architecture, but I haven't really mentioned why should you bother. It turns out that microservices can have some serious pros compared to monolithic architecture - but they won't be applicable in every case. There can also be some serious cons, but this will be a topic for another post, hopefully coming soon. So, let's start with the goodies.
<!--more-->

## The obvious ones

### Independent deployments

One of the core benefits of microservice architecture is the ability to independently deploy each service. That can not only speed up the development process but also help with fixing bugs, testing new stuff and so on. Of course, you will need to have some good deployment pipeline (with a possibility to revert deployments as well), but it's a must for microservices anyway.

### Scaling

The fact that one service runs independently from others allows you to assign resources independently. This is especially convenient with cloud deployments, where you can dynamicaly set resources available for given instance (scale up) and spawn more instances (scale out). Of course, the latter requires some load balancing at least, but this can be done relatively easy with what cloud providers offer. Those abilities can help you limit the costs while having a possibility to improve performance when more customers start to use your service. And the fact that you can do it independently for each of the services allows for more granular control.

## The less obvious ones

### Domain separation

When you're inside a monolith, it's quite easy to blend things together and forget about separation of domains. That leads to tighter coupling between components, and often to unexpected bugs related to that. When you're in a microservice, you're basically left with what you've got there. And as you shouldn't have much, there isn't much to mix with. Microservices are working really great with domain driven design, helping you to keep the separation between domains and communicate with other domains using event-based approach. And given the rule of thumb that each service should be responsible for its own data, it's much harder to break some other things by changing some commonly used object (whether it's a data structure, a database or an API). Also, the API of the service tends to be much smaller than the API of the modules within a monolith, which keeps people less tempted to use the things which they shouldn't. Think of microservice as a separate domain, a bounded context, or whatever label you'll put on it - just make sure it does what it should, and not more.

### Ability to try new things

How many times did you want to try this new approach, a new framework, or even platform or language? And how many times did you end up doing the same old thing again, because, you know, it's the way it's done and cannot be changed? Well, in the microservice world things are much easier. Usually trying some new approach or framework is a piece of cake - just pick one that you want when you create the service and you're good to go. Things get usually more complicated with platforms and languages. The former requires additional steps to ensure it can actually be deployed somewhere, monitored, maintained and so on. The latter requires some agreement across the team, but if there are people willing to go this way, you don't have blockers like cross-language compatibility which can be a pain in some cases. In general, the microservice architecture gives you more freedom of choice, and that is usually a great news - people want to learn and develop new skills and it's a great way to enable this.

### Continuous improvement

It's nice to do some bigger refactoring sometimes, but it can be very painful as well, especially when there are lots of things going on inside your monolith. When you're in microservice things get a little simpler. As you have a smaller codebase and fewer people are working on it, it's much easier to do any kind of improvements. Upgrading a dependency, doing some cleanup or changing the approach a little bit is less painful in microservice compared to the monolith. The smaller impact of those changes also means that it's safer to do it, so you're more willing to do it. That leads to more maintainable, better quality code, which is a great thing, especially when your codebase is growing and you cannot afford for bigger changes that will impact other parts of your system.

### Team separation

When your team grows it starts to be much harder to synchronize changes that different people are making. Code reviews, merging pull requests and deployments are becoming more of a pain. Tracking the changes that got into each release becomes harder, testing is not easy as well. If you want to have continuous deployment, things are even worse - when you constantly want to deploy monolith, the environment will be unstable most of the time. With microservices, you can make those changes with more gradual manner, and you can have smaller teams focused on each part of the system. That enables you to scale the development team, let them be responsible for given part and also build up devops culture. Anyway, every team member won't be able to grasp the whole of your system (at least on the code level) and letting people focus on smaller chunks can be more effective and less stressful.

### More completed projects

And when we're speaking about the people, there is one benefit that will also help build up the team morale. People like accomplishing something - finishing tasks and projects can be such an accomplishment. With microservices you can have smaller projects which quite often can get into "completed" state, making people feel that they've finished something. Of course, there may be some maintenance work, but, when managed well, microservices can bring a more positive attitude to your team. And a happy team is one of the best things that can happen in a software development world.

## Summary

As you can see, there are quite a few benefits of using the microservice architecture. Some of them are easy to overlook, so it's good to sometimes look at the approach from a broader perspective. But of course microservices are not a bed of roses and there are several challenges to take as well. In the next post I'll try to list the most important problems, but for, now let's not forget the serious benefits I've mentioned above. Hope they will make you more willing to try it out by yourself!
