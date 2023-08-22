# Using props with reserved names

I have a component where I want to pass in a `class` property, so I can set an element like `<a href="..." {class}>`. But doing this will break:

```svelte
<script>
  export let class = "";
</script>
```

The word `class` is reserved in JavaScript. On HTML elements, the property is called `className`. Turns out Svelte can do that, too. This [StackOverflow answer](https://stackoverflow.com/questions/55957386/use-reserved-word-as-prop-name) shows how to do it.

My component did it roughly like this:

```svelte
<script>
  let className = "";
  export { className as class };
</script>

<div class={className}>
    Content here
</div>
```
