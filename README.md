Loading modules on the client is not recommended because it requires multiple loads, which are generally synchronous, as well as polluting the global name space with references to the loaded modules. It is nonetheless a popular paradigm for the quick & dirty proof-of-concept phase of a project. It is in this spirit that we offer a Hypergrid build file on our CDN, which includes `Hypergrid.require`.

As the project matures, however, more serious development will demand bundling the files ahead of time (with Browserify, webpack, _etc._) into a single loadable module.

Wrapped modules (_i.e., Hypergrid Client Modules) can help to ease this eventual transition because they make available on the client the same CommonJS conventions as used by the bundlers (`require`, `module.exports`, and `exports`). That is, except for the single-line wrapper, the files conform to the same module pattern as the bundler requires. Therefore, to make the transition, simply unwrap the files and they are ready for the bundler.

`hyperwrap` is a very simple bash script that wraps a JavaScript file as a Hypergrid Client Module.<br>
`hyperunwrap` (see) unwraps them.

### Installation
In a bash shell:
```bash
curl https://fin-hypergrid.github.io/script >hyperloadify
```
(See also [global installation](#global-installation) instructions.)

### Synopsis
```
sh hyperloadify [ key ] <input >output
```

### Usage
```bash
sh hyperloadify mod <mod.js >wrapped-mod.js # wraps module with key "mod"
sh hyperloadify <index.js >wrapped-index.js # main module has no key
```

### Utilizing wrapped files
Your app loads Hypergrid first, followed by each wrapped app module, ending with the main module:
```html
<script src="https://fin-hypergird.github.io/core/3.0.0/build/fin-hypergrid.js"></script>
<script src="build/mod.js"></script>
<script src="build/index.js"></script>
```
All wrapped modules are stored so they can be required by subsequently loaded modules. The main module `requires` all the others, either directly or indirectly, and typically instantiates the grid(s) on the `window.load` event.

### Theory of Operation
The output of `hyperloadify modname` looks like this:
```js
fin.Hypergrid.modules['modname']=(function(require,module,exports){
// (the contents of src/mod.js appear here)
;return module.exports})(fin.Hypergrid.require,fin.$={},fin.$.exports={});delete fin.$;
```
Notes:
* The module is a "closed over" [IIFE](https://en.wikipedia.org/wiki/Immediately-invoked_function_expression)
* The module's output (`module.exports`) is stored into `fin.Hypergrid.modules` for access by other modules
* `require()` is available for use inside the module, thus providing access to other loaded modules (actually calls `fin.Hypergrid.require(name)` which returns `fin.Hypergrid.modules[name]`)
* When key is not specified, the output is the same except that the output is not stored (the [L-value](https://en.wikipedia.org/wiki/L-value) in the above is omitted).

### Predefined modules
In addition to your previously loaded wrapped modules, the following predefined modules are available to `Hypergrid.require`:

Module | Description
--- | ---
`var Hypergrid = require('fin-hypergrid')`|The Hypergrid constructor.
`var Hypergrid = require('fin-hypergrid/src/...')`|...
`var Hypergrid = require('datasaur-base')`|Useful for building your own data model.
`var Hypergrid = require('datasaur-local')`|Useful for extending the default data model.

### Global installation
Making the installation global has two advantages:
1. The script can be run from any folder
2. The script may be run using its name alone (without preceding it with `sh`)

To convert to a global installation:
```bash
chmod +x hyperloadify # make executable (the script starts with the required #!/bin/bash "shebang" line)
mv hyperloadify /usr/local/bin # or some other folder in $PATH such as $HOME/bin
```
