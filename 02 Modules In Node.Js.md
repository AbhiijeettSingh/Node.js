
## Modules
- A module is an encapsulated and reusable chunk of code that has its own context
- In Node.js each file is treated as a separate module

### Types of Modules
1. Local modules - Modules that we create in our application
2. Built-in Modules - Modules that Node.js ships with out of the box.
3. Third party modules - Modules written by other developers that we can use in our application


### ---------------LOCAL MODULES--------------------

### CommonJs
- CommonJs is a standard that states how a module should be structured and shared
- Node.js adopted CommonJs when it started out and is what you will see in code bases

~~~js
// add.js 
const add = (num1, num2)=>{
	return num1 + num2;
}
console.log(add(2,5))
~~~

~~~js
// index.js
require('./add') // require function runs the moudle of which path is given
console.log('hello from index.js');

// output will be 
// 7
// hello from index.js
~~~

### Local Modules Summary
- In node.js each file is a module that is isolated by default
- To load a module into another file, we use the require function
- When index.js is executed, the code in the module is also executed
- If the file we are requiring is a JavaScript file, we can skip specifying the extension and node.js will infer it on our behalf


### ------------MODULE EXPORTS---------------------

- The value of modue.exports is the value that is returned by require function when it is called.
- By default value of module.exports = { } // empty object


### ------------MODULE SCOPE----------------------

- Each module in Node.js has its own scope and by which node.js achieve this is immediately invoked functions IIFE 
~~~js
(function(){
	console.log("Hey I am IIFE Function");
})();
~~~

- With IIFES each function gets its own private scope
- Under the hood Node.js also uses the same for modules

### Immediately Invoked Function Expression (IIFE) in Node.js
- Before a module's code is executed, Node.js will wrap it with a function wrapper that provides module scope
- This saves us from having to worry about conflicting variables or functions
- there is proper encapsulation and reusability is unaffected

### Module Scope Summary
- Each loaded module in Node.js is wrapped with an IIFE that provides private scoping of code
- IIFE allows you to repeat variable or function names without any conflicts

### ----------MODULE WRAPPER---------------------
- Every module in node.js gets wrapped in an IIFE before begin loaded 
- IIFE helps keep top-level variables scoped to the module rather than the global object
- The IIFE that wraps every module contains 5 paramets which are pretty important for the functioning of a module. 

~~~js
//MODULE WRAPPER
( function(exports, require, module, __filename, __dirname){
	// code written by user module
})
~~~

~~~js
console.log('Module ---> ',module)
console.log('__dirname ---> ',__dirname)
console.log('__filename ---> ',__filename)
console.log('require ---> ',require)
console.log('exports ---> ',exports)
~~~
Output 
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication
$ node iife.js 
Module --->  Module {
  id: '.',
  path: 'C:\\Users\\abhij\\OneDrive\\Desktop\\Authentication',
  exports: {},
  filename: 'C:\\Users\\abhij\\OneDrive\\Desktop\\Authentication\\iife.js',
  loaded: false,
  children: [],
  paths: [
    'C:\\Users\\abhij\\OneDrive\\Desktop\\Authentication\\node_modules',
    'C:\\Users\\abhij\\OneDrive\\Desktop\\node_modules',
    'C:\\Users\\abhij\\OneDrive\\node_modules',
    'C:\\Users\\abhij\\node_modules',
    'C:\\Users\\node_modules',
    'C:\\node_modules'
  ]
}
__dirname --->  C:\Users\abhij\OneDrive\Desktop\Authentication
__filename --->  C:\Users\abhij\OneDrive\Desktop\Authentication\iife.js
require --->  [Function: require] {
  resolve: [Function: resolve] { paths: [Function: paths] },
  main: Module {
    id: '.',
    path: 'C:\\Users\\abhij\\OneDrive\\Desktop\\Authentication',
    exports: {},
    filename: 'C:\\Users\\abhij\\OneDrive\\Desktop\\Authentication\\iife.js',
    loaded: false,
    children: [],
    paths: [
      'C:\\Users\\abhij\\OneDrive\\Desktop\\Authentication\\node_modules',
      'C:\\Users\\abhij\\OneDrive\\Desktop\\node_modules',
      'C:\\Users\\abhij\\OneDrive\\node_modules',
      'C:\\Users\\node_modules',
      'C:\\node_modules'
    ]
  },
  extensions: [Object: null prototype] {
    '.js': [Function (anonymous)],
    '.json': [Function (anonymous)],
    '.node': [Function (anonymous)]
  },
  cache: [Object: null prototype] {
    'C:\\Users\\abhij\\OneDrive\\Desktop\\Authentication\\iife.js': Module {
      id: '.',
      path: 'C:\\Users\\abhij\\OneDrive\\Desktop\\Authentication',
      exports: {},
      filename: 'C:\\Users\\abhij\\OneDrive\\Desktop\\Authentication\\iife.js',
      loaded: false,
      children: [],
      paths: [Array]
    }
  }
}
exports --->  {}
~~~

