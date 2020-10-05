# Singleton Pattern 

[Why Enum Singleton are better in Java?](https://javarevisited.blogspot.com/2012/07/why-enum-singleton-are-better-in-java.html)

Enum Singletons are new way to implement Singleton pattern in Java by using Enum with just one instance. Though Singleton pattern in Java exists from long time Enum Singletons are relatively new concept and in practice from Java 5 onwards after introduction of Enum as keyword and feature. This article is somewhat related to my earlier post on Singleton, [10 interview questions on Singleton pattern in Java](javarevisited.blogspot.sg/2011/03/10-interview-questions-on-singleton.html) where we have discussed common questions asked on interviews about Singleton pattern. This post is about why should we use Enum as Singleton in Java, What benefit it offers compared to conventional singleton methods etc. 


## Enum Singletons are easy to write.

This is by far biggest advantage, if you have been writing Singletons prior ot Java 5 than you know that even with double checked locking you can have more than one instances. Though that issue is fixed with Java memory model improvement and guarantee provided by volatile variables from Java 5 onwards but it still tricky to write for many beginners. compared to double checked locking with synchronization Enum singletons are cake walk. If you don't believe than just compare below code for conventional singleton with double checked locking and Enum Singletons: 

### Singleton using Enum in Java 

This is the way we generally declare Enum Singleton , it may contain instace variable and instance method but for sake of simplicity I haven’t used any, just beware that if you are using any instance method than you need to ensure thread-safety of that method if at all it affect the state of object. By default creation of Enum instance is thread safe but any other method on Enum is programmers responsibility. 

```java
/** 
* Singleton pattern example using Java Enum. 
*/ 
public enum EasySingleton{ 
    INSTANCE; 
} 
```

You can acess it by `EasySingleton.INSTANCE`, much easier than calling `getInstance()` method on Singleton. 

### Singleton example with double checked locking 

Below code is an example of double checked locking in Singleton pattern, here `getInstance()` method checks two times to see whether `INSTANCE` is null or not and that’s why it’s called double checked locking pattern, remember that double checked locking was broken before Java 5 but with the guranteed of volatile variable in Java 5 memory model, it should work perfectly. 

```java
/** 
* Singleton pattern example with Double checked Locking 
*/ 
public class DoubleCheckedLockingSingleton{ 

     private volatile DoubleCheckedLockingSingleton INSTANCE; 

     private DoubleCheckedLockingSingleton(){} 

     public DoubleCheckedLockingSingleton getInstance(){ 
         if(INSTANCE == null){ 
            synchronized(DoubleCheckedLockingSingleton.class){ 
                //double checking Singleton instance 
                if(INSTANCE == null){ 
                    INSTANCE = new DoubleCheckedLockingSingleton(); 
                } 
            } 
         } 
         return INSTANCE; 
     } 
} 
```
You can call `DoubleCheckedLockingSingleton.getInstance()` to get access of this Singleton class. 

Now Just look at amount of code needed to create a lazy loaded thread-safe Singleton. With Enum Singleton pattern you can have that in one line because **creation of Enum instance is thread-safe and guranteed by JVM.** 

People may argue that there are better way to write Singleton instead of Double checked locking approach but every approach has there own advantages and disadvantages like I mostly prefer static field Singleton intialized during classloading as shown in below example, but keep in mind that is not a lazy loaded Singleton: 

### Singleton pattern with static factory method 

This is one of my favorite method to impelemnt Singleton pattern in Java, Since Singleton instance is static and final variable it initialized when class is first loaded into memory so creation of instance is inherently thread-safe. 

```java
/** 
* Singleton pattern example with static factory method 
*/ 
public class Singleton{ 
    //initailzed during class loading 
    private static final Singleton INSTANCE = new Singleton(); 
    //to prevent creating another instance of Singleton 
    private Singleton(){} 

    public static Singleton getSingleton(){ 
        return INSTANCE; 
    } 
} 
```

You can call `Singleton.getSingleton()` to get access of this class. 


## Enum Singletons handle Serialization by themselves.

Another problem with conventional Singletons are that once you implement serializable interface they are no longer remain Singleton because `readObject()` method always return a new instance just like constructor in Java. you can avoid that by using `readResolve()` method and discarding newly created instance by replacing with Singeton as shwon in below example : 

```java
//readResolve to prevent another instance of Singleton 
private Object readResolve(){ 
    return INSTANCE; 
} 
```

**This can become even more complex if your Singleton Class maintain state, as you need to make them transient, but witn Enum Singleton, Serialization is guarnateed by JVM.**

## Creation of Enum instance is thread-safe.

As stated in point 1 since creation of Enum instance is thread-safe by default you don't need to worry about double checked locking. 

In summary, given the Serialzation and thread-safety guaranteed and with couple of line of code enum Singleton pattern is best way to create Singleton. You can still use other popular methods if you feel so but I still have to find a convincing reason not to use Enum as Singleton, let me know if you got any.

## Interview question on Singleton Pattern in Java 

1. What is Singleton class? Have you used Singleton before?<br> 
   Singleton is a class which has only one instance in whole application and provides a `getInstance()` method to access the singleton instance. There are many classes in JDK which is implemented using Singleton pattern like `java.lang.Runtime` which provides `getRuntime()` method to get access of it and used to get free memory and total memory in Java. 

2. Which classes are candidates of Singleton? Which kind of class do you   make Singleton in Java? <br>
   Any class which you want to be available to whole application and whole only one instance is viable is candidate of becoming Singleton. One example of this is `Runtime` class , since on whole java application only one runtime environment can be possible making Runtime Singleton is right decision. Another example is a utility classes like `Popup` in GUI application, if you want to show popup with message you can have one `PopUp` class on whole GUI application and anytime just get its instance, and call `show()` with message. 

3. Can you write code for getInstance() method of a Singleton class in Java?<br>Same as above.

4. Is it better to make whole getInstance() method synchronized or just critical section is enough? Which one you will prefer? <br>
   This is again related to double checked locking pattern, well synchronization is costly and when you apply this on whole method than call to `getInstance()` will be synchronized and contented. Since synchronization is only needed during initialization on singleton instance, to prevent creating another instance of Singleton, It’s better to only synchronize critical section and not whole method. Singleton pattern is also closely related to factory design pattern where `getInstance()` serves as static factory method. 

5. What is lazy and early loading of Singleton and how will you implement it?<br>As there are many ways to implement Singleton like using double checked locking or Singleton class with static final instance initialized during class loading. Former is called lazy loading because Singleton instance is created only when client calls getInstance() method while later is called early loading because Singleton instance is created when class is loaded into memory. 

6. Give me some examples of Singleton pattern from Java Development Kit?<br>
    1. `Java.lang.Runtime` with `getRuntime()`.  
    2. `Java.awt.Toolkit` with `getDefaultToolkit()`.
    3. `Java.awt.Desktop` with `getDesktop()`.

7. What is double checked locking in Singleton?<br>
Double checked locking is a technique to prevent creating another instance of Singleton when call to getInstance() method is made in multi-threading environment. In Double checked locking pattern as shown in below example, singleton instance is checked two times before initialization.Double checked locking should only be used when you have requirement for lazy initialization otherwise use Enum to implement singleton or simple static final variable. See code above.

8. How do you prevent for creating another instance of Singleton using `clone()` method? <br>
 Preferred way is not to implement Cloneable interface as why should one wants to create clone() of Singleton and if you do just throw Exception from clone() method as “Can not create clone of Singleton class”. 

 9. How do you prevent for creating another instance of Singleton using reflection? <br>
 Since constructor of Singleton class is supposed to be private it prevents creating instance of Singleton from outside but Reflection can access private fields and methods, which opens a threat of another instance. This can be avoided by throwing RunTime Exception from constructor as “Singleton already initialized”

 10. Why Singleton is Anti pattern? <br>
With more and more classes calling getInstance() the code gets more and more tightly coupled, monolithic, not testable and hard to change and hard to reuse because of not configurable, hidden dependencies. Also, there would be no need for this clumsy double checked locking if you call getInstance less often (i.e. once).

11. Singleton vs Static Class? 

12. When to choose Singleton over Static Class? 

13. Can you replace Singleton with Static Class in Java? 

14. Difference between Singleton and Static Class in java? 

15. Advantage of Singleton over Static Class?