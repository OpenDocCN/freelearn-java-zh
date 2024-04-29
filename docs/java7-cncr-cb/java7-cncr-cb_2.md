# 第二章：基本线程同步

在本章中，我们将涵盖：

+   同步一个方法

+   在同步类中排列独立属性

+   在同步代码中使用条件

+   使用锁同步代码块

+   使用读/写锁同步数据访问

+   修改锁的公平性

+   在锁中使用多个条件

# 介绍

并发编程中最常见的情况之一是多个执行线程共享资源。在并发应用程序中，多个线程读取或写入相同的数据，或者访问相同的文件或数据库连接是正常的。这些共享资源可能引发错误情况或数据不一致，我们必须实现机制来避免这些错误。

这些问题的解决方案是通过**关键部分**的概念得到的。关键部分是指访问共享资源的代码块，不能同时由多个线程执行。

为了帮助程序员实现关键部分，Java（以及几乎所有编程语言）提供了**同步**机制。当一个线程想要访问关键部分时，它使用这些同步机制之一来查找是否有其他线程正在执行关键部分。如果没有，线程就进入关键部分。否则，线程被同步机制挂起，直到正在执行关键部分的线程结束。当多个线程等待一个线程完成关键部分的执行时，JVM 会选择其中一个，其余的等待他们的轮到。

本章介绍了一些教授如何使用 Java 语言提供的两种基本同步机制的方法：

+   关键字`synchronized`

+   `Lock`接口及其实现

# 同步一个方法

在这个示例中，我们将学习如何使用 Java 中最基本的同步方法之一，即使用`Synchronized`关键字来控制对方法的并发访问。只有一个执行线程将访问使用`Synchronized`关键字声明的对象的方法。如果另一个线程尝试访问同一对象的任何使用`Synchronized`关键字声明的方法，它将被挂起，直到第一个线程完成方法的执行。

换句话说，使用`Synchronized`关键字声明的每个方法都是一个关键部分，Java 只允许执行对象的一个关键部分。

静态方法有不同的行为。只有一个执行线程将访问使用`Synchronized`关键字声明的静态方法之一，但另一个线程可以访问该类对象的其他非静态方法。在这一点上你必须非常小心，因为如果一个是静态的，另一个不是，两个线程可以访问两个不同的`Synchronized`方法。如果这两个方法都改变了相同的数据，就可能出现数据不一致的错误。

为了学习这个概念，我们将实现一个示例，其中有两个线程访问一个共同的对象。我们将有一个银行账户和两个线程；一个向账户转账，另一个从账户取款。没有同步方法，我们可能会得到不正确的结果。同步机制确保账户的最终余额是正确的。

## 准备就绪

这个示例已经在 Eclipse IDE 中实现。如果你使用 Eclipse 或其他 IDE，比如 NetBeans，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`Account`的类来模拟我们的银行账户。它只有一个名为`balance`的`double`属性。

```java
public class Account {
      private double balance;
```

1.  实现`setBalance()`和`getBalance()`方法来写入和读取属性的值。

```java
  public double getBalance() {
    return balance;
  }

  public void setBalance(double balance) {
    this.balance = balance;
  }
```

1.  实现一个名为`addAmount()`的方法，该方法增加传递给方法的特定金额的余额值。只有一个线程应该更改余额的值，因此使用`synchronized`关键字将此方法转换为临界区。

```java
  public synchronized void addAmount(double amount) {
    double tmp=balance;
    try {
      Thread.sleep(10);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    tmp+=amount;
    balance=tmp;
  }
```

1.  实现一个名为`subtractAmount()`的方法，该方法减少传递给方法的特定金额的余额值。只有一个线程应该更改余额的值，因此使用`synchronized`关键字将此方法转换为临界区。

```java
  public synchronized void subtractAmount(double amount) {
    double tmp=balance;
    try {
      Thread.sleep(10);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    tmp-=amount;
    balance=tmp;
  }
```

1.  实现一个模拟 ATM 的类。它将使用`subtractAmount()`方法来减少账户的余额。这个类必须实现`Runnable`接口以作为线程执行。

```java
public class Bank implements Runnable {
```

1.  将一个`Account`对象添加到这个类中。实现初始化该`Account`对象的类的构造函数。

```java
  private Account account;

  public Bank(Account account) {
    this.account=account;
  }
```

1.  实现`run()`方法。它对一个账户进行`100`次`subtractAmount()`方法的调用以减少余额。

```java
  @Override
   public void run() {
    for (int i=0; i<100; i++){
      account.sustractAmount(1000);
    }
  }
```

1.  实现一个模拟公司的类，并使用`Account`类的`addAmount()`方法来增加账户的余额。这个类必须实现`Runnable`接口以作为线程执行。

```java
public class Company implements Runnable {
```

1.  将一个`Account`对象添加到这个类中。实现初始化该账户对象的类的构造函数。

```java
  private Account account;

  public Company(Account account) {
    this.account=account;
  }
```

1.  实现`run()`方法。它对一个账户进行`100`次`addAmount()`方法的调用以增加余额。

```java
  @Override
   public void run() {
    for (int i=0; i<100; i++){
      account.addAmount(1000);
    }
  }
```

1.  通过创建一个名为`Main`的类并包含`main()`方法来实现应用程序的主类。

```java
public class Main {

  public static void main(String[] args) {
```

1.  创建一个`Account`对象并将其余额初始化为`1000`。

```java
    Account  account=new Account();
    account.setBalance(1000);
```

1.  创建一个`Company`对象和一个`Thread`来运行它。

```java
    Company  company=new Company(account);
    Thread companyThread=new Thread(company);  
```

1.  创建一个`Bank`对象和一个`Thread`来运行它。

```java
    Bank bank=new Bank(account);
    Thread bankThread=new Thread(bank);
```

1.  将初始余额写入控制台。

```java
    System.out.printf("Account : Initial Balance: %f\n",account.getBalance());
Start the threads.
    companyThread.start();
    bankThread.start();
```

1.  使用`join()`方法等待两个线程的完成，并在控制台中打印出账户的最终余额。

```java
    try {
      companyThread.join();
      bankThread.join();
      System.out.printf("Account : Final Balance: %f\n",account.getBalance());
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
```

