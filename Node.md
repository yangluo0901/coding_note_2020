# Node.js Note

### Install and Update

#### 1. install Node.js

+ install from Node.js offical website
+ `brew install node`

#### 2. Update Node.js

we use `n`, a useful Node version manager

+ install `n` by using `npm`: `npm install -g n`
+ install different version of `node.js` : `n <version-number>`, or `n latest` to get the latest version

#### 3. Update NPM

`sudo npm install -g npm@latest` 



### <u>1. Authentication with Passport</u>

* **additional packages required** :

  * `passport`
  * `passport-local`
  * `passport-local-mongoose`
  * `express-session`

* **require** these packages in the `js` file

  ```javascript
  const session = require("express-session");
  const passport = require("passport");
  const passportLocalMongoose = require("passport-local-mongoose");
  //passport-local-mongoose hashes the password automatically
  ```

* configure **session**, see this step as constructing a schema of a container to contain all info later on

  ```javascript
  app.use(session({
      secret: "thisismysecre",  // is used to sign session ID
      resave: false,            // typically false. 
      saveUninitialized: false, // typically false
  }));
  ```

* **initialize** passport

  ```javascript
  app.use(passport.initialize());
  app.use(passport.session()); // for persistent login sessions
  ```

* **create** a collection schema and **plug in (enable)** the `passport-local-mongoose`

  ```javascript
  // connect to db
  mongoose.connect("mongodb://localhost:27017/userDB", {useNewUrlParser: true, useUnifiedTopology: true});
  
  mongoose.set("useCreateIndex", true);
  
  const userSchema = new mongoose.Schema({
  	email: String,
      password: String
  });
  
  // plugin
  userSchema.plugin(passportLocalMongoose);
  
  //initialize the mode
  const User = mongoose.model("User", userSchema);
  ```

* **passport** authenticates requests based on different **strategies** which can be seen as rules, now we configure **strategies**,  **serializeUser**, and  **deserializeUser**.

  ```javascript
  // simplified way to create a default local strategy which comes from "passport-local"
  passport.use(User.createStrategy()); 
  // pass serialize and deserialize function to passport to enable them de/serialize user info
  passport.serializeUser(User.serializeUser()); // serialize info into session
  passport.deserializeUser(User.deserializeUser()); //deserialize the session to info
  ```

  **without**  `passport-local-mongoose`,  code will be like:

  ```javascript
  passport.serializeUser(function(user, done) {
    done(null, user.id);
  });
   
  passport.deserializeUser(function(id, done) {
    User.findById(id, function (err, user) {
      done(err, user);
    });
  });
  ```

  ```javascript
  passport.use(new LocalStrategy(
    function(username, password, done) {
      User.findOne({ username: username }, function (err, user) {
        if (err) { return done(err); }
        if (!user) { return done(null, false); }
        if (!user.verifyPassword(password)) { return done(null, false); }
        return done(null, user);
      });
    }
  ));
  ```

  

* **register**, please **note** that `passport.authenticate()` will invoke `login()` automatically, `req.login()` is primarily used when a new user sign up and automatically login the new user

  ```javascript
  app.post('/register', function(req, res){
      User.register({username: req.body.username}, req.body.password, 
                    function(err, user){
          if(err) {
              throw err;
          	res.redirect("/register");
          }else{
              req.login(user, function(err){ 
                  if(err) throw err;
                  res.redirect("/secrets");
              })
          }
      })
  })
  ```

  

