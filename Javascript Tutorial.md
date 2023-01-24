
## Data types in javascript

- ### primitive types
	- Numbers
	- Strings
	- Boolean
	- Undefined
	- Null
	- Symbol
	- Big Int
- ### Non primitve types
	- Objects

### Advance Concepts in javascript
- Nested function Scope
- Closures
- currying
- this keyword
- Prototype
- Protototypal Inheritance
- Class
- Iterables and Iterators
- Generators


### Scope
- Block Scope
- Function Scope
- Global Scope

### Now let's learn nested function scope


~~~javascript

function outer(){

	let b = 20;
	fucntion inner(){
		let c = 30;
		console.log(b, c);
	}
}
~~~

### closures

Functions bundled together with its lexcial scope is closures.
In JavaScript, when we return a function from another function, we are effectively returning a combination of the function definition along with the function's scope. This would let the
function definition have an associated persistent memory which could hold on to live data
between executions. That combination of the function and its scope chain is what is called a
closure in JavaScript.

### Function Currying
Currying is a process in functional programming in which we transform a function with multiple arguments into a sequence of nesting functions that take one argument at a time.
function f(a,b,c) is transformed to f(a)(b)(c)

~~~javascript
function sum(a,b,c){
    return a+b+c;
}

function curry(fn){

    return function(a){

        return function(b){

            return function(c){

                return fn(a,b,c);

            }

        }

    }

}
let curriedFunction = curry(sum);
console.log(curriedFunction(10)(20)(30))
~~~
### this
The JavaScript this keyword which is used in a function, refers to the object it belongs to
It makes functions reusable by letting you decide the object value
this value is determined entirely by how a function is called

### How to determine 'this'?
Implicit binding
Explicit binding
New binding
Default binding

### Implicit Binding
~~~javascript 
const person = {
	name: 'Vishwas',
	sayMyName: function () {
		console.log(My name is ${this.name}`)
	},
}
person.sayMyName()
~~~

### Explicit Binding

~~~javascript
const person = {
	name: 'Vishwas',
	sayMyName: function () {
		console.log(My name is ${this.name}`)
	},
}
function sayMyName() {
console.log(My name is ${this.name}`)
} I
sayMyName.call(person)
~~~

### New Binding

~~~javascript
function Person (name) {
	// here this refers to {} empty object when it is called with new kewyword

this.name = name
}
const p1 = new Person ('Vishwas')
const p1 = new Person('Vishwas')
~~~


### Default Binding
~~~javascript
function sayMyname(name){
	console.log(name, this);
}
// when no above theree binddings are find then this keyword refers to global object

sayMyname("Abhijeet");
~~~

### Order of precedence of this keyword
- New binding
- Explicit binding
- Implicit binding
- Default binding


# Prototypes

In javascript every function has a property called prototype which points to a object; 
we can use that prototype object to share all sharable properties and functions. 

~~~js
fucntion Person(name1,name2){
	this.name1 = name1;
	this.name2 = name2;
}
const person1 = new Person ('Bruce', 'Wayne')
const person2 = new Person ('Clark', 'Kent')
// by setting getFullName to prototype every instance that is created from Person constructor function can access this property now. 

Person.prototype.getFullName = function () {
	return this.firstName + + this.lastName
1
}
console.log(person2.getFullName())
console.log(person2.getFullName())

~~~

The another user of prototype is inheritance.
