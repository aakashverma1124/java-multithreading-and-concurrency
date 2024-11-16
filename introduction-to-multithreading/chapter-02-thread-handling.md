# Chapter 2: Thread Handling

## Joining Thread

Let's understand what are daemon threads first. Consider the following example where we are setting up the thread to be a daemon thread.

If you execute the below code, you might or might not see `Hello World from Thread-0` output but you will see `Hello World from main` in the console. That is because the main thread exits right after starting the thread and print statement. Once it exits, the JVM also kills the spawned thread. Therefore as soon as the main thread job is done, the JVM will not wait for the daemon thread to complete.

### Code Example

```java
package com.completeinterview.multithreading;

class Task implements Runnable {

  @Override
  public void run() {
    while (true) {
      System.out.println("Hello World from " + Thread.currentThread().getName());
      try {
        Thread.sleep(500);
      } catch (InterruptedException ex) {
        System.out.println(ex.getMessage());
      }
    }
  }
}

public class Driver {
  public static void main(String[] args) {
    Task task = new Task();
    Thread thread = new Thread(task);
    thread.setDaemon(true);
    thread.start();
    System.out.println("Hello World from " + Thread.currentThread().getName());
  }
}
```

## Predicted Output
```
Hello World from Thread-0
Hello World from main
```
> You might see `Hello World from main` output before `Hello World from Thread-0` output which completely depends on the order of thread execution.

Note: To make it more readable, the above program could be re-written as below.

```java
package com.completeinterview.multithreading;

public class Driver {
  public static void main(String[] args) {
    Thread thread =
        new Thread(
            () -> {
              while (true) {
                System.out.println("Hello World from " + Thread.currentThread().getName());
                try {
                  Thread.sleep(500);
                } catch (InterruptedException ex) {
                  System.out.println(ex.getMessage());
                }
              }
            });
    thread.setDaemon(true);
    thread.start();
    System.out.println("Hello World from " + Thread.currentThread().getName());
  }
}
```

## `join()` method

```java
package com.completeinterview.multithreading;

public class Driver {
  public static void main(String[] args) throws InterruptedException {
    Thread thread =
        new Thread(
            () -> {
              while (true) {
                System.out.println("Hello World from " + Thread.currentThread().getName());
                try {
                  Thread.sleep(500);
                } catch (InterruptedException ex) {
                  System.out.println(ex.getMessage());
                }
              }
            });
    thread.setDaemon(true);
    thread.start();
    System.out.println("Hello World from " + Thread.currentThread().getName());
    thread.join();
  }
}
```

### Output
```
Hello World from main
Hello World from Thread-0
Hello World from Thread-0
Hello World from Thread-0
Hello World from Thread-0
Hello World from Thread-0
Hello World from Thread-0
Hello World from Thread-0
Hello World from Thread-0
Hello World from Thread-0
Hello World from Thread-0
.
.
.
```

I hope you enjoyed this chapter. Connect with me on [LinkedIn](https://www.linkedin.com/in/aakashverma1124/), [Instagram](https://www.instagram.com/aakashverma1102/), and [Discord](https://discord.gg/hgvaFFXvjM).


