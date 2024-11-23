# Chapter 7: Locks

Before understanding what locks are, let's see one example where John and Sam are trying to withdraw some amount from the same bank account. In this example, we are using `synchronized` keyword on `withdraw(Integer amount)` method to ensure that only one thread can access the method at a time making sure no dirty reads are made.

### Code Example

```java
package com.completeinterview.multithreading;

class BankAccount {
  private Integer balance = 1000;

  public synchronized void withdraw(Integer amount) throws InterruptedException {
    System.out.println(Thread.currentThread().getName() + " attempting to withdraw " + amount);
    if (balance >= amount) {
      System.out.println(Thread.currentThread().getName() + " proceeding for withdrawal");
      Thread.sleep(3000); // assuming it takes some time to withdraw amount
      balance -= amount;
      System.out.println(
          Thread.currentThread().getName()
              + " completed transaction successfully. Balance: "
              + balance);
    } else {
      System.out.println("Insufficient balance.");
    }
  }
}

public class Driver {
  public static void main(String[] args) throws InterruptedException {
    BankAccount bankAccount = new BankAccount();
    Thread thread1 =
        new Thread(
            () -> {
              try {
                bankAccount.withdraw(200);
              } catch (InterruptedException e) {
                throw new RuntimeException(e);
              }
            },
            "John");

    Thread thread2 =
        new Thread(
            () -> {
              try {
                bankAccount.withdraw(200);
              } catch (InterruptedException e) {
                throw new RuntimeException(e);
              }
            },
            "Sam");

    thread1.start();
    thread2.start();
  }
}
```

### Output
```
John attempting to withdraw 200
John proceeding for withdrawal
John completed transaction successfully. Balance: 800
Sam attempting to withdraw 200
Sam proceeding for withdrawal
Sam completed transaction successfully. Balance: 600
```

> **Note:** If you don't use `synchronized` keyword, you might see the following output if you run it multiple times.
```
John attempting to withdraw 200
Sam attempting to withdraw 200
John proceeding for withdrawal
Sam proceeding for withdrawal
Sam completed transaction successfully. Balance: 800
John completed transaction successfully. Balance: 800
```

## What's the problem here?
Look at the output carefully and question yourself: **Why can't the first print statement be executed for both the threads?**

A `synchronized` block doesn't support the fairness amongst the threads. The problem here is, once the first thread acquires the lock on `withdraw()` method, the second thread doesn't have access to this method at all until the entire block is not executed by the first thread. Because the `synchronized` block is fully contained within a method, we don't have control on locking or releasing the lock. And this is why we have the concept of Lock available in `java.util.concurrent.Locks` package. We can use one of the implementation of `Lock` i.e. `ReentrantLock` to acquire and release the lock whenever we need to.

Ideally we should acquire a lock when we rea the current balance from the bank account i.e. when we execute the following conditional statement.
```
if (balance >= amount) {}
```

Let's look at the same example using `Locks`.

### Code Example

```java
package com.completeinterview.multithreading;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class BankAccount {
  private Integer balance = 1000;
  private final Lock lock = new ReentrantLock();

  public void withdraw(Integer amount) throws InterruptedException {
    System.out.println(Thread.currentThread().getName() + " attempting to withdraw " + amount);
    lock.lock();
    if (balance >= amount) {
      System.out.println(Thread.currentThread().getName() + " proceeding for withdrawal");
      Thread.sleep(3000); // assuming it takes some time to withdraw amount
      balance -= amount;
      System.out.println(
          Thread.currentThread().getName()
              + " completed transaction successfully. Balance: "
              + balance);
    } else {
      System.out.println("Insufficient balance.");
    }
    lock.unlock();
  }
}

public class Driver {
  public static void main(String[] args) throws InterruptedException {
    BankAccount bankAccount = new BankAccount();
    Thread thread1 =
        new Thread(
            () -> {
              try {
                bankAccount.withdraw(200);
              } catch (InterruptedException e) {
                throw new RuntimeException(e);
              }
            },
            "John");

    Thread thread2 =
        new Thread(
            () -> {
              try {
                bankAccount.withdraw(200);
              } catch (InterruptedException e) {
                throw new RuntimeException(e);
              }
            },
            "Sam");

    thread1.start();
    thread2.start();
  }
}
```
### Output
```
John attempting to withdraw 200
John proceeding for withdrawal
Sam attempting to withdraw 200
John completed transaction successfully. Balance: 800
Sam proceeding for withdrawal
Sam completed transaction successfully. Balance: 600
```

### Another Possible Output
```
John attempting to withdraw 200
Sam attempting to withdraw 200
John proceeding for withdrawal
John completed transaction successfully. Balance: 800
Sam proceeding for withdrawal
Sam completed transaction successfully. Balance: 600
```

