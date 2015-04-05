Crash
=====
v.1.0.0

Crash performs optimized 2D collisions, powered by [RBush] and [SAT.js], written in javascript.  
It's most obvious use-case is in game engines, but it's flexible enough to be used anywhere.  
Crash is perfectly happy in the browser and on Node.js.


## Contents
* [Contents](#contents)
* [Installation](#installation)
  * [Node.js](#nodejs)
  * [Browser](#browser)
  * [Require.js](#requirejs)
* [Getting Started](#getting-started)
  * [Adding Colliders](#adding-colliders)
  * [Testing for collisions](#testing-for-collisions)
  * [Unleashing the power of Crash](#unleashing-the-power-of-crash)
  * [What kind of sorcery is this !?](#what-kind-of-sorcery-is-this-)
  * [But what's up with that `moved()`?](#but-whats-up-with-that-moved)
* [API](#api)
* [Contributing](#contributing)
* [License](#License)


## Installation
At the moment, package managers are not yet set up. Just download the `crash.js` file from this repo and load it in your project.  
When you have installed Crash, head over to the [Getting Started section][getting-started].

### Node.js
Add the following snippet to your code:
```javascript
var Crash = require("path/to/crash.js");
```
Now, you can use the [API] on the `Crash` variable.

### Browser
Add the following snippet to your HTML file:
```html
<script type="text/javascript" src="path/to/crash.js"></script>
```
Now, you can use the [API] on the global `Crash` variable (`window.Crash`).

### Require.js
If you're using [require.js] in your project, use the following snippet to load Crash:
```javascript
define(["path/to/crash"], function(Crash) {
    // Your code...
});
```
Now, you can use the [API] on the `Crash` variable in your module.




## Getting Started
Before you can do anything useful with Crash, you have to initialize it. You can do this easily by calling:
```javascript
Crash.init();
```
This method will make sure that RBush is being initialized correctly, so Colliders can be added.  
`Crash.init()` accepts one argument, a number called `maxEntries`. This is specific to RBush, so I refer to their [documentation][rbush-docs]. This argument is not required, though, so just leave it out for now and use the default value.  

> __Fun Fact:__ Actually, when `Crash.init()` has not been called yet, a lot of things just work, and nothing should break. You can use the full API safely, although some things may not work correctly (specifically those that need rbush). Although it should be safe, it's a good habit to call `Crash.init()` before you do anything.


### Adding Colliders
Now that everything is ready to roll, let's add some colliders. All colliders in Crash inherit from the `Crash.Collider` class, which provides some basic methods that perform household tasks, like moving, updating its AABB and testing collisions. That would lead us too far, though, so I refer to the full [API docs][API] for more info.  
All that stuff is awesome, but a `Collider` on its own is not very useful: it doesn't have a shape. Before you can use a Collider, you have to give it a shape, but, luckily, Crash has some built-in ones for us. Let's try them out!

```javascript
var point =   new Crash.Point  (new Crash.Vector(0,0));
var circle =  new Crash.Circle (new Crash.Vector(5,2),  10);
var box =     new Crash.Box    (new Crash.Vector(-40,0), 10, 15);
var polygon = new Crash.Polygon(new Crash.Vector(3,7),  [new Crash.Vector(0,0), new Crash.Vector(5,0), new Crash.Vector(2,3)]);
```

Wow, what's all that!? Let's clarify this step by step.

1. Each shape has its own constructor, so a point is initialized with `Crash.Point` etc.
2. The first argument to each constructor is a Vector, setting the base position for the Collider. So, for a Point, this would be its position, for a Circle it would be the center and for a Box it would be the bottom-left corner.
3. Some constructors take a few extra arguments: 
 * Circle: the radius
 * Box: the width and height
 * Polygon: an array of Vectors, relative to the base position
4. All the above arguments are required.
5. These constructors also take two more (optional) arguments:
 * insert: a boolean indicating whether the collider should be inserted into RBush. More info on this is following in the next few steps.
 * data: some data to add to the collider. This can be anything you want and doesn't do anything for Crash; it's just there for your convenience.
 
> __Fun Fact:__ the position of a Box isn't necessarily the bottom-left corner, it can actually be any corner, as long as you use it consistently. The bottom-left corner is the most obvious choice, though.
 
> __Important Fun Fact:__ there is a very important difference between Point and Vector: a Point is a Collider, so it can be used for collision checks. A Vector, on the other hand, is just a thing that defines positions in Colliders, like the center of a Circle or the corners of a Polygon.

> __Yet Another Fun Fact:__ the unit you use for the numbers is completely up to you. Crash only stores the numbers, you can interpret them as you wish, so you can use pixels, millimeters or even some game-specific unit you invented!


### Testing for collisions
Now that we have some colliders, we would probably like to know if they are colliding. To do this, we use the `Crash.test()` method:

```javascript
if(Crash.test(circle, box)) {
    alert("Oh my, there is a collision!");
}
```

This is already quite nice, but not very useful: now we know that our colliders are touching, but we don't know how to undo this crash (pun intended)! Enter `Response`, the all-knowing crash guru.  
First, let's create one:

```javascript
var res = new Crash.Response();
```

The Response class is copied from SAT, so for further information about the kind of info it provides, I refer to the [SAT.js docs][sat-docs].

Then, let's do a subtle change to our testing code:

```javascript
if(Crash.test(circle, box, res)) {
    alert("Oh my, there is a collision!");
}
```

Now, we can query the Response for some useful information and undo this embarassing crash:

```javascript
if(Crash.test(circle, box, res)) {
    alert("Oh my, there is a collision!");
    var overlap = res.overlapV;
    circle.moveBy(-overlap.x, -overlap.y);
}
```

And just like that, our colliders aren't touching anymore!

> __Fun Fact:__ you could also use circle.test(box, res). This just abstracts `Crash.test()`, but it's a little more concise.



### Unleashing the power of Crash

Wasn't that exciting!? Just wait for what's to come!  
I assume you don't want to plow through heaps of loops and complex code to do everything we did in the previous steps for *every* frame and *every* collider.  
That's where Crash's real power comes in. Let's `insert()` our colliders and get to some serious collision checking!

```javascript
Crash.insert(point);
Crash.insert(circle);
Crash.insert(box);
Crash.insert(polygon);
```

You could have achieved the same by passing `true` for the `insert` argument of the constructors, like I mentioned at the beginning.

> __Fun Fact:__ just like `Crash.test()`, you could use `collider.insert()`, as a convenience method.

Now that our colliders have been inserted, we can let Crash do all the hard work: it will do all the collision checks for us! And because Crash leverages the power of RBush, only the checks that make sense will actually be performed, causing a huge gain in CPU time.

Before we can let Crash do anything, we would like to make sure it can report to us what it's doing, to let us react to collisions. To do that, we add a listener with `Crash.onCollision()`. This listener will be called every time a collision occurs.

```javascript
var listener = function(a, b, res, cancel) {
    alert("Oh my, there is a collision!");
}
Crash.onCollision(listener);
```

Here, `a` and `b` are the colliders that are colliding, `res` is the all-knowing Response, and cancel is a function that cancels all further collision checks for this collider. This may be useful if you move the collider (and all checks become invalid) and/or when you use `Crash.check()`, which we will cover in the next step.

> __Fun Fact:__ you can easily remove a listener with `Crash.offCollision(listener)`. Just make sure you saved it somewhere when you added it, so you can pass it to `offCollision()` as an argument!

And now, we're ready for some serious stuff:

```javascript
Crash.testAll(circle);
```

This runs collision checks for all inserted colliders that may be colliding with our circle.
As our circle is colliding with all the other colliders, except our box, the listener we added previously will be called once for `point` and once for `polygon`, but not for `box`. For every call, `a` equals `circle`, `b` the respective collider and `res` gives us some info about the collision.

> __Fun Fact:__ Note that, like with `Crash.test()`, we can pass in a Response, but we don't have to. If we don't, Crash will make one for us and pass that around to the listeners. How convenient!



### What kind of sorcery is this !?
We can even go one step further: let's make Crash do __everything__!

For this to work, we need to make sure that we call `Crash.moved()` on all the colliders that have moved:

> __Fun Fact:__ you probably already guessed: you can use `collider.moved()` instead!

```
Crash.moved(point);
Crash.moved(circle);
```

This way, Crash will know it only has to call `testAll` for these colliders, the ones that have moved.

And, now, ladies and gentleman, the holy grail of collision checking:

```javascript
Crash.check();
```

And it's done. All checks have run, our colliders are where they should be. With just one function call. Such wow.


#### But what's up with that `moved()`?
All the built-in methods (like `moveTo`, `setOffset` and `rotate`) already call this for you, so you don't have to worry about this. You should only worry when you insert a collider for the first time. Normally, when designing your game (or anything else), you would make sure your colliders aren't already colliding when you load them. When this is the case, though, you can call `Crash.checkAll()` just after you loaded your colliders. This will do the same as `check()`, but for all colliders, not just the ones that moved. Neat, isn't it?










## API

API docs are under construction.  
In the meantime, you can waste some time looking at the [source code]; it's only 500 lines of nice, fluffy code!



## Contributing

All contributions are very welcome!  
Typos, bug fixes, code cleanup, documentation, tests, a website, you name it!

Questions and feature requests belong in the [issue tracker], __with the right tags__.




## License

The MIT License (MIT)

Copyright (c) 2014-2015 Tuur Dutoit

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.














[RBush]: https://github.com/mourner/rbush
[rbush-docs]: https://github.com/mourner/rbush/blob/master/README.md
[SAT.js]: https://github.com/jriecken/sat-js
[sat-docs]: https://github.com/jriecken/sat-js/blob/master/README.md
[require.js]: http://requirejs.org
[getting-started]: https://github.com/TuurDutoit/crash#getting-started
[API]: https://github.com/TuurDutoit/crash#api
[source code]: https://github.com/TuurDutoit/crash/blob/master/crash.js
[issue tracker]: https://github.com/TuurDutoit/crash/issues