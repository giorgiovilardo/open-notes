## Intro

### What problem does HTMX it solve?

Modern jQuery, avoid big heavy stuff like react/vue and complex toolchains. Adds interactivity by just using html attributes. It takes HTML responses from the server and injects them into the page without virtual dom and other techniques. Endpoints MUST return html. Avoid keeping state on the client.

### How it's done?

It has many utilities, i.e.:

* Server requests - use the `hx-VERB="/path"` attribute on the elements. `<button hx-post="/foo">`
* Content loading - `hx-target, hx-swap, hx-select, etc.` they help deciding where and what to put on the page
* Other stuff - `hx-trigger, hx-confirm, hx-push-url` - manipulation of browser history, triggered requests, etc.
* Events - `htmx:load, htmx:configRequest, htmx:beforeSwap` - "meta" stuff, i.e. configRequest to configure your ajax req.

## Making requests

### Simple stuff

Warn: add `hx-request='{"noHeaders": true}'` if by any chance in the future the code does not work anymore. Useful with 3rd party resources.

`hx-VERB` and a path, local or nonlocal (use the meta config for nonlocal reqs)

`hx-target` chooses which element to update with the data. Be careful as even if hx-target accepts a css selector, it will not update multiple targets by default.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://unpkg.com/htmx.org@2.0.4"></script>
  <meta name="htmx-config" content='{"selfRequestsOnly":false}'>
  <title>Document</title>
</head>
<body>
  <div>
    <button hx-get="https://httpbin.org/html">Fetch</button>
    </div>
    
    <button hx-post="https://httpbin.org/post" hx-target=".target">Post</button>

    <div class="target"></div>
</body>
</html>
```

The default behaviour is to load the response of the request as a CHILD of the requesting element.

HTMX can make any element able to do requests; things like a `div`  can now fetch too, on click, even if the browser doesnt recognize them as clickable.

### Request triggers

Default behaviour is to click to trigger a request; but we can trigger requests with the `hx-trigger` attribute

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://unpkg.com/htmx.org@2.0.4"></script>
  <meta name="htmx-config" content='{"selfRequestsOnly":false}'>
  <title>Document</title>
</head>
<body>
  <div
    hx-get="https://httpbin.org/html"
    hx-target=".content"
    hx-trigger="mouseenter"
  >Fetch</div>

  <div class="content"></div>
</body>
</html>
```

Be super careful as the container is still there and will trigger a request with every `mouseenter`!

Small quirk: `change` events on `input` will fire once the `input` loses focus, not every keystroke (remember React way of updating state and input together)

### Trigger modifiers

To solve the problem of repeated triggers, we can add modifiers to the trigger string.

`once` makes the trigger fire ... just once

```html
<div
  hx-get="https://httpbin.org/html"
  hx-target=".content"
  hx-trigger="mouseenter once"
>Fetch</div>
```

`delay:1s` fires the trigger after the delay 

```html
<button
  hx-get="https://httpbin.org/html"
  hx-target=".content"
  hx-trigger="click delay:1s"
>Fetch</button>
```

`from:selector` triggers when you fire the event on the selected element with from

```html
<div
  hx-get="https://httpbin.org/html"
  hx-trigger="click from:.fetcher"
></div>

<button class="fetcher">Fetch</button>
```

#### A simple form with `from` and loading indicator

Loading indicator is easily implemented by using the class `htmx-indicator` on an element. Normally the element must be a children of the element with the `hx-` attribute and will occupy its normal space while invisible, as invisibility is just `opacity: 0`. To use it as a non-children, add `hx-indicator` attribute on the element making the request.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://unpkg.com/htmx.org@2.0.4"></script>
  <meta name="htmx-config" content='{"selfRequestsOnly":false}'>
  <title>Document</title>
  <style>
    .indicator {
      border: 2px solid red;
    }
    p {
      color: black;
    }
  </style>
</head>
<body>
  <form hx-trigger="click from:.submit" hx-post="https://httpbin.org/post" hx-target=".target" hx-indicator=".indicator">
    <label for="name">name</label>
    <input type="text" name="name" id="name">

    <label for="tel">tel</label>
    <input type="tel" name="tel" id="tel">

    <label for="email">email</label>
    <input type="email" name="email" id="email">

    <button type="button" class="submit">submit</button>
  </form>

  <div class="htmx-indicator indicator"><p>loading...</p></div>
  <div class="target"></div>
</body>
</html>
```

`hx-verb` can be directly used on the button inside the form instead of using `from` to have a more "default" behaviour. Every input in the form will be sent.

### Adding data to the request

By using the `hx-vals` attribute, one can add more fields that will be sent (always as formdata):

```html
<form
    hx-trigger="click from:.submit"
    hx-post="https://httpbin.org/post"
    hx-target=".target"
    hx-indicator=".indicator"
    hx-vals='{"stop": "loss"}'
  >
    <label for="name">name</label>
    <input type="text" name="name" id="name">

    <label for="tel">tel</label>
    <input type="tel" name="tel" id="tel">

    <label for="email">email</label>
    <input type="email" name="email" id="email">

    <button type="button" class="submit">submit</button>
  </form>
