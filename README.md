# mock-require

#### Simple, intuitive mocking of Node.js modules.

[![Build Status](https://travis-ci.org/boblauer/mock-require.svg)](https://travis-ci.org/boblauer/mock-require)

## About

mock-require is useful if you want to mock `require` statements in Node.js.  I wrote it because I wanted something with a straight-forward API that would let me mock anything, from a single exported function to a standard library.

## Usage

```javascript
var mock = require('mock-require');

mock('http', { request: function() {
  console.log('http.request called');
}});

var http = require('http');
http.request(); // 'http.request called'
```

## API

### `mock(path, mockExport)`

__path__: `String`

The module you that you want to mock.  This is the same string you would pass in if you wanted to `require` the module.

This path should be relative to the current file, just as it would be if you were to `require` the module from the current file.  mock-require is smart enough to mock this module everywhere it is required, even if it's required from a different file using a different relative path.

__mockExport__ : `object/function`

The function or object you want to be returned from `require`, instead of the `path` module's exports.

__mockExport__ : `string`

The module you want to be returned from `require`, instead of the `path` module's export.  This allows you to replace modules with other modules.  For example, if you wanted to replace the `fs` module with the `path` module (you probably wouldn't, but if you did):

```javascript
mock('fs', 'path');
require('fs') === require('path'); // true
```
This is useful if you have a mock library that you want to use in multiple places.  For example:

`test/spy.js`:
```javascript
module.exports = function() {
    return 'this was mocked';
};
```

`test/a_spec.js`:
```javascript
var mock = require('mock-require');
mock('../some/dependency', './spy');
...
```

`test/b_spec.js`:
```javascript
var mock = require('mock-require');
mock('../some/other/dependency', './spy');
...
```

### `mock.stop(path)`

__path__: `String`

The module you that you want to stop mocking.  This is the same string you would pass in if you wanted to `require` the module.

This will only modify variables used after `mock.stop` is called.  For example:

```javascript
var mock = require('mock-require');
mock('fs', { mockedFS: true });

var fs1 = require('fs');

mock.stop('fs');

var fs2 = require('fs');

fs1 === fs2; // false
```

### `mock.stopAll()`

This function can be used to remove all registered mocks without the need to remove them individually using `mock.stop()`.

```javascript
mock('fs', {});
mock('path', {});

var fs1 = require('fs');
var path1 = require('path');

mock.stopAll();

var fs2 = require('fs');
var path2 = require('path');

fs1 === fs2; // false
path1 === path2; // false
```

### `mock.reRequire(path)`

__path__: `String`

The filepath for a module whose cache you want to refresh.

Imagine you are trying to test a file "PARENT" which has already been `require`d previously (e.g. in an earlier test file) and is therefore cached by Node. You want to mock one of PARENT's dependencies ("DEP"). Unfortunately, when you `require` PARENT, it fetches the cached PARENT with the old DEP, so your mock ("MOCK_DEP") has no effect. Calling `mock.reRequire('./path/to/PARENT')` forces PARENT to reload all of its dependencies, this time including MOCK_DEP.

```javascript
var fs = require('fs');
var fileToTest = require('./fileToTest'); //requires fs as a dependency
mock('fs', {}); // fileToTest is still using the unmocked fs module

fileToTest = reRequire('./fileToTest'); // fileToTest is now using your mock
```

Note that if PARENT requires dependencies that in turn require DEP, those dependencies will still have the unmocked version. You must `mock.reRequire` all modules whose caches you want to refresh. A relatively safe approach is to `mock.reRequire` or `mock` all of PARENT's dependencies, although you may skip any dependencies you are certain do not have DEP in their dependency tree.

```javascript
var fs = require('fs');
var anotherDep = require('anotherDep') // requires fs as a dependency
var fileToTest = require('./fileToTest'); // requires fs and anotherDep as a dependency
mock('fs', {}); // fileToTest is still using the unmocked fs module
mock('anotherDep', {}); // do this to make sure fs is being mocked consistently

fileToTest = reRequire('./fileToTest');
```

## Test

```
npm test
```
