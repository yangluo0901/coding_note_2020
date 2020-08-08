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

