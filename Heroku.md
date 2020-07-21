# Deploy your app on Heroku with MongoDB Atlas

### Connect your app to MongoDB Cloud by Atlas

* Login MongoDB Atlas account, create new cluster or use existing cluster
* Go to **Database Access**, create a new user, you  can set **role** as **admin**
* Go to **Network Access**, make **IP Whitelist**, in the most case, set as **0.0.0.0/0** to allow any access
* Go to **Cluster** section, --> **connect** --> **Connect with the mongo shell**, follow the process
* **Cluster** --> **connect** --> **Connect to app**, follow the instructions
* now you should be able to run app locally but store data in the could database



### Deploy your app to Heroku

##### Prepare your local project

* add a **Procfile** to app's root directory, this file tells Heroku which commands to run to start the app. Add ` web: node app.js ` to the file

* The app was listening to the local port, we now need to make it listen to the **port** that Heroku generates. Add code below to `app.js` or any other **start script**. By using the code below, we can either run code remotely or locally

  ```javascript
  let port = process.env.PORT;
  if (port == null || port == "") {
    port = 3000;
  }
  app.listen(port);
  ```

+ Prepare `package.json` file if it is not generated

+ Specify the right **version of Node** by add

  ```javascript
  "engines": {
      "node": "your node version"
    },
  ```

+ add and commit all your files (except for those ignored) by git

##### Configure Heroku

* install **Heroku CLI** if it is not installed and **login** Heroku
* Add a **Heroku Git remote**, by `heroku create` in the terminal, confirm if Heroku remote is added by `git remote -v`
* `git push heroku remote`