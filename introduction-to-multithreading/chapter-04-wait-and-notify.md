# Chapter 4: Wait and Notify

`wait()`, `notify()`, and `notifyAll()` are the methods used for thread synchronization. What is the meaning of thread synchronization? Let's understand.

Suppose we have two threads and we want them to communicate with each other. For example, we want to print numbers from 1 to 50. But we want to achieve this in the following manner:
- Every even number should be printed by `thread1`
- Every odd number should be printed by `thread2`

Now you must be thinking about why do we even need to make them communicate with each other? We can simply do it with the following code snippet:

```java
package com.completeinterview.multithreading;

class Printer {
  public void printEven(int num) {
    System.out.print(num + " ");
  }

  public void printOdd(int num) {
    System.out.print(num + " ");
  }
}

public class Driver {

  public static void main(String[] args) {
    Printer printer = new Printer();

    Thread thread1 =
        new Thread(
            () -> {
              for (int i = 2; i <= 50; i += 2) {
                printer.printEven(i);
              }
            });

    Thread thread2 =
        new Thread(
            () -> {
              for (int i = 1; i < 50; i += 2) {
                printer.printOdd(i);
              }
            });

    thread1.start();
    thread2.start();
  }
}
```

### Output
```
1 3 2 5 4 7 9 11 13 15 17 19 6 8 10 21 12 14 16 18 20 22 24 26 28 30 32 23 25 27 29 34 31 33 35 37 39 41 43 45 47 36 38 40 42 44 46 48 50 49 
```

> The above code definitely prints numbers from 1 to 50 but it randomly prints a few even numbers and then a few odd numbers. And if you run it multiple times, you might see different outputs but the format will be same i.e. it will print a few even numbers and then a few odd numbers or vice-versa.

What do we want to achieve is, we still want to print even numbers using `thread1` and odd numbers using `thread2`, but in an alternative order i.e. `1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 ...... 48 49 50`. And therefore, we really need to make them communicate with each other. And here is the concept of `wait()` and `notify()` comes into the picture.

Here is what we're going to do:
- Firstly, the `thread2` will print an odd number and then it will notify the `thread1` to print the next number and wait until `thread1` notifies again.
- Then the `thread1` will print an even number and then it will notify the `thread2` and it will wait until `thread2` notifies again.
- Just try to understand in a way that two people are communicating with each other and doing some stuff one at a time and notifying other and then waiting till they get notified again to proceed.

So, let's look at it in action.

### Code Example

```java
package com.completeinterview.multithreading;

class Printer {

  private boolean isOddTurn = true;

  public synchronized void printEven(int num) {
    while (isOddTurn) {
      try {
        wait();
      } catch (InterruptedException ignored) {

      }
    }
    System.out.print(num + " ");
    isOddTurn = true;
    notify();
  }

  public synchronized void printOdd(int num) {
    while (!isOddTurn) {
      try {
        wait();
      } catch (InterruptedException ignored) {

      }
    }
    System.out.print(num + " ");
    isOddTurn = false;
    notify();
  }
}

public class Driver {

  public static void main(String[] args) {
    Printer printer = new Printer();

    Thread thread1 =
        new Thread(
            () -> {
              for (int i = 2; i <= 50; i += 2) {
                printer.printEven(i);
              }
            });

    Thread thread2 =
        new Thread(
            () -> {
              for (int i = 1; i < 50; i += 2) {
                printer.printOdd(i);
              }
            });

    thread1.start();
    thread2.start();
  }
}
```

### Output
```
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50
```

> We had to use `synchronized` keyword because `isOddTurn` is a shared resource and both threads are trying to access it concurrently.

> Another reason for using `synchronized` keyword is because `wait()` and `notify()` requires lock ownership. These methods are part of Java's intrinsic locking mechanism and are designed to work in `synchronized` blocks or methods only. When a thread calls `wait()` on an object, it must hold the lock on that object. Without holding the lock, the JVM throws the `IllegalMonitorStateException`.

## notifyAll()
This method is the same as the `notify()` one except that it wakes up all the threads that are waiting on the object's monitor.

I hope you enjoyed this chapter. Connect with me on [LinkedIn](https://www.linkedin.com/in/aakashverma1124/), [Instagram](https://www.instagram.com/aakashverma1102/), and [Discord](https://discord.gg/hgvaFFXvjM).