### What is high-order component
You may find in your React project, same code is repeated in multiple components. It violate the "Don't repeat yourself"(`DRY`) principle.
>
Every piece of knowledge must have a single, unambiguous, authoritative representation within a systm.
>

To find a way to extact the same piece of logic from multiple compnents and elegently apply back to them is a question. `high-order compnent(HOC)` come to a picture. 

A hight-order component in React is a `design pattern` used to share common functionality between components without repeating code. HOC is not a component, it is a function. Let's have a look of what a `HOC`:
```javascript
import React, { Component } from 'react';

export default ChildComponent => {
  class ComposeComponent extends Component {

    // ... commont logic here

    render() {
      return <ChildComponent {...this.props} />;
    }
  }

  return ComposeComponent;
};
```
A `HOC` function takes a `ChildComponent` as parameter and returns a `ComposeComponent`. Inside the `ComposeCompnent` it render the `ChildCompnent` with some comment functionalities. In an other word, `HOC` transforms a component into another component and adds additional data or functionality. Two popular `HOC` impelementations are `connect` from `react-redux` and `withRouter` from `react-router`.

### How to use

**src/components/hoc.jsx**
```javascript
import React, { Component } from 'react';

export default ChildComponent => {  // ChildComponent must be in pascal case
  class ComposeComponent extends Component {
    componentDidMount(){
      console.log('component did mount from HOC');
    }

    render() {
      return <ChildComponent {...this.props} />;  // {...this.props} is very important
    }
  }

  return ComposeComponent;
```
**src/components/Foo.jsx**
```javascript
import React, { Component } from 'react';
import hoc from 'components/hoc';

class Foo extends Component{
  componentDidMount(){
      console.log('component did mount from Foo');
  }

  render(){
    ...
  }
}

export default hoc(Foo); // looks familiar to connect(mapStateToProps)(Foo)

// console output
// component did mount from Foo
// component did mount from HOC
```
Both of `hoc` and `Foo` have `componentDidMount` function. When render, the main compnent code get executed first follow by the one from `HOC`. Here is the take away from high-order compnent:
1. `HOC` is a function takes a compnent and returns a `ComposeComponent`
2. the component parameter must be in `Pascal Case`
3. common logic can be placed in side the `ComposeComponent`
4. `{...this.props}` is very important. It ensure all `props` passed from parent compnent are available in the `ChildCompnent` e.g. `Foo`
5. export default with `hoc(Foo)`

Do you familar with the last line of code in `Foo`? 
```javascript
export default hoc(Foo);
```
It is very similar to the `connect` in `react-redux` isn't it?
```javascript 
export default connect(mapStateToProps, mapActionToProps)(Foo);
```
Now we know the `connect` is actually a `high-order compnent` with `curry function`.

### Real life example
Here is a case that `CommentBox` compnent cannot be shown when user is not authorized. If the user is not authorized, navigate to home page. The code would be like:
**src/components/CommentBox.jsx**
```javascript
import React, { Component } from 'react';
import { connect } from 'react-redux';
import * as actions from 'actions';

class CommentBox extends Component {
  componentDidMount() {
      this.shouldNavigateAways();
    }

  componentDidUpdate() {
    this.shouldNavigateAways();
  }

  shouldNavigateAways() {
    if (!this.props.auth) {
      this.props.history.push('/');
    }
  }

  ... code omitted
  render() {
    return (
      ... code omitted
    );
  }
};

function mapStateToProps(state) {
    return { auth: state.auth };
}

export default connect(mapStateToProps, actions)(requireAuth(CommentBox));
```
Now we want to extract these four function to a high-order component named `requireAuth.jsx`
1. `componentDidMount`
2. `componentDidUpdate`
3. `shouldNavigateAways`
4. `mapStateToProps`
Notice that the code `this.props.auth` comes from redux `connect` and `this.props.history` comes from `Route`. The high-order compnent has to make sure the these two props are also a available to its `ChildComponent`. So the code above is transformed into two files:

**src/compnents/requireAuth.jsx**
```javascript
import React, { Component } from 'react';
import { connect } from 'react-redux';

export default ChildComponent => {
  class ComposeComponent extends Component {
    componentDidMount() {
      this.shouldNavigateAways();
    }

    componentDidUpdate() {
      this.shouldNavigateAways();
    }

    shouldNavigateAways() {
      if (!this.props.auth) {
        this.props.history.push('/');
      }
    }

    render() {
      return <ChildComponent {...this.props} />;
    }
  }

  function mapStateToProps(state) {
    return { auth: state.auth };
  }

  return connect(mapStateToProps)(ComposeComponent);
};
```

**src/components/CommentBox.jsx**
```javascript
import React, { Component } from 'react';
import { connect } from 'react-redux';
import * as actions from 'actions';
import requireAuth from 'components/requireAuth';

class CommentBox extends Component {
  ... code omitted
  render() {
    return (
      ... code omitted
    );
  }
};

export default connect(null, actions)(requireAuth(CommentBox));
```
Since the `mapStateToProps` function are extracted into `HOC`, so the `mapStateToProps` and `action` are passed into `CommentBox` via two `connect` one from `HOC`, the other from `CommentBox`. Now the `requireAuth` can be reused in multiple compnents.