* **login**, note that it becomes the app's responsibility to invoke login() if using custom callback

  ```javascript
   /* option 1, call authenticate() inside the router handler, please note that it becomes the app's responsibility to invoke login() if using custom callback*/
  app.post('/login', function(req, res){
      const user = new user({
          email: req.body.username,
          password: req.body.password
      });
  
      passport.authenticate('local', function(err, user){
          if(err) throw err;
          if(!user) return res.redirect("/login");
          req.login(user, function(err)[
              if(err) throw err;
          	return res.redirect("/secrets");
          ])
      })(req, res)  // (req, res) is required to give the access to request to callback 						functions
  
  })
  
  /* option 2 user authenticate as middleware */
  app.post('/login', 
          passport.authenticate('local',{ successRedirect: '/secrets',
                                         failureRedirect: '/login',
                                         failureFlash: 'invalid login info',
                                         successFlash: 'Great!'
  										}),
           function(req, res){
      		...
  		}
          );
  ```

  

* **logout**

  ```
  app.get("/logout", function(req, res){
  	req.logout();
  	res.redirect("/login");
  })
  ```

  

### <u>2. Custom Strategy</u>

* **install** `passport-custom`, `npm install passport-custom`

  ```javascript
  passport.use(new CustomStrategy(
    function(req, done) {
      User.findOne({
        username: req.body.username
      }, function (err, user) {
        done(err, user);
      });
    }
  ));
  ```


## <u>3. Third Parity OAuth (Google)</u>

* go to http://www.passportjs.org/packages/passport-google-oauth20/, install **passport-google-oauth20**

* **register** the app with Google by going to **Google Developer Console** 

  * create **New Project**
  * go to **OAuth consent screen**, select **External**, add **Application Name**, choose **Scopes**, then **Save**
  * go **Credentials**, create **OAuth client ID** --> select **app type** --> add`http://lcoalhost:3000` to **Authorized JavaScript Origin** which is the place host the app --> add uri into **Authorized redirect URIs** which is where google redirects (along with `accessToken`, `profile`, etc.) to after authenticate a user
  * Store **Client ID** and **Client Secret** in `.env` file

* add code to `app.js`

  ```javascript
  const GoogleStrategy = require('passport-google-oauth20').Strategy;
  findOrCreate = require('mongoose-findorcreate');// enbale findOrCreate function
  
  // ... other code, mongoose setup, passport initialization
  
  // cannot use simpified way
  passport.serializeUser(function(user, done) {
      done(null, user.id);
  });
  
  passport.deserializeUser(function(id, done) {
      User.findById(id, function(err, user) {
          done(err, user);
      });
  });
  
  passport.use(new GoogleStrategy({
          clientID: process.env.GOOGLE_CLIENT_ID,
          clientSecret: process.env.GOOGLE_CLIENT_SECRET,
          callbackURL: "http://localhost:3000/auth/google/secrets",
      // due to sunset the support of google + for this API, add:
          userProfileURL: "https://www.googleapis.com/oauth2/v3/userinfo"
      },
      function(accessToken, refreshToken, profile, cb) {
          User.findOrCreate({ googleId: profile.id }, function (err, user) {
              // findorcreate is not a mongoose func, use an extra package
              // install and require mongoose-findorcreate package, and plug in this 				package into the schema
              return cb(err, user);
          });
      }
  ));
  
  // once button is pressed, authenticate user with Google Strategy
  app.get('/auth/google', 
      passport.authenticate('google', { scope: ["profile"] })); // authenticate on the google server
  
  app.get('/auth/google/secrets', // Google sends back after authentication
      passport.authenticate('google', { failureRedirect: '/login' }), // authenticate locally
      function(req, res) {
          // Successful authentication, redirect home.
          res.redirect('/secrets');
      });
  
  ```

  

### 4. `export default` vs `module.exports`

When you have

```js
export default {
  bar() {}
}
```

The actual object exported is of the following form:

```js
exports: {
  default: {
    bar() {}
  }
}
```

When you do a simple import (e.g., `import foo from './foo';`) you are actually getting the default object inside the import (i.e., `exports.default`). This will become apparent when you run babel to compile to ES5.

When you try to import a specific function (e.g., `import { bar } from './foo';`), as per your case, you are actually trying to get `exports.bar` instead of `exports.default.bar`. Hence why the bar function is undefined.

When you have just multiple exports:

```js
export function foo() {};
export function bar() {};
```

