%title: Nextjs Code-Along
%author: mike


-> # Slide 1 <-

Welcome Today we will add our todo list

--------------------------------------------------
-> # Slide 2 <-
==============

Step 1

* cd todo_app
* touch components/post-list.js
* touch components/error-message.js

-------------------------------------------------
-> # Slide 3  <-

Step 2

Inside components/error-message.js with a little css

```
export default ({ message }) => (
  <aside>
    {message}
    <style jsx>{`
      aside {
        padding: 1.5em;
        font-size: 14px;
        color: white;
        background-color: red;
      }
    `}</style>
  </aside>
)
```


-------------------------------------------------

-> # Slide 4  <-

Step 3

First lets create our query inside components/post-list.js 

```
import React, { Fragment, useState } from 'react'
import { useQuery } from 'graphql-hooks'
import ErrorMessage from './error-message'

export const allPostsQuery = `
  query allPosts($first: Int!, $skip: Int!) {
    allPosts(orderBy: createdAt_DESC, first: $first, skip: $skip) {
      id
      title
      votes
      url
      createdAt
    }
    _allPostsMeta {
      count
    }
  }
`

export default function PostList() {}


```

-------------------------------------------------

-> # Slide 5  <-

Step 4

Inside components/post-list.js lets create our hooks

```
import React, { Fragment, useState } from 'react'
import { useQuery } from 'graphql-hooks'
import ErrorMessage from './error-message'

export const allPostsQuery = `query allPosts($first: Int!, $skip: Int!) {...}`

export default function PostList() {
  const [skip, setSkip] = useState(0)
  const { loading, error, data, refetch } = useQuery(allPostsQuery, {
    variables: { skip, first: 10 },
    updateData: (prevResult, result) => ({
      ...result,
      allPosts: [...prevResult.allPosts, ...result.allPosts],
    }),
  })

  if (error) return <ErrorMessage message="Error loading posts." />
  if (!data) return <div>Loading</div>

  const { allPosts, _allPostsMeta } = data

  const areMorePosts = allPosts.length < _allPostsMeta.count
}


```

-------------------------------------------------

-> # Slide 6  <-

Step 5

Inside components/post-list.js finally our render

```
import React, { Fragment, useState } from 'react'
import { useQuery } from 'graphql-hooks'
import ErrorMessage from './error-message'

export const allPostsQuery = `query allPosts($first: Int!, $skip: Int!) {...}`

export default function PostList() {
  ...
  return (
    <Fragment>
      <section>
        <ul>
          {allPosts.map((post, index) => (
            <li key={post.id}>
              <div>
                <span>{index + 1}. </span>
                <a href={post.url}>{post.title}</a>
              </div>
            </li>
          ))}
        </ul>
        {areMorePosts ? (
          <button onClick={() => setSkip(skip + 10)}>
            {' '}
            {loading && !data ? 'Loading...' : 'Show More'}{' '}
          </button>
        ) : (
          ''
        )}
      </section>
    </Fragment>
  )
}


```

-------------------------------------------------

-> # Slide 7  <-

Step 6

Inside components/post-list.js finally our render

```
...
  return (
    <Fragment>
      <section>
        ....
        <style jsx>{`
          section {
            padding-bottom: 20px;
          }
          li {
            display: block;
            margin-bottom: 10px;
          }
          div {
            align-items: center;
            display: flex;
          }
          a {
            font-size: 14px;
            margin-right: 10px;
            text-decoration: none;
            padding-bottom: 0;
            border: 0;
          }
          span {
            font-size: 14px;
            margin-right: 5px;
          }
          ul {
            margin: 0;
            padding: 0;
          }
          button:before {
            align-self: center;
            border-style: solid;
            border-width: 6px 4px 0 4px;
            border-color: #ffffff transparent transparent transparent;
            content: '';
            height: 0;
            margin-right: 5px;
            width: 0;
          }
        `}</style>
      </section>
    </Fragment>
  )
}


```

-------------------------------------------------

-> # Slide 8  <-

Step 7

Now lets create todos

* touch components/submit.js 

-------------------------------------------------

-> # Slide 9  <-

Step 8

First lets create our mutation inside components/submit.js 

```
import React from 'react'
import { useMutation } from 'graphql-hooks'

const CREATE_POST = `
mutation createPost($title: String!, $url: String!) {
  createPost(title: $title, url: $url) {
    id
    title
    votes
    url
    createdAt
  }
}`

```

-------------------------------------------------

-> # Slide 10  <-

Step 9

Inside components/submit.js lets create our component with a little css

