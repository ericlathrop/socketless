# How this library is versioned

Socketless _strictly_ adheres to [semver](https://semver.org)'s major.minor.patch versioning:

- patch version changes indicate bug fixes and/or internal-only code changes,
- minor version changes indicate new functionality that does not break backward compatibility,
- major version changes indicate backward-incompatible external API changes.

# Current version history

## v2.2.0 (3 November 2023)

Added `init()` to clients, to offer an alternative to the constructor with all instance properties available for use. This is particularly important for web clients where we may need a "startup state" that we cannot set in the constructor as the protected `this.state` instance property does not exist yet.

## v2.1.5 (31 October 2023)

README.md update to make sure the initial example shows off everything rather than browserless client-server connections. This is important enough to merit a patch release.

## v2.1.4 (31 October 2023)

bug fix when trying to connect to a server that isn't running. Previous this would throw a SyntaxError claiming that the URL provided was invalid, when the actual problem isn't that the URL is invalid, it's just not accessible. That's not an error: that's the internet.

## v2.1.3 (28 October 2023)

bug fix in browser state freezing when there is no state to update.

## v2.1.2 (28 October 2023)

bug fix in how the client reconnect function propagated the socket disconnect signal.

## v2.1.1 (28 October 2023)

Added missing docs for the `.reconnect()` function.

## v2.1.0 (28 October 2023)

Clients now have a `.reconnect()` function for reconnecting to the server, making things like "letting the client run while we restart the server" much easier.

## v2.0.1 (28 October 2023)

- Fixed client crashing if the browser tried to call server functions without (or before) the web client was connected to the server.
- Updated the way errors are thrown by the SocketProxy's `apply` function, so that you can actually tell where things go wrong.

Previous you would get:

```
socketless.js:426 Uncaught (in promise) Error:
    at Object.apply (socketless.js:426:17)
    at async #authenticate (test.js:36:14)
```

Where Firefox would give you a few more steps, but Chrome wouldn't. Nnow you'll get:

```
test.js:37 Uncaught (in promise) CallError: Server unavailable
    at #authenticate (test.js:36:32)
    at BrowserClient.init (test.js:18:23)
    at createBrowserClient (socketless.js:507:27)
    at test.js:48:15
```

Without the line for `apply` because _you should not need to care about `socketless` internals_, and with both Firefox and Chrome reporting the full stack trace.

## v2.0.0 (25 October 2023)

Locked down the sync state in the browser. This breaks backwards compatibility by no longer allowing the browser to modify its `this.state` in any way, as any modifications would throw off the json diff/patch mechanism used by state syncing.

# Previous version history

## v1.0.8 (25 October 2023)

Improved error reporting when trying to deep copy using JSON.parse(JSON.stringify)

## v1.0.7 (23 October 2023)

Fixed a phrasing "bug" around how throws were reported across a proxied call.

## v1.0.6 (22 October 2023)

Fixed a bug in how generateSocketless injects the browser/webclient string identifier.

## v1.0.5 (22 October 2023)

Fixed a bug in how rfc6902 was resolved.

## v1.0.4 (22 October 2023)

Removed the webclient/socketless.js build step, which was only necessary to allow the bundling that was removed in v1.0.2

## v1.0.3 (22 October 2023)

Fixed a bug in upgraded-socket that caused functions to be called with the wrong `bind()` context.

## v1.0.2 (22 October 2023)

removed the build step, it just interfered with debugging, and the footprint of the library with sourcemap was bigger than just including the `src` dir.

## v1.0.1 (22 October 2023)

added a source map for better debugging

## v1.0.0 (21 October 2023)

Full rewrite

- code is now ESM
- the transport mechanism is now based on Proxy objects
- fewer files!
- less code!
- easier to maintain! (...hopefully)
- the release gets compiled into a single `library.js`` file

# Pre-1.0 versions

## v0.13.9

Added webclient middleware support

## v0.13.8

Added crash-protection

## v0.13.7

Added HTTPS support

## v0.13.5

Added sequence tracking to state syncing mechanism

## v0.13.3

Added URL parameter parsing

## v0.13.0 (16 January 2021)

Added custom routes for webclients

## v0.12.7

Added automatic client id generation at the server, so that the server has a keyable value, and can communicate that back to the client if necessary.

## v0.12.5

Fixed webclient syncing (sequencing was failing, and so each update led to a secondary full sync)

## v0.12.4

Removed a warning while parsing an override function chain, because the behaviour was correct but the warning was nonsense.

## v0.12.3

Removed morphdom from the dependency list. It had no reason to be there.

## v0.12.0 (26 February 2020)

Changed the APIs and added Jest testing to make sure they actually work
Updated the README.md to be a hell of a lot clearer
Created a web page for https://pomax.github.io/socketless

## v0.11.8

Fixed an edge case bug where direct-syncing web clients when using `$` rather than `:` as namespace separator in Client functors caused the syncing code to do everything right up to the point where it had to call the correct API function. And then called the `:` one instead of the `$` one.

## v0.11.6

Stripped the demo code for the npm published version, because why bloat the library with 150KB of source that you're not going to care about if you just need this as one of the tools in your codebase toolbox?

## v0.11.5

Removed the `jsonpatch` dependency, saving another 250KB of on-disk footprint. Previously, `rfc6902` did not have a dist dir with a for-browser min.js, but as of v3.0.4 it does, and so `jsonpatch` no longer has any reason to exist as dependency, or in the code.

## v0.11.2, 0.11.3

Tiny tweaks

## v0.11.1

Update the web client `update()` function to be called as `update(state)` purely to spare you having to issue `const state = this.state;` as first line of your update function.

## v0.11.0 (2 August 2019)

Changed how client/webclient state synchronization works: with `directSync` the client instance is reflected to the web client directly. Without it (which is the default behaviour) the client's `this.state` will be reflected as the web client's `this.state`. Previously, the client's `this.state` would be reflected as the web client's state without `this.state` scoping, which was super fragile. This way, things are both more robust, and more obvious: if you're using `this.state`, you're using `this.state` both in the client and the webclient. If you're not... you're not.

Also, I swapped `socket.io`, which was useful for initial prototyping, for `ws`, which is almost 4MB smaller. There's no reason to use a huge socket management library when web sockets are supported by basically everything at this point in time.

## v0.10.0 (25 July 2019)

Fixed the way function names are discovered, so that `SomeClass extends ClientClass` can be used as argument to `createClientServer()`. Previously, only the class's own function names were checked, so an empty class extensions -which you'd expect to inherit all functions- was considered as "not implementing anything". So that was bad and is now fixed.

## v0.9.1

Fixed incorrect dependency path resolution for generating socketless.js for web clients.

## v0.9.0 (13 July 2019)

This version improves the sync mechanism for web clients, using a far more sensible sync locking mechanism: if a synchronization is already in progress, rather than just firing off a second one in parallel, queue up the _signal_ that another sync is required, but do nothing else. When a sync finishes, it checks whether that signal is active, and if so, deactivates it and performs one more sync operation. This means that if 10 sync operations are called in rapid succession, the first call starts a sync, the second call sets the signal, the third through tenth do _nothing_, and when the first sync finishes, it sees that one more sync is required. This saves socket listener allocation, processing, and time. Triple win!

## v0.8.0 (12 July 2019)

This adds a web client, which is a regular client that a browser can connect in order to run a thin-client UI on top of the true client's state. It's pretty frickin sweet. It also comes with a full multiplayer mahjong game demo, because the only way I could thinkg of to properly test and debug the web client code was to sit down and write a serious piece of code with it. It's heavily code-commented, and is probably far more elaborate than you'd expect a tutorial/demonstrator to be.

Good. More projects need full-blown codebases as examples for how to use them.

## v0.7.0 (28 June 2019)

This adds explicit API objects back in, but as optional third argument to `generateClientServer`. It also adds in broadcasting by clients, based on clients knowing their own API and so knowing which function they want to have triggered by a broadcast.

(Server broadcasting is simply calling `this.clients.forEach(c => c.namespace.fname(...)`)

## v0.6.0 (27 June 2019)

This version changes the way you use the library, removing the need for a separate API object at the cost of having to namespace all your functions in your client and server classes. The `generateClientServer` function can no be called with an API object, and must be called as `generateClientServer(ClientClass, ServerClass)` instead. This will perform auto-discovery of all the API functions and their namespaces, and removes the need to pass the client and server classes when actually building clients and servers.

The README.md was effectively entirely rewritten based on these new patterns and conventions.

## v0.5 (26 June 2019) and below

- API-definition-based parsing with a lot of explicit passing of Client and Server classes
