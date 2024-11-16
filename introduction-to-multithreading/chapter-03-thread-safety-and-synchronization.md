# Chapter 3: Thread Safety and Synchronization

## Thread Safe
A class and its public APIs are labelled as thread safe if multiple threads can consume the exposed APIs without causing race conditions or state corruption for the class.

## Synchronization
Java's most fundamental construct for thread synchronization is the `synchronized` keyword. It can be used to restrict access to critical sections one thread at a time.

## Understanding the background of the problem

Look at the below code snippet carefully:

```java
package com.completeinterview.multithreading;

class Counter {
  private Integer count = 0;

  public void increment() {
    this.count += 1;
  }

  public Integer getCount() {
    return this.count;
  }
}

public class Driver {
  public static void main(String[] args) throws InterruptedException {

    Counter counter = new Counter();

    Thread thread1 =
        new Thread(
            () -> {
              for (int i = 0; i < 1000; i++) {
                counter.increment();
              }
            });

    Thread thread2 =
        new Thread(
            () -> {
              for (int i = 0; i < 1000; i++) {
                counter.increment();
              }
            });

    thread1.start();
    thread2.start();

    System.out.println("Total Count: " + counter.getCount());
  }
}
```
In the above code, we have a `Counter` class with a `count` variable that is incremented 1000 times by two threads. Can you guess the value of the `count` variable? If you run it multiple times, you will see different output.

If you look carefully, here 3 different threads are running simultaneously i.e. `main` thread, `thread1` and `thread2`. So `System.out.println("Total Count: " + counter.getCount());` is executed before the other two threads completes and that's the reason, you won't see exact 2000 printed on the console. 

So can we use `join()` to join multiple threads? Okay let's see.

```java
package com.completeinterview.multithreading;

class Counter {
  private Integer count = 0;

  public void increment() {
    this.count += 1;
  }

  public Integer getCount() {
    return this.count;
  }
}

public class Driver {
  public static void main(String[] args) throws InterruptedException {

    Counter counter = new Counter();

    Thread thread1 =
        new Thread(
            () -> {
              for (int i = 0; i < 1000; i++) {
                counter.increment();
              }
            });

    Thread thread2 =
        new Thread(
            () -> {
              for (int i = 0; i < 1000; i++) {
                counter.increment();
              }
            });

    thread1.start();
    thread2.start();

    thread1.join();
    thread2.join();

    System.out.println("Total Count: " + counter.getCount());
  }
}
```

Now we have joined both the threads using `join()` method. Now do you think we would get 2000? The answer is No. We won't get 2000 because the `join()` method definitely waits for the threads to complete before executing `System.out.println("Total Count: " + counter.getCount());` but the problem here is the `increment()` method is not thread safe.

The problem is, while incrementing the counter these threads might be simultaneously reading the same value and that's the reason we are not able to see 2000 printed on the console.

If we want to make sure that it prints 2000 we need to make sure that the `increment()` method is thread safe. And therefore we have a `synchronized` keyword in java.

### Code Example

```java
package com.completeinterview.multithreading;

class Counter {
  private Integer count = 0;

  public synchronized void increment() {
    this.count += 1;
  }

  public Integer getCount() {
    return this.count;
  }
}

public class Driver {
  public static void main(String[] args) throws InterruptedException {

    Counter counter = new Counter();

    Thread thread1 =
        new Thread(
            () -> {
              for (int i = 0; i < 1000; i++) {
                counter.increment();
              }
            });

    Thread thread2 =
        new Thread(
            () -> {
              for (int i = 0; i < 1000; i++) {
                counter.increment();
              }
            });

    thread1.start();
    thread2.start();

    thread1.join();
    thread2.join();

    System.out.println("Total Count: " + counter.getCount());
  }
}
```

### Output
```
Total Count: 2000
```

I hope you enjoyed this chapter. Connect with me on [LinkedIn](https://www.linkedin.com/in/aakashverma1124/), [Instagram](https://www.instagram.com/aakashverma1102/), and [Discord](https://discord.gg/hgvaFFXvjM).