## 它是如何工作的...

在这个示例中，您已经开发了一个应用程序，该应用程序增加和减少了模拟银行账户余额的类的余额。该程序对`addAmount()`方法进行了`100`次调用，每次调用都会将余额增加`1000`，并对`subtractAmount()`方法进行了`100`次调用，每次调用都会将余额减少`1000`。您应该期望最终余额和初始余额相等。

您已经尝试使用一个名为`tmp`的变量来存储账户余额的值，因此您读取了账户余额，增加了临时变量的值，然后再次设置了账户余额的值。此外，您还使用了`Thread`类的`sleep()`方法引入了一点延迟，以便执行该方法的线程休眠 10 毫秒，因此如果另一个线程执行该方法，它可能会修改账户余额，从而引发错误。正是`synchronized`关键字机制避免了这些错误。

如果您想看到共享数据并发访问的问题，请删除`addAmount()`和`subtractAmount()`方法的`synchronized`关键字并运行程序。没有`synchronized`关键字，当一个线程在读取账户余额的值后休眠时，另一个方法将读取账户余额，因此两个方法都将修改相同的余额，其中一个操作不会反映在最终结果中。

正如您在下面的屏幕截图中所看到的，您可能会得到不一致的结果：

![它是如何工作的...](img/7881_02_01.jpg)

如果您经常运行程序，您将获得不同的结果。线程的执行顺序不受 JVM 保证。因此，每次执行它们时，线程都将以不同的顺序读取和修改账户的余额，因此最终结果将不同。

现在，按照之前学到的方法添加`synchronize`关键字，并再次运行程序。如下截图所示，现在您可以获得预期的结果。如果经常运行程序，您将获得相同的结果。请参考以下截图：

![工作原理...](img/7881_02_02.jpg)

使用`synchronized`关键字，我们可以保证并发应用程序中对共享数据的正确访问。

正如我们在本节介绍中提到的，只有一个线程可以访问使用`synchronized`关键字声明的对象的方法。如果一个线程（A）正在执行一个`synchronized`方法，另一个线程（B）想要执行同一对象的其他`synchronized`方法，它将被阻塞，直到线程（A）结束。但是如果 threadB 可以访问同一类的不同对象，则它们都不会被阻塞。

## 还有更多...

`synchronized`关键字会降低应用程序的性能，因此您只能在并发环境中修改共享数据的方法上使用它。如果有多个线程调用`synchronized`方法，只有一个线程会一次执行它们，而其他线程将等待。如果操作不使用`synchronized`关键字，则所有线程可以同时执行操作，从而减少总执行时间。如果您知道某个方法不会被多个线程调用，请不要使用`synchronized`关键字。

您可以使用带有`synchronized`方法的递归调用。由于线程可以访问对象的`synchronized`方法，因此可以调用该对象的其他`synchronized`方法，包括正在执行的方法。它不必再次访问`synchronized`方法。

我们可以使用`synchronized`关键字来保护对一段代码的访问，而不是整个方法。我们应该以这种方式使用`synchronized`关键字来保护对共享数据的访问，将其余操作排除在此块之外，从而获得更好的应用性能。目标是使关键部分（一次只能由一个线程访问的代码块）尽可能短。我们已经使用`synchronized`关键字来保护对更新建筑物中人数的指令的访问，排除了不使用共享数据的此块的长操作。当您以这种方式使用`synchronized`关键字时，必须将对象引用作为参数传递。只有一个线程可以访问该对象的`synchronized`代码（块或方法）。通常，我们会使用`this`关键字来引用执行方法的对象。

```java
    synchronized (this) {
      // Java code
    }
```

# 安排同步类中的独立属性

当您使用`synchronized`关键字来保护一段代码时，您必须将一个对象引用作为参数传递。通常，您会使用`this`关键字来引用执行方法的对象，但您也可以使用其他对象引用。通常，这些对象将专门为此目的创建。例如，如果一个类中有两个独立的属性被多个线程共享，您必须同步对每个变量的访问，但如果一个线程同时访问其中一个属性，另一个线程访问另一个属性，则不会有问题。

在本节中，您将学习如何通过一个示例来解决这种情况的编程，该示例模拟了一个具有两个屏幕和两个售票处的电影院。当售票处出售票时，它们是为两个电影院中的一个而不是两个，因此每个电影院中的空座位数是独立的属性。

## 准备工作

本节示例已使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE（如 NetBeans），请打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`Cinema`的类，并向其添加两个名为`vacanciesCinema1`和`vacanciesCinema2`的`long`属性。

```java
public class Cinema {

  private long vacanciesCinema1;
  private long vacanciesCinema2;
```

1.  在`Cinema`类中添加两个额外的`Object`属性，命名为`controlCinema1`和`controlCinema2`。

```java
  private final Object controlCinema1, controlCinema2;
```

1.  实现`Cinema`类的构造函数，初始化类的所有属性。

```java
  public Cinema(){
    controlCinema1=new Object();
    controlCinema2=new Object();
    vacanciesCinema1=20;
    vacanciesCinema2=20;
  }
```

1.  实现`sellTickets1()`方法，当第一个电影院的一些票被售出时调用。它使用`controlCinema1`对象来控制对`同步`代码块的访问。

```java
  public boolean sellTickets1 (int number) {
    synchronized (controlCinema1) {
      if (number<vacanciesCinema1) {
        vacanciesCinema1-=number;
        return true;
      } else {
        return false;
      }
    }
  }
```

1.  实现`sellTickets2()`方法，当第二个电影院的一些票被售出时调用。它使用`controlCinema2`对象来控制对`同步`代码块的访问。

```java
  public boolean sellTickets2 (int number){
    synchronized (controlCinema2) {
      if (number<vacanciesCinema2) {
        vacanciesCinema2-=number;
        return true;
      } else {
        return false;
      }
    }
  }
```

1.  实现`returnTickets1()`方法，当第一个电影院的一些票被退回时调用。它使用`controlCinema1`对象来控制对`同步`代码块的访问。

```java
  public boolean returnTickets1 (int number) {
    synchronized (controlCinema1) {
      vacanciesCinema1+=number;
      return true;
    }
  }
```

