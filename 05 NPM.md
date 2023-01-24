What is npm?
1. It is world's largest software library (registy)
2. It is a software package manager

**npm is world's largest software library**
- A book library contains books written by various authors
- Similarly npm is a library or a registry which contains code packages written by various developers
- It is a large public database of JavaScript code that developers from all over the world can use to share and borrow code
- If you author a "code packag", you can publish it to the npm registry for others to use 
- On the other hand If you come across a code package that is authored by someone else and solves the problem you have at hand, you can borrow that code without having to reinvent the wheel.

npm is a software package manager
- Developers publish and consume code packages
- How does a developer publish a package?
- How does a developer consume a package?
- What happens if the code package author decides to change a function name in a packages?
- How would one update an already installd package?
- What if the packages I am consuming is dependent on another package?
- That is where npm comes into the picuture. A command line interface tool that lets us manage packages in a project 

npm and other Package Managers
- pnpm and Yarn
- npm is default package manager that is installed when we install node. 
- npm did stand for node package manager when it first started out 
- however as time has evolved. Now npm is a package manager for the JavaScript programming language 

Why Learn about npm?
- When building enterprise scale application, we often need to rely on code written by other developers we need npm


### --------------------------Package.Json--------------------
What it is?
- package.json is npm's configuration file
- It is a json file that typically lives in the root directory of your package and holds various metadata relevant to the package

Why it is?
- package.json is the central place to configure and describe how to interact with and run your packages
- It is primarily used by npm CLI
- To create package.json file use 
~~~bash
npm init -y
~~~

### ----------------------Installing Packages------------------

~~~bash
npm install packagenName
~~~

To uninstall
~~~bash
npm uninstall packageName
~~~

### ----------------------Using Packages--------------
- After installing package we need to require it in a file
~~~js
const localModule = require('./path-to-module')
~~~

























