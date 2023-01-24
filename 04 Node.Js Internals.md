## ------------------ LIBUV---------------------------
**What is libUV?**
- It is corss platform open source library written in c language.

Why do we need LibUV?
- libuv handles asynchronous non-blocking operation in Node.js. It abstract away all the complexity of dealing with the operating system.

How It handles Asynchrnous?
- With help of 
	- Thread Pool
	- Event loop


### ------------------- Thread Pool---------------------

To understand thread pool let's see the code from fs module which reads the contents of a file.

~~~js
const fs = require('node:fs')
console.log('First')

fs.readFile('./file.txt','utf-8',(err,data)=>{
	console.log('file contents')
})

console.log('last')
~~~

Output:
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/A2Z sheets/CppSTL
$ node path.js 
First
last
file contents
~~~

from this output it is clear that readFile() method is asynchronous and non-blocking. It is allowing the execution of code further down the line while the file contents are being read.

but how node.js is able to do this?
the answer is libuv's thread pool

here you can imagine the conversation between javascript main thread and libuv when the main thread encounters an asynchronous  method


<u> Main thread: </u>

> "*Hey libuv, I need to read file contents but that is a time consuming task. I don't want to block further code from being executed during this time. Can I offload this task to you*?" 
>

<u> Libuv </u>
> "*Sure, main thread. Unlike you, who is single threaded, I have a pool of threads that I can use to run some of these time consuming tasks. When the task is done the file contents are retrieved and the associated callback function can be run.* "


![[Main thread visullization.png]]

Let's understand thread pool in a bit more detail now what we are going to do is basically execute a method and measure the time it takses to runt that method. 

we will do this across different scenarios which will help shed more light on what exactly happens behind the scenes. 

for our experiment we're going to use a brand new module which is the crypto module let's import it at the top

The crypto module provides cryptographic functionalities and similar to the fs module it does use libuv's thread pool for some of its methods 

We are going to use one specific method from the crypto module namely pbk df2
so crypto.pbkdf2 this stands for pssword based  key derivation function 2 and is one of the popular method to hash passords before storing them. you don't have to worry about what the method does or arguments we need to pass in you just have to rememeber it is cpu intensive method that takes a long time and is offloaded to libuv's thread pool 

all right for our first experiment we're goint to measure the time taken to run the synchronous version of pbkdf2. there are lot of experiments to run 
~~~js
const crypto = require('node:crypto')

let start = Date.now()
console.log(start);

crypto.pbkdf2Sync('password','salt',100000,512,'sha512')
console.log('Hash: ', Date.now() - start);
~~~

output:
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/A2Z sheets/CppSTL
$ node index.js
1673394460488
Hash:  615
~~~

It took 615ms to hash the password

if we run same program with 2-3 function calls see what happens

~~~js
const crypto = require('node:crypto')
let start = Date.now()
console.log(start);
crypto.pbkdf2Sync('password','salt',100000,512,'sha512')
crypto.pbkdf2Sync('password','salt',100000,512,'sha512')
crypto.pbkdf2Sync('password','salt',100000,512,'sha512')
console.log('Hash: ', Date.now() - start);
~~~

output: 
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/A2Z sheets/CppSTL
$ node index.js
1673394633659
Hash:  1863
~~~

Time literally tripled. here these functions are synchronous therefore they executed one by one.
![[Pasted image 20230111052346.png]]

Note:
==Every method in node.js that thas the sync suffix always runs on the main thread and is blocking==

Now let's execute same method but this time method nature will be asynchronous it will not contain sync suffix

~~~js
const crypto = require('node:crypto')

const MAX_CALLS = 3;

const start = Date.now()

for(let i = 0; i<MAX_CALLS; i++){

    crypto.pbkdf2('password','salt',100000,512,'sha512',()=>{

        console.log(`Hash: ${i+1}`,Date.now()-start);

    })

}
~~~

output:
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/A2Z sheets/CppSTL
$ node index.js
Hash: 3 753
Hash: 1 761
Hash: 2 774
~~~

Here we can see each call took apporoximately 760ms to run they are definitely not taking twice time like it was in the synchronous method. 

As a graph we can visualize this as a parallel execution 

![[Pasted image 20230111053623.png]]

this means each call to pvkdf2 runs in a separate thread and where are these separate threads? in libuv's thread pool 

