@{
    Layout = "post";
    Title = "Not so short story of the software testing";
    Date = "2014-10-19T10:37:48+00:00";
    Tags = "";
    Description = "Today I'll tell you a story about testing.";
}

Today I'll tell you a story about testing.

<!--more-->

## The story begins

Some long, long time ago human race started to code. Everyone was happy that the software they're writing was working (well, in most cases it wasn't, but let's skip it). So they kept writing even more software, and begun creating _systems_, not _only_ programs. And then, one day, someone asked the programmer: "How can I be sure that the system you've developed works as expected?". "Well, you can test it.", he answered. And the whole story of software testing begun.

## Evolution

First, it was all manual. Someone just thrown few numbers into system and checked if output was correct. Then someone discovered that he can program the same, and the first automated tests were there. They would probably be called now _integration tests_, as they were crossing the whole system, not only one piece of software.
  
As systems got even more complicated, developers introduced _layers_, and discovered that each layer can be tested independently. By coincidence, object-oriented programming was on everybody's tongue, so someone said "Hey, let's test just one class at once!". And we arrived at the very beginning of every modern programmer's testing experience - writing _unit tests_.

## Expansion

So, now you test _each and every_ class in your code. You've got perfect code coverage figure. And you run your software, and bang! It explodes. Database has change since the last time you've checked. So, as an experienced developer, you go and write integration tests (sounds familiar?). Then to make sure your perfect UI (it's always perfect) works well too, you write _coded UI tests_. And now you think you can sleep well.

## Bad things happen

Until one day critical bug is found on production environment. You wake up at night, write _regression test_ to verify it and fix it ASAP. Once again, tests saved you and let you have less nightmares.
  
But the customer is always unhappy. With your latests release performance of the software degraded drastically and everyone asks why. As you're now an unquestioned expert in software testing, you sit down and write _performance tests_ that measure each and every important piece of code, compare the results and tell you where is the bottleneck you have to fix. Once again you are a hero, and the world is saved, they don't have to wait 500 ms for search results. Now it only takes 400.

## What else should I know?

There are plenty of tests that you should be aware of. Smoke tests that let you quickly judge software environment stability; security tests that cover most common security flaws and let you know how unsecure your system is; and a whole bunch of other manual or automated tests that can be proceeded.

## Wait, manual?! I can write a code!

Yes, manual. Manual testing is as important as automated (or even more). There are millions of things that computers can't notice, things that cannot (and shouldn't) be automated. Some may say that everything can be coded; the answer is yes, but what's the cost? Is it worth automating _everything_?

## But hey, I'm a developer, not tester!

You are, and your responsibility is to make sure (and prove in many cases) that what you've developed is highest possible quality and works _as expected_. Of course, there should be some QA team that would help you with this, but remember that they are to _support_ you, not to _replace_ you. I would cover more on each team member responsibilities in one of the next posts.

The story end here for today. Hope you've enjoyed it and you would start thinking more about the quality next time you estimate your task. The road to glory is here long and bumpy, but don't be afraid! The world would be thankful!