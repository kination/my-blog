---
layout: post
title:  Reloading React project, Javascript to Typescript
date:   2018-08-15
description: 
tags:
- react
- redux
- javascript
- typescript
permalink: react-js-to-ts
---


Typescript is a superset of Javascript that compiles to pure Javascript, like `CoffeeScript` or `Dart`. When I first heard of this, thought as 'Why do we need to learn a new thing, which will create javascript file? Is it better to make javascript directly?'. But now it has been standard development language of `Angular` framework, become official programming language in Google, and it is being spread more widely in various projects. Yeah, there should be a reason.


## Why Typescript?
Javascript has been started as script language, and usage was limited for handling DOM activities, supporting html in web page. Now, client-side application has been very complicated by rising of client-side rendering, and it handles not only rendering, but also flow of data received from server. More than that, NodeJS allowed javascript language to be used in system level. 
Now, there are lots of large-scale application which created by javascript only, and they found it is missing several features necessary to be able to productively write and maintain these system.

TypeScript...
...
...

Also, specification of javascript is being changed, such as ES6, ES7, and it is quite a pain to followup every year. Thanks to Typescript developers, its transpiler is being update to support newest features of javascript, so you don't need to think about this deeply.


## React-redux sample
https://redux.js.org/advanced/exampleredditapi

This is one of famouse asyncrounous web application example. This includes lot of important features for developing actual React web apps such as `action` and `reducer` from `Redux` pattern, middleware usage, and how to handle asyncrounous process like http connection. 

![Screenshot](/assets/post_img/react-js-to-ts/reddit-api-sample.png)

This app has dropdown with value 'reactjs' and 'frontend' included. These are the subreddits, and it will fetch the headlines from API server when selected. It has 3 possible state which can happen.
- Select subreddit
- Request headlines from Reddit API server
- Receive headlines from Reddit API server

This cases has been defined as action.

{% highlight javascript %}
{% raw %}
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
{% endraw %}
{% endhighlight %}

This actions will be changed by status of application. When subreddit has been selected, status is set as `SELECT_SUBREDDIT` first, and will prepare to fetch data from server. Just before fetching, status become `REQUEST_POSTS` and keep it until request is finished. After data has been fetched from server, status changes again to `RECEIVE_POSTS`.

{% highlight javascript %}
{% raw %}
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
{% endraw %}
{% endhighlight %}

Reducers, will define how data in components needs to be changed by received data in actions. Data type is depend on action type such as `REQUEST_POSTS`, `RECEIVE_POSTS`.

{% highlight javascript %}
{% raw %}
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
{% endraw %}
{% endhighlight %}

And function `mapStateToProps` is connecting returning objects from reducer and properties defined in components like below.

{% highlight javascript %}
{% raw %}
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
{% endraw %}
{% endhighlight %}

This is the rough description of what is happening in this example application. For more details about `react+redux`, please look on [guide page](https://redux.js.org/).


## Go as Typescript

![Screenshot](/assets/post_img/react-js-to-ts/js-to-ts.png) 


{% highlight typescript %}
{% raw %}
...
{% endraw %}
{% endhighlight %}






## Reference
- https://www.typescriptlang.org/
- https://github.com/Microsoft/TypeScript-React-Conversion-Guide
- https://blogs.msdn.microsoft.com/somasegar/2012/10/01/typescript-javascript-development-at-application-scale/

