# Domain Driven Design

Domain-driven design (DDD) proposes to attack the software crisis and communication issues from a different angle.

## What Is Domain-Driven Design?
DDD covers the whole software development lifecycle, from gathering req to low-level design. 
<br>
Align Design with busines domain. <br>

DDD proposes to align business + technical concerns => through analysis of the business domain. 

Modeling of business domain. 

Provides tools for design decisions on two levels. 
`strategic` and `tactic` level. 

### Strategic (Sort of high-level)
What should be solved or addressed?
A robust framework for making high-level architectural decsions,
<br>
+ Including tasks such as decomposing system to modular tasks. 
+ Define Interaction between components

### Tactical Strucure (sort of low-level -> interface/impl)

making low-level design decisions relating to the implementation of the system’s components and their business logic.

## Strategic Design
<br>

### Analyzing Business Domains


Core domain <br>
Core Sub domain <br>
Supporting sub domain

<br>

### Discovering Domain Knowledge

```
Knowledge Discovery,
Communication

```
Domain-driven design proposes a better way of getting the knowledge from domain experts to software engineers — using a ubiquitous language.

```

Domain knowledge into analysis model
Analysis model into requirements
Requirements into system design
System design into source code

```

#### Language of the Business

You need to speak the language of the business. No tech Jargons.! 

The ubiquitous language must be precise and consistent. 

The ubiquitous language is a model of the business domain. 

What is a model?. Abstraction of reality represented through the diagrams or language so that everyone can understand. (Essentially you are playing BA type of role to some extent.)

Ubiquitous language is a cornerstone of domain-driven design. It allows us to share knowledge of the business domain effectively, and it fosters communication between different stakeholders. 


### Managing Complexity with Bounded Contexts
(System -> Subsystem (Bounded contexts))

Monolithic Bounded context -> subdomain bounded context

## Context Mapping

Bounded Context model the business domain for the solving of a particular problem. 

Models are (sort of) tied to a particular bounded contexts, evolved and impemented indepedently.  

These bounded contexts will exist with other components. (through contracts). 

Then, the next step is the interactions are addressed through the solution design. 

Relationships and integrations between different components are addressed through solution design. 

### DDD Patterns
#### Cooperation
```
Well established communication. 
1. Partnership . (Ad hoc - One team notifies of a contract change, the other team implements)
2. Shared Kernal patterns (Through compiled library - Change to the contract will break the )

```

#### Customer Supplier
Customer(Downstream) / Supplier(Downstream)

the conformist, anticorruption layer, and open-host service patterns.

<b>the conformist</b> <br>
Supplier sets the contract and customer takes it.

<b>Anticorruption Layer</b> <br>
Supplier sets the contract and customer cannot take all of them because of changes often, contract is not upto the standards and etc. Those type of scenarios, Have layer that maps to the internal to problem domain that  you need to deal with. 

If there are any change in the interface, it would only affect the interface not the implementation. (Common principle of , code it to the interface.)



#### Open Host Service
Power is skewed towards the consumers. Host is willing to protect consumers and provide best service possible.

#### Separate ways

My happen becuase of communication issues. Not willing to co-operate 
<br>
Generic Subdomains.
- Logging service

##### Model differences

### Context Map
The last step is that provide a context map, visual representation of the systems bounded contexts and integration between them. 

1. High-level design
2. Communication design
3. Organization issues.

In summay, the bounded contexts are not independent. They have to interact with each other. 

<b>Partnership</b><br>
Bounded contexts are integrated in an ad hoc manner.

<b>Shared kernel</b><br>
The integration contract is defined as a compiled library that is referenced by both bounded contexts.

<b>Conformist</b><br>
The consumer conforms to the service provider’s model.

<b>Anticorruption layer</b><br>
The consumer translates the service provider’s model into a model that fits the consumer’s needs.

<b>Open-host service</b><br>
The service provider implements a published language—a model optimized for its consumers’ needs.

<b>Separate ways</b><br>
It’s less expensive to duplicate particular functionality than to collaborate and integrate it.


## Tactical Design

Business logic implementation Patterns


### Transaction Script
ETL

### Active Record
Business logic is simple, However, the business logic may operate on more complex data structures. 

Operating on such data structures via plain transaction script would result in lots of repetitive code (?. More real world example)
ORM, CRUD 

### Domain model
instead of CRUD interfaces, we are dealing with complicated business rules and invariants that have to be protected.

Eric Evans -> Domain-Driven Design: Tackling Complexity in the Heart of Software (Addison-Wesley), as a tool for implementing core subdomains.

DDD’s tactical patterns are the building blocks of such an object model: 
```
aggregates, 
value objects, 
domain events, 
aggregation roots
```

How do you handle transaction boundary?. 

1. Come up with Hierarchy of objects. (Relationships) 
Aggregated business entites. (Sort of how do you address impedence mismatch scenarios)

2. Address how do you reference other aggregates.
(Transaction boundaries)
FOr ex: Strong consistency vs eventual consistency

3. Find out the Aggregation root -(Epicenter)

From Aggregation root, pushes the event to other parts of the systems. 

### Event-Sourced Domain Model
Same as domain model pattern. Core business logic is complex and belongs to a core subdomain. 

#### Event Sourcing
Every change in the system's state must be expressed and recorded as a domain event. 
Ex: CRM System

#### Source of Truth 
DB Stores the system domain events is the only strongly consistent storage. 

1. Load the aggregate domain events
2. Reconstitute a state representation.
3. Execute the business logic and produce new domain events
4. Commmit the new domain events to the database.

## Architectural Patterns 
Which architectural pattern to use is a crucial tactical design decision. 

### Ports & Adapters

### Command-Query Responsibility Segregation

#### Polyglot Modeling
OLTP, OLAP and search

##### COMMAND EXECUTION MODEL
CQRS devotes a single model to executing operations that modify the system’s state (system commands)—in other words, OLTP.

Use Cases
The CQRS pattern can be useful for applications that need to work with the same data in multiple models.
CQRS naturally lends itself to event-sourced domain models. The event sourcing model makes it impossible to query records based on the aggregates’ states, but CQRS enables this by projecting the states into queryable databases.


## Event Stroming

To define business domain events, commands, aggregates and even possible bounded contexts. 
Intent is to share the knowlege between different stakeholders, alignment of their mental models. 

How do you explore 
Events
Go over the domain events and orgainize them
Commands
Policy
Aggregate

## Evolutionary Design
thoughtful design will turn into a big ball of mud if it isn’t evolved on par with the changes in its business domain.
Keep up with the change

## Getting Started with Domain-Driven Design
Where do you start?
Ubiquitous language
