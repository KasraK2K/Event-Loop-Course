![[EventLoop.gif]]


## Synchronous
All user written synchronous JavaScript code takes priority over async code that the runtime would like to eventually execute

```js
console.log("console log 1")
process.nextTick(() => console.log("This is process.nextTick 1"))
console.log("console log 2")

// console log 1
// console log 2
// This is process.nextTick 1
```


## Microtask Queue
The microtask queues are `process.nextTick` and `Promise`.
All callbacks in nextTick queue are executed before callbacks in promise queue

```js
Promise.resolve().then(() => console.log("This is Promise.resolve 1"))
process.nextTick(() => console.log("This is process.nextTick 1"))

// This is process.nextTick 1
// This is Promise.resolve 1
```


### Two main reasons to use `nextTick`
1. To allow users to handle errors, cleanup any then unneeded resource, or perhaps try the request again before the event loop continues
2. To allow a callback to run after the call stack has unwound but before the event loop continues
Use of process.nextTick is discouraged as it cause the rest of the event loop to starve
If you endlessly call process.nextTick. the control will never make it pas the microtask queue

```js
process.nextTick(() => console.log("This is process.nextTick 1"))
process.nextTick(() => {
	console.log("This is process.nextTick 2")
	process.nextTick(() => console.log("This is the inner process.nextTick inside process.nextTick"))
})
process.nextTick(() => console.log("This is process.nextTick 3"))

Promise.resolve().then(() => console.log("This is Promise.resolve 1"))
Promise.resolve().then(() => {
	console.log("This is Promise.resolve 2")

	process.nextTick(() => console.log("This is the inner process.nextTick inside Promise.resolve then block"))
})
Promise.resolve().then(() => console.log("This is Promise.resolve 3"))

// This is process.nextTick 1
// This is process.nextTick 2
// This is process.nextTick 3
// This is the inner process.nextTick inside process.nextTick
// This is Promise.resolve 1
// This is Promise.resolve 2
// This is Promise.resolve 3
// This is the inner process.nextTick inside Promise.resolve then block
```
