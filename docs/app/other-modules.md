---
order: 11
---

Other Modules
=============

These are various [wq.app modules] that provide minor additional functionality, or form a bridge between [wq.app] and its [third-party dependencies].

## wq/appcache.js
[wq/appcache.js] provides event connectors for reporting the status of the [application cache] to the user.

### API
wq/appcache.js is typically imported via [AMD] as `ac`, though any local variable name can be used.  `ac` provides an `init()` function which accepts a single `callback` argument that will be called every time the application cache status changes, with the current state and a suitable user-friendly message.  `ac.update()` can be used to manually trigger the callback.

```javascript
define(['wq/appcache', ...], function(ac, ...) {
ac.init(_onUpdate);

function _onUpdate(status, msg) {
    $("p.status").html(msg);
}

});
```

## wq/autocomplete.js
[wq/autocomplete.js] provides a simple implementation of an AJAX autocomplete via the HTML5 [datalist] element.

### API

wq/autocomplete.js is typically imported via AMD as `auto`, though any local variable name can be used.  `auto` provides an `init()` function that optionally accepts a Mustache template to use when rendering `<option>`s for the datalist.  `init()` automatically registers an event to ensure the autocomplete mechanism is triggered whenever new pages are shown.

```javascript
define(['wq/autocomplete', ...], function(auto, ...) {
auto.init()
});
```
To configure the `<datalist>` for an autocompleted form input, a number of `data-*` attributes can be used.  `data-url` configures the URL to use for the AJAX request. `data-query` defines the name of the URL parameter to use (default is "q").  `data-min` defines the minimum number of characters the user needs to enter before the autocomplete kicks in (default 3).

```xml
<input list="example-list">
<datalist id="example-list" data-url="/autocomplete.json" data-query="q" data-min="4">
</datalist>
```

The [search] module in [wq.db] can be used as a convenient AJAX API for wq/autocomplete.js.

When using a custom template, the following attributes will be available on the context object:

name | purpose
-----|---------
`list` | An array of available options, as returned by the web service.  These would normally have `id` and `label` attributes.
`count` | The number of items in the list
`multi` | Whether there is more than one item in the list.

## wq/console.js
[wq/console.js] provides a shim for code using `console.log`, which will fail in environments where `console` doesn't exist (looking at you, IE).  The alternative of course is to never use `console.log` in production code.

### API
wq/console.js should always be imported as `console`, for obvious reasons.

```javascript
define(['wq/console', ...], function(console) {
console.log(...);
});
```

## wq/json.js
[wq/json.js] is a generic AJAX/JSON module to allow the jQuery dependency to be factored out of `wq/map.js` (and eventually other modules).  The ultimate goal is to allow the non-UI modules in wq.app to be used without jQuery.  wq/json.js provides a single function, `json.get(url, [params], [jsonp])`, that accepts a URL, an optional parameter object, and an optional flag to trigger JSONP.   `json.get()` returns a [Promise] that is resolved or rejected depending on the result of the AJAX call.

## wq/markdown.js
[wq/markdown.js] adds Markdown and syntax highlighting to wq/template.js by providing an `{{html}}` template default.

### API

wq/markdown.js is typically imported via AMD as `md`, though any local variable name can be used.  `md` provides an `init()` function that optionally accepts the name of the template default to create, and the name of the expected context variable containing markdown (with defaults of `"html"` and `"markdown"`, respectively).  `md` also provides a `parse()` function that can be called on arbitrary markdown, and a `postProcess()` function that can be used to process HTML after it is converted from Markdown.  `md.init()` should be called after `tmpl.init()`.

```javascript
define(['wq/template', 'wq/markdown', ...], function(tmpl, md, ...) {

tmpl.init(...);
md.init();

// Result: <h1>Example</h1>
tmpl.render("{{html}}", {'markdown': '# Example'});

// Result: <h1>Example</h1>
md.parse("# Example");

});
```

wq/markdown.js uses the [third-party dependencies] **marked.js** for Markdown processing and **highlight.js** for code syntax highlighting.  The parsers for Bash, CSS, JavaScript, Markdown, Python, SCSS, and XML are included by default.

