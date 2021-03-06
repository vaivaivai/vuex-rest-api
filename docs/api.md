# API
The following sections explain the API of the `Vapi` class.

### `constructor(options:Object):Vapi`

Creates a new `Vapi` instance and returns it.

```js
const vapi = new Vapi(options)
```

The parameter `options` consists of the following properties:

#### `# axios`
- **Type**: `axios` (instance)  
- **Default**: `axios` (instance)  
- **Usage**: The axios instance to use for the requests. This is pretty useful if you use a package like websanova/vue-auth which sets automatically the Authorization header. So you don't need to care. If you don't pass an instance, it will use the global axios instance.  

#### `# baseURL`
- **Type**: `string`
- **Usage**: The API's base URL without a specific endpoint's path. It's usage is optional. If you don't set it, it will use the base URL of the axios instance. Please note that `baseURL` has a higher priority than the baseURL set in the passed axios instance. You can also set base URL in the request config when you add an action. The priority is as following:

  `baseURL > axios instance base URL > request config base URL`
```js
{
  baseURL: "https://jsonplaceholder.typicode.com"
}
```

#### `# queryParams`  
- **Type**: `boolean` 
- **Default**: If you don't set a property's default value the value is `null`.  
- **Usage**: If you want to append the params to the request URL, set this property to true. You can also set this option in every action you need it if you don't need it for every action.
```js
{
  queryParams: true
}
```

#### `# state`
- **Type**: `Object`
- **Default**: Every property will default to `null`.
- **Usage**: The default state of your properties.  
```js
{
  state: {
    post: null, // this is unnecessary (default is null)
    posts: []
  }
}
```
Sets post to `null` and posts to an empty array.

### `add(options):Vapi`

Adds an action to access an API endpoint and returns the `Vapi` instance.

The parameter `options` consists of the following properties:

#### `# action` (required)
- **Type**: `string`  
- **Usage**: The name of the action.
```js
{
  action: "getPosts"
}
```

#### `# method`
- **Type**: `string`  
- **Default**: `"get"`  
- **Usage**: The HTTP method to request the API. Following HTTP Methods are allowed at the moment:
  - get
  - delete
  - head
  - post
  - put
  - patch

```js
{
  method: "get"
}
```

##### shorthand syntax
You can also use the http method instead of `add` to omit to set the `options.method` like this. This works with get, delete, post, put and patch:
```js
// regular way
vrap.add({
  method: "delete"
  // other options...  
})

//shorthand
vrap.delete({
  // other options...
})
```

#### `# property`
- **Type**: `string`
- **Default**: `null`
- **Usage**: The property of the state which should be automatically changed if the resolve is successfully.
```js
{
  property: "posts"
}
```

##### When to set `property` in spite of it's optionality
Sometimes you have to set the state by yourself. In that case you may consider to avoid setting `property`. Please consider the following consequences:

- `state.pending.<property name>` and `state.error.<property name>` won't be set. So you **can't** check these properties to see if the request is still pending or failed. Nevertheless you can still set the `onError` property to react to the error case.
- Because the property's name is unknown *vuex-rest-api* can't set the initial state for this property. You have to set the initial state by yourself.
- In the case of successful requests there will be nothing done with the payload. You have to set the `onSuccess` method to set the state (with the payload) by yourself.

#### `# path` (required)
- **Type**: `Function|string`  
- **Usage**: This property can either be a function or a path describing the rest of the API address (without the base URL).

##### Usage with a function
Here we pass a custom path function. This is necessary, because we also need to pass an id. Please note that *id* is passed in an object. This is necessary because you could pass multiple arguments.
```js
{
  path: ({id}) => `/post/${id}`
}
```

##### Usage with a string
Maybe the API endpoint needs no parameters. Then you can use a string like this:
```js
{
  path: "/posts"
}
```

#### `# headers`
- **Type**: `Function|Object`  
- **Usage**: This property allows to provide dynamic headers for every request. It can either be a function or an object.

##### Usage with a function
Here we pass a custom headers function. This allows us to change the extra headers for each request. The data has to be passed over the `params` bag.

