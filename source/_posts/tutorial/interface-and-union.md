---
title: Interface and Union
layout: page
toc: true
---

# Interface and Union

This tutorial follow the {% post_link tutorial/getting-started previous one %} to add support for GraphQL Interface and Union.

We'll use the final source code of the {% post_link code/getting-started previous tutorial %} as a starting point.

If you don't know what GraphQL Interface and Input are, read this documentation beforehand:  
* [Official GraphQL doc for Interface](http://graphql.org/learn/schema/#interfaces)
* [Official GraphQL doc for Input](http://graphql.org/learn/schema/#input-types)

We will enhance our Todo list by adding two type of Todo:
* Ponctual Todo
* Recurent Todo

We will use **Interface** to declare a common base between those two inputs.

**The full source code for this tutorial can be found {% post_link code/interface-and-union here %}**.

## Setup our Interface

First let's transform the Todo ObjectDefinition to an Iterface.  
We will use the **@Interface** decorator here:

```typescript

@Interface( {
    name: 'Todo'
} )
export class Todo {
    @Field( { type: 'ID' } ) id: string;
    @Field() title: string = "";
    @Field() content: string = "Empty todo";
}
```

Then let's define both our PonctualTodo and Recurent Todo:

```typescript
@ObjectDefinition( { implements: [Todo]  } )
export class PonctualTodo {
    @Field( { type: 'ID' } ) id: string;
    @Field() title: string = "";
    @Field() content: string = "Empty todo";
    @Field() date: string = "Date";
}

@ObjectDefinition( { implements: [Todo]  } )
export class RecurentTodo {
    @Field( { type: 'ID' } ) id: string;
    @Field() title: string = "";
    @Field() content: string = "Empty todo";
    @Field() occurence: string = "Date";
}
```

Here, we use the **implements** field of the **@ObjectDefinition** to define which interface is implemented by each types.

> Note that the **implements** field is also present on the **@ObjectImplementation**

> Also note that the **name** field on both type is *not* specified.  
In this case, AeroGraphQL will infer the GraphQL name from the Typescript class name.  
Here the GraphQL name will be **RecurentTodo** and **PonctualTodo**.

## Wire them in the Schema:

Don't forget to add those new type in the schema components list:

```typescript
@Schema( {
    rootQuery: 'RootQuery',
    components: [ RootQuery, User, UserImpl, Todo, PonctualTodo, RecurentTodo ]
} )
export class MySchema extends BaseSchema {
}

```

## Update the fake database:

Update our fake database with the new fields:

```typescript

let todos: { [ key: string ]: (PonctualTodo | RecurentTodo) [] } =
    {
        Bob: [
            { id: '0', title: 'Todo1', content: 'Bob Todo1 content', occurence: 'Every Week' },
            { id: '1', title: 'Todo2', content: 'Bob Todo2 content', date: 'Friday'  },
            { id: '2', title: 'Todo3', content: 'Bob Todo3 content', occurence: 'Every Day'  }
        ],
        Alice: [
            { id: '3', title: 'Todo1', content: 'Alice Todo1 content', date: 'Mondy'  },
            { id: '4', title: 'Todo2', content: 'Alice Todo2 content', date: 'Saturday'  },
            { id: '5', title: 'Todo3', content: 'Alice Todo3 content', occurence: 'Every Week'  } ],
        Steeve: [
            { id: '6', title: 'Todo1', content: 'Steeve Todo1 content', occurence: 'Every Month'  },
            { id: '7', title: 'Todo2', content: 'Steeve Todo2 content', date: 'Tuesday'  },
            { id: '8', title: 'Todo3', content: 'Steeve Todo3 content', occurence: 'Every Day'  } ]
    };
```

** That's it ! ** 

You can now query you GraphQL server using [GraphQL query fragment](http://graphql.org/learn/queries/#fragments) to allow polymorphism such as:

```
{
  user( name:"Bob") {
    id
    todos {
      id
      __typename
      ... on PonctualTodo {
        date
      }
      ... on RecurentTodo {
        occurence
      }
    }
  }
}
```

which should return:

```
{
  "data": {
    "user": {
      "id": "0",
      "todos": [
        {
          "id": "0",
          "__typename": "RecurentTodo",
          "occurence": "Every Week"
        },
        {
          "id": "1",
          "__typename": "PonctualTodo",
          "date": "Friday"
        },
        {
          "id": "2",
          "__typename": "RecurentTodo",
          "occurence": "Every Day"
        }
      ]
    }
  }
}
```

## How does it work ?

If you have experiences with plain GraphQL you might be asking you do we guess the type of each particular items ?  
After all they are just plain Javascript object, without anything special...

Well AeroGraphQL add it's own logic to deduce the Type of a given object.

Checkout {% post_link notes/resolve-type this page on resolveType function %} for more informations.

## What's next

Next {% post_link tutorial/dependency-injection tutorial %} will walk you through using the dependency injection system to enhance your resolvers..