## wq/online.js
[wq/online.js] provides event connectors for handling changes to online/offline status, for example when "Airplane Mode" is activated.  wq/online.js uses the boolean `navigator.onLine` and related events internally.  However, note that connectivity is really a continuum, and is affected by a variety of factors including network speed and signal strength (see [wq/wq#10]).


### API
wq/online.js is typically imported via AMD as `ol`, though any local variable name can be used.  Like wq/appcache.js, `ol` provides an `init()` function which accepts a single `callback` argument that will be called every time the online status changes, with the current state and a suitable user-friendly message.  `ol.update()` can be used to manually trigger the callback.

```javascript
define(['wq/online', ...], function(ol, ...) {

ol.init(_onUpdate);

function _onUpdate(status, msg) {
    $("p.status").html(msg);
}

});
```

## wq/progress.js
[wq/progress.js] provides a simple way to create AJAX-powered auto-updating HTML5 [progress] elements.  wq/progress.js is meant to be used with a JSON web service that provides updates as to the current status of a long-running task.  wq/progress.js was originally created for use with the data import tasks in the [dbio] module.

### API

wq/progress.js is typically imported via AMD as `progress`, though any local variable name can be used.  `progress` provides a number of functions:

 * `progress.init()` takes up to four arguments: a URL route path (see [wq/router.js]) for pages that are expected to have `<progress>` elements, and up to three callback functions (`onComplete`, `onFail`, and `onProgress`).  All three of the functions will be passed the `<progress>` element and the JSON data from the web services.
 * `progress.start($progress)` starts polling for a specified, jQuery-wrapped `<progress>` element
 * `progress.stop($progress)` cancels polling for a started progress process

```javascript
define(['wq/progress', ...], function(progress, ...) {

progress.init('tasks/<slug>', _onComplete);

function _onComplete($progress, data) {
    $('p.result').html("Task Completed");
}

});
```

To configure a `<progress>` instance, a number of `data-*` attributes can be used.  `data-url` configures the URL to use for the AJAX request to update the progress status. `data-interval` defines the polling frequency in seconds (default 0.5).

```xml
<progress data-url="/getstatus.json" data-interval=0.25></progress>
```

For older browsers, the `<progress>` bar will automatically fall back to text showing the current status.

wq/progress.js assumes a specific structure for the data from the web service.  The following attributes should be specified on the returned JSON object:
 * `total`: the total number of items being processed
 * `current`: the rank of the currently processing item.  (`current / total` will be used to determine the % complete)
 * `status`: A text status indicating task state.  A status of `"SUCCESS"` or `"FAILURE"` will cause polling to cease and the `onComplete` or `onFailure` callbacks to be called.  The status names are taken from the [task state names in Celery].


## wq/spinner.js
[wq/spinner.js] is a simple API wrapper around jQuery Mobile's built-in spinner.

### API

wq/spinner.js is typically imported via AMD as `spin`, though any local variable name can be used.  `spin` provides two functions: `spin.start()` and `spin.stop()`.  `spin.start()` accepts up to three arguments: a text label to show in the spinner, a duration (in seconds) to display the spinner, and an object with other options to pass to the underlying [jQuery Mobile loader] API.

```javascript
define(['wq/spinner', ...], function(spin, ...) {

// No text; disable after someAsyncMethod is complete
spin.start();
someAsyncMethod(function() {
    spin.stop()
});

// Text, auto hide after 2 seconds, custom theme
spin.start("Loading...", 2, {'theme': 'b'});

});
```

[wq.app modules]: https://wq.io/docs/app
[wq.app]: https://wq.io/wq.app
[third-party dependencies]: https://wq.io/docs/third-party

[wq/appcache.js]: https://github.com/wq/wq.app/blob/master/js/wq/appcache.js
[wq/autocomplete.js]: https://github.com/wq/wq.app/blob/master/js/wq/autocomplete.js
[wq/console.js]: https://github.com/wq/wq.app/blob/master/js/wq/console.js
[wq/json.js]: https://github.com/wq/wq.app/blob/master/js/wq/json.js
[wq/markdown.js]: https://github.com/wq/wq.app/blob/master/js/wq/markdown.js
[wq/online.js]: https://github.com/wq/wq.app/blob/master/js/wq/online.js
[wq/progress.js]: https://github.com/wq/wq.app/blob/master/js/wq/progress.js
[wq/spinner.js]: https://github.com/wq/wq.app/blob/master/js/wq/spinner.js
[wq/router.js]: https://wq.io/docs/router-js

[AMD]: https://wq.io/docs/amd
[application cache]: https://developer.mozilla.org/en-US/docs/Web/HTML/Using_the_application_cache
[datalist]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/datalist
[search]: https://wq.io/docs/search
[progress]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/progress
[wq/wq.app#26]: https://github.com/wq/wq.app/issues/26
[wq/wq#10]: https://github.com/wq/wq/issues/10
[wq collectjson]: https://wq.io/docs/collectjson
[jQuery Mobile loader]: http://api.jquerymobile.com/1.3/loader/
[dbio]: https://wq.io/dbio
[wq.db]: https://wq.io/wq.db
[task state names in Celery]: http://docs.celeryproject.org/en/latest/userguide/tasks.html#states
[Promise]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