Note:
==A few async methods like fs.readFile and crypto.pbkdf2 run on a separate thread in libuv's thread pool. They do run synchronously in their own thread but as far as the main thread is concerned, it appears as if the method is running asynchronously==

### -----------------------THREAD POOL SIZE----------------------
- Welcome back in the previous video we learned about libuv's thread pool and how the main thread offloads some of the async methods like pbk df2 into the thread pool we executed pbkdf2 three times which all executed in parallel indicating we have at least three threads in the pool  but the question is how many threads are in total?
- let's find out this this third experiment
- For experiment 3 let's set MAX_CALLS to 4 and re run the program 
~~~js
const crypto = require("node:crypto");
const MAX_CALLS = 4;

const start = Date.now();
for (let i = 0; i < MAX_CALLS; i++) {
  crypto.pbkdf2("password", "salt", 100000, 512, "sha512", () => {
    console.log(`Hash: ${i + 1}`, Date.now() - start);
  });
}
~~~

Output: 
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication   
$ node index.js
Hash: 4 1268
Hash: 2 1285
Hash: 1 1324
Hash: 3 1326
~~~
We see alomost the same time for all four hashes there is slight difference but on an average it is close to 1300

- Now let's change MAX_CALLS to 5
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication   
$ node index.js
Hash: 4 1531
Hash: 3 1569
Hash: 1 1601
Hash: 2 1610
Hash: 5 2830
~~~
- Now we see something different hash 5 nearly takes twice of other 4 hashes which can only mean one thing libuv's threadpool has four threads 
-  When we run pbkdf2 5 times first 4 hashes got their own thread and complete in nearly the same time the fifth call however has to wait for the thread to be free
- When hash 1 is complete hash 5 runs on the thread and finishes resulting in twice the amount of time taken by the 1st hash 
- ![[Pasted image 20230122184805.png]]
- *So Libuv's thread pool has 4 threads* 

- Now you might ask can we increase teh size of thread in thread pool? so that more calls of pbkdf2 can run in parallel increasing the performance 
- well the answere is YESS
- And that brings us to experiment number 4 
- In this experiment we will increase the thread pool size 
- The way to do that is by setting process environment variable
~~~js
process.env.UV_THREADPOOL_SIZE = 5;
~~~

- By increasing the thread pool size, we are able to improve the total time taken to run multiple calls of an asynchronous method like pbkdf2 

- If you increase it beyond number of CPU cores your machine has average time taken per method also increases 
- My laptop has 4 cores only therefore I will not benefit by increasing the thread pool size 
- ![[Pasted image 20230122185651.png]]
- ![[Pasted image 20230122190848.png]]
- ![[Pasted image 20230122190919.png]]
- If we have 16 threads the OS schedulers switch betweent theads so that each thread get equal time context switching between threads which will increase overal time of threads 

NOTE: *increasing the thread pool size can help with performance but that is limited by the number of available CPU cores*

This is how under the hood Libuv helps executing some of the async methods in nodejs now hang on why some of the async methods and not all? we will see in next section. 

### --------------------NETWORK I/O-------------------------

- Let's do next experiment
~~~js
const https = require('https');
const MAX_CALLS = 50;
const start = Date.now();
for (let i = 0; i < MAX_CALLS; i++) {
    https
    .request('https://www.google.com',(res)=>{
        res.on('data',()=>{});
        res.on('end',()=>{
            console.log(`Request: ${i +1 } `, Date.now()- start);
        })
    })
    .end();
}
~~~

Output : 
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication   
$ node index.js
Request: 5  730
Request: 4  768
Request: 6  772
Request: 50  871
Request: 15  889
Request: 16  898
Request: 34  900
Request: 32  916
Request: 22  919
Request: 35  938
Request: 48  960
Request: 41  983
Request: 24  985
Request: 19  991
Request: 2  993
Request: 28  1001
Request: 47  1006
Request: 10  1017
Request: 49  1025
Request: 40  1028
Request: 38  1030
Request: 46  1032
Request: 20  1032
Request: 31  1034
Request: 44  1035
Request: 39  1040
Request: 27  1050
Request: 43  1058
Request: 1  1059
Request: 29  1062
Request: 21  1070
Request: 45  1117
Request: 17  1121
Request: 42  1122
Request: 18  1133
Request: 14  1154
Request: 37  1341
Request: 25  1346
Request: 23  1348
Request: 8  1353
Request: 12  1383
Request: 13  1385
Request: 36  1389
Request: 26  1410
Request: 30  1413
Request: 3  1433
Request: 7  1434
Request: 11  1438
Request: 9  1449
Request: 33  1484
~~~
- Surprisingly the average is aprox 1200 
- Although Cypto.pbkdf2 and htts.request are asynchrnous http.request method does not seem to use the thread pool
- https.request does not seem to be affected by the number of CPU cores either

