---
layout: post
title: "Effective Types: Type Wrappers with Flow"
date: "2016-01-11 22:07"
listed: true
published: true
categories:
- Effective Types
- Flow
- JavaScript
- Type Safety
- Newtypes
---

Type Wrappers are a way of taking an existing type (like strings, numbers, etc) and wrapping them in _another_ type, both so they can't be confused with other things of the same type, and to represent things like validation in a way that Flow can check. They are variously known as Nominal Type Aliases, [Opaque Type Aliases](https://docs.hhvm.com/hack/type-aliases/opaque), or [Newtypes](https://wiki.haskell.org/Newtype)<sup><a href="#note-1">1</a></sup>. Let's get to it. 😊


## Creating and Using a Type Wrapper

I have a library called [flow-classy-type-wrapper](https://npmjs.com/package/flow-classy-type-wrapper); towards the end of this post, we'll get into the guts of how this works, but let's arm-wave for the moment. Here's an example of it in use:

```ts
import { TypeWrapper } from 'flow-classy-type-wrapper';

class Email extends TypeWrapper<string> {}

function validateEmail(x: string): (Email | null) {
  // Some very simplistic validation here:
  if (x.match(/@/)) {
    return Email.wrap(x);
  }
  return null;
}

function sendEmail(to: Email) {
  const toAsString = Email.unwrap(to);
  // A stand-in for sending an email:
  console.log("Sending to " + toAsString + "...");
}

const input = "hello@world.com";
const email = validateEmail(input);

if (email === null) {
  throw new Error("Was not email: " + input);
}
sendEmail(email);
```

Let's break this down into smaller pieces:

```ts
class Email extends TypeWrapper<string> {}
```

This extends the imported [`TypeWrapper` class](https://github.com/damncabbage/flow-classy-type-wrapper/blob/master/src/TypeWrapper.js), putting `string` in as its type parameter. If this looks unfamiliar to you, I recommend you [learn about these parameters](/blog/2017/01/effective-types-a-parameterised-type-primer-flow/) before continuing.

In any case: we have a class representing a new type, `Email`, that wraps another, `string`. It has two [static methods](http://odetocode.com/blogs/scott/archive/2015/02/02/static-members-in-es6.aspx) attached to it, `wrap` and `unwrap`. We'll use them in a moment.

```ts
function validateEmail(x: string): (Email | null) {
  if (x.match(/@/)) {
    return Email.wrap(x);
  }
  return null;
}
```

This is a function that accepts a `string`, and spits out either an `Email`, or `null`, depending whether it passes our rather flimsy valid-email check.

To get an `Email` value, we use the aforementioned `wrap` method; its type signature, for our example, looks like `function wrap(x: string): Email`. We use this `wrap` method to "wrap up" our string value so that it can't be directly used as a string anymore; the only thing that can make use of this wrapped-up value are things that expect `Email` specifically, which we'll see an example of below:

```ts
function sendEmail(to: Email) {
  const toAsString = Email.unwrap(to);
  console.log("Sending to " + toAsString + "..."); // <-- A stand-in for sending an email.
}
```

This function accepts an `Email`, and then unwraps it to get at the string which, in our toy example, just gets printed out. Much like `wrap` above, this `unwrap` function has a type signature that looks like `function unwrap(x: Email): string`.

Lastly:

```ts
const input = "hello@world.com";
const email = validateEmail(input);

if (email === null) {
  throw new Error("Was not email: " + input);
}
sendEmail(email);
```

We're pretending to accept user input here (a string). With that input, we put it through our validation function, and are made to (by Flow) handle the possibility of the result being `null`. After we've checked that, we know we have an `Email` value. We pass it on to `sendEmail` to use.

So what is this type wrapping useful for?


## Why

### In-line Documentation

The simplest benefit: it's a way of giving names to values of a particular variety, and goes a little way towards providing some documentation that's also checked by Flow.

On a visual level I'd personally say that a function with a signature `sendEmail(from: Email, to: Email, subject: string)` is easier to read than `sendEmail(to: string, from: string, subject: string)`. We also get the benefits of being able to look up what describes or produces an `Email` specifically, instead of any old string.


### Preventing Mixups

Say we have a function that takes two strings, and returns a potential user:

```ts
function createUser(email: string, password: string): Promise<User> {
  // ...
}

// Just validates and returns a string or null:
function validateEmail(email: string): (string | null) {
  // ...
}

// Much later:

// Let's pretend we got this email+password from user input.
const email = validateEmail("bob@example.com");
const password = "canhefixit";
if (email !== null) {
  const bob = createUser(email, password);
}
```

There's the _possibility_ of getting those strings mixed up, eg. `createUser(password, email)`. It's _unlikely_ in such a simple case, but with more complex ones like `fs.writeFile(string, string, string)` or `fs.writeSync(handle, buffer, number, number, number)` (both from the [Node.js standard library](https://nodejs.org/api/fs.html#fs_fs_writesync_fd_buffer_offset_length_position)), you can see how things can get mixed up if you have a bunch of arguments of the same type.

And yep, you could have them as part of an object with `email` and `password` fields instead, eg. `{ email: string, password: string }`. That helps not mix things up, but the wrapped version of the value is still safe even when _separated_ from that object:

```ts
const args: { email: string, password: string } =
  { email: "bob@example.com", password: "abc" };

const email = args.email;
// ... The type is now just string; we've lost the context.
```

```ts
const args: { email: Email, password: string } =
  { email: Email.wrap("bob@example.com"), password: "abc" };

const email = args.email;
// ... The type is still Email.
```


### Changing Code

We might need to change `createUser` to take a username instead, eg. shifting email off to something a user can add later after signup:

```ts
function createUser(username: string, password: string): Promise<User> {
  // ...
}

function validateEmail(email: string): (string | null) {
  // ...
}

// Much later:
const email = validateEmail("bob@example.com");
const password = "canhefixit";
if (email !== null) {
  const bob = createUser(email, password);
}
```

Uh oh. `bob@example.com` isn't a valid username; suddenly this user creation step fails. Hopefully, if we have tests for this path through the code, we might have caught this. But what if we use a typed wrapper to make it so that Flow tells us *immediately* if we change something like this?

```ts
// ---- Before change: takes an Email ----
function createUser(email: Email, password: string): Promise<User> {
  // ...
}

function validateEmail(email: string): (Email | null) {
  // ...
}

// Much later:
const email = validateEmail("bob@example.com"); // type: Email | null
const password = "canhefixit";
if (email !== null) {
  const bob = createUser(email, password);
}
```

```
// ----- After change: takes a Username -----
function createUser(username: Username, password: string): Promise<User> {
  // ...
}

// Much later:
const email = validateEmail("bob@example.com"); // type: Email | null
const password = "canhefixit";
if (email !== null) {
  const bob = createUser(email, password);
  // 💥 Boom! 💥
  // createUser(email, password);
  //            ^^^^^ Email. This type is incompatible with
  //                  the expected param type of
  // function createUser(username: Username, password: string): Promise<User> {
  //                               ^^^^^^^^ Username
}
```



### "Gatekeeper" Functions

Lets look at `validateEmail` from above

```ts
function validateEmail(x: string): (Email | null) {
  // ...
}
```

We have a function that produces `Email`. What if, to make an `Email`, we had to put the input through the `validateEmail` function?

```
// ----- email.js -----
import { TypeWrapper } from 'flow-classy-type-wrapper';

// Set up our Email type-wrapper around string, but don't export it, instead...
class Email extends TypeWrapper<string> {}

// ... Export only the type, for use in type signatures.
export type EmailType = Email;

// Export the unwrap function.
export const unwrap = Email.unwrap;

// Export the validateEmail function.
export function validateEmail(x: string): (Email | null) {
  // ...
}
```

```ts
// ----- stuff.js -----
// We can only import the type, email validation, and email unwrapping; we
// can't create our own Email anymore.
import type { EmailType as Email } from './email';
import { validateEmail, unwrap }

const email = validateEmail("bob@example.com");
if (email === null) throw new Error("Invalid email");

// We know we have an Email now; we could give it to createUser() here, or pass
// it to somewhere else deep in our program, knowing that it's a Email.
createUser(email, "canhefixit");
```

We only export `validateEmail`, `unwrap` and the `Email` type itself. In doing so, make sure that *any* `Email` values being tossed around our app have been passed through the `validateEmail` function; there's no other way to make one without using an `any` backdoor, and we can use [eslint to warn whenever we try to use these](https://www.npmjs.com/package/eslint-plugin-flowtype#eslint-plugin-flowtype-rules-no-weak-types).


## A Sliding Scale of Safety

As we've just done, you can ratchet up the safety, at the expense of restricting how you construct and use these values. I personally use wrapped types as much as practical, them having saved my butt more than a few times (eg. where a "string" has been passed through a few tiers of functions, to be accidentally mixed up somewhere else). But feel free to use this with some values you want to carefully control, like User IDs, escaped strings, etc.


## So what *are* the Email Values?

Just strings! "Wrapping" the types never transforms them. The idea of `Email` being a separate thing is entirely for Flow's benefit, and evaporates when the program is run.


## What's the Performance Overhead for This?






<small id="note-1">1. I've intentionally avoided calling this Flow technique "newtyping", mostly because Haskell's `newtype` lets you do things that we can't with this, eg. `newtype Foo a = Foo (a -> String)`, where it's not a straight "value wrapping".</small>




For these things:
* Preventing accidents?
* Documentation?
* Giving meaning to a smaller set of values.







## A String's a String's a String

When you're coming up with objects to store data, or taking arguments to a function, it's awfully tempting to just make values or arguments a number or a string or some other simple type. Take this definition for a `getUser()` function, for example:

```javascript
function getUser(id: string): Promise<User> {
  return db.getFirst(
    'SELECT * FROM users where id = :id LIMIT 1',
    { id: id }
  );
}

// eg.
getUser("1234-abcd-1122").then(...)
```

Easy! Except now we've seen the seeds of a problem we might be unlucky enough to run into later.

When we say `id: string`, we're saying that the ID could be any possible string, from a blank string to the Complete Works of Shakespeare. This plainly isn't the case: we have what looks like a GUID or some other "ID"-like string, likely limited to some length, and in this case made up of only hexidecimal digits (0 to 9, A to F) and dashes.

So to convey this information, to both other devs and the computer, you should probably give it a name. And to make sure we can't accidentally mix it up, we should have the computer check that we've not put a string or something else in the place where we expect an ID.




Ideas:
* Currency (that's possibly a newtype thing?)
* Time
* Positive Numbers
  * Validated inputs ---> more a gatekeeper thing?
Get into Phantom types later, eg. for Length of a particular Unit, or Amount of Currency (later, article about generating files with types, almost as a form of config, with lookup functions to convert from string).

http://www.medscape.com/viewarticle/744329_4
https://twitter.com/ethan_agan/status/760580878393307137
https://twitter.com/exposernine/status/758479255969861633
https://twitter.com/tabatkins/status/804163271200645121
http://degoes.net/articles/newtypes-suck
And though we're not making a Mars Climate Orbiter here, we do still mix things up.
A number by itself, a string by itself, is meaningless without context. By wrapping up these raw values in a different type, you can give them meaning that is carried around with them until the last minute, where you unwrap and dump out the raw contents.

explicitly wrap and unwrap to prevent problems where you accidentally use something like a number, but where the internal representation is a number. Just because the internal is something, it doesn't mean it's valid to use it like that. It shouldn't be valid to reverse a UserID GUID, or add one to a Random Number Generator seed (TODO: Link).

mixing up args
type errors with some intent
  how do i make one of these

objects/records/unwrapped are initially fine, but separate them out and the newtypes stay safe where
records don't

So to start with you should really be making sure what you have at least looks like an ID.

We could forget that `getUser()` takes an 


## Type Whatnows?


The safety you get out of this 

One of the nice things about the implementation this wrapper/newtype concept in the languages that have them is that it's usually with no overhead when you're running the code; you run the type-check, it passes, and then the compilation process just makes these wrappers evaporate. And we can pretty much[1] do that in Flow too! Let's find out how.  


### In Flow

```javascript
class UserID {
  constructor(_: empty): void {}
  static wrap(x: string): UserID {
    return ((x: any): UserID);
  }
  static unwrap(x: UserID): string {
    return ((x: any): string);
  }
}

const id = UserID.wrap("123-abcd");
const user = { id: id, name: "Bert" }
const userSummary = UserID.unwrap(id) + ": " + user.name;
```

```javascript
class TypeWrapper<Inner> {
  constructor(_: empty): void {}
  wrap(x: Inner): this {
    return (x: any);
  }
  unwrap(x: this): Inner {
    return (x: any);
  }
}

class UserID extends TypeWrapper<string> {}

const id = UserID.wrap("123-abcd");
const user = { id: id, name: "Bert" }
const userSummary = UserID.unwrap(id) + ": " + user.name;
```




1. When the Flow types are stripped out, you're still left with two things: the class definition (which is tiny, and paid once), and a bunch of function calls that are basically `function(x){ return x; }`. Which sounds wasteful, until you realise that, [in performance tests](https://jsperf.com/damncabbage-x-vs-id-x), the difference [doesn't even clear the margin of error](https://i.imgur.com/2LYwdFG.png).
