%title: Nextjs Code-Along
%author: mike


-> # Slide 1 <-

Welcome Today we will intergrate gql into our app

--------------------------------------------------
-> # Slide 2 <-
==============

Step 1

* cd todo_app
* mkdir lib
* touch lib/init-graphql.js
* touch lib/with-graphql-client.js
* touch pages/_app.js



-------------------------------------------------
-> # Slide 3  <-

Step 2

Inside lib/init-graphql.js initialize client 

```
import { GraphQLClient } from 'graphql-hooks'
import memCache from 'graphql-hooks-memcache'
import unfetch from 'isomorphic-unfetch'

let graphQLClient = null

function create(initialState = {}) {
  return new GraphQLClient({
    ssrMode: typeof window === 'undefined',
    url: 'https://api.graph.cool/simple/v1/cixmkt2ul01q00122mksg82pn',
    cache: memCache({ initialState }),
    fetch: typeof window !== 'undefined' ? fetch.bind() : unfetch, // eslint-disable-line
  })
}

```


-------------------------------------------------

-> # Slide 4  <-

Step 3

Inside lib/init-graphql.js Make sure data isn't shared between connections

```
import { GraphQLClient } from 'graphql-hooks'
import memCache from 'graphql-hooks-memcache'
import unfetch from 'isomorphic-unfetch'

let graphQLClient = null
...

export default function initGraphQL(initialState) {
  // Make sure to create a new client for every server-side request so that data
  // isn't shared between connections (which would be bad)
  if (typeof window === 'undefined') {
    return create(initialState)
  }

  // Reuse client on the client-side
  if (!graphQLClient) {
    graphQLClient = create(initialState)
  }

  return graphQLClient
}

```

-------------------------------------------------

-> # Slide 5  <-

Step 4

lib/init-graphql.js final

```
import { GraphQLClient } from 'graphql-hooks'
import memCache from 'graphql-hooks-memcache'
import unfetch from 'isomorphic-unfetch'

let graphQLClient = null

function create(initialState = {}) {
  return new GraphQLClient({
    ssrMode: typeof window === 'undefined',
    url: 'https://api.graph.cool/simple/v1/cixmkt2ul01q00122mksg82pn',
    cache: memCache({ initialState }),
    fetch: typeof window !== 'undefined' ? fetch.bind() : unfetch, // eslint-disable-line
  })
}

export default function initGraphQL(initialState) {
  if (typeof window === 'undefined') {
    return create(initialState)
  }

  // Reuse client on the client-side
  if (!graphQLClient) {
    graphQLClient = create(initialState)
  }

  return graphQLClient
}

```

-------------------------------------------------

-> # Slide 6  <-

Step 5

inside lib/with-graphql-client.js first ...

```
import React from 'react'
import initGraphQL from './init-graphql'
import Head from 'next/head'
import { getInitialState } from 'graphql-hooks-ssr'

export default App => {
  return class GraphQLHooks extends React.Component {
    static displayName = 'GraphQLHooks(App)'
    static async getInitialProps(ctx) {}
}

```

-------------------------------------------------

-> # Slide 7  <-

Step 6

inside lib/with-graphql-client.js first ...

```
...
    static async getInitialProps(ctx) {
      const { AppTree } = ctx

      let appProps = {}
      if (App.getInitialProps) {
        appProps = await App.getInitialProps(ctx)
      }

      // Run all GraphQL queries in the component tree
      // and extract the resulting data
      const graphQLClient = initGraphQL()
      let graphQLState = {}
      if (typeof window === 'undefined') {
        try {
          // Run all GraphQL queries
          graphQLState = await getInitialState({
            App: <AppTree {...appProps} graphQLClient={graphQLClient} />,
            client: graphQLClient,
          })
        } catch (error) {
          // Prevent GraphQL hooks client errors from crashing SSR.
          // Handle them in components via the state.error prop:
          // https://github.com/nearform/graphql-hooks#usequery
          console.error('Error while running `getInitialState`', error)
        }

        // getInitialState does not call componentWillUnmount
        // head side effect therefore need to be cleared manually
        Head.rewind()
      }

      return {
        ...appProps,
        graphQLState,
      }
    }
...
```


-------------------------------------------------

-> # Slide 8  <-

Step 7

inside lib/with-graphql-client.js final

```
import React from 'react'
import initGraphQL from './init-graphql'
import Head from 'next/head'
import { getInitialState } from 'graphql-hooks-ssr'

export default App => {
  return class GraphQLHooks extends React.Component {
    static displayName = 'GraphQLHooks(App)'
    static async getInitialProps(ctx) {
      const { AppTree } = ctx

      let appProps = {}
      if (App.getInitialProps) {...}

    constructor(props) {
      super(props)
      this.graphQLClient = initGraphQL(props.graphQLState)
    }

    render() {
      return <App {...this.props} graphQLClient={this.graphQLClient} />
    }
  }
}

```

-------------------------------------------------

-> # Slide 9  <-

Step 8


Inside pages/app.js

```

import App from 'next/app'
import React from 'react'
import withGraphQLClient from '../lib/with-graphql-client'
import { ClientContext } from 'graphql-hooks'

class MyApp extends App {
  render() {
    const { Component, pageProps, graphQLClient } = this.props
    return (
      <ClientContext.Provider value={graphQLClient}>
        <Component {...pageProps} />
      </ClientContext.Provider>
    )
  }
}

export default withGraphQLClient(MyApp)

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
      <h1>About Todo</h1>
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

update pages/index.html

```
import App from '../components/app'
import Header from '../components/header'

export default () => (
  <App>
    <Header />
    <P>yurrrrrrr</p>
  </App>
)

```
  
-------------------------------------------------

-> # Slide 12  <-

Step 11

lets run this mf*

* yarn dev
* go to http://localhost:3000/


-------------------------------------------------
-> # Conclusion  <-