1.  实现`returnTickets2()`方法，当第二个电影院的一些票被退回时调用。它使用`controlCinema2`对象来控制对`同步`代码块的访问。

```java
  public boolean returnTickets2 (int number) {
    synchronized (controlCinema2) {
      vacanciesCinema2+=number;
      return true;
    }
  }
```

1.  实现另外两个方法，返回每个电影院的空位数。

```java
  public long getVacanciesCinema1() {
    return vacanciesCinema1;
  }

  public long getVacanciesCinema2() {
    return vacanciesCinema2;
  }
```

1.  实现`TicketOffice1`类，并指定它实现`Runnable`接口。

```java
public class TicketOffice1 implements Runnable {
```

1.  声明一个`Cinema`对象，并实现该类的构造函数来初始化该对象。

```java
  private Cinema cinema;

  public TicketOffice1 (Cinema cinema) {
    this.cinema=cinema;
  }
```

1.  实现`run()`方法，模拟对两个电影院的一些操作。

```java
  @Override
   public void run() {
    cinema.sellTickets1(3);
    cinema.sellTickets1(2);
    cinema.sellTickets2(2);
    cinema.returnTickets1(3);
    cinema.sellTickets1(5);
    cinema.sellTickets2(2);
    cinema.sellTickets2(2);
    cinema.sellTickets2(2);
  }
```

1.  实现`TicketOffice2`类，并指定它实现`Runnable`接口。

```java
public class TicketOffice2 implements Runnable {
```

1.  声明一个`Cinema`对象，并实现该类的构造函数来初始化该对象。

```java
  private Cinema cinema;

  public TicketOffice2(Cinema cinema){
    this.cinema=cinema;
  }
```

1.  实现`run()`方法，模拟对两个电影院的一些操作。

```java
  @Override
  public void run() {
    cinema.sellTickets2(2);
    cinema.sellTickets2(4);
    cinema.sellTickets1(2);
    cinema.sellTickets1(1);
    cinema.returnTickets2(2);
    cinema.sellTickets1(3);
    cinema.sellTickets2(2);
    cinema.sellTickets1(2);
  }
```

1.  通过创建一个名为`Main`的类并向其中添加`main()`方法来实现示例的主类。

```java
public class Main {

  public static void main(String[] args) {
```

1.  声明并创建一个`Cinema`对象。

```java
    Cinema cinema=new Cinema();
```

1.  创建一个`TicketOffice1`对象和`Thread`来执行它。

```java
    TicketOffice1 ticketOffice1=new TicketOffice1(cinema);
    Thread thread1=new Thread(ticketOffice1,"TicketOffice1");
```

1.  创建一个`TicketOffice2`对象和`Thread`来执行它。

```java
    TicketOffice2 ticketOffice2=new TicketOffice2(cinema);
    Thread thread2=new Thread(ticketOffice2,"TicketOffice2");
```

1.  启动两个线程。

```java
    thread1.start();
    thread2.start();
```

1.  等待线程完成。

```java
    try {
      thread1.join();
      thread2.join();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
```

1.  将两个电影院的空位数写入控制台。

```java
    System.out.printf("Room 1 Vacancies: %d\n",cinema.getVacanciesCinema1());
    System.out.printf("Room 2 Vacancies: %d\n",cinema.getVacanciesCinema2());
```

## 它是如何工作的...

当使用`同步`关键字保护一段代码时，使用一个对象作为参数。JVM 保证只有一个线程可以访问使用该对象保护的所有代码块（请注意，我们总是谈论对象，而不是类）。

### 注意

在这个示例中，我们有一个对象来控制对`vacanciesCinema1`属性的访问，因此每次只有一个线程可以修改这个属性，另一个对象控制对`vacanciesCinema2`属性的访问，因此每次只有一个线程可以修改这个属性。但可能会有两个线程同时运行，一个修改`vacancesCinema1`属性，另一个修改`vacanciesCinema2`属性。

当运行此示例时，您可以看到最终结果始终是每个电影院预期的空位数。在下面的屏幕截图中，您可以看到应用程序执行的结果：

![它是如何工作的...](img/7881_02_03.jpg)

## 还有更多...

`同步`关键字还有其他重要的用途。请参阅*另请参阅*部分，了解其他解释此关键字用法的示例。

## 另请参阅

+   在第二章的*基本线程同步*中的*在同步代码中使用条件*示例中

# 在同步代码中使用条件

并发编程中的一个经典问题是**生产者-消费者**问题。我们有一个数据缓冲区，一个或多个生产者将数据保存在缓冲区中，一个或多个消费者从缓冲区中取数据。

由于缓冲区是共享数据结构，我们必须使用同步机制来控制对它的访问，比如`同步`关键字，但我们有更多的限制。如果缓冲区已满，生产者就不能将数据保存在缓冲区中，如果缓冲区为空，消费者就不能从缓冲区中取数据。

对于这种情况，Java 提供了在`Object`类中实现的`wait()`、`notify()`和`notifyAll()`方法。线程可以在`同步`代码块中调用`wait()`方法。如果它在`同步`代码块之外调用`wait()`方法，JVM 会抛出`IllegalMonitorStateException`异常。当线程调用`wait()`方法时，JVM 会让线程进入睡眠状态，并释放控制`同步`代码块的对象，允许其他线程执行由该对象保护的其他`同步`代码块。要唤醒线程，必须在由相同对象保护的代码块中调用`notify()`或`notifyAll()`方法。

在这个示例中，您将学习如何使用`同步`关键字和`wait()`、`notify()`和`notifyAll()`方法来实现生产者-消费者问题。

## 准备工作

这个示例的实现使用了 Eclipse IDE。如果您使用 Eclipse 或其他 IDE，比如 NetBeans，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`EventStorage`的类。它有两个属性：一个名为`maxSize`的`int`属性和一个名为`storage`的`LinkedList<Date>`属性。

```java
public class EventStorage {

  private int maxSize;
  private List<Date> storage;
```

1.  实现初始化类属性的类构造函数。

```java
  public EventStorage(){
    maxSize=10;
    storage=new LinkedList<>();
  }
```

1.  实现`同步`方法`set()`以将事件存储在存储中。首先，检查存储是否已满。如果满了，调用`wait()`方法直到存储有空余空间。在方法结束时，调用`notifyAll()`方法唤醒所有在`wait()`方法中睡眠的线程。