### ---------MODULE CACHING-----------------
- In node.js when we require a new module it is loaded and cached for subsequent loading
- cached is fancy name for remember so what actually happens is that when we require any module for the first time whatever is returned by the require function it is remembered by the node.js and whenever we require same modle again the thing is which is returned earlier is again returned and the entire module is not loaded and run again. 
- when we require module again so node.js will think hey I remembered this module has already loaded before let me reuse that instead of doing the additional work of loading it brand new in our case it is going to load the same object as line one 

### -------------- IMPORT EXPORT PATTERNS---------

~~~Js
// 1st pattern
const add = (a,b)=>{
	return a+b;
}

module.exports = add;
~~~


~~~js
// 2nd Pattern
module.exports.add = (a,b)=>{
	return a+b; 
}
~~~

~~~js
// 3rd pattern
const add = (a,b)=>{
	return a+b;
}
const substract = (a,b)=>{
	return a-b;
}
module.exports = { sum, substract};
~~~

~~~js
// 4th pattern
module.exports.add = (a,b)=>{
	return a+b;
}
module.exports.substract = (a,b)=>{
	return a-b;
}
~~~

~~~js
// 5th pattern
// initially in every module module.exports = exports = {} empty object;
exports.add = (a,b)=>{
	return a+b;
}
exports.substract = (a,b)=>{
	return a-b;
}
~~~

### --------MODULE.EXPORTS VS EXPORTS------------

~~~js
// never do
exports = { sum, substract}
// because initially module.exports = exports = {} but when we assign exports = {sum, substract} that reference is broken and exports = {sum, substract} but module.exports is still empty object {};
~~~

Example demonstrating this concept using code
~~~js
const sum = (a,b)=>{
        return a+b;
}
const substract = (a,b)=>{
    return a-b;
}

console.log(exports)
console.log(module.exports)
console.log(module.exports == exports)

exports = {sum, substract};

console.log(exports)
console.log(module.exports)
console.log(module.exports == exports)
~~~

Output 
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication
$ node index.js
{}
{}
true
{ sum: [Function: sum], substract: [Function: substract] }
{}
false
~~~


### ---------------------ES MODULES-------------------------

CommonJs
 - Each file is treated as a module
 - variables, functions, classes, etc. are not accessible to other files by default
 - Explicitly tell the module system which parts of your code should be exported via module.exports or exports
 - To import code into a file, use the require() function


 ES Modules 
  - At the time Node.js was created, there was no built-in module system in JavaScript
  - Node.js defaulted to CommonJs as its module system 
  - As of ES2015, JavaScript does have a standardized module system as part of the language itself
  - That module system is called EcmaScript Modules or ES Modules or ESM for short
  - File extension for ESM in node.js is .mjs 

Export patterns in ESM 
 - 1st pattern
    ~~~js 
    const add = (a,b)=>{
	    return a+b
	}
	export default add;
	~~~

 - 2nd pattern
  ~~~js
export default add = (a,b)=>{
	return a+b;
}
~~~

- 3rd pattern to export more than 1 thing as default 
~~~js
const add = (a,b)=>{
	return a+b;
}
const substract = (a,b)=>{
	return a-b;
}
export default = {
	add, 
	substract
}
~~~

These three are examples of default exports while importing we can import them as any name syntax to import is 
~~~js
import name from 'path'
~~~

In ESM we also have named exports where the variable or function name should be same while importing 
Example of names exports are 
~~~js
export const add = (a,b)=>{
	return a+b;
}

export const substract = (a,b)=>}
	return a-b;
}
~~~

We can import these in two ways 
~~~js
// 1st patter to import 
import * as math from './Math.mjs'

// 2nd pattern to import is destructuring 
import {add, substract} from './Math.mjs'
~~~

### ES Module Summary
- Es Modules is the ECMAScript standard for modules
- It was introduced with ES2015 
- Node.js 14 and above support  ES Modules
- Instead of module.exports, we use the export keyword
- The export can be default or named
- We import the exported variables or functions using import keyword
- If it is a default export, we can assign any name while importing
- If it is a named export, the import name must be the same

### -----------IMPORTING JSON----------------
- If we require json file in any js file using requuire the node.js will parse json data into javascript object 

For example let's say we have data.json file and index.js file and we will require json data.json file into index.js file

~~~json
// data.json file

{
	"name" : "abhijeet",
	"age" : 20,
	"education": "BCA",
	"Occupation": "Software engineer"
}
~~~

~~~js
// index.js file
const data = require('./data.json')
console.log(data)
~~~
 Output
 ~~~bash
 Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication
$ node index
{
  name: 'abhijeet',
  age: 20,
  education: 'BCA',
  Occupation: 'Software engineer'
}
~~~

- for json file also we can ignore .json extension while requiring but in that case node.js we look for js file with that name first and if it does not foud js file then it will look for JSON file. 

- Watch mode in node.js look for changes in file and if it finds any changes it runs the file again

### Section Summary 
- What is a module and what is the need for modules?
- Types of modules in Node.js
- Local modules
- CommonJs module format
- Module wrapper (IIFE)
- Module Caching
- Patterns for importing and exporting modules in CommonJs and ESM Format
- Importing JSON data and watch mode.
