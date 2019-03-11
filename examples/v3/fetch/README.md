# Fetch with Nuxt.js 3

Nuxt.js introduces a new hook called `fetch` in any of your Vue components. 

> **INFO** 💡<br>
> Nuxt.js also add the [fetch polyfill](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) so you can easily make HTTP request inside your components.

> :warning: WIP<br>
> Working only in Nuxt development you can see the [draft pull request](https://github.com/nuxt/nuxt.js/pull/5118)<br>
> If you want to run it locally, please clone Nuxt repo, `git checkout feat/async-data`, install the dependencies and run `yarn nuxt examples/v3/fetch/`

## When to use fetch?

Every time you need to fetch **asynchronous** data. `fetch` is called on server-side when rendering the route, and on client-side when navigating.

It also introduces `$isFetching` at the component level, letting you display a placeholder while  `fetch` is being called *on client-side*.

> **INFO** 💡<br>
> You can tell Nuxt to call `fetch` only on client-side by specifing `fetchOnServer: false`.<br>
> With it, `$isFetching` will be `true` when server-rendering the component.

## Example

Let's have a blog with our home page listing our posts:

`pages/index.vue`
```vue
<template>
  <div>
    <h1>Blog posts</h1>
    <p v-if="$isFetching">Fetching posts...</p>
    <ul v-else>
      <li v-for="post of posts" :key="post.id">
        <n-link :to="`/posts/${post.id}`">{{ post.title }}</n-link>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data () {
    return {
      posts: []
    }
  },
  async fetch () {
    this.posts = await fetch('https://jsonplaceholder.typicode.com/posts').then((res) => res.json())
  }
}
</script>
```

If you go directly to [http://localhost:3000/](http://localhost:3000/), you will see directly the full list of posts which has been server-rendered.

<img width="669" alt="Screenshot 2019-03-11 at 23 04 57" src="https://user-images.githubusercontent.com/904724/54161334-1f9e8400-4452-11e9-97bf-996a6e69d9db.png">


> **INFO** 💡<br>
> Nuxt will smartly detect what data you mutated inside `fetch` and optimised the JSON included in the returned HTML.

Now, let's add `pages/posts/_id.vue` page to display a post on `/posts/:id`.

`pages/posts/_id.vue`
```vue
<template>
  <div v-if="$isFetching">Fetching post #{{$route.params.id}}...</div>
  <div v-else>
    <h1>{{ post.title }}</h1>
    <pre>{{ post.body }}</pre>
    <p><n-link to="/">Back to posts</n-link></p>
  </div>
</template>
    
<script>
export default {
  data () {
    return {
      post: {}
    }
  },
  async fetch() {
    this.post = await fetch(`https://jsonplaceholder.typicode.com/posts/${this.$route.params.id}`).then((res) => res.json())
  }
}
</script>
```

When navigating, you should now see "Loading post #..." on client-side, and no loading when refreshing a post.

<img width="669" alt="fetch-nuxt3" src="https://user-images.githubusercontent.com/904724/54161844-d3544380-4453-11e9-9586-7428597db40e.gif">

> **INFO** 💡<br>
> In the component having `fetch` hook, you will also have access to `this.$fetch()` to re-call `fetch` hook (`$isFetching` will become `true` again).