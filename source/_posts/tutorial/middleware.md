---
title: Middleware
layout: page
toc: true
---

# Middleware

This tutorial follow the {% post_link tutorial/dependency-injection previous one %} and will teach you how to use the Middleware system of AeroGraphQL to create a reusable authentication layer.

We'll use the final source code of the {% post_link code/dependency-injection previous tutorial %} as a starting point.

**The full source code for this tutorial can be found {% post_link code/middleware here %}**

## What's an AeroGraphQL middleware

AeroGraphQL middlewares are similar to [ExpressJs middlewares](https://www.safaribooksonline.com/blog/2014/03/10/express-js-middleware-demystified/) in the sense that they are simple functions applied sequentially in order to process an given request.

Here are some key concepts about AeroGraphQL middlewares:

* As opposed to ExpressJs, AeroGraphQL Middleware are class that implement an **execute** method.
* An AeroGraphQL middleware applies to a GraphQL query.
* An AeroGraphQL middleware can be used either as:
    * Guard, accepting or rejecting input queries dependig on certain conditions.
    * Resolver, feeding the GraphQL context object with new values needed for subsequent operations.
    * A mix of both.
* An AeroGraphQL middleware chain is always associated with an **@Resolver** method:
    * The **@Resolver** method is always the last element in the chain to be executed.
    * The **@Resolver** method will only be executed if all of their associated middlewares succeed.
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

```typescript
import { Middleware, MiddlewareInterface } from 'aerographql';

@Middleware()
class AuthMiddleware implements MiddlewareInterface<boolean> {
    execute( src: any, args: any, context: any, options: any ) {
        console.log( 'From the middleware' );
        return true;
    }
}
```

The key points here are:
* We use the **@Middleware** decorator to tag this class as a Middleware
* We implement the MiddlewareInterface interface.

> The MiddlewareInterface interface take a Generic parameter that represent the type returned by the middleware.

Let's have a look at the signature of the **execute** function:  
It's take four parameters.  
The first three ones should looks familiar:
**src, args, context** are the standard parameters passed to any [GraphQL resolver functions](http://graphql.org/learn/execution/#root-fields-resolvers)  
The **options** parameter is specific to AeroGraphQL middleware, and is used to customize the behavior of each middleware. More on that soon...

## Use it

Now that we have our middleware we need to use it.  
In our case we like to call this middleware on a new endpoint: **viewer** which will return the currently logged in user if any (the viewer term come from the [relay terminology](https://www.learnrelay.org/queries/what-is-a-query/))

Change our RootQuery implementation:

```typescript
@ObjectImplementation( { name: 'RootQuery' } )
export class RootQuery {
    constructor( private userService: UserService ) { }

    @Resolver( { type: User } )
    user( @Arg() name: string ): User | Promise<User> {
        return this.userService.find( name );
    }

    @Resolver( { type: User, nullable: true, middlewares: [ { provider: AuthMiddleware } ] } )
    viewer( ) {
        return null;
    }
}
```

Restart your server: `yarn start` and execute the following query: 
```
{
  viewer {
    name
  }
}
```
Checkout the server output and you sould see that the middleware was correctly called.

> There is two places where a middleware chain can be defined:
> * At the class level, in the **@ObjectImplementation** configuration: 
In this case the middleware chain will be called for every **@Resolver** of this class
> * At the resolver level, in the **@Resolver** configuration:  
In this case, the middleware chain will only be called for this specific resolver, overriding the one defined at the class level, if  any.

Now we have a new endpoint that return the current user.

This user will be extracted in the **AuthMiddleware** by using the payload of a JWT token where we store the user name.

## Implement JWT authentication

We are going to quickly implement a [JWT](https://jwt.io/) authentication mecanism using the new middleware.

First add the [jsonwetoken](https://github.com/auth0/node-jsonwebtoken) library for nodejs:
```
yarn add jsonwebtoken @types/jsonwebtoken
```

Then implement a fake ExpressJS middleware that will inject a new header in the request containing a valid JWT token:

*This token should normally be provided by the original request* 

```typescript
let fakeJWT = ( req: any, rep: any, next: any ) => {
    req.headers[ 'Authorization' ] = jsonwebtoken.sign( { name: "Bob" }, 'secret' );
    next();
}
```

Then add this middleware to ExpressJs, before the Apollo server middleware:

```typescript
this.app.use( '/graphql', bodyParser.json(), fakeJWT, graphqlExpress( { schema: mySchema.graphQLSchema } ) );
```

Pass the ExpressJs request object down to the GraphQL context: 

```typescript
this.app.use( '/graphql', bodyParser.json(), fakeJWT, graphqlExpress( ( req ) => {
    return { schema: mySchema.graphQLSchema, context: { req } };
} ) );
```

Now everything is in place to implement the middleware itself...

```typescript
interface Context {
    req: express.Request;
    user: User; // We will come back to this field later
}

@Middleware()
class AuthMiddleware implements MiddlewareInterface<any> {
    constructor( private userService: UserService ) { }
    execute( src: any, args: any, context: Context, options: any ) {
        let token  = context.req.headers[ 'Authorization' ] as string;
        try {
            jsonwebtoken.verify( token, 'secret' );
        }
        catch ( e ) {
            throw 'Invalid token' + e;
        }

        let payload = jsonwebtoken.decode( token ) as any;
        let u = this.userService.find( payload.name );
        return u;
    }
}
```

Quick review of this code:
* We define an interface to add type informations to our context.
    * The user field will be used just after.
* We use Dependecy injection to get a reference to the UserService (checkout the constructor signature).
* We extract the Authorization header from the ExpressJs request stored in the context.
* We verify the token and throw an error if the token is invalid.
* If the token is valid, we decode it, and use the JWT payload and the UserService to get a reference to the authenticated User.
* We return this user.

## Store result of a Middleware

This sound great but what happen with our authenticated User ?

Well, actually, it just get lost.

The **AuthMiddleware** succeed because a truthy value was returned, but this value was not stored anywhere.  

To remediate, change the RootQuery **@ObjectImplementation** configuration where this middleware is used:
```typescript
@ObjectImplementation( { name: 'RootQuery' } )
export class RootQuery {
    constructor( private userService: UserService ) { }

    @Resolver( { type: User } )
    user( @Arg() name: string ): User | Promise<User> {
        return this.userService.find( name );
    }

    @Resolver( { type: User, nullable: true, middlewares: [ { provider: AuthMiddleware, resultName: 'user' } ] } )
    viewer( previous: any, context:Context ) {
        return context.user;
    }
}
```

> You can store the result value of a middleware in the current context by using the **resultName** field of the middleware configuration object in the **@ObjectImplementation** decorator

Now, restart the server: `yarn start` and execute this query: 
```
{
  viewer {
    name
  }
}
```

The viewer query resolver should execute the **AuthMiddleware** beforehand which pass the authenticated user down to the resolver which return it to the client !

**Congratulation, you now have a fully operational middleware.**

## Conclusion

This episode conclude our tutorial on AeroGraphQL basic features !

Checkout the {% post_link api API references %} for more informations.