```java
  public synchronized void set(){
      while (storage.size()==maxSize){
        try {
          wait();
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
      }
      storage.offer(new Date());
      System.out.printf("Set: %d",storage.size());
      notifyAll();
  }
```

1.  实现`同步`方法`get()`以获取存储的事件。首先，检查存储是否有事件。如果没有事件，调用`wait()`方法，直到存储有事件为止。在方法结束时，调用`notifyAll()`方法唤醒所有在`wait()`方法中睡眠的线程。

```java
  public synchronized void get(){
      while (storage.size()==0){
        try {
          wait();
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
      }
       System.out.printf("Get: %d: %s",storage.size(),((LinkedList<?>)storage).poll());
      notifyAll();
  }
```

1.  创建一个名为`Producer`的类，并指定它实现`Runnable`接口。它将实现示例的生产者。

```java
public class Producer implements Runnable {
```

1.  声明一个`EventStore`对象并实现初始化该对象的类构造函数。

```java
  private EventStorage storage;

  public Producer(EventStorage storage){
    this.storage=storage;
  }
```

1.  实现调用`EventStorage`对象的`set()`方法`100`次的`run()`方法。

```java
   @Override
  public void run() {
    for (int i=0; i<100; i++){
      storage.set();
    }
  }
```

1.  创建一个名为`Consumer`的类，并指定它实现`Runnable`接口。它将实现示例的消费者。

```java
public class Consumer implements Runnable {
```

1.  声明一个`EventStorage`对象并实现初始化该对象的类构造函数。

```java
  private EventStorage storage;

  public Consumer(EventStorage storage){
    this.storage=storage;
  }
```

1.  实现`run()`方法。它调用`EventStorage`对象的`get()`方法`100`次。

```java
  @Override
   public void run() {
    for (int i=0; i<100; i++){
      storage.get();
    }
  }
```

1.  通过实现一个名为`Main`的类并添加`main()`方法来创建示例的主类。

```java
public class Main {

  public static void main(String[] args) {
```

1.  创建一个`EventStorage`对象。

```java
    EventStorage storage=new EventStorage();
```

1.  创建一个`Producer`对象和一个`Thread`来运行它。

```java
    Producer producer=new Producer(storage);
    Thread thread1=new Thread(producer);
```

1.  创建一个`Consumer`对象和一个`Thread`来运行它。

```java
    Consumer consumer=new Consumer(storage);
    Thread thread2=new Thread(consumer);
```

1.  启动两个线程。

```java
    thread2.start();
    thread1.start();
```

## 它是如何工作的...

这个例子的关键是`EventStorage`类的`set()`和`get()`方法。首先，`set()`方法检查存储属性中是否有空闲空间。如果满了，调用`wait()`方法等待空闲空间。当其他线程调用`notifyAll()`方法时，线程会被唤醒并再次检查条件。`notifyAll()`方法不能保证线程会被唤醒。这个过程会重复，直到存储中有空闲空间并且可以生成新的事件并存储它。

`get()`方法的行为类似。首先，它检查存储中是否有事件。如果`EventStorage`类为空，调用`wait()`方法等待事件。当其他线程调用`notifyAll()`方法时，线程会被唤醒并再次检查条件，直到存储中有事件为止。

### 注意

您必须不断检查条件，并在`while`循环中调用`wait()`方法。直到条件为`true`为止，您才能继续。

如果您运行此示例，您将看到生产者和消费者如何设置和获取事件，但存储中从未有超过 10 个事件。

## 还有更多...

`synchronized`关键字还有其他重要的用途。请参阅*另请参阅*部分，了解解释此关键字用法的其他配方。

## 另请参阅

+   第二章中的*在同步类中排列独立属性*配方，*基本线程同步*

# 使用锁同步代码块

Java 提供了另一种用于同步代码块的机制。这是一种比`synchronized`关键字更强大和灵活的机制。它基于`Lock`接口和实现它的类（如`ReentrantLock`）。这种机制具有一些优势，如下所示：

+   它允许以更灵活的方式构造同步块。使用`synchronized`关键字，您必须以结构化的方式获取和释放同步代码块的控制权。`Lock`接口允许您获得更复杂的结构来实现您的临界区。

+   `Lock`接口提供了比`synchronized`关键字更多的功能。其中一个新功能是`tryLock()`方法。此方法尝试获取锁的控制权，如果无法获取（因为它被其他线程使用），则返回该锁。使用`synchronized`关键字时，当线程（A）尝试执行同步代码块时，如果有另一个线程（B）正在执行它，线程（A）将被挂起，直到线程（B）完成同步块的执行。使用锁，您可以执行`tryLock()`方法。此方法返回一个`Boolean`值，指示是否有另一个线程运行由此锁保护的代码。

+   `Lock`接口允许对读和写操作进行分离，具有多个读取者和仅一个修改者。

+   `Lock`接口的性能比`synchronized`关键字更好。

在这个配方中，您将学习如何使用锁来同步代码块，并使用`Lock`接口和实现它的`ReentrantLock`类创建临界区，实现一个模拟打印队列的程序。

## 准备就绪...

这个配方的示例是使用 Eclipse IDE 实现的。如果您使用 Eclipse 或其他 IDE，如 NetBeans，请打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`PrintQueue`的类，它将实现打印队列。

```java
public class PrintQueue {
```

1.  声明一个`Lock`对象，并使用`ReentrantLock`类的新对象对其进行初始化。

```java
  private final Lock queueLock=new ReentrantLock();
```

1.  实现`printJob()`方法。它将接收`Object`作为参数，并不会返回任何值。

```java
  public void printJob(Object document){
```

1.  在`printJob()`方法内部，通过调用`lock()`方法获取`Lock`对象的控制权。

```java
    queueLock.lock();
```

1.  然后，包括以下代码来模拟打印文档：

```java
    try {
      Long duration=(long)(Math.random()*10000);
      System.out.println(Thread.currentThread().getName()+ ": PrintQueue: Printing a Job during "+(duration/1000)+ 
" seconds");
      Thread.sleep(duration);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
```

1.  最后，使用`unlock()`方法释放`Lock`对象的控制权。

