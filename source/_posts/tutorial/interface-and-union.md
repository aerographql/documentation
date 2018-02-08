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

```javascript

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

```javascript
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

```javascript
@Schema( {
    rootQuery: 'RootQuery',
    components: [ RootQuery, User, UserImpl, Todo, PonctualTodo, RecurentTodo ]
} )
export class MySchema extends BaseSchema {
}

```

## Update the fake database:

Update our fake database with the new fields:

```javascript

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

If you have experiences with GraphQL you might be asking, but you do we guess the type of each particular items ?  
After all they are just plain Javascript object, without anything special...

Well AeroGraphQL add it's own logic to deduce the Type of a given object.

It will try several approach to guess it:

1. If the returned value was created using the **new operator** on an object annotated with either @ObjectDefinition or @ObjectImplementation, then AeroGraphQL will look for specific metadata to infer the type.

2. If ther returned value was a literal javascript object, AeroGraphQL will look for a special field called **__typename** containing ... the type name

3. If the __typename does not exits, AeroGraphQL will look for a **customResolveType** field in the **@Interface** or **@Union** decorator configuration for a function with the following signature: `type ResolveTypeFunction = ( value: any, context: any, info: any ) => string;`.  
The implementer should write the function with the approriate logic to detect the object type from the *value* parameter

4. And finally, if neither **__typename** nor **customResolveType** was provided, AeroGraphQL will use a simple heuristic to find it's type by analyzing the shape of each type implementing a common interface and by finding a unique field on each type that could be used as a discriminant to infer the object's type.  
This is what happen here.  
*This is one point that make AeroGraphQL easy to use*
**This computation happen at Startup, so there is nearly zero overhead for that.**

## What's next

Next {% post_link tutorial/dependency-injection tutorial %} will walk you through using the dependency injection system to enhance your resolvers..
