# Terminal/Command Line

### 1. Set Environment Varaibles    

* print current environment variables: `printenv` 
*  to set temporary env variables: `export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/`  
*  to append value to existing path: `export <varaible_name> = <new_value>:$<variable_name>`, __EX__: `export PATH=	Users/himanshu/Documents/bin:$PATH` 
*  to set __permanent__ environment variable:     
__Step 1__:open or create a file `.bash_profile`, add `export...` command mentioned above to the file.    
__Step 2__:Type `source <path>/.bash_profile`    
__Explanation__: the `source ...` command can be used to load any functions file into the current shell script or a command prompt    

### 2.Find file by name:    
`find <path> -name <file_name>`    
or `find <path> -iname <sub_file_name>` to find a file with a file name which includes `<sub_file_name>`     

###3. Switch JDK version from the terminal:    
* __Step 1:__ `/usr/libexec/java_home -V`, terminal will print the path of current JDK, like:    
`13.0.2, x86_64:	"Java SE 13.0.2"	/Library/Java/JavaVirtualMachines/jdk-13.0.2.jdk/Contents/Home
`
`11.0.6, x86_64:	"Java SE 11.0.6"	/Library/Java/JavaVirtualMachines/jdk-11.0.6.jdk/Contents/Home    
`    
* __Step 2__: `export JAVA_HOME=<path_of_JDK>`    

###4. Create Alias    

Place the code `alias <alias_name>= "<command line>"` into `.bash_profile`     
__EX__: `alias mongod = " --dbpath ~/mongdb/Data/db`

### 3. Kill Process:

+ **Windows**: `netstat -ano | findstr :<port #>`,  command line would print some thing like

  ```commonlisp
  TCP    0.0.0.0:5000           0.0.0.0:0              LISTENING       2096
    TCP    [::]:5000              [::]:0                 LISTENING       2096
  ```

  `2096`  is the PID, run `tskill <PID #>`