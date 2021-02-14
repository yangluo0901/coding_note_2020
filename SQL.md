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

1. **`DateTime` vs. `TimeStamp`** 

   Very similar, except for MySQL converts `TIMESTAMP` values from the current time zone to UTC for storage, and convert it back from UTC to the current time zone for retrieval.    

3. **Difference among `CASCADE`, `SET NULL`, `RESTRICT`, `SET DEFAULT`**    

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

4. **Connect to MySQL server fail**    

   <u>*Solution*</u> is to __revert back__ to native password authentication:    

   + type in ` ALTER USER '< user_name>@localhost' IDENTIFIED WITH mysql_native_password BY '< password>'`    
   + __Tips__:    
     to change password of root, type in:    
     `SET PASSWORD FOR '<user_name>@localhost' = PASSWORD('<new_password');`    
     `flush privileges;`      

5. **Import data to database**    

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

7. **Change sql_mode:**    

   __Step 1__: Run `SELECT @@GLOBAL.SQL_MODE` to see current sql_mode  
   __Step 2__: Copy the current sql_mode but remove `FULLY_GROUP_BY_ONLY`  
   __Step 3__: find your `my.cnf` file  
   __Step 4__: Add `sql_mode=<your copy>` to `my.cnf` file under `[mysqld]` section  
   __Step 5__: `Mysql.server restart`







