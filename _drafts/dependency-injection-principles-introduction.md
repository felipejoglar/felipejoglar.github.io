---
title: 'Dependency Injection Principles: An introduction'
description: Dependency Injection has always been a hot topic in the android community. In this post, we will dive into what dependency injection is and what it is for. How making use of it we can improve the design of our apps.
author: felipe
series: dependency-injection
image:
  url: /assets/images/dependency-injection-principles-introduction/introduction
  caption: 'Photo by <a href="https://unsplash.com/@diana_pole?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Diana Polekhina</a> on <a href="https://unsplash.com/photos/clear-glass-tube-with-brown-liquid-dw6tvK_PuxM?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>'
---

*Dependency Injection* (DI) is not a goal in itself. It is a tool, a technique. And, as the most of programming techniques its purpose is to deliver working software as efficient as possible. One of its aspects is maintainability.

An excellent way to make code more maintainable is through *loose coupling*. Loose coupling makes the code extensible, and extensibility makes it maintainable. DI is nothing more than a technique that enables loose coupling.

> Dependency Injection is a set of software design principles and patterns that enables us to develop loosely coupled code [^1].

## Benefits of DI

DI might seem overkill in many cases. Instead of just creating and using a component, we create an abstraction, an implementation, and then we wire it all together to create something we could have just done in one line. But when complexity increases, the overhead diminishes. 

In software development requirements change and are often fuzzy. The features we implement also tend to be complex. DI help us address this issues by enabling loose coupling.

### Extensibility

Code can be extended and reused in ways not explicitly planned for. Loose coupling enables us to write code that is [open for extensibility, but closed for modification](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle). The only place where we need to modify the code is the application's entry point.

For example, imagine we have a component `MessageProcessor` to operate on several types of `Message`. It would be nice to have a way to add a new type of `Message` without the need to modify the current `MessageProcessor`. This can be done with the use of DI, along with other design principles.

```kotlin
// Message.kt
data class Message(
    val type: Type,
    val value: String,
) {
    enum class Type { TEXT, IMAGE }
}

// MessageProcessor.kt
interface MessageProcessor {
    fun canProcess(message: Message): Boolean
    fun process(message: Message)
}
```

With the two components above we can have different types of processors. Each one would do an operation depending on the type of `Message`.

```kotlin
// TextMessageProcessor.kt
class TextMessageProcessor : MessageProcessor {
    override fun canProcess(message: Message): Boolean {
        return message.type == Message.Type.TEXT
    }

    override fun process(message: Message) {
        // Process text message
    }
}

// ImageMessageProcessor.kt
class ImageMessageProcessor : MessageProcessor {
    override fun canProcess(message: Message): Boolean {
        return message.type == Message.Type.IMAGE
    }

    override fun process(message: Message) {
        // Process image message
    }
}
```

And then, a single component will be in charge of handling the incoming messages.

```kotlin
// AppMessageHandler.kt
class AppMessageHandler(
    private val processors: List<MessageProcessor>,
) {
    fun process(message: Message) {
        processors.forEach { processor ->
            if (processor.canProcess(message)) {
                processor.process(message)
            }
        }
    }
}
```

So now, to extend the behaviour with a new type of `Message` we only need to create a new `MessageProcessor` and include it in the collection of processors that the `AppMessageHandler` receives as constructor parameter.

### Parallel development

Separation of concerns makes it possible to develop code in parallel. When software projects grow, it become neccesary to have multiple developers working in parallel in the same code base. Each member, or team, is often asigned for an area of the overall application. To separate responsibilities, each member develops one or more components that will need to be integrated into the finished application.

### Maintainability

As responsibility of each component becomes constrained and clearly defined, maintainance of the application becomes easier. This is a consequence of the *Single Responsibility Principle*, which states that each component should have only a single responsibility. Adding new features become easier, because it's clearer where changes shouhld be applied. More often than not, we don't need to change existing code, rather we will add new components and recompose the application. The *Open/Closed Principle* in action again.

Here we can imagine the message processing example from above. If we didn't apply DI and those design principles, we could have created an `AppMessageHandler` like this:

```kotlin
// AppMessageHandler.kt
class AppMessageHandler{
    fun process(message: Message) {
        when(message.type) {
            Message.Type.TEXT -> {
                // Process text message
            }
            Message.Type.IMAGE -> {
                // Process image message
            }
        }
    }
}
```
It may seem not a big deal as we have only two types of messages in this example. But as the complexity of the application grows, so does this `AppMessageHandler`. This will become a burden in maintainability regards, and this class is opened to be modified for every small change in any of the processing mechanisms for the messages. Making it hard to maintain. But if we keep the responsibilities isolated having different processors, each one in charge of their own stuff, it becomes easier to modify, fix and extend.

### Testability

Description

[^1]: Steven van Deursen and Mark Seeman, [Dependency Injection Principles, Practices and Patterns](https://www.manning.com/books/dependency-injection-principles-practices-patterns) (Manning, 2019), 4.