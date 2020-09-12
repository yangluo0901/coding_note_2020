# React.js

## Troubleshooting:

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

## Note:

### 1. `exact` in `React-router-dom`

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

### 2. `return` in `Array.map()` doesn't return anything

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

### 3. How does React/React-redux update component when State/Store changes

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

+ `useEffect` , it is called after the each render if no **dependency** array is defined. If an empty **dependency** array is provided, `useEffect` is only revoked after the first render.
+ `connect`, the `connect` function generates a wrapper component that subscribes to the store. When an action is dispatched, the wrapper component's callback is notified. It then runs `mapStateToProps` function and **shallow-compares** the result object at this moment against the previous result object, if results are different, then it passes the results to the component as props. <u>In one sentence, connect enables the component access store, compare states and update component by itself.</u>

### 4. How to fetch data to the component? Why component does not reflect update when update store from `useEffect` ?

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

### 5. Order of the functions run 

```js
console.log(" i am running first, but child's functions run earlier if there is any");
console.log("i will  not be called if re-render");
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

### 6. Understand `useState`

```js
export default function App() {
  let text = "";
  console.log("render .......");
  const changeHandler = (e) => {
    console.log(e.target.value);
    text = e.target.value;
    console.log(text);
  };
  return (
    <div className="App">
      <input type="text" value={text} onChange={changeHandler} />
    </div>
  );
}
```

when we type in "a b c", console result is `render ...   a a b b c c`, value in the `input` does not change,  `value` attribute in the `input` is empty

**<u>findings :</u>**

+ which means it does not re-render.
+ the `value` in the `e.target.value` is not the same with `value` attribute in the `input` tag

```js
export default function App() {
  const [text, setText] = useState("");
  console.log("render .......");
  const changeHandler = (e) => setText(e.target.value);
  return (
    <div className="App">
      <input type="text" value={text} onChange={changeHandler} />
    </div>
  );
}
```

**<u>procedure</u>**: type in letter --> trigger `changeHandler` --> set `text = e.target.value` -->  due to `useState` is used, once `text` updates, `react` re-renders --> `value` attribute in the `input` tag updates to the latest value --> we see what we typed shows in the `input` area

### 7. `useMemo`

Let's look at the example below first:

```js
export default function App() {
  const [text, setText] = useState("");

  const [word, setState] = useState("");

  const changeHandler = (e) => setText(e.target.value);

  const addItem = () => setState(text);

  const veryHeavyCalculation = () => {
    console.log("veryHeavyCalculation is triggered");
    return word + "result";
  };

  // const results = useMemo(veryHeavyCalculation, [word]);
  const results = veryHeavyCalculation();

  console.log("rendering .....");
  return (
    <Fragment>
      <input type="text" value={text} onChange={changeHandler} />
      <button onClick={addItem}> Add Item </button>
      <p>{results}</p>
    </Fragment>
  );
}

```

The idea of this code is that, user can add word into `<p></p>`;

Please note that contents in  `<p></p>` does not necessarily change unless the button is clicked 

**However** **Every time** a letter is typed in to the input, `text` changes, `react` re-renders, `const results = veryHeavyCalculation();` is called  **again and again**. 

**output**:

```
veryHeavyCalculation is triggered 
rendering ..... 
veryHeavyCalculation is triggered 
rendering ..... 
veryHeavyCalculation is triggered 
rendering ..... 
...
```

It wont be a problem if `veryHeavyCalculation` is light weighted, however, things change dramatically, if `veryHeavyCalculatioin` is **heavy** like what its name indicates. Lets see what happens if we utilize `useMemo` :

```js
export default function App() {
  const [text, setText] = useState("");

  const [word, setState] = useState("");

  const changeHandler = (e) => setText(e.target.value);

  const addItem = () => setState(text);

  const veryHeavyCalculation = () => {
    console.log("veryHeavyCalculation is triggered");
    return word + "result";
  };

  const results = useMemo(veryHeavyCalculation, [word]);
  // const results = veryHeavyCalculation();

  console.log("rendering .....");
  return (
    <Fragment>
      <input type="text" value={text} onChange={changeHandler} />
      <button onClick={addItem}> Add Item </button>
      <p>{results}</p>
    </Fragment>
  );
}


