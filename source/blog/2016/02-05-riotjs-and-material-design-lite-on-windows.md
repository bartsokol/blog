@{
    Layout = "post";
    Title = "Riot.js and Material Design Lite on Windows";
    Date = "2016-02-05T23:00:00+00:00";
    Tags = "web javascript css html5 riot.js mdl windows";
    Description = "";
}

Recently I wanted to build a small web app for my own purposes (more like an experiment) and I was thinking how to approach it. For whatever reason I decided to go with SPA-like approach, having simple HTTP service as a backend and some simple yet powerfull enough web app as a frontend. And as you probably know, the choice now is overwhelming - think Angular, React, Polymer, and so on. All of those libraries give you a lot of possibilities, but also come at cost. Not only they are heavy to download, but also they have quite steep learning curve and bring a lot of stuff which you'll never use in a simple app. In the search of alternative solutions I encountered small library called [Riot.js](http://riotjs.com/), introducing itself as "React-like user interface micro-library". Quickly looking at some samples and docs I decided to give it a try. And as some things weren't as simple as I expected, I decided to put together a quick start guide with solutions to the issues I've encountered. So here you are :)

<!--more-->

## Installing required tools

As with any JS framework nowadays you need to start with Node.js and NPM*. Download latest stable version from [Node.js website](https://nodejs.org/) and install it on your system. Once you're done, open console and we can start installing stuff. We need to install [Bower](http://bower.io/), Riot.js and simple HTTP server to make things easier:
```
npm install -g bower
npm install -g riot
npm install -g http-server
```
** Some side note here - most of it is not needed; you can download everything from Riot site, but it won't be so cool, so you'd better stick with proposed approach*

## Bootstrapping the project

Once ready, you can start bootstrapping your project. Create some folder where you're going to keep your project, then navigate to it in your console window and type next magic command:
```
bower install riot --save
```
You should have now `bower_components` folder with Riot library inside. You can now create index file in your editor of choice (if you have no idea what to use, give [Atom](https://atom.io/) or [Brackets](http://brackets.io/) a try). Create `index.html` file with HTML5 document structure and start adding required references. At the bottom of your `body` tag add reference to Riot.js:
```
<body>
	…
	<script src="/bower_components/riot/riot.min.js"></script>
</body>
```
There is also riot+compiler version available, but given the style library I will offer you to use it is better to stick with the minimalistic approach.

## Your first Riot tag

Once you're done with the previous steps, you can start adding your first tag. Create a folder you're going to use to store your tags in (I'd go with `tags` for purpose of this guide) and create a new file. Name it as you want to name your tag (not a requirement, but seems to be a good choice) with `.tag` extension, keeping in mind that recommended naming scheme is two-part: use names like `login-form`, `login-failed`, `item-list`, `item-details` etc. Again, not a requirement from Riot, but recommendation from W3C, so better stick with it.
Inside your new file start creating your new tag. If you decided to create `item-list`, wrap everything in `<item-list>` tag:
```
<item-list>
	…(your HTML code goes here)…
	…(and JS code below it)…
</item-list>
```
And save it as `item-list.tag`. Your first custom tag is ready! Well, almost. You need to use it somehow on your page. Navigate to `index.html` file in your editor and add a bit of plumbing code there:
```
<body>
	…
	<script src="/bower_components/riot/riot.min.js"></script>
	<script>
		riot.mount('item-list')
	</script>
</body>
```
This will mount the tag into your document's DOM. The only thing left is to somehow translate your tag so it can be actually used on your page. And here we encounter the biggest issue.

## Finding the right command

All Riot.js tags need to be compiled before they can be used on your page. You can just use version of Riot.js with compiler built in, but it won't work well with many style frameworks, including MDL (more on that later). So, let's go with compilation - shouldn't be hard. Let's try:
```
riot tags/item-list.tag
```
Mission failed. At least for me. Command not found. Nah, not the `riot` command, it is present in path. The issue is inside `riot` command, which is trying to run `sh.exe` - which is not present on my system. So, what can we do now? We have to go deeper. In the installation folder for Node modules you need to search for riot module and check if you have the right command inside. For me it was found here:
```
C:\Users\{username}\AppData\Roaming\npm\node_modules\riot\node_modules\.bin\riot.cmd
```
This one uses node.exe to run Riot JavaScript code, and it's working fine. So you can stick with it, create some script which will compile it, create some Gulp task or whatever - just remember to point it to right script*. Once your tag is compiled, you should get `item-list.js` file in your `tags` folder. If yes, then let's move to next step.

** Yes, I should probably raise an issue (or even PR) to Riot.js to solve it. Maybe later. Or maybe you can do it? :)*

## Including your tag in your code

So, you have compiled your first tag and it's now time to use it. Modify your `index.html` file by referencing compiled tag file and putting your shiny new tag somewhere in your (HTML) body:
```
<body>
	…
	<item-list></item-list>
	…
	<script src="/bower_components/riot/riot.min.js"></script>
	<script src="/tags/item-list.js"></script>
	<script>
		riot.mount('item-list')
	</script>
</body>
```
Now let's see if it works - you can use `http-server` command installed earlier to serve the files and view them in browser. By default it should start listening on 8080 port, so once you run it navigate to your browser and open `localhost:8080`. You should see your brand new tag displayed on the page. If not then check all the steps above again ;)

## Adding some style with Material Design Lite

Material Design Lite (MDL) is CSS/JS style framework created by Google, implementing their design language called (ta-da) Material Design. It's maybe not so comprehensive as Bootstrap or other great frameworks, but it provides some high-quality components to build great looking websites utilizing this neat design language. Starting is very simple - you just need to add references to MDL to your HTML file, either from CDNs or from local source. For local development I prefer to have everything locally - you can do it by installing MDL with Bower:
```
bower install mdl --save
```
Now you can add it to your `index.html` file, CSS in `head` section, JS in body:
```
<head>
	…
	<link rel="stylesheet" href="/bower_components/material-design-lite/material.min.css">
</head>
<body>
	…
	<item-list></item-list>
	…
	<script src="/bower_components/material-design-lite/material.min.js"></script>
	<script src="/bower_components/riot/riot.min.js"></script>
	<script src="/tags/item-list.js"></script>
	<script>
		riot.mount('item-list')
	</script>
</body>
```
Now you can use MDL classes in your elements. Navigate to [MDL website](https://www.getmdl.io/) for details how to use it. Once you save your changes, remember to recompile your tags each time you change them! One of the solutions for this is to use [riot plugin in Atom](https://github.com/jumilla/atom-riot) which offer auto tag compilation on save. You can also use some file watchers (e.g. using Gulp) to do it for you.

At this point I should also say few words on why you should compile everything manually instead of simply using riot+compiler script. Unfortunately because of the way MDL is constructed (and some other frameworks as well) it doesn't handle well dynamic DOM, which is how riot+compiler works - it generates DOM objects on the fly. Without precompiled tags many JS-based functionalities provided by MDL just won't work, which will negatively impact both visual and functional aspects of your application. I think the compilation step is not so hard once you set up everything, and MDL seems to be worth the effort :)

## Summary

Now you should have simple yet powerful startup point for your next website or webapp. Setting everything from scratch should let you understand how it works and customize many aspects of your app with ease.
Using two simple and light libraries you get a lot of things for free, without a lot of effort or cost of investing in larger frameworks. You get modularity, easy management of components and separation of styling from functionality. For sure it won't be solution for every web app you'll want to create, but for small to mid size solutions it should be quite a good choice. And as almost everything is build using very basic tools and in simple manner, moving to some other frameworks in future should be also pretty easy. That's why I praise minimalistic solutions - they let you focus on the purpose of doing things, not the way how to do it.