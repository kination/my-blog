---
layout: post
title:  Reloading React project, JavaScript to TypeScript
date:   2018-09-29
description: 
tags:
- react
- redux
- javascript
- typescript
permalink: react-js-to-ts
---


`TypeScript` is a superset of JavaScript that compiles to pure JavaScript, like `CoffeeScript` or `Dart`. When I first heard of this, thought as 'Why do we need to learn a new thing, which will create JavaScript file? Is it better to make JavaScript directly?'. But now it has been standard development language of `Angular` framework, become official programming language in Google, and it is being spread more widely in various projects. Yeah, there should be a reason.


## Why TypeScript?
JavaScript has been started as script language, and usage was limited for handling DOM activities, supporting html in web page. Now, client-side application has been very complicated by rising of client-side rendering, and it handles not only rendering, but also flow of data received from server. More than that, NodeJS allowed JavaScript language to be used in system level. 
Now, there are lots of large-scale application which created by JavaScript only, and they found it is missing several features necessary to be able to productively write and maintain these system.

TypeScript has lot of similarity with JavaScript, and make it as `type safe`. It means it can avoid lots of problem caused by unmatching type between values, and catch estimated errors in developing. It helps making stable large-scale web services, and this make TypeScript as one of the popular trend.

Also, specification of JavaScript is being changed, such as ES6, ES7, and it is quite a pain to followup every year. Thanks to TypeScript developers, its transpiler is being update to support newest features of JavaScript, so you don't need to think about this deeply.

It has been a few week since I started to look on TypeScript. I was on a work refactoring legacy service based on `React` to new UI, and one of joined engineer suggested of this. So before working on, I plan to make simple TypeScript based React application.


## React-redux sample
<https://redux.js.org/advanced/exampleredditapi>

This is one of famouse asyncrounous web application example. This includes lot of important features for developing actual React web apps such as `action` and `reducer` from `Redux` pattern, middleware usage, and how to handle asyncrounous process like http connection. 

![Screenshot](/assets/post_img/react-js-to-ts/reddit-api-sample.png)

This app has dropdown with value 'reactjs' and 'frontend' included. These are the subreddits, and it will fetch the headlines from API server when selected. It has 3 possible state which can happen.
- Select subreddit
- Request headlines from Reddit API server
- Receive headlines from Reddit API server

This cases has been defined as action.

```javascript
export const REQUEST_POSTS = 'REQUEST_POSTS'
export const RECEIVE_POSTS = 'RECEIVE_POSTS'
export const SELECT_SUBREDDIT = 'SELECT_SUBREDDIT'

export function selectSubreddit(subreddit) {
  return {
    type: SELECT_SUBREDDIT,
    subreddit
  }
}

function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}

function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}
```

This actions will be changed by status of application. When subreddit has been selected, status is set as `SELECT_SUBREDDIT` first, and will prepare to fetch data from server. Just before fetching, status become `REQUEST_POSTS` and keep it until request is finished. After data has been fetched from server, status changes again to `RECEIVE_POSTS`.

```javascript
class AsyncApp extends Component {
    ...
    handleChange(nextSubreddit) {
        this.props.dispatch(selectSubreddit(nextSubreddit))
        this.props.dispatch(fetchPosts(nextSubreddit))
    }
    ...
}

function fetchPosts(subreddit) {
  return dispatch => {
    dispatch(requestPosts(subreddit))   // REQUEST_POSTS
    return fetch(`https://www.reddit.com/r/${subreddit}.json`)
      .then(response => response.json())
      .then(json => dispatch(receivePosts(subreddit, json)))    // RECEIVE_POSTS
  }
}
```

Reducers, will define how data in components needs to be changed by received data in actions. Data type is depend on action type such as `REQUEST_POSTS`, `RECEIVE_POSTS`.

```javascript
function posts(state = {},action) {
  switch (action.type) {
    case REQUEST_POSTS:
      return Object.assign({}, state, {
        isFetching: true
      })
    case RECEIVE_POSTS:
      return Object.assign({}, state, {
        isFetching: false,
        items: action.posts,
        lastUpdated: action.receivedAt
      })
    default:
      return state
  }
}
```

And function `mapStateToProps` is connecting returning objects from reducer and properties defined in components like below.

```javascript
function mapStateToProps(state) {
  const { selectedSubreddit, postsBySubreddit } = state
  const { isFetching, lastUpdated, items: posts} = postsBySubreddit[selectedSubreddit] || {
    isFetching: true,
    items: []
  }

  return {
    selectedSubreddit,
    posts,
    isFetching,
    lastUpdated
  }
}