Here, we are not locking the entire method instead we are making sure only the code block is locked where the shared variables are accessed. Now you might be thinking that the same logic could be achieved using `synchronized` block also. Yes, you are correct. Let's see.

### Code Example

```java
package com.completeinterview.multithreading;

class BankAccount {
  private Integer balance = 1000;

  public void withdraw(Integer amount) throws InterruptedException {
    System.out.println(Thread.currentThread().getName() + " attempting to withdraw " + amount);
    synchronized (this) {
      if (balance >= amount) {
        System.out.println(Thread.currentThread().getName() + " proceeding for withdrawal");
        Thread.sleep(3000); // assuming it takes some time to withdraw amount
        balance -= amount;
        System.out.println(
            Thread.currentThread().getName()
                + " completed transaction successfully. Balance: "
                + balance);
      } else {
        System.out.println("Insufficient balance.");
      }
    }
  }
}

public class Driver {
  public static void main(String[] args) throws InterruptedException {
    BankAccount bankAccount = new BankAccount();
    Thread thread1 =
        new Thread(
            () -> {
              try {
                bankAccount.withdraw(200);
              } catch (InterruptedException e) {
                throw new RuntimeException(e);
              }
            },
            "John");

    Thread thread2 =
        new Thread(
            () -> {
              try {
                bankAccount.withdraw(200);
              } catch (InterruptedException e) {
                throw new RuntimeException(e);
              }
            },
            "Sam");

    thread1.start();
    thread2.start();
  }
}
```

### Output
```
John attempting to withdraw 200
Sam attempting to withdraw 200
John proceeding for withdrawal
John completed transaction successfully. Balance: 800
Sam proceeding for withdrawal
Sam completed transaction successfully. Balance: 600
```

## Big Question 🤔 What the hell is `Lock` then? How is it helpful?
Let's take a very interesting example that will help you understand the exact difference and will answer the question.

Assume `Sam` takes more time than usual. For example there is some network issue and instead of taking 3-4 seconds to withdraw the amount, it is taking a little more time may be 10-15 seconds. What will happen is `John` will keep waiting for the lock to be release in case of `synchronized` block. The reason for this is that we don't have any control in case of `synchronized` block. 

How is `Lock` useful here? In case of explicit locking we have one `tryLock(long time, TimeUnit unit)` method that will wait for the lock to be released for the given unit of time and if the lock is not released in that time frame, the thread will be disabled. This way the thread won't be waiting indefinitely for the lock to be released which will eventually help us saving the resources or we can say it will save us from the problem of `Starvation`.

### Code Example

```java
package com.completeinterview.multithreading;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class BankAccount {
  private Integer balance = 1000;
  private final Lock lock = new ReentrantLock();

  public void withdraw(Integer amount) throws InterruptedException {
    System.out.println(Thread.currentThread().getName() + " attempting to withdraw " + amount);
    if (lock.tryLock(3000, TimeUnit.MILLISECONDS)) {
      if (balance >= amount) {
        System.out.println(Thread.currentThread().getName() + " proceeding for withdrawal");
        Thread.sleep(10000); // assuming it takes some time to withdraw amount
        balance -= amount;
        System.out.println(
            Thread.currentThread().getName()
                + " completed transaction successfully. Balance: "
                + balance);
      } else {
        System.out.println("Insufficient balance.");
      }
      lock.unlock();
    }
  }
}

public class Driver {
  public static void main(String[] args) throws InterruptedException {
    BankAccount bankAccount = new BankAccount();
    Thread thread1 =
        new Thread(
            () -> {
              try {
                bankAccount.withdraw(200);
              } catch (InterruptedException e) {
                throw new RuntimeException(e);
              }
            },
            "John");

    Thread thread2 =
        new Thread(
            () -> {
              try {
                bankAccount.withdraw(200);
              } catch (InterruptedException e) {
                throw new RuntimeException(e);
              }
            },
            "Sam");

    thread1.start();
    thread2.start();
  }
}   
```

### Output
```
Sam attempting to withdraw 200
John attempting to withdraw 200
Sam proceeding for withdrawal
Sam completed transaction successfully. Balance: 800
```

In this case, `Sam` starts the transaction but due to some reason it takes too long for him to complete the transaction. Meanwhile, `John` starts the transaction and keeps waiting.
But with the help of explicit locking we are able to have our own logic and fine-grained control over the thread handling. Here, `John` only waits for 3 seconds and if he is not able to acquire lock within 3 seconds, the transaction gets terminated.

I hope you enjoyed this chapter. If you are unclear with anything, please join my Discord server and post your questions there, I'll be happy to answer them. Link is given below.

Connect with me on [LinkedIn](https://www.linkedin.com/in/aakashverma1124/), [Instagram](https://www.instagram.com/aakashverma1102/), and [Discord](https://discord.gg/hgvaFFXvjM).