### Network I/O
- https.request is a network input/output operation and not a CPU bound operation 
- It does noe use the thread pool
- Libuv instead delegates the work to operating system kernel and whenever possible it will poll the kernel and see if the request has completed 

### Libuv and Async Methods Summary
- In Node.js async methods are handled by libuv
- They are handled in two ways
	1. Native async mechanism
	2. Thread pool
- Whenever possible libuv will use native async mechanism in the OS so as to avoid blocking the main thread 
- Since this a part of the kernel, there is different mechnism for each OS. We have epoll for Linux, Kqueue for MacOS and IO Complettion Port on Windows 
- Relying on native async mechanism makes node Scalable as the only limitation is the operating system kernel 
- Example of this is a network I/O operation 
- If there is no async support and the task is file I/O or CPU intensive, libuv uses the thread pool to avoid blocking the main thread 
- Although the thread pool preserve asynchronocity with respect to NOde's main thread, it can still become a bottleneck if all threads are busy

### -----------------EVENT LOOP IN NODE.JS------------------

### Async Code Execution
- JavaScript is a Synchronous, blocking, single-threaded language
- To make async programming possible, we need the help of libuv
- ![[Pasted image 20230122194612.png]]
- 
![[Pasted image 20230122194750.png]]

![[Pasted image 20230122194842.png]]
![[Pasted image 20230122194858.png]]
![[Pasted image 20230122195003.png]]
![[Pasted image 20230122195014.png]]
![[Pasted image 20230122195044.png]]

![[Pasted image 20230122195109.png]]


### Few Questions
- Whenever an async task completes in libuv, at what poin does Node decide to run the assciated callback function on the call stack?
- What about async methods like setTimeout and setInterval which also delay the execution of a callback function?
- If two async tasks such as setTimeout and readFile completes at the same time, how does Node decides which callback function to run first on the call stack?

### Event Loop
- It is a C program and is part of Libuv
- A design pattern that orchestrates or co-ordinates teh execution of synchronous and asynchronous code in Node.js
- The way we will understand this is two steps 
	1. Visual Representation
	2. Run a few experiments
![[Pasted image 20230122200015.png]]

- The event loop is is a loop that is alives untill your node.js applicaiton is up and running 
- In every iteration of loop we come across six different queues each queue holds one or more than one callback function that needs to be eventually executed on the call stack  and ofcourse the type of a callback functions are different for each queue

- first we have the timer queue
	- This contains callback associated with setTimeout funciton and setInterval 
- second we have is I/O queue 
	- This contains callback associated with all the async methods we have seen so far e.g methods assoicated with fs and http module 
- Third we have is check queue 
	- This contains the callbacks associated with a function called setImmediate() this function is specific to node and not something you would come across when writing javaScript for the browser 
- Fourth we have is close queue
	- This contains callbacks associated with close event of an async task 

- finally we have is Microtask queue at the center
	- This is actually two separate queues 
		1. The first queue is nextTick queue and contains callbacks associated with function called process.net() which is again specific to node.js
		2. The second queue is promise queue which contains callback associated with the native promise in JavaScript and 
- one very important point to note is that timer, I/O, check, close queues are all part of libuv. the two microtasks however are not part of libuv hopefully the color difference conveys that. nevertheless they are still part of node's runtime and play an important role in the order of execution of callback speaking of which let's understand next 

- The arrow heads are already giveaway but it is very easy to get confused 
### Event Loop - Execution Order
- User written synchronous JavaScript code takes priority over async code that the runtime would like to execute
- Which mean only after the callback is empty, the even loop comes into picture
- within the even loop thoough the sequence of execution follows certain rules and there are quite a few rules you have to wrap your head around let's go over them one at a time

