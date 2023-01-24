### Built-in Modules
- Modules that node.js ships with
- Also referred to as core modules
- Import the module before you can use it
- path
- events
- fs
- stream
- http
## Path Module
- The path module provides utilities function to work with files and directories paths
- Path module has 14 utilities function but 7 are used commonly and we will look those seven methods
- The `path` module in Node.js is a built-in module that provides a way to work with file and directory paths. It provides a way to join, resolve, and normalize file paths, as well as other useful path-related functions
- When importing a built in - module the node prefix is optional even if we omit node: and save the file it will work perfectly 
- however node protocol has many benefits first it makes it perfectly clear that the import is a node.js built-in module begginers don't always realize this second it makes the import identifier a valid absolute URL third it avoids conflicts for future Node.js built-in module 
- The `path` module in Node.js is a built-in module that provides a way to work with file and directory paths. It provides a way to join, resolve, and normalize file paths, as well as other useful path-related functions

1.  `path.extname(path)`: This function returns the file extension of the path passed as the argument. For example:
~~~js
const path = require('path'); 
console.log(path.extname('path/to/my/file.txt')); // Output: '.txt'`
~~~


2.  `path.join(path1, path2, ...pathN)`: This function joins all the given paths together and normalizes the resulting path. It can be used to join multiple paths together in a cross-platform way. For example:

~~~js
const path = require('path'); 
console.log(path.join('path', 'to', 'my', 'file')); // Output: 'path/to/my/file' (on a unix-like system)`
~~~

3.  `path.resolve(path1, path2, ...pathN)`: This function resolves a sequence of paths or path segments into an absolute path. It will take into account the current working directory and resolve any relative path segments. For example:
~~~js
const path = require('path'); 
console.log(path.resolve('path', 'to', 'my', 'file')); // Output: '/path/to/my/file' (on a unix-like system)`
~~~

4.  `path.basename(path, [ext])`: This function returns the last portion of a path, similar to the Unix basename command. It can also return the base name of the file without the extension by passing the extension as the second argument. For example:

~~~js
const path = require('path');
console.log(path.basename('path/to/my/file.txt')); // Output: 'file.txt' console.log(path.basename('path/to/my/file.txt', '.txt')); // Output: 'file'`
~~~

5.  path.parse(path)`: This function returns an object containing the different parts of the path, such as the root, dir, base, ext, and name. For example:

~~~js
const path = require('path'); 
console.log(path.parse('path/to/my/file.txt')); /* Output: {    root: '',   dir: 'path/to/my',   base: 'file.txt',   ext: '.txt',   name: 'file'  } */`
~~~

6.  `path.isAbsolute(path)`: This function returns a boolean indicating whether the given path is an absolute path or not. For example:
~~~js
const path = require('path'); 
console.log(path.isAbsolute('/path/to/my/file')); // Output: true console.log(path.isAbsolute('path/to/my/file')); // Output: false`
~~~


7.  `path.format(pathObject)`: This function returns a path string from an object that was previously returned by path.parse(). For example:

~~~js
const path = require('path'); 
const pathObject = {      
	root: '',     
	dir: 'path/to/my',     
	base: 'file.txt',     
	ext: '.txt',     
	name: 'file'  
}; 
console.log(path.format(pathObject)); // Output: 'path`
~~~

### -------------------CALLBACK PATTERNS-------------------
- In JavaScript, Functions are first class objects
- A function can be passed as an argument to a function
- A function can also be returned as value from other function
- A function which is passed as an argument is called as callback function in Js
- A function which accept function as a prameter or return function is called as Higher Order Function in Js

### Types of Callbacks
- Synchronous Callbacks
	- A callback which is executed immediately is called a synchronous callback
	- most practical examples are functions passed to sort, map and reduce functions 
- Asynchronous Callbacks
	- A callback that is often used to continue or resume code execution after an asynchrnous operation has completed
	- In Node.js have an asynchronous nature to prevent blocking of execution
	- Ex. reading data from a file, fetching data from a databases or handling a network request 
![[Pasted image 20230121205308.png]]


### ---------------- Events Module----------------------
- The events module allows us to work with events in Node.js
- An event is an action or an occurence that has happened in our application that we can respond to 
- using the events module, we can dispatch our own custom events and respond to those custom events in a non - blocking manner.

