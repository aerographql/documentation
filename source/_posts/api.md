---
title: API
layout: page
toc: true
---

<a class='anchor' id='Public-API'></a>

# Public API

<a class='anchor' id='Decorators'></a>

## Decorators

<a class='anchor' id='Arg'></a>

#### @Arg

`@Arg( config: ArgConfig )`

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

<a class='anchor' id='Field'></a>

#### @Field

`@Field( config: FieldConfig )`

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

<a class='anchor' id='Inject'></a>

#### @Inject

`@Inject( token: string )`

Apply this decorator on class constructor' s parameter to explicitly define the token that will be used to lookup the dependency in the Injector.

**example:**
```
class MyService {
    constructor( @Inject( 'MongoDBUserService' ) private userService: UserService ) {}
}
```
In this case the DI system will look for a dependency identified by the **MongoDBUserService** token and inject it for the **userService** parameter


<a class='anchor' id='Injectable'></a>

#### @Injectable

`@Injectable( )`

Apply this decorator on a class to tag to make it injectable within the dependency injection system.  

<a class='anchor' id='InputObject'></a>

#### @InputObject

`@InputObject( config: InputObjectConfig )`

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

<a class='anchor' id='Interface'></a>

#### @Interface

`@Interface( config: InterfaceConfig )`

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

