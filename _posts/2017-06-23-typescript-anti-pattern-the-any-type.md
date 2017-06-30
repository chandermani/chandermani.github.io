---
layout: post
title: TypeScript anti-pattern, the 'any' type
author: Chandermani
tags:
- typescript
- javascript
- patterns
- anti-patterns
- typings
---

If you are doing extensive typescript development and have a codebase that is mix of typescript and javascript, I am sure you'll be using the `any` type quite often. Mostly to reference javascript objects/functions in typescript.

While using `any` is about convenience, **I consider it as an anti-pattern**, and this post all about _why_ and _what_ can be done to avoid the `any` trap.


# The `any` anti-pattern

Let's understand what does typescript documentation say about `any`.

> The `any` type is a powerful way to work with existing JavaScript, allowing you to gradually opt-in and opt-out of type-checking during compilation.

The except seems to be validating the fact that `any` enables us to use javascript code in typescript, albeit without the type safety net in place. **Then why is it an anti-pattern?**

The reason is quite simple!

**With `any` we lose the type safety and hence the benefits that are integral to any type safe language.** 

I'll even argue that if you are using too much of `any`, you are writing javascript in typescript. Instead, with a little bit of effort we can reduce `any` usage to the bare minimum.

How do we avoid `any`?

**By simply creating interface definitions for any javascript code we use.**


# Defining type definitions

We all are familiar with the approach of including *type definitions* (or *typings*) for third party javascript libraries that we use with typescript. Anytime we use popular libraries such as *jquery*, *angular*, *react* in typescript we include the type definitions for these libraries as well.

We need to do something similar with our own javascript code that we reference in typescript. We need to create typings. Let's take a quick example. Assuming you have a javascript `UserService` class defined with few function and attributes.

```javascript
function UserService() {
    this.endPoint = '/api/users/';
    this.getUsers = function() { ... }
    this.getUser = function(userId) { ... }
    this.createUser = function(user) { ... }
}
```
A type definition for the same `UserService` in typescript would be:

```typescript
export interface UserService {
    endpoint: string;
    getUsers();
    getUser(userId: string);
    createUser(user: User);
}

export interface User {
    id: string;
    name: string;
}

```
Now instead of passing `UserService` as `userservice: any` in typescript we can use `userservice: UserService` and get all the intellisense and type safety!

While defining type/interface for every javascript piece that we reference may seem be a daunting task, in the long run it leads to a more maintainable code. Also remember, we do not have to create the complete type definition for the javascript class we are using, and can limit ourselves to defining types for properties/functions that we plan to use in typescript code. Type definition generation can be a gradual process!

Let's make best use of this awesome language!