```

**output**:

```
rendering ..... 
rendering ..... 
rendering ..... 
rendering ..... 
...
```

**conclusion**

`useMemo` memoizes value, prevent unnecessary re-render. Please **note** that, `useMemo` does more work behind, do not overuse it unless the memoized value comes from heavy computations.

### 8. `useCallback`

lets look at the example below

```js
export default function App() {
  const [value1, setValue1] = useState(0);

  const [value2, setValue2] = useState(10);

  const add1 = () => setValue1(value1 + 1);

  const add2 = () => setValue2(value2 + 2);

  const veryHeavyCalculation = () => {
    console.log("veryHeavyCalculation is triggered");
    return value2 * 1.1;
  };

  const another = veryHeavyCalculation();

  console.log("rendering .....");
  return (
    <Fragment>
      <p>value1 is {value1}</p>
      <button onClick={add1}> Add Value1 </button>
      <p>another value is {another}</p>
      <button onClick={add2}> Add another value </button>
    </Fragment>
  );
}

```

We notice that `another value` is independent with `value1`, every time we change the value of `value1`, it is not necessary for `another value` to change, right?

but **output**:

```
veryHeavyCalculation is triggered 
rendering ..... 
veryHeavyCalculation is triggered 
rendering ..... 
veryHeavyCalculation is triggered 
rendering ..... 
```

**solution**:

```js
export default function App() {
  const [value1, setValue1] = useState(0);

  const [value2, setValue2] = useState(10);

  const add1 = () => setValue1(value1 + 1);

  const add2 = () => setValue2(value2 + 2);

  const veryHeavyCalculation = () => {
    console.log("veryHeavyCalculation is triggered");
    return value2 * 1.1;
  };

  const another = useMemo(veryHeavyCalculation, [value2]);

  console.log("rendering .....");
  return (
    <Fragment>
      <p>value1 is {value1}</p>
      <button onClick={add1}> Add Value1 </button>
      <p>another value is {another}</p>
      <button onClick={add2}> Add another value </button>
    </Fragment>
  );
}

```

**conclusion**:

similar with `useMemo` to memoizes the value to prevent unnecessary renders, `useCallback` memoizes functions



### 9.`useRef` : 

**<u>it does not hold state, so it will NOT re-render</u>**

**Execution Order of `useEffect`**:

```JS
const Test = () => {
  const [user, setUser] = useState({ name: "Yang", weight: 165 });
    
  const usePrevious = (value) => {
    console.log(" Step-1");
      
    const ref = useRef();
    useEffect(() => {
      ref.current = value;
      console.log(" Step-6");
      console.log(ref.current);
    }, [value]);

    console.log(" Step-2");
    console.log(ref.current);
    return ref.current;
  };

  const someother = () => {
    console.log("step-4");
    useEffect(() => {
      console.log(" Step-8");
    });
    return 0;
  };

  const previousUser = usePrevious(user);

  console.log(" Step-3");
  console.log(previousUser);

  useEffect(() => {
    console.log("inside effect Step-7");
    console.log(previousUser);
    console.log(user);
    if (!isEqual(previousUser, user)) {
      console.log("Effect is running");
    }
  });
  const zero = someother();
  console.log(" Step-5  ");

  const handler = () => {
    setUser((prev) => {
      return {
        ...prev,
        weight: Math.random() >= 0.5 ? prev.weight + 1 : prev.weight
      };
    });
  };

  return (
    <>
      <p>Current weight : {user.weight}</p>
      {/* <p>
        <b>refFromUseRef</b> value: {refFromUseRef.current}
      </p> */}
      <p>
        <b>previous weight </b>:
        {typeof previousUser !== "undefined" && previousUser.weight}
      </p>

      <button onClick={handler}>Cause re-render</button>
    </>
  );
};
```

**output of the first render**

```console
 Step-1 
 Step-2 
Object {name: "Yang", weight: 165}
 Step-3 
Object {name: "Yang", weight: 165}
step-4 
 Step-5   
 Step-6 
Object {name: "Yang", weight: 166}
inside effect Step-7 
Object {name: "Yang", weight: 165}
Object {name: "Yang", weight: 166}
Effect is running 
 Step-8 
