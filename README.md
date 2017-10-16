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

###