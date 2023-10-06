# Browser behavior when replacing script tags

The hypermedia-driven approach by HTMX encourages the use of `<script>` tags for local state management.
This can lead to surprising results if a script is part of the HTML you get from an Ajax request.

Not thinking about this I created a bug that was obvious after I understood it.

## Problem

Consider the following contrived example, where we have some constant client-side state that determines whether we can play media or not.

```html
<div id="media">
  <button id="play-button" hx-get="/play-button.html" hx-trigger="click" hx-target="#media" hx-swap="outerHTML">Play
    media</button>
  <script>
    const canPlayMedia = "mediaCapabilities" in navigator;
    if (!canPlayMedia) {
      console.log("can't play media");
      const playButton = document.getElementById("play-button");
      playButton.disabled = true;
    }
  </script>
</div>
```

Here, we disable the button on client side, if the client has no "media capabilities".
We declare `canPlayMedia` as `const`, assuming that the ability to play media doesn't change during a page visit.
On clicking the button, we replace the whole `div` by the media to be played and a variant of that button with different text:

```html
<div id="media">
  Playing media ...
  <button id="play-button" hx-get="/play-button.html" hx-trigger="click" hx-target="#media" hx-swap="outerHTML">Play
    next media</button>
  <script>
    const canPlayMedia = "mediaCapabilities" in navigator;
    if (!canPlayMedia) {
      console.log("can't play media");
      const playButton = document.getElementById("play-button");
      playButton.disabled = true;
    }
  </script>
</div>
```

Notice how we send the `<script>` tag again with the same script.

The UI updates as expected, but if we check the browser console we'll find that an error is printed:

```
Uncaught SyntaxError: Identifier 'canPlayMedia' has already been declared
```

## Reproduce it

```sh
docker build -t foo . 
docker run -p 8080:80 foodocker build -t foo . && docker run -p 8080:80 foo
```

## Cause

Scripts get executed as soon as they are fetched, and the state induced by a script does not disappear if the script becomes onloaded.
In my case the issue was not as obvious in the example because I was breaking down an HTML page into parts using a template engine.

## Consequence

Ultimately one could argue that this is a classic problem of global state.
In future I will either be more mindful of the HTML parts I'm loading, or I'll avoid global state.
