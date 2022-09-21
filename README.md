# How to implement infinite scroll with RTK Query

## Introduction

You have a paged query and you don't know how to make an infinite scroll out of it with RTK Query?
This project will probably come to help you!

## The dilemma

Unfortunately, RTK Query doesn't support infinite scrolling out-of-the-box yet, as stated by [this comparison table](https://redux-toolkit.js.org/rtk-query/comparison#comparing-feature-sets) with other similar tools.

## Possible solutions

I discovered [this issue](https://github.com/reduxjs/redux-toolkit/discussions/1163) on GitHub, which was opened more than one year ago with a lot of possible solutions.
The problem with those is that they're using multiple generated hooks by RTK Query but with different pagination arguments, or they're holding up a local state with the result of different pages, which is not ideal since, under the hood, RTK Query uses Redux, that already has the state of our paginated queries.

## Back to the basics

Remember how everything with Redux was just a matter of reducers, actions and selectors?
Every endpoint you define with the RTK Query API exposes [a set of handy utilities](https://redux-toolkit.js.org/rtk-query/api/created-api/endpoints), including `initiate` and `select`. When using hooks, you will most likely never need to use these directly, as they automatically dispatch them as needed.

These two will help us build our infinite query hook:

- [`initiate`](https://redux-toolkit.js.org/rtk-query/api/created-api/endpoints#initiate) is a plain Redux thunk action creator that you can dispatch to trigger data fetch queries or mutations.

```typescript
import { useState } from 'react'
import { useAppDispatch } from './store/hooks'
import { api } from './services/api'

const App = () => {
	const dispatch = useAppDispatch()
	const [postId, setPostId] = useState(1)

	useEffect(() => {
		// Add a subscription.
		const subscription = dispatch(api.endpoints.getPost.initiate(postId))

		// Return the `unsubscribe` callback to be called in the `useEffect` cleanup step
		return subscription.unsubscribe
	}, [dispatch, postId])

	// ...
}
```

- [`select`](https://redux-toolkit.js.org/rtk-query/api/created-api/endpoints#select) is a function that accepts a cache key argument, and generates a new memoized selector for reading cached data for this endpoint using the given cache key. The generated selector is memoized using Reselect's createSelector.
  RTK Query defines a Symbol named skipToken internally. If skipToken is passed as the query argument to these selectors, the selector will return a default uninitialized state. This can be used to avoid returning a value if a given query is supposed to be disabled.

```typescript
import { useState, useMemo } from 'react'
import { useAppDispatch, useAppSelector } from './store/hooks'
import { api } from './services/api'

const App = () => {
	const dispatch = useAppDispatch()
	const [postId, setPostId] = useState(1)

	// `useMemo` is used to only call `.select()` when required.
	// Each call will create a new selector function instance
	const selectPost = useMemo(
		() => api.endpoints.getPost.select(postId),
		[postId],
	)

	// `result` contains `data`, `isLoading`, etc.
	const result = useAppSelector(selectPost)

	// ...
}
```