1. Any callbacks in the micro task queues are executed. First, tasks in the nexTick queues and only then tasks in the promise queue.
2. All callbacks within the timer queues are executed
3. Callbacks in the micro task queues if present are executed. Again, first tasks in the nextTick queues and then tasks in the promise queue
4. All callback within the I/O queue are executed
5. Callbacks in the micro task queues if present are executed. nextTick queues followed by the promise queue
6. All callbacks in the Check queues are executed 
7. Callbacks in the micro task queues if present are executed. Again, first tasks in the nextTick queues and then tasks in the promise queue
8. All callbacks in the close queues are executed 
9. For one final time in the same loop, the micro task queues are executed. nextTic queues followed by promise queues 

- If there are more callbacks to be processed, the loop is kep alive for one more run and the same steps are repeated. 
- On the other hand, If all the callbacks are executed and there is no more code to process, the event loop exits. 

### Few Questions
1. Whenever an async task completes in libuv, at what poin does Node decide to run the assciated callback function on the call stack?
		Ans. Callback fucntions are executed only when the call stack is empty. The normal flow of execution will not be interupted to run a callback function. 

2.  What about async methods like setTimeout and setInterval which also delay the execution of a callback function?
		Ans. setTimeout and setInterval callbacks are given first priority

3. If two async tasks such as setTimeout and readFile completes at the same time, how does Node decides which callback function to run first on the call stack?
		Ans. Timer callbacks are executed before I/O callbacks even if both are ready at the exact same time. 


### -------------- MICROTASKS QUEUES----------------------

- To understand how code is executed in different queues we will run differnet experiments using code snippet. 
- for the first set of expremint we are going to deal with just two micro task queues before we run our experiment let's learn how how we can queue up a callback function in each of these queues
- To queue up callbacks function in the next tick queue we use a built-in process.nextTick() method  the syntax is as follows
~~~js
process.nextTick( ()=>{
	console.log('This is process.nextTick 1 ')
})
~~~

