# Welcome to the e2e Testing Meetup

## Setup instructions

In advance of the meetup event please have the following installed:

## Install Git

This can be a bit different depending on your OS but a good resource is [git-scm](http://git-scm.com/downloads).

## Install Node.js

Files and instructions can be found on the [Node.js website](https://nodejs.org/).

## Install CasperJS and PhantomJS

On a Mac OS using [brew](http://brew.sh/) - open a terminal and enter:
```
$ brew install phantomjs
$ brew install casperjs
```
On a Windows/Linux use the download page and install the [CasperJS package](http://casperjs.org/).  Be sure to add both to your path.

Confirm PhantomJS and CasperJS are installed on your machine; you should be able to get the following:

```
$ phantomjs --version
2.0.0
$ casperjs --version
CasperJS version 1.1.0-DEV at /Users/username/Sites/casperjs, using phantomjs version 1.9.2
...
```

Note: You may also wish to install SlimerJS (npm install -g slimerjs) however it is rather large and will take a while.
You may wish to use it after the workshop (however it is not critical to complete the workshop.
BEWARE prepared to download it with a speedy internet connection!


# Outline

## What are these tools

CasperJS - a navigation scripting and testing utility for PhantomJS and SlimerJS.  It provides a way
to take control of a browser and drive it via code.

PhantomJS - a headless WebKit scriptable with a JavaScript API.  Basically a browser without the
visual parts that humans use to view web pages.  Think of it as how Siri would browse the web...

## Casper API

Casper includes several modules: Casper, Colorizer, Mouse, Tester, and Utils.  We will focus on the two more
popular modules: Casper and Tester.

First up, the Casper module provides some basic remote control features (one of which enables scraping).

## Basic Scraping

Let's build a basic web scraping tool to familiarize ourselves with the main module.

Create a project folder and add create example.js file with the following content:

```javascript
var casper = require('casper').create();

casper.start('http://casperjs.org/', function() {
    this.echo(this.getTitle());
});

casper.run();
```

Then run casper:
```
$ casperjs example.js
```
And expect:
```
CasperJS, a navigation scripting and testing utility for PhantomJS and SlimerJS
```

## More Advanced

Update your example.js script and run it again using:

```javascript
var casper = require('casper').create();
casper.start().then(function() {
    this.open('http://registry.npmjs.org/casperjs', {
        method: 'get',
        headers: {
            'Accept': 'application/json'
        }
    });
});
casper.run(function() {
    require('utils').dump(JSON.parse(this.getPageContent()).description);
    this.exit();
});
```

Expect:
```
A navigation scripting & testing utility for PhantomJS and SlimerJS
```
Which is the in fact the description for the [CasperJS package](https://www.npmjs.com/package/casperjs) on NPM,
go ahead and check it out.  While you are at it, try changing the url to another website, to try scraping another
url.  Keep in mind that the script is currently expecting JSON, and note that the "description" object is explicitly
specified in our example.  You may need to make some adjustments as you scrap other pages.

Here are a few examples of urls that also return json data:
* http://jsonplaceholder.typicode.com/posts/1
* http://api.geonames.org/citiesJSON?north=44.1&south=-9.9&east=-22.4&west=55.2&lang=de&username=demo
* https://www.flickr.com/services/rest/?method=flickr.test.echo&format=json&api_key=d779846a3d1e2d8b97a1a561401f159b

## CasperJS is well documented

All of the [CasperJS modules](http://docs.casperjs.org/en/latest/modules/index.html) are well documented.

## Casper Test API

The Casper Test module is very similar, but has a few subtle differences.  First, construct the spec (test) file by
creating a folder and file tests/example.spec.js with the following:
```javascript
casper.test.begin('Google search retrieves 10 or more results', 2, function suite(test) {
    casper.start("http://www.google.com/", function() {
        test.assertTitle("Google", "google homepage title is the one expected");
        test.assertExists('form[action="/search"]', "main form is found");
        this.fill('form[action="/search"]', {
            q: "san diego code school"
        }, true);
    });
    casper.run(function() {
        test.done();
    });
});
```
and then you run the test using:
```
$ casperjs test example.js
```

Note that Casper needs to know that you want to run the test module, and by default it will look for a folder called
test, and attempt to run all files in the test folder that have the .js extension.

or you can specify the spec file(s), you would like to run:
```
$ casperjs test tests
```

Expect the tests to pass since there are more than 10 results in google when searching for San Diego Code School.

Google works!

But wait, I don't believe what I can't see.  Prove it...

Try updating your spec file with:
```javascript
casper.test.begin('Google search retrieves 10 or more results', 2, function suite(test) {
    casper.start("http://www.google.com/", function() {
        test.assertTitle("Google", "google homepage title is the one expected");
        test.assertExists('form[action="/search"]', "main form is found");
        this.fill('form[action="/search"]', {
            q: "san diego code school"
        }, true);
        this.capture('see-it-believe-it.png', {
            top: 0,
            left: 0,
            width: 800,
            height: 1200
        });
    });
    casper.run(function() {
        test.done();
    });
});
```
Then step back and admire the screen shot saved in your current working folder.

There are other command line flags you can pass.  Check them out:
```
$ casperjs --help
```

When you are first learning to use the test tool you may find it helpful to use the following flags

--verbose --loglevel=debug

or take lots and lots of screen shots.

Now that we know how to start a test, let's make some additional assertions and check the responses are
as expected.  Update your spec file before the casper.run function:
```javascript
casper.test.begin('Google search retrieves 10 or more results', 4, function suite(test) {
    casper.start("http://www.google.com/", function() {
        test.assertTitle("Google", "google homepage title is the one expected");
        test.assertExists('form[action="/search"]', "main form is found");
        this.fill('form[action="/search"]', {
            q: "san diego code school"
        }, true);
    });
    casper.then(function() {
        test.assertTitle("san diego code school - Google Search", "google title is ok");
        test.assertEval(function() {
            return __utils__.findAll("h3.r").length >= 10;
        }, "google search for \"san diego code school\" retrieves 10 or more results");
    });
    casper.run(function() {
        test.done();
    });
});
```

There is a lot going on here.  First we are telling Casper that we expect 4 tests to run.  This estimate will cause
your suite to throw error messages if it misses the expected numeber of tests (when too many or too few actually run).  Next,
after we start, we make a few assertions, and then use a built in function to pre fill the form.  Note that when you use the fill()
or click() methods you will need to use the async .then method to perform the next step.

In this case we envoke the casper.then(), make a few more assertions, including a special one "assertEval".

test.assertEval is a synchronous method, and it will evaluate the javascript passed into it immediately.  In this case
 we are able to do some DOM manipulation, and then use Casper's Util module to output the results.  You could also run other JS
 code inside your test, and then pass it back for a final assertion.

Try constructing a few tests and make some assertions using a few of the methods found here:
* http://docs.casperjs.org/en/latest/modules/tester.html

## Next steps

If you want to see what is happening as your tests run in a "normal" browser, you should use Slimer.

SlimerJS files and instructions can be found on the [Slimer.js website](http://slimerjs.org/).

Now go forth and test!!
