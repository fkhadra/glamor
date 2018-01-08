<!-- TOC -->
# Glamor
---

# Table of Contents
- [Api](#api)
  - [css](#css)
  - [css.global | css.insert](#cssglobal--cssinsert)
  - [css.fontFace](#cssfontface)
  - [css.keyframes](#csskeyframes)
  - [simulate](#simulate)
  - [speedy](#speedy)
  - [flush()](#flush)
- [Helpers](#helpers)
  - [style](#style)
  - [pseudo](#pseudo)
  - [select | $](#select--)
  - [parent](#parent)
  - [compose | merge](#compose--merge)
  - [media](#media)
- [Moving from css to glamor](#moving-from-css-to-glamor)
  - [Apply a style to an element](#apply-a-style-to-an-element)
  - [Pseudoclasses](#pseudoclasses)
  - [Multiple styles to an element](#multiple-styles-to-an-element)
  - [Child selectors](#child-selectors)
  - [parent selectors](#parent-selectors)
  - [Siblings](#siblings)
  - [Media queries](#media-queries)
  - [Global css rule](#global-css-rule)
  - [Fallback values](#fallback-values)
  - [Font-face](#font-face)
  - [Animations](#animations)
  - [css reset / normalize](#css-reset--normalize)
  - [Combined selectors](#combined-selectors)
- [Aphrodite shim](#aphrodite-shim)
- [Composing / Modularity](#composing--modularity)
- [Conditional styles](#conditional-styles)
- [Glamor createElement](#glamor-createelement)
  - [Usage](#usage)
  - [Integration with typescript](#integration-with-typescript)
- [CSS as values](#css-as-values)
- [Use real css [experimental, buggy]](#use-real-css-experimental-buggy)
- [What happens when I call css(...rules)?](#what-happens-when-i-call-cssrules)
- [jsxstyle shim](#jsxstyle-shim)
- [Performance considerations](#performance-considerations)
- [Plugin system](#plugin-system)
- [Selectors](#selectors)
- [server side rendering](#server-side-rendering)
- [styled-components [experimental, buggy]](#styled-components-experimental-buggy)
- [StyleSheet](#stylesheet)
- [Theme](#theme)
- [multi-index caches with WeakMaps](#multi-index-caches-with-weakmaps)
- [Why I like glamor](#why-i-like-glamor)

<!-- /TOC -->

## Api
---

### css

In glamor, css rules are treated as values. The `css` function lets you define these values. 

This is a 'rule'
```css
.abc { color: red }
```

These rules can be grouped to form more rules
```css
/* this is a combined rule for .abc */
.abc { color: red }
.abc:hover { color: blue }
html.ie9 .abc span.title { font-weight: bold }
@media(min-width: 300px) { 
  .abc { font-size: 20 }
} 
```

We can write this in javascript
```jsx
import { css } from 'glamor'

let abc = css({
  color: 'red',
  ':hover': { color: 'blue' },  
  'html.ie9 & span.title': { fontWeight: 'bold' }, 
  '@media(min-width: 300px)': { fontSize: 20 }
})
```


Combine multiple rules
```jsx
let combined = css(
  abc, 
  { color: 'blue', ':after': { content: '"..."'} }, 
  { ':hover': { textDecoration: 'underline' } },
  someOtherRule,
  // ...more
)
```

- glamor will make sure that rules are merged in the correct order, 
managing nesting and precedence for you. [(Learn more about selectors and nesting)](https://github.com/threepointone/glamor/blob/master/docs/selectors.md) 
- There are a number of helpers to simplify creating rules. [See the full list here](https://github.com/threepointone/glamor/blob/master/docs/helpers.md)
- in dev mode, adding a `label` string prop will reflect its value in devtools. useful when debugging.


You can use these rules with elements 
```jsx
// as classes
<div className={abc}>
  hello world
</div>

// or props
<div {...abc}>
  hello world
</div>
```


Define fallback values for browsers that don't support features
```jsx
let gray = css({
  color: ['#ccc', 'rgba(0, 0, 0, 0.5)']
})
```
is equivalent to
```css
.gray {
  color: #ccc;
  color: rgba(0, 0, 0, 0.5)
}
```


---


###  css.global | css.insert

append a raw css rule at most once to the stylesheet. the ultimate escape hatch.

```jsx
// these don't 'return' anything, 
// can't nest selectors, do
// *cannot* be combined with other rules.

css.global('html, body', { padding: 0 })
// or if prefer raw css and/or need media queries etc 
// send one rule at a time 
css.insert('html, body { padding: 0 }')
css.insert('@media print {...}')
```

*Note*: You have to insert rules one by one! The following will not work in production mode:
```
css.insert('html, body { padding: 0 } strong{ padding: 10p }')
```


### css.fontFace

loads the given font-face at most once into the document, returns the font family name

```jsx
let family = css.fontFace({
  fontFamily: 'Open Sans',
  fontStyle: 'normal',
  fontWeight: 400,
  src: "local('Open Sans'), local('OpenSans'), url(https://fonts.gstatic.com/s/...ff2')",
  unicodeRange: "U+0000-00FF, U+0131, ... U+E0FF, U+EFFD, U+F000"
})
// ...
<div {...style({ fontFamily: family })}>
  no serifs!
</div>
```

for anything more complicated, use something like [typography.js](https://kyleamathews.github.io/typography.js/)

---

### css.keyframes

adds animation keyframes into the document, with an optional name.

```jsx
let bounce = css.keyframes('bounce', { // optional name
  '0%': { transform: 'scale(0.1)', opacity: 0 },
  '60%': { transform: 'scale(1.2)', opacity: 1 },
  '100%': { transform: 'scale(1)' }
})
// ...
<div {...css({
  animation: `${bounce} 2s`,
  width: 50, height: 50,
  backgroundColor: 'red'
})}>
  bounce!
</div>
```

use sparingly! for granular control, use javascript and pencil and paper.

---


### simulate

![hover](http://i.imgur.com/mW7J8kg.gif)

in development, lets you trigger any pseudoclass on an element

---


### speedy

toggle speedy mode. By default, this is off when `NODE_ENV` is `development`, and on when `production`.


### flush()

TO FILL

## Helpers
---

### style 

`style(props)`

Defines a `rule` with the given key-value pairs. returns an object (of shape `{'data-css-<id>': ''}`),
to be added to an element's attributes. This is *not* the same as element's `style`,
and doesn't interfere with the element's `className` / `class`

```jsx
<div {...style({ backgroundColor: '#ccc', borderRadius: 10 })}>
  <a {...style({ label: 'blueText', color: 'blue' })} href='github.com'>
    click me
  </a>
</div>
```

---

### pseudo

`<pseudo>(props)`

where `<pseudo>` is one of :
```
active      any           checked     _default    disabled    empty
enabled     first         firstChild  firstOfType fullscreen  focus
hover       indeterminate inRange     invalid     lastChild   lastOfType
left        link          onlyChild   onlyOfType  optional    outOfRange
readOnly    readWrite     required    right root  scope       target
valid       visited
```

defines a `rule` for the given pseudoclass selector

```jsx
<div {...hover({ backgroundColor: '#ccc', display: 'block'})}>
  <input
    {...style({ color: 'gray', fontSize: 12 })}
    {...focus({ color: 'black' })}
    {...hover({ fontSize: 16 })} />
</div>
```

---

`<pseudo>(param, props)`

where `<pseudo>` is one of :
```
dir  lang  not  nthChild  nthLastChild  nthLastOfType  nthOfType
```

like the above, but parameterized with a number / string

```jsx
dir('ltr', props), dir('rtl', props)
lang('en', props), lang('fr', props), lang('hi', props) /* etc... */
not(/* selector */, props)
nthChild(2, props), nthChild('3n-1', props), nthChild('even', props) /* etc... */
nthLastChild(/* expression */, props)
nthLastOfType(/* expression */, props)
nthOfType(/* expression */, props)
```

---

`<pseudo>(props)`

where `<pseudo>` is one of
```
after  before  firstLetter  firstLine  selection  backdrop  placeholder
```

similar to the above, but for pseudo elements.

```jsx
<div {...before({ content: '"hello "' })}>
  world!
</div>
// note the quotes for `content`'s value
```

---

### select | $

`select(selector, props)` / `$(selector, props)`

an escape hatch to define styles for arbitrary css selectors. your selector is appended 
directly to the css rule, letting you define 'whatever' you want. 

```jsx
<div {...$(':hover ul li:nth-child(even)', { color: 'red' })}>
  <ul>
    <li>one</li>
    <li>two - red!</li>
    <li>three</li>
  </ul>
</div>
```

(nb1: don't forget to add a leading space for 'child' selectors. eg - `$(' .item', {...})`. 
(nb2: `simulate()` does not work on these selectors yet.)

---

### parent

`parent(selector, style)`

an escape hatch to target elements based on it's parent 

```jsx
<div {...parent('.no-js', { backgroundColor: '#ccc' })}> 
  this is gray when js is disabled   
</div>

TODO - pseudo selectors for the same
```
---

### compose | merge

`compose(...rules)` / `merge(...rules)`

combine rules, with latter styles taking precedence over previous ones.

```jsx
<div {...
  compose(
    style(props),
    hover(props),
    { color: 'red' },
    hover(props)) }>
      mix it up!
</div>
```

---

### media

`media(query, ...rules)`

media queries!

```jsx
<div {...media('(min-width: 500px) and (orientation: landscape)', 
            { color: 'blue' }, hover({ color: 'red' }))}>
  resize away
</div>
```

also included are some presets 

`presets.mobile` - `(min-width: 400px)`
`presets.phablet` - `(min-width: 550px)`
`presets.tablet` - `(min-width: 750px)`
`presets.desktop` - `(min-width: 1000px)`
`presets.hd` - `(min-width: 1200px)`

and use as -
```jsx
media(presets.tablet, {...})
```

---


## Moving from css to glamor

### Apply a style to an element 
--- 

css
```css
.box { color: red; }
```
```html
<div class='box'> 
  this is a nice box. 
<div>
```

glamor 
```jsx 
import { css } from 'glamor'

let box = css({ color: 'red' })
// ...
<div {...box}>
  this is a nice box. 
</div>

// or 
<div className={box}>
  this is a nice box. 
</div>


```

### Pseudoclasses
---

css
```css
.box:hover { color: blue; }
```

glamor
```jsx
import { css } from 'glamor'

let boxHover = css({ 
  ':hover': {
    color: 'blue' 
  } 
})

// or 

import { hover } from 'glamor'
let boxHover = hover({ color: 'blue' })
```


### Multiple styles to an element
---

css
```html
<div class="bold myClass"/>
```

glamor 
```jsx
import { css } from 'glamor'

<div {...bold} {...myClass} />

// or, unlike css, to maintain precendence order 

<div {...css(bold, myClass)} />

// also works with classes

<div className={css(bold, myClass)} />
```

[(more examples for composing rules)](https://github.com/threepointone/glamor/blob/master/src/ous.js)


### Child selectors
---

css
```css
#box { display: block; }
.bold { font-weight: bold; }
.one  { color: blue; }
#box:hover .two { color: red; }
```
```html
<div id="box">
  <div class="one bold">is blue-bold!</div>
  <div class="two">hover red!</div>
</div>
```

glamor 
```jsx
import { css } from 'glamor'

let box = css({
  display: 'block',
  '& .bold': { fontWeight: 'bold' },
  '& .one': { color: 'blue' },
  ':hover .two': { color: 'red' }
})

// or 
import { css, select as $ } from 'glamor'
let box = css(
  { display: 'block' },
  $('& .bold', { fontWeight: 'bold' }),
  $('& .one', { color: 'blue' }),
  $(':hover .two', { color: 'red' }),  
)


<div {...box}>
  <div className="one bold">is blue-bold!</div>
  <div className="two">hover red!</div>
</div>

```

It's also possible to use Glamor's generated `classNames` for nested styles:

glamor 
```jsx
import { css } from 'glamor'

const child = css({
  // Child styles ...
});
const parent = css({
  // Parent styles ...

  [`& .${child}`]: {
    // Nested child styles
  }
});


<div className={parent}>
  <div className={child}>...</div>
</div>

```

your components could also accept props to be merged into the component 

```jsx
let defaultStyle = { color: 'blue' }
export const Button = ({ css, children, ...props }) => 
  <button {...props} {merge(defaultStyle, css)}>
    {children}
  </button>

<Button css={hover({ color: 'red' })} />
```

[todo - vars and themes]

### parent selectors 
---

css
```css
.no-js .something #box { color: gray; }
```

glamor 
```jsx
import {css, parent} from 'glamor'

let box = css({
  '.no-js .something &': { color: 'gray' }
})

// or 

import { css, parent } from 'glamor'

let box = parent('.no-js .something', 
  { color: 'gray' })

<div {...box} /> 
```


### Siblings
---

use `+` and `~` selectors

css
```css
.list li:first-of-type + li {
  color: red
}
```
```html
<ul class='list'>
  <li>one</li>
  <li>two - red!</li>
  <li>three</li>
</ul>
```

glamor
```jsx
import {select as $} from 'glamor'

let ul = $('& li:first-of-type + li', {
  color: 'red'
})

// ...

<ul {...ul}>
  <li>one</li>
  <li>two - red!</li>
  <li>three</li>  
</ul>
```


### Media queries
---

css
```css
.box {
  position: 'relative',
  width: '100%',
  maxWidth: 960,
  margin: '0 auto',
  padding: '0 20px',
  boxSizing: 'border-box'
}

.box:after {
  content: '""',
  display: 'table',
  clear: 'both'
}

@media (min-width: 400px) {
  .box {
    width: 85%;
    padding: 0
  }
}

@media (min-width: 550px) {
  .box:nth-child(2n) {
    width: 80%
  }
}
```

glamor
```jsx
import {css, after, media, nthChild} from 'glamor'

const container = css(
  {
    position: 'relative',
    width: '100%',
    maxWidth: 960,
    margin: '0 auto',
    padding: '0 20px',
    boxSizing: 'border-box'
  },
  after({
    content: '""',
    display: 'table',
    clear: 'both'
  }),
  { 
    '@media(min-width: 400px)': {
      width: '85%',
      padding: 0
    }
  },
  // or use helpers 
  media('(min-width: 550px)', nthChild('2n', {
    width: '80%'    
  }))  
)
```


### Global css rule
---

css 
```css
html, body { padding: 0 }
```

glamor 
```jsx
import { css } from 'glamor'

css.global('html, body',  { padding: 0 })
// or as raw css
css.insert('html, body { padding: 0 }')
```

### Fallback values
---

css
```css
.box {
  display: flex;
  display: block;
}
```

glamor 
```jsx
let box = style({
  display: ['flex', 'block']
})
```

### Font-face
---

[todo]

### Animations
---

css 
```css
@keyframes bounce { 
  0%: { transform: scale(0.1); opacity: 0; }
  60%: { transform: scale(1.2); opacity: 1; }
  100%: { transform: scale(1); }
}

.box {
  animation: bounce 2s;
  width: 50px;
  height: 50px;
  backgroundColor: 'red';
}
```

glamor
```jsx
let bounce = css.keyframes({ 
  '0%': { transform: 'scale(0.1)', opacity: 0 },
  '60%': { transform: 'scale(1.2)', opacity: 1 },
  '100%': { transform: 'scale(1)' }
})

let box = css({
  animation: `${bounce} 2s`,
  width: 50,
  height: 50,
  backgroundColor: 'red'
}) 
```

### css reset / normalize
---

```jsx
import `glamor/reset`
```

### Combined selectors
---

css
```css
.f1 { font-size: 1rem };
.red { background: red };
.f1.red { font-size: 2rem };
```
```html
<div class="f1 red">
  I'm red and 2rem!
</div>
```

glamor
```js
// With regular selectors (not ideal for namespacing/isolation)
const rule = style({
  '&.f1' : { fontSize: '1rem' },
  '&.red' : { background: 'red' },
  '&.f1.red' : { fontSize: '2rem' }
 })

<div class={`${rule} f1 red`}></div>


// Or use merge to output a single css selector
const f1 = ({ fontSize: '1rem' });
const red = ({ background: 'red' });
const f1Red = merge(f1, red, { fontSize: '2rem' });

<div class={rule}></div>


// Or for a more traditional css approach...
const f1 = ({ fontSize: '1rem' });
const red = ({ background: 'red' });
const f1Red = style({
  [`&.${f1}.${red}`]: { fontSize: '2rem' }
});

// ...but you still have to use all three selectors
<div class={`${f1} ${red} ${f1red}`}></div>
```


## Aphrodite shim
--- 

glamor ships with a aphrodite-like StyleSheet api. 

usage - 
```jsx
import { StyleSheet, css } from 'glamor/aphrodite'

let styles = StyleSheet.create({
  red: { color: 'red' },
  hoverBlue: {
    ':hover': { color: 'blue' }
  }
})

// ...
<div className={css(styles.red, style.hoverBlue)}>
  this is red, and turns blue on hover
</div>  
```


a full example is [available](https://github.com/threepointone/glamor/blob/master/examples/aphrodite.js).

## Composing / Modularity
---

while it's tempting to add some form of a `Stylesheet` construct, we'd
rather defer to the developer's preference. In general, we recommed using
simple objects, functions, and components to create abstractions. You can
also lean on `compose(...styles)` for combining rules. Some examples -

```jsx
// name your rules with vars/consts/whatevs
let container = style({ color: THEME.primary }),
  item = compose({ backgroundColor: '#ccc' }, hover({ backgroundColor: 'white' })),
  selected = style({ fontWeight: 'bold' })
}
// ...and when rendering
<div {...container}>
  <ul>
    <li {...item}> one </li>
    <li {...item} {...selected}> two </li>
    <li {...item}> three </li>
  </ul>
</div>


// encapsulate custom behavior/attributes with components / functions
let types = {
  'primary': 'blue',
  'secondary': 'gray',
  'disabled': 'transparent'
}

let Button = ({ type, children, onClick = ::console.log }) =>
  <button {...style({ backgroundColor: types[type] })}
    onClick={onClick}>
      {children}
  </button>

// and when rendering ...
  <div>
    are you ready?
    <Button type='primary'>click me!</Button>
  </div>

// compose glamor styled react components (with propMerge the order of css rules/styles doesn't matter 
// because it will take care of a proper merge) 
const specificStyle = style({ height: '100px' });
const defaultStyle = style({ height: '200px' });

let DefaultContainer = (props) => <div {...propMerge(defaultStyle, props)}/>
let SpecificContainer = (props) => <DefaultContainer {...specificStyle}>{props.children}</div>

// or make your own helper function alá aphrodite et al
let sheet = createSheet({
  container: {...},
  item: {
    ...,
    ':hover': {
      ...
    }
  }
})

// and when rendering
<div {...sheet.container}>
  ...etc...
</div>
```

## Conditional styles
---

All the functions exposed by glamor do cleanup of arguments. False, null, undefined and {} values will be removed.

Following invocations produce same css rules:
```jsx
style({ fontSize: '1rem',  color: null }) 
style({ fontSize: '1rem',  color: false }) 
style({ fontSize: '1rem',  color: undefined }) 
style({ fontSize: '1rem' }) 
compose([ { fontSize: '1rem' }, false, undefined, null, {} ])
```

You can use it to build conditional styles this way:

```jsx
// undefined values are ignored
const Component = ({ width, height }) => <div css={{ width, height }} />

<Component width={10} height={10} />
// css: { width: 10, height: 10 }
<Component width={10} />
// css: { width: 20 }
<Component />
// will insert nothing to css

// make use of false values to write conditions
const Component = ({ hidden }) => 
  <div css={{ 
    backgroundColor: 'green', 
    width: 200, 
    height: 100, 
    display: hidden && 'none' 
  }}/>
// or
const Component = ({ hidden }) => 
  <div css={[ 
    { 
      backgroundColor: 'green', 
      width: 200, 
      height: 100 
    }, hidden && { 
      display: 'none' 
    } 
  ] }/>
```  

any other ideas? [share!](https://github.com/threepointone/glamor/edit/master/docs/composition.md)

## Glamor createElement
---

a drop-in replacement for React.createElement, allowing you to pass a `css` prop to elements. The props is converted to a className and added to the element.

### Usage 

```jsx
import { createElement } from 'glamor/react'
/* @jsx createElement */

<div css={{ color: 'red' }}>
  I'm red!
</div>
```

The props accepts arrays as well, so you could do 
```jsx
<div css={[{ color: 'red' }, someRule, { ... }]}> ... </div>
```

Use the following settings to avoid the `createElement` boilerplate 

`.babelrc`
---
```json
{
  ...
  "plugins": [
    [
      "transform-react-jsx",
      { "pragma": "Glamor.createElement" }
    ]
  ]
}
```

`webpack.config.js`
---
```js
...
plugins: [
  new webpack.ProvidePlugin({
    Glamor: 'glamor/react'
  })
]
```

### Integration with typescript
---

If you're looking to make this work with typescript, [follow these instructions](https://github.com/threepointone/glamor/issues/283)

## CSS as values
---

Consider the hello world of css 
```css
.error { color: red; font-weight:bold; }
```

This is usually read as 'all elements with class "error" will have bold red text'. Now, we could also read this as 'apply "error" class to the desired element to get bold red text'. This is good; our 'mental model' now has a whole set of rules, somewhat indexed by selector, that we can apply on elements to make them look a particular way. This approach is used heavily, and fairly successfully in plain css frameworks. Consumers use classes like .button, .info, .form-field which behave as an api of sorts for the f/w, and style their elements / components. 

However, this has one problem. Suppose I wanted an element to have bold red text only when hovered on (and possibly add some more properties). What do I do then? 
- I have to edit the selector, which is not possible because it's in a dependency. 
- I have to copy paste the whole rule, which misses the point of the f/w, and would miss out on further changes to .error.
- I have to reimplement :hover with javascript and apply it ahahahahaha please no

This is because CSS complects an element target, state and style into a single rule, and provides no real way to extend previous definitions. The example might seem contrived, but you can see how this would become critical with complex rules, for multiple pseudo states / selectors / media queries / etc. It's particularly grating because in javascript, we merge and compose values all the time with Object.assign, lodash, or even simple loops+assignment. Being unable to take this 'unit' of css and use it to compose further is annoying, isn't modular, leads to bugs, and (insert ThoughtLeader™ rant). 

Anyway. What possible solutions do we have?

- [custom css properties / vars](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_variables)

```css
:root {
  --error-color: red;
  --error-weight: bold;
}

.x { 
  color: var(--error-color);
  font-weight: var(--error-weight);
}
.y:hover { 
  color: var(--error-color); 
  font-weight: var(--error-weight); 
  font-style: italic 
}
```
- this allows us to have 'references' and share variables, though it's only available for property values. If your browser supports it, these can be dynamic! (Thanks @davidkpiano!)


- the [`@apply rule` spec](https://tabatkins.github.io/specs/css-apply-rule/) 
```css
:root {
  --error: { color: red; font-weight: bold }
}

.x { @apply --error }
.y:hover { @apply --error; font-style: italic }
```
This is great! `@apply` allows custom property sets, which can be further composed. You'll have to get consumers to compile to this spec, but it seems like a good feature to land.

- similarly, your favorite css preprocessor might have some form of mixin construct to apply rules(eg - [sass](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#mixins), [less](http://lesscss.org/features/#mixins-feature). sass also has [an extend construct](http://www.sass-lang.com/documentation/file.SASS_REFERENCE.html#_in_directives) that's very close to what we need. These are nice too, but can't be dynamic, and have the mental overhead of having to learn a custom DSL. Speaking of which...

css-in-js has the advantage of being able to leverage javascript, and gives us good alternatives. Some examples - 

- [cxs](https://github.com/jxnblk/cxs)
```jsx
let error = { color: 'red' }
let x = cxs(error)
let y = cxs({ ':hover': { ...error, fontStyle: 'italic' })
```
This is much nicer, and feels more natural in the js world. Values could be dynamic too! But you can't compose previously created rules, and developers have to manually merge styles. This might not be a big deal, and has the advantage of being fairly framework independent (depending on what features you use)

- styled-components 
```jsx
let error = css` color: red; font-weight: bold; `
let X = styled.div`
  ${error}
`
let Y = styled.div`
  :hover {
    ${error};
    font-style: italic;
  }
`
```
This is really nice, and being backed by the postcss parser makes it fairly safe. Depending on your feelings about concatenating strings, SC might be the solution for you. 

- glamor! (because this is clearly self serving)
```jsx
let error = css({ color: 'red' })
let x = error // no need to wrap, just use directly 
let y = css({ ':hover' : [x, { fontWeight: 'bold' }])
```
with glamor, rules are first class values, and can be used at any point in an object. You can even pass arrays of rules at any point to compose them in order, and glamor takes care of nesting and precedence and whatnot. 


css-in-js gives us an opportunity to compose css and styles in ways we haven't been able to before, and in a fluent, accessible manner. I recommend you try it!


To get to my point - I think 'css frameworks' in the future will consist of more decoupling between selectors and styles, and we'll be functionally creating values to apply on elements. Exporting them as regular js objects can benefit the entire css-in-js ecosystem (and perhaps even compile other css targets like postcss/sass/etc). That's... good, I guess?

## Use real css [experimental, buggy]
---

tl:dr; 
  - you can now write 'real' css in your javascript
  - with syntax highlighting and linting
  - that gets precompiled / extracted 
  - glamor goodies apply 

To use, add 'glamor/babel' to the `plugins` field in your babel config. 

```jsx
import { css } from 'glamor'

let rule = css`
  color: red;
  font-family: helvetica, sans-serif;
  :hover {
    color: blue
  }
  @media(min-width: 300px) {
    color: green
  }
`
// ...
<div className={rule}>
  zomg!
</div>
```

longer 

```jsx
let rule = css`  
  color: yellow; /* 'real' css syntax */
  
  /* pseudo classes */  
  :hover {
    /* just javascript */
    color: ${ Math.random() > 0.5 ? 'red' : 'blue' };
  }
  
  /* contextual selectors */
  & > h1 { color: purple }  
  html.ie9 & span { padding: 300px }
  
  /* compose with objects */
  ${{ color: 'red' }}
  
  /* or more rules */
  ${ css`color: greenish` }
  
  /* media queries */
  @media (min-width: 300px) {
    color: orange;
    border: 1px solid blue;
    ${{ color: 'brown' }}
    /* increase specificity */
    && {
      color: blue;
      ${{ color: 'browner' }}
    }
  }
`
```

### Syntax highlighting 
---
- atom - via [language-babel](https://github.com/styled-components/styled-components#syntax-highlighting)
- sublime text - [open PR](https://github.com/babel/babel-sublime/pull/289)

### Linting
---
via [stylelint-processor-styled-components](https://github.com/styled-components/stylelint-processor-styled-components)


### babel plugin
---

The babel plugin replaces the inline css with glamor friendly json. Everybody wins! This - 
```jsx
css` color: red `
```
becomes this -
```jsx
css({ color: 'red' })
```
eliminating the need for the css parser in the js bundle. wowzah.

### Caveat
---

- interpolations *may* fail for unaccounted patterns. open an issue if you find one. 


### Styled-components
---

[right this way](https://github.com/threepointone/glamor/blob/master/docs/styled.md)

### Todo
---

- move to [rework](https://github.com/reworkcss/css)/anything else for parsing 
- `@supports`
- infinite nesting
- global css
- keyframes, font faces
- override pragma
- match more of the css spec (currently ~2.1)
- fallback values
- in-editor linting
- more interpolation points
- better parsing errors
- tests!!!
- guidance for difference types of 'static' css extraction



## What happens when I call css(...rules)?
---

`->` denotes possible return points

Phases, in order - 

- check weakmap cache `->`
- recursively normalize / flatten / merge 
- hashify
- check hash cache `->`
- check insertion cache 
  - split into css rule objects 
  - apply plugins (vendor prefixing, etc)
  - generate css
  - insert into stylesheet, update insertion cache 
- create rule 
- update hash, weakmap caches
- `->`

1. weakmap cache
---

As a first line of defense, we check to see whether the inputs have been used before to generate rules, and return cached instances if available, [as detailed here.](https://github.com/threepointone/glamor/blob/master/docs/weakmaps.md)

2. normalization 
---

we accept different types of inputs - objects, arrays, other rules, and nested variations thereof, allowing them to be composed as the developer pleases. Internally though, it makes sense to 'flatten' them down, making it simpler to merge values and whatnot. This also has the advantage of being able to dedupe paths, and compose complicated selectors split out in different objects. [todo- more details on the algorithm?]


3. hashify
---

the big idea here - 2 separate calls to `css({color: 'red'})` should refer to the same css rule. So unlike 'real' css where one has to decide the name of the class in advance, we let our computers figure out a unique classname for the given input. The simplest way (I think), is to generate a hash (I use murmur2) for the given input, and save the generated rule against it; so any time we use an input that maches the hash, we can return the cached rule instead.

4. insertion cache 
---

This cache simply checks whether a rule with a given id/hash has been inserted into the stylesheet yet. You might be wondering why we have both the hash and insertion caches. This comes into play when interacting with SSR prerendered css. We want to be able to populate the hash caches etc, yet skip inserting the css for rules that have already been rehydrated. So... yeah, this is why. Simple :) 

5. plugins
---

The normalized style is broken into different bits, corresponding to individual css rules, then passed and transformed through the plugin chain [as detailed here.](https://github.com/threepointone/glamor/blob/master/docs/plugins.md) These include support for array fallbacks and vendor prefixes by default.

6. generate css
---

This is as straightforward as you'd imagine. The different different bits from the previous phase are converted into css. Of note, we use a vendored version of React's `CSSPropertyOperations` to convert the object into a css string. 

7. insert into stylesheet 
---

We use our own abstraction over the browser's stylesheet to insert the rule into the dom. This abstraction also works on node, letting us do stuff like SSR, etc. It also uses different modes of inserting styles based on the environment, 
[as detailed here](https://github.com/threepointone/glamor/blob/master/docs/stylesheet.md)

8. create rule 
---

Finally, we create an object to return. It has the shape - 
```jsx
{
  'data-css-<hash>': 'possible label',
  toString: () => 'css-<hash>' // marked non-enumerable 
}
```

It's a funny looking thing, but has the advantage of being able to be spread on the props of an element, or coerced into a string to be used a classname. For the curious, it's actually 'expensive' to create this object, causing a deoptimization because they all have unique keys. Bet you're happy we use the caches now, huh? :)


### Further possible enhancements 
---

- I'm unhappy with the placement of the plugins phase; indeed, I want something more powerful, being able to tap into any of the phases. 
- because of some of the implicit 'global singletons' here (the stylesheet class, caches, etc), it's harder to provide support for iframes and web component. 
- while we still need proper objects to be returned to be able to take advantage of the weak map caches, etc, we could make it faster by avoiding the `data-css-<hash>` key for environments where developers only use the rule has classnames. Seems like a premature optimization though, and you'd be better off following the [performance guidelines](https://github.com/threepointone/glamor/blob/master/docs/performance.md)
 
## jsxstyle shim
---

This is a clone of [jsxstyle](https://github.com/petehunt/jsxstyle/)

Usage:

```jsx
import { View } from 'glamor/jsxstyle'

// ...

<View 
  color='red'   // regular style properties 
  backgroundColor='#ccc'
  css={{ outline: '1px solid black' }} // or a style object
  hover={{ color: 'blue' }} // pseudo classes
  select={[' li:nth-child(3)', { textDecoration: 'underline' }]} // arbitrary selectors
  media={[ '(min-width: 400px)', {
    width: '85%',
    padding: 0
  } ]} // media queries
  compose={[...]}  // add as many more rules 
  component='ul' // use any tag/component
  style={{ border: '1px solid green' }} // 'inline' style  
  className='mylist' // combine with aphrodite/css modules/etc 
  onClick={() => alert('what what!')} // event handlers
  props={{ disabled: true }} // pass props to the underlying element
>
  <li>one</li>
  <li>two</li>
  <li>three</li>
  <li>four</li>
</View>

// also available - Block, InlineBlock, Flex, Row, Column

```

## Performance considerations
---

When used plainly, glamor is fast and efficient enough for most sites. However, you can squeeze out more power by following some guidelines - 


- by far the biggest performance boost you can get is by compiling your code with `NODE_ENV=production` in conjunction with webpack's DefinePlugin / browserify's envify, etc. This triggers glamor to internally use `insertRule` instead of `appendChild` for adding styles to the stylesheet. You can optionally toggle this manually by calling `speedy(true/false)`
- by [server rendering your html AND css](https://github.com/threepointone/glamor/blob/master/docs/server.md), you can prepopulate / rehydrate some values when the page loads up, preventing fresh inserts on booting. This makes initial page load speed *much* faster. 
- we can also lean on [glamor's WeakMap caches](https://github.com/threepointone/glamor/blob/master/docs/weakmaps.md) by using 'static' / predefined objects as arguments, instead of passing fresh objects every time. You can automate some of this behavior by including `glamor/babel-hoist` to your babel plugin list
- In general, treat glamor as you would any other data-structure oriented library; try to prevent too many allocations, profile the critical paths in your application to identify and optimize hot paths, and be kind to immigrants. It'll all work out, I promise.


## Plugin system
---

glamor ships with a plugin system, allowing you to add custom transforms 
on selectors and styles before they get converted to css.

A plugin is a function that recieves `{ selector, style }`,
and is expected to return an object of the same shape. 

Here is an example plugin that adds ':any-link' support 
```jsx

function anyLink({ selector, style }) {
  selector = selector.split(',').map(x => x.trim())
    .forEach(p => {
      if(p.indexOf(':any-link') >=0 ) {
        result.push(p.replace(/\:any\-link/g, ':link'))
        result.push(p.replace(/\:any\-link/g, ':visited'))
      }
      else {
        result.push(p)
      }
    }).join(', ')
  return ({ selector, style })  
}

```

To add a plugin -
```jsx
import { plugins } from 'glamor'
plugins.add(anyLink)
```

NB: plugins are executed in the *opposite* order of insertion. So, if you've added `a` and `b`, `b` will get called before `a` ([#55](https://github.com/threepointone/glamor/issues/55))


You can remove it with -
```jsx
plugins.remove(anyLink)
// or to clear all plugins 
plugins.clear()
```

plugins for vendor prefixes and array fallbacks are included by default. 


[todo] - plugins for @-rules - 
- media 
- keyframes 
- animation
 
## Selectors
---

glamor uses objects to represent css rules, where 'sub'-rules can be defined with selectors/@media queries/@supports statements.

glamor supports 2 types of selectors -

- regular selectors like `:hover`, `:nth-child(2)`, `:active span.title`, `> h1`
```jsx
css({
  ':hover': {},
  '> h1': {},
  ' span.title': {} // note the leading space for this child selector  
})
```

- contextual selectors: `&` will be replaced by the 'pointer' to the target rule
```jsx
// these 2 are equivalent 
css({
  ':hover': {}
})
css({
  '&:hover': {}
})
// handy for tricky selectors
css({
  'html.ie9 & span.title': {},
  '&&': {} // increases specificity of itself
})

```

- media queries are defined similarly
```jsx
css({
  color: 'red',
  '@media (min-width: 300px)': {
    color: 'blue'
  },
  // you can use some presets
  [presets.Tablet]: { ... } // also -  Hd, Mobile, Desktop, Tablet, Phablet
})
```


- `@supports` statements are predictably similar (note - does not work in IE, UC browsers)
```jsx
css({
  '@supports (display: table-cell)': {
    ...
  }
})

```

Objects can be nested infinitely
```jsx
css({
  ':hover': {
    color: 'blue',
    ':active': {
      color: 'red'
    }
  },
  // media queries / @supports too!
  '@media (min-width: 300px)': {
    color: 'green',
    '@media print': {
      color: 'yellow'
    }
  }
})

// is equivalent to
css({
  ':hover': { color: 'blue' },
  ':hover:active': { color: 'red' },
  '@media (min-width: 300px)': { color: 'green' },
  '@media (min-width: 300px) and print': { color: 'yellow' }
})
```



- rules can be arrays 
```jsx
css({
  color: 'red',
  ':hover': [
    { color: 'blue', fontWeight: 'bold' },
    { textDecoration: 'underline' }
  ]
})
```


- that might not seem like much, but rules can be other rules too, so...
```jsx
let bold = css({ fontWeight: bold })
let rule = css({
  color: 'red',
  '> h1': bold,
  ':hover': [{ color: 'blue' }, bold, ...more]
})
```

This gives us extreme flexibility in defining and combining rules for various situations. 


## server side rendering
---

`renderStatic(fn:html)`
`renderStaticOptimized(fn:html)`
`rehydrate(ids)`

this api is mostly copied from [aphrodite](https://github.com/Khan/aphrodite);
render your component inside of a callback, and glamor will gather all
the calls you used and return an object with html, css, and an array of ids
to `rehydrate` the library for fast startup

```jsx
// on the server
import { renderStatic } from 'glamor/server'
let { html, css, ids } = renderStatic(() =>
  ReactDOMServer.renderToString(<App/>)) // or `renderToStaticMarkup`
```

```html
<!-- when rendering your html -->
<html>
  <head>
    <!-- to avoid certain characters getting encoded
      as html entities (like quotes in css 'content' property),
      we use dangerouslySetInnerHTML to inject our css
    -->
    <style dangerouslySetInnerHTML={{ __html: css }} />
    <!-- alternatively, you'd save the css to a file
      and include it here with
    <link rel='stylesheet' href='path/to/css'/>
     -->
  </head>
  <body>
    <div id='root'>${html}</div>
    <script>
      // optional!
      window._glam = ${JSON.stringify(ids)}
    </script>
    <script src="bundle.js"></script>
  </body>
</html>
```
```jsx
// optional!
// when starting up your app
import { rehydrate } from 'glamor'
rehydrate(window._glam)
ReactDOM.render(<App/>, document.getElementById('root'))
```

- if using `rehydrate`, it HAS to execute before you run any code that defines any styles. [This can get tricky with es6 imports.](https://github.com/threepointone/glamor/issues/37#issuecomment-257831193)

caveat: the above will include all the css that's been generated in the app's lifetime.
This should be fine in most cases. If you seem to be including too many unused styles,
use `renderStaticOptimized` instead of `renderStatic`. This will parse the generated
html and include only the relevant used css / ids.

WARNING: if you're bundling your *server side* code with webpack/browserify/etc (as opposed to just browser code), be warned of a subtle issue with excluding node_modules from the module. More details in [this twitter thread](https://twitter.com/andrewingram/status/771370174587043840), and [this issue](https://github.com/threepointone/glamor/issues/37). tldr - be certain to exclude *all* glamor modules, not just the root.


inline (experimental)
---

you can just inline critical css into your html

```jsx
// on the server
import inline from 'glamor/inline'
let html = inline(
  ReactDOMServer.renderToString(<App/>)
) 
```

there is no step 2!

## styled-components [experimental, buggy]
---

an implementation of the styled-components API with glamor 

To use, add 'glamor/babel' to the `plugins` field in your babel config. 

```jsx
import { styled } from 'glamor/styled'

const Button = styled.button`  
  background: ${props => props.primary ? 'palevioletred' : 'white'};  /* Adapt the colors based on primary prop */
  color: ${props => props.primary ? 'white' : 'palevioletred'};
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
`

```

pros
---

- all glamor goodies apply; SSR, composition, etc 
- the babel plugin precompiles the code to glamor objects


cons 
--- 
- very feature incomplete 
- parser doesn't match all of the css spec (currently about ~2.1)
- shallow nesting (for now)
- interpolations *may* fail for unaccounted patterns. open an issue if you find one. 

todo 
---

- react-native
- strip non native props 
- `<ThemeProvider />`
 
## StyleSheet
---

This doc is about the motivations/ design/ implementation of the backing StyleSheet class. 

requirements - 

- create style tags as necessary 
- synchronously insert css rules, one by one 
- work on server side 
- serialize / flush css 
 
## Theme
---

you have a base component that includes some styles. 
```jsx
class Button extends React.Component {
  render() {
    return <button {...this.props} {...style({ backgroundColor: 'blue' })}>
      {this.props.children}
    </button>
  }
}
```

Consumers of this component will want to style this component from any point of the hierarchy. 
- A `<Submit/>` button will want to add `{ fontSize: 20, color: 'white' }`
- The `<ButtonGroup/>` that includes it will want to adjust margins and border colors 
- The `<LoginForm/>` will add padding and perhaps a custom `fontFamily`
- This is further complicated by the fact that the `<LoginForm/>` is not a direct parent of the `<Button/>`

[(for more details)](https://medium.com/@taion/the-problem-with-css-in-js-circa-mid-2016-14060e69bf68)

This is a solution for react+glamor.

We create a theme, and attach it to the '<Button/>' class

```jsx
//button.js

import { makeTheme } from 'glamor/react'

export const buttonTheme = makeTheme() 

@buttonTheme()
class Button extends React.Component {
  render() {
    return <button {...this.props} {...merge(style({ backgroundColor: 'blue' }), this.props[btnStyle.name])}>
      {this.props.children}
    </button>
  }
}
```

alternately, we can move the default styling into `btnStyle.base`
```jsx
// btn.js

import { makeTheme } from 'glamor/react'

export const buttonTheme = makeTheme()

@buttonTheme({ backgroundColor: 'blue' })
class Button extends React.Component {
  render() {
    return <button {...this.props} {...this.props[btnStyle.name]}>
      {this.props.children}
    </button>
  }
}
```

Now, at any point in the component tree, we can decorate components with styles to be merged into the button
```jsx
// btn-group.js
import { Button, buttonTheme } from './btn.js'
import { hover, firstChild, lastChild, merge } from 'glamor'

@buttonTheme.add(merge(
  hover({ backgroundColor: 'gray' }), 
  firstChild({ borderTopLeftRadius: 10 }), 
  lastChild({ borderTopRightRadius: 10 })
))
export class ButtonGroup extends React.Component {
  render() {
    return <div>
      <Button>one</Button>
      <Button>two</Button>
      <Button>three</Button>
    </div>
  }
}

// form.js
import { buttonTheme } from './btn.js'
import { ButtonGroup } from './btn-group.js'
import { LoginButton } from './login-btn.js'

@buttonTheme.add({ fontSize: 20, margin: 20 })
export class Form extends React.Component {
  render() {
    return <div>
      woah, where are the buttons?
      <ButtonGroup/>
      <LoginButton/> 
    </div>
  }
}

```


`@vars()`

a decorator for css vars, inherited as they're applied deeper in the tree. 

```jsx

@vars()
class Button extends React.Component {                                      
  render() { // use available vars
    let { bgColor = 'gray', color ='white' } = this.props.vars // add defaults
    return <button {...style({ backgroundColor: bgColor, color })}>
      {this.props.children}
    </button>
  }
}

@vars({ bgColor: 'blue' }) // override / 'inherit' 
class ButtonGroup extends React.Component {                                       
  render() {
    return <div>
      <Button>one</Button>  
      <Button>two</Button>
    </div>  
  }
}

@vars({ bgColor: 'green' }) // green buttons in this branch
class LoginForm extends React.Component { 
  render() {
    return <div>
      <input value='ooga'/>
      <Button>login</Button>
    </div>  
  }
}


```

## multi-index caches with WeakMaps
---

### a quick summary of caching in javascript 

- Regular caches are associated with _values_. Stuff like http caches compare the _values_ of requests before deciding whether to 'return' the cached response or not. 
- For simple cases, we would convert this value to a string of some form, and use this string as a key on a js object to memoize some value. Similar to how one would memoize a factorial function, for example. 
- The drawback here is that not every object is serializable to a string, especially when we have to deal with the _identity_ of an object, instead of its value. Further, we can only rarely tell when to evict a value, usually flushing the cache periodically instead. 
- Maps give us a way to associate an object with the _identity_ of an object; we can use the object itself as a key on the map. This is great; for example, we'd use this to associate metadata with objects like dom elements, without writing over the properties of the element itself. 
- However, when used as a cache, we face the drawback that keys have to be manually evicted, else they stick around in memory causing a leak if it's not flushed out regularly (eventually leading you to reimplement GC for the Map. ugh). 
- WeakMaps are awesome for caches that are associated with the _identity_ of objects; they hold _weak_ references to their keys, meaning that when 'nothing' is referring to them anymore, they go poof and disappear, cleaned up with regular GC. This means you don't have to manually evict a key; e.g., when using a WeakMap with the previous example, when the dom element is destroyed, the meta data associated with it would disappear too.
- It has the drawback that the keys on the WeakMap cannot be enumerated (ie - you can't look inside the WeakMap to tell what it's holding at that moment). You also can't use strings or numbers as keys. 

Ok? ok. 

### So, what does a cache built with a WeakMap look like?

```jsx
// this takes a function as input, and returns another 
// function that applies caching on its input

function cachify(fn) {
  const cache = new WeakMap();
  return (arg) => {
    if (cache.has(arg)) {
      return cache.get(arg);
    }
    
    const value = fn(arg);
    cache.set(arg, value);
    return value;
  }
}

const cachedFn = cachify(expensiveFn);
cachedFn(x); 
cachedFn(x); // cache hit!
```

### So, how can we use it with glamor? 

Consider the core function, `css(...rules)`

- `rules` can be objects, or arrays, or other rules, but we can be sure that at a given callsite, it'll usually have a stable number of arguments, usually the same.
- `rules` are cached against their values, so we can guarantee 2 different `rules` with the same value will have the same identity.
- One thing we could also do, is use predefined objects with `css()` instead of defining fresh objects every time it's called. This way, we would pick up on the identity of the object passed in, and return a cached version if available.
    
  ```jsx 
  // instead of -

  <div className={css({ color: 'red' })} />

  // we would do

  const red = { color: 'red' };
  // ...
  <div className={css(styles.red)} />

  // or even 
  const red = css({ color: 'red' });
  // ...
  <div className={red} />  
  ```
- Indeed, we could use a babel plugin that recognizes `css()` calls with 'static' objects, and hoist those objects up to the highest scope possible. This plugin exists in glamor as `glamor/babel-hoist`. DX!!!

### What about multiple arguments?

`css()` is non commutative; i.e., `css(a, b) != css(b, a)` (or rather, might not be equal). This is a good opportunity to build a 'tree' of WeakMap nodes, keyed at levels by their respective arguments, terminating in the value of the `css()` call itself. 

That's quite a mouthful. Here's some code to wash it down. 

```jsx
const caches = [null, new WeakMap(), new WeakMap(), new WeakMap()]; // only 1, 2, and 3 element caches

function cachify(fn) {
  return (...args) => {
    if (caches[args.length]) {
      const cache = caches[args.length];
      let ctr = 0;
      
      while (ctr < args.length - 1) {      
        // navigate to the 'leaf', creating nodes as you traverse
        if (!cache.has(args[ctr])) {
          cache.set(args[ctr], new WeakMap());
        }
        cache = cache.get(args[ctr]);
        ctr++;
      }
      
      if (cache.has(args[args.length - 1])) { 
        return cache.get(args[ctr]);
      }
    }
    
    const value = fn(...args);
    
    if (caches[args.length]) {
      const cache = caches[args.length];
      let ctr = 0;

      while (ctr < args.length - 1) {
        // like above, navigate to the leaf
        cache = cache.get(args[ctr]);
        ctr++;
      }
      cache.set(args[ctr], value);
    }
    
    return value;
  }
}

css = cachify(css);
css(a, b); 
css(a, b); // cache hit!
```

### This sounds great! Any caveats?

- You'll need to polyfill WeakMap on older browsers for the above to kick in.
- This isn't useful in pathological cases, when you're dynamically defining many tens of thousands of _unique_ rules in multiple combinations. So unless you're actually geocities.com, rendering every site on the same server, you should be good. Even then, it's not too bad, you'll hit the hash caches instead of the WeakMap ones.
- We only maintain these caches for 1, 2 and 3 argument inputs. This means that the behavior isn't very explicit, and *might* lead to strange perf characteristics with more arguments passed in a single `css()` call. Buyer beware!

### About that tweet...

- I [tweeted a couple of screenshots](https://twitter.com/threepointone/status/813804426989182976/photo/1) showing how with the above work, glamor blows past other libraries in the [css-in-js peformance benchmark](https://github.com/hellofresh/css-in-js-perf-tests) that's floating around. This is kind of cheating though, and I implied that pretty heavily. What's happening there, is that the test passes in shared objects for every run, and keeps picking up the cached rule, thereby defeating the purpose of the benchmark. I contend, however, that the benchmark itself is flawed. For one, it measures server side rendering performance (sort of)... Further, caching input objects sounds like a great idea :) In the css context, expecting 'static' objects isn't strange at all, and the optimization works well.
- keen observers will note that glamor fails the nested rule test pretty hard. That's because [that test generates unique objects for every run](https://github.com/hellofresh/css-in-js-perf-tests/blob/master/src/nested-test/styles.js), missing the WeakMap cache (despite having the same 'value'). It would be interesting to be able to detect such behavior, and warn in dev mode. An idea for another day...

## Why I like glamor
---

facts 

- lets you write real css
- It's synchronous, yet fast. 
- supports all of css.
- has escape hatches and js fallbacks for the trickier bits. 
- use a rule as a dom prop, or as a classname. both work. 
- it hits all the points in [Ryan's Random Thoughts on Inline Styles](https://www.youtube.com/watch?v=EkPcGS4TzdQ)
- framework independent, yet has goodies for your f/w of choice 
- [todo] tooling support 

opinions

- very little boilerplate. beyond importing the module and required functions, none at all, really. 
- It actively steals the best ideas from every other css-in-js libs
- by treating css as 'values', composition is easier *and* simpler
- all the goodies are in the same module, so you don't need to `npm install` multiple packages for a good stack
- I want it to be the best for other people, not just me, and won't stop till I'm satisfied. 


