---
title: Connascence
date: 2023-05-08
description: "In this series of blog posts, we're going to look at how we can use connascence to help drive improvements to our process and write better code. To do this, we're going to have to understand what connascence is, what it means, and how to identify it within our codebases."
tags:
   - connascence
---

In this series of blog posts, we're going to look at how we can use connascence to help drive improvements to our process and write better code. To do this, we're going to have to understand what connascence is, what it means, and how to identify it within our codebases.

## What is connascence?

> A relationship between two or more elements of software in which changing one necessitates changing the others in order to maintain overall correctness; a metric for such a relationship.

Connascence is a term used to describe the relationship and taxonomy of dependencies between two software components. Connascence as a term was first used by Meilir Page-Jones in the 90's in the book [What Every Programmer Should Know about Object-Oriented Design](https://amzn.to/3s6Keb1), which was later updated to use UML instead of Page-Jones's own modelling language in [Fundamentals of Object-Oriented Design in UML (Addison-Wesley Object Technology Series)](https://amzn.to/3I8QdSd)

Put more simply, connascence describes the level of coupling within a codebase, which can be used to reason about the code quality. The more coupled a codebase is, the more difficult it becomes to change.

There are two broad categories of connascence- static and dynamic. Within these categories there are 9 different types or levels. Finally, connascence also has three properties which can help us decide how bad the particular level of connascence is.

The concept of connascence has not seen widespread popularity in the development community. However, there are a few pieces of information out there such as [this talk by Jim Weirich](https://vimeo.com/10837903) and [a talk by Kevin Rutherford](https://www.youtube.com/watch?v=fSr8LDcb0Y0). You could probably find more if you were to look too, but you're here now so maybe that's all you'll need. 

## What can we do with connascence?
We can use connascence to guide our software development practices, to write more maintainable code. By keeping connascence in mind when writing code, we can reduce the coupling within. By learning to identify connascence when we see it, we can perform refactoring to reduce connascence and improve quality.

That's where this series of posts come in. We're going to go over the different levels of connascence and how to spot them in the wild (or at least in some contrived examples), as well as touch on the different properties of connascence to decide when we need to act. We'll then work through how to remove or replace the connascence with something else to avoid trouble with tightly coupled code in the future.

We're going to cover the following areas:

### Static Connascence

- Connascence of Name
- Connascence of Type
- Connascence of Meaning
- Connascence of Position
- Connascence of Algorithm

### Dynamic Connascence

- Connascence of Execution
- Connascence of Timing
- Connascence of Value
- Connascence of Identity

### Properties of Connascence

- Strength
- Locality
- Degree

## Anything else? 
I think connascence is a good way to talk about the way two pieces of code might be coupled, and the problems this might cause. This doesn't mean that this is the only way though, and nothing here should be taken as the gospel truth or a silver bullet to end all your woes. Dive into this information, try it out in your own codebases, and come up with your own opinion of whether this resonates with you or not. 