- To queue up callbacks in promise queue there are different ways but for our experiment we will use just one  promise.resolve.then with a callback function when the promise is resolved the callback function is passed to then block is a fucntion that will be queued up in the promise queue Syntax is below
~~~js
	Promise.resolve().then(()=>{
		console.log("This is a promise resolve1
		")
	})
~~~

- Now that we know how to queue up callbacks in microtask queues let's start our experiment 
1. Experiment 1
~~~js
console.log('console.log 1')
process.nextTick(()=>{
    console.log('this is process.next 1')
})
console.log('console.log 2')
~~~

Output : 
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication
$ node index.js 
console.log 1
console.log 2
this is process.next 1
~~~

Conclusion/ Interference: 
- All user written synchronous JavaScript code tasks priority over async code that the runtime would like to eventually execute. 

Visualization how the above code has been executed
![[Pasted image 20230123173627.png]]



![[Pasted image 20230123173709.png]]






![[Pasted image 20230123173743.png]]


- After these callback registered in queue will push to stack and then will be executed the lines written inside code. 
- 

2. Experiment 2 
	- for this experiment we gonna focus only on two microstask queues 
	~~~js
	Promise.resolve().then(()=>console.log('this is Promise.resolve 1'))
	process.nextTick(()=>console.log('this is process.nextTick 1'))
	
	~~~


Output :
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication
$ node index.js 
this is process.nextTick 1
this is Promise.resolve 1
~~~

conclusion/Inference:
- All callbacks in nextTick queues are executed before callbacks in promise queues. 

~~~js
process.nextTick(()=>console.log('this is process.nextTick 1'));
process.nextTick(()=>{
    console.log('this is process.nextTick 2');
    process.nextTick(()=>{
        console.log('this is the inner next inside next tick')
    });
});
process.nextTick(()=>console.log('this is process.nextTick 3'))
Promise.resolve().then(()=>console.log('this is Promise.resolve 1'));
Promise.resolve().then(()=>{
    console.log('this is Promise.resolve 2')
    process.nextTick(()=>{
        console.log('this is the next tick inside Promise then block')
    })
})

Promise.resolve().then(()=>console.log('this is Promise.resolve 3'))
~~~

Output 
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication
$ node index.js
this is process.nextTick 1
this is process.nextTick 2
this is process.nextTick 3
this is the inner next inside next tick
this is Promise.resolve 1
this is Promise.resolve 2
this is Promise.resolve 3
this is the next tick inside Promise then block
~~~

==The conclusion after running this code snippet was that first of all next tick queues will be executed after that promise queueue callbacks will be executed and if during the execution of promise queue if any callback get registered in next tick queue it will get executed once promise queue is empty i.e when all callbacks in promise queue get executed then new callbacks that are registered in next tick queue will be executed. ==



Important Notes
- Use of process.nextTick is discouraged as it can cause the rest of the event loop to starve
- If you endlessly call process.nextTick. the control will never make it past the microtask queue

Two main reasons to use process.nextTick
1. To allow users to handle errors, cleanup any then unneeded resources, or perhaps try the request again before teh even loop continues 
2. To allow a callback to run after the call stack has unwound but before the even loop continues  

### ---------------TIMER QUEUE-----------------------
- To queue callback function in timer queue we can use either setTimeout or setInterval function. Syntax is below
~~~js
setTimeout(()=>{
	console.log('this is setTimeout 1')
},0);
~~~

Experiment -
~~~js
setTimeout(()=>console.log('this is setTimeout 1'),0)
setTimeout(()=>console.log('this is setTimeout 2'),0)
setTimeout(()=>console.log('this is setTimeout 3'),0)
process.nextTick(()=>console.log('this is process.nextTick 1'));
process.nextTick(()=>{
    console.log('this is process.nextTick 2');
    process.nextTick(()=>{
        console.log('this is the inner next inside next tick')
    });
});
process.nextTick(()=>console.log('this is process.nextTick 3'))
Promise.resolve().then(()=>console.log('this is Promise.resolve 1'));
Promise.resolve().then(()=>{
    console.log('this is Promise.resolve 2')
    process.nextTick(()=>{
        console.log('this is the next tick inside Promise then block')
    })

})
Promise.resolve().then(()=>console.log('this is Promise.resolve 3'))
~~~

Output:
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication   
$ node index.js 
this is process.nextTick 1
this is process.nextTick 2
this is process.nextTick 3
this is the inner next inside next tick
this is Promise.resolve 1
this is Promise.resolve 2
this is Promise.resolve 3
this is the next tick inside Promise then block
this is setTimeout 1
this is setTimeout 2
this is setTimeout 3

~~~

Inference
- Callbacks in the microtask queues are executed before callbacks in the timer queue.

New Experiment
~~~js
setTimeout(()=>console.log('this is setTimeout 1'),0)
setTimeout(()=>{
    console.log('this is setTimeout 2')
    process.nextTick(()=>{
        console.log('This is the inner next tick inside setTimeout')
    })
},0)
setTimeout(()=>console.log('this is setTimeout 3'),0)
process.nextTick(()=>console.log('this is process.nextTick 1'));
process.nextTick(()=>{
    console.log('this is process.nextTick 2');
    process.nextTick(()=>{
        console.log('this is the inner next inside next tick')
    });
});
process.nextTick(()=>console.log('this is process.nextTick 3'))
Promise.resolve().then(()=>console.log('this is Promise.resolve 1'));
Promise.resolve().then(()=>{
    console.log('this is Promise.resolve 2')
    process.nextTick(()=>{
        console.log('this is the next tick inside Promise then block')
    })
})
Promise.resolve().then(()=>console.log('this is Promise.resolve 3'))
~~~

Output:
~~~
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication
$ node index.js 
this is process.nextTick 1
this is process.nextTick 2
this is process.nextTick 3
this is the inner next inside next tick
this is Promise.resolve 1
this is Promise.resolve 2
this is Promise.resolve 3
this is the next tick inside Promise then block      
this is setTimeout 1
this is setTimeout 2
This is the inner next tick inside setTimeout        
this is setTimeout 3
~~~

**conclustion form this experiment**
==First off all callbacks in microtask queues will be executed after that timer queue's callbacks will be executed after executing each callback of timer queue event loop will check if there is any new callback registered in microtask queue if it has any new callbacks then first they will get executed and then again callbacks of timer queue will be executed this process will continue.==

==Timer queue callbacks are executed in FIFO Order==

### ---------------------- I/O QUEUE-------------------------
- To queue callbacks in I/O queue there are number of methods 
- Most of the async methods from the built-in modules queue the callback function in the I/O queue
- for this experiment I will use fs.readFile

Note
==
==When Running setTimeout with delay of 0ms and I/O async method, the order of execution can never be guaranteed==
- why can the order of execution never be guaranteed?
	Ans.   Because when we set time to 0ms Node.js and browser put that callback in timer queue after 1 ms in lamen terms Node.js and browsers do not accept the delay of 0ms the put 0 instead of 1 so if event loop has already started before 1ms and there is no callback in timer queue then callback of I/O queues will get executed. 
- I/O queue callbacks are executed after Microtask queues callbacks and Timer queue callback 

### ----------------- I/O Polling ---------------------------

- To queue a callback function into check queue, we can use a function called setImmediate. Syntax is below
~~~js
setImmediate(()=>{
	console.log("this is setImmediate 1")
})
~~~

Experiment
~~~js
const fs = require('node:fs')
fs.readFile(__filename,()=>{
    console.log('this is readFile 1')
})

process.nextTick(()=>console.log('this is process.nextTick 1'))
Promise.resolve().then(()=>console.log('this is Promise.resolve 1'));
setTimeout(()=>console.log('this is setTimeout 1'),0);
setImmediate(()=>console.log('this is setImmediate 1'))
for(let i = 0; i<200000; i++){}
~~~

Output: 
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication
$ node main.js 
this is process.nextTick 1
this is Promise.resolve 1
this is setTimeout 1
this is setImmediate 1
this is readFile 1
~~~
Note 
=
- Event loop has to poll wheather I/o operations are completed or not and queu up only the completed operations related callbacks 
- Which means when the control enters the I/O queu for the first time it is still empty since it is empty the conrol proceeds to polling part of event loop where it asks readFile wheather it is completed. readFile says yess and event loop queues the associated callback to I/O queue. however the execution is passed to I/O queue and the callback has to wait for its turn. Control then proceeds to the check queue it finds one callback and log it on the console
- I/O events are polled and callback functions are added to the I/O queu only after the I/O is complete. 


### ------------------ CHECK QUEUE --------------------------------

Experiment
~~~js
const fs = require('node:fs')
fs.readFile(__filename,()=>{
    console.log('this is readFile 1')
    setImmediate(()=>console.log('this is setImmediate 1 inside readFile 1'))
})
process.nextTick(()=>console.log('this is process.nextTick 1'))
Promise.resolve().then(()=>console.log('this is Promise.resolve 1'));
setTimeout(()=>console.log('this is setTimeout 1'),0);
for(let i = 0; i<200000; i++){}
~~~
Output:
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication   
$ node check.js 
this is process.nextTick 1
this is Promise.resolve 1
this is setTimeout 1
this is readFile 1
this is setImmediate 1 inside readFile 1
~~~

- Check queues callbacks are executed after Microtask queues callbacks, Timer queu callbacks and I/O queue callbacks are executed. 

### ---------------------------CLOSE QUEUE-------------------
- To queue callbacks into close queue we add close eventlisteners callbacks associated to close event listeners are queued inside close queue. 
~~~js
const fs = require('fs');
const readableStream = fs.createReadStream(__filename);
readableStream.close()
readableStream.on('close',()=>{
    console.log('this is from readableStream close event callback ')
})

setImmediate(()=>{
    console.log(('this is setImmediate 1'))
})
setTimeout(()=>{
    console.log('this is setTimeout 1')
},0);

Promise.resolve().then(()=>console.log('this is Promise resolve 1'))
process.nextTick(()=>console.log('this is process.nextTick 1'));
~~~

Output: 
~~~bash
Abhijeet@Abhijeet-Singh MINGW64 ~/OneDrive/Desktop/Authentication   
$ node close.js 
this is process.nextTick 1
this is Promise resolve 1
this is setTimeout 1
this is setImmediate 1
this is from readableStream close event callback
~~~

- Close queue callbacks are executed after all other queues callbacks in a given iteration of the event loop 

### Event Loop Summary
- The event loop is a C program that ochestrates or co-ordinates the execution of synchronous and asynchronous code in Node.js
- I co-ordinates the execution of callbacks in six different queues
- They are nextTick, Promise, timer, I/O, check and close queues 
- We use process.nextTick() method to queues into the nextTick queue
- We resolve or reject a Promise to queue into the Promise queue
- Execute a async method to queue into the I/O queue
- Use setImmediate function to queue into the check queue and finally
- Attack close event listeners to queue into the close queue
- The order of execution follows the same order listed here
- nextTick and Promise queues are executed in between each queue and also in between each callback execution in the timer and check queues 








































