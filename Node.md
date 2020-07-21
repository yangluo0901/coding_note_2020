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

  

