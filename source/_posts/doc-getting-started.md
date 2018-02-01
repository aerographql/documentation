---
title: Getting started
layout: page
toc: true
---

# Getting started with AeroGraphQL

This guide will provide you a step by step tutorial to setup a full GraphQL server that will provide an API to manipulate a multiple users todo list.

*In this tutorial, i'm using yarn, but this will of course work the same with npm.*

> This tutorial assume you are already familiar with GraphQL schema definition and Typescript.  
If you are not, first read [this documentation](http://graphql.org/learn/schema/) for the GraphQL part and [this introduction](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html) for Typescript

## Setup

First setup a new nodejs project with **Typescript**.  
We also install [**ts-node**](https://github.com/TypeStrong/ts-node) as it will make our life easier later on.

```
mkdir todo
cd todo
yarn init
yarn add typescript
yarn add ts-node
yarn add express
yarn add @types/express
yarn add body-parser
yarn add @types/body-parser
```
Then create the following Typescript config file **tsconfig.json** at the root of your  project:
```javascript
{
    "compilerOptions": {
        "target": "es6",
        "module": "commonjs",
        "moduleResolution": "node",
        "sourceMap": true,
        "emitDecoratorMetadata": true,
        "experimentalDecorators": true,
        "noImplicitAny": true,
        "lib": [
            "es6",
            "dom",
            "esnext.asynciterable"
        ],
        "outDir": "./dist"
    },
    "include": [
        "./src/**/*.ts"
    ]
}
```
The only noticable things here is the **emitDecoratorMetadata** and **experimentalDecorators** options which are needed as AeroGraphQL rely on decorators to define the GraphQL schema.

Create a **src** directory and a **main.ts** file in this directory:
```
mkdir src
cd src
touch main.ts
```

In your **package.json** add a script to run the server:
```
"scripts": {
    "start": "ts-node ./src/main.ts"
}
```

Finally install **aerographql-server**:

```
yarn add aerographql-server
```

> Note that this will also install others AeroGraphQL packages:
>
> * aerographql-core
> * aerographql-schema
> 
> Which will be used later on


## Create a simple user type

Now let's write some code:

First start by creating the user type definition in **main.ts**:

```javascript
import { ObjectDefinition } from 'aerographql-schema';

@ObjectDefinition( {
    name: 'User'
} )
export class User {

}
```
This will create a new Typesctipt class and will implicitly create the corresponding GraphQL object type.

You may have noticed the **name** field in the decorators parameters:  
It define the name of the GraphQL object this decorated class refers to.

In this case, a GraphQL object with the name **User** will be created.

> ---
> * **@ObjectDefinition** allow to declare a new GraphQL Type
> * The graphql object name is defined by the name field of the decorator (Not the name of the class)
> ---

## Declare fields

For now we just have empty objects, let's add some fields to our user using the **@Field** decorator:

```javascript
import { Field, ObjectDefinition, ObjectImplementation } from 'aerographql-schema';

@ObjectDefinition( {
    name: 'User'
} )
export class User {
    @Field( { type: 'ID' }) id: string;
    @Field() name: string = "";
    @Field() description: string = "Empty description";
    @Field() age: number = 0;
    @Field() admin: boolean = false;
}
```
As you have noticed, the **type** field on the **@Field** decorator is optional.  
Aerographql will try to guess it using it's type annotation.

This work well for string, number, boolean.  
But for the **id** field, we like to tell GraphQL that this particular field is of type **ID**, so we overload the default type here.

We could also use the **ID** type provided by AeroGraphQL instead of the **'ID'** string, avoiding us the overload:
```
import { ID } from 'aerographql-server';
..
   @Field( ) id: ID;
..
```

> ---
> * **@Field** decorate allow to create a simple GraphQL field.
> * **@Field** are only defined within an @ObjectDefinition class
> ---

**On thing to note:**  
At this point the typescript **User** class is perfectly valid and can be used as any regular javascript object:  
`let myUser = new User()`
And you'll have a brand new object representing a user, with all it's fields already initialized...
This will be pretty usefull when implementing complex resolvers later on...


## Create the Todo object

Using what we already know, let's create the todo object the same way:

```javascript
import { Field, ObjectDefinition } from 'aerographql-schema';

@ObjectDefinition( {
    name: 'Todo'
} )
export class Todo {
    @Field( { type: 'ID' }) id: string;
    @Field() title: string = "";
    @Field() content: string = "Empty todo";
}
```
Pretty straightforward.

## Create our first resolver

Now it would be nice if we could query every todo of a given user.

To do that we need GraphQL **resolvers**.

AeroGraphQL wrap GraphQL resolvers in an elegant way.   
Let's explore that:

```javascript
import { Resolver, Arg, ObjectImplementation } from 'aerographql-schema';

@ObjectImplementation( {
    name: 'User'
} )
export class UserImpl {

    @Resolver( { type: Todo } ) 
    todos( @Arg( { nullable: true }) search: string ) {
        return null;
    }
}
```

This is a work in progress task, but let's explain each parts:

First, use **@ObjectImplementation** to declare a class that will act as a container storing each resolver associated with the GraphQL type specified in the **name** attributes.

Then use the **@Resolver** decorator to declare a specific resolver.
> Reminder: In GraphQL a resolver is the method called to retrieve the value of a given field.

The name of the field associated with this resolver is the name of the method, here **todos**.

The type of this field is defined in the **@Resolver** parameter's **type** attribute, here **Todo**.  

*Note that here we pass a class not a string.  
We could also use string: `@Resolver( { type: 'Todo' } )`  
While this sound similar, the first way is the prefered one, as it will be more easier for your IDE to refactor code and to track changes.*

Finally, we use the **@Arg** decorator to define an argument for this field:  
* The name of the argument in GraphQL will be the name of the argument as defined here in Typescript.
* It's type will be infered by AeroGraphQL from the Typescript type annotation
* It will be a nullable parameter ( **nullable: true** )


> ---
> * **@Resolver** decorator allow to create a field on a GraphQL object type and to provide it's resolver implementation.
> * **@Resolver** are only defined within an @Ibjectimplementation class
> * **@Arg** decorator define an argument for the resolver.
> ---

At this point the core implementation of this resolver is missing, we will come to that later on.

## Create a RootQuery type:
```javascript
import { Field, ObjectImplementation } from 'aerographql-schema';

@ObjectImplementation( {
    name: 'RootQuery'
} )
export class RootQuery {

    @Resolver( { type: User } )
    user( @Arg() name: string ): User | Promise<User> {
        return null;
    }
}
```

Here we define a new GraphQL Object **RootQuery** with just one field **user**.
It's resolver will allow us to query for a given user using it's name as parameter.

> Note that the type of this resolver is **User** this mean that this resolver will either return a plain User object or a promise on a User object

## Create the schema

Now that all the pieces are here, let's stitch them all together using a schema:
```javascript
import { Schema } from 'aerographql-schema';

@Schema( {
    rootQuery: 'RootQuery',
    components: [ RootQuery, User, UserImpl, Todo ]
} )
export class MySchema {
}
```

Here we declare a GraphQL shema composed of various components listed in the **components** array.

Once again, pretty straightforward.  
Note that except for the **rootQuery** attribute, everything is described using explicit types, once again, this will helps during refactoring times.

## Expose this schema 

Now let's expose this schema on a GraphQL server:

```javascript
import { ApolloServer, BaseApolloServer } from 'aerographql-server';
import * as express from 'express';
import * as bodyParser from 'body-parser';

@ApolloServer( {
    name: 'Aerograph server',
    schema: MySchema
} )
export class Server extends BaseApolloServer {

}
```
Here we create a server based on Apollo server to expose our schema.

Now let's plug that into a traditional express application

```javascript
let server = new Server();
this.app = express();
this.app.use( '/graphql', bodyParser.json(), server.getGraphQLMiddleware() );
this.app.use( '/graphiql', server.getGraphiQLMiddleware() );
this.app.listen( 3000, () => {
    console.log( 'Up and running !' );
} );
```

## Implement our resolvers

At this point, you could browse to http://localhost:3000/graphiql to explore our brand new GraphQL API ! Sweet :) 

Congratulation, you now have a GraphQL server up and running, let's try to run some queries on it.  
Try this one in graphiql:

```
{
  user( name: "Bob") {
    id
    name
  }
}
```
This query will return the following error: `Cannot return null for non-nullable field RootQuery.user.`

To solve that, we are going to quickly implement the RootQuery.user resolver.

First add a dummy list of users and then change our resolver implementation:  
*This user list would normally come from your Database using a dedicated service, we will come back to that in the next tutorial on using Dependencies Injection*

```javascript
let users: User[] = [
    { admin: false, age: 25, description: 'Description of Bob', name: 'Bob', id: '0'},
    { admin: true, age: 36, description: 'Description of Alice', name: 'Alice', id: '1'},
    { admin: false, age: 28, description: 'Decription of Steeve', name: 'Steeve', id: '3'}
];

import { Field, ObjectImplementation } from 'aerographql-schema';

@ObjectImplementation( {
    name: 'RootQuery'
} )
export class RootQuery {

    @Resolver( { type: User } )
    user( @Arg() name: string ): User | Promise<User> {
        return users.find( u => u.name === name );
    }
}
```

Now re run the same previous query ... ;) ... this should now work.

Now let's try this query:
```
{
  user( name: "Bob") {
    id
    name
    todos {
        title
        content
    }
  }
}
```
You should get the following error: `Cannot return null for non-nullable field User.todos.`

Let's implement the user's todos resolver correctly

```javascript
let todos: { [ key: string ]: Todo[] } =
    {
        Bob: [
            { id: '0', title: 'Todo1', content: 'Bob Todo1 content' },
            { id: '1', title: 'Todo2', content: 'Bob Todo2 content' },
            { id: '2', title: 'Todo3', content: 'Bob Todo3 content' }
        ],
        Alice: [
            { id: '3', title: 'Todo1', content: 'Alice Todo1 content' },
            { id: '4', title: 'Todo2', content: 'Alice Todo2 content' },
            { id: '5', title: 'Todo3', content: 'Alice Todo3 content' } ],
        Steeve: [
            { id: '6', title: 'Todo1', content: 'Steeve Todo1 content' },
            { id: '7', title: 'Todo2', content: 'Steeve Todo2 content' },
            { id: '8', title: 'Todo3', content: 'Steeve Todo3 content' } ]
    };

@ObjectImplementation( {
    name: 'User'
} )
export class UserImpl {

    @Resolver( { type: Todo, list: true} )
    todos( user: User, @Arg( { nullable: true } ) search: string ) {
        return todos[user.name];
    }
}
```

Here we have defined a dummy todo list for each user.  
*Once again, this would normaly be fetched from your favorite database, for now we will use this simple stub.*


In the resolver itself, a few things have changed:
* First as this resolver return a list of Todo, we add the **list: true** attribute to the **@Resolver** decorator
* We added a first parameter to our resolver: **name: User**.   
  This first parameter will be the result of the previous resolver in the GraphQL resolution chain.  
  Here this will be the User returned by the RootQuery.user field.   
  Checkout this to learn more about [GraphQL execution model](http://graphql.org/learn/execution/#root-fields-resolvers)

Okay, there we are, now retry the previous query !

Here is the result you should have:
```
{
  "data": {
    "user": {
      "id": "0",
      "name": "Bob",
      "todos": [
        {
          "title": "Todo1",
          "content": "Bob Todo1 content"
        },
        {
          "title": "Todo2",
          "content": "Bob Todo2 content"
        },
        {
          "title": "Todo3",
          "content": "Bob Todo3 content"
        }
      ]
    }
  }
}
```

## Conclusion

***Congratulation, you just setup your first AeroGraphQL server.***

Has you might have seen through this tutorial, you have mainly worked with strongly typed data to define both your GraphQL Schema and theirs implementation.

You know have a glimpse of what AeroGraphQL can bring over using GraphQL directly.

* Strongly typed system
* You can reuse defined type in your resolvers implementation
* Everything is in one place: both the GraphQL schema definition, the Typescript types definitions, and the resolvers implementations.

## What's next

Next tutorial will walk you through using dependencies injection system to enhance your resolver implementation...
