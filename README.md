# How to Promises

**Promises guide on Nodejs perspective.**  
This repository is designed to help developers understand and apply Promises effectively, helping you to develop your APIs
and avoid common problems. 

## Table of Contents

1. [Introduction](#introduction)
   - [What are Promises?](#what-are-promises)
2. [Promise Constructor](#promise-constructor)
3. [Instance Methods](#instance-methods)
   - [`.then`](#then)
   - [`.catch`](#catch)
   - [`.finally`](#finally)
4. [Async and Await](#async-and-await)
   - [`.Important Hints about async functions`](#important-hints-about-async-functions) 
5. [Static Methods](#static-methods)
   - [`Promise.all`](#promiseall)
   - [`Promise.allSettled`](#promiseallsettled)
   - [`Promise.race`](#promiserace)
   - [`Promise.any`](#promiseany)
6. [For await of](#for-await-of)
7. [CPU-Intensive VS I/O Operations](#CPU-Intensive-vs-I/O-Operations)
8. [REFS](#REFS)

## Introduction

Promises are a foundational feature of modern JavaScript designed to simplify handling asynchronous operations. They represent a placeholder for a value that will be available at some point in the future, either as a result of a successful operation (**fulfilled**) or a failure (**rejected**). Unlike callback-based approaches, Promises provide a structured and predictable way to manage asynchronous flows, avoiding common pitfalls like callback hell.

Promises have three states:
1. **Pending**: The initial state, where the operation has neither succeeded nor failed.
2. **Fulfilled**: The operation completed successfully, and the resulting value is available.
3. **Rejected**: The operation failed, and the reason for the failure is provided.

---

## Promise Constructor

The `Promise` constructor is used to create a new Promise object. The MDN Docs says "it is primarily used to wrap callback-based APIs that do not already support promises". 

So, it takes a single argument, a **function** (commonly referred to as the executor function), which is executed immediately upon the promise’s creation. The executor function receives two arguments:

1. **resolve**: A function used to mark the promise as fulfilled and provide a result.
2. **reject**: A function used to mark the promise as rejected and provide a reason for failure.

It’s common to see people using `new Promise` as a tool to transform anything into a Promise. You shouldn't. I suggest using `new Promise` the way MDN suggests.

#### Example: Transforming `setTimeout` into a Promise

The `setTimeout` function is a classic example of a callback-based asynchronous operation. Using the `Promise` constructor, we can transform it into a Promise for more manageable asynchronous workflows:

```javascript
const delay = (ms) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(`Resolved after ${ms}ms`);
    }, ms);
  });
};

delay(1000)
  .then((result) => console.log(result)) // Logs: Resolved after 1000ms
  .catch((error) => console.error(error)); // Handles any errors
```

We are going to use the upper sintax a lot in this doc.
The fact is that this is an easy way to emulate a promise operation..
So when reading.. New Promise of Set Timeout Fn can be any Promise you want to resolve.
Sometimes we are going to use Promise.resolve too with a similar purpose.

## Instance Methods

Promises provide several methods that allow chaining and handling asynchronous operations. These methods are part of the `Promise` prototype and operate on specific promise instances. Key methods include:

### `.then`

The `.then` method is used to define what happens when a promise is **fulfilled**. It receives a callback function as its argument, which is executed with the value of the resolved promise.

```javascript
const successPromise = Promise.resolve('Success');

successPromise
  .then((value) => console.log(value)) // Logs: Success
  .then(() => console.log('Chaining works!')); // Logs: Chaining works!
```

### `.catch`

The `.catch` method is used to handle errors or rejections. It catches any error that occurs in the promise chain.

```javascript
const errorPromise = Promise.reject('Something went wrong');

errorPromise
  .catch((error) => console.error(error)); // Logs: Something went wrong
```

### `.finally`

The ```.finally``` method executes a callback function once the promise settles, regardless of whether it was fulfilled or rejected. This is often used for cleanup operations.

```javascript
const conditionalPromise = new Promise((resolve, reject) => {
  const condition = true;
  condition ? resolve('Resolved') : reject('Rejected');
});

conditionalPromise
  .then((value) => console.log(value))
  .catch((error) => console.error(error))
  .finally(() => console.log('Promise settled')); // Always logs: Promise settled
```

### The "Tricky Thing" About `.then` and why you should avoid it

The `.then` method schedules its callback to run asynchronously. This can lead to unexpected behavior where the execution of code after `.then` does not wait for the promise to resolve.

```javascript
console.log('Start');

Promise.resolve('Operation with .then')
  .then((value) => {
    console.log(value); // Logs: Operation with .then
  });

console.log('End');
/**Output
Start
End
Operation with .then
*/
```
### Why does this happen ?

The `.then` method schedules its callback to run in the microtask queue, which is processed after the current synchronous code execution completes. This means that console.log('End') runs before the .then callback executes.

### Microtask Queue ?

The microtask queue is a high-priority task queue in the JavaScript event loop. It is where microtasks are stored, which include tasks like:

Resolving or rejecting Promises.
Calling .then, .catch, and .finally handlers.
Operations from APIs such as queueMicrotask.
Microtasks are executed immediately after the currently executing JavaScript code completes and before any tasks in the macrotask queue (e.g., setTimeout or setImmediate) are processed. This behavior ensures that Promise handlers run as soon as possible, but still asynchronously.

#### How to solve this ???

The tricky behavior of `.then` callbacks scheduling their execution in the **microtask queue** can often be solved by using `async/await`. This syntax provides a more synchronous-like flow for handling asynchronous operations, making the code easier to read and debug.

```javascript
async function main() {
  console.log('Start');
  
  const result = await new Promise((resolve) => {
    setTimeout(() => resolve('Operation completed'), 1000);
  });
  
  console.log(result); // Logs: Operation completed
  console.log('End');
}

main();
```

When await is used, it also relies on the microtask queue to resume execution. However, unlike .then, await pauses the execution of the entire function it is in until the Promise resolves. This makes the flow appear more synchronous and easier to follow.

## Async and Await

`async` and `await` are modern JavaScript features introduced in ES2017 (ES8). They provide a cleaner and more intuitive way to work with Promises, allowing asynchronous code to be written in a synchronous-like manner. This is particularly useful in Node.js for handling asynchronous workflows such as file I/O, API requests, and database queries.

#### How `async` and `await` work

1. `async` Functions: Declaring a function as async means it will always return a Promise, even if the function body does not explicitly return one.

2. `await` Keyword: Inside an async function, await pauses the execution of the function until the awaited Promise resolves. This ensures sequential execution of asynchronous code.

#### Main Difs with `.then`

`.then`:
Queues the callback in the microtask queue.
Does not pause surrounding code execution.
await:

`async` and `await`
Also queues the callback in the microtask queue but pauses the execution of the entire async function until the Promise resolves.

### Important Hints about async functions

#### 1. A async function awlays returns a promise. 

So you dont need to use things like `Promise.resolve`to create a promise mock.. You can just use async functions.

#### 2. Stack Trace Behavior: `return vs. return await`

A subtle but important difference exists between return and return await inside async functions. While both ultimately resolve a Promise, returning without await loses the stack trace context. This can make debugging harder when dealing with complex asynchronous workflows.

TLDR: `Use await in every layer you are coding` dont skip await on your functions

ref: https://github.com/goldbergyoni/nodebestpractices/issues/737

#### 3. Await Sequences

When using multiple `await` statements in an `async` function, they are executed **sequentially** by default. This means each `await` waits for the previous Promise to resolve before the next one starts.

#### Example: Serial Execution

```javascript
async function fetchDataSequentially() {
  console.log('Start fetching data');

  const data1 = await new Promise((resolve) =>
    setTimeout(() => resolve('Data 1'), 1000)
  );
  console.log(data1); // Logs: Data 1

  const data2 = await new Promise((resolve) =>
    setTimeout(() => resolve('Data 2'), 1000)
  );
  console.log(data2); // Logs: Data 2

  console.log('Finished fetching data');
}

fetchDataSequentially();

/**Output
Start fetching data
Data 1
Data 2
Finished fetching data - this will take 2 seconds
*/
```
#### Why Does This Happen?
Each await statement pauses the execution of the async function until the Promise resolves. This creates a serial execution where each Promise is resolved one after the other, even if they are independent.

#### 4. Await Sequences using for loop

When using `for...of` loops with `await`, each iteration is executed **serially**. This means the loop waits for the current iteration's Promise to resolve before moving to the next iteration.

```javascript
async function fetchSequentiallyWithForLoop() {
  console.log('Start fetching');

  const tasks = ['Task 1', 'Task 2', 'Task 3'];

  for (const task of tasks) {
    const result = await new Promise((resolve) =>
      setTimeout(() => resolve(task), 1000)
    );
    console.log(result); // Logs each task in order, with a delay of 1 second per task - 3
  }

  console.log('Finished fetching');
}

fetchSequentiallyWithForLoop();
/** Output:
Start fetching
Task 1
Task 2
Task 3
Finished fetching
*/
```

#### 5. Fail Fast vs Fail Safe in Await Sequences
The behavior of the for...of loop with await can change depending on how you handle errors. Below are the two common approaches:

#### Fail Fast
In the fail fast approach, if any Promise in the sequence rejects, the loop stops immediately, and the error is propagated. This is the default behavior of for...of with await.
 
```javascript
async function failFast() {
  console.log('Start fetching');

  const tasks = ['Task 1', 'Task 2', 'Task 3'];

  try {
    for (const task of tasks) {
      // Simulate async task creation with potential failure
      const result = await new Promise((resolve, reject) => {
        if (task === 'Task 2') {
          reject('Error in Task 2'); // Simulate an error for Task 2
        }
        setTimeout(() => resolve(task), 1000); // Simulate async task
      });

      // Log success
      console.log(result); // Logs Task 1, then stops at Task 2
    }
  } catch (error) {
    console.error('Error caught:', error); // Logs: Error caught: Error in Task 2
    return; // Early exit from the loop on error
  }

  console.log('Finished fetching'); // This only logs if there are no errors
}

failFast();
/** Output:
Start fetching
Task 1
Error caught: Error in Task 2
*/

```

#### Explanation:

The loop stops as soon as the Promise for Task 2 rejects.
The error is caught in the try...catch block, but the subsequent tasks (Task 3) are not executed.

#### Fail Safe

In the fail safe approach, you ensure that the loop continues execution even if one of the Promises rejects. This is done by wrapping each await in a try...catch block.

```javascript
 async function failSafe() {
  console.log('Start fetching');

  const tasks = ['Task 1', 'Task 2', 'Task 3'];

  for (const task of tasks) {
    try {
      const result = await new Promise((resolve, reject) => {
        if (task === 'Task 2') {
          reject('Error in Task 2'); // Simula erro na Task 2
        }
        setTimeout(() => resolve(task), 1000); // Simula uma tarefa assíncrona
      });

      console.log(result); // Logs o resultado da tarefa
    } catch (error) {
      console.error('Error handled locally:', error); // Logs: Error handled localmente
    }
  }

  console.log('Finished fetching');
}

failSafe();
/** Output:
Start fetching
Task 1
Error handled locally: Error in Task 2
Task 3
Finished fetching
*/
```

#### Explanation:

Each await is individually wrapped in a try...catch block, so errors in one Promise do not affect the others.
The loop continues execution for all tasks, logging errors only for the ones that fail.

#### 6. Initializing Promises Before Resolving Them

When you initialize multiple Promises upfront and then `await` them later, they are resolved simultaneous, even if you use `await` sequentially. This is because the Promises start executing as soon as they are created, regardless of when `await` is used.


```javascript
function tenSecondsPromise() {
  return new Promise((resolve) =>
    setTimeout(() => resolve('10 seconds'), 10000)
  );
}

function twoSecondsPromise() {
  return new Promise((resolve) =>
    setTimeout(() => resolve('2 seconds'), 2000)
  );
}

function oneSecondPromise() {
  return new Promise((resolve) =>
    setTimeout(() => resolve('1 second'), 1000)
  );
}

async function executePromisesSimultaneously() {
  console.time('executePromisesSimultaneously');

  // Start all Promises simultaneously
  const promise1 = tenSecondsPromise();
  const promise2 = twoSecondsPromise();
  const promise3 = oneSecondPromise();

  // Await them sequentially
  console.log(await promise1); // Logs: 10 seconds
  console.log(await promise2); // Logs: 2 seconds
  console.log(await promise3); // Logs: 1 second

  console.timeEnd('executePromisesSimultaneously'); // Logs: ~10 seconds
}

executePromisesSimultaneously();
```

In this example the log will come on the correct order bc of the await nature.
But time time will be 10 secs, since you initialized the promises before await for them. 

#### 7. Dont solve your promises inside an Array Method

When working with Promises in combination with array methods like .map, .forEach, or .filter, it's common to run into issues if the Promise resolution is not properly managed. These methods do not inherently handle asynchronous operations, which can lead to unexpected behavior.

Array methods are not async or promise aware

refs(if you google it you will find tons, i just did it):
1. https://stackoverflow.com/questions/64978604/async-await-map-not-awaiting-async-function-to-complete-inside-map-function-befo

2. https://medium.com/dailyjs/async-loops-and-why-they-fail-part-2-f66e5ea04113

##### Example of async aware .map

```javascript
async function asyncMap(array, callback) {
  const results = [];
  for (let i = 0; i < array.length; i++) {
    const result = await callback(array[i], i, array); // Sequentially await each callback
    results.push(result);
  }
  return results;
}
```

## Static Methods

### 1. `Promise.all`

- **Definition**: `Promise.all` takes an iterable of Promises and returns a single Promise that resolves when **all of the input Promises are fulfilled**. If any Promise rejects, the returned Promise immediately rejects with the reason of the first rejection.
- **Use Case**: Use when you need **all tasks to succeed** to proceed.
- Promise all works with the concept of Fail Fast

```javascript
const promises = [
  Promise.resolve('Result 1'),
  Promise.resolve('Result 2'),
  Promise.reject('Error 1'),
];

Promise.all(promises)
  .then((results) => console.log('All Results:', results))
  .catch((error) => console.error('Caught Error:', error)); // Logs: Caught Error: Error 1
```


### 2. `Promise.allSettled`

- **Definition**: Promise.allSettled takes an iterable of Promises and returns a Promise that resolves once all input Promises settle (fulfilled or rejected). It provides the status and value/reason for each Promise.
- **Use Case**: when you need to process results regardless of success or failure.
- Promise allSettled works with the concept of Fail Safe, and errors as Values

```javascript
const promises = [
  Promise.resolve('Result 1'),
  Promise.reject('Error 1'),
  Promise.resolve('Result 2'),
];

Promise.allSettled(promises).then((results) => {
  console.log('All Settled:', results);
  /** Output:
  [
    { status: 'fulfilled', value: 'Result 1' },
    { status: 'rejected', reason: 'Error 1' },
    { status: 'fulfilled', value: 'Result 2' }
  ]
  */
});
```

### 3. `Promise.race`
- **Definition**: `Promise.race` takes an iterable of Promises and returns a single Promise that resolves or rejects as soon as the first input Promise settles (either fulfills or rejects).
- **Use Case**: Use when you need the result of the fastest task.

```javascript
const promises = [
  new Promise((resolve) => setTimeout(() => resolve('Fast Success'), 500)),
  new Promise((resolve) => setTimeout(() => resolve('Slow Success'), 2000)),
  new Promise((_, reject) => setTimeout(() => reject('Fast Error'), 1000)),
];

Promise.race(promises)
  .then((result) => console.log('First Resolved:', result)) // Logs: First Resolved: Fast Success
  .catch((error) => console.error('First Rejected:', error));
```

### 4. `Promise.any`
- **Definition**: `Promise.any` takes an iterable of Promises and returns a Promise that resolves as soon as any of the input Promises is fulfilled. If all Promises reject, it rejects with an AggregateError..
- **Use Case**: Use when you need the result of the fastest task.

```javascript
const promises = [
  Promise.reject('Error 1'),
  Promise.reject('Error 2'),
  Promise.resolve('First Success'),
];

Promise.any(promises)
  .then((result) => console.log('First Success:', result)) // Logs: First Success: First Success
  .catch((error) => console.error('All Rejected:', error)); // Logs: AggregateError: All Promises were rejected
```

### 5. `Promise.resolve`
- **Definition**: `Promise.resolve` is a static method that returns a Promise object that is resolved with the given value. If the value is already a Promise, it simply returns that Promise unchanged.
- **Use Case**: `Promise.resolve` when you need to wrap a synchronous value or non-Promise object into a resolved Promise.

```javascript
const existingPromise = Promise.resolve('Already resolved');

const promise = Promise.resolve(existingPromise);

promise.then((result) => {
  console.log(result); // Logs: Already resolved
});
```

Personally i just use if for mocks

### 6. `Promise.reject`
- **Definition**: `Promise.reject` is a static method that returns a Promise object that is rejected with the specified reason..
- **Use Case**: `Promise.reject` to create a Promise that is immediately rejected, often for testing or error-handling scenarios.

```javascript
const errorPromise = Promise.reject('Something went wrong');

errorPromise.catch((error) => {
  console.error(error); // Logs: Something went wrong
});
```

Personally i just use if for mocks

## For await of
for await...of shines when working with asynchronous data streams, such as reading data chunks from a file or API.
Lots of people use this in the for of await use case...
Actually, it wont have value. You should use for await of in async data streams.

```javascript
const { Readable } = require('stream');

// Create a readable stream with async iteration
const stream = Readable.from(['Chunk 1', 'Chunk 2', 'Chunk 3'], { objectMode: true });

async function processStream() {
  for await (const chunk of stream) {
    console.log(`Processing: ${chunk}`);
  }
  console.log('Stream processing complete');
}

processStream();
/** Output:
Processing: Chunk 1
Processing: Chunk 2
Processing: Chunk 3
Stream processing complete
*/
```

### `for of { await }` vs `for await of`

1. `for...of { await }`
In this approach, you iterate over an array of Promises using a for...of loop and explicitly use await within the loop body. Each await pauses the loop execution until the current Promise resolves.

2. `for await of`
This approach is specifically designed to work with asynchronous iterables, where each iteration automatically awaits the Promise. It simplifies handling asynchronous streams or dynamic data sources

## 7. CPU-Intensive VS I/O Operations 
I made this topic bc i see it everywhere, i see it in lots of codebases and this scales messy.

A common mistake developers make is mixing CPU-bound tasks (like data processing, sorting, encryption, or compression) with I/O-bound operations (like reading/writing files, making API calls, or database queries) in the same thread of execution. This approach not only hinders the scalability of your application but also creates inefficiencies that can lead to performance bottlenecks.

## Why Should You Avoid Mixing?
Node.js operates on a single-threaded event loop designed for non-blocking I/O. When you introduce CPU-bound tasks into the same flow as I/O operations, you block the event loop, which delays the processing of other requests.


#### what to avoid
```javascript
const fs = require('fs').promises;

async function processFile(filePath) {
  const fileData = await fs.readFile(filePath); // I/O operation
  console.log('File read complete');

  const processedData = fileData
    .toString()
    .split('\n')
    .map((line) => line.toUpperCase()) // CPU-intensive processing
    .join('\n');

  await fs.writeFile('processed-file.txt', processedData); // Another I/O operation
  console.log('File write complete');
}

processFile('large-file.txt');
```

#### what to do
```javascript
async function processFile(filePath) {
  const fileData = await fs.readFile(filePath); // I/O operation
  console.log('File read complete');
  processFileData(fileData); // Hand off CPU-intensive task
}

function processFileData(data) {
  const processedData = data
    .toString()
    .split('\n')
    .map((line) => line.toUpperCase())
    .join('\n');
  console.log('Processing complete');
  fs.writeFile('processed-file.txt', processedData)
    .then(() => console.log('File write complete'))
    .catch(console.error);
}

processFile('large-file.txt');
```

#### When should i mix CPU and I/O Operations ?
When you are scripting and you rly dont need to care about performance/scaling. 


## REFS
1. MDN Docs (https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Promise)
2. Nodejs Docs (https://nodejs.org/docs/latest/api/)
3. A "Lab" Repo a did last Year (https://github.com/samsantosb/Javascript-Promises)
4. And github/stackoverflow discussions i mentioned