> Please note that headers provided via this property will override it's counterpart you maybe set via `requestConfig.headers`.

```js
// setting the header
{
  headers: ({foo, bar}) => ({
      "FOO": foo,
      "BAR": bar
    })
}

// calling the mapped action and passing the data over the params object
this.getPosts({
  params: { 
    foo: "foo-header",
    bar: "bar-header"
  } 
})
```

##### Usage with a string
If the headers don't have to be evaluted on every request, just pass them via an object. Alternatively you could also set the headers via the `requestConfig.headers` property.
```js
{
  headers: {
    "FOO": "foo-header",
    "BAR": "bar-header"
  }
}
```

#### `# beforeRequest`
- **Type**: `Function`  
- **Default**: `undefined`  
- **Usage**: This function will be called before resolving the action, so you can update the state optimistically. If you need to handle the rejected/resolved response just use the `onSuccess` and `onError` functions.
```js
{
  beforeRequest: (state, { params, data }) => {
    state.posts = state.posts.filter(post => post.id !== params.id)
  }
}
```

#### `# onSuccess`
- **Type**: `Function`
- **Default**: `undefined`
- **Usage**: This function will be called after successfully resolving the action. If you define this property, only the corresponding pending and error properties will be set, but not `state[property]`.
```js
{
  onSuccess: (state, payload, axios) => {
    // if you set the onSuccess function you have to set the state manually
    state.posts = payload.data
    state.post = payload.data[0]
  }
}
```

#### `# onError`
- **Type**: `Function`  
- **Default**: `undefined`  
- **Usage**: This function will be called if the action request fails. If you define this property, only the corresponding `pending` and `error` properties of the set `property` will be set, but not `state[property]`.
```js
{
  onError: (state, error, axios) => {
    Toast.showError(`Oops, there was following error: ${error}`)

    // if you set the onError function you have to set the state manually
    state.post = null
  }
}
```

#### `# requestConfig`
- **Type**: `Object`  
- **Default**: `{}`  
- **Usage**: An [`axios.requestConfig`](https://github.com/mzabriskie/axios#request-config) object. Please note that the passed HTTP method (see `options.method` above) won't be changed.
```js
{
  requestConfig: {
    //excerpt from (https://github.com/mzabriskie/axios#request-config)
    // `paramsSerializer` is an optional function in charge of serializing `params`
    // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
    paramsSerializer: function(params) {
      return Qs.stringify(params, {arrayFormat: 'brackets'})
    },
  }
}
```

#### `# queryParams`
- **Type**: `boolean`  
- **Default**: `undefined`  
- **Usage**: If you want to append the params to the request URL, set this property to true.
```js
{
  queryParams: true
}
```

### `getStore(options):Object`

Creates an object you can pass to Vuex to add a store.

The parameter `options` consists of the following properties:

#### `# createStateFn`
- **Type**: `boolean`  
- **Default**: `false`
- **Usage**: Decides if the state should be returned as a function or an object. This option has to be changed if you want to share the state of your created store. Read chapter *module reuse* in the [Vuex documentation](https://vuex.vuejs.org/en/modules.html) for more details.
```js
{
  createStateFn: false
}
```

The returned object looks like this if you would call it with the settings of the example:
```js
{
  state: {
    pending: {
      posts: false,
      post:  false
    },
    error: {
      posts: null,
      post:  null
    },
    posts:   [],
    post:    null
  }),
  mutations: {
    LIST_POSTS:            Function,
    LIST_POSTS_SUCCEEDED:  Function,
    LIST_POSTS_FAILED:     Function,
    GET_POST:              Function,
    GET_POST_SUCCEEDED:    Function,
    GET_POST_FAILED:       Function,
    UPDATE_POST:           Function,
    UPDATE_POST_SUCCEEDED: Function,
    UPDATE_POST_FAILED:    Function
  },

  //every action's signatures is function(params, data)
  actions: {
    listPosts:  Function,
    getPost:    Function,
    updatePost: Function
  }
}
```

As you can see, it just created the store for us. No more, no less.