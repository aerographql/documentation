---
title: Middleware
layout: page
toc: true
---

# Middleware

This tutorial follow the {% post_link tutorial/dependency-injection previous one %} and will teach you how to use the Middleware system of AeroGraphQL to create an authentification layer.

We'll use the final source code of the {% post_link code/dependency-injection previous tutorial %} as a starting point.

**The full source code for this tutorial can be found {% post_link code/dependency-injection here %}**

## What's an AeroGraphQL middleware

AeroGraphQL middleware are similar to [ExpressJs middleware](https://www.safaribooksonline.com/blog/2014/03/10/express-js-middleware-demystified/) in the sense that they are simple function applied sequentially in order to process an inpur request.

Here are some key concepts about AeroGraphQL middlewares:

* As opposed to ExpressJs AeroGraphQL Middleware are class that implement an **execute** method
* An AeroGraphQL middleware applies to the GraphQL Queries.
* An AeroGraphQL middleware can be used either as:
    * Guard, accepting or rejecting input queries dependig on certain conditions.
    * Resolver, feeding the GraphQL context object with new values needed for subsequent operations.
    * A mix of both.
* An AeroGraphQL middleware chain is always associated with an **@Resolver** method:
    * The **@Resolver** method is always the last element in the chain to be executed.
    * The **@Resolver** method will only be executed if all of their associated middleware succeed.
    * As soon as a middleware fail, the rest of the chain won't be executed, including the final **@Resolver**
* An AeroGraphQL middleware is considered as successful if it return either:
    * A truthy value
    * An accepted promise
* An AeroGraphQL middleware is considered as unsuccessful if it return either:
    * A falsy value
    * An resjected promise
* An AeroGraphQL middleware is a part of the Dependecy Injection system and can have dependencies defined in it's constructor

## Create our first Middleware

First, write the Middleware class itself:

```javascript
import { Middleware, BaseMiddleware } from 'aerographql-schema';

@Middleware()
class AuthMiddleware implements BaseMiddleware<boolean> {
    execute( src: any, args: any, context: any, options: any ) {
        console.log( 'From the middleware' );
        return true;
    }
}
```

The key points here are:
* We use the **@Middleware** decorator to tag this class as a Middleware
* We implement the BaseMiddleware interface.

> The BaseMiddleware interface take a Generic parameter that represent the type returned by the middleware.

Let's have a look at the signature of the **execute** function:  
It's take four parameters.  
The first three ones should looks familiar:
**src, args, context** are the standard parameters passed to any [GraphQL resolver functions](http://graphql.org/learn/execution/#root-fields-resolvers)  
The **options** parameter is specific to AeroGraphQL middleware, and is used to customize the behavior of each middleware. More on that soon...

## Use it

Now that we have our middleware we need to use it.  
In our case we like to call this middleware on every query on the RootQuery object:

Add to the RootQuery class decorator configuration the **middlewares** field:


```javascript
@ObjectImplementation( { name: 'RootQuery', middlewares: [ { provider: AuthMiddleware } ]  } )
export class RootQuery {
    constructor( private userService: UserService ) { }

    @Resolver( { type: User } )
    user( @Arg() name: string ): User | Promise<User> {
        return this.userService.find( name );
    }
}
```

Restart your server: `yarn start` and execute the following query: 
```
{
  user( name: "Bob" ) {
    id
  }
}
```
Checkout the server output and you sould see that the middleware was correctly called.

> There is two places where a middleware chain can be defined:
> * At the class level, in the **@ObjectImplementation** configuration: 
In this case the middleware chain will be called for every **@Resolver** of this class
> * At the resolver level, in the **@Provider** configuration:  
In this case, the middleware chain will only be called for this specific resolver, overriding the one defined at the class level, if  any.
