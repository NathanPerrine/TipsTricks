# TipsTricks

# Sveltekit


### Date formatter Util

In the `lib/utils.ts'` file, add the following block
```ts
type DateStyle = Intl.DateTimeFormatOptions['dateStyle']

export function formatDate(date: string, dateStyle: DateStyle = 'medium', locales = 'en-US') {
	const formatter = new Intl.DateTimeFormat(locales, { dateStyle })
  return formatter.format(new Date(date))
}
```

Then in the page it's needed, add the following
```ts
import {formatDate} from '$lib/utils'

formatDate(post.date, undefined, 'en')
```
In this example, we're using the `formatDate` function on the `post.date` date. We don't want to change the date style default of `medium` so we're passing undefined there, and we can change the locale to any valid locale such as `fr-Fr` or `de-DE`


### Local storage with themes
*https://rodneylab.com/using-local-storage-sveltekit/*

Create `themes.ts` in `lib/shared/stores/themes` and add the following code to it.
```ts
import { browser } from "$app/environment";
import { writable } from "svelte/store";

const defaultValue = 'sunset';
const initialValue = browser ? window.localStorage.getItem('theme') ?? defaultValue : defaultValue;

export const theme = writable<string>(initialValue);

theme.subscribe((value) => {
  if (browser) {
    window.localStorage.setItem('theme', value);
    document.querySelector('html')?.setAttribute('data-theme', value)
  }
});

export { theme as default };
```
```ts
const defaultValue = 'sunset'
const initialValue = browser ? window.localStorage.getItem('theme') ?? defaultValue : defaultValue;
```
This code snippet sets the `defaultValue` of for the theme you want to use, in this case we're using `sunset`.
Then we set the `initialValue`. First we check if we're in the browser, if not we default to the `defaultValue` (at the end). Next we check for the `window.localStorage.getItem('theme')` variable, if it's there we set that as our initial value, if not we again fall back to the default.

Place the [theme controller](https://daisyui.com/components/theme-controller/) somewhere in an easily accessible space on your website, such as the navbar. I removed the default `class="theme-controller"` from the input element and made it look like this
```html
<input type="checkbox" on:click={toggleTheme} bind:checked={themeCheck}/>
```
then in the component/page script add the following
```ts
  function toggleTheme() {
    $theme === 'sunset' ? theme.set('winter') : theme.set('sunset')
  }

  // true = sun, false = moon
  let themeCheck : boolean = true
  // sets the theme toggle to the correct position based on theme setting in local storage
  onMount(async () => {
    $theme ==='sunset' ? themeCheck = false : true;
  })
```
This does a few things. First of all when the component mounts, we check the current theme, and use that to determine if we should be in light or dark mode and adjust the displayed icon since we have the `checked` attribute of the input bound to the value of `themeCheck`.
Next, upon clicking theme changer we call the function `toggleTheme`, which simply gets the current theme value, and calls `theme.set()` of the opposite value we want.

#### Stop flicker on page load with theme in local storage
In the `head` section of `app.html`, add the following script block:
```ts
<script lang="ts">
      let selectedTheme = window.localStorage.getItem('theme')
      if (selectedTheme) {
      document.querySelector('html')?.setAttribute('data-theme', selectedTheme)
    }
</script>
```
This will check if the `theme` variable is saved in local storage, and if it is set the `data-theme` tag of the `html`

## Basics

### Typescript api response
To use typescript with an API, start by creating an interface in the `server.ts`:
```ts
interface PersonObj {
  name: string
  tally: number
}
```
Perform your fetch
```ts
  const res = await fetch('https://advent.sveltesociety.dev/data/2023/day-one.json')
  const json = await res.json()
```
map the response to the interface
```ts
  const people = json.map((person: PersonObj) => {
    return {
      name: person.name,
      tally: person.tally
    }
  })
```
Little extra error checking doesn't hurt
```ts
  if (!res.ok || !people){
    throw new Error("Not found")
  }
```
return the typed response
```ts
  return {
    people
  }
```
In the `+page.svelte` file, you can use the data!
```ts
{#each data.people as person, i }
  {person.name} {person.tally}
{/each}
```