```
import React from 'react'
import { useMutation } from 'graphql-hooks'

const CREATE_POST = `mutation createPost($title: String!, $url: String!) {...}`

export default function Submit({ onSubmission }) {
  const [createPost, state] = useMutation(CREATE_POST)

  return (
    <form onSubmit={event => handleSubmit(event, onSubmission, createPost)}>
      <h1>Submit</h1>
      <input placeholder="title" name="title" type="text" required />
      <input placeholder="url" name="url" type="url" required />
      <button type="submit">{state.loading ? 'Loading...' : 'Submit'}</button>
      <style jsx>{`
        form {
          border-bottom: 1px solid #ececec;
          padding-bottom: 20px;
          margin-bottom: 20px;
        }
        h1 {
          font-size: 20px;
        }
        input {
          display: block;
          margin-bottom: 10px;
        }
      `}</style>
    </form>
  )
}


```


-------------------------------------------------

-> # Slide 11  <-

Step 10

lets add submit to components/post-list.js

```
import React, { Fragment, useState } from 'react'
import { useQuery } from 'graphql-hooks'
import ErrorMessage from './error-message'
import Submit from './submit'

export const allPostsQuery = `query allPosts($first: Int!, $skip: Int!) {...}`

export default function PostList() {
  ...
  return (
    <Fragment>
      <Submit
        onSubmission={() => {
          refetch({ variables: { skip: 0, first: allPosts.length } })
        }}
      />
      <section>
        ...
      </section>
    </Fragment>
  )
}

```

-------------------------------------------------

-> # Slide 12  <-

Step 11

Now lets update todos with up and down votes

* touch components/post-upvoter.js 


-------------------------------------------------

-> # Slide 13  <-

Step 12

First lets create our mutation inside components/upvoter.js 

```
import React from 'react'
import { useMutation } from 'graphql-hooks'

const UPDATE_POST = `
  mutation updatePost($id: ID!, $votes: Int) {
    updatePost(id: $id, votes: $votes) {
      id
      __typename
      votes
    }
  }
`
```

-------------------------------------------------

-> # Slide 14  <-

Step 13

Inside components/upvoter.js lets create our component

```
import React from 'react'
import { useMutation } from 'graphql-hooks'

const CREATE_POST = ` mutation updatePost($id: ID!, $votes: Int) {...}`

export default function PostUpvoter({ votes, id, onUpdate }) {
  const [updatePost] = useMutation(UPDATE_POST)

  return (
    <button
      onClick={async () => {
        try {
          const result = await updatePost({
            variables: {
              id,
              votes: votes + 1,
            },
          })

          onUpdate && onUpdate(result)
        } catch (e) {
          console.error('error upvoting post', e)
        }
      }}
    >
      {votes}
    </button>
  )
}


```


-------------------------------------------------

-> # Slide 14  <-

Step 13

Inside components/upvoter.js lets add some style

```
import React from 'react'
import { useMutation } from 'graphql-hooks'

const CREATE_POST = ` mutation updatePost($id: ID!, $votes: Int) {...}`

export default function PostUpvoter({ votes, id, onUpdate }) {
  const [updatePost] = useMutation(UPDATE_POST)

  return (
    <button
      onClick={async () => {...
    }}
    >
      {votes}
      <style jsx>{`
        button {
          background-color: transparent;
          border: 1px solid #e4e4e4;
          color: #000;
        }
        button:active {
          background-color: transparent;
        }
        button:before {
          align-self: center;
          border-color: transparent transparent #000000 transparent;
          border-style: solid;
          border-width: 0 4px 6px 4px;
          content: '';
          height: 0;
          margin-right: 5px;
          width: 0;
        }
      `}</style>
    </button>
  )
}


```


-------------------------------------------------

-> # Slide 15  <-

Step 14

lets add upvoter to components/post-list.js

```
import React, { Fragment, useState } from 'react'
import { useQuery } from 'graphql-hooks'
import ErrorMessage from './error-message'
import PostUpvoter from './post-upvoter'
import Submit from './submit'

export const allPostsQuery = `query allPosts($first: Int!, $skip: Int!) {...}`

export default function PostList() {
  ...
  return (
    <Fragment>
      <Submit
        onSubmission={() => {...}
      />
      <section>
        <ul>
          {allPosts.map((post, index) => (
            <li key={post.id}>
              <div>
                <span>{index + 1}. </span>
                <a href={post.url}>{post.title}</a>
                <PostUpvoter
                  id={post.id}
                  votes={post.votes}
                  onUpdate={() => {
                    refetch({ variables: { skip: 0, first: allPosts.length } })
                  }}
                />
              </div>
            </li>
          ))}
        </ul>
        ...
      </section>
    </Fragment>
  )
}

```

-------------------------------------------------

-> # Slide 16  <-

Step 15

Finaly lets update our pages/index.js

```
import App from '../components/app'
import Header from '../components/header'
import PostList from '../components/post-list'

export default () => (
  <App>
    <Header />
    <PostList />
  </App>
)

```

-------------------------------------------------

-> # Slide 17  <-

Step 16

lets run this mf*

* yarn dev
* go to http://localhost:3000/


-------------------------------------------------
-> # Conclusion  <-
