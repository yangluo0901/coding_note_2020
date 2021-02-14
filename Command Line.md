# Command Line

1.	**"%" in cmd  **
	
	+ The percent sign is used in batch files to represent command line parameters:` %1`, `%2`, ...
	+ Two percent signs with any characters in between them are interpreted as a variable: `echo %myvar%`

+ Two percent signs without anything in between (in a batch file) are treated like a single percent sign in a command (not a batch file): `%%f`
	
2.	**set environment variables **
	
	+ To set user environment variables: `setx <variable_name> "%USERPROFILE%\lala\lala\path"`
	  	to print out the variable value:  `echo %<variable_name>%`

+ To set system/global environment variables: setx /M setx <variable_name> "%USERPROFILE%\lala\lala\path"
	
3.	**unset environment variables **
	
	+ To delete user environment variables: `reg delete HKEY_CURRENT_USER\Environment /v <variable_name> /f`. If `/f` is left off, we would be prompted: <u>*"Delelte the registry value <varaible name> (Yes/No)?"*</u>
	+ To delete system environment variables:  reg delete "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment" /v <variable_name> /f

4. **Kill Process:**

    `netstat -ano | findstr :<port #>`,  command line would print some thing like

   ```commonlisp
   TCP    0.0.0.0:5000           0.0.0.0:0              LISTENING       2096
     TCP    [::]:5000              [::]:0                 LISTENING       2096
   ```

   `2096`  is the PID, run `tskill <PID #>`



