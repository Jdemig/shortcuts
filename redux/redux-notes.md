# Redux Notes

Redux is a predictable state container for JavaScript apps.

It helps you write applications that behave consistently, run in different
environments (client, server, and native), and are easy to test.

Redux is a tool/library for managing both data-state and UI-state in 
JavaScript applications.

Redux stores all your application state in one place, called a "store".
Components then "dispatch" state changes to the store, not directly to other
components.  The components that need to be aware of state changes can
"subscribe" to the store.

The whole state of your app is stored in an object tree inside a single store.
The only way to change the state tree is to emit an action, an object
describing what happened.  To specify how the actions transform the state
tree, you write pure reducers.

That's it!


## Installation

```sh
npm install --save redux
```


## Three Guiding Principles

1) **Single Source of Truth**: The state of your whole application is stored in
   an object tree within a single store.

2) **State is Read-Only**: The only way to change the state is to emit an action,
   an object describing what happened.

3) **Changes are made with Pure Functions**: To specify how the state tree is
   transformed by actions, you write pure reducers.


## Actions

[Actions][action] are payloads of information that send data from your
application to your store.  Actions are just plain JavaScript objects with a
globally unique `type` property.

```js
{
  type: 'ADD_TODO',
  text: 'Demo an action'
}
```

In order to enable tooling, you can shape your actions to match the [Flux
Standard Action][fsa] (FSA).

```js
{
  type: 'ADD_TODO',
  payload: {
    text: 'Demo an FSA'
  }
}
```

See https://redux.js.org/docs/basics/Actions.html and
http://redux.js.org/docs/recipes/ReducingBoilerplate.html#actions for more
details.

**Action Creators**

Action creators are functions that create actions.  They are a good way to
hide details about the structure of the action.  This makes it easy to switch
to using FSAs in the future.

```js
function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text: 'Demo action creators'
  }
}
```


## Reducers

Reducers specify how the application's state changes in response to actions.

A reducer is a pure function that takes the previous state and an action, and
returns the next state.

```js
(prevState, action) => nextState
```

It's very important that the reducer stays pure.  Things you should never do
inside a reducer:

* Mutate its arguments.
* Perform side effects like API calls and routing transitions.
* Call non-pure functions, e.g. `Date.now()` or `Math.random()`.

**Splitting Reducers**

As the application grows, a single reducer can become very verbose making it
hard to comprehend.  A technique called reducer composition can help by
breaking the main reducer into smaller, more managable reducers that each
handle a small portion of the state tree.  See
https://redux.js.org/docs/basics/Reducers.html#splitting-reducers for details
on how this is done.

Making use of `combineReducers` can help reduce some boilerplate.

```js
// ./reducers/index.js
import React from 'react'
import Redux from 'redux'

import bar from './reducers/bar'
import foo from './reducers/foo'

const rootStateReducer = Redux.combineReducers({
  foo,
  bar,
})

export default rootStateReducer
```

See https://redux.js.org/docs/api/combineReducers.html for details.


## Store

The Store is the object that brings the actions and reducers together.  The
store has the following responsibilities:

* Holds application state.
* Allows access to state via [`getState()`][get-state].
* Allows state to be updated via [`dispatch(action)`][dispatch].
* Registers listeners via [`subscribe(listener)`][subscribe].
* Handles unregistering of listeners via the function returned by
  `subscribe(listener)`.

> It's important to note that you'll only have a single store in a Redux
> application.  When you want to split your data handling logic, you'll use
> reducer composition instead of many stores.

**Creation**

```js
import { createStore } from 'redux'
import rootStateReducer from './reducers'

let store = createStore(rootStateReducer)
```

**Creation with Preloaded State**

```js
import { createStore } from 'redux'
import rootStateReducer from './reducers'

// Example of preloaded state:
const preloadedState = window.STATE_FROM_SERVER

let store = createStore(rootStateReducer, preloadedState)
```

**Creation with Enhancements**

The third argument accepts a [store enhancer][store-enhancer].  A store
enhancer is a higher-order function that composes a
[store creator][store-creator] to return a new, enhanced store creator.  The
only store enhancer that ships with Redux is
[`applyMiddleware()`][applyMiddleware].

```js
import { createStore, applyMiddleware } from 'redux'
import reduxThunk from 'redux-thunk';
import rootStateReducer from './reducers'

let store = createStore(rootStateReducer, ..., applyMiddleware(reduxThunk))
```

See https://redux.js.org/docs/api/createStore.html for details.


## State

The shape of the app's state is entirely up to the developer.

**Designing the State Shape**

In Redux, all the application state is stored as a single object.  It's a good
idea to think of its shape before writing any code.  You'll often find that
you need to store some data, as well as some UI state, in the state tree.
This is fine, but try to keep the data separate from the UI state.

> In a more complex app, you're going to want different entities to reference
> each other.  We suggest that you keep your state as normalized as possible,
> without any nesting.  Keep every entity in an object stored with an ID as a
> key, and use IDs to reference it from other entities, or lists.  Think of
> the app's state as a database.  This approach is described in
> [normalizr's][normalizr] documentation in detail.


## Usage with React

```sh
npm install --save react-redux
```

`connect` from React Redux generates an [HOC][] of another component, usually a
presentational component.  See the [connect API][connect-api] for details.

```js
import { connect } from 'react-redux'

import PresentationalComponent from './components/Presentationl'


const mapStateToProps = (state) => {
  // Create presentational component props from current state.
  return props
}

const mapDispatchToProps = (dispatch) => {
  // Create component props from dispatch function.  These are usually event
  // handlers passed to the presentational component.
  return props
}

const ContainerComponent = connect(
  mapStateToProps,
  mapDispatchToProps
)(PresentationalComponent)

export default ContainerComponent
```

```js
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore } from 'redux'
import { Provider } from 'react-redux'

import rootStateReducer from './reducers'
import App from './components/App'

let store = createStore(rootStateReducer)

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

See https://redux.js.org/docs/basics/UsageWithReact.html for details.


[action]: https://redux.js.org/docs/Glossary.html#action
[applyMiddleware]: https://redux.js.org/docs/api/applyMiddleware.html
[connect-api]: https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options
[dispatch]: https://redux.js.org/docs/api/Store.html#dispatch
[fsa]: https://github.com/acdlite/flux-standard-action
[get-state]: https://redux.js.org/docs/api/Store.html#getState
[hoc]: https://reactjs.org/docs/higher-order-components.html
[normalizr]: https://github.com/paularmstrong/normalizr
[store-creator]: https://redux.js.org/docs/Glossary.html#store-creator
[store-enhancer]: https://redux.js.org/docs/Glossary.html#store-enhancer
[subscribe]: https://redux.js.org/docs/api/Store.html#subscribe