```java
finally {
      queueLock.unlock();
    }    
```

1.  创建一个名为`Job`的类，并指定它实现`Runnable`接口。

```java
public class Job implements Runnable {
```

1.  声明一个`PrintQueue`类的对象，并实现初始化该对象的类的构造函数。

```java
  private PrintQueue printQueue;

  public Job(PrintQueue printQueue){
    this.printQueue=printQueue;
  }
```

1.  实现`run()`方法。它使用`PrintQueue`对象发送打印作业。

```java
  @Override
  public void run() {
    System.out.printf("%s: Going to print a document\n", Thread.currentThread().getName());
    printQueue.printJob(new Object());
    System.out.printf("%s: The document has been printed\n", Thread.currentThread().getName());    
  }
```

1.  通过实现一个名为`Main`的类并向其中添加`main()`方法，创建应用程序的主类。

```java
public class Main {

  public static void main (String args[]){
```

1.  创建一个共享的`PrintQueue`对象。

```java
    PrintQueue printQueue=new PrintQueue();
```

1.  创建 10 个`Job`对象和 10 个线程来运行它们。

```java
    Thread thread[]=new Thread[10];
    for (int i=0; i<10; i++){
      thread[i]=new Thread(new Job(printQueue),"Thread "+ i);
    }
```

1.  启动 10 个线程。

```java
    for (int i=0; i<10; i++){
      thread[i].start();
    }
```

## 它是如何工作的...

在下面的屏幕截图中，您可以看到一个执行的部分输出，例如：

![它是如何工作的...](img/7881_02_04.jpg)

示例的关键在于`PrintQueue`类的`printJob()`方法。当我们想要使用锁实现临界区并确保只有一个执行线程运行代码块时，我们必须创建一个`ReentrantLock`对象。在临界区的开始，我们必须使用`lock()`方法获取锁的控制权。当一个线程（A）调用此方法时，如果没有其他线程控制着锁，该方法将给予线程（A）锁的控制权，并立即返回以允许该线程执行临界区。否则，如果有另一个线程（B）执行由此锁控制的临界区，`lock()`方法将使线程（A）进入休眠状态，直到线程（B）完成临界区的执行。

在临界区的结束，我们必须使用`unlock()`方法释放锁的控制权，并允许其他线程运行此临界区。如果在临界区结束时不调用`unlock()`方法，那些正在等待该块的其他线程将永远等待，导致死锁情况。如果在临界区中使用 try-catch 块，请不要忘记将包含`unlock()`方法的语句放在`finally`部分中。

## 还有更多...

`Lock`接口（以及`ReentrantLock`类）包括另一个方法来获取锁的控制权。这就是`tryLock()`方法。与`lock()`方法最大的区别在于，如果使用它的线程无法获得`Lock`接口的控制权，该方法将立即返回，而不会使线程进入休眠状态。该方法返回一个`boolean`值，如果线程获得了锁的控制权，则返回`true`，否则返回`false`。

### 注意

请注意，程序员有责任考虑此方法的结果并相应地采取行动。如果该方法返回`false`值，则预期您的程序不会执行临界区。如果执行了，您的应用程序可能会产生错误的结果。

`ReentrantLock`类还允许使用递归调用。当一个线程控制着一个锁并进行递归调用时，它将继续控制着锁，因此调用`lock()`方法将立即返回，线程将继续执行递归调用。此外，我们还可以调用其他方法。

### 更多信息

您必须非常小心地使用`Locks`以避免**死锁**。当两个或更多线程被阻塞等待永远不会被解锁的锁时，就会发生这种情况。例如，一个线程（A）锁定了一个锁（X），而另一个线程（B）锁定了一个锁（Y）。如果现在，线程（A）尝试锁定锁（Y），而线程（B）同时尝试锁定锁（X），那么两个线程将无限期地被阻塞，因为它们正在等待永远不会被释放的锁。请注意，问题出现在于两个线程尝试以相反的顺序获取锁。附录*并发编程设计*解释了一些设计并发应用程序并避免这些死锁问题的好建议。

## 另请参阅

+   在第二章的*基本线程同步*中的*同步方法*配方

+   在第二章的*基本线程同步*中的*在锁中使用多个条件*配方中

+   在第八章的*测试并发应用*中的*监视锁*接口配方

# 使用读/写锁同步数据访问

锁提供的最重要的改进之一是`ReadWriteLock`接口和`ReentrantReadWriteLock`类，它是唯一实现它的类。这个类有两个锁，一个用于读操作，一个用于写操作。可以有多个线程同时使用读操作，但只能有一个线程使用写操作。当一个线程执行写操作时，不能有任何线程执行读操作。

在本示例中，您将学习如何使用`ReadWriteLock`接口来实现一个程序，该程序使用它来控制对存储两种产品价格的对象的访问。

## 准备就绪...

您应该阅读*Synchronizing a block of code with a Lock*一节，以更好地理解本节。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`PricesInfo`的类，用于存储两种产品的价格信息。

```java
public class PricesInfo {
```

1.  声明两个名为`price1`和`price2`的`double`属性。

```java
  private double price1;
  private double price2;
```

1.  声明一个名为`lock`的`ReadWriteLock`对象。

```java
  private ReadWriteLock lock;
```

1.  实现初始化三个属性的类的构造函数。对于`lock`属性，我们创建一个新的`ReentrantReadWriteLock`对象。

```java
  public PricesInfo(){
    price1=1.0;
    price2=2.0;
    lock=new ReentrantReadWriteLock();
  }
```

1.  实现`getPrice1()`方法，该方法返回`price1`属性的值。它使用读锁来控制对该属性值的访问。

```java
  public double getPrice1() {
    lock.readLock().lock();
    double value=price1;
    lock.readLock().unlock();
    return value;
  }
```

1.  实现`getPrice2()`方法，该方法返回`price2`属性的值。它使用读锁来控制对该属性值的访问。

```java
  public double getPrice2() {
    lock.readLock().lock();
    double value=price2;
    lock.readLock().unlock();
    return value;
  }
```

1.  实现`setPrices()`方法，用于设置两个属性的值。它使用写锁来控制对它们的访问。

```java
  public void setPrices(double price1, double price2) {
    lock.writeLock().lock();
    this.price1=price1;
    this.price2=price2;
    lock.writeLock().unlock();
  }
```

