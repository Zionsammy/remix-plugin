# Remix Plugin

> For the old api, go to [api](./api.md)

ALPHA : This version is still a work in progress and some breaks may be expected (especially names). But the overall stucture should remain unchanged.

## Getting Started

Installation : 
```bash
npm install remix-plugin
```

or with a unpkg : 
```html
<script src="https://unpkg.com/remix-plugin"></script>
```

## Remix Extension
`RemixExtension` helps you communicate with the IDE.

To import it you can use ES6 if installed with npm : 
```javascript
import { RemixExtension } from 'remix-plugin'
const extension = new RemixExtension()
```

Or with a global variable if you used unpkg : 
```javascript
const { RemixExtension } = remixPlugin
const extension = new RemixExtension()
```

---

### Loaded
`RemixExtension` listen on a first handshake from the IDE before beeing able to communicate back. For that you need to wait for the Promise `loaded` to be called.

```javascript
extension.loaded().then(_ => /* Do Something now */)
```

Or with the `async` / `await` syntax : 
```javascript
await extension.loaded()
// Do Something now
```

### Listen
To listen to an event you need to provide the name of the plugin your listening on, and the name of the event : 
```javascript
extension.listen(/* pluginName */, /* eventName */, ...arguments)
```

For exemple if you want to listen to Solidity compilation : 
```javascript
extension.listen('solidity', 'compilationFinished', (target, source, version, data) => {
    /* Do Something on Compilation */
  }
)
```

> See all available event [below](#api).

### Call 
You can call some methods exposed by the IDE with with the method `call`. You need to provide the name of the plugin, the name of the method, and the arguments of the methods : 
```javascript
await extension.call(/* pluginName */, /* methodName */, ...arguments)
```
> Note: `call` is alway Promise

For example if you want to upsert the current file : 
```typescript
async function upsertCurrentFile(content: string) {
  const path = await extension.call('fileManager', 'getCurrentFile')
  await extension.call('fileManager', 'setFile', path, content)
}
```

> Note: Be sure that your plugin is loaded before making any call.

### Emit
Your plugin can emit events that other plugins can listen on.
```javascript
extension.emit(/* eventName */, ...arguments)
```

Let's say your plugin build a deploy a Readme for your contract on IPFS : 
```javascript
async function deployReadme(content) {
  const [ result ] = await ipfs.files.add(content);
  extension.emit('readmeDeployed', result.hash)
}
```

> Note: Be sure that your plugin is loaded before making any call.

<div align="center">
<img src="./doc/imgs/remix-extension.png" width="300">
</div>

### Testing your plugin
You can test your plugin direcly on the [alpha version of Remix-IDE](https://remix-alpha.ethereum.org). Go to the `pluginManager` (plug icon in the sidebar), and click "Connect to a Local Plugin".

Here you can add : 
- A name : this is the name used by other plugin to listen to your events.
- A displayName : Used by the IDE.
- The url : May be a localhost for testing.

> Note: No need to do anything if you localhost auto-reload, a new `handshake` will be send by the IDE.

### Publish your plugin
This is not available now.

# API
This API is a Work In Progress and will be extended in the future.

## fileManager

### Events
currentFileChanged
```typescript
extension.listen('fileManager', 'currentFileChanged', (fileName: string) => {
  // Do something
})
```

### Methods
getFilesFromPath
```typescript
const filesTree = await extension.call('fileManager', 'getFilesFromPath', path)
```

getCurrentFile
```typescript
const currentFile = await extension.call('fileManager', 'getCurrentFile')
```

getCurrentFile
```typescript
const content = await extension.call('fileManager', 'getFile', path)
```

setFile
```typescript
await extension.call('fileManager', 'setFile', path, content)
```

## solidity

### Events
compilationFinished
```typescript
extension.listen('solidity', 'compilationFinished', (fileName: string, source: Object, version: string, data: Object) => {
  // Do something
})
```

## txlistener

### Events
newTransaction
```typescript
extension.listen('txlistener', 'newTransaction', (tx: Object) => {
  // Do Something
})
```