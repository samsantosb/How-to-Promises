# How to Promises

**Promises guide on Nodejs perspective.**  
This repository is designed to help developers understand and apply Promises effectively, with a focus on building robust backend applications.

## Table of Contents

1. [Introduction](#introduction)
   - [What are Promises?](#what-are-promises)
2. [Promise Constructor](#promise-constructor)
3. [Instance Methods](#instance-methods)
   - [`.then`](#then)
   - [`.catch`](#catch)
   - [`.finally`](#finally)
4. [Async and Await](#async-and-await)
5. [Static Methods](#static-methods)
   - [`Promise.all`](#promiseall)
   - [`Promise.allSettled`](#promiseallsettled)
   - [`Promise.race`](#promiserace)
   - [`Promise.any`](#promiseany)

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

### Example: Transforming `setTimeout` into a Promise

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

## Instance Methods

Promises provide several methods that allow chaining and handling asynchronous operations. These methods are part of the `Promise` prototype and operate on specific promise instances. Key methods include:

### `.then`

The `.then` method is used to define what happens when a promise is **fulfilled**. It receives a callback function as its argument, which is executed with the value of the resolved promise.

#### Example

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

#### Example

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

### WTF is the Microtask Queue ?

The microtask queue is a high-priority task queue in the JavaScript event loop. It is where microtasks are stored, which include tasks like:

Resolving or rejecting Promises.
Calling .then, .catch, and .finally handlers.
Operations from APIs such as queueMicrotask.
Microtasks are executed immediately after the currently executing JavaScript code completes and before any tasks in the macrotask queue (e.g., setTimeout or setImmediate) are processed. This behavior ensures that Promise handlers run as soon as possible, but still asynchronously.

### How to solve this ???

The tricky behavior of `.then` callbacks scheduling their execution in the **microtask queue** can often be solved by using `async/await`. This syntax provides a more synchronous-like flow for handling asynchronous operations, making the code easier to read and debug.

#### Example: `async/await` in Action

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

### Async and Await

`async` and `await` are modern JavaScript features introduced in ES2017 (ES8). They provide a cleaner and more intuitive way to work with Promises, allowing asynchronous code to be written in a synchronous-like manner. This is particularly useful in Node.js for handling asynchronous workflows such as file I/O, API requests, and database queries.

### How `async` and `await` work

1. `async` Functions: Declaring a function as async means it will always return a Promise, even if the function body does not explicitly return one.

2. `await` Keyword: Inside an async function, await pauses the execution of the function until the awaited Promise resolves. This ensures sequential execution of asynchronous code.

### Main Difs with `.then`

`.then`:
Queues the callback in the microtask queue.
Does not pause surrounding code execution.
await:

`async` and `await`
Also queues the callback in the microtask queue but pauses the execution of the entire async function until the Promise resolves.

### Important Hints about async functions

### 1. A async function awlays returns a promise. 

So you dont need to use things like `Promise.resolve`to create a promise mock.. You can just use async functions.

### 2. Stack Trace Behavior: `return vs. return await`

A subtle but important difference exists between return and return await inside async functions. While both ultimately resolve a Promise, returning without await loses the stack trace context. This can make debugging harder when dealing with complex asynchronous workflows.

TLDR: `Use await in every layer you are coding` dont skip await on your functions

ref: https://github.com/goldbergyoni/nodebestpractices/issues/737

### 3. Await Sequences

When using multiple `await` statements in an `async` function, they are executed **sequentially** by default. This means each `await` waits for the previous Promise to resolve before the next one starts.

### Example: Serial Execution

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
Finished fetching data
*/
```
### Why Does This Happen?
Each await statement pauses the execution of the async function until the Promise resolves. This creates a serial execution where each Promise is resolved one after the other, even if they are independent.

### 4. Await Sequences using for loop

When using `for...of` loops with `await`, each iteration is executed **serially**. This means the loop waits for the current iteration's Promise to resolve before moving to the next iteration.

#### Example: Serial Execution in a `for...of` Loop

```javascript
async function fetchWithForLoop() {
  console.log('Start fetching');

  const tasks = [
    new Promise((resolve) => setTimeout(() => resolve('Task 1'), 1000)),
    new Promise((resolve) => setTimeout(() => resolve('Task 2'), 1000)),
    new Promise((resolve) => setTimeout(() => resolve('Task 3'), 1000)),
  ];

  for (const task of tasks) {
    const result = await task;
    console.log(result); // Logs: Task 1, Task 2, Task 3 (one per second)
  }

  console.log('Finished fetching');
}

fetchWithForLoop();
/** Output:
Start fetching
Task 1
Task 2
Task 3
Finished fetching
*/
```

### 5. Fail Fast vs Fail Safe in Await Sequences
The behavior of the for...of loop with await can change depending on how you handle errors. Below are the two common approaches:


### Fail Fast
In the fail fast approach, if any Promise in the sequence rejects, the loop stops immediately, and the error is propagated. This is the default behavior of for...of with await.
 

```javascript
async function failFast() {
  console.log('Start fetching');

  const tasks = [
    Promise.resolve('Task 1'),
    Promise.reject('Error in Task 2'), // This will cause the loop to stop
    Promise.resolve('Task 3'),
  ];

  try {
    for (const task of tasks) {
      const result = await task;
      console.log(result);
    }
  } catch (error) {
    console.error('Error caught:', error); // Logs: Error caught: Error in Task 2
  }

  console.log('Finished fetching');
}

failFast();
/** Output:
Start fetching
Task 1
Error caught: Error in Task 2
Finished fetching
*/
```

## Explanation:

The loop stops as soon as the Promise for Task 2 rejects.
The error is caught in the try...catch block, but the subsequent tasks (Task 3) are not executed.

## Fail Safe

In the fail safe approach, you ensure that the loop continues execution even if one of the Promises rejects. This is done by wrapping each await in a try...catch block.

```javascript
 async function failSafe() {
  console.log('Start fetching');

  const tasks = [
    Promise.resolve('Task 1'),
    Promise.reject('Error in Task 2'), // This error is handled locally
    Promise.resolve('Task 3'),
  ];

  for (const task of tasks) {
    try {
      const result = await task;
      console.log(result);
    } catch (error) {
      console.error('Error handled locally:', error); // Logs: Error handled locally: Error in Task 2
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

## Explanation:

Each await is individually wrapped in a try...catch block, so errors in one Promise do not affect the others.
The loop continues execution for all tasks, logging errors only for the ones that fail.

### 6. Initializing Promises Before Resolving Them

When you initialize multiple Promises upfront and then `await` them later, they are resolved simultaneous, even if you use `await` sequentially. This is because the Promises start executing as soon as they are created, regardless of when `await` is used.


```javascript
async function fetchInParallel() {
  console.log('Start fetching');

  // Initialize all Promises upfront
  const task1 = new Promise((resolve) => setTimeout(() => resolve('Task 1'), 3000)); // Longer duration
  const task2 = new Promise((resolve) => setTimeout(() => resolve('Task 2'), 1000));
  const task3 = new Promise((resolve) => setTimeout(() => resolve('Task 3'), 500));  // Shorter duration

  // Await them sequentially
  console.log(await task1); // Logs: Task 1
  console.log(await task2); // Logs: Task 2
  console.log(await task3); // Logs: Task 3

  console.log('Finished fetching');
}

fetchInParallel();
/** Output:
Start fetching
Task 3
Task 2
Task 1
Finished fetching
*/
```