To understand events module better let's have an example of real world scenario

- Let's say you're feeling hungry and head out to Dominos to have Pizza
- At the counter, you place your ordre of pizza
- When you place the order, the line cook sees the order on the screen and bakes the pizza for you
- Order being placed is the event
- Baking a pizza is a response to that event 

~~~js
const EventEmitter = require('node:events');
const emitter = new EventEmitter();

emitter.on("order-pizza",(size, topping)=>{

    console.log(`Order received! Baking a ${size} pizza with ${topping}`);

})
emitter.emit("order-pizza",'large','mushroom')
~~~
- We can register multiple events for same event below I have given example

~~~js
const EventEmitter = require('node:events');
const emitter = new EventEmitter();

emitter.on("order-pizza",(size, topping)=>{
    console.log(`Order received! Baking a ${size} pizza with ${topping}`);
})
emitter.on("order-pizza",(size)=>{
	if(size===large){
		console.log("serving complimentary drink")
	}
})
emitter.emit("order-pizza",'large','mushroom')
~~~

Output:
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication
$ node events.js 
This log is before event happening
Order received! Baking a large pizza with mushroom
serving complimentary drink
~~~


### -------------EXTENDING FROM EVENEMITTER CLASS----------
- Our own classes can extend event module and and can have their own custom events and then respond to those events.
- I have demonstrated this behaviour with exmaple below 
~~~js 
// pizza-shop.js file
const EventEmitter = require('node:events');
class PizzaShop extends EventEmitter {
    constructor(){
        super()
        this.orderNumber = 0;
    }
    order(size,topping){
        this.orderNumber++;
        this.emit('order',size, topping)
    }
    displayOrderNumber(){
        console.log('Current Order Number: ',this.orderNumber )
    }
}

module.exports = PizzaShop;
~~~

~~~js
// drink-Machine.js
class DrinkMachine{
    serveDrink(size){
        if(size == 'large'){
            console.log('Hey Your Drink is Complimentary 🍻')
        }else{
            console.log('Here is your drink 🍺')
        }
    }
}
module.exports = DrinkMachine;
~~~

~~~js
// index.js
const PizzaShop = require('./pizza-shop')
const DrinkMachine = require('./drink-Machine')

const pizzaShop = new PizzaShop();
const drinkMachine = new DrinkMachine();

pizzaShop.on('order',(size,topping)=>{
    console.log(`Order received! Baking a ${size} pizza with ${topping}`);
    drinkMachine.serveDrink(size)
})

pizzaShop.order('large','topping')
pizzaShop.displayOrderNumber();

console.log('-------------------------------')

pizzaShop.order('small','topping')
pizzaShop.displayOrderNumber();
~~~

Output
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication
$ node index.js 
Order received! Baking a large pizza with topping
Hey Your Drink is Complimentary �
Current Order Number:  1
-------------------------------
Order received! Baking a small pizza with topping
Here is your drink �
Current Order Number:  2
~~~

### ---------CHARACTER SETS AND ENCODINGS--------------

### Now let's learn few others concepts
- character sets
- Encoding
- Streams and buffers
- Asynchronous JavaScript

**Binary Data** - Computers store and represent data in binary format which is collection of 0s and 1s. 
Each 0 or 1 is called a binary digit or bit for short 
To work with a piece of data, a computer needs to convert that data into its binary representation. 

**Character in binary Format**
V?
- Computers will first convert the character to a number, then convert that number to its binary representation
- Computers will first convert V to a number that represents V
- 86 is the numeric representation of the character V
- It is also called character code
- how does the computer know V should be represented as 86.?

~~~js
// to know that number of any character you call use method charCodeAt() in browser terminal

"V".charCodeAt();
~~~

**Character Sets** - Character sets are predefined list of characters represented by numbers Popular sets - Unicode and ASCII