```

This will send the key `stop: loss` as form-data together with the rest of the form.

A javascript function can be invoked by doing something like this:

```html
<script>
  const now = () => {
    let now = new Date();
    return now.toISOString();
  };
</script>

<form
  hx-trigger="click from:.submit"
  hx-post="https://httpbin.org/post"
  hx-target=".target"
  hx-indicator=".indicator"
  hx-vals='js:{"date": now()}'
>
```

Another way of adding data is by using the `hx-include` attribute.

```html
<input type="text" class="included" name="name" id="name" placeholder="Enter your name">

<input type="text" class="included" name="surname" id="surname" placeholder="Enter your surname">

<button
  hx-post="https://httpbin.org/post"
  hx-target=".target"
  hx-include=".included"
>Submit</button>

<div class="target"></div>
```

`hx-include,` as other similar attributes, accepts a css selector, so you can include multiple fields.

### Different element insertion policies

#### `hx-swap` and the `innerHTML` default

HTMX by default will inject the response from the endpoint in the DOM as a children of the element making the request, basically by modifying the element `innerHTML` property.

#### `outerHTML`

Via the `hx-swap` attribute, we can change this policy. I.e., by setting `hx-swap="outerHTML"` we can substitute the target itself with the response.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://unpkg.com/htmx.org@2.0.4"></script>
  <meta name="htmx-config" content='{"selfRequestsOnly":false}'>
  <title>Document</title>
</head>
<body>
  <button
    hx-get="https://httpbin.org/html"
    hx-target=".target"
    hx-swap="outerHTML"
  >Submit</button>
  
  <div class="target"></div>
</body>
</html>
```

In this example, after the first submit, `div.target` will cease to exists, so subsequent Submit clicks will generate an error.

#### A more complex scenario

Many more swap targets exist, so let's setup a more realistic scenario, with many different divs:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://unpkg.com/htmx.org@2.0.4"></script>
  <meta name="htmx-config" content='{"selfRequestsOnly":false}'>
  <title>Document</title>
  <style>
    body {
      font-size: 2em;
    }
    .parent {
      border: 10px solid red;
    }
    .destination {
      border: 10px solid blue;
    }
    .child {
      border: 10px solid black;
    }
  </style>
</head>
<body>
  <button
    hx-get="https://httpbin.org/html"
    hx-target=".destination"
    hx-swap="innerHTML"
  >Submit</button>
  
  <div class="parent">
    Parent
    <div class="destination">
      Destination
      <div class="child">
        Child
      </div>
    </div>
  </div>
</body>
</html>
```

The default, `innerHTML` value will be inside the blue border and destroy `.child`, while `outerHTML` will destroy `.destination` and sit inside `.parent`.

#### `afterbegin`

Places the response before the first child of the target.

![[htmx_afterbegin.png]]

#### `beforebegin`

Places the response before the target element, leaving the order of other elements in the parent untouched.

![[htmx_beforebegin.png]]

#### `beforeend`

Places the response after the last child of the target.

![[htmx_beforeend.png]]

#### `afterend`

Places the element immediately after the target element.

![[htmx_afterend.png]]

The names are a bit wonky, but it might help to think about `begin` and `end` as demarcation points of your element block, so the after/before can go into parent elements.

### Multiple element content swap

This is a bit trickier without using an API we own but...go to https://ptsv3.com/ , register a dump name, and output whatever you want from there.

Basic setup:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://unpkg.com/htmx.org@2.0.4"></script>
  <meta name="htmx-config" content='{"selfRequestsOnly":false}'>
  <title>Document</title>
  <style>
    body {
      font-size: 2em;
    }
    .parent {
      border: 10px solid red;
    }
    .destination {
      border: 10px solid blue;
    }
    .child {
      border: 10px solid black;
    }
  </style>
</head>
<body>
  <button
    hx-get="https://corsproxy.io/?url=https://ptsv3.com/t/dddd/post"
    hx-target=".destination"
    hx-request='{"noHeaders": true}'
    hx-swap="afterbegin"
  >Submit</button>
  
  <div id="parent" class="parent">
    Parent
    <div id="destination" class="destination">
      Destination
      <div id="child" class="child">
        Child
      </div>
      End Destination
    </div>
    End Parent
  </div>
</body>
</html>
```

Make the dumper return this:

```html
<div>
  <h3 id="child" hx-swap-oob="innerHTML">Swapped inside child...</h3>
  Swapped inside real target
</div>
```

The magic comes from the `hx-swap-oob` attribute, that accepts same value as hx-swap. `oob` means out of band, so we can replace multiple items at the same time with this. (State controlled by the server). It can be combined with many modifiers and keyword to influence where the swap will happen.

### Selecting something from a response

In case you need specify a certain part of the response you want to take, use `hx-select="selector"`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://unpkg.com/htmx.org@2.0.4"></script>
  <meta name="htmx-config" content='{"selfRequestsOnly":false}'>
  <title>Document</title>
  <style>
    body {
      font-size: 2em;
    }
    .destination {
      border: 10px solid blue;
    }
  </style>
</head>
<body>
  <button
    hx-get="https://httpbin.org/html"
    hx-target=".destination"
    hx-request='{"noHeaders": true}'
    hx-select="h1"
  >Submit</button>
  
  <div class="destination">
    Destination
  </div>
</body>
</html>
```