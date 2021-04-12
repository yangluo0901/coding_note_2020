# SQL    

## 1. Troubleshooting

1. **SQL query such as `SELECT` does not reflect the database change immediately after CRUD**    **solution**:`commit`

   ```sql
   SQL> DELETE FROM CUSTOMERS
      WHERE AGE = 25;
   SQL> COMMIT;
   ```

2. **Error: `The MySQL server is running with the --secure-file-priv option so it cannot execute this statementî when use "LOAD DATA  ..."The MySQL server is running with the --secure-file-priv option so it cannot execute this statementî when use "LOAD DATA  ..."`**    

   Add `LOCAL` keyword, `LOAD DATA LOCAL` then you will see another __error__: `"used command is not allowed in this version"`. __Cause__: in the newer version of MySQL server, parameter `local_infile = OFF`, __fix__:    

   - find out the file `my.cnf`, for __Mac__: type `find -name <file_name>`, in my Mac, path is `/usr/local/etc/my.cnf`    

   * add `[myslqd]`  
     `local_infile = ON` into `my.cnf` file    

## 2. Note

1. ### **`DateTime` vs. `TimeStamp`** 

   Very similar, except for MySQL converts `TIMESTAMP` values from the current time zone to UTC for storage, and convert it back from UTC to the current time zone for retrieval.    

3. ### **Difference among `CASCADE`, `SET NULL`, `RESTRICT`, `SET DEFAULT`**    

   `CASCADE`: it will propagate the change when the parents changes. 

   ```sql
   CREATE TABLE products
   ( product_id INT PRIMARY KEY,
     product_name VARCHAR(50) NOT NULL,
     category VARCHAR(25)
   );
   
   CREATE TABLE inventory
   ( inventory_id INT PRIMARY KEY,
     product_id INT NOT NULL,
     quantity INT,
     min_level INT,
     max_level INT,
     CONSTRAINT fk_inv_product_id
       FOREIGN KEY (product_id)
       REFERENCES products (product_id)
       ON DELETE CASCADE
   );
   
   ```

   `Inventory` has a foreign key that referencing `Products`, if one of product is deleted in the `Products` table, the entry in the `Inventory` table will be deleted as well, since object cannot referencing to `null`

   `SET NULL`: sets the column value to null when a referenced table goes away (parent means the referencing table)    

   `RESTRICT`: prevent opertion of `DELETE`    

   `SET DEFAULT`: sets the default value    

4. ### **Connect to MySQL server fail**    

   <u>*Solution*</u> is to __revert back__ to native password authentication:    

   + type in ` ALTER USER '< user_name>@localhost' IDENTIFIED WITH mysql_native_password BY '< password>'`    
   + __Tips__:    
     to change password of root, type in:    
     `SET PASSWORD FOR '<user_name>@localhost' = PASSWORD('<new_password');`    
     `flush privileges;`      

5. ### **Import data to database**    

   ```mysql
   LOAD DATA  LOCAL INFILE '<path_to_csv_file>'
   			INTO TABLE <dest_table> 
   			FIELDS TERMINATED BY ','
   			OPTIONALLY ENCLOSED BY '"'
   			LINES TERMINATED BY '\n'
   			IGNORE 1 LINES (dest_col_1, @dummy,dest_col_2, @var)
   			SET <dest_col_n> = @var + "hello";
   ```

   __Explanation__: System will go through the column one by one in the order that presented in csv file, lets say `data_table( user_id, age, gender, descritption)`, in the case above, system assigns all values of `user_id` to `dest_col_1`, ingnore `age` column, assgins all values of `gender` to `dest_col_2`, and assign the all values of `description` + "hello" (for each row) to the `< column_name>`    

7. ### **Change sql_mode:**    

   __Step 1__: Run `SELECT @@GLOBAL.SQL_MODE` to see current sql_mode  
   __Step 2__: Copy the current sql_mode but remove `FULLY_GROUP_BY_ONLY`  
   __Step 3__: find your `my.cnf` file  
   __Step 4__: Add `sql_mode=<your copy>` to `my.cnf` file under `[mysqld]` section  
   __Step 5__: `Mysql.server restart`

