![[EventLoop.gif]]


## I/O Queue
Callbacks in the microtask queues are executed before callbacks in the I/O queue

```js
fs.readFile(__filename, () => {
	console.log("This is readFile 1")
})

process.nextTick(() => console.log("This is process.nextTick 1"))
Promise.resolve().then(() => console.log("This is Promise.resolve 1"))

// This is process.nextTick 1
// This is Promise.resolve 1
// This is readFile 1
```


When running setTimeout with delay 0ms and an I/O async method, the order of execution can never be guaranteed. This is because of Node.js setTimeout written with C++ and has formula to change zero millisecond to one millisecond like this:

```c++
double intervalMillisecond = std::max(oneMillisecond, interval * oneMillisecond)
```

So we can't guarantee to which one of codes below runs first

```js
setTimeout(() => console.log("This is setTimeout 1"), 0)

fs.readFile(__filename, () => {
	console.log("This is readFile 1")
})
```


I/O queue callbacks are executed after Microtask queue callbacks and Timer queue callbacks
I/O queue are pulled, and callback function are added to the I/O queue only after the I/O is complete

```js
fs.readFile(__filename, () => {
	console.log("This is readFile 1")
})

process.nextTick(() => console.log("This is process.nextTick 1"))
Promise.resolve().then(() => console.log("This is Promise.resolve 1"))
setTimeout(() => console.log("This is setTimeout 1"), 0)
setImmediate(() => console.log("This is setImmediate 1"), 0)

for (let i = 0; i < 2000000000; i++) {}

// This is process.nextTick 1
// This is Promise.resolve 1
// This is setTimeout 1
// This is setImmediate 1
// This is readFile 1
```


It's good to know if setImmediate run just after I/O queue callback exist (For example when we use setImmediate inside I/O queue callback), The I/O queue runs before setImmediate
Check queue callbacks are executed after Microtask queue callbacks, Timer queue callbacks and I/O queue callbacks are executed

```js
fs.readFile(__filename, () => {
	console.log("This is readFile 1")
	setImmediate(() => console.log("This is inner setImmediate inside readFile 1"))
})

process.nextTick(() => console.log("This is process.nextTick 1"))
Promise.resolve().then(() => console.log("This is Promise.resolve 1"))
setTimeout(() => console.log("This is setTimeout 1"), 0)

for (let i = 0; i < 2000000000; i++) {}

// This is process.nextTick 1
// This is Promise.resolve 1
// This is setTimeout 1
// This is readFile 1
// This is inner setImmediate inside readFile 1
```


Microtask queues callbacks are executed after I/O callbacks and before check queue callbacks
```js
fs.readFile(__filename, () => {
	console.log("This is readFile 1")
	setImmediate(() => console.log("This is inner setImmediate inside readFile 1"))
	process.nextTick(() => console.log("This is inner process.nextTick inside readFile 1"))
	Promise.resolve().then(() => console.log("This is inner Promise.resolve inside readFile 1"))
})

process.nextTick(() => console.log("This is process.nextTick 1"))
Promise.resolve().then(() => console.log("This is Promise.resolve 1"))
setTimeout(() => console.log("This is setTimeout 1"), 0)

for (let i = 0; i < 2000000000; i++) {}

// This is process.nextTick 1
// This is Promise.resolve 1
// This is setTimeout 1
// This is readFile 1
// This is inner process.nextTick inside readFile 1
// This is inner Promise.resolve inside readFile 1
// This is inner setImmediate inside readFile 1
```

```js
setImmediate(() => console.log("This is setImmediate 1"))

setImmediate(() => {
	console.log("This is setImmediate 2")
	process.nextTick(() => console.log("This is process.nextTick 1"))
	Promise.resolve().then(() => console.log("This is Promise.resolve 1"))
})

setImmediate(() => console.log("This is setImmediate 3"))

// This is setImmediate 1
// This is setImmediate 2
// This is process.nextTick 1
// This is Promise.resolve 1
// This is setImmediate 3
```


When running setTimeout with delay 0ms and setImmediate method, the order of execution can never be guaranteed
```js
setTimeout(() => console.log("This is setTimeout 1"), 0)
setImmediate(() => console.log("This is setImmediate 1"))
```

For guaranty, we can add time-consuming for loop

```js
setTimeout(() => console.log("This is setTimeout 1"), 0)
setImmediate(() => console.log("This is setImmediate 1"))
for (let i = 0; i < 2000000000; i++) {}

// This is setTimeout 1
// This is setImmediate 1
```