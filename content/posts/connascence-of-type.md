---
title: Connascence of Type
date: 2023-05-11
description: "Connascence of type is the second weakest form of static connascence. Connascence of type refers to when multiple components must agree on the type of a particular entity."
tags:
   - connascence
---

Connascence of type is the second weakest form of static connascence. Connascence of type refers to when multiple components must agree on the type of a particular entity.

Consider the following class we saw previously in [connascence of name](/posts/connascence-of-name):

```kotlin
class Dinosaur {
  fun feed(food: Food) {
    // Feed the dinosaur
  }
}
```

I've added the `food` parameter here to show the connascence a little more clearly. The `feed` function invites connascence of type, in that it will only work if you pass something in with the type of `Food`. If you try to pass anything else in, you won't be able to and the code won't compile. 

Now based on this you might think that this type of connascence is only applicable to statically typed languages, but that isn't actually true. Statically typed languages are just the easiest languages to catch them in. If we were using a dynamically typed language like Javascript or Python, you would still have connascence of type between your components, it'll just be implicit rather than explicit. Consider this same function in Javascript:

```javascript
function feed(food) {
  dinosaur.satiation += food.calories
}
```

I've filled in the comment to make it a bit more obvious where the connascence is here. This function still has connascence of type even without an explicit type system- this code won't work for any argument passed in that doesn't conform to the expected "type", (which we previously called `Food`), in that it needs to have a property called `calories` that we can use to fill up our dinosaur's satiation. 

After having it laid out like this, connascence of type is probably as obvious as connascence of name- if you change the type contract in one place, you need to change it everywhere that relies on that type contract (as in, things that are coupled to, or have _connascence of type_ with) in order to preserve the correct functionality. 

Just like connascence of name, you can't get rid of connascence of type- it's just as fundamental to the way we work with different components and entities in programming as the name is. Whenever we change the type of something that a component relies on, we always have to change all code that refers to the contract of the old type.

And once again likewise with connascence of name, you can lessen the impacts of connascence of type in a codebase. To accomplish this, you do things like reduce the number of different components that interact with a single component- this reduces the blast radius of changes and means you'll need to make less changes across the codebase if you keep this information relatively self-contained. 

A fairly contrived example of this (but that actually happens in the real world, honest) would be having connascence of type across your codebase by sharing a representation between your presentation layer and your data layer- say by returning the same entity from your API that you store in the database. 

```kotlin
data class Dinosaur(val id: String, val name: String)

class DinosaurApi() {
  fun handleGetById(id: String): Dinosaur {
      return repository.selectFromDinosaursWhereIdEquals(id)
  }
}

interface DinosaurRepository() {
  fun selectFromDinosaursWhereIdEquals(id: String): Dinosaur
}
```

This might look perfectly reasonable whilst the representations are the same, but this introduces connascence of type across your application. This is slightly different to what we were saying before. Where before we said that whenever we change the type of something in one place, we need to change it elsewhere. That's true, but in this case we're altering the contents and behaviour of a type that is already shared, which means it'll change in both places at once. In this case, the data type changing in the database impacts the data type we return from the application- which might not always be desirable. 

```kotlin
data class Dinosaur(val id: String, val name: String, val peopleEaten: Int)
```

There's certain information that we'd like to store that we probably don't want to expose. By coupling our API to our database via the `Dinosaur` type, we've introduced some connascence that we'd like to get rif of. Introducing a service or translation layer that can take one representation of an entity and convert it into another is one way of reducing (though not removing) connascence of type in this way- you constrain the connascence to a "lower level", more tight-knit group of components which limits the blast radius of a type change to just those components. 

```kotlin
data class DinosaurModel(val id: String, val name: String)
data class DinosaurEntity(val id: String, val name: String, val peopleEaten: Int)

class DinosaurApi() {
  fun handleGetById(id: String): Dinosaur {
      val entity = repository.selectFromDinosaursWhereIdEquals(id)
      return DinosaurModel(entity.id, entity.name)
  }
}

interface DinosaurRepository() {
  fun selectFromDinosaursWhereIdEquals(id: String): DinosaurEntity
}
```

The biggest takeaway for connascence of type is that it's unavoidable, but by limiting the blast radius of use you're less likely to introduce a problem. 
