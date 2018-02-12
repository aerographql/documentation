---
title: Test
layout: page
toc: true
---

# Using tests tools

AeroGraphQL provide several tools and utilities to help you when writting unit tests.

Those tools relies on other npm packages that are not installed by default with AeroGraphQL:
* **node-mocks-http**
* **apollo-server-express**

> When writting unit tests, please be sure to install those packages beforehand

Each testing tools can be imported using the following import statement:
`import * as Test from "aerographql/dist/test";`

> In the following example we are using [Jest](https://facebook.github.io/jest/) as our testing library along with [ts-jest](https://github.com/kulshekhar/ts-jest) 

## Testing Middleware

To test middleware in isolation, use the **executeMiddlewares** tools.

It's signature is:
` function executeMiddlewares(descs: MiddlewareDescriptor[], args?: ExecuteMiddlewaresArgs, additionalProviders?: (Function | Provider)[]): Promise<any[]>;`

**example:**
```typescript
import { MiddlewareDescriptor, Middleware, MiddlewareInterface, } from './middleware';
import * as Test from 'aerographql/dist/test';

it( 'should return the right value inthe context', async () => {

    @Middleware()
    class MA implements MiddlewareInterface<string> {
        execute( src: any, args: any, context: any, options: any ) { return 'A'; }
    }
    let descs: MiddlewareDescriptor[] = [ { middleware: MA, options: 'Options', resultName: 'result' } ];

    let context:any = {};
    let p = Test.executeMiddlewares( descs, { context } ).then( ( r ) => {
        expect( context ).toHaveProperty( 'result');
        expect( context.result ).toEqual( 'A' )
        return r;
    } );
    return expect( p ).resolves.toBeTruthy();

} );
```

Here we want to test the **MA** middleware and ensure that it return the correct return value.

To do that, we provide the **executeMiddlewares** a list of **MiddlewareDescriptor** and a context object which will be passed down to the middleware execute function.

The **executeMiddlewares** return a Promise that we will use to check the result of the context after execution of the middleware.

Note that we pass the **resultName** option to the descriptor in order to specify where to store the middleware result in the context.
