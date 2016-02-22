# cyclic-dependency-graph

A state machine that handles building a cyclic directed graph from
the dependencies in a codebase.


## Install

`npm install --save cyclic-dependency-graph`


## Example

```js
import {createGraph} from 'cyclic-dependency-graph';

// The files that we want to build the graph from
const entryPoint1 = '/path/to/file_1';
const entryPoint2 = '/path/to/file_2';

const graph = createGraph({
  getDependencies(node) {
    console.log(`Dependencies requested for ${node}`);

    // Get the dependencies for the node ...

    if (someErr) {
      return Promise.reject(someErr);
    }

    return Promise.resolve(['/path/to/dependency', '...']);
  }
});

// Inform the graph that it should treat our files as entry points
graph.setNodeAsEntry(entryPoint1);
graph.setNodeAsEntry(entryPoint2);

graph.events.on('error', ({error, node}) => {
  console.error(
    `Error when tracing ${node}: ${error}`
  );
});

graph.events.on('traced', ({node}) => {
  console.log(`Traced: ${node}`);
});

graph.events.on('completed', ({diff, errors}) => {
  if (errors.length) {
    return console.error('Errors during tracing!');
  }

  console.log('Graph state: ' + graph.getState());
});

// Start the process of building the graph
graph.traceFromNode(entryPoint1);
graph.traceFromNode(entryPoint2);
```


## Data structures

### Graph state

Stored as a flat `Map` structure (using the Map implementation from `immutable`).

A map structure was used primarily for simplicity and the low costs associated with
node lookup and traversal.


### Node

```js
{
  name: '...',
  dependencies: Set(),
  dependents: Set()
  isEntryNode: Bool()
}
```


### Diff

```js
{
  from: Map(),
  to: Map()
}
```


## Events

### started

```js
{
  state: Map()
}
```

### completed

```js
{
  errors: [
    Error(),
    // ...
  ],
  diff: Diff()
}
```

### traced

```js
{
  node: '...',
  diff: Diff()
}
```

### error

```js
{
  node: '...',
  error: Error(),
  diff: Diff()
}
```