**Character Encoding** - 
- Character encoding dictates how to represent a number in a character set as binary data before it can be stored in a computer. 
- It dictates how many bits to use to represent the number.
- Example of a character encoding system is UTF-8 
- UTF-8 states that characters should be encoded in bytes (8 bits)
- Eight 1s or 0s should be used to represent the code or any character in binary .
- 4 ==> 100 ==> 00000100
- V ==> 86 ==> 01010110
- Similar guidelines also exist on how images and videos should be encoded and stored in binary format.
### summary
- Binary data - 0s and 1s that computers can understand
- Character sets - Predefined lists of characters represented by numbers
- Character encoding - Dictates how to represent a number in a character set as binary data

### -------------STREAMS AND BUFFERS--------------------

### Streams 
- A **Stream**  is a sequence of data that is being moved from one point to another over time.
- Ex. a stream of data over the internet being moved from one computer to anther.
- Ex. a stream of data being trasferred from one file to another within the same computer.
- In node.js the idea is to Process streams of data in chunks as they arrive instead of waiting for the entire data to be available before processing.
- Ex. watching a video on YouTube.
- The data arrives in chunks and you watch in chunks while the rest of the data arrives over time.
- Ex. transfering file contents from fileA to FileB.
- prevents unnecessary data downloads and memory usage



### **Buffers**
- To understand buffers consider a scenario in park 
we have a Roller Coster![[Pasted image 20230107035740.png]]

30 - Seating capacity 
Scenario 1 
100- People arrival
30 - People accomodated
70 - People in queue (waiting)

Scenario 2
1 - Person arrives (waiting)
Wait for at least 10

- The bottom line or crux is you cannot control the pace at which people comes you can only decide when is the right time to send people on ride.

- If people are already on ride or there are very few people then they have to wait as it turns out
- the Area where peple wait is nothing but the buffer.
- Node.js cannot control the pace at which data arrives in the stream 
- It can only decide when is the right time to send the data for processing.
- If there is data already processed or too little data to process, Node puts the arriving data in a buffer
- It is an intentionally small are that Node maintains in the runtime to process a stream of data. 
- Ex Streaming a video online 
- If your internet connection is fast enough, the speed of the stream will be fast enough to instantly fill up the buffer and send it out for processing
- That will repeat till the stream is finished
- If your connection is slow, after processing the first chunk of data that arrived, the video player will display a loading spinner which indicates it is waiting for more data to arrive
- Once the buffer is filled up and the data is processed, the video player shows the video
- While the video is playing, more data will continue to arrive and wait in the buffer
- Now what is the connection between binary data, character set, character encoding, stream and buffer ?
~~~js
const buffer = new Buffer.from("Vishwas","utf-8");
console.log(buffer.toJSON())
console.log(buffer) // contains raw binary data 
console.log(buffer.toString())
~~~
output 
~~~bash 
Abhijeet@Abhijeet-Singh MINGW64 /d/pepcoding backend/Node built in modules
$ node path.js 
{
  type: 'Buffer',
  data: [
     86, 105, 115,
    104, 119,  97,
    115
  ]
}
<Buffer 56 69 73 68 77 61 73>
Vishwas
~~~
- A Buffer contains raw binary data which is displayed as output when logged to console. What nodejs does that is it prints the values in hexadecimal at console so that it is not filled with zeros and ones

- Buffer has limited memory what I mean is here we have created buffer from vishwas so it has memory of 7 bytes now if we try to write buffer with buffer.write('Code') it will only replace first 4 data and rest of the 3 will be same as it was earlier. similarly if I write codevolution it will only write first 7 and rest will be ignored.

### ------------------Asynchronous Javascript----------------

Javascript is a ==**synchronous, blocking, single-threaded language**==

**Problem with synchronous, blocking, single-threaded model of JavaScript**
- we have to wait for network requests that could take 1s or even 10s

**To Achieve Asynchronos Javascript**
- Just Javascript is not enough
- We need new pieces which are outiside of JavaScript to help us write asynchronous code
- For frontend, this is where web browsers come into play. For back-end, this is where Node.js comes into play
- Web browsers and Node.js define functions and APIs that allow us to register functions that should not be executed synchronously, and should instead be invoked asynchronously when some kind of event occurs.
- for example that could be the passage of time (setTimeout or setInterval), the user's interaction with the mouse (addEventListener), data being read from a file system or the arrival of data over the network (callbacks, Promises, async-await)
- You can let your code do several things at the same time without stopping or blocking 


