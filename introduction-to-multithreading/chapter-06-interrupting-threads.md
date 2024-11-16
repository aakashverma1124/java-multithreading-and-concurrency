# Chapter 6: Interrupting Threads

In Java, interrupting a thread involves setting a flag that suggests the thread should stop execution. This mechanism is particularly useful when a long-running task needs to be aborted or when shutting down a multi-threaded application gracefully.

## InterruptedException

You will come across this exception being thrown from functions. When a thread `wait()`s or `sleep()`s then one way for it to give up waiting/sleeping is it to be interrupted. If a thread is interrupted while waiting/sleeping, it'll wake up and immediately throw InterruptedException.

The `Thread` class exposes the `interrupt()` method which can be used to interrupt a thread that is blocked in a `sleep()` or `wait()` call.

Below is an example, where a thread is initially made to sleep for an hour but then interrupted by the main thread.

```java
package com.completeinterview.multithreading;

class InterruptExample {
  public static void example() throws InterruptedException {
    Thread thread =
        new Thread(
            () -> {
              try {
                System.out.println("Sleepy thread, Sleeping for an hour");
                Thread.sleep(1000 * 60 * 60);
              } catch (InterruptedException ex) {
                System.out.println(
                    "The interrupt flag is cleared : "
                        + Thread.interrupted()
                        + " "
                        + Thread.currentThread().isInterrupted());
                Thread.currentThread().interrupt();
                System.out.println("Ohh someone woke me up.");
                System.out.println(
                    "The interrupt flag is set now : "
                        + Thread.currentThread().isInterrupted()
                        + " "
                        + Thread.interrupted());
              }
            });

    thread.start();
    System.out.println("Waking up a sleeping thread by invoking interrupt() method.");
    thread.interrupt();
    System.out.println("Woke up sleepy thread.");
    thread.join();
  }
}

public class Driver {

  public static void main(String[] args) throws InterruptedException {
    InterruptExample.example();
  }
}
```

### Output
```
Waking up a sleeping thread by invoking interrupt() method.
Woke up sleepy thread.
Sleepy thread, Sleeping for an hour
The interrupt flag is cleared : false false
Ohh someone woke me up.
The interrupt flag is set now : true true
```

### Explanation

- When `thread.interrupt()` is called from main thread, the interrupt flag of the thread is set to true.
- The sleeping thread is interrupted, so an `InterruptedException` is thrown. At this point, the interrupt flag is cleared automatically by the JVM, as per the contract of `InterruptedException`.
- And that's the reason `false false` is printed. (You might be expecting `true true`).
- Then we executed `Thread.currentThread().interrupt()`, but this time no `InterruptedException` is thrown because after the thread was interrupted and the `InterruptedException` was thrown, the thread is no longer in a blocking state.

An `InterruptedException` is thrown only when a thread is interrupted while it is in a blocking state, such as:
- `Thread.sleep()`
- `Object.wait()`
- `Thread.join()`
- `BlockingQueue.put()` or `take()`

Setting the interrupt flag (Thread.currentThread().interrupt()) only marks the thread as interrupted—it doesn't directly throw an exception unless the thread performs a blocking operation again.

I hope you enjoyed this chapter. If you are unclear with anything, please join my Discord server and post your questions there, I'll be happy to answer them. Link is given below.

Connect with me on [LinkedIn](https://www.linkedin.com/in/aakashverma1124/), [Instagram](https://www.instagram.com/aakashverma1102/), and [Discord](https://discord.gg/hgvaFFXvjM).