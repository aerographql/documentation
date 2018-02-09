---
title: ObjectDefinition and ObjectImplementation
layout: page
toc: true
---

# Object Definition and Implementation

There is a one to one relationship between GraphQL Interface, Scalar, Input, Union and their corresponding Typescript class:

AeroGraphQL define:
* **@Interface()** to create a GraphQL interface
* **@Scalar()** to create a GraphQL scalar
* **@Input()** to create a GraphQL input
* **@Uniont()** to create a GraphQL union

But to create a GraphQL object AeroGraphQL provide two decorators:
* **@ObjectDefinition** 
* **@ObjectImplementation**

We recap here what are their differences and how they work together to create a full GraphQL object:

## GraphQL object's structure:

The Final GraphQL Object's structure as exposed by the schema is:  
**the union of each fields and each resolvers defined in every @ObjectDefinition and @ObjectImplementation class sharing the same GraphQL name**

> Both decorators must be provided with a configuration object. This object contain a **name** field which identify GraphQL object the class refers to:
```typescript
@ObjectDefinition( { name: 'User' } )
export class User { ... } 

@ObjectImplementation( { name: 'User' } )
export class UserImpl { ... } 
```

## Fields and Resolvers

In GraphQL terms:
* a **@Field** is a [field without argument](http://graphql.org/learn/queries/#fields)
* a **@Resolver** is a [field with arguments](http://graphql.org/learn/queries/#arguments) and an associated resolver function.

In AeroGraphQL **@Field** can only be defined in **@ObjectDefinition** while **@Resolver** can only be defined in **ObjectImplementation**  

## Intended usages

While classes annotated with **@ObjectDefinition** are never instanciated by AeroGraphQL, classes annotated with **@ObjectImplementation** are automatically instanciated by AeroGraphQL.

The reason is that ObjectImplementation are meant to be used within the dependency injection system in order to interact with Services defining the Busines logic, while ObjectDefinition are meant to be used as **model** for your database schema.
