---
title: Home
layout: page
toc: false
---

#### **AeroGraphQL is not yet ready for production and currently under development.**

# What is AeroGraphQL ?

> **AeroGraphQL** is a small and opinionated [Typescript](https://www.typescriptlang.org/index.html) toolkit to create [GraphQL](http://graphql.org/learn/) server using a declarative approach.


* Define your [GraphQL schema](http://graphql.org/learn/schema/) using Typescript annotations and integrate seamlessly your [GraphQL resolvers](http://graphql.org/learn/execution/) using those same types, directly in Typescript.

* Use the builtin {% post_link tutorial/middleware middleware mecanism %} to implement authentication, authorization, guards or resolvers logics.

* Organize and test your code easily with the provided {% post_link tutorial/dependency-injection dependency Injection %} system and the {% post_link notes/test test utilities %}.

* Run your server using any GraphQL server, like the well known [Apollo Server](https://www.apollographql.com/docs/apollo-server/).

# What does it look like ?

The goal of AeroGraphQL is to automaticly create the GraphQL schema while you write your Typescript *business objects*, so you don't need to maintain a separate GraphQL schema definition aside of their implementations.

First, define your schema object's types using Typescript annotation:

```typescript
@ObjectDefinition( { name: 'User' } )
export class UserType {
    @Field( { type: 'ID' } ) id: string;
    @Field( ) name: String;
    @Field( { nullable: true }) description: String;
}
```

Then implement your resolver:

```typescript
@ObjectImplementation( { name: 'RootQuery' } )
export class RootQuery {

    constructor( private userService: UserService ) { }

    @Resolver( { type: UserType } )
    user( @Arg() name: String ) {
        return this.userService.fetchTheUserFromDB( name );
    }
}
```

Compose your Schema:

```typescript
@Schema( {
    rootQuery: 'RootQuery',
    components:[ UserType, RootQuery ]
} )
export class MySchema extends BaseSchema {
}

```

Then inject this schema in your favorite GraphQL server.  
[Apollo Server](https://www.apollographql.com/docs/apollo-server/) with [Express](http://expressjs.com/fr/) in this case:

```typescript
let mySchema = new MySchema();
this.app = express();
this.app.use( '/graphql', bodyParser.json(), graphqlExpress( { schema: mySchema.graphQLSchema } );
this.app.listen( config.get( 'server.port' ), () => {
    console.log( 'Up and running !' );
} );

```

This server will expose the following GraphQL schema:

```
type User {
    id: ID!
    name: String!
    description: String
}

type RootQuery {
    user( name: String! ): User!
}

schema {
    query: RootQuery
}
```
With each resolvers automaticaly wired to their implementations.

# Where to go next ?

**Follow the {% post_link tutorial/getting-started Getting started guide %} to learn more about AeroGraphQL !**  
or  
**Checkout the {% post_link api API references %}.**
