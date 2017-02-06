# tdux [![Build Status][build]](https://circleci.com/gh/bcherny/tdux) [![npm]](https://www.npmjs.com/package/tdux) [![mit]](https://opensource.org/licenses/MIT)

[build]: https://img.shields.io/circleci/project/bcherny/tdux.svg?branch=master&style=flat-square
[npm]: https://img.shields.io/npm/v/tdux.svg?style=flat-square
[mit]: https://img.shields.io/npm/l/tdux.svg?style=flat-square

> Better, Type Safe Redux.

## Highlights

- 100% type-safe:
  - Statically guarantees that a reducer is defined for each Action
  - Statically guarantees that emitters are called with the correct Action data given their Action name
  - Statically guarantees that listeners are called with the correct Action data given their Action name
- Mental model similar to [Redux](https://github.com/reactjs/redux), with several improvements over Redux:
  - Store is decoupled from emitter
  - Emitters are reactive; in fact, they use [Rx](https://github.com/Reactive-Extensions/RxJS) Observables!
  - Listeners are on specific Actions
  - Listeners are called with both current and previous values (convenience borrowed from Angular $watch/Object.Observe)

## Conceptual Overview

1. Create a Tdux `Emitter` with a set of supported `Action`s
2. Register reducers on the emitter (a "reducer" is a mapping from a given `Action` to its side effects)
3. Components in your app `dispatch` `Action`s on your emitter
4. `Actions` first trigger side-effects (via their respective reducers), then trigger any callbacks listening on that `Action`

## Installation

```sh
npm install tdux --save
```

## Usage

```ts
import { Emitter } from 'tdux'

// Mock store
const store: { [id: number]: boolean } = {}

// Enumerate actions
type Actions = {
  INCREMENT_COUNTER: number
  OPEN_MODAL: boolean
}

// Define Tdux Emitter
class App extends Emitter<Actions> { }

// Create bus and register reducers (throws a compile time error unless both of these keys are defined, and return values of the right types)
const app = new App({
  INCREMENT_COUNTER: ({ id, value }) => {
    const previousValue = store[id]
    store[id] = value
    return previousValue
  },
  OPEN_MODAL: ({ id, value }) => {
    const previousValue = store[id]
    store[id] = value
    return previousValue
  }
})

// Trigger an action (throws a compile time error unless id and value are set, and are of the right types)
app.emit('OPEN_MODAL', { id: 123, value: true })

// Listen on an action (basic) (throws a compile time error if this event does not exist)
app.on('OPEN_MODAL')
   .subscribe(_ => _.value)

// Listen on an action (advanced)
app.on('INCREMENT_COUNTER')
   .filter(_ => _.id === 42)
   .debounce()
   .subscribe(_ => console.log(`Counter incremented from ${_.previousValue} to ${_.value}!`))
```

## Tests

```sh
npm test
```
