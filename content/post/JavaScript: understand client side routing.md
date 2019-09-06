### What is routing?
When browsing a website, entering different `URL` on address bar would have different web page. 

Url | route path| description
------------- | ------------- | -------------
www.mycompany.com | `/` | the home page
www.mycompany.com/employee | `/employee` | a list of employees
www.mycompany.com/employee/johndon | `/employee/johndon` | the details of employee named John Don

Web page routing originally came from server side. In old schools day, web server maintains a route map which is a list of key value pairs of route path and method `{"employee", getAllEmployee}`.

When a web server receives a `http` request from client side, the server parse the route path `employee` from `URL`, look for a route from route map, and then execute the paired method `getAllEmployee`. In side the method, it may query DB for data and then feed into a web template and lastly return a HTML to client side for rendering.

For a static web side, the route path indicates the file path of the web page.

#### pros and cons
* pros: `SEO` friendly, more secure
* cons: low user experience (refresh page), high server resource consuming.

Nowadays, as Javascript front end frameworks catch the hype, the client side routing has come to place. Different from server side route method, the client side route function dose not generate HTML, it toggles on and off `DOM` element from displaying. Visiting different route path shows different component. There are two aways to implement client side routing:
1. Hash
2. History

### Hash router
Before `HTML 5`, we have not choice but using `location.hash` to track the route path. The `location.hash` is the string value after a `#` in `URL`. Like the url below, the hash value is `#/sponsors`
```javascript
http://www.mycompany.com/#/sponsors

console.log(location.hash) // "#/sponsors"
```
`Hash` is featured with:
1. The `hash` value does not send back to server. It is a status of a `URL`
2. Browser history records `hash` change, which means we can use `forward` or `backward` browser to navigate pages.
3. Leveraging global event `hashchange` making client side route much easier.
4. Triggering route is easy. Just do `location.hash="#newPath"`

#### BaseRouter
Let's implement a routing base on a given route map. This is also the parent class of both `Hash route` and `History route`.
```javascript
class BaseRouter {
  constructor(routeMap) {
    this.container = document.querySelecotr('#container');
    this.routeMap = routeMap; // e.g. [{path: '/', component: home}, {path: '/employee', component: employeeList}]
  }

  // inject path component to container
  render(path){
    let route = this.routeMap.find(item => item.path === path);
    route = route ? route : this.routeMap.find(item => item.path === '*');
    this.container.innerHTML = route.component;
  }
}

export default BaseRouter;
```
First of all, we need to define a `container` element which is used to display different page content a.k.a component. It can be a `div` tag or and `body` tag. Then a `routeMap` should be provided. It is an array of route objects e.g.
```javascript
[
  {path: '/', component: home},
  {path: '/employee', component: employeeList},
  {path: '*', component: 404} // make sure place the `*` as the last array item or all URL go to the same page.
]
```
In the `render` function, we looking for the `path` value from routeMap and then inject the associated component into the `container` element. This is the basic implementation of a router. Now we need to connect it to `Hash` in `URL`

#### Hash route implementation
Before we start implement the `Hash` router code, think about, what functions javascript would call to navigate page.
1. `goToPage` a function trigger routing
2. `navigateHistory` a function navigate forward and backward of history

Let's consider the code:
```javascript
class HashRouter extends BaseRouter {
  constructor(routeMap){
    super(routeMap);
    this.handler();

    window.addEventListener('haschange', e => {
      this.handler();
    });

  }

  // call the render function from parent class
  // with current hash
  handler(){
    let hash = window.location.hash;
    hash = hash ? hash.slice(1) : '/';
    this.render(hash);
  }

  // change hash value. It is going to trigger the 'hashchange' event
  goToPage(path){
    window.location.hash = path;
  }

  navigateHisotry(n) {
    window.history.go(n);
  }
}
```

### History router
`hash router` works well but `#` does not look nice in `URL`. Thanks to `HTML5`, A new `History API` make `History router` come to picture.  `History.pushState()`, it adds a new history record
```javascript
window.history.pushState(null, null, path);
```
Function asks for three parameter: `state`, `title` and `URL`. We can set the first two as `null` as we don't concern but the last one  `URL`,

*  `History.pushState` change the `window.history` data without refreshing page.
* A `popState` event can be used to track `URL` change.
* Since the `state` parameter is always null, `popState` event won't be triggered.
* Use the `window.location.pathname` to get current path.

#### History route implementation
Similar to `Hash` router, the History Router is outputting:
1. `goToPage` a function trigger routing
2. `navigateHistory` a function navigate forward and backward of history

```javascript
class HistoryRouter extends BaseRouter {
  constructor(routeMap){
    super(routeMap);
    this.handler();

    window.addEventListener('popState', e => {
      this.handler();
    });
  }

  // call the render function from parent class
  // with current path
  handler(){
    const path = window.location.pathname;
    this.render(path ? path : '/');
  }

  goToPage(path){
    window.hisotry.pushState(null, null, path);
    this.handler();   // manually trigger the page change as `popState` cannot be triggered.
  }

  navigateHisotry(n){
    window.history.go(n);
  }
}
```
### Change web server config
To use the `History` router, we have to make a config change on the web server, but `Hash` router does not. Why? To answer that, we have to understand how modern javascript framework work on routing.

Traditionally, the `URL` path has one to one mapping to the source file. When a web server receive a request like _http://www.mycompany.com/employee/johndon_, it looks into the file system of the source file. Trying to find a folder named 'employee' and then look for 'johndon.html' file and then send the file back to client. If the file cannot be found, by default, web server return `404.html` page.

The modern javascript framework website does not have the mapping files, it cares about the root `URL` (_http://www.mycompany.com_) only. The `index.html` page must be loaded at the very first time even though we want _http://www.mycompany.com/employee/johndon_. 

While loading the `index.html`, javascript kicks in and read the path (_/employee/johndon_) and then render the corresponding component. So behind the sense, the browser render the `index.html` page first and then quickly toggle to `employee profile` page. It happens lighting fast that human cannot notice.

To make the `History` router working, we have to change the web server config to let it return `index.html` instead of `404.html` when the file not found.

`Hash` router, on the other hand, does not required web server config change as the `hash` value does not post back to server. For instance, visiting _http://www.mycompany.com/#/employee/johndon_, the hash value _/#/employee/johndon_ stay in browser. Only the _http://www.mycompany.com_ request get send back to server. Obviously, it always returns `index.html`.

### IE support
Last thing to mention. `Internet Explorer version 8.0` or later supports `Hash` router. `Internet Explorer version 10.0` or later supports `History` router. 