1.  创建一个名为`Reader`的类，并指定它实现`Runnable`接口。该类实现了`PricesInfo`类属性值的读取器。

```java
public class Reader implements Runnable {
```

1.  声明一个名为`PricesInfo`的对象，并实现初始化该对象的类的构造函数。

```java
  private PricesInfo pricesInfo;

  public Reader (PricesInfo pricesInfo){
    this.pricesInfo=pricesInfo;
  }
```

1.  为这个类实现`run()`方法。它读取两个价格的值 10 次。

```java
  @Override
  public void run() {
    for (int i=0; i<10; i++){
      System.out.printf("%s: Price 1: %f\n", Thread.currentThread().getName(),pricesInfo.getPrice1());
      System.out.printf("%s: Price 2: %f\n", Thread.currentThread().getName(),pricesInfo.getPrice2());
    }
  }
```

1.  创建一个名为`Writer`的类，并指定它实现`Runnable`接口。该类实现了`PricesInfo`类属性值的修改器。

```java
public class Writer implements Runnable {
```

1.  声明一个名为`PricesInfo`的对象，并实现初始化该对象的类的构造函数。

```java
  private PricesInfo pricesInfo;

  public Writer(PricesInfo pricesInfo){
    this.pricesInfo=pricesInfo;
  }
```

1.  实现`run()`方法。它在修改两个价格的值之间休眠两秒，共修改三次。

```java
  @Override
  public void run() {
    for (int i=0; i<3; i++) {
      System.out.printf("Writer: Attempt to modify the prices.\n");
      pricesInfo.setPrices(Math.random()*10, Math.random()*8);
      System.out.printf("Writer: Prices have been modified.\n");
      try {
        Thread.sleep(2);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
  }  
```

1.  通过创建一个名为`Main`的类并向其中添加`main()`方法来实现示例的主类。

```java
public class Main {

  public static void main(String[] args) {
```

1.  创建一个`PricesInfo`对象。

```java
    PricesInfo pricesInfo=new PricesInfo();
```

1.  创建五个`Reader`对象和五个`Thread`来执行它们。

```java
    Reader readers[]=new Reader[5];
    Thread threadsReader[]=new Thread[5];

    for (int i=0; i<5; i++){
      readers[i]=new Reader(pricesInfo);
      threadsReader[i]=new Thread(readers[i]);
    }
```

1.  创建一个`Writer`对象和一个`Thread`来执行它。

```java
    Writer writer=new Writer(pricesInfo);
      Thread  threadWriter=new Thread(writer);
```

1.  启动线程。

```java
    for (int i=0; i<5; i++){
      threadsReader[i].start();
    }
    threadWriter.start();
```

## 它是如何工作的...

在下面的截图中，您可以看到此示例的一个执行输出的一部分：

![它是如何工作的...](img/7881_02_05.jpg)

正如我们之前提到的，`ReentrantReadWriteLock`类有两个锁，一个用于读操作，一个用于写操作。在读操作中使用的锁是通过`ReadWriteLock`接口中声明的`readLock()`方法获得的。这个锁是一个实现了`Lock`接口的对象，所以我们可以使用`lock()`、`unlock()`和`tryLock()`方法。在写操作中使用的锁是通过`ReadWriteLock`接口中声明的`writeLock()`方法获得的。这个锁是一个实现了`Lock`接口的对象，所以我们可以使用`lock()`、`unlock()`和`tryLock()`方法。程序员有责任确保正确使用这些锁，使用它们的目的与它们设计的目的相同。当您获得`Lock`接口的读锁时，您不能修改变量的值。否则，您可能会遇到数据不一致的错误。

## 另请参阅

+   在第二章的*Synchronizing a block of code with a Lock*一节中，*基本线程同步*

+   在第八章的*监视锁接口*食谱中，*测试并发应用程序*

# 修改锁的公平性

`ReentrantLock`和`ReentrantReadWriteLock`类的构造函数接受一个名为`fair`的`boolean`参数，允许您控制这两个类的行为。`false`值是默认值，称为**非公平模式**。在此模式下，当有一些线程等待锁（`ReentrantLock`或`ReentrantReadWriteLock`）并且锁必须选择其中一个来访问临界区时，它会选择一个而没有任何标准。`true`值称为**公平模式**。在此模式下，当有一些线程等待锁（`ReentrantLock`或`ReentrantReadWriteLock`）并且锁必须选择一个来访问临界区时，它会选择等待时间最长的线程。请注意，前面解释的行为仅用于`lock()`和`unlock()`方法。由于`tryLock()`方法在使用`Lock`接口时不会使线程进入睡眠状态，因此公平属性不会影响其功能。

在本食谱中，我们将修改在*使用锁同步代码块*食谱中实现的示例，以使用此属性并查看公平和非公平模式之间的区别。

## 做好准备...

我们将修改在*使用锁同步代码块*食谱中实现的示例，因此请阅读该食谱以实现此示例。

## 如何做...

按照以下步骤实现示例：

1.  实现在*使用锁同步代码块*食谱中解释的示例。

1.  在`PrintQueue`类中，修改`Lock`对象的构造。新的指令如下所示：

```java
  private Lock queueLock=new ReentrantLock(true);
```

1.  修改`printJob()`方法。将打印模拟分为两个代码块，在它们之间释放锁。

```java
  public void printJob(Object document){
    queueLock.lock();
    try {
      Long duration=(long)(Math.random()*10000);
      System.out.println(Thread.currentThread().getName()+": PrintQueue: Printing a Job during "+(duration/1000)+" seconds");
      Thread.sleep(duration);
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
       queueLock.unlock();
    }
    queueLock.lock();
    try {
      Long duration=(long)(Math.random()*10000);
      System.out.println(Thread.currentThread().getName()+": PrintQueue: Printing a Job during "+(duration/1000)+" seconds");
      Thread.sleep(duration);
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
          queueLock.unlock();
       } 
  }
```

1.  在`Main`类中修改启动线程的代码块。新的代码块如下所示：

```java
    for (int i=0; i<10; i++){
      thread[i].start();
      try {
        Thread.sleep(100);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
```

## 它是如何工作的...

