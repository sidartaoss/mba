# SOLID

## Robert C. Martin

## Single Responsibility Principle - SRP

``
A class should have one and only one reason to change.
One actor that is the source of that change.
``

The Single Responsibility Principle may be the worst name of the principles, because the name implies something that that isn't what the principle is trying to say. 

So many people will read the title of the principle without reading the definition of the principle, and they will take the wrong definition away.

The Single Responsibility Principle says that a class or a module should have one and only one reason to change. That is not saying that the module should have a single responsibility. No; one and only one reason to change.

## Open Closed Principle

``
A software artifact should be open for extension but closed for modification.
``

How do you write systems such that the behavior can be modified without modifying the code?

You do that by dependency inversion.

You do that by separating the high level policy from the low level details, and you make sure that the source code dependencies and those lowest level modules are inverted to point at abstractions, and the high level policy has source code dependencies that point towards abstractions. 

This is how you do the open-closed principle; you separate high level policy from low level detail and you route the source code dependencies to point in the direction of high level policy and abstractions.

Add new code, just don't change any old code.

Is it possible?

Almost.

In our current software systems, there is almost no way to protect every module from change. Some modules are going to have to be open to modifications. But you can limit them. You can minimize the number of modules that are open for modification. 

Generally speaking, you can focus those modifiable modules down towards main. Main is the first program to execute; main probably calls some configuration functions, and within those functions, you may have to make some modifications. 

But everything else in the system can be designed, if you are carefull, so that you can add new features, but only adding new code, without changing any existing code.

How do you do that?
The solution in the 90's: Think really hard...

And then we realized two things, after many years trying this: the designs that we created were almost impossible to maintain, because they were so large, and so complicated.

And secondly, and more importantly, it turns out that customers are really good in outsmarting you.

So we want to get the customer start making changes as early as possible to that we can begin to integrate the designs that protect us from those changes long before we have written too much code. 

We want to get that stuff into the design as early as possible, and that is the agile mindset for the way we integrate the open closed principle, the agile design process: we put things out in front of the customer, frequently and early, and when they make changes, that is the moment that we protect ourselves from those changes.

## Liskov Substitution Principle (LSP)

How do you know if you're violating this principle? Any time you create a subclass that does less than the base class or does something that the users of the base class don't expect, than you have violated this principle.

And you know that you have violated this principle when sometime later you are forced to put if statements in that check the type of object that has been passed to you.

## Interface Segregation Principle

``
Don't depend on things you don't need.
``

In general, when you depend on something, you want to make sure that that thing you are depending on does not contain things that you don't need.

When you depend on things you don't need, when those things that you don't need change, you are going to be subject to changes yourself.

Depend on abstractions that protect you against change.

## Dependency Inversion Principle

``
Depend on abstraction
``

All the other principles use this principle as their kind of low level implementation.
Almost all the design patterns in the Design Patterns book (GoF) use this principle as their low level implementation.

``
Depend in the direction of abstraction.
Depend in the direction of high level policy.
``

``
High level policy should not depend upon detail.
Detail should depend upon high level policy.
``

And how do you do that?

You insert an interface between the high level policy and the low level detail, and that allows you to turn the source code dependency around from the low level detail towards the interface.
And this is power, more power than software developers ever had.

Should we create an interface for everyhting or only when we see that the code can change in the future?

I don't create interfaces unless I need them.
Now, when do I need them?

Well, I need them when if I have a change requested by the customer that I need to isolate myself from; then I'm going to create an interface.

### Referência
MBA ARQUITETURA FULL CYCLE. SOLID e Design Patterns. 2024. Disponível em: https://plataforma.fullcycle.com.br/. Acesso em: 29 nov. 2024.