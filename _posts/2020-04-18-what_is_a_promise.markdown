---
layout: post
title:      "What is a Promise?"
date:       2020-04-18 23:15:32 +0000
permalink:  what_is_a_promise
---

This post is to give you a very *basic* understanding of how promises work. For more details refer to [official documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

First, a promise is an object. It is used to store a potential value, a value that will be available in the future. When the promise "resolves," handlers like `.then()` and `.catch()` are used to manipulate the success value or error.

A promise exists in three different states:
1. Initial state is "pending"
2. Can be resolved to "fulfilled"
3. Can be resolved to "rejected"

### Constructor

```
var promiseObj = new Promise (callbackFunction(resolve, error)
```

The constructor of the Promise class accepts a callback function, which accepts two functions as parameters, `resolve()` and `error()`.

In the callback function, we specify what the Promise should potentially evalulate to. This value can be the response of an HTTP request like in fetch or other asynchronous code. *If the request or asynchronous code is successful, we want to invoke the `resolve` function with the success value and if not successful we invoke the `error` function with the error. **The success value or error value is respectively passed to a chained `.then()` or `.catch() `handler.***

Take a look at this example. This example is only including the `resolve` arg for simplicity. I am also using a simple value of 2 and `setTimeout()` to represent our "potential future" value. 

```
const wait = () => {
    return new Promise(resolve => {
        const y = 2;
        setTimeout(() => resolve(y), 3000) 
        }
    )
}

```

We have a function `wait()` that returns a new Promise object. 
The callback function sets y = 2, and after 3 seconds, `setTimeout()` will invoke `resolve()` with value `y`. 

```
wait()
// => Promise (pending)
// after 3 seconds
// => Promise (resolved)
// => undefined
```

When we call `wait()`, first it will return a Promise with status "pending". Then, after 3 seconds the status will change to "resolved." Don't worry about the return value `undefined` of the Promise object.

### Using `.then()` to Handle Resolved Value
**Now we can chain the `.then()` handler to manipulate the resolved value.**

```
wait().then(result => console.log(result + 5))
```

`.then()` accepts a function which is passed the resolved value, `result`.
And here we are simply logging `result + 5`.

```
wait().then(result => console.log(result + 5))

// => Promise (pending)
// after 3 seconds
// => Promise (resolved)
// => undefined
// => 7
```

So first when we call wait(), a new Promise object is returned with status "pending".
It runs the callback function, and continues to stay "pending", until it invokes `resolve().`
Finally, when `resolve()` is invoked, in this case after 3 seconds, any handlers like `.then()` are used to manipulate that value. In this case we are logging `result + 5.`

### Illustrating Asynchronousity
*In which order does all this get executed, when there are other lines of code??*
Any handlers are invoked, after the promise is resolved. In other words, **handlers are executed asynchronously.**

```
//using same wait() function
console.log('a');
wait().then(result => console.log(result + 5))
console.log('b');
```

1. Console logs `a`.
2. `wait()` returns a Promise object. The callback runs into asynchronous code, `setTimeout().` While waiting for 3000 ms delay, JS goes on to execute other lines of code.
3. Console then logs `b`.
4. After delay has passed, `.then()` will log `result + 5`.

```
// => "a"
// => Promise (pending)
// => "b"
// after 3 seconds
// => Promise (resolved)
// => undefined
// => 7
```

Promises make it convenient and simple for handling a value that is going to be resolved asynchronously. `.then()` and `.catch()` allow you to write asynchronous code in a simple and intuitive way.

Thanks for reading!

Sources:

- [https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261)
- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)








