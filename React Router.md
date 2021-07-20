# React Router

> [React Router: Declarative Routing for React.js](https://reactrouter.com/web/guides/quick-start)

## Installation

```sh
npm install react-router-dom
```

## Basic

基本结构：

```jsx
<Router>
    <Link to="/homepage"></Link>

    <Switch>
        <Route path="/homepage">
            <Homepage/>
        </Route>
    </Switch>
</Router>
```

> `<Link>` renders an `<a>` with a `href`

In this example we have 3 “pages” handled by the router: a *home* page, an *about* page, and a *users* page. As you click around on the different `<Link>`s, the router renders the matching `<Route>`.

```jsx
import React from "react";
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Link
} from "react-router-dom";

export default function App() {
  return (
    <Router>
      <div>
        <nav>
          <ul>
            <li>
              <Link to="/">Home</Link>
            </li>
            <li>
              <Link to="/about">About</Link>
            </li>
            <li>
              <Link to="/users">Users</Link>
            </li>
          </ul>
        </nav>

        {/* A <Switch> looks through its children <Route>s and
            renders the first one that matches the current URL. */}
        <Switch>
          <Route path="/about">
            <About />
          </Route>
          <Route path="/users">
            <Users />
          </Route>
          <Route path="/">
            <Home />
          </Route>
        </Switch>
      </div>
    </Router>
  );
}

function Home() {
  return <h2>Home</h2>;
}

function About() {
  return <h2>About</h2>;
}

function Users() {
  return <h2>Users</h2>;
}
```



### Primary Components

- routers, like `<BrowserRouter>` and `<HashRouter>`
- route matchers, like `<Route>` and `<Switch>`
- and navigation, like `<Link>`, `<NavLink>`, and `<Redirect>`



#### Router 父组件：`<BrowserRouter>` & `<HashRouter>`

- A `<BrowserRouter>` uses regular URL paths. These are generally the best-looking URLs, but they require your server to be configured correctly. Specifically, your web server needs to serve the same page at all URLs that are managed client-side by React Router. Create React App supports this out of the box in development, and [comes with instructions](https://create-react-app.dev/docs/deployment#serving-apps-with-client-side-routing) on how to configure your production server as well. (Vue: history mode)
- A `<HashRouter>` stores the current location in [the `hash` portion of the URL](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/hash), so the URL looks something like `http://example.com/#/your/page`. Since the hash is never sent to the server, this means that no special server configuration is needed. (Vue: hash mode)

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter } from "react-router-dom";

function App() {
  return <h1>Hello React Router</h1>;
}

ReactDOM.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  document.getElementById("root")
);
```



#### Router Matchers:  `<Switch>` & `<Route>`

When a `<Switch>` is rendered, it searches through its `children` `<Route>` elements to find one whose `path` matches the current URL. When it finds one, it renders that `<Route>` and ignores all others. This means that you should put `<Route>`s with more specific (typically longer) `path`s **before** less-specific ones.



If no `<Route>` matches, the `<Switch>` renders nothing (`null`).



`<Route path>` matches the **beginning** of the URL, not the whole thing. So a `<Route path="/">` will **always** match the URL. Because of this, we typically put this `<Route>` last in our `<Switch>` as fallback option.

`<Route exact path="/">` which **does** match the entire URL.



```jsx
<Switch>
	<Route path="/homepage">
    ...
  </Route>
  <Route path="/about">
    ...
  </Route>
  <Route path="/">
    {/*matches everything: fallback option*/}
    ...
  </Route>
</Switch>
```



#### Navigation: `<Link>`, `<NavLink>` & `<Redirect>`

`<Link>` component to create links in your application. Wherever you render a `<Link>`, an anchor (`<a>`) will be rendered in your HTML document.

The `<NavLink>` is a special type of `<Link>` that can style itself as “active” when its `to` prop matches the current location.

`<Redirect from=“” to="">`: force navigation.

```jsx
<Link to="/">Home</Link>
// <a href="/">Home</a>

<NavLink to="/react" activeClassName="hurray">
  React
</NavLink>

// When the URL is /react, this renders:
// <a href="/react" className="hurray">React</a>

// When it's something else:
// <a href="/react">React</a>

<Redirect to="/login" />
```



### Hooks

> Please note: You need to be using React >= **16.8** in order to use any of these hooks!

- [`useHistory`](https://reactrouter.com/web/api/Hooks/usehistory)
- [`useLocation`](https://reactrouter.com/web/api/Hooks/uselocation)
- [`useParams`](https://reactrouter.com/web/api/Hooks/useparams)
- [`useRouteMatch`](https://reactrouter.com/web/api/Hooks/useroutematch)



#### [`useHistory`](https://reactrouter.com/web/api/Hooks/usehistory)

The `useHistory` hook gives you access to the [`history`](https://reactrouter.com/web/api/history) instance that you may use to navigate.

```jsx
import { useHistory } from "react-router-dom";

function HomeButton() {
  let history = useHistory();

  function handleClick() {
    history.push("/home");
  }

  return (
    <button type="button" onClick={handleClick}>
      Go home
    </button>
  );
}
```



#### [`useLocation`](https://reactrouter.com/web/api/Hooks/uselocation)

The `useLocation` hook returns the [`location`](https://reactrouter.com/web/api/location) object that represents the current URL. You can think about it like a `useState` that returns a new `location` whenever the URL changes.

```jsx
function usePageViews() {
  let location = useLocation();
  React.useEffect(() => {
    ga.send(["pageview", location.pathname]);
  }, [location]);
}
```



#### [`useParams`](https://reactrouter.com/web/api/Hooks/useparams)

`useParams` returns an object of key/value pairs of URL parameters. Use it to access `match.params` of the current `<Route>`.



#### [`useRouteMatch`](https://reactrouter.com/web/api/Hooks/useroutematch)

The `useRouteMatch` hook attempts to [match](https://reactrouter.com/web/api/match) the current URL in the same way that a `<Route>` would. It’s mostly useful for getting access to the match data without actually rendering a `<Route>`.

返回相应url的[match](https://reactrouter.com/web/api/match)对象



```jsx
import { useRouteMatch } from "react-router-dom";

function BlogPost() {
  let match = useRouteMatch("/blog/:slug");

  // Do whatever you want with the match...
  return <div />;
}
```

相当于

```jsx
import { Route } from "react-router-dom";

function BlogPost() {
  return (
    <Route
      path="/blog/:slug"
      render={({ match }) => {
        // Do whatever you want with the match...
        return <div />;
      }}
    />
  );
}
```



