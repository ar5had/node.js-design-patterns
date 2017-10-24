# node.js-design-patterns
Examples and exercises of Book - Node.js Design Patterns, Second Edition

## Chapter 1

### Immutable object
 If you want to create immutable object, **const** is not enough, so you should use ES5's method `Object.freeze()` or deep-freeze module.


### Getters and Setters
```js
const person = {
  	name: 'ashraf',
  	surname: 'khan',
	get fn() {
    	return this.name + ' ' + this.surname;
    },
  	set fn(fullName) {
		[this.name, this.surname] = fullName.split(' ');
	}
};
console.log(person.fn);
console.log(person.fn = 'arshad khan');
console.log(person.surname, person.name);
```
It's worth noticing that the second call to console.log prints `arshad khan`. This happens because set function returns the value that is returned by the get function for the same property, in this case `get fn`.


### Maps
One new feature that makes map really interesting is the possibility of using functions and objects as keys of map, and this is something that is not entirely possible using plain objects, because objects auto. casts all the keys to string. This opens new opportunities; for example, we can build a micro testing framework leveraging this feature:
```js
const tests = new Map();
tests.set(()=>2+2, 4);
tests.set(()=>2-2, 0);
for(const entry of tests) {
	console.log(entry[0]() === entry[1]);
}
```
**Note:** All the entries respect the order in which they are inserted.

## Chapter 2

### Defining globals
Even if all the variables and functions that are declare in a module are defined in its local scope, it i still possible to define az global variable. In fact, the module system exposes a special variabble called `global`, which can be used for this purpose. Everything that is assigned to this variable will end up automatically in the global scope.

### module.exports vs exports
The variable `export` is just a reference to the initial value of `module.exports`; we have seen that such a value is essentially a simple object literal created before the module is loaded.

### `require` function is synchronous
Therefore, this code is wrong -
```js
setTimeout(()=>{
    module.exports = () {...};
}, 100);
```
### `require.resolve(pathString)`
This method can be invoked to resolve module from path string.

### `require.cache`
The module cache is exposed via the `require.cache` variable, so it is possible to directly access it if needed. A common use case is to invalidate any cahced module by deleting the  relative key in the `require.cache` variable, a practive very useful during testing but very dangerous if applied in normal cicumstances.

### Circular Dependencies
Module a.js:
```js
"use strict";

exports.loaded = false;

const b = require('./b');

module.exports = {
  bWasLoaded: b.loaded,
  loaded: true
};
```

Module b.js:
```js
"use strict";

exports.loaded = false;

const a = require('./a');

module.exports = {
  aWasLoaded: a.loaded,
  loaded: true
};
```

Module main.js:
```js
"use strict";

const a = require('./a');
const b = require('./b');
console.log(a);
console.log(b);
```

Results:
```
{bWasLoaded: true, loaded: true}
{aWasLoaded: false, loaded: true}
```

If main.js was -
```js
"use strict";

const b = require('./b');
const a = require('./a');
console.log(a);
console.log(b);
```
then results will be -
```
{bWasLoaded: false, loaded: true}
{aWasLoaded: true, loaded: true}
```

### Substack Pattern
```js
module.exports = (messsage) => {
    console.log(`info: ${message}`);
};

module.exports.verbose = (message) => {
    console.log(`verbose: ${message}`);
};

```

Expose the main functionality of a module by exporting only one function. Use the exported function as  namespace to expose any auxiliary functionality.

### Exporting a constructor or factory
```js
// file logger.js
function Logger(name) {
    this.name = name;
}

Logger.prototype.log = function(message) {
    console.log(`[${this.name}]: ${message}`);
};

// file main.js
const Logger = require('./logger');
const dblogger = new Logger('DB');
dblogger.log('Some info!');
```

A variation of this pattern consists of applying a guard against invocations that doesn't use `new` instruction. This little trick allows us to use our module as a factory.
```js
// es5
function Logger(name) {
    if(!(this instanceof Logger)) {
        return new Logger(name);
    }

    this.name = name;
}

// es6
function Logger(name) {
    if(!new.target) {
        return new Logger(name);
    }

    this.name = name;
}
```

### Exporting an instance
Since the module is cached, every module that requires the `logger` module shares the same instance of the object, thus sharing its state. This pattern is very much like creating a singleton; however, it doesn't guarantee the uniqueness of the instance across the app, as it happens in traditional singleton pattern.
When analyzing the resolving alog, we have seen in fact, that a module might be installed multiple times inside the dependency tree of an app. This results in multiple  instances of the same logical module all running in the context of the same node.js app.

Moral of the story: don't export instances but export constructor instead.

