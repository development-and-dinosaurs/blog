---
title: Connascence of Name
date: 2023-05-09
description: "Connascence of name is the weakest form of connascence in the static category. Connascence of name refers to when multiple components must agree on the name of a particular entity."
image: images/posts/connascence-of-name-banner.png
imageAltAttribute: A bad AI generated image of two dinosaurs at a conference, one has a tag that identifies him by name. Both need to agree on this name in order to communicate effectively. 
tags:
   - connascence
---

Connascence of name is the weakest form of connascence in the static category. Connascence of name refers to when multiple components must agree on the name of a particular entity.

Consider the following class:

```kotlin
class Dinosaur {
  fun feed() {
    // Feed the dinosaur
  }
}
```

Our `Dinosaur` class has connascence of name- both for the class declaration and the method declaration. If we change the class from `Dinosaur` to `Chicken`, anything that references the class by name will break. If we change the method name from `feed()` to `devour()` or anything else, anything that is using that method will break. All components that use `Dinosaur` are dependent on the names that it uses.

Pretty obvious right? As previously mentioned, connascence of name is the weakest form of connascence, but it still does describe the concept of connascence in a simple, easy to understand manner. If you change something in one place (like the name of a class), you have to change it somewhere else to maintain correctness.

You can't get rid of connascence of name- it's fundamental to the way we refer to different components and entities in programming. Whenever we change the name of a component in it's declaration, we always have to change all code that refers to the old name. This is why we refer to connascence of name as the weakest form of connascence- it's littered all over our codebases, and there's not anything we can do about it. Fortunately as you might have surmised already- it's also not a big deal.

To lessen the impacts of connascence of name in a codebase, you can do things like reduce the number of different components that interact with a single component- this reduces the blast radius of name changes. Additionally, taking care when naming things and choosing a good name will help you not have to change it in the future- but this is easier said than done!
