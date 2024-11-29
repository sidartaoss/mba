# DDD (Domain Driven Design)

## Vaughn Vernon

## What is DDD

Domain-Driven Design is a way of thinking about software model design. 

It's a knowledge-crushing approach where technical experts and domain experts are teaming up to solve complex business problems with software.

And Domain-Driven Design provides contextual, language-centric, strategical and tactical modeling tools, and the primary tool, the sort of foundational tool of Domain-Driven Design is the Bounded Context.

And inside the Bounded Context is the Ubiquitous Language, where a team are expressing their ideas, about the way they think to solve a specific problem, they're conversing, they are having meaningfull dialogs with one another, and, as a result, they are implementing a design and are implementing a software model to solving complex business problems.

Bounded Context
- Ubiquitous Language

## Why use DDD

``
Conway's Law: "Organizations which design systems (in the broad sense used here) are constrained to produce designs which are copies of the communication structures of these organizations." (1967)
``

This is for solving complex business domain problems.

## Strategic Design

Bounded Context: a place where the model and all of the model elements have relevance. 

They have a very carefully defined definition, and the definition is very likely different in this context for any of the terms used. 

Let's say that we have the term policy. What is a policy in this context? Well, it has a very specific meaning. In a different bounded context, the word policy could be similar to the policy here, but have a quite different meaning that policy does in the original context. So, this is one of the main uses of bounded context.

And strategic design includes team language contexts. So, a bounded context uses context for language, a ubiquitous language, but it is really focused on just one aspect of the business problem.

So, language context is very important with domain-driven design.

1. Enterprise Scope
2. System Scope
3. Problem Space Scope
4. Context

If you look inside an enterprise software environment, note that there are all kinds of systems, there are generally many legacy systems that tend to be the big ball of mud, numbered as 2, System Scope.

We also have some cleaner System Scope, because they are relatively new, perhaps they've used domain-driven design to work on those System Scopes, and they integrate with some of the legacy scopes, some of the big ball of mud scopes, but they themselves are designed for a clean and sort of loose coupling, almost disconnect or very light touch connection to system scopes that are outside of their own system.

Finally you see the third and fourth areas of the enterprise, so whe have a software initiative that is known as the Problem Space Domain, or the Problem Space Scope. This means that we are focused on developing software that solves a rather course-grained set of problems, and that evolves to more than one bounded context.

## Domain and subdomains

You can see that there are 5 different teams, 5 different areas of expertise, and 5 different areas where domain experts and technical experts are working together as a single team. 

This is known as the problem space domain, and the individual bounded contexts that are inside of this problem space domain, along with any other parts that they are integrated with, which we're not seeing here, but perhaps legacy or other clean newer system scopes, these are known as subdomains.

So, at the problem space level, you're actually thinking more about the problem space domain and subdomains, whereas when we take on the solution space, then these subdomains, each of those are now sort of transitioning into a solution space, where they become a bounded context.

Problem Space					Solution Space
	- Subdomains					- Bounded Contexts

## Aggregates, Values, Events

An aggregate is some kind of entity that, as you can see here, provides a transactional scope, aka a database transaction of this entire set of objects.

## Model Behavior

So, having business rules inside a bounded context model is very important as part of the model behavior.

## Domain Services

Are known as stateless operations. We have a domain service here, QuoteGenerator, and it is generating a quote, and the result , the outcome of that stateless operation, or domain service, is a policy quote, a noun, an entity, which, itself, again, is an aggregate inside the database transactional boundary.

## Business Logic

Business logic, represented by different kinds of policies is important, so, not only do we have some business rules, but we have the exact calculations based on the kinds of things that we want to have happened accross the model.

## Command to Event

Commands lead to events

So, not only can events happen from commands, but they can happen from non-command sources, such as time.

## Architecture

Ports And Adapters.

## Event Sourcing

Event Sourcing is used to persist the state of an aggregate.

Why do we use event sourcing? 

It's generally used for any time that you need to have a discret fact collected by everything that's happened within a domain model. 

It can be compliance, compliance to some government rules, or some industry rules, such as medical, such as financial, you want to basically a snapshot, a fact for everything that has happened, and therefore, you can have an audit, a history persisted for everything that has happened. 

But, as a second use, it  also represents the full state, this entire stream represents the full state of the aggregate.

## CQRS

We also have a pattern known as CQRS. 

This is where a user submits commands, imperatives do this, and once those commands are carried out, the results of the command being carried out, for example, through domain events, will be projected into what is known as the query model.

## Monolith + Microservices

- Saga
    - Choreography
    - Orchestration