# serve-handler

[![Build Status](https://circleci.com/gh/zeit/serve-handler.svg?&style=shield)](https://circleci.com/gh/zeit/serve-handler)
[![codecov](https://codecov.io/gh/zeit/serve-handler/branch/master/graph/badge.svg)](https://codecov.io/gh/zeit/serve-handler)
[![Join the community on Spectrum](https://withspectrum.github.io/badge/badge.svg)](https://spectrum.chat/zeit)

This package represents the core of [serve](https://github.com/zeit/serve) and static deployments running on [Now](https://zeit.co/now). It can be plugged into any HTTP server and is responsible for routing requests and handling responses.

In order to customize the default behaviour, you can also pass custom routing rules, provide your own methods for interacting with the file system and much more.

## Usage

Get started by installing the package using [yarn](https://yarnpkg.com/lang/en/):

```js
yarn add serve-handler
```

You can also use [npm](https://www.npmjs.com/) instead, if you'd like:

```js
npm install serve-handler
```

Next, add it to your HTTP server. Here's an example using [micro](https://github.com/zeit/micro):

```js
const handler = require('serve-handler');

module.exports = async (request, response) => {
  await handler(request, response);
};
```

That's it! :tada:

## Options

If you want to customize the package's default behaviour, you can use the third argument of the function call to pass any of the configuration options listed below. Here's an example:

```js
await handler(request, response, {
  cleanUrls: true
});
```

You can use any of the following options:

| Property                                             | Description                                               |
|------------------------------------------------------|-----------------------------------------------------------|
| [`public`](#public-string)                           | Set a sub directory to be served                          |
| [`cleanUrls`](#cleanurls-booleanarray)               | Have the `.html` extension stripped from paths            |
| [`rewrites`](#rewrites-array)                        | Rewrite paths to different paths                          |
| [`redirects`](#redirects-array)                      | Forward paths to different paths or external URLs         |
| [`headers`](#headers-array)                          | Set custom headers for specific paths                     |
| [`directoryListing`](#directorylisting-booleanarray) | Disable directory listing or restrict it to certain paths |
| [`unlisted`](#unlisted-array)                        | Exclude paths from the directory listing                  |
| [`trailingSlash`](#trailingslash-boolean)            | Remove or add trailing slashes to all paths               |
| [`renderSingle`](#rendersingle-boolean)              | If a directory only contains one file, render it          |

### public (String)

By default, the current working directory will be served. If you only want to serve a specific path, you can use this options to pass a custom directory to be served relative to the current working directory.

For example, if serving a [Jekyll](https://jekyllrb.com/) app, it would look like this:

```json
{
  "public": "_site"
}
```

**NOTE:** The path cannot contain globs or regular expressions.

### cleanUrls (Boolean|Array)

By default, all `.html` files can be accessed without their extension.

If one of these extensions is used at the end of a filename, it will automatically perform a redirect with status code [301](https://en.wikipedia.org/wiki/HTTP_301) to the same path, but with the extension dropped.

You can disable the feature like follows:

```json
{
  "cleanUrls": false
}
```

However, you can also restrict it to certain paths:

```json
{
  "cleanUrls": [
    "/app/**",
    "/!components/**"
  ]
}
```

**NOTE:** The paths can only contain globs that are matched using [minimatch](https://github.com/isaacs/minimatch).

### rewrites (Array)

If you want your visitors to receive a response under a certain path, but actually serve a completely different one behind the curtains, this option is what you need.

It's perfect for [single page applications](https://en.wikipedia.org/wiki/Single-page_application) (SPAs), for example:

```json
{
  "rewrites": [
    { "source": "app/**", "destination": "/index.html" },
    { "source": "projects/*/edit", "destination": "/edit-project.html" }
  ]
}
```

You can also use so-called "routing segments" as follows:

```json
{
  "rewrites": [
    { "source": "/projects/:id/edit", "destination": "/edit-project-:id.html" },
  ]
}
```

Now, if a visitor accesses `/projects/123/edit`, it will respond with the file `/edit-project-123.html`.

**NOTE:** The paths can contain globs (matched using [minimatch](https://github.com/isaacs/minimatch)) or regular expressions (match using [path-to-regexp](https://github.com/pillarjs/path-to-regexp)).

### redirects (Array)

In order to redirect visits to a certain path to a different one (or even an external URL), you can use this option:

```json
{
  "redirects": [
    { "source": "/from", "destination": "/to" },
    { "source": "/old-pages/**", "destination": "/home" }
  ]
}
```

By default, all of them are performed with the status code [301](https://en.wikipedia.org/wiki/HTTP_301), but this behavior can be adjusted by setting the `type` property directly on the object (see below).

Just like with [rewrites](#rewrites-array), you can also use routing segments:

```json
{
  "redirects": [
    { "source": "/old-docs/:id", "destination": "/new-docs/:id" },
    { "source": "/old", "destination": "/new", "type": 302 }
  ]
}
```

In the example above, `/old-docs/12` would be forwarded to `/new-docs/12` with status code [301](https://en.wikipedia.org/wiki/HTTP_301). In addition `/old` would be forwarded to `/new` with status code [302](https://en.wikipedia.org/wiki/HTTP_302).

**NOTE:** The paths can contain globs (matched using [minimatch](https://github.com/isaacs/minimatch)) or regular expressions (match using [path-to-regexp](https://github.com/pillarjs/path-to-regexp)).

### headers (Array)

Allows you to set custom headers (and overwrite the default ones) for certain paths:

```json
{
  "headers": [
    {
      "source" : "**/*.@(jpg|jpeg|gif|png)",
      "headers" : [{
        "key" : "Cache-Control",
        "value" : "max-age=7200"
      }]
    }, {
      "source" : "404.html",
      "headers" : [{
        "key" : "Cache-Control",
        "value" : "max-age=300"
      }]
    }]
  }
}
```

If you define the `ETag` header for a path, the handler will automatically reply with status code `304` for that path if a request comes in with a matching `If-None-Match` header.

**NOTE:** The paths can only contain globs that are matched using [minimatch](https://github.com/isaacs/minimatch).

### directoryListing (Boolean|Array)

For paths are not files, but directories, the package will automatically render a good-looking list of all the files and directories contained inside that directory.

If you'd like to disable this for all paths, set this option to `false`. Furthermore, you can also restrict it to certain directory paths if you want:

```json
{
  "directoryListing": [
    "/assets/**",
    "/!assets/private"
  ]
}
```

**NOTE:** The paths can only contain globs that are matched using [minimatch](https://github.com/isaacs/minimatch).

### unlisted (Array)

In certain cases, you might not want a file or directory to appear in the directory listing. In these situations, there are two ways of solving this problem.

Either you disable the directory listing entirely (like shown [here](#directorylisting-booleanarray)), or you exclude certain paths from those listings by adding them all to this config property.

```json
{
  "unlisted": [
    ".DS_Store",
    ".git"
  ]
}
```

The items shown above are excluded from the directory listing by default.

**NOTE:** The paths can only contain globs that are matched using [minimatch](https://github.com/isaacs/minimatch).

### trailingSlash (Boolean)

By default, the package will try to make assumptions for when to add trailing slashes to your URLs or not. If you want to remove them, set this property to `false` and `true` if you want to force them on all URLs:

```js
{
  "trailingSlash": true
}
```

With the above config, a request to `/test` would now result in a [301](https://en.wikipedia.org/wiki/HTTP_301) redirect to `/test/`.

### renderSingle (Boolean)

Sometimes you might want to have a directory path actually render a file, if the directory only contains one. This is only useful for any files that are not `.html` files (for those, [`cleanUrls`](#cleanurls-booleanarray) is faster).

This is disabled by default and can be enabled like this:

```js
{
  "renderSingle": true
}
```

After that, if you access your directory `/test` (for example), you will see an image being rendered if the directory contains a single image file.

## Error templates

The handler will automatically determine the right error format if one occurs and then sends it to the client in that format.

Furthermore, this allows you to not just specifiy an error template for `404` errors, but also for all other errors that can occur (e.g. `400` or `500`).

Just add a `<status-code>.html` file to the root directory and you're good.

## Middleware

If you want to replace the methods the package is using for interacting with the file system and sending responses, you can pass them as the fourth argument to the function call.

These are the methods used by the package (they can all return a `Promise` or be asynchronous):

```js
await handler(request, response, undefined, {
  stat(path) {},
  createReadStream(path, config) {}
  readdir(path) {},
  sendError(absolutePath, response, acceptsJSON, root, handlers, config, error) {}
});
```

**NOTE:** It's important that – for native methods like `createReadStream` – all arguments are passed on to the native call.

## Use Cases

There are two environments in which [ZEIT](https://zeit.co) uses this package:

### Production

When deploying your static project to [Now](https://zeit.co/now), you can add any of the [configuration options](https://github.com/zeit/serve-handler#options) to the `static` property within `now.json` and they will be considered when handling requests to your deployment.

### Development

When testing your project on your local device, we suggest using [serve](https://github.com/zeit/serve).

Since it comes with support for `serve-handler` out of the box, you can create a `serve.json` file to customize its behavior. It will also read the configuration from `static` inside `now.json`.

## Author

Leo Lamprecht ([@notquiteleo](https://twitter.com/notquiteleo)) - [ZEIT](https://zeit.co)