You will end up having this object:

```js
exports: {
  foo() {},
  bar() {}
}
```

And thus `import { bar } from './foo';` will work. This is the similar case with `module.exports` you are essentially storing an exports object as above. Hence you can import the bar function.

### 5. response structure:

```json
config: {url: "/api/users", method: "post", data: "			  										{"name":"Ahri","email":"ahri@riot.com","password":"123456"}", headers: 					{…}, transformRequest: Array(1), …}
data: {token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjp7I…								DQ4fQ.SXyV6dPvlxehIoFiCBpO6lzuxgROVlELgglKiVk5GhM"}

headers: {connection: "close", content-length: "195", content-type: "application/json; 			charset=utf-8", date: "Thu, 06 Aug 2020 16:47:28 GMT", etag: "W/"c3-					zqR70NWdg3HPYWfEik1cmoRTOOg"", …}

request: XMLHttpRequest {readyState: 4, timeout: 0, withCredentials: false, upload: XMLHttpRequestUpload, onreadystatechange: ƒ, …}
status: 200
statusText: "OK"
__proto__: Object
```

### 6. upload file to Google cloud

#### Prepare Google Cloud

+ go to google cloud, create a new project called "tutorial", select "tutorial" as the current project from the navbar 
+ Create a service account for the project, search "service account", click "create service account", set "Storage Admin" as the role
+ Create credential, click "create key", and select "json" type,  to generate and download the key for verification, copy this key file to the project directory, in my case, the root of the project directory
+ select "Storage", and create a new bucket, config: **Location type**: multi-region; **default storage class**: standard; **access to objects**: Fine-grained
+ change the public access of the bucket to " Public to internet";

#### Express

We are going to use `multer` to store file or files into `file`/`files` object in `req`. So that the text data will be store in`body` object and file data will be store in `file`/`files`, for more info about `multer`: https://www.npmjs.com/package/multer

```js
const multer = require("multer");
const upload = new multer();// we can define the saved location if store file locally
const router = express.Router();
const {Storage} = require("@google-cloud/storage"); // get the access from google cloud

//connect to the storage
const storage = new Storage({
    keyFileName: path.join(__dirname, "you key file path relative to current file"),
    projectId: "tutorial-291022"
});

// get the bucket by the bucket name
const bucket = storage.bucket("tutorial");

router.post("/post_file", [upload.any()], async (req, res) => {
    //upload.any() will take any file data
    try{
        const {originalname, buffer} = req.files[0]; 
        // save the file into the specific folder in the cloud
        // create a file object
        const file = bucket.file(`<folder name>/${originalname}`); 
        
        // create a input stream to the file object in order to write to the file
        const fileStream  = file.createWriteStream();
        
        // write the uploaded file (buffer in this case) into the file object at the end
        fileStream.end(buffer);
        
        //  finish event: occurs after "end" event and ready to close the stream
        fileStream.on("finish", async() => {
            const url = 			`https://storage.googleapis.com/<folderName>/${bucket.name}/${file.name}`;
            // save the url into database
            const user = await User.findById (req.user.id);
            user.avatar = url;
            await user.save();
            return res.json({message: "avatar uploaded!"})
        })
    }catch(err){
        ... // handle error
    }
})

```

#### 7. Github OAuth API

use github OAuth app to authorize your app and get access to your repos

+ go to you github -> settings -> developer settings -> Personal access tokens-> generate new token, copy this token into `.env` file
+ in express. js file

```js
try {
		const uri = encodeURI(
			`https://api.github.com/users/${req.params.username}/repos?per_page=5&sort=created:asc`
		);
		const headers = {
			"user-agent": "node.js",
			Authorization: `token ${process.env.GITHUBTOKEN}`,
		};
		const githubRes = await axios.get(uri, { headers });
		return res.json(githubRes.data);
	} catch (error) {
		console.error(error);
		res.status(500).send("Server Error!");
	}
```

