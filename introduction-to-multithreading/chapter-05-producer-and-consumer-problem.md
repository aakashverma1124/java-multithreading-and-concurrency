# Problem Statement
The **Producer-Consumer Problem** is a classic synchronization problem where two types of threads (producer and consumer) share a common, fixed-size buffer. The goal is to ensure that producer doesn't overwrite data when the buffer is full, and consumer doesn't consume data when the buffer is empty. This must be achieved using `wait()` and `notify()` for thread synchronization.

## Scenario
### Producer:

- Generates data items and add them to a shared buffer.
- Must wait if the buffer is full until a consumer removes an item.

### Consumer:

- Retrieves data items from the shared buffer for processing.
- Must wait if the buffer is empty until a producer adds an item.

### Shared Buffer:

- A bounded buffer (e.g., size 5) shared between producer and consumer.

### Constraints:

- Only one thread (producer or consumer) should access the buffer at a time.
- Use `wait()` and `notify()` for thread communication to avoid busy waiting.

### Synchronization Goals:

- Producer should pause when the buffer is full.
- Consumer should pause when the buffer is empty.
- Efficiently coordinate producer and consumer actions without race conditions.

## Solution
As the problem explains this problem is a classical problem to understand the concept of thread synchronization using `wait()` and `notify()` methods.

### Code Example

```java
package com.completeinterview.multithreading;

class Buffer {
  private final int[] buffer;
  private final int capacity;
  private int index;

  public Buffer(int capacity) {
    this.buffer = new int[capacity];
    this.capacity = capacity;
    this.index = 0;
  }

  public synchronized void add(int item) throws InterruptedException {
    while (index == capacity) {
      wait();
    }
    buffer[index++] = item;
    System.out.println("Produced: " + item);
    notify();
  }

  public synchronized void remove() throws InterruptedException {
    while (index == 0) {
      wait();
    }
    int item = buffer[--index];
    System.out.println("Consumed: " + item);
    notify();
  }
}

class Producer implements Runnable {
  private final Buffer buffer;

  public Producer(Buffer buffer) {
    this.buffer = buffer;
  }

  @Override
  public void run() {
    for (int i = 1; i <= 10; i++) {
      try {
        buffer.add(i);
        Thread.sleep(1000);
      } catch (InterruptedException ignored) {

      }
    }
  }
}

class Consumer implements Runnable {
  private final Buffer buffer;

  public Consumer(Buffer buffer) {
    this.buffer = buffer;
  }

  @Override
  public void run() {
    for (int i = 1; i <= 10; i++) {
      try {
        buffer.remove();
        Thread.sleep(1000);
      } catch (InterruptedException ignored) {

      }
    }
  }
}

public class Driver {

  public static void main(String[] args) {
    Buffer buffer = new Buffer(5);
    Producer producer = new Producer(buffer);
    Consumer consumer = new Consumer(buffer);

    Thread producerThread = new Thread(producer);
    Thread consumerThread = new Thread(consumer);

    producerThread.start();
    consumerThread.start();
  }
}
```

### Output
```
Produced: 1
Consumed: 1
Produced: 2
Consumed: 2
Produced: 3
Consumed: 3
Produced: 4
Consumed: 4
Produced: 5
Consumed: 5
Produced: 6
Consumed: 6
Produced: 7
Consumed: 7
Produced: 8
Consumed: 8
Produced: 9
Consumed: 9
Produced: 10
Consumed: 10
```

You might play with `Thread.sleep(x_value)` by passing different values for `x_value` to see the difference in the output. If you keep different values, you might see different rate or production and consumption of the values.

I hope you enjoyed this chapter. Connect with me on [LinkedIn](https://www.linkedin.com/in/aakashverma1124/), [Instagram](https://www.instagram.com/aakashverma1102/), and [Discord](https://discord.gg/hgvaFFXvjM).