Asynchronous JavaScript Summary
- JavaScript is a synchronous, blocking, single-threaded language
- This nature however is not beneficial for writing apps
- We want non-blocking asynchronous behaviour which is made possible by a  browser for FE and Node.js for backend
- This style of programming where we don't block the main JavaScript thread is fundamental to Node.js and is at the heart of how some of the built-in module code is written


### ------------------- Fs Module-----------------------
The file system module allows you to work with the file system on your computer

~~~js
// primarily we use 4 methods of fs module
let fs = require('node:fs')

// reading file
console.log(fs.readFileSync('path','option utf-8 etc.'))
fs.readFile('path','option','error first callback(err,data)=>{
			
}')

// Writing file 
fs.writeFileSync("path",'content',{flag: 'a'}) //flag option is used to append content otherwise it always overwrites

fs.writeFile('path','content',{flag: 'a'}, (err)=>{
	if(err){
		console.log(err)
	}else{
		console.log('file is written')
	}
})~~~

### -------------FS MODULE PROMISES-----------------
~~~js
const fs = require('node:fs/promises');

fs.readFile('path','utf-8')
.then(data=> console.log(data))
.catch(err=> console.log(err))

async function readFile(path){
	try{
		const data = fs.readFile('path','utf-8');
	}catch(err){
		console.log(err)
	}
}
~~~

- The callback version of fs module is preferred over promises module when  maximal performance is required both in terms of execution time and memory allocation but if that is not the major concern then stick to promises module.

### -----------------Stream in Node.js-------------------
- A stream is a sequence of data that is being moved from one point to another over time
- Ex: a stream of data being transferred from one file to another within the same computer
- Work with data in chunks instead of waiting for the entire data to be available at once
- If you're transferring file contents from fileA to fileB, you don't wait for entire fileA content  to be saved in temporary memory before moving it into fileB
- Instead, the content is transferred in chunks over time which prevents unnecessary memory usage
- Stream is infact a built-in node module that inherits from the event emitter class
- other modules internally use streams for their functioning
- buffers that stream uses has a default size of 64 kilobytes
- Another module that uses streams is http module http reques is readable stream while http response is writable stream 

~~~js
const fs = require('node:fs')
const readableStream = fs.createReadStream('./file.txt',{
    encoding: 'utf-8',
    highWaterMark: 2
})
const writableStream = fs.createWriteStream('./file2.txt')
let counter = 0;
readableStream.on('data',(chunk)=>{
    console.log(chunk,counter)
    counter++;
    writableStream.write(chunk)
})
// content of file.txt is hello codevolution 
~~~

Output: 
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication   
$ node index.js
he 0
ll 1
o  2
co 3
de 4
vo 5
lu 6
ti 7
on 8
  9
~~~


Types of Streams
- Readable streams from which data can be read
- Writable streams to which we can write data
- Duplex streams that are both Readable and Writable
- Transform streams that can modify or transform the data

Ex: Reading from a file as readable stream
Ex: Writing to a file as writable stream
Ex: Sockets as a duplex stream
Ex: File compression where you can write compressed data and read
de-compressed data to and from a file as a transform stream

###  ----------------Pipes in Node.js-----------------------

In non technical terms we know what pipe is e.g a pipe connects home tank to kitchen sink. Tank feeds watar into pipe which can be released through the tap in the sink. 
From a pipe point of view we are reading data from tank and writing into tap sink
 
In node.js a pipe is very similar it takes a redable stream and connects 
it to a writable stream we use pipe method on a readable stream to implement the functionality. 

![[Pasted image 20230122005513.png]]

~~~js
const fs = require('node:fs')
const readableStream = fs.createReadStream('./file.txt',{
    encoding: 'utf-8',
    highWaterMark: 2
})
const writableStream = fs.createWriteStream('./file2.txt')
readableStream.pipe(writableStream)
~~~

The great thing about pipe is that it returns destination stream which enables chaining however the destination stream should be readable, duplex or transform stream 


### ------------------- http module in Node.js--------------------

