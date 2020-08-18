# React.js

### Troubleshooting:

#### 1. Error: `react-scripts not found`  after `npm install`:

* **option 1**: remove **node_modules**  and **package-lock.json** by ` rm -rf node_modules pakcage-lock.json`. Then reinstall by `npm install`
* **option 2** : check if you have **react-scripts** installed, if not `npm install react-scripts`

#### 2. Error: listen EADDRINUSE: address already in use :::5000

​	this is because `nodemon` restart the server before react kill the process. In this case, we need to make `nodemon` kill process:

1. Install the `kill-port` node package as a dev dependency: `npm install kill-port --save-dev`

2. Create a `nodemon.json` file in the root of your project, containing (replace 18000 with your port, eg 3000), Please __note__  that  below example is a global config, you can place this config into` package.json`

   ```json
   { "events": {	
       		"restart": "kill-port 18000",
               "crash": "kill-port 18000"     
               },     
    "delay": "1500"  
   }
   ```

   

   

3. Then, in your `package.json` file, have something like this:

   `"scripts": {    "start-dev": "nodemon app.js",  },`

#### 3. `exact` in `React-router-dom`

In this example, nothing really. The `exact` param comes into play when you have multiple paths that have similar names:

For example, imagine we had a `Users` component that displayed a list of users. We also have a `CreateUser` component that is used to create users. The url for `CreateUsers` should be nested under `Users`. So our setup could look something like this:

```js
<Switch>
  <Route path="/users" component={Users} />
  <Route path="/users/create" component={CreateUser} />
</Switch>
```

Now the problem here, when we go to `http://app.com/users` the router will go through all of our defined routes and return the FIRST match it finds. So in this case, it would find the `Users` route first and then return it. All good.

But, if we went to `http://app.com/users/create`, it would again go through all of our defined routes and return the FIRST match it finds. React router does partial matching, so `/users` partially matches `/users/create`, so it would incorrectly return the `Users` route again!

The `exact` param **disables the partial matching** for a route and makes sure that it only returns the route if the path is an EXACT match to the current url.

So in this case, we should add `exact` to our `Users` route so that it will only match on `/users`:

```js
<Switch>
  <Route exact path="/users" component={Users} />
  <Route path="/users/create" component={CreateUser} />
</Switch>
```

#### 4. `return` in `Array.map()` doesn't return anything

```javascript
alerts.map((alert) => {
			return (
				<div key={alert.id} className={`alert alert-${alert.type}`}>
					{alert.msg}
				</div>
			);
		});
```

Please note that the `return` inside the `map` function would just return `<div>...</div>` element locally, we need to add another `return` before `map()` to return everything inside the `map()` globally!

```javascript
return alerts.map((alert) => {
			return (
				<div key={alert.id} className={`alert alert-${alert.type}`}>
					{alert.msg}
				</div>
			);
		});
```

#### 5. How does React/React-redux update component when State/Store changes

+ **React**: it will re-render every time, state changes, such as when we use `useState` hook:

  ```js
  const {counter, setCounter} = useState(10);
  // when counter changes (might be caused by an event)
  // react will render the DOM
  ```

  **<u>Be Careful !!!!        infinite loop</u>**

  what happens in below code: 

  `setCounter()` increments `counter` by 1 

  **-->**  react compare the current value of `counter` (1) against the previous value of `counter` (0)

   **-->** react detects the difference and re-render the component

  **-->** `setCounter` will be called again in the second round, `counter` becomes 2  

  **-->** react re-render again **-->** `setCounter` is revoked again in the third round, counter becomes 3 

  **-->** infinite loop

  <u>That is why normally `setState` function is placed inside `event` call back, so that `setState` can not be access in the following round if event does not occur</u> 

 ```js
const myComponent = () = {
    const {counter, setCounter} = useState(0);
	setCounter(counter ++);
	return (
    ...
    )
}
 ```

+ `useEffect` , it is called after the each render if no **dependency** array is not defined. If an empty **dependency** array is provided, `useEffect` is only revoked after the first render.
+ `connect`, the `connect` function generates a wrapper component that subscribes to the store. When an action is dispatched, the wrapper component's callback is notified. It then runs `mapStateToProps` function and **shallow-compares** the result object at this moment against the previous result object, if results are different, then it passes the results to the component as props. <u>In one sentence, connect enables the component access store, compare states and update component by itself.</u>

#### 6. How to fetch data to the component? Why component does not reflect update when update store from `useEffect` ?

**Key point** is that the state in the store might not be saved immediately or saved fast enough before rest of code runs

