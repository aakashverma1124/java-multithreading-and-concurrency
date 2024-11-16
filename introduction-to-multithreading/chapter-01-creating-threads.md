# Chapter 1: Creating Threads

In java frameword, we can create theads in multiple ways:

## 1. Using a class which extends `Thread` class

### Code Example

```java
package com.completeinterview.multithreading;

class Task extends Thread {

  @Override
  public void run() {
    for (int i = 0; i < 10; i++) {
      System.out.println("Hello " + Thread.currentThread().getName());
    }
  }
}

public class Driver {
  public static void main(String[] args) {
    Task task = new Task();
    task.start();

    for (int i = 0; i < 10; i++) {
      System.out.println("World " + Thread.currentThread().getName());
    }
  }
}
```

### Output
```
World main
World main
World main
World main
World main
World main
Hello Thread-0
World main
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
World main
Hello Thread-0
World main
World main
```

## Using Runnable Interface

When we create a thread, we need to provide the created thread code to execute, or in other words we need to tell the thread what task to execute. The code can be provided as an object of a class that implements the `Runnable` interface. As the name implies, the interface forces the implementing class to provide a run method which in turn is invoked by the thread when it starts.


### Case 1: Using a class that implements the `Runnable` interface

### Code Example

```java
package com.completeinterview.multithreading;

class Task implements Runnable {

  @Override
  public void run() {
    for (int i = 0; i < 10; i++) {
      System.out.println("Hello " + Thread.currentThread().getName());
    }
  }
}

public class Driver {
  public static void main(String[] args) {

    Task task = new Task();
    Thread thread = new Thread(task);
    thread.start();

    for (int i = 0; i < 10; i++) {
      System.out.println("World " + Thread.currentThread().getName());
    }
  }
}
```

### Output
```
World main
World main
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
World main
World main
World main
World main
World main
World main
World main
World main
Hello Thread-0
Hello Thread-0
```

### Case 2: Instead of creating a new class that implements the `Runnable` interface, we can create an anonymous class and pass it as a parameter to the `Thread` constructor.

### Code Example

```java
package com.completeinterview.multithreading;

public class Driver {
  public static void main(String[] args) {
    Thread thread =
        new Thread(
            new Runnable() {
              @Override
              public void run() {
                for (int i = 0; i < 10; i++) {
                  System.out.println("Hello " + Thread.currentThread().getName());
                }
              }
            });

    thread.start();

    for (int i = 0; i < 10; i++) {
      System.out.println("World " + Thread.currentThread().getName());
    }
  }
}
```

### Output
```
World main
World main
Hello Thread-0
World main
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
World main
Hello Thread-0
World main
World main
World main
World main
World main
World main
```

### Case 3: Avoid using anonymous classes and use lambda expressions instead.

### Code Example

```java
package com.completeinterview.multithreading;

public class Driver {
  public static void main(String[] args) {
    Thread thread =
        new Thread(
            () -> {
              for (int i = 0; i < 10; i++) {
                System.out.println("Hello " + Thread.currentThread().getName());
              }
            });
    thread.start();
    for (int i = 0; i < 10; i++) {
      System.out.println("World " + Thread.currentThread().getName());
    }
  }
}
```

### Output
```
World main
World main
World main
World main
World main
World main
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
World main
World main
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
Hello Thread-0
World main
World main
Hello Thread-0
```
