---
title: Api reference
layout: page
toc: true
---

# Decorators

### @Arg( ArgConfig )

Apply this decorator to a **@Resolver** 's parameter to associate it  with a GraphQL argument on the field defined by the resolver.

```
interface ArgConfig {
    nullable?: Boolean;
    list?: boolean;
    type?: string | Function;
}
```

* **nullable:** Define whether this argument is nullable or not.  
*Default to false*

* **list:** Define whether this argument is a list or not.  
*Default value is infered by looking at the type of the parameter where this decorator is applied on, using Typescript metadata.*

* **type:** Define the type of this argument.  
If a string is provided, it must match a name of a GraphQL object available in the final schema.  
*Default to the type of the function parameter infered from Typescript metadata*


### @Field( FieldConfig )

Apply this decorator on a member of a class decorated with **@ObjectDefinition** to associate it with a GraphQL field on the object.

```
interface FieldConfig {
    nullable?: Boolean
    type?: string | Function
    description?: string;
}
```

* **nullable:** Define whether this field is nullable or not.  
*Default to false*

* **type:** Define the type of this field.  
If a string is provided it must match a name of a GraphQL object available in the final schema.  
*Default to the type of the attribute infered from Typescript metadata*

* **description:** Description associated with this GraphQL field.  
*Default to null*


### @InputObject( InputObjectConfig )

Apply this decorator on a class to create a new GraphQL input


```
interface InputObjectConfig {
    name?: string;
    description?: string;
}
```

* **name:** Name of this input in the GraphQL schema  
*Default to the name of the class decorated*

* **description:** Description associated with this GraphQL input.  
*Default to null*

### @Interface( InterfaceConfig )

Apply this decorator on a class to create a new GraphQL interface

```
interface InterfaceConfig {
    name?: string;
    description?: string;
    resolveType?: ResolveTypeFunction;
}

type ResolveTypeFunction = ( value: any, context: any, info: any ) => string;
```

* **name:** Name of this interface in the GraphQL schema  
*Default to the name of the class decorated*

* **description:** Description associated with this GraphQL interface.  
*Default to null*