在下面的屏幕截图中，您可以看到此示例的一次执行输出的一部分：

![它是如何工作的...](img/7881_02_06.jpg)

所有线程的创建间隔为 0.1 秒。请求控制锁的第一个线程是**线程 0**，然后是**线程 1**，依此类推。当**线程 0**运行由锁保护的第一个代码块时，我们有九个线程等待执行该代码块。当**线程 0**释放锁时，立即再次请求锁，因此我们有 10 个线程尝试获取锁。由于启用了公平模式，`Lock`接口将选择**线程 1**，因此它是等待时间最长的线程。然后选择**线程 2**，然后是**线程 3**，依此类推。直到所有线程都通过了由锁保护的第一个代码块，它们才会执行由锁保护的第二个代码块。

一旦所有线程执行了由锁保护的第一个代码块，再次轮到**线程 0**。然后是**线程 1**，依此类推。

要查看与非公平模式的区别，请更改传递给锁构造函数的参数并将其设置为`false`值。在下面的屏幕截图中，您可以看到修改后示例的执行结果：

![它是如何工作的...](img/7881_02_07.jpg)

在这种情况下，线程按照它们被创建的顺序执行，但每个线程都执行两个受保护的代码块。但是，这种行为不能保证，因为如前所述，锁可以选择任何线程来让其访问受保护的代码。在这种情况下，JVM 不能保证线程的执行顺序。

## 还有更多...

读/写锁在其构造函数中也有公平参数。此参数在这种类型的锁中的行为与我们在本食谱介绍中解释的相同。

## 另请参阅

+   在第二章中的*使用锁同步代码块*示例中，*基本线程同步*

+   在第二章中的*使用读/写锁同步数据访问*示例中，*基本线程同步*

+   在第七章中的*实现自定义锁类*示例中，*自定义并发类*

# 在锁中使用多个条件

一个锁可以与一个或多个条件关联。这些条件在`Condition`接口中声明。这些条件的目的是允许线程控制锁，并检查条件是否为`true`，如果为`false`，则暂停，直到另一个线程唤醒它们。`Condition`接口提供了挂起线程和唤醒挂起线程的机制。

并发编程中的一个经典问题是**生产者-消费者**问题。我们有一个数据缓冲区，一个或多个将数据保存在缓冲区中的**生产者**，以及一个或多个从缓冲区中取出数据的**消费者**，正如本章前面所述

在这个示例中，您将学习如何使用锁和条件来实现生产者-消费者问题。

## 准备就绪...

您应该阅读*使用锁同步代码块*示例，以更好地理解这个示例。

## 如何做...

按照以下步骤实现示例：

1.  首先，让我们实现一个类，模拟文本文件。创建一个名为`FileMock`的类，具有两个属性：一个名为`content`的`String`数组和一个名为`index`的`int`。它们将存储文件的内容和将被检索的模拟文件的行。

```java
public class FileMock {

  private String content[];
  private int index;
```

1.  实现类的构造函数，初始化文件内容为随机字符。

```java
  public FileMock(int size, int length){
    content=new String[size];
    for (int i=0; i<size; i++){
      StringBuilder buffer=new StringBuilder(length);
      for (int j=0; j<length; j++){
        int indice=(int)Math.random()*255;
        buffer.append((char)indice);
      }
      content[i]=buffer.toString();
    }
    index=0;
  }
```

1.  实现`hasMoreLines()`方法，如果文件有更多行要处理，则返回`true`，如果已经到达模拟文件的末尾，则返回`false`。

```java
  public boolean hasMoreLines(){
    return index<content.length;
  }
```

1.  实现`getLine()`方法，返回由索引属性确定的行并增加其值。

```java
  public String getLine(){
    if (this.hasMoreLines()) {
      System.out.println("Mock: "+(content.length-index));
      return content[index++];
    } 
    return null;
  }
```

1.  现在，实现一个名为`Buffer`的类，它将实现生产者和消费者共享的缓冲区。

```java
public class Buffer {
```

1.  这个类有六个属性：

+   一个名为`buffer`的`LinkedList<String>`属性，用于存储共享数据

+   定义一个名为`maxSize`的`int`类型，用于存储缓冲区的长度

+   一个名为`lock`的`ReentrantLock`对象，用于控制修改缓冲区的代码块的访问

+   两个名为`lines`和`space`的`Condition`属性

+   一个名为`pendingLines`的`boolean`类型，它将指示缓冲区中是否有行

```java
  private LinkedList<String> buffer;

  private int maxSize;

  private ReentrantLock lock;

  private Condition lines;
  private Condition space;

  private boolean pendingLines;
```

1.  实现类的构造函数。它初始化先前描述的所有属性。

```java
  public Buffer(int maxSize) {
    this.maxSize=maxSize;
    buffer=new LinkedList<>();
    lock=new ReentrantLock();
    lines=lock.newCondition();
    space=lock.newCondition();
    pendingLines=true;
  }
```

1.  实现`insert()`方法。它接收`String`作为参数，并尝试将其存储在缓冲区中。首先，它获取锁的控制权。当它拥有它时，它会检查缓冲区是否有空间。如果缓冲区已满，它会调用`space`条件中的`await()`方法等待空闲空间。当另一个线程调用`space`条件中的`signal()`或`signalAll()`方法时，线程将被唤醒。发生这种情况时，线程将行存储在缓冲区中，并调用`lines`条件上的`signallAll()`方法。正如我们将在下一刻看到的，这个条件将唤醒所有等待缓冲区中行的线程。

```java
  public void insert(String line) {
    lock.lock();
    try {
      while (buffer.size() == maxSize) {
        space.await();
      }
      buffer.offer(line);
      System.out.printf("%s: Inserted Line: %d\n", Thread.currentThread().getName(),buffer.size());
      lines.signalAll();
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      lock.unlock();
    }
  }
```

1.  实现`get()`方法。它返回缓冲区中存储的第一个字符串。首先，它获取锁的控制权。当它拥有它时，它会检查缓冲区中是否有行。如果缓冲区为空，它会调用`lines`条件中的`await()`方法等待缓冲区中的行。当另一个线程调用`lines`条件中的`signal()`或`signalAll()`方法时，该线程将被唤醒。当发生这种情况时，该方法获取缓冲区中的第一行，调用`space`条件上的`signalAll()`方法，并返回`String`。

