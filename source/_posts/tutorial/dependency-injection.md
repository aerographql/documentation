---
title: Dependency injection
layout: page
toc: true
---

# Dependency injection

This tutorial follow the {% post_link tutorial/interface-and-union previous one %} and will teach you how to use the Dependency Injection system of AeroGraphQL to enhance your resolvers and make your code DRY.

We'll use the final source code of the {% post_link code/interface-and-union previous tutorial %} as a starting point.

**The full source code for this tutorial can be found {% post_link code/dependency-injection here %}**.

## Why do we need Dependency Injection

Let's have a look at the current implementation of our RootQuery User's resolver:

```typescript
@ObjectImplementation( { name: 'RootQuery' } )
export class RootQuery {

    @Resolver( { type: User } )
    user( @Arg() name: string ): User | Promise<User> {
        return users.find( u => u.name === name );
    }
}
```

Actually, this method rely on a global variable *users* to implement it's business logic.  

* But what happen if this business logic become bigger and more complex ?
* What if some pieces of this logic have to be shared with other resolvers ?

This is where Dependency Injection comes in !

> Dependency Injection allow you to define pieces of reusable code that can be shared between your resolvers.  
> Those pieces of code can depends on each other forming a dependency tree resolved by AeroGraphQL.  

Most of the time, you will use DI (Dependency Injection) to create services.

## Writing the service and injecting it

We start by writing the service itself:

```typescript

import { Injectable } from 'aerographql';

@Injectable() 
class UserService {
    find( name: string ) {
        return users.find( u => u.name === name );
    }
}

```

One thing to note: we use the **@Injectable** decorator on our class.  
This decorator will force Typescript to emit metadata that will be use by AeroGraphQL internally.  


Now let's change our RootQuery to use a new **UserService** that will handle all the database communication logic:

```typescript
@ObjectImplementation( { name: 'RootQuery' } )
export class RootQuery {
    constructor( private userService: UserService ) {}

    @Resolver( { type: User } )
    user( @Arg() name: string ): User | Promise<User> {
        return this.userService.find( name );
    }
}
```

Here, we just tell the RootQuery class, that it depend on a depency called **UserService**. 
This dependency will be automatically injected into the root query class instance when needed.

*This automatic resolution was possible because the UserService was decorated with he **@Injectable** decorator.*

> ---
> * To define a dependency simply add a constructor to the desired class, and add to that constructor a parameter matching the desired dependency.  
> * AeroGraphQL will use the Typescript type annotation to correctly instanciate and inject the dependency.  
> ---

** AeroGraphQL DI system is heavily inspired by the [Angular DI system](https://angular.io/guide/dependency-injection) **

To make this example fully operational, we need to inform AeroGraphQL how and **what** to inject when someone is requesting the **UserService** dependency.

This is done by configuring the **Injector**

## Configure the injector

The injector is an object that act as a container for each dependency in the system.

Before beeing used, dependencies must be registered within this injector.

> It's not necessary to explicitly create an Injector manually to use the DI system.  
> In fact, **each AeroGraphQL Schema store internaly an instance of an injector** ( checkout the [BaseSchema](https://github.com/aerographql/packages/blob/master/packages/schema/src/schema/base-schema.ts) class )  
> Most of the time you won't have to work with the injector directly.

To configure an Injector, you must provide it a list of **Providers**.  
A provider is something that will tell the injector how to *provide* a given dependency to classes asking for it.

A list of providers can be passed to the AeroGraphQL schema in order to configure it's own injector:

```javacript
@Schema( {
    rootQuery: 'RootQuery',
    components: [ RootQuery, User, UserImpl, Todo, PonctualTodo, RecurentTodo ],
    providers: [ UserService ]
} )
export class MySchema extends BaseSchema {
}
```

> ---
> 
> We use the **providers** field of the **@Schema** decorator to pass additional provider to the internal Schema Injector
>
> ---

### The provider object

Here we directly pass the **UserService** class as a provider.  
Like with the Angular DI system, this is a shorthand for a more complete syntax using a provider object :

```javacript
@Schema( {
    rootQuery: 'RootQuery',
    components: [ RootQuery, User, UserImpl, Todo, PonctualTodo, RecurentTodo ],
    providers: [ { token: 'UserService', factory: UserService } ]
} )
export class MySchema extends BaseSchema {
}
```

`{ token: 'UserService', factory: UserService }` is the provider object.

What's interesting here, is that it open the door for swapping out, or replacing implementation of a given Service:  
Imagine that you have two implementations of the UserService:
* One for a mongodb backend
* One for a PostgresSQL backend

Using the DI system you can configure your whole application to use a specific implementation without even changin a single line of code in any class that depend on this dependency:

```javacript
// For a dependency identified by the 'UserService' token, provide the Mongodb implementation
@Schema( {
...
    providers: [ { token: 'UserService', factory: MongoDBUserService } ]
} )
// or, for that same token provide an other implementation:
@Schema( {
...
    providers: [ { token: 'UserService', factory: PostgresUserService } ]
} )
```

## More on providers

### Factory providers

Until now, we have used **Factory providers**.  

When the injector is created, each factory provider will create a new instance it using the factory provided using the **new operator**.  

In our case, the following provider: ` UserService` (which is equal to `{ token: 'UserService', factory: UserService }` ) will result in an instance of UserService created like this: `instance = new UserService( depsA, depsB )` where depsA and depsB are other *imaginary* dependencies created beforehand. 

> There is only one single instance of a given dependency in an injector. This dependency is created at startup time

### Value providers

AeroGraphQL DI system also offer an other type of provider: **Value providers**.

This allow you to pass any kind of Javascript object as a dependency.

This kind of provider won't be *instanciated* with the new operator like a factory provider.

Here is how to pass a single value as a provider:

```typescript
import { Inject } from 'aerographql';

// First configure the schema internal injector

@Schema( {
    rootQuery: 'RootQuery',
    components: [ RootQuery, User, UserImpl, Todo, PonctualTodo, RecurentTodo ],
    providers: [ { token: 'DBConfig', value: { config: { mongoDBIP: '127.0.0.1', mongoDBPort: 27017 } } } ]
} )
export class MySchema extends BaseSchema {}

// Then inject this dependency somewhere

@Injectable()
class MongoDBUserService {
    constructor( @Inject('DBConfig') private config: any ) {}
}
```

Note the use of the **@Inject** decorator to tell the DI system which dependency to inject.

In the previous case, we use a simpler syntax:
`constructor( private userService: UserService )` because UserService is a **class** AeroGraphQL was able to infer which dependency to inject by using the generated Typescript metadata.

But when the dependency is not a class, we must explicitly tell the DI system which token to lookup for this dependency.

In fact the previous syntax is a shorthand for the full one: 
` constructor( @Inject('UserService') private userService: UserService )`


## What's next

Next {% post_link tutorial/middleware tutorial %} will walk you through using middleware to implement authentication..