```js
import {connect} from "react-redux";


const myComponent = ({ file:{files}, getFiles }) =>{
    
    useEffect(() ={
        getFiles();
        const options = [];
        files.forEach(file => options.push({file.name : file})) // some other operation such as refine array
    }, [])
    return (
    	<Select options={files} />
        )
    )
}

const mapStateToProps = state =>({
    file: state.file
})
export default connect(mapStateToProps, {getFiles})(myComponent);
```

1. before mount and render `myComponent`, `file.files=[]`;
2. after the first render, `getFiles()` inside the `useEffect` is called, dispatch `action` to`reducer` to update `file` state in the `store`
3. please **<u>note</u>** that `action` -> `reducer` may take a while which might be done after rest of the code inside the `useEffect`, that results in empty `options` . **<u>This not what we want</u>**
4. finally, `file` is updated `file.files =[...]` , due to `connect`, react re-render the component, `myComponent` in our case, **however** `useEffect()` will **NOT** be called this time since an empty array is provided. `options` keeps empty.
5. No change reflected on the `<Select>` part.



##### Solution 1: 

```js
const myComponent = ({ file:{files}, getFiles }) =>{
    
    useEffect(() ={
        getFiles();
    }, [])
    return (
    	{file.files.length !== 0 && file.files.map( file => <p> file.content <p/>)}
        )
    )
}

const mapStateToProps = state =>({
    file: state.file
})
```

We will see we display data directly instead of doing some operation inside the `useEffect` which cannot be  accessed in the second round when `react` detects the update in `file.files`. what happens listed below:

1. before mount and render `myComponent`, `file.files=[]`;
2. after the first render, `getFiles()` inside the `useEffect` is called, dispatch `action` to`reducer` to update `file` state in the `store`
3. during `file` in the `store` updating, before it finishes updating, the code `return (...)` runs, component is **empty** at this moment (for very short period of time)
4. once `file` in the `store` finishes,  `file.files =[...]`, updating and `react` detects the change, `react` re-render the component, this time `useEffect()` is not reached
5. due to `file.files =[...]`, component now is not empty

<u>**but what if we do want to do some manipulation with the data before displaying them**?</u>

##### Solution 2

```js
import {connect} from "react-redux";


const myComponent = ({ file:{files}, getFiles }) =>{
    
    useEffect(() ={
        getFiles();
    }, []);
	const options = [];
     files.forEach(file => options.push({file.name : file}))
    return (
    	<Select options={files} />
        )
    )
}

const mapStateToProps = state =>({
    file: state.file
})
export default connect(mapStateToProps, {getFiles})(myComponent);
```

we can move the code that manipulates arrays out of `useEffect`, in this case, `const options = [];
     files.forEach(file => options.push({file.name : file}))` will be reached in the second round after `state` finishes updating and `react` detects the change. Please **Note** that, we cannot use `useState` in this case, or state will be changed, re-render occurs again then state will be changed again ... infinite loop.



##### Solution 3:

```js
import {connect} from "react-redux";


const myComponent = ({ file:{files}, getFiles }) =>{
    
    useEffect(() ={
        getFiles();
        const options = [];
        files.forEach(file => options.push({file.name : file})) // some other operation such as refine array
    }, [files.length])
    return (
    	<Select options={files} />
        )
    )
}

const mapStateToProps = state =>({
    file: state.file
})
export default connect(mapStateToProps, {getFiles})(myComponent);
```

add `files.length` as the dependency, every time the length changes, `useEffect` is revoked.

1. before mount and render `myComponent`, `file.files=[]`;
2. after the first render, `getFiles()` inside the `useEffect` is called, dispatch `action` to`reducer` to update `file` state in the `store`
3. in the first round, `const options = [];
           files.forEach(file => options.push({file.name : file}))` is reached before `store` finishes the update, there would be no option in the `<Select>`
4. once `store` finishes updating, re-render, `useEffect` called again,  `file.files = [...]` ( this is the result from the first round) even second round updating is not finished. Now there are option in the `<Select>`
5. once the second round updating finishes, no re-render any more, since `files.length `  is still the same as the result of the first round



#### Order of the functions run 

```js
console.log(" i am running first, but child's functions run earlier if there is any");
console.log("i will  not be revoked if re-render");
const myComponent = ({ file:{files}, getFiles }) =>{
    
    useEffect(() ={
        conosole.log("i am the third one to run");
        getFiles();
        const options = [];
        files.forEach(file => options.push({file.name : file})) 
    }, []);

    console.log("i am running secondly, before hooks run");
    return (
    	<Select options={files} />
        )
    )
}

const mapStateToProps = state =>({
    file: state.file
})
```

