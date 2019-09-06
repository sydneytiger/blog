### How react-redux work
Redux plays an important role in React application. It provides a central data store that all react components are able to access via the `connect` function. As shown in the diagram below, components like `CommentBox` and `CommentList` are able to access the `store`. Every time the `store` data get update, it notifies all connected components. Whereas, the `Footer` component does not require any `store` data. No `connect` is set in `Footer`. 

![Redux structure](https://user-images.githubusercontent.com/1787825/63941211-70f90900-caae-11e9-888e-4f1f8f6b400c.jpg)

To make the `store` accessible, the root component `App` is the best place to wrap with `Provider`. So that all components can connect to `Provider` via `App`. Let's look into the code

**src/index.js**
```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import App from 'components/App';
import { Provider } from 'react-redux';
import { createStore, applyMiddleware } from 'redux';
import reduxPromise from 'redux-promise';
import reducers from 'reducers';

const store = createStore(reducers, initState, applyMiddleware(reduxPromise));
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.querySelector('#root'));
```

**src/components/App.jsx**
```javascript
import React from 'react';
import CommentBox from 'components/CommentBox';
import CommentList from 'components/CommentList';
import Footer from 'components/Footer';

export default () => {
  return (
    <div className="container">
      <div><CommentBox /></div>
      <div><CommentList /> </div>
      <div><Footer /></div>
    </div>
  );
}
```
**src/components/CommentBox.jsx**
```javascript
import React, { Component } from 'react';
import { connect } from 'react-redux';

class CommentBox extends Component {
  ...
}

function mapStateToProps(state) {
  return {
    comments: state.comments
  }
}

export default connect(mapStateToProps)(CommentBox);
```
The `mapStateToProps` is a key function. It tells `connect` that which data in the central `store` the `CommentBox` component interested in and then map the data into `props`. This is how data pass down to a component.

### Make Provider reusable
Now, let's refactor the `Provider` a bit by creating a new file called `Root.jsx`
**src/Root.jsx**
```javascript
import React from 'react';
import { Provider } from 'react-redux';
import { createStore, applyMiddleware } from 'redux';
import reduxPromise from 'redux-promise';
import reducers from 'reducers';

export default ({ children, initState = {} }) => {
  const store = createStore(reducers, initState, applyMiddleware(reduxPromise));

  return (
    <Provider store={store}>
      {children}
    </Provider>
  );
};
```
Note that I have destructed the `props` into `children` and `initState` variables. 
`children` is a special React property of all elements inside current tag e.g.
```javascript
<MyHeader>
  <MyBody>
    <MyDiv />
  </MyBody>
</MyHeader>

// <MyDiv /> is the children property of MyBody component. 
// <MyBody><MyDiv /></MyBody> is the children property of MyHeader component.
```
`initState` is an initial `store` data. It will pass in `createStore` function.

Now the redux `Provider` function has been extracted. To use it, we should modify our `index.js` a bit.
**src/index.js**
```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import App from 'components/App';
import Root from 'Root';

ReactDOM.render(
  <Root>
    <App />
  </Root>,
  document.querySelector('#root'));
```
### Why and where to reuse the Provider
You may ask, `App` is the appshell, when wrapping it with `react-redux` `Provider`, the entire application will work well. Why do you extract out to a `Root`. It is because of unit testing. In React, we break down an app into many components. Each component has it own unit test. `Jest`, the test framework runs on `jsdom` which is not a browser environment and create the testing component only, not the entire app. It will looks like:
![Redux test structure](https://user-images.githubusercontent.com/1787825/64100110-3ba23300-cdae-11e9-81bb-3fc5f6b90544.jpg)
So in a unit test, we should use `Root` to wrap the component instead of the `App`. Here is a example code:
```javascript
import React from 'react';
import CommentBox from 'components/CommentBox';
import { mount } from 'enzyme';
import Root from 'Root';

describe('<CommentBox />', () => {
  let wrapper;

  beforeEach(() => {
    wrapper = mount(<Root><CommentBox /></Root>);
  });

  afterEach(() => {
    wrapper.unmount();
  });

  // ... test suites here
```