6. ### connect MySQL to django

   + install mysql-server

   + install mysqlclient, which is used to connect to the server. `conda install mysqlclient`

   + create a new database by either using MySQL workbench or through mysql shell `mysql -u root -p`

   + update settings.py

     ```python
     DATABASES = {
         'default': {
             'ENGINE': 'django.db.backends.mysql',
             'NAME': 'first_app',
             'USER': 'root',
             'PASSWORD': 'root'
         }
     }
     ```

   + `python manage.py makemigrations` , then `python manage.py migrate`

7. ### How does session work in Django

   1. #### How does cookie work?

      + cookies is a small piece of data **stored in the user's browser**
      + process:
        + browser send a request to the server
        + server send the response along with one or more cookies to the browser
        + Browser saves the cookies, from now on, the browser will send every request along with cookies to the server until the cookie expires

   2. #### How does session work?

      + Instead of store the data in browser, session are stored in server
      + Process:
        + browser send a request to the server
        + Server (django) creates a unique string called session ID or SID, this SID associates with the data
        + then server send a cookie containing SID to browser, like a key
        + then browser send this every request with this key to the server
        + server verify by using the SID

   3. #### Implement in Django

      + Django implements sessions by using **middleware**, there is `django.contrib.sessions.middleware.SessionMiddleware` under `MIDDLEWARE` section in `setting.py`, this **middleware is responsible for generating unique SID**

        ```python
        MIDDLEWARE = [
            'django.middleware.security.SecurityMiddleware',
            'django.contrib.sessions.middleware.SessionMiddleware',
            'django.middleware.common.CommonMiddleware',
            'django.middleware.csrf.CsrfViewMiddleware',
            'django.contrib.auth.middleware.AuthenticationMiddleware',
            'django.contrib.messages.middleware.MessageMiddleware',
            'django.middleware.clickjacking.XFrameOptionsMiddleware',
        ]
        ```

      + an app called `django.contrib.sessions` is used to store the session data into database, so we need to include in this app in `INSTALLED_APP` section

        ```python
        INSTALLED_APPS = [
            'django.contrib.admin',
            'django.contrib.auth',
            'django.contrib.contenttypes',
            'django.contrib.sessions',
            'django.contrib.messages',
            'django.contrib.staticfiles',
            'first_app',
            'second_app'
        ]
        ```

   4. #### [When Sessions are saved](https://docs.djangoproject.com/en/3.2/topics/http/sessions/#:~:text=When%20sessions%20are%20saved&text=When%20set%20to%20True%20%2C%20Django,has%20been%20created%20or%20modified.&text=Similarly%2C%20the%20expires%20part%20of,the%20session%20cookie%20is%20sent.) 

      By default, Django only saves to the session database **when the session has been modified** – that is if any of its dictionary values have been assigned or deleted, **be careful with the last case** 

      ```python
      # Session is modified.
      request.session['foo'] = 'bar'
      
      # Session is modified.
      del request.session['foo']
      
      # Session is modified.
      request.session['foo'] = {}
      
      # Gotcha: Session is NOT modified, because this alters
      # request.session['foo'] instead of request.session.
      request.session['foo']['bar'] = 'baz'
      ```

      In the last case of the above example, we can tell the session object explicitly that it has been modified by setting the `modified` attribute on the session object:

      ```
      request.session.modified = True
      ```

      To change this default behavior, set the [`SESSION_SAVE_EVERY_REQUEST`](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-SESSION_SAVE_EVERY_REQUEST) setting to `True`. When set to `True`, Django will save the session to the database on every single request.

      Note that the **session cookie is only sent when a session has been created or modified**. If [`SESSION_SAVE_EVERY_REQUEST`](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-SESSION_SAVE_EVERY_REQUEST) is `True`, the session cookie will be sent on every request.

   5. #### Set Expiration for sessions

      | Variable Name                   | Explanation                                                  |
      | :------------------------------ | ------------------------------------------------------------ |
      | SESSION_COOKIE_AGE              | This variable is used to set cookie expiration time in seconds. By default, it is set to 1209600 seconds or 2 weeks. If `SESSION_EXPIRE_AT_BROWSER_CLOSE` is not set then Django uses this variable to set cookie expiration time. Here is how you can set session cookie expiration time to 5 days: `SESSION_COOKIE_AGE = 3600*24*5` |
      | SESSION_EXPIRE_AT_BROWSER_CLOSE | This variable controls whether to expire the session cookie when the user closes the browser. By default, it is set to `False`. If set to `True`, session cookie lasts until the browser is closed, irrespective of the value of `SESSION_COOKIE_AGE`. |

       

