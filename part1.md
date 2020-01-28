%title: Nextjs Code-Along
%author: mike


-> # Slide 1 <-

Welcome Today we will be making a todo app using next.js and graphql using hooks

--------------------------------------------------
-> # Slide 2 <-
==============

Step 1

* yarn add create-next-app-cli
* yarn create next-app todo_app
* cd todo_app



-------------------------------------------------
-> # Slide 3  <-

Step 2

install these libraries

* yarn add graphql-hooks graphql-hooks-memcache graphql-hooks-ssr 
* yarn add isomorphic-unfetch prop-types


-------------------------------------------------

-> # Slide 4  <-

Step 3

As you can see an app was created for us by the cli we are now going to remove the unneeded code

* rm -rf todo_app/components/nav.js
* rm -rf todo_app/pages/index.js
* touch components/app.js
* touch components/index.js


-------------------------------------------------

-> # Slide 5  <-

Step 4

inside components/app.js

```
...
    {children}
    <style jsx global>{`
      * {
        font-family: Menlo, Monaco, 'Lucida Console', 'Liberation Mono',
          'DejaVu Sans Mono', 'Bitstream Vera Sans Mono', 'Courier New',
          monospace, serif;
      }
      body {
        margin: 0;
        padding: 25px 50px;
      }
      a {
        color: #22bad9;
      }
      p {
        font-size: 14px;
        line-height: 24px;
      }
      article {
        margin: 0 auto;
        max-width: 650px;
      }
      button {
        align-items: center;
        background-color: #22bad9;
        border: 0;
        color: white;
        display: flex;
        padding: 5px 7px;
      }
      button:active {
        background-color: #1b9db7;
        transition: background-color 0.3s;
      }
      button:focus {
        outline: none;
      }
    `}</style>
...
```

-------------------------------------------------


-> # Slide 5  <-

Step 4

lets add some style to components/app.js

```
export default ({ children }) => (
  <main>
    {children}
  </main>
)

```

-------------------------------------------------

-> # Slide 6  <-

Step 5

inside pages/index.js

```
import App from '../components/app'

export default () => (
  <App>
  <P>yurrrrrrr</p>
  </App>
)


```

-------------------------------------------------

-> # Slide 7  <-

Step 6

* touch components/header.js
* touch pages/about.js


-------------------------------------------------

-> # Slide 8  <-

Step 7

Inside components/header.js

```
import Link from 'next/link'
import { withRouter } from 'next/router'

const Header = ({ router: { pathname } }) => (
  <header>
    <Link href="/">
      <a className={pathname === '/' ? 'is-active' : ''}>Home</a>
    </Link>
    <Link href="/about">
      <a className={pathname === '/about' ? 'is-active' : ''}>About</a>
    </Link>
  </header>
)

export default withRouter(Header)


```

-------------------------------------------------

-> # Slide 9  <-

Step 8


Inside components/header.js lets add some style

```
...
const Header = ({ router: { pathname } }) => (
  <header>
  	...
    <style jsx>{`
      header {
        margin-bottom: 25px;
      }
      a {
        font-size: 14px;
        margin-right: 15px;
        text-decoration: none;
      }
      .is-active {
        text-decoration: underline;
      }
    `}</style>
  </header>
)
export default withRouter(Header)

```


-------------------------------------------------

-> # Slide 10  <-

Step 9

Inside pages/about.js lets add some style

```
import App from '../components/app'
import Header from '../components/header'

export default () => (
  <App>
    <Header />
    <article>
      <h1>About Diss</h1>
      <p> 
      	Some Text 
      </p>
    </article>
  </App>
)
```

-------------------------------------------------

-> # Slide 11  <-

Step 10

* touch components/header.js
* touch pages/about.js


-------------------------------------------------

-> # Slide 12  <-

Step 11

lets run this mf*

* yarn dev
* go to http://localhost:3000/


-------------------------------------------------
-> # Conclusion  <-
