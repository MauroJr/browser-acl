# browser-acl

[![build status](http://img.shields.io/travis/mblarsen/browser-acl.svg)](http://travis-ci.org/mblarsen/browser-acl)
[![Coverage Status](https://coveralls.io/repos/github/mblarsen/browser-acl/badge.svg?branch=master)](https://coveralls.io/github/mblarsen/browser-acl?branch=master)
[![Known Vulnerabilities](https://snyk.io/test/github/mblarsen/browser-acl/badge.svg)](https://snyk.io/test/github/mblarsen/browser-acl)
[![NPM version](http://img.shields.io/npm/v/browser-acl.svg)](https://www.npmjs.com/package/browser-acl/) [![](https://img.shields.io/npm/dm/browser-acl.svg)](https://www.npmjs.com/package/browser-acl/)

> Simple ACL library for the browser inspired by Laravel's guards and policies.

Go to [vue-browser-acl](https://github.com/mblarsen/vue-browser-acl) for the official Vue package.

## Install

```
yarn add browser-acl
```

## Setup

```javascript
import Acl from 'browser-acl'
const acl = new Acl()

acl.rule('view', Post)
acl.rule('moderate', Post, (user) => user.isModerator())
acl.rule(['edit', 'delete'], Post, (user, post) => post.userId === user.id)
```

Policies are also supported:

```javascript
acl.policy({
  view: true,
  edit: (user, post) => post.userId === user.id),
}, Post)
```

Note: policies takes precedence over rules.

## Usage

```javascript
// true if user owns post
acl.can(user, 'edit', post)

// true if user owns at least posts
acl.some(user, 'edit', posts)

// true if user owns all posts
acl.every(user, 'edit', posts)
```

You can add mixins to your user class:

```javascript
acl.mixin(User) // class not instance

user.can(user, 'edit', post)
user.can.some(user, 'edit', posts)
user.can.every(user, 'edit', posts)
```

## Minification

The default subject mapper makes use of "poor-man's reflection", meaning it will
use the name of the input subject's constructor to group the subjects.

When using webpack or similar this method can break if you are not careful. Since
code minifiers will rename functions you have to make sure you only rely on the
function to set up your rules and asking for permission.

### Bad

This will break with minifiers since there is no way to know the subject name
of the subject after minification.

```javascript
acl.rule('edit', 'Post')
```

### Better

This works with minifiers:

```javascript
acl.rule('edit', Post)
```

### Best

This works with minifiers:

```javascript
acl.register(Post, 'Post')
acl.rule('create', 'Post') // <-- works as expected
acl.rule('edit', Post)     // <-- and so does this
```

### Alternative

You can also override the `subjectMapper` function and a property to you objects with
the subject name.

See [subjectMapper](#subjectmapper)

# API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

### Table of Contents

-   [Acl](#acl)
    -   [rule](#rule)
    -   [policy](#policy)
    -   [register](#register)
    -   [can](#can)
    -   [some](#some)
    -   [every](#every)
    -   [mixin](#mixin)
    -   [subjectMapper](#subjectmapper)

### rule

You add rules by providing a verb, a subject and an optional
test (that otherwise defaults to true).

If the test is a function it will be evaluated with the params:
user, subject, and subjectName. The test value is ultimately evaluated
for thruthiness.

Examples:

```javascript
acl.rule('create', Post)
acl.rule('edit', Post, (user, post) => post.userId === user.id)
acl.rule('delete', Post, false) // deleting disabled
```

**Parameters**

-   `verbs` **([Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)> | [string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String))**
-   `subject` **([Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function) \| [Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String))**
-   `test` **[Boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** =true (optional, default `true`)

Returns **[Acl](#acl)**

### policy

You can group related rules into policies for a subject. The policies
properties are verbs and they can plain values or functions.

If the policy is a function it will be new'ed up before use.

```javascript
  class Post {
    constructor() {
      this.view = true       // no need for a functon

      this.delete = false    // not really necessary since an abscent
                             // verb has the same result
    },
    edit(user, subject) {
      return subject.id === user.id
    }
  }
```

Policies are useful for grouping rules and adding more complex logic.

**Parameters**

-   `policy` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)** A policy with properties that are verbs
-   `subject` **([Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function) \| [Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String))**

Returns **[Acl](#acl)**

### register

Explicitly map a class or constructor function to a name.

You would want to do this in case your code is heavily
minified in which case the default mapper cannot use the
simple "reflection" to resolve the subject name.

Note: If you override the subjectMapper this is not used,
bud it can be used manually through `this.registry`.

**Parameters**

-   `klass` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)** A class or constructor function
-   `subjectName` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)**

### can

Performs a test if a user can perform action on subject.

The action is a verb and the subject can be anything the
subjectMapper can map to a subject name.

E.g. if you can to test if a user can delete a post you would
pass the actual post. Where as if you are testing us a user
can create a post you would pass the class function or a
string.

```javascript
  acl->can(user, 'create', Post)
  acl->can(user, 'edit', post)
```

Note that these are also available on the user if you've used
the mixin:

```javascript
  user->can('create', Post)
  user->can('edit', post)
```

**Parameters**

-   `user` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)**
-   `verb` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)**
-   `subject` **([Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function) \| [Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String))**
-   `args` **...any** Any other param is passed into rule

Returns **any** Boolean

### some

Like can but subject is an array where only some has to be
true for the rule to match.

Note the subjects do not need to be of the same kind.

**Parameters**

-   `user` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)**
-   `verb`
-   `subjects` **[Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;([Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function) \| [Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String))>**
-   `args` **...any** Any other param is passed into rule

Returns **any** Boolean

### every

Like can but subject is an array where all has to be
true for the rule to match.

Note the subjects do not need to be of the same kind.

**Parameters**

-   `user` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)**
-   `verb`
-   `subjects` **[Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;([Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function) \| [Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String))>**
-   `args` **...any** Any other param is passed into rule

Returns **any** Boolean

### mixin

Mix in augments your user class with a `can` function object. This
is optional and you can always call `can` directly on your
Acl instance.

    user.can()
    user.can.some()
    user.can.every()

**Parameters**

-   `User` **[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)** A user class or contructor function

### subjectMapper

Rules are grouped by subjects and this default mapper tries to
map any non falsy input to a subject name.

This is important when you want to try a verb against a rule
passing in an instance of a class.

-   strings becomes subjects
-   function's names are used for subject
-   objects's constructor name is used for subject

Override this function if your models do not match this approach.

E.g. say that you are using plain data objects with a type property
to indicate the "class" of the object.

```javascript
  acl.subjectMapper = s => typeof s === 'string' ? s : s.type
```

`can` will now use this function when you pass in your objects.

See [register()](#register) for how to manually map
classes to subject name.

**Parameters**

-   `subject` **([Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function) \| [Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object) \| [string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String))**

Returns **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** A subject
