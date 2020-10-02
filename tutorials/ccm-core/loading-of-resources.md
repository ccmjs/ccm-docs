### Overview
_ccmjs_ provides a service for asynchronous loading of resources. It could be used with the method {@link ccm.load}.
You can load resources like HTML, CSS, Images, JavaScript, Modules, JSON and XML data on-demand and cross-domain.
On a single call several resources can be loaded at once. It can be flexibly controlled which resources are load in serial and which in parallel.
{@link ccm.load} can be used in an [instance configuration]{@link ccm.types.instance_config} to define dependencies to other resources.

### Simplest Case: Loading by URL
In the simplest case, only an URL is passed as a parameter for a resource to be loaded:

```javascript
ccm.load( 'style.css' );
```

The URL of the resource can be a relative or an absolute path.
The resource does not have to be within the same domain and can be loaded cross-domain.

### Loading by Object
Instead of an URL, a [resource object]{@link ccm.types.resource_obj} can be passed, which then contains other information besides the URL, via which the loading of the resource is even more flexible controllable.
For example, when loading a resource, it can be specified in which context it is loaded.
With this, the CSS contained in a CSS file can be loaded into a specific [Shadow DOM](https://en.wikipedia.org/wiki/Web_Components#Shadow_DOM):

```javascript
const shadow = document.createElement( 'div' );
shadow.attachShadow( { mode: 'open' } );

ccm.load( { url: 'style.css', context: shadow } );
```

### Method Result

The method {@link ccm.load} is an asynchronous function and returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) that provides the result.
The following example loads the content of an HTML file:

```javascript
// Promise
ccm.load( 'hello.html' ).then( result => {} );
```

```javascript
// async await
const result = await ccm.load( 'hello.html' );
```

```html
<!-- hello.html -->
Hello, <b>World</b>!
```

The variable `result` now contains the HTML string `"Hello, <b>World</b>!"`.

## Error Handling

If loading of a resource fails, the resulting [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) will be rejected:

```javascript
// Promise
ccm.load( 'not_exits.html' ).catch( error => {} );
```

```javascript
// async await
try {
  await ccm.load( 'not_exists.html' );
}
catch ( error ) {}
```

The variable `error` then contains an object with informations about the error.
The following table shows what information this object contains:

Property   | Value
-----------|-------------------------------------------------------
`call`     | Action data of the original method call.
`data`     | Object of the [XMLHttpRequest](https://www.w3schools.com/xml/xml_http.asp).
`error`    | Error object of the failed [XMLHttpRequest](https://www.w3schools.com/xml/xml_http.asp).
`resource` | Passed [resource object]{@link ccm.types.resource_obj}.

In case of a [XMLHttpRequest](https://www.w3schools.com/xml/xml_http.asp) the [HTTP status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) and the response text can be checked via `data` and with `error` you can get the responded error message:

```javascript
try {
  await ccm.load( 'not_available.html' );
}
catch ( error ) {
  if ( error.data.status === 404 )     // not found?
    alert( error.error.message );      // => show error message of XMLHttpRequest
  else if ( error.data.responseText )  // error with response text?
    alert( error.data.responseText );  // => show response text
  else
    alert( 'Something went wrong' );
}
```

Watch the `error` variable in the developer console to see what other useful data is included.

## Loading with Timeout

For loading resources, a [timeout]{@link ccm.timeout} can be set:

```javascript
ccm.timeout = 10000;  // timeout after 10 seconds
```

In this example, resources that last longer than 10 seconds would fail. By default, there is no time limit.
If a result is received after the timeout has expired, a message like this appears in the developer console:

```html
[ccmjs] loading of https://my.domain/bigdata.php succeeded after timeout (10000ms)
```

## Loading of Multiple Resources at Once

On a single {@link ccm.load} call several resources can be loaded at once:

```javascript
const results = await ccm.load( 'hello.html', 'style.css', 'image.png' );
```

When multiple resources are loaded, the [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) provides an array instead of a single value as the result.
The array contains the results in the order in which the parameters were passed.
If loading a resource does not supply anything specific, the default result is the URL of the resource. This applies, for example, when loading CSS and images.
The variable `results` now contains:

```json
[ "Hello, <b>World</b>!", "style.css", "image.png" ]
```

If loading of at least one of the resources fails, the result is still an array. For failed resources, the array will contain the error object instead of the result:

```javascript
try {
  await ccm.load( 'not_exists.html', 'style.css', 'image.png' );
}
catch ( error ) {
  console.log( error );  // [ {error object}, "style.css", "image.png" ]
}
```

## Parallel and Serial Loading of Resources

It can be flexibly controlled which resources are loaded serially and which ones in parallel.
By default, resources are loaded in parallel.
When resources are to be loaded one after another, they simply need to be passed as an array:

```javascript
ccm.load( [ 'hello.html', 'style.css' ] );
```

In the example, the two resources are now loaded serially.
The serial and parallel loading can be flexibly controlled as deep as desired.
With each deeper array level you switch between serial and parallel loading:

```javascript
ccm.load(
  'hello.html',     // Array Level 0: Parallel
  [
    'style.css',    // Array Level 1: Serial
    'image.png',
    [
      'data.json',  // Array Level 2: Parallel
      'script.js'
    ],
    'logo.gif'
  ],
  'picture.jpg'
);
```

The example loads the resources in the following timeline:

Resource      | Timeline
--------------|-------------------------
`hello.html`  | ******------------------
`style.css`   | ******------------------
`image.png`   | ------******------------
`data.json`   | ------------******------
`script.js`   | ------------******------
`logo.gif`    | ------------------******
`picture.jpg` | ******------------------

## Loading of HTML
Loading an HTML file results in a string that contains the content of the loaded HTML file:

```html
<!-- hello.html -->
Hello, World!
```

```javascript
const result = await ccm.load( 'hello.html' );
```

The variable `result` now contains the HTML string `"Hello, <b>World</b>!"`.
An HTML string can be converted into an HTML element with the help method {@link ccm.helper.html} and
with {@link ccm.helper.html2json} you can transform an HTML string to JSON.
An HTML file can contain multiple HTML templates.
Each template must then be wrapped with `<ccm-template key="mykey">`, where `mykey` is a unique [key]{@link ccm.types.key} for the template:

```html
<!-- templates.html -->

<ccm-template key="main">
  <header>...</header>
  ...
  <footer>...</footer>
</ccm-template>

<ccm-template key="entry">
  <h3>...</h3>
  <p>...</p>
</ccm-template>
```

```javascript
const result = await ccm.load( 'templates.html' );
```

The variable `result` then contains an object with the loaded HTML templates:

```json
{
  "main": " <header>...</header> ... <footer>...</footer> ",
  "entry": " <h3>...</h3> <p>...</p> "
}
```

## Loading of CSS

CSS is loaded by adding a `<link rel="stylesheet" type="text/css" href="url">` in the [DOM](https://en.wikipedia.org/wiki/Document_Object_Model), where `url` is the URL of the CSS file.
Use the `context` property of a [resource object]{@link ccm.types.resource_obj} to control in which element the `<link>` tag will be appended.
With this, the CSS contained in a CSS file can be loaded into a [Shadow DOM](https://en.wikipedia.org/wiki/Web_Components#Shadow_DOM):

```javascript
const shadow = document.createElement( 'div' );
shadow.attachShadow( { mode: 'open' } );

await ccm.load( { url: 'style.css', context: shadow } );
```

By default, the CSS is loaded in the `<head>` of the webpage:

```javascript
await ccm.load( 'style.css' );
```

```html
<head>
  ...
  <link rel="stylesheet" type="text/css" href="style.css">
</head>
```

With the `attr` property of a [resource object]{@link ccm.types.resource_obj} you can add additional HTML attributes to the `<link>` tag.
For example this allows you to load CSS with [Subresource Integrity](https://developer.mozilla.org/de/docs/Web/Security/Subresource_Integrity):

```javascript
await ccm.load( {
  url: 'style.css',
  attr: {
    integrity: 'sha384-TNlMHgEvh4ObcFIulNi29adH0Fz/RRputWGgEk/ZbfIzua6sHbLCReq+SZ4nfISA',
    crossorigin: 'anonymous'
  }
} );
```

```html
<head>
  ...
  <link rel="stylesheet" href="style.css" integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous">
</head>
```

Loading of a CSS resolves, when the `onload` event of the `<link>` tag fires.
When the exact same `<link>` tag is already present in the context, then loading of CSS succeeds
without appending the same `<link>` tag again, because the CSS seems to be already loaded in this context.
The result value is the URL of the CSS file.

## Loading of Images
The method {@link ccm.load} can be used for preloading images:

```javascript
ccm.load( 'image1.png', 'image2.jpg', 'image3.gif' );
```

In the example the three images are loaded in parallel and are than in the browser cache.
Loading of an image resolves, when the `onload` event of the intern created image object fires.
The result value is the URL of the image file.

## Loading of JavaScript

Loading a JavaScript file will execute the JavaScript code contained in the file.
The JavaScript is loaded by adding a `<script src="url" async>` in the [DOM](https://en.wikipedia.org/wiki/Document_Object_Model), where `url` is the URL of the JavaScript file.
Loading of JavaScript resolves, when the `onload` event of the `<script>` tag fires.
After the JavaScript code has executed, the no more needed `<script>` tag will be removed from the [DOM](https://en.wikipedia.org/wiki/Document_Object_Model).
The result value is the URL of the JavaScript file, but the loaded JavaScript code can also set the result individually:

```javascript
/* script.js */
ccm.files[ 'script.js' ] = { foo: 'bar' };
```

```javascript
const result = await ccm.load( 'script.js' );
console.log( result );  // => { "foo": "bar" }
```

The result value of a loaded JavaScript file is what the contained JavaScript code puts in `ccm.files[ 'filename' ]`, where `filename` is the filename of the JavaScript file.
Otherwise, the result is the URL of the file as usual.
Using this convention, a JavaScript file can provide data across domains.
The publicly fetched data in the global namespace {@link ccm.files} will be directly deleted.
In case of a minimized JavaScript file, the `.min` in the filename can be omitted:

```javascript
/* script.min.js */
ccm.files['script.js']={foo:'bar'};
```

```javascript
const result = await ccm.load( 'script.min.js' );
console.log( result );  // => { "foo": "bar" }
```

With the `attr` property of a [resource object]{@link ccm.types.resource_obj} you can add additional HTML attributes to the `<script>` tag.
For example this allows you to load JavaScript with [Subresource Integrity](https://developer.mozilla.org/de/docs/Web/Security/Subresource_Integrity):

```javascript
await ccm.load( {
  url: 'script.js',
  attr: {
    integrity: 'sha384-QoLtnRwWkKw2xXw4o/pmW2Z1Zwst5f16sRMbRfP/Ova1nnEN6t2xUwiLOZ7pbbDW',
    crossorigin: 'anonymous'
  }
} );
// <script src="script.js" async integrity="sha384-QoLtnRwWkKw2xXw4o/pmW2Z1Zwst5f16sRMbRfP/Ova1nnEN6t2xUwiLOZ7pbbDW" crossorigin="anonymous">
```

## Loading of Modules

Loading a module gives you an object as result that contains all the exported members of the module:

```javascript
/* module.mjs */
export const name = 'John';
export const data = { foo: 'bar' };
export function sayHello( name ) { console.log( `Hello, ${name}!` ); }
```

```javascript
const result = await ccm.load( 'module.mjs' );
result.sayHello( result.name );  // => 'Hello, John!'
console.log( result.data );      // => {"foo":"bar"}
```

In the example `result` now contains all exported members of the module.
An `import * as result from 'url'` is executed internally, where `result` is the result object and `url` is the URL of the module file.
It is also possible to get a specific exported member only as result:

```javascript
const sayHello = await ccm.load( 'module.mjs#sayHello' );
sayHello( 'Jane' );  // => 'Hello, Jane!'
```

Then intern an `import {key} as result from 'url'` is executed, where `key` is the name of the specific member.
The member name can be set at the end of the URL after a `#`.
This allows the definition of dependencies to a certain function within an [instance configuration]{@link ccm.types.instance_config}:

```javascript
const config = {
  sayHello: [ 'ccm.load', 'module.mjs#sayHello' ]
}
```

You can also get a specific subset of exported members.
For this, several member names can be specified at the end of the URL separated by a `#`:

```javascript
const subset = await ccm.load( 'module.mjs#data#name' );
console.log( subset );  // => {"data":{"foo":"bar"},"name":"John"}
```

Just like loading JavaScript, with the `attr` property of a [resource object]{@link ccm.types.resource_obj} you can add additional HTML attributes to the `<script>` tag.

## Loading of JSON Data

In the following example JSON data is load from a server interface:

```javascript
const result = await ccm.load( 'hello.php' );
console.log( result );  // => "Hello, World!"
```

```php
/* hello.php */
<?php
echo 'Hello, World!';
?>
```

As default, the JSON data is load by [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) with the HTTP method `POST`.
Loading of JSON cross-domain works only if [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) or [JSONP](https://en.wikipedia.org/wiki/JSONP) is working.
Individual HTTP parameters can be set with the `params` property of a [resource object]{@link ccm.types.resource_obj}:

```javascript
const result = await ccm.load( {
  url: 'echo.php',
  params: {         // sets HTTP parameters
    name: 'John'
  }
} );
console.log( result );  // => "Hello, John!"
```

```php
/* echo.php */
<?php
echo 'Hello, '.filter_input( INPUT_POST, 'name', FILTER_SANITIZE_STRING );
?>
```

The used HTTP method can be set by the `method` property:

```javascript
const result = await ccm.load( {
  url: 'hello.php',
  method: 'GET'      // sets HTTP method
} );
```

With the `headers` property, you can set additional HTTP headers:

```javascript
const result = await ccm.load( {
  url: 'hello.php',
  headers: {  // sets additional HTTP headers
    Authorization: 'Basic ' + btoa( user + ':' + token )
  }
} );
```

JSON data can also be loaded via [JSONP](https://en.wikipedia.org/wiki/JSONP) if the server side supports it:

```javascript
const result = await ccm.load( {
  url: 'https://other.domain.com/data.php',
  method: 'JSONP'  // turns on JSONP
} );
console.log( result );  // => {"foo":"bar"}
```

```php
/* data.php */
<?php
$callback = filter_input( INPUT_GET, 'callback', FILTER_SANITIZE_STRING );
echo $callback.'({"foo":"bar"});';
?>
```

With [JSONP](https://en.wikipedia.org/wiki/JSONP) the JSON data is load via `<script>` tag. So just like loading JavaScript, with the `attr` property of a [resource object]{@link ccm.types.resource_obj} you can add additional HTML attributes to the `<script>` tag.
[JSONP](https://en.wikipedia.org/wiki/JSONP) is only necessary if the data has to be loaded cross-domain.
[JSONP](https://en.wikipedia.org/wiki/JSONP) is not required if [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) is already working.

Another option is to let {@link ccm.load} use the [fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch).
The second parameter of `fetch(url,init)` can be passed via the `init` property of a [resource object]{@link ccm.types.resource_obj}:

```javascript
const result = await ccm.load( {
  url: 'https://other.domain.com/data.php',
  method: 'fetch',  // uses fetch API
  init: {
    method: 'POST',
    mode: 'cors',
    cache: 'no-cache',
    credentials: 'same-origin',
    headers: {
      'Content-Type': 'application/json'
    },
    redirect: 'follow',
    referrerPolicy: 'no-referrer'
  }
} );
console.log( result );  // => {"foo":"bar"}
```

This allows you to use the full power of the [fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) in resource dependencies:

```javascript
const config = {
  sayHello: [ 'ccm.load', { url: 'sayHello.php', method: 'fetch', init: {...} } ]
}
```

## Loading of XML Data

The following example loads an XML file:

```javascript
const result = await ccm.load( 'data.xml' );
console.log( result );  // => #document
```

The `result` variable than contains a [XMLDocument](https://developer.mozilla.org/en-US/docs/Web/API/XMLDocument) containing the loaded XML.
Loading of XML cross-domain works only if [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) is working.

## Loading a Resource Regardless of its File Extension

Normally {@link ccm.load} automatically recognizes at the file extension how the resource should be loaded.
When the `type` property of a [resource object]{@link ccm.types.resource_obj} is specified, the file extension of the resource is ignored and the resource is loaded as the specified type.
If `type` is not specified and the file extension is unknown, loading of JSON is assumed.
This allows you the dynamic loading of resources.

### Dynamic Loading of CSS

The following example loads CSS from a PHP interface:

```javascript
ccm.load( { url: 'style.php', type: 'css' } );
```

```php
/* style.php */
<?php
header( 'Content-Type: text/css' );
?>
b { color: red; }
```

Although the resource does not have the file extension `.css`, it will be loaded like a CSS file.
The `<head>` now contains: `<link rel="stylesheet" type="text/css" href="style.php">`.

### Dynamic Loading of an Image

This example preloads an image that comes from a PHP interface:

```javascript
ccm.load( { url: 'image.php', type: 'image' } );
```

```php
<?php
header( 'Content-Type: image/png' );
readfile( 'intern/image.png' );
?>
```

If the PHP interface would additionally implement user authentication, images could be made available to specific user groups. An HTTP parameter could also be used to control which image should be delivered.

### Dynamic Loading of JavaScript

In this example JavaScript is load from a PHP interface:

```javascript
const result = await ccm.load( { url: 'script.php', type: 'js' } );
```

```php
/* js.php */
ccm.files[ 'js.php' ] = { foo: '<? echo 'bar'; ?>' };
console.log( 'Hello, <? echo 'World'; ?>!' );
```

The `<head>` now contains: `<script src="script.php">`.

Result in the Developer Console: `Hello, World!`

Result of the resolved Promise: `{"foo":"bar"}`

## Other Aspects

A [resource object]{@link ccm.types.resource_obj} passed to {@link ccm.load} is cloned to prevent external manipulations.

In a [resource object]{@link ccm.types.resource_obj}, the reference to an [instance]{@link ccm.types.instance} can also be passed for the property `context`.
The resource is then loaded into the [Shadow DOM](https://en.wikipedia.org/wiki/Web_Components#Shadow_DOM) of the [instance]{@link ccm.types.instance}.

When a resource is loaded into a specific context, care must be taken that this context has [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction) contact.
The chosen context should not be an on-the-fly element or part of it.
This is required so that the HTML element used to load the resource is evaluated by the browser.