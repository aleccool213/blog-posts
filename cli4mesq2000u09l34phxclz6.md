---
title: "Using EMCAScript decorators in TypeScript 5.0"
datePublished: Fri Apr 28 2023 13:43:13 GMT+0000 (Coordinated Universal Time)
cuid: cli4mesq2000u09l34phxclz6
slug: using-emcascript-decorators-in-typescript-50
canonical: https://blog.logrocket.com/using-modern-decorators-typescript/
tags: news, typescript, decorators

---

[The State of Developer Ecosystem 2022](https://www.jetbrains.com/lp/devecosystem-2022/) crowned TypeScript the fastest-growing programming language. It’s not hard to see why. This popular superset of JavaScript provides type-checking, enums, and other enhancements. But often, TypeScript introduces long-awaited features that are not yet part of the ECMAScript standard that JavaScript relies on.

One example is the reintroduction of decorators in the [soon-to-be-released TypeScript 5.0](https://devblogs.microsoft.com/typescript/announcing-typescript-5-0-rc/); decorators is a meta-programming technique that can be found in other programming languages. If you’re an application developer or library author who is interested in using the new official TypeScript decorators, you’ll want to adopt the new syntax and understand the differences between the old and new feature sets. The API differences are extensive and it is unlikely that old decorators will work with the new syntax out of the box.

In this article, we’ll review the history of using decorators in TypeScript, discuss the benefits associated with decorators in TypeScript 5.0, walk through a demo using modern decorators, and explore how to refactor existing decorators.

***N.B.,*** *all the APIs have changed extensively in TypeScript 5.0; for this article, we’ll focus on class method decorators*

*Jump ahead:*

* History of TypeScript Decorators
    
* Decorators in TypeScript 5.0
    
* Decorator factory demo
    
* Refactoring existing decorators
    
* Understanding the limitations of modern decorators
    

## History of TypeScript Decorators

[Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html) is a feature that enables developers to reduce boilerplate by quickly adding functionality to classes, class properties, and class methods. When TypeScript first introduced decorators it did not follow the ECMAScript specification. This wasn’t great for developers, since ideally emitted code from any JavaScript compiler should comply with web standards!

Using decorators required setting an --experimentalDecorators experimental compiler flag. Several popular TypeScript libraries, such as [type-graphql](https://typegraphql.com/) and [inversify](https://inversify.io/), rely on this implementation.

Here’s an example of a simple class method decorator, demonstrating the enhanced ergonomics of the new syntax:

```typescript
function debugMethod(
  _target: unknown,
  memberName: string,
  propertyDescriptor: PropertyDescriptor
) {
  return {
    get() {
      const wrapperFunction = (...arguments_: unknown[]) => {
        const now = new Date(Date.now());

        console.log("start time", now.toISOString());

        propertyDescriptor.value.apply(this, arguments_);

        const end = new Date(Date.now());

        console.log("end time", end.toISOString());
      };

      Object.defineProperty(this, memberName, {
        value: wrapperFunction,

        configurable: true,

        writable: true,
      });

      return wrapperFunction;
    },
  };
}

class ComplexClass {
  @debugMethod
  public complexMethod(a: number): void {
    console.log("DOING COMPLEX STUFF!");
  }
}
```

In the above code, we can see that the `debugMethod` decorator overrides the class method property using Object.defineProperty, but in general, the code isn’t easy to follow. Also, the arguments are not type-safe, which limits our safety inside the `wrapperFunction.` Additionally, the compiler will not fail if this decorator is used on an invalid use case, such as a class property.

We could [use TypeScript generics](https://blog.logrocket.com/using-typescript-generic-type-create-reusable-components/) to try to achieve type safety, but TypeScript does not infer generic types and this makes them a pain to consume. Thus, writing complex decorators is difficult due to the unknown values users can input into them.

The modern version of decorators, which will be officially rolled out in TypeScript 5.0, no longer requires a compiler flag and follows [the official ECMAScript Stage-3 proposal](https://github.com/tc39/proposal-decorators). Alongside a stable implementation that follows ECMAScript standards, decorators now work seamlessly with the TypeScript type system, enabling more enhanced functionality than the original version.

With the new implementation of decorators in TypeScript 5.0, these aspects are greatly improved. Let’s take a look.

## Decorators in TypeScript 5.0

TypeScript 5.0 offers better ergonomics, improved type safety, and more. Here’s a similar example of a TypeScript 5.0 decorator that overrides a class method:

```typescript
function debugMethod(originalMethod: any, _context: any) {
  function replacementMethod(this: any, ...args: any[]) {
    const now = new Date(Date.now());

    console.log("start time", now.toISOString());

    const result = originalMethod.call(this, ...args);

    const end = new Date(Date.now());

    console.log("end time", end.toISOString());

    return result;
  }

  return replacementMethod;
}

class ComplexClass {
  @debugMethod
  complexMethod(a: number): void {
    console.log("DOING STUFF!");
  }
}
```

***N.B.,*** *to* [*try out TypeScript in an online playground*](https://www.typescriptlang.org/play)*, just switch the version to “nightly” or “&gt;5.0”*

With the new implementation, simply returning the function can now replace it; there’s no need for the Object.defineProperty. This makes decorators easier to implement and understand. Alongside this improvement, let’s make it completely type-safe:

```typescript
function debugMethod<
  TThis,
  TArgs extends [string, number],
  TReturn extends number
>(
  originalMethod: Function,

  context: ClassMethodDecoratorContext<
    TThis,
    (this: TThis, ...args: TArgs) => TReturn
  >
) {
  function replacementMethod(this: TThis, a: TArgs[0], b: TArgs[1]): TReturn {
    const now = new Date(Date.now());

    console.log("start time", now.toISOString());

    const result = originalMethod.call(this, a, b);

    const end = new Date(Date.now());

    console.log("end time", end.toISOString());

    return result;
  }

  return replacementMethod;
}
```

Our decorator function in TypeScript 5.0 is greatly improved and now supports the following:

* Using generics to type a method’s arguments and return a value; the method must accept a string and a number, `TArgs`, and return a number, `TReturn`
    
* Typing the `originalMethod` as a `Function`
    
* Using the `ClassMethodDecoratorContext` inbuilt helper type; this exists for all decorator types
    

We can test to see if our decorator is truly type-safe by inspecting errors when it is used incorrectly:

![Using the TypeScript 5.0 decorator with incorrectly typed arguments.](https://paper-attachments.dropboxusercontent.com/s_19D4B78D1F1C72C5172016AA8E1797DE2FBDF440146335E743AA96678C0F41DC_1678709249215_Xnip2023-03-12_14-15-40.jpg align="left")

Now, let’s look at an actual use case for the new TypeScript 5.0 decorators.

## Decorator factory demo

We can use the type safety available in the TypeScript 5.0 decorators to create functions that return a decorator, otherwise known as a [decorator factory](https://blog.logrocket.com/practical-guide-typescript-decorators/#use-cases-typescript-decorators). Decorator factories allow us to customize the behavior of our decorators by passing some parameters in the factory.

For our demo, we’ll create a decorator factory that changes the class method argument based on its own arguments. This is possible with a TypeScript type ternary operator. Our example is inspired by REST API frameworks like NestJS.

We’ll call our module rest-framework. Let’s start by creating a blank TypeScript project using ts-node:

```bash
$ mkdir rest-framework

$ cd rest-framework

$ npm init -y

$ npm install -D typescript@5.0.4 @types/node ts-node

$ touch index.ts

$ echo "console.log('Hello, world!');" > index.ts
```

Next, we’ll define the script to build and run the project in package.json:

```json
{

  // ...

  "scripts": {

    "build": "tsc",

    "start": "ts-node index.ts"

  }

}
```

Let’s run npm start to see it in action:

```bash
$ npm start


Hello, world!
```

Now, let’s define our types:

```typescript
interface RouteOptionsAuthEnabled {
  auth: true;
}

interface RouteOptionsAuthDisabled {
  auth: false;
}

type RouteArguments = [string] | [];

type RouteDecorator<TThis, TArgs extends RouteArguments> = (
  originalMethod: Function,

  context: ClassMethodDecoratorContext<
    TThis,
    (this: TThis, ...args: TArgs) => string
  >
) => void;

// Next, let’s define the factory decorator:

function Route<
  TThis, // The user can enable or disable auth
  TOptions extends RouteOptionsAuthEnabled | RouteOptionsAuthDisabled
>(
  options: TOptions
): RouteDecorator<
  TThis, // Do not accept a function that uses a string for an argument if auth is disabled
  TOptions extends RouteOptionsAuthEnabled ? [string] : []
> {
  return <TThis>(
    target: (
      this: TThis,

      ...args: TOptions extends RouteOptionsAuthEnabled ? [string] : []
    ) => string,

    context: ClassMethodDecoratorContext<
      TThis,
      (
        this: TThis,

        ...args: TOptions extends RouteOptionsAuthEnabled ? [string] : []
      ) => string
    >
  ) => {};
}
```

Now we have a route decorator that changes the class method parameter types depending on the user’s options.

Let’s create an example Route class to act as our test case:

```typescript
class Controller {
  @Route({ auth: true })
  get(authHeaderValue: string): string {
    console.log("get http method handled!");

    return "response";
  }

  @Route({ auth: false })
  post(): string {
    console.log("post http method handled!");

    return "response";
  }
}
```

We can see that TypeScript fails to compile if we try to use `authHeaderValue` in the post route:

![](https://paper-attachments.dropboxusercontent.com/s_19D4B78D1F1C72C5172016AA8E1797DE2FBDF440146335E743AA96678C0F41DC_1681851657521_image.png align="left")

The decorator factory use case is a simple example, but it demonstrates the power of what type-safe decorators can do.

## Refactoring existing decorators

If you’re using an existing TypeScript decorator, you’ll want to refactor to use the API and take advantage of its many benefits. Basic decorators can be easily refactored to the new ones, but the difference is substantial enough that advanced use cases will take effort.

For best results, follow these steps to refactor existing decorators:

* Write unit tests for your decorators
    
* Remove or falsify the experimentalDecorators TypeScript compiler flags
    
* Read this extensive [summary of how the new proposal works](https://2ality.com/2022/10/javascript-decorators.html)
    
* Understand the limitations of modern decorators (we’ll cover this in more detail later in this article)
    
* Rewrite decorators using no types and use any in place of all types
    
* Make sure unit tests pass
    
* Add types
    

## Understanding the limitations of modern decorators

The modern decorator implementation is great news for TypeScript developers, but notable features are missing. First, there’s no support for decorating method parameters. This is within the spec of the proposal, so hopefully it will be included in the final spec. Its omission is notable because popular libraries, like type-graphql, utilize this in important ways, such as writing resolvers:

```typescript
class Resolver {
  @Query((returns) => Recipe)
  async recipe(@Arg("recipeId") recipeId: string) {
    return this.recipeRepository.findOneById(recipeId);
  }
}
```

Second, TypeScript 5.0 cannot emit decorator metadata. Subsequently, it doesn’t integrate with the [Reflect API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect) and won’t work with the [reflect-metadata](https://github.com/rbuckton/reflect-metadata) npm package.

Third, the --emitDecoratorMetadata flag, which was previously used to access and modify metadata for given decorators, is no longer supported. Unfortunately, there’s no real way to achieve the same functionality by getting the metadata at runtime. Some cases can be refactored. For example, let's define a decorator that validates a function’s parameter types at runtime:

```typescript
function validateParameterType(
  target: any,
  propertyKey: string | symbol
): void {
  const methodParameterTypes: (string | unknown)[] =
    Reflect.getMetadata("design:paramtypes", target, propertyKey) ?? [];

  const firstParameterType = methodParameterTypes[0];

  if (typeof firstParameterType !== "string") {
    throw new TypeError("First parameter must be a string");
  }
}
```

We can achieve similar functionality with the improved type safety provided by TypeScript 5.0. We simply add the arguments of the method we are decorating, like so:

```typescript
function debugMethod<TThis, TArgs extends [string], TReturn>(
) {

...
```

In theory, we could use this approach to refactor decorators that depend on getting types from Reflect for `design:type`, `design:paramtypes`, and `design:returntype`. This is a different way to write decorators; it is not a simple refactor because it requires using TypeScript type inference to refactor how types are acquired and validated.

## Conclusion

The new decorator implementation in TypeScript 5.0 follows the official ECMAScript Stage-3 proposal and is now type-safe, making it easier to implement and understand. However, some notable features are missing, such as support for decorating method parameters and the ability to emit decorator metadata.

Basic decorators can be easily refactored to the TypeScript 5.0 version, but advanced use cases will require more effort. Developers can refactor existing decorators to use the new API and take advantage of the associated benefits. They can be less dependent on external libraries and are less likely to refactor code in the future. These changes to TypeScript's implementation of decorators are a benefit to the broader ecosystem, but community adoption could take some time.