* **resolveType:** GraphQL use a custom logic to deduce the type of an object when a field return a polymorphic type (like an interface). See [this](http://localhost:4000/documentation/interface-and-union/#How-does-it-work).  
However if this logic does not suit your needs you can override it by providing this callback.  
It's role is to convert an input value to a string corresponding to the GraphQL object type name.  
*Default to null*

<a class='anchor' id='Middleware'></a>

#### @Middleware

`@Middleware( )`

Apply this decorator on a class to tag it as an AeroGraphQL middleware.  

*Any class decorated with **@Schema** must implement the MiddlewareInterface interface.*


<a class='anchor' id='ObjectDefinition'></a>

#### @ObjectDefinition

`@ObjectDefinition( config: ObjectDefinitionConfig )`

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


<a class='anchor' id='ObjectImplementation'></a>

#### @ObjectImplementation

`@ObjectImplementation( config: ObjectImplementationConfig )`

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
    middleware: Function;
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


<a class='anchor' id='Resolver'></a>

#### @Resolver

`@Resolver( config: ResolverConfig )`

Apply this decorator on methods of a class decorated with **@ObjectImplementation** or with **@Interface** to associate it with a GraphQL resolver.

The return value of a method decorated with **@Resolver** will be returned as the result of the Query on this field.  

If the method return a **Promise** AeroGraphQL will:
* Use the resolved value as the result of the Query if the Promise is resolved.
* Use the rejected value as a GraphQL error if the Promise is rejected.



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
A reference to an AeroGraphQL annotated class can be passed.  
*Default to the type of the attribute infered from Typescript metadata*

* **nullable:** Define whether this field is nullable or not.    
*Default to false*

* **list:** Description whether this field is a list or not.  
*Default value is infered by looking at the type of the return value of this function using Typescript metadata.*

* **description:** Description associated with this GraphQL resolver.  
*Default to null*

* **middlewares:** List of middleware that must be called before executing this Resolver.  
**If middleware were defined at the ObjectImplementation level, they will be overided by those one.**  
*Default to []*

<a class='anchor' id='Scalar'></a>

#### @Scalar

`@Scalar( config: ScalarConfig )`

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

<a class='anchor' id='Schema'></a>

#### @Schema

`@Schema( config: SchemaConfig )`

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

* **providers:** Additional providers that must be added to the injector stored in this schema.  
*Default to []*

* **components:** List of reference to decorated class that are part of this schema.

<a class='anchor' id='Union'></a>

#### @Union

`@Union( config: UnionConfig )`

Apply this decorator on a class to create a new GraphQL union.  

```
interface UnionConfig {
    name?: string;
    types: Function[];
    description?: string;
    resolveType?: ResolveTypeFunction;
}
```

* **name:** Name of this union in the GraphQL schema.  
*Default to the name of the class decorated*

* **types:** List of reference to class that are members of this union.

* **description:** Description associated with this GraphQL union.  
*Default to null*

* **resolveType:** GraphQL use a custom logic to deduce the type of an object when a field return a polymorphic type (such as an union).  
However if this logic does not suit your needs you can override it by providing this callback.  
It's role is to convert an input value to a string corresponding to the GraphQL object type name.  
*Default to null*


<a class='anchor' id='Interfaces'></a>

## Interfaces

<a class='anchor' id='MiddlewareDescriptor'></a>

#### MiddlewareDescriptor

```typescript
interface MiddlewareDescriptor {
    middleware: Function;
    options?: any;
    resultName?: string;
}
```

Use this interface to provide informations about how to use a Middleware in either an **@ObjectImplementation** an **@Resolver** or with the **executeMiddlewares** test function.

* **middleware** The middleware class associated
* **options** Optional value passed as the fourth parameter of the middleware execute function
* **resultName** Specify where to store the middleware result in the current GraphQL context. If not specified, result won't be saved.


<a class='anchor' id='MiddlewareInterface'></a>

#### MiddlewareInterface

```typescript
interface MiddlewareInterface<T=any> {
    execute( src: any, args: any, context: any, options: any ): T | Promise<T>;
}
```

Base interface for each Middleware.

The execute function take four parameters:
* **src** The previous object, which for a field on the root Query type is often not used.
* **arg** The arguments provided in the current GraphQL request.
* **context** The GraphQL context object. Note the when multiple Middleware are executed sequentialy, you can check the content of the context for result of previous middleware.
* **options** The option object passed to the **MiddlewareDescriptor** if any.


<a class='anchor' id='ScalarInterface'></a>

#### ScalarInterface

```typescript
interface ScalarInterface {
    parseValue: ( value: any ) => any;
    serialize: ( value: any ) => any;
    parseLiteral: ( ast: ASTNode ) => any;
}
```

Any class decorated with **@Scalar** must implement this interface.

See [this medium post](https://medium.com/graphql-mastery/how-to-design-graphql-queries-and-mutations-part-3-custom-scalars-78d441869258) to learn more about how to implement those methods.


<a class='anchor' id='Class'></a>

## Class

<a class='anchor' id='ID'></a>

#### ID
```typescript
class ID extends String { }
```
Use this class when defining fields to avoid the need to explictly specify type information:
```typescript
// With explicit type information
@Field({type: 'ID' }) id: string;

// With implicit type information
import { ID } from 'aerographql';
@Field() id: ID;
```

<a class='anchor' id='Float'></a>

#### Float
```typescript
class Float extends Number { }
```
Use this class when defining fields to avoid the need to explictly specify type information:
```typescript
// With explicit type information
@Field({type: 'Float' }) id: number;

// With implicit type information
import { Float } from 'aerographql';
@Field() floatField: Float;
```

<a class='anchor' id='Int'></a>

#### Int
```typescript
class Int extends Number { }
```
Use this class when defining fields to avoid the need to explictly specify type information:
```typescript
// With explicit type information
@Field({type: 'Int' }) id: number;

// With implicit type information
import { Int } from 'aerographql';
@Field() intField: Int;
```

<a class='anchor' id='BaseSchema'></a>

#### BaseSchema

```typescript
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


<a class='anchor' id='Test-API'></a>

# Test API

<a class='anchor' id='Class-1'></a>

## Class

<a class='anchor' id='TestServer'></a>

#### TestServer

```typescript
class TestServer {
    constructor( schema: BaseSchema, context: any = null );
    execute<T = any>( query: string, vars: any = null, operationName: string = null ) : Promise<T> ;
}
```

Use this class to write end-to-end test for a given GraphQL query.  

**example:**
```typescript
it( 'should execute query correctly', () => {
    let schema = new MySchema();
    let s = new TestServer( schema );
    let q = `{ query { fieldA fieldB } }`
    return expect( s.execute( q ) ).resolves.toEqual( { data: { query2: { fieldA: 0, fieldB: 0 } } } );
} )
```

<a class='anchor' id='Function'></a>

## Function


<a class='anchor' id='createInjectable'></a>

#### createInjectable

```typescript
function createInjectable<T=any>( ctr: Function, additionalProviders: ( Function | Provider )[] = [] ): T 
```

<a class='anchor' id='executeMiddlewares'></a>

#### executeMiddlewares

```typescript
interface ExecuteMiddlewaresArgs {
    source?: any;
    args?: any;
    context?: any
}

function executeMiddlewares( descs: MiddlewareDescriptor[], args: ExecuteMiddlewaresArgs = {}, additionalProviders: ( Function | Provider )[] = [] ): Promise<any[]>
```

Create and configure a middleware chain ready to be tested.

**example:**
```typescript
it( 'should store middlewares results', () => {
    @Middleware()
    class MA implements MiddlewareInterface<string> {
        execute( src: any, args: any, context: any, options: any ) { return 'A'; }
    }

    let descs: MiddlewareDescriptor[] = [
        { middleware: MA,  resultName: 'A' }
    ];
    let context: any = {};
    let p = executeMiddlewares( descs, { context } ).then( ( result ) => {
        expect( context ).toEqual( { A: 'A' } );
        return result;
    } );
    return expect( p ).resolves.toBeTruthy();
} );
```


