# Django





## 1. Troubleshooting/Set up

 1. **windows/Mac set up virtual env with python 3**

  + find path of python 3 buy using`where python`, then `virtualenv --python C:\Users\yluo\AppData\Local\Programs\Python\Python37\python.exe venv`

  * for **Mac**, `virtualenv -p python3 <path + vritualenv name>`

2. **install mysqlclient, `error: "ld: library not found for -lssl`**

      ```
      clang: error: linker command failed with exit code 1 (use -v to see invo	cation)
         make: *** [mysql2.bundle] Error "
      ```

      + Step-1 brew install openssl
      + Step-2 export LIBRARY_PATH=$LIBRARY_PATH:/usr/local/opt/openssl/lib/	

3. **start app with path**
  cd into the destination directory, use Django-admin startapp appname;

  ​			

## 2. Note

1. **static_root vs static_url**
   + **STATIC_ROOT**: the absolute path to the directory where "collectstatic" will collect static files to for deployment
   + **STATIC_URL**: default: None, is simply the prefix or url that is prepended to your static files and is used by the static method in Django templates mostly. **EX**:  when you have `STATIC_URL` defined as `/static/`, then your users would request static files from `/static/file-name.example` (a relative                        URL on your server).

2. **fecth erro JsonResponse from view**

```javascript
response =JsonResponse({"error": "Something Wrong with the form"})
       	 response.status_code = 403
        	return response

js file:	error: function (response) {
                		alert(response.responseJSON.error);
	}
```