```

**conclusion**:  

execute all regular functions (not hooks) first in the order of they are called, then hooks in the order of they are called

### 10. Deep Comparison

```js
const Test = () => {
  const [user, setUser] = useState({ name: "Yang", weight: 165 });

    // use container to avoid the case that undefined !== null or []
  const usePrevious = (value, container) => {
    console.log(" Step-1");
    const ref = useRef();

    useEffect(() => {
      ref.current = value;
      console.log(" Step-5");
      console.log(ref.current);
    }, [value]);

    console.log(" Step-2");
    console.log(ref.current);
    return ref.current;
  };

  const previousUser = usePrevious(user);

  console.log(" Step-3");
  console.log(previousUser);

  useEffect(() => {
    console.log("inside effect Step-6");
    console.log(previousUser);
    console.log(user);
    if (!isEqual(previousUser, user)) {
      console.log("Effect is running");
    }
  });
 
  console.log(" Step-4");
  
  const handler = () => {
    setUser((prev) => {
      return {
        ...prev,
        weight: Math.random() >= 0.5 ? prev.weight + 1 : prev.weight
      };
    });
  };

  return (
    <>
      <p>Current weight : {user.weight}</p>
      {/* <p>
        <b>refFromUseRef</b> value: {refFromUseRef.current}
      </p> */}
      <p>
        <b>previous weight </b>:
        {typeof previousUser !== "undefined" && previousUser.weight}
      </p>

      <button onClick={handler}>Cause re-render</button>
    </>
  );
};

```

**part of output when weight increases to 166**:

```
 Step-5 
Object {name: "Yang", weight: 166}
inside effect Step-6 
Object {name: "Yang", weight: 165}
Object {name: "Yang", weight: 166}
```

we noticed that `ref.current`in the `usePrevious`.`useEffect` is updated to the **newer** version in which weight becomes **166**.  However, in the outside `useEffect`, `previousUser` is still the **old**  version in which weight is **165**.

Inside `usePrevious` returns **old** version of state first, then call `useEffect` to update state but does not re-render, so `previousUser` is still the **old** version. But the **newer** version is there, it is just not rendered, the **newer** version will be returned when next change happens. In this way, `previousUser` always hold the previous version.

But please note that, the initial state of `previousUser`will be `undefined`.

### 11. Function inside vs outside `useEffect`

+ **bad example (confusion as well):**

```js
const State = () => {
  const hello = () => console.log("hello");

  useEffect(()=>{
    hello();
  }, [hello]);
  return (
    <Fragment>
      <p> current value is {state}</p>
      <button onClick={() => setState((prev) => prev + 1)}> Add </button>
    </Fragment>
  );
};
```

Based on what I search online and look up the docs, `hello()` will be called infinitely due to new **object** of `hello` function is created every time re-render. **However** based on my test, `hello()` is only called once.

**any way** for better practice:

```js
  const hello = useCallback(() => {
    console.log("hello");
  }, []);
