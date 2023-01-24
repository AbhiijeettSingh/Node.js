# -------------ECMASCRIPT--------------
Node.js is an open source, cross platform JavaScript Runtime Environement

Before understanding what is JavaScript Runtime Environment we need to understand few other topics 
- ECMASCRIPT
- ECS-262
- JavaScipt Engines

### Going back in time...
- In 1993, the first web browser with the user interface called Mosaic was released
- In 1994, the lead developers of Mosaic founded a company called Netscape and released a more polished browser called Netscape Navigator.
- Web pages could only be static and there was no interactivity after a page was loaded
- In 1995, Netscape created a new scripting language called JavaScript
- The name was purely for marketing purpose as Java was the hot new language back then

### The Advent of Microsoft
- in 1995, Microsoft debuted their browser Internet Explorer
- Microsoft realised that JavaScript fundamentally changed the user experience of the web and wanted the same for internet explorer.
- But there was no specification to follow
- In order to match Netscape In 1996, Microsoft reverse-engineered the Navigator interpreter to create its own scripting language called Jscript
- Jscript was similar to JavaScript but there was still some differences
- The differences made it difficult for developers to make their websites work well in both browsers
- In fact two implementations are so different that "Best viewed in Netscape" and "Best viewed in Internet Explorer" badges became common for most companies who can't afford to build separate for both browsers

To solve this problem Netscape submitted JavaScript to Ecma International

### ECMA International
- In Nov 1996, Netscape submitted JavaScript to Ecma International
- what is Ecma International? It is an industry association dedicated to the standardization of information and communication systems.
- Netscape wanted a standard specification that all browser vendors could conform to as it would help keep other implementations consistent across browsers
- For each new specification Ecma provides a standard specification and a commitee.
- In case of JavaScript, the standard is called ECMA-262 and the commitee that works on ECMA-262 is called Technical Commitee 39 (TC39)
- Ecma however decided to use the term "ECMAScript" to talk about the official language
- The reason for thi is because Oracle (who acquired Microsystems ) own the trademark for the term JavaScript
- so ECMAScript refers to the standard language whereas JavaScript is what we use in practice and builds on the top of ECMAScript.

### ECMAScript Versions
- 1997 - ECMAScript 1
- 1998 - ECMAScript 2
- 1999 - ECMAScript 3
- ECMAScript 4 never released
- 2009 - ECMAScript 5
- 2015 - ECMAScript 2015 (ES6)
- One version every year since 2015

### ECMAScript Summary
- ECMA-262 is the language specification
- ECMAScript is the language that implements ECMA-262
- JavaScript is basically ECMAScipt at its core but builds on top of that

# -------- CHROME'S V8 ENGINE---------

- JavaScript code that we write cannot be understood by the compuer
- A javaScript engine is a program that converts javascript code that we write into machine code that allows a computer to perform spefic tasks
- JavaScript engines are developed by web browser vendors
- V8 - Open-source Js Engine developed by Google
- SpederMonkey - Js Engine by Mozilla Firefox
- JavaScriptCore - Js Engine by Apple for Safari
- Chakra - Js Engine by Microsoft for Edge hmm but in their lates version they are also using V8 interesting 

### Chrome's V8 Engine & Node.js

chromes's V8 engine by google sits at the core of Node.js
- visit this website to know more https://v8.dev/
- V8 is Google's open source JavaScript engine.
- V8 implements ECMAScript as specified in ECMA-262.
- V8 is written in C++ and is used in Google Chrome, the open source browser from Google
- V8 can run standalone, or can be embedded into any C++ application. This poin is the reason for birth of Node.js
- By embedding V8 into your own C++ application, you can write C++ code that gets executed when a user writes JavaScript code
- In other words You can add new features to JavaScript itself.
- Since C++ is great for lower level operations like file handling, database connections and network operations, by embedding V8 into you own C++ program you have the power to add all of that fucntionality in JavaScript.
- The C++ Program we're talking about is Node.js

### Chrome's V8 Engine Summary
- A JavaScript engine is a program that executes JavaScript code
- In 2008, Google created its own JavaScript engine called V8
- V8 is written in C++ and can be used independently or can be embedded into other C++ programs
- That allows you to write your own C++ programs which can do everything that V8 can do and more 
- Your C++ program can run ECMASCript and additional features that you choose to incorporate
- For example , features that are available in C++ but not available with JavaScript that is the idea behind Node.js

# ------JAVASCRIPT RUNTIME------------
- JavaScript runtime is an environment which provides all the necessary components in order to use and run a JavaScript program
- Every browser has a JavaScript Engine
- A JavaScript Engine is one component in the JavaScript Runtime
- ![[Pasted image 20230120200243.png]]
- All the APIs are provided by the web browsers
- Queues are place where asynchronous tasks wait before they get executed
- Event loop takes care that async task are executed in right order
- V8 engine can only execute ECMAScript but we  know that 
- JavaScritp = ECMAScript + All these APIs
- That's why Browsers require not only Engine but Environement

# ----------What is Node.js---------------
Node.js is  an open-source, cross-platform JavaScript runtime environement
node.js runs the JavaScript program outside the browser

What can I build with NOde.js?
- Traditional websites
- Backend services like APIs
- Real-time application
- Streaming services
- CLI tools
- MultiPlayer games
-Node.js allows you to build complex and powerful applications

Three main folders in Node.js codebase are
- deps (contains external librarys which are necessary for node.js functioning it has V8, UV )
- src (contains c++ source code of node.js it has ser of c++ features that node.js brings to the table )
- lib (contains js code that developers can write to access c++ features like fs. js)
- ![[Pasted image 20230120201853.png]]

### Node.js Summary
- Node.js is an open-source, cross-platform JavaScript runtime environment
- It is not a language, It is not a framework
- Capable of execturing JavaScript code outside a browser
- It can executes not only the standard ECMAScript language but also new features that are made available through C++ bindings using the V8 engine.
- It consists of C++ files which form the core features and JavaScript files which expose common utilities and some of the C++ features for easier consumption.

### Executing JavaScript Code using Node
There are two ways to do it
- Node REPL
- terminal (node filename)

### Browser Vs Node.js
In the browser, most of the time what I do is interacting with the DOM, or other web platform APIs like Cookies. I don't have document, Window and all other objects that are provided by the browser

In browser, we don't have all the nice APIs that node.js provides through its modules. For example file system access functionality.

With Node.js I conrol the environment means I can choose which node.js version to use but in browser it depends on user