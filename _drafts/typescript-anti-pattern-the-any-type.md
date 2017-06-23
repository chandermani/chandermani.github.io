If you are doing extensive typescript development and have a codebase that is mix of typescript and javascript, I am sure you'll be using the `any` type quite often. Mostly to reference javascript objects/functions in typescript.

While using `any` is about convenience, I consider it as an anti-pattern, and this post all about *why* and *what* can be done to avoid the `any` trap.

Let's understand what does typescript documentation say about `any`.

> The `any` type is a powerful way to work with existing JavaScript, allowing you to gradually opt-in and opt-out of type-checking during compilation.

The except seems to be validating the fact that `any` allows us to use javascript code in typescript without the type safety net in place.

**Then why is it an anti-pattern?**

There is a very simple and obvious reason:

**With `any` we lose type safety, which can be avoided with a little more effort.** 

I can even argue that if you are using too much of `any` you are actually writing javascript in typescript. I also consider it an anti-pattern because there are some better ways to keep the type safety intact even when working side by side with javascript code.

How do we avoid `any`?

By simply defining the interface definition for any javascript code we use. 

You maybe familiar with the approach of including *type definitions* (or *typings*) for third party javascript libraries that you have been using with your typescript code. There are typings available for a number of popular javascript libraries such as jquery, angular, react and many other. You include these type definitions as when working with these libraries in your typescript code.

We need to do something similar for our own javascript code that we reference in typescript. Let's take a quick example. Assuming you have a javascript `UserService` class defined with few function and attributes.

```javascript
function UserService() {
    this.endPoint='/api/users/';
    this.getUsers = function() { ... }
    this.getUser = function(userId) { ... }
    this.createUser = function(user) { ... }
}
```
A type definition for the same `UserService` in typescript would be:

```typescript
export interface UserService {
    endpoint:string;
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

While defining type/interface for every javascript code snippet we reference may seem an overkill, in long run it leads to a more maintainable code. Also remember, we do not have to create the complete type definition for the javascript class we are using, and can limit ourselves to properties and function that are used with out typescript code. Type definition generation can be a gradual process.