```

### 10. Deploy a MERN app on Heroku

+ install `heroku`, --> `heroku login` --> `heroku create` --> `heroku git:remote -a <domain name>` now a `heroku` remote is added into the directory

+ add `.env` into `.gitignore`, go to `heroku` **dashboard** --> **settings** --> section **Config Vars**, add **key**s and  **value**s from `.env` into this section

+ **add right port**, we use a certain port for development, `heroku` will assign a port to the app, use code like  `const PORT = process.env.PORT || 5000;`

+ **specify node version**: add `"engines":{ "node": <version> }` into **backend** `package.json` below `"dependencies"`

+ **remove `nodemon`**: remove `nodemon` from `dependencies` if it is there

+ **build react app on` heroku`**: we need to build the react app into optimized version for production by using `npm run build`, after that `/client/build/index.html` will be generated, the `index.html` is the gateway for the react app. We can either `build` locally and push into `heroku` or write a script to `build` after pushing the app into heroku

  `heroku-postbuild: "NPM_CONFIG_PRODUCTION=false npm install --prefix client && npm run build --prefix client"`, 

+ **add a Procfile**: add a file named `Procfile` that specifies how `heroku` runs the app. Write `web: node <server file name>` into the file. If `Procfile` is not specified, ` heroku` will run `start` script defined inside **backend** `package.json/scripts`. So make sure `"start"` command is defined in `package.json` file.

+ **config server to run react app on `heroku`**,  we configured how to run `server.js` , how to `build` react app on heroku, now we are going to configure how to connect the react APP to the backend. 

  add code below into `server.js`

  ```js
  if (process.env.NODE_ENV === "production") {
  	app.use(express.static("client/build")); // config static folder
  	app.get("*", (req, res) => { // connect front end to backend
  		res.sendFile(path.resolve(__dirname, "client", "build", "index.html"));
  	});
  }
  ```

  

### 12 . Select Component

+ install **`react-select`**

+ ```js
  import select from "react-select";
  
  const Component = () => {
      
      // initialState is {} if it is a single select
      const[selected, setSelected] = useState([]);
      
      onSubmit = (e) => {
          e.preventDefault();
          //...submit
      }
      
      return 
      <form onSubmit={onSubmit}>
          <Select
      		style={{
                     menuPortal: (base) => ({
                         ...base,
                         zIndex: 9999,
                     }),
  			}}
  			menuPortalTarget={document.body}
  			closeMenuOnSelect={false}
      		name="select"
      		isMulti
      		options={toBeSelected}
    		value={selected}
      		onChange={(selected) => setSelected(selected)}
      	>
          </Select>
      </form>
      
  }
  ```
  
  the format of options is `{value:..., label:... }`, normally, `value` is the id of the item, `value` need to be extract from the form data in the backend.
  
  ```
  style={{
                     menuPortal: (base) => ({
                         ...base,
                         zIndex: 9999,
                     }),
  			}}
  			menuPortalTarget={document.body}
  ```
  
  is used to avoid the drop down menu overlapping by other components
  
  

### 13. TinyMCE in React

+ install  `@tinymce/tinymce-react`

```js
import { Editor } from "@tinymce/tinymce-react";

const Component = () => {
    const setup = {
		height: 500,
		menubar: false,
		plugins: [
			"advlist autolink lists link image charmap print preview anchor",
			"searchreplace visualblocks code fullscreen",
			"insertdatetime media table paste code help wordcount",
		],
		toolbar:
			"undo redo | formatselect | bold italic backcolor | \
          alignleft aligncenter alignright alignjustify | \
          bullist numlist outdent indent | removeformat | help",
		textpattern_patterns: [
			{ start: "*", end: "*", format: "italic" },
			{ start: "**", end: "**", format: "bold" },
			{ start: "__", end: "__", format: "bold" },
			{ start: "`", end: "`", format: "code" },
			{ start: "#", format: "h1" },
			{ start: "##", format: "h2" },
			{ start: "###", format: "h3" },
			{ start: "####", format: "h4" },
			{ start: "#####", format: "h5" },
			{ start: "######", format: "h6" },
			{ start: "1. ", cmd: "InsertOrderedList" },
			{ start: "* ", cmd: "InsertUnorderedList" },
			{ start: "- ", cmd: "InsertUnorderedList" },
		],
	};
	
	const[tinyContent, setContent] = useState("");

    const editorHandler = (content, editor) => {
        setContent(content)
    }
	return <form onSubmit={onSubmit}>
        		<Editor
					apiKey={apiKey}
					init={setup}
					value={tinyContent}	
					onEditorChange={editorHandler}
				/>
        	</form>
}

```

use `onEditorChange` to get **content** of the editor.



### 13. Highlight code block in React.js

+ we will use **highlight.js**, `npm install highlight.js`
+ use code:

```js
import React, { useEffect } from "react";
import hljs from "highlight.js";
import "highlight.js/styles/github.css";

const Highlight = () => {
  useEffect(() => {
    // Prism.highlightAll();
    updateCodeSyntaxHighlighting();
  });

  const updateCodeSyntaxHighlighting = () => {
    document.querySelectorAll("pre code").forEach((block) => {
      hljs.highlightBlock(block);
    });
  };
  const code = `
  lets see the code below:
  <pre>
  <code>
  const MemoChild = ({ value, name }) => {
    const veryHeavyCalculation = () => {
      console.log("veryHeavyCalculation is triggered");
      return "result ---   " + value;
    };
    const result = veryHeavyCalculation();
    console.log("rendering " + name);
    return <p>{result}</p>;
  };
  </pre>
  </code>
  When we click the "component 1 event" button, counter1 increments by 1, React re-renders MemoCompo .

  `;

  return (
    <div
      dangerouslySetInnerHTML={{
        __html: code
      }}
    />
  );
};

```

