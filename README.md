Typically, operations are executed synchronously, meaning that they are executed one after another.
Each operation only starts once the one immediately before it is done.

```
        {Start}
           |
           |
     +---------------+
     |  operation 1  |
     +---------------+
           |
     +---------------+
     |  operation 2  |
     +---------------+
           |
     +---------------+
     |  operation 3  |
     +---------------+
```

In the context of javascript code, these operations directly correspond to javascript statements.
Usually, executing these operations sequentially is not an issue, or even necessary, because they depend on each other, and they take just microseconds to complete.
However, when an operation takes a long time (seconds, minutes or even longer), it blocks all operations that follow from starting.

```
        {Start}
           |
           |
     +---------------+
     |  operation 1  |
     +---------------+
           |
     +-----------------+
     |  operation 2    |
     | takes 3 minutes |
     +-----------------+
           |
     +----------------------------------+
     |  operation 3                     |
     | won't start until op. 2 is done  |
     +----------------------------------+
```
     
This is when it becomes useful to execute those operations asynchronously, meaning they no longer wait for each other to finish

```
        {Start}
           |
           |
     +---------------+
     |  operation 1  |
     +---------------+
           |
     +-----------------------------------+
     |  spawn other chain of operations  |  ----------->    {Start}
     +-----------------------------------+                     |
           |                                                   |
     +---------------+                                  +----------------------+
     |  operation 3  |                                  | operation 11         |
     +---------------+                                  | may take a long time |
           |                                            +----------------------+
     +---------------+
     |  operation 4  |
     +---------------+
```
   
In this example, operation 3 would immediately be executed once the second chain was created, regardless of how long the second chain takes.

Promises are wrappers for asynchronous operations.
You may think of them as the asynchronous counterpart of a function

```javascript
function myFunction() {
    return 'hello from inside a function';
}

console.log(myFunction());

const myPromise = new Promise((resolve) => {
    resolve('hello from inside a promise');
});

myPromise.then((value) => {
    console.log(value);
});
```  

The Promise constructor takes an executor function which contains the actual promise logic.

```javascript
new Promise((resolve, reject) => { ... });
//          ^--------------------------^
//           executor function
```

This executor function automatically receives two functions as arguments, which are usually called `resolve` and `reject`.

The purpose of `resolve` can be compared to that of the `return` keyword inside a regular function.

The `reject` function is necessary for signalling that a promise has failed, because unlike with synchronous operations,
where the failure of one operation would immediately cause the chain of operations to stop, asynchronous operations don't influence
their neighbouring operations at all.
You may omit the `reject` function from the arguments entirely if your promise cannot fail at all.

However, the `resolve` function cannot be omitted, because a promise that doesen't resolve never finishes:

```javascript
new Promise(() => {
    // never calls resolve
}).then(() => {
    console.log('This will never be printed');
});
```

The main advantage of having a resolve function over the return keyword is, that you can resolve a promise from inside a callback function
or an event listener. Something that wouldn't be possible with the return keyword:

```javascript
// Bad: If Promises finished with 'return':
new Promise(() => {
    setTimeout(() => {
        return 'timeout over'; // returns from the timeout callback, but not from the Promise executor
    }, 1000);
});

// Good: With resolve:
new Promise((resolve) => {
    setTimeout(() => {
        resolve('timeout over'); // Resolves the promise
    }, 1000);
});
```

You can use the `.then()` function to associate further action to a promise

```javascript
new Promise((resolve) => {
    setTimeout(() => {
        resolve(Math.random())
    }, 3000);
}).then((randomValue) => {
    console.log(`after waiting three seconds, I received the random value ${randomValue}`);
});
```

`then` returns a promise, which means, you can chain multiple `then` calls

```javascript
new Promise((resolve) => {
    resolve(5);
}).then((value) => {
    return value * 2;
}).then((value) => {
    console.log(value);
});
```

You can listen to errors generated by `reject` with the `catch` function

```javascript
somePromiseThatMayFail
    .then((returnValue) => {
        console.log(`the promise returned ${returnValue}`);
    })
    .catch((error) => {
        console.log(`an error happened: ${error}`);
    });
```

If you want to have code executed regardless of whether there was an error or not, use `finally`

```javascript
somePromiseThatMayFail
    .then((returnValue) => {
        console.log(`the promise returned ${returnValue}`);
    })
    .catch((error) => {
        console.log(`an error happened: ${error}`);
    })
    .finally(() => {
        // For example clean-up operations
    });
```