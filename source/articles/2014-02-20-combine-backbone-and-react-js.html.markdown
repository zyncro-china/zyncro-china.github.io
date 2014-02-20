---
title: Combine Backbone and React.js
date: 2014-02-20 03:42 UTC
tags:
---

## Get the best of both Backbone and React.js

Backbone is a small and simple framework for Single Page Application. But
rendering views is hard, so let's try to use React.js together, and let
React.js to finish the hard work for us.

## React.js Framework
### JSX syntax
We can use coffeescript to make writting component easier directly in the
coffeescript without compiling first
### Object Definition and Intializaion

1. to define a component: `React.createClass`
1. use the component, and pass the props: `React.renderComponent`
1. called automatically by React before a component is rendered: `componentWillMount`
1. called before unmount component: `componentWillUnmount`

### Reactive state
render() methods are written declaratively as functions of `this.props` and
`this.state.` The framework guarantees the UI is always consistent with the
inputs.

React.js will automatically update the view according to the state

1. initialize the state: `getInitialState`
1. update the state: `this.setState()`

### React events
event handler should only returns false to avoid the default action

### refs
refs are defined in the JSX, and use `getDOMNode()` to get the native DOM element

```javascript
text = this.refs.text.getDOMNode().value.trim()

```

## Communication

### Communication from React Component to Backbone
pass the callback as the React Component's callback using `props`

### Communication from Backbone to React Component
use Backbone Model's event binding, like:

```coffeescript
this.props.model.on 'change',=>
  @forceUpdate()
```
