---
title: Resolve type
layout: page
toc: true
---

# Polymorphic types

The GraphQL spec allow to create polymorphic types.  
By polymorphic types we mean a single type that can have different shapes, this could be:
* A **Union**
* Or an **Interface**

When an AeroGraphQL resolver is returning a Polymorphic type, GraphQL need to know how to infer the type of the concrete object behind this Polymorphic one.

To do that, we use a dedicated function called the **resolveType** function, with the following signature:
`type ResolveTypeFunction = ( value: any, context: any, info: any ) => string`

The goal of this method is to return a string matching the type name of the input value.

It's parameters are:
* **value** The object on which we are trying to find the concrete type
* **context** The context object associated with this request
* **info** The GraphQL specification for this particular field

> Whith a plain GraphQL implementation, it's up to the user to implement this method when necessary.
> **AeroGraphQL provide a default implementation of it which work in most situtations.**

# Default resolveType

The default AeroGraphQL implementation of the resolveType function works as follow:

1. AeroGraphQL will try to infer the type by looking at Metadata associated with the object:  
If the *value* object was created using the **new operator** on a type annotated with AeroGraphQL decorators (@ObjectDefinition, @ObjectImplementation...) , then metadata are availables which contains the type name of this object.  
But what happen if the object was not created using the new operator ?

2. AeroGraphQL will look for a field **__typename** on the object which contain the object type name.

3. If this field does not exist, AeroGraphQL will try to infer the type using the following heuristic:  
> For each concrete type of the polymorphic one, it will search for a field which only exist in a single concrete type and use this type as a discriminant to infer the concrete type.  

**example:**
```typescript
@Interface( { name: 'InterfaceA', description: 'Desc', } )
class InterfaceA {
    @Field( { type: 'Int' } ) fieldA: number;
    @Field( { type: 'String' } ) fieldB: string;
}

@ObjectDefinition( { name: 'TestType1', implements: [ InterfaceA ] } )
class TypeA {
    @Field( { type: 'Int' } ) fieldA: number = 0;
    @Field() fieldB: string = "String";
    @Field() fieldC: string = "String";
}

@ObjectDefinition( { name: 'TestType2', implements: [ InterfaceA ] } )
class TypeB {
    @Field( { type: 'Int' } ) fieldA: number = 0;
    @Field() fieldB: string = "String";
    @Field() fieldD: number = 1;
}
```

* If a field return an **InterfaceA** and the actual object contain a field name **fieldC**, then AeroGraphQL will infer that it's of concrete type **TestType1**.  
* If a field return an **InterfaceA** and the actual object contain a field name **fieldD**, then AeroGraphQL will infer that it's of concrete type **TestType2**.  

**All of the computation needed to determine the discriminant field happen at startup time, so there is nearly zero overhead when using the default implementation of the resolveType**

# Custom resolveType
Obviously this approach will not work everytime. 
If you need a more sophisticated approach you can override the default **resolveType** function with a custom one.

You can do that when defining interfaces or unions by passing a custom function fo the **resolveType** field of the configuration object.
Checkout the API reference for:
* [Interface](https://aerographql.github.io/documentation/api/#Interface)
* [Union](https://aerographql.github.io/documentation/api/#Union)
