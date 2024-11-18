![[EventLoop.gif]]


## Time Queue
1. Callbacks in the microtask queues are executed before callbacks in the timer queue
2. Timer queue callbacks are executed in FIFO order

```js
setTimeout(() => console.log("This is setTimeout 1"), 0)
setTimeout(() => {
	console.log("This is setTimeout 2")
	process.nextTick(() => console.log("This is the inner process.nextTick inside setTimeout"))
}, 0)
setTimeout(() => console.log("This is setTimeout 3"), 0)

// This is setTimeout 1
// This is setTimeout 2
// This is the inner process.nextTick inside setTimeout
// This is setTimeout 3
```