export default connect(mapStateToProps)(AsyncApp)
```

This is the rough description of what is happening in this example application. For more details about `react+redux`, please look on [guide page](https://redux.js.org/).


## Go on to TypeScript

![Screenshot](/assets/post_img/react-js-to-ts/js-to-ts.png)

Now I'll go on making similar service based on TypeScript. I'll focus on only differences compare to JS version here. For quick work, I'll make basic template with `create-react-app`. It offers option to generate skeleton as TypeScript.

```
$ create-react-app my-app --scripts-version=react-scripts-ts
```

Let's look on component first.

```typescript
...
interface IProps {
  selectedSubreddit: string,  
  lastUpdated: number,
  posts: [],
  handleChange: () => void
}

const mapStateToProps = (appState: AppState) => ({
  selectedSubreddit: appState.subreddit,
  lastUpdated: appState.posts.lastUpdated,
  posts: appState.posts.posts
})

const mapDispatchToProps = (dispatch: any) => ({
  handleChange: (subreddit: string) => {
    dispatch(selectSubreddit(subreddit))
    dispatch(fetchPosts(subreddit))
  }
})

class AsyncApp extends React.Component<IProps> {
  render() {
    const { selectedSubreddit, posts, lastUpdated, handleChange } = this.props
...
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(AsyncApp)
```

I defeind properties which will be used in this component as `interface`. In JavaScript, it defined value with `PropTypes`, but because TypeScript can define value types('string', 'numbers'), you don't need this here.
And when making `state`, `dispatch` to properties, it needs to define the type of parameter.

```typescript
export type AppState = {
  subreddit: string,
  posts: PostState,
}

export type PostState = {
  posts: [],
  lastUpdated: number
}
```

'AppState' is a defined type which includes all kinds of state values managed in application. It stores 'subreddit' as value of selected subreddit, and 'posts' for post data.

To define actions and reducers, it also need to define format of returning values as interface, and param types.

```typescript
export interface IRequestPosts {
  type: ActionTypeStates.REQUEST_POSTS,
  subreddit: string
}
...

export function selectSubreddit(subreddit: string): ISelectSubreddit {
  return {
    type: ActionTypeStates.SELECT_SUBREDDIT,
    subreddit
  }
}
...
```

I updated full sources on <https://github.com/kination/my-scratch/tree/master/web/react-redux-ts>


## So, is it good?
Good thing is, you can catch lot of expected error while typing, not after running. For example, if I change type value like below:

```typescript
export type AppState = {
  subreddit: string,
  posts: PostState,
}

export type PostState = {
  posts: [],
//  lastUpdated: number
}
```

the code editor(I'm using VS code...it's good!)...

![Screenshot](/assets/post_img/react-js-to-ts/type-error.png)

...will find value is missing and show error immediately. This means you don't need to worry about problems related with type when it compiled well. The more application becomes large, more state values will be included in service. By avoiding this kind of issue will reduce lots of latent bugs.

But bad thing is, because it requires type definitions strictly, you have to type more than JavaScript for same process. So the thing you are planning to make are just a simple scratch, this could only make you more tired. Thankfully, there are lots of modules to reduce typing for type-safe React application, so you could get more help there.


## Reference
- <https://www.typescriptlang.org/>
- <https://github.com/Microsoft/TypeScript-React-Conversion-Guide>
- <https://blogs.msdn.microsoft.com/somasegar/2012/10/01/typescript-javascript-development-at-application-scale/>
- <https://basarat.gitbooks.io/typescript/>
- <https://github.com/Lemoncode/react-typescript-samples>
- <https://redux.js.org>