* **resolveType:** GraphQL use a custom logic to deduce the type of an object when a field return a polymorphic type (like an interface). (see [this](http://localhost:4000/documentation/interface-and-union/#How-does-it-work))  
However if this logic does not suit your needs you can override it by providing this callback.  
It's role is to convert an input value to a string corresponding to the GraphQL object type name.  
*Default to null*


### @ObjectDefinition( ObjectDefinitionConfig )

Apply this decorator on a class to create a new GraphQL object with only fields.  
Or to add fields to an existing GraphQL object with the same name.

```
interface ObjectDefinitionConfig {
    name?: string;
    description?: string;
    implements?: Function[];
}
```

* **name:** Name of the GraphQL object this class refers to  
*Default to the name of the class decorated*

* **description:** Description associated with this GraphQL object.  
*Default to null*

* **implements:** Specify which GraphQL interface this object implement.   
*Default to []*

### @ObjectImplementation( ObjectImplementationConfig )

Apply this decorator on a class to create a new GraphQL object with only resolvers.  
Or to add resolvers to an existing GraphQL object with the same name.

```
interface ObjectDefinitionConfig {
    name?: string;
    description?: string;
    implements?: Function[];
    middlewares?: MiddlewareDescriptor[];
}

export interface MiddlewareDescriptor {
    provider: Function;
    options?: any;
    resultName?: string;
}
```

* **name:** Name of the GraphQL object this class refers to  
*Default to the name of the class decorated*

* **description:** Description associated with this GraphQL object.  
*Default to null*

* **implements:** Specify which GraphQL interface this object implement.   
*Default to []*

* **middlewares:** List of middleware that must be called before executing a resolver in thi ObjectImplementation.  
*Default to []*
    * **provider** is the class constructor for the desired middleware  
    * **options** Option passed to the middleware  
    * **resultName** A field name reachable in the GraphQL context, where the result of this middleware will be stored   

### @Resolver( ResolverConfig )

Apply this decorator on a method of a class decorated with **@ObjectImplementation** to associate it with a GraphQL resolver on the object.

```
interface ResolverConfig {
    type?: string | Function,
    nullable?: boolean,
    list?: boolean,
    description?: string,
    middlewares?: MiddlewareDescriptor[];
}
```

* **type:** Define the return value type of this resolver.  
If a string is provided it must match a name of a GraphQL object available in the final schema.  
A reference to a AeroGraphQL annotated class can be passed.  
*Default to the type of the attribute infered from Typescript metadata*

* **nullable:** Define whether this field is nullable or not.    
*Default to false*

* **list:** Description whether this field is a list or not.  
*Default value is infered by looking at the type of the return value of this function using Typescript metadata.*

* **description:** Description associated with this GraphQL resolver.  
*Default to null*

* **middlewares:** List of middleware that must be called before executing this Resolver.  
If middleware were defined at the ObjectImplementation level, they will be overided by those one.  
*Default to []*

### @Scalar( ScalarConfig )

Apply this decorator on a class to create a new GraphQL scalar.  

*Any class decorated with **@Scalar** must implement the **ScalarInterface** interface.*

```
interface ScalarConfig {
    name: string;
    description?: string;
}
```

* **name:** Name of this scalar in the GraphQL schema  
*Default to the name of the class decorated*

* **description:** Description associated with this GraphQL scalar.  
*Default to null*

### @Schema( SchemaConfig )

Apply this decorator on a class to create a new GraphQL schema.  

*Any class decorated with **@Schema** must extends the BaseSchema class.*

```
interface SchemaConfig {
    rootQuery: string,
    rootMutation?: string,
    providers?: ( Function | Provider )[],
    components: Function[]
}
```

* **rootQuery:** Name of the GraphQL object used as the root query.

* **rootMutation:** Name of the GraphQL object used as the root mutation.  
*Default to null*

* **providers:** Additional providers that must be added to the injector stored in this schema. See [this](http://localhost:4000/documentation/dependency-injection/#Configure-the-injector)  
*Default to []*

* **components:** List of reference to decorated class that are part of this schema.


### @Union( UnionConfig )

Apply this decorator on a class to create a new GraphQL union.  

```
interface UnionConfig {
    name?: string;
    types: Function[];
    description?: string;
    resolveType?: ResolveTypeFunction;
}
```

* **name:** Name of this union in the GraphQL schema  
*Default to the name of the class decorated*

* **types:** List of reference to class that are members of this union

* **description:** Description associated with this GraphQL union.  
*Default to null*

* **resolveType:** GraphQL use a custom logic to deduce the type of an object when a field return a polymorphic type (such as an union). (see [this](http://localhost:4000/documentation/interface-and-union/#How-does-it-work))  
However if this logic does not suit your needs you can override it by providing this callback.  
It's role is to convert an input value to a string corresponding to the GraphQL object type name.  
*Default to null*

# Interfaces

### ScalarInterface

```
interface ScalarInterface {
    parseValue: ( value: any ) => any;
    serialize: ( value: any ) => any;
    parseLiteral: ( ast: ASTNode ) => any;
}
```

Any class decorated with **@Scalar** must implement this interface.

See [this medium post](https://medium.com/graphql-mastery/how-to-design-graphql-queries-and-mutations-part-3-custom-scalars-78d441869258) to learn more about how to implement those methods.

# Class

### BaseSchema

```
class BaseSchema {
    rootInjector: Injector;
    graphQLSchema: GraphQLSchema;
    toString();
}
```

Any class decorated with **@Schema** must derive from this base class.  
After instanciation of a class deriving from **BaseSchema**, the user will have access to:
* **rootInjector** : The internal Injector stored in this schema
* **graphQLSchema** : The standard GraphQL schema that can be later on used by any GraphQL compliant server 
