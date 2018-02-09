---
title: AeroGraphQL
layout: page
toc: false
---

### Before everything:  
**AeroGraphQL is not yet ready for production and currently under development.**

# What is AeroGraphQL ?

> **AeroGraphQL** is a small and opinionated [Typescript](https://www.typescriptlang.org/index.html) toolkit to create [GraphQL](http://graphql.org/learn/) server using a declarative approach.


* Define your [GraphQL schema](http://graphql.org/learn/schema/) using Typesctipt annotations and intergrate seamlessly your [GraphQL resolvers](http://graphql.org/learn/execution/) using those same types, directly in Typescript.

* Implement your authorization strategies using the builtin middleware mecanism.

* Organize and test your code easily with the provided dependecies injection system and the test utilities.

* Run your server using the well known [Apollo Server](https://www.apollographql.com/docs/apollo-server/).

# What it look like ?

Define your schema types using typescript annotation:

```javaScript
@ObjectDefinition( {
    name: 'User'
} )
export class UserType {
    @Field( { type: 'ID' } ) id: string;
    @Field( ) name: String;
    @Field( { nullable: true }) description: String;
}
```

Implement your resolvers:

```javascript
@ObjectImplementation( {
    name: 'RootQuery'
} )
export class RootQuery {

    constructor( private userService: UserService ) {
    }

    @Resolver( { type: UserType } )
    user( @Arg() name: String ) {
        return this.userService.fetchTheUserFromDB( name );
    }
}
```

Compose your Schema:

```javascript
@Schema( {
    rootQuery: 'RootQuery',
    components:[ UserType, RootQuery ]
} )
export class MySchema extends BaseSchema {

}

```

Then inject this schema in your favorite GraphQL server.  
[Apollo Server](https://www.apollographql.com/docs/apollo-server/) with [Express](http://expressjs.com/fr/) in this case:

```javascript
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
With each resolvers automaticaly wired.

# Where to go next ?

**Follow the {% post_link tutorial/getting-started Getting started guide %} to learn more about AeroGraphQL !**