- Before learning http module let's look at first how web works  
### How Web Works
- Computers connected to the internet are called clients and servers 
- Clients are internet-connected devices such as computers or mobile phones along with web-accessing software available on those devices such as a web browser
- Servers on the other hand are computers that store web pages, sites, or apps
- When you type url in browser client device request to server 
- A copy of web page is dowloaded from server and sent as a response to dispalay to the client's browser 
- This model is popularly know as clien server model 
- Now we understand data is transfered between client and server but in what format data is being transfered?
- what if request sent by client is not understood by server and vice-versa?
- Well this is where http comes into picture 
- HTTP stand for hyper text transfer protocol 
- A protocol that defines a fromat for clients and server to speak to each other 
- The client sends an HTTP request and the server responds with an HTTP response 

### HTTP and Node
- We can create a web server using Node.js 
- Node.js has access to operating system functionality like Networking 
- Node has an event loop to run tasks asynchronously and is perfect for creating web servers that can simultaneously handle large volumes of requests 
- The node server we create should still respect the HTTP format
- The HTTP module allows creation of web servers that can transfer data over HTTP 


### -----------------CREATING A SERVER USING NODE--------

~~~js
const http = require('node:http')
const server = http.createServer((req,res)=>{
    res.writeHead(200,{"Content-type": "text/plain"});
    res.end("Hello Lodu")
})
server.listen(5000,()=>{
    console.log('server is listening')
})
~~~

### Summary 
- First we require http module
- Then we use createServer method to create server
- Then we listen created server to port 

### -----------JSON RESPONSE-------------------

~~~js
const http = require('node:http')
const server = http.createServer((req,res)=>{
    const superHero = {
        firstName: "Shakti",
        lastName: "Maan"
    }

    res.writeHead(200,{"Content-type": "Application/JSON"});
    res.end(JSON.stringify(superHero))
})
server.listen(5000,()=>{
    console.log('server is listening')
})
~~~

### -------------HTML RESPONSE-------------------
~~~js
const http = require('node:http')
const server = http.createServer((req,res)=>{
   
    res.writeHead(200,{"Content-type": "text/html"});
    res.end('<h1>Hello Lodu</h1>')
})
server.listen(5000,()=>{
    console.log('server is listening')
})
~~~

### Sending html file

~~~js
const http = require('node:http')
const fs = require('node:fs')

const server = http.createServer((req,res)=>{
    const html = fs.readFileSync('./index.html','utf-8')
    res.writeHead(200,{"Content-type": "text/html"});
    res.end(html)
})

server.listen(5000,()=>{
    console.log('server is listening')
})
~~~

### ------------------ HTML Template------------------------
- It is done using templating engines like pug or EJS
- here for our understanding we will use most basic solution that is string replacement
~~~js
const http = require('node:http')
const fs = require('node:fs')

const server = http.createServer((req,res)=>{
    let html = fs.readFileSync('./index.html','utf-8')
    html = html.replace('{{name}}','Abhijeet')
    res.writeHead(200,{"Content-type": "text/html"});
    res.end(html)
})
server.listen(5000,()=>{
    console.log('server is listening')
})
~~~

### -----------------------ROUTING------------------------
- To do routing we can use req.url it returns requested url then we can use this url using swithc statements or if-else if statements  to match and responds to different requested URLs
- Combination or req.url and req.method can help with doing any possible routing
- req.method returns requested method type get, post, put, delete 
- But in real world application we typically relly on web framework express is popular framework for Node.js


### ------------------WEB FRAMEWORK-------------------

### Http Module so far
- How to create a server
- How to access the request information
- How to send a response
- How to modify the response header
- How to set a response status
- How to responds with plain text, HTML (templates), JSON
- How to route requests to different application 

### Web Framework
- A framework simply abstracts the lower level code allowing you to focus on the requirements than the code itself
- For example, Angular, React, Vue are all framework/libraries that help you build user interface without having to rely on the lower level DOM API in JavaScript
- There are frameworks to build web or mobile applications without having to rely on the HTTP module in node.js
- Ex. express,nest,hapi,koa and sails
- They build on top of the HTTP module making it easier for you to implement all the features 

### Section Summary
- Built in modules
- Path module
- Callback patters
- Events module
- EventEmitter class
- Character sets and encoding
- Streams and buffers
- Asynchronous JavaScript
- fs module
- Streams
- Pipes
- HTTP module
