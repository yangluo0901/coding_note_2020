# Data type and structure

#### <u>Reference type vs Primitive type</u>

+ **Primitive type**: `boolean`, `char`, `byte`, `short`, `int`, `long`, `float` and `double`. 
  + Memory location stores **actual data** held by the primitive type. 
  + When  a value of a primitive is assigned to variable of the same type, <u>a copy is made</u>. 
  + When a primitive a passed into a method, <u>only a copy of the primitive is passed</u>. The called method does not have access to the original primitive value and there **cannot** change it. The called method can change the copied value.
+ Reference type: object, String, arrays, etc. 
  + Memory location stores a reference to the data. 
  + When a reference type is assigned to another reference type, both will point to the same object.
  + When an object is passed to a method, the called method can change the contents of the object passed to it but not the address of the object

```java
class QuickStart {
    public static void main(String[] args) {
        int num = 10;
        change(num);
        
        // here str and node are references
		// References are passed into change(), and a copy of reference
        // is created for each reference
        String str = "hello";
        System.out.println("reference of \"hello\": " + str.hashCode());
        System.out.println("reference of str: " + str.hashCode());
        change(str);

        Node node = new Node(10);
        System.out.println("reference of node: " + node.hashCode());
 		change(node);
        
        System.out.println(num);
        System.out.println(str);
        System.out.println(node.val);

    }

    public static void change(int num) {
        num = 20;
    }

    public static void change(String str) {
        System.out.println("copy of str reference: " + str.hashCode());
        str = "new string"; 
        System.out.println("AFTER, str reference: " + str.hashCode());

    }

    public static void change(Node node) {
        System.out.println("copy of node reference: " + node.hashCode());
        node.val = 100;
        System.out.println("AFTER, node reference: " + node.hashCode());

        // node = new Node(200);
    }
}

output:
-----------------------------------------------------------------------------------------
reference of "hello": 99162322
reference of str: 99162322
copy of str reference: 99162322
AFTER, str reference: 668404369
reference of node: 697960108
copy of node reference: 697960108
AFTER, node reference: 697960108
10
hello
100
```

**explanation**:

+ Since the `int` is **primitive**, only a copy of `num` is passed to `change()`, `change()` does not have a access to the original primitive value; That is why `num` in the main method does not changes it value
+ `str` and `node` are **referenced object**s, `str` is pointed to a new string "new string", its object ID changes, not `str` does not reference to "hello"; `node` is the reference to a `Node` which has value 10, method have access to this `Node` 's  content, reference address does not change.
+ But if we `node = new Node(200)`, now `node` is pointed to the new Node, the corresponding output would be 10.
+ There are two ways to change string value, 1. use `StringBuilder` which is mutable 2. make `change` return a new reference and assign it to `str`

```java
class QuickStart {
    public static void main(String[] args) {
        StringBuilder sb = new StringBuilder("hello");
        System.out.println("Before, string: " + sb.toString());
        System.out.println(sb.hashCode());
        change(sb);
        System.out.println("After, string: " + sb.toString());

    }

    public static void change(StringBuilder sb) {
        sb.append(" world!"); // simiar with `node.val = 200` 
        System.out.println(sb.hashCode());
    }

}
output
-----------------------------------------------------------------------------------------
Before, string: hello
2065530879
2065530879
After, string: hello world!

```

#### <u>Abstract Class vs Interface</u>





#### <u>Singleton Pattern</u>

```java
Class Singleton{
  private static Singleton _instance = null;
  private Singleton(){};
  public static Singleton getInstance(){
		if(_instance == null){
      _instance = new Singleton();
    }
    return _instance;
  }
}
```



#### <u>final, finally, and finalize</u>

**final**:

+ variable
+ class
+ Method

**finally** used with a `try/catch/block` and guarantees that a section of code will be executed, even if an exception is thrown. The finally block will be executed after the try and catch blocks, but before control transfers back to its origin

**finalize** the automatic garbage collector calls the finalize()  just before actually destroying the object

```java
protected void finalize() throws Throwable{
	...
}
```



#### <u>Generics in Java</u>	

The idea is to allow types (Integer, String, Boolean,...) to be a parameter to methods, class and interfaces. Using generic, it is possible to create class and methods that work with different data types.

**generic class**

```java
class Test<T> 
{ 
    // An object of type T is declared 
    T obj; 
    Test(T obj) {  this.obj = obj;  }  // constructor 
    public T getObject()  { return this.obj; } 
} 
   
// Driver class to test above 
class Main 
{ 
    public static void main (String[] args) 
    { 
        // instance of Integer type 
        Test <Integer> iObj = new Test<Integer>(15); 
        System.out.println(iObj.getObject()); 
   
        // instance of String type 
        Test <String> sObj = 
                          new Test<String>("GeeksForGeeks"); 
        System.out.println(sObj.getObject()); 
    } 
}
```

**generics method:**

```java
public static < T > void printArray( T[] inputArray ) {
      // Display array elements
      for(T element : inputArray) {
         System.out.printf("%s ", element);
      }
      System.out.println();
   }

```

