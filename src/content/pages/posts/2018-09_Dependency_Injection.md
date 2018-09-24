---
created_at: 2018-09-05
kind: article
publish: true
title: "Dependency Injection"
image: "nodejs-elasticsearch.jpg"
tags:
- dependency
- injection
- IOC
- container
sitemap:
  priority: 0.8
abstract: >
  In this post I will be showing what a dependency is, what dependency injection is and how you can use it in projects
---
![Dependency](http://res.cloudinary.com/xeviant/image/upload/v1503528012/sample.jpg)
## The Dependency Analogy
Often times in our programs we've come across places where a part of our code depends on another in other to function appropriately, in this 
case, what is been depended on is a `Dependency`. An analogy can be seen when you're writing a program that stores user information in the database, for
the program to work, it needs to connect to the database somehow before it can store the information in the database. In other word we would say that the database connection
is a dependency that your program is depending on.

Dependencies can also be seen as things required by some part of our code to function, seeing from the example above we can say that the part of the program that
stores user information in the database cannot function properly until the database connection (the dependency) is provided for (injected into) that part of the code.

## What to make out of this
With this thinking we can see that dependencies exists everywhere within our programs and it's necessary we understand how to manage them properly within our apps.
Taking a real lif approach, for a fuel powered car to operate at all, it requires fuel, the fuel in this case is a `dependency` that is injected into the tank, once there is no
more fuel in the tank, the car will stop working.

Bringing it back to Object oriented programming