```java
  public String get() {
    String line=null;
    lock.lock();    
    try {
      while ((buffer.size() == 0) &&(hasPendingLines())) {
        lines.await();
      }

      if (hasPendingLines()) {
        line = buffer.poll();
        System.out.printf("%s: Line Readed: %d\n",Thread.currentThread().getName(),buffer.size());
        space.signalAll();
      }
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      lock.unlock();
    }
    return line;
  }
```

1.  实现`setPendingLines()`方法，建立`pendingLines`属性的值。当生产者没有更多行要生产时，将调用它。

```java
  public void setPendingLines(boolean pendingLines) {
    this.pendingLines=pendingLines;
  }
```

1.  实现`hasPendingLines()`方法。如果有更多行要处理，则返回`true`，否则返回`false`。

```java
  public boolean hasPendingLines() {
    return pendingLines || buffer.size()>0;
  }
```

1.  现在轮到生产者了。实现一个名为`Producer`的类，并指定它实现`Runnable`接口。

```java
public class Producer implements Runnable {
```

1.  声明两个属性：`FileMock`类的一个对象和`Buffer`类的另一个对象。

```java
  private FileMock mock;

  private Buffer buffer;
```

1.  实现初始化两个属性的类的构造函数。

```java
  public Producer (FileMock mock, Buffer buffer){
    this.mock=mock;
    this.buffer=buffer;  
  }
```

1.  实现`run()`方法，读取`FileMock`对象中创建的所有行，并使用`insert()`方法将它们存储在缓冲区中。完成后，使用`setPendingLines()`方法通知缓冲区不会再生成更多行。

```java
   @Override
  public void run() {
    buffer.setPendingLines(true);
    while (mock.hasMoreLines()){
      String line=mock.getLine();
      buffer.insert(line);
    }
    buffer.setPendingLines(false);
  }
```

1.  接下来是消费者的轮次。实现一个名为`Consumer`的类，并指定它实现`Runnable`接口。

```java
public class Consumer implements Runnable {
```

1.  声明一个`Buffer`对象并实现初始化它的类的构造函数。

```java
  private Buffer buffer;

  public Consumer (Buffer buffer) {
    this.buffer=buffer;
  }
```

1.  实现`run()`方法。在缓冲区有待处理的行时，它尝试获取并处理其中的一行。

```java
   @Override  
  public void run() {
    while (buffer.hasPendingLines()) {
      String line=buffer.get();
      processLine(line);
    }
  }
```

1.  实现辅助方法`processLine()`。它只休眠 10 毫秒，模拟对行进行某种处理。

```java
  private void processLine(String line) {
    try {
      Random random=new Random();
      Thread.sleep(random.nextInt(100));
    } catch (InterruptedException e) {
      e.printStackTrace();
    }    
  }
```

1.  通过创建一个名为`Main`的类并向其中添加`main()`方法来实现示例的主类。

```java
public class Main {

  public static void main(String[] args) {
```

1.  创建一个`FileMock`对象。

```java
    FileMock mock=new FileMock(100, 10);
```

1.  创建一个`Buffer`对象。

```java
    Buffer buffer=new Buffer(20);
```

1.  创建一个`Producer`对象和一个`Thread`来运行它。

```java
    Producer producer=new Producer(mock, buffer);
    Thread threadProducer=new Thread(producer,"Producer");
```

1.  创建三个`Consumer`对象和三个线程来运行它。

```java
    Consumer consumers[]=new Consumer[3];
    Thread threadConsumers[]=new Thread[3];

    for (int i=0; i<3; i++){
      consumers[i]=new Consumer(buffer); 
      threadConsumers[i]=new Thread(consumers[i],"Consumer "+i);
    }
```

1.  启动生产者和三个消费者。

```java
    threadProducer.start();
    for (int i=0; i<3; i++){
      threadConsumers[i].start();
    }
```

## 它是如何工作的...

所有的`Condition`对象都与一个锁相关联，并且是使用`Lock`接口中声明的`newCondition()`方法创建的。在我们可以对条件进行任何操作之前，必须控制与条件相关联的锁，因此条件的操作必须在以`Lock`对象的`lock()`方法调用开始的代码块中，并以相同`Lock`对象的`unlock()`方法结束。

当一个线程调用条件的`await()`方法时，它会自动释放锁的控制权，以便另一个线程可以获取它并开始执行相同的临界区或由该锁保护的另一个临界区。

### 注意

当一个线程调用条件的`signal()`或`signallAll()`方法时，等待该条件的一个或所有线程被唤醒，但这并不保证使它们休眠的条件现在是`true`，因此必须将`await()`调用放在`while`循环中。在条件为`true`之前，不能离开该循环。条件为`false`时，必须再次调用`await()`。

在使用`await()`和`signal()`时必须小心。如果在条件中调用`await()`方法，但从未在该条件中调用`signal()`方法，线程将永远休眠。

在休眠时，线程可能会被中断，在调用`await()`方法后，因此必须处理`InterruptedException`异常。

## 还有更多...

`Condition`接口有`await()`方法的其他版本，如下所示：

+   `await(long time, TimeUnit unit)`: 线程将休眠直到：

+   它被中断了

+   另一个线程在条件中调用`signal()`或`signalAll()`方法

+   指定的时间已经过去

+   `TimeUnit`类是一个枚举，具有以下常量：`DAYS`、`HOURS`、`MICROSECONDS`、`MILLISECONDS`、`MINUTES`、`NANOSECONDS`和`SECONDS`

+   `awaitUninterruptibly()`: 线程将休眠直到另一个线程调用`signal()`或`signalAll()`方法，这是不可中断的

+   `awaitUntil(Date date)`: 线程将休眠直到：

+   它被中断了

+   另一个线程在条件中调用`signal()`或`signalAll()`方法

+   指定的日期到达

您可以使用条件与读/写锁的`ReadLock`和`WriteLock`锁。

## 另请参阅

+   在《第二章》（ch02.html“第二章基本线程同步”）的*使用锁同步代码块*配方中

+   在《第二章》（ch02.html“第二章基本线程同步”）的*使用读/写锁同步数据访问*配方
