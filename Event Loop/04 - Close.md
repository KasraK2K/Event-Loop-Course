![[EventLoop.gif]]


## Close Queue
Close queue callbacks are executed after all other requests callbacks in a given iteration of the event loop

```js
const fs = require("fs")

const readableStream = fs.createReadStream(__filename)
readableStream.close()

readableStream.on("close", () => {
	console.log("This is from readableStream close event callback")
})
setImmediate(() => console.log("This is setImmediate 1"))
setTimeout(() => console.log("This is setTimeout 1"), 0)
process.nextTick(() => console.log("This is process.nextTick 1"))
Promise.resolve().then(() => console.log("This is Promise.resolve 1"))

// This is process.nextTick 1
// This is Promise.resolve 1
// This is setTimeout 1
// This is setImmediate 1
// This is from readableStream close event callback
```