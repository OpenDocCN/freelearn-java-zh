# 并发编程设计

在本章中，我们将涵盖以下主题：

+   尽可能使用不可变对象

+   通过排序锁来避免死锁

+   使用原子变量而不是同步

+   尽可能短时间持有锁

+   将线程的管理委托给执行器

+   使用并发数据结构而不是自己编程

+   使用懒初始化时采取预防措施

+   使用 fork/join 框架而不是执行器

+   避免在锁内使用阻塞操作

+   避免使用已弃用方法

+   使用执行器而不是线程组

+   使用流来处理大数据集

+   其他技巧和窍门

# 简介

实现一个并发应用程序是一项困难的任务。在执行过程中，你同时有多个线程，并且它们共享资源，如文件、内存、对象等。你必须对你的设计决策非常小心。一个糟糕的决定可能会以影响你的程序的方式导致性能下降或简单地引发数据不一致的情况。

在本章中，我提供了一些建议，以帮助您做出正确的设计决策，这将使您的并发应用程序变得更好。

# 尽可能使用不可变对象

当你使用面向对象的编程在 Java 中开发应用程序时，你创建了一些由属性和方法组成的类。类的方法决定了你可以对该类执行的操作。属性存储定义对象的 数据。通常，在每一个类中，你实现一些方法来设置属性值。此外，对象在应用程序运行时也会发生变化，你使用这些方法来改变它们的属性值。

当你开发一个并发应用程序时，你必须特别注意多个线程共享的对象。你必须使用同步机制来保护对这些对象的访问。如果你不使用它，你可能在应用程序中遇到数据不一致的问题。

当你与并发应用程序一起工作时，你可以实现一些特殊类型的对象。它们被称为**不可变对象**；它们的主要特征是它们在创建后不能被修改。如果你需要更改不可变对象，你必须创建一个新的对象，而不是更改对象的属性值。

当你在并发应用程序中使用此机制时，它具有以下优点：

+   这些对象一旦创建，就不能被任何线程修改，因此你不需要使用任何同步机制来保护对其属性的访问。

+   你不会遇到任何数据不一致的问题。由于这些对象的属性不能被修改，你将始终能够访问数据的一致副本。

这种方法的唯一缺点是开销：创建新对象而不是修改现有对象。

Java 提供了一些不可变类，例如`String`类。当您有一个`String`对象，并尝试为其分配新值时，您实际上是在创建一个新的`String`对象，而不是修改对象的旧值。例如，查看以下代码：

```java
    String var = "hello"; 
    var = "new";

```

在第二行，JVM 创建了一个新的`String`对象。

# 准备工作

此示例的配方已使用 Eclipse IDE 实现。如果您使用 Eclipse 或不同的 IDE，例如 NetBeans，请打开它并创建一个新的 Java 项目。

# 如何做到这一点...

按照以下步骤实现不可变类：

1.  将类标记为`final`。它不应该被另一个类扩展。

1.  所有属性都必须是`final`和`private`。您只能给属性赋值一次。

1.  不要提供可以分配属性值的函数。属性必须在类的构造函数中初始化。

1.  如果任何字段值对象是可变的（例如，`java.util.Date`），在 getter 字段中始终返回防御性副本。

1.  不要从不可变类构造函数中泄露`this`引用（例如，在构造函数完成之前泄露`this`引用的以下代码）：

```java
        public final NotSoImmutable implements Listener { 
          private final int x; 
          public NotSoImmutable(int x, Observable o) { 
            this.x = x; 
            o.registerListener(this); 
          } 
        }

```

# 它是如何工作的...

如果您想实现一个存储人的姓名和姓氏的类，您通常会实现如下：

```java
    public class PersonMutable { 
      private String firstName; 
      private String lastName; 
      private Date birthDate; 

      public String getFirstName() { 
        return firstName; 
      } 

      public void setFirstName(String firstName) { 
        this.firstName = firstName; 
      } 

      public String getLastName() { 
        return lastName; 
      } 

      public void setLastName(String lastName) { 
        this.lastName = lastName; 
      } 
      public Date getBirthDate() { 
        return birthDate; 
      } 

      public void setBirthDate(Date birthDate) { 
        this.birthDate = birthDate; 
      } 

    }

```

您可以通过遵循前面解释的规则将此类转换为不可变类。以下就是结果：

```java
    public final class PersonImmutable { 

      final private String firstName; 
      final private String lastName; 
      final private Date birthDate; 

      public PersonImmutable (String firstName, String lastName,
                              String address, Date birthDate) { 
        this.firstName=firstName; 
        this.lastName=lastName; 
        this.birthDate=birthDate; 
      } 

      public String getFirstName() { 
        return firstName; 
      } 

      public String getLastName() { 
        return lastName; 
      }

```

```java
      public Date getBirthDate() { 
        return new Date(birthDate.getTime()); 
      } 

    }

```

实际上，您遵循了不可变类的基本原则，如下所示：

+   类被标记为`final`。

+   属性被标记为`final`和`private`。

+   属性的值只能在类的构造函数中建立。

    它们的方法返回属性的值，但不会修改它们。

+   对于可变属性（在我们的例子中是`birthDate`属性），我们通过创建一个新对象来返回`get()`方法的防御性副本。

# 更多内容...

不可变对象并不总是可以使用。分析您应用程序的每个类，以确定您是否可以将它们实现为不可变对象。如果您无法将一个类实现为不可变类，并且其对象被多个线程共享，您必须使用同步机制来保护对类属性的访问。

# 参见

+   本章中关于“使用原子变量代替同步”的配方

# 通过排序锁来避免死锁

当您需要在应用程序的方法中获取多个锁时，您必须非常小心地控制锁的获取顺序。错误的选择可能导致死锁情况。

在这个配方中，您将实现一个死锁情况的示例，然后学习如何解决它。

# 如何做到这一点...

按照以下步骤实现示例：

1.  创建一个名为`BadLocks`的类，其中包含两个方法，分别命名为`operation1()`和`operation2()`：

```java
        public class BadLocks { 

          private Lock lock1, lock2; 

          public BadLocks(Lock lock1, Lock lock2) { 
            this.lock1=lock1; 
            this.lock2=lock2; 
          } 

          public void operation1(){ 
            lock1.lock(); 
            lock2.lock(); 

            try { 
              TimeUnit.SECONDS.sleep(2); 
            } catch (InterruptedException e) { 
              e.printStackTrace(); 
            } finally { 
              lock2.unlock(); 
              lock1.unlock(); 
            } 
          } 

          public void operation2(){ 
            lock2.lock(); 
            lock1.lock(); 

            try { 
              TimeUnit.SECONDS.sleep(2); 
            } catch (InterruptedException e) { 
              e.printStackTrace(); 
            } finally { 
              lock1.unlock(); 
              lock2.unlock(); 
            } 
          } 

        }

```

1.  让我们分析前面的代码。如果一个线程调用 `operation1()` 方法，而另一个线程调用 `operation2()` 方法，你可能会遇到死锁。如果 `operation1()` 和 `operation2()` 同时执行它们各自的第一句话，那么 `operation1()` 方法将等待获取 `lock2` 的控制权，而 `operation2()` 方法将等待获取 `lock1` 的控制权。现在你有一个死锁的情况。

1.  要解决这个问题，你可以遵循以下规则：

+   如果你必须在不同的操作中控制多个锁，尝试在所有方法中以相同的顺序锁定它们。

+   然后，以相反的顺序释放它们，并将锁及其解锁封装在单个类中。这样，你就不需要在代码中分散同步相关的代码。

# 它是如何工作的...

使用这个规则，你将避免死锁情况。例如，在前面提到的案例中，你可以将 `operation2()` 改为首先获取 `lock1`，然后获取 `lock2`。现在如果 `operation1()` 和 `operation2()` 同时执行它们各自的第一句话，其中一个将被阻塞等待 `lock1`，而另一个将获取 `lock1` 和 `lock2` 并执行它们的操作。之后，被阻塞的线程将获取 `lock1` 和 `lock2` 锁，并执行其操作。

# 还有更多...

你可能会遇到一种情况，其中某个要求阻止你在所有操作中以相同的顺序获取锁。在这种情况下，你可以使用 `Lock` 类的 `tryLock()` 方法。此方法返回一个 `Boolean` 值，以指示你是否控制了锁。你可以尝试使用 `tryLock()` 方法获取你需要的所有锁来完成操作。如果你无法控制其中一个锁，你必须释放你可能拥有的所有锁，并重新开始操作。

# 参见

+   本章中 *尽可能短时间持有锁* 的配方

# 使用原子变量而不是同步

当你需要在多个线程之间共享数据时，你必须使用同步机制来保护对这块数据的访问。你可以在修改数据的方法的声明中使用 `synchronized` 关键字，以确保一次只有一个线程可以修改数据。另一种可能性是使用 `Lock` 类来创建一个临界区，其中包含修改数据的指令。

自版本 5 以来，Java 包含原子变量。当一个线程使用原子变量执行操作时，类的实现包括一个机制来检查操作是否一步完成。基本上，操作获取变量的值，在一个局部变量中更改值，然后尝试用新值替换旧值。如果旧值仍然是相同的，它就会进行更改。如果不是，方法将重新开始操作。Java 提供以下类型的原子变量：

+   `AtomicBoolean`

+   `AtomicInteger`

+   `AtomicLong`

+   `AtomicReference`

在某些情况下，Java 的原子变量比基于同步机制的解决方案（尤其是当我们关心每个单独变量的原子性时）提供更好的性能。`java.util.concurrent` 包中的某些类使用原子变量而不是同步。在这个菜谱中，你将开发一个示例，展示原子属性如何比同步提供更好的性能。

# 准备工作

这个菜谱的示例已经使用 Eclipse IDE 实现。如果你使用 Eclipse 或其他 IDE，如 NetBeans，打开它并创建一个新的 Java 项目。

# 如何操作...

按照以下步骤实现示例：

1.  创建一个名为 `TaskAtomic` 的类并指定它实现 `Runnable` 接口：

```java
        public class TaskAtomic implements Runnable {

```

1.  声明一个名为 `number` 的私有 `AtomicInteger` 属性：

```java
        private final AtomicInteger number;

```

1.  实现类的构造函数以初始化其属性：

```java
        public TaskAtomic () { 
          this.number=new AtomicInteger(); 
        }

```

1.  实现 `run()` 方法。在一个包含 1,000,000 步的循环中，使用 `set()` 方法将步数作为值分配给原子属性：

```java
        @Override 
        public void run() { 
          for (int i=0; i<1000000; i++) { 
            number.set(i); 
          } 
        }

```

1.  创建一个名为 `TaskLock` 的类并指定它实现 `Runnable` 接口：

```java
        public class TaskLock implements Runnable {

```

1.  声明一个名为 `number` 的私有 `int` 属性和一个名为 `lock` 的私有 `Lock` 属性：

```java
        private Lock lock; 
        private int number;

```

1.  实现类的构造函数以初始化其属性：

```java
        public TaskLock() { 
          this.lock=new ReentrantLock(); 
        }

```

1.  实现 `run()` 方法。在一个包含 1,000,000 步的循环中，将步数分配给整数属性。你必须在分配之前获取锁，并在分配之后释放它：

```java
        @Override 
        public void run() { 
          for (int i=0; i<1000000; i++) { 
            lock.lock(); 
            number=i; 
            lock.unlock(); 
          } 

        }

```

1.  通过创建一个名为 `Main` 的类并添加 `main()` 方法来实现示例的主类：

```java
        public class Main { 
          public static void main(String[] args) {

```

1.  创建一个名为 `atomicTask` 的 `TaskAtomic` 对象：

```java
        TaskAtomic atomicTask=new TaskAtomic();

```

1.  创建一个名为 `lockTask` 的 `TaskLock` 对象：

```java
        TaskLock lockTask=new TaskLock();

```

1.  声明线程数并创建一个 `Thread` 对象数组来存储线程：

```java
        int numberThreads=50; 
        Thread threads[]=new Thread[numberThreads]; 
        Date begin, end;

```

1.  启动指定数量的线程来执行 `TaskLock` 对象。计算并写入其执行时间到控制台：

```java
        begin=new Date(); 
        for (int i=0; i<numberThreads; i++) { 
          threads[i]=new Thread(lockTask); 
          threads[i].start(); 
        }

```

```java
        for (int i=0; i<numberThreads; i++) { 
          try { 
            threads[i].join(); 
          } catch (InterruptedException e) { 
            e.printStackTrace(); 
          } 
        } 
        end=new Date(); 

        System.out.printf("Main: Lock results: %d\n",
                          (end.getTime()-begin.getTime()));

```

1.  启动指定数量的线程来执行 `TaskAtomic` 对象。计算并写入其执行时间到控制台：

```java
        begin=new Date(); 
        for (int i=0; i<numberThreads; i++) { 
          threads[i]=new Thread(atomicTask); 
          threads[i].start(); 
        } 

        for (int i=0; i<numberThreads; i++) { 
          try { 
            threads[i].join(); 
          } catch (InterruptedException e) { 
            e.printStackTrace(); 
          } 
        } 
        end=new Date(); 

        System.out.printf("Main: Atomic results: %d\n",
                          (end.getTime()-begin.getTime()));

```

# 它是如何工作的...

当你执行示例时，你会看到使用原子变量的 `TaskAtomic` 任务的执行时间总是比使用锁的 `TaskLock` 任务的执行时间更好。如果你使用 `synchronized` 关键字而不是锁，你将获得类似的结果。

这个菜谱的结论是，利用原子变量将比其他同步方法提供更好的性能。如果你没有适合你需求的原子类型，也许你可以尝试实现你自己的原子类型。

# 参见

+   在第八章 实现自己的原子对象 的 *自定义并发类* 菜谱中

# 尽可能短时间持有锁

锁，就像其他同步机制一样，允许定义一个临界区，一次只有一个线程可以执行。您必须非常小心地定义临界区。它必须只包括真正需要互斥的指令。如果临界区包括长时间操作，这一点尤为重要。如果临界区包括不使用共享资源的长时间操作，应用程序的性能将比可能的情况更差。

在本菜谱中，您将实现一个示例，以查看临界区内部有长时间操作的任务与临界区外部有长时间操作的任务之间的性能差异。

# 准备工作

该菜谱的示例已使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE，例如 NetBeans，请打开它并创建一个新的 Java 项目。

# 如何做到这一点...

按照以下步骤实现示例：

1.  创建一个名为 `Operations` 的类：

```java
        public class Operations {

```

1.  实现一个名为 `readData()` 的 `public static` 方法。它使当前线程休眠 500 毫秒：

```java
        public static void readData(){ 
          try { 
            TimeUnit.MILLISECONDS.sleep(500); 
          } catch (InterruptedException e) { 
            e.printStackTrace(); 
          } 
        }

```

1.  实现一个名为 `writeData()` 的 `public static` 方法。它使当前线程休眠 500 毫秒：

```java
        public static void writeData(){ 
          try { 
            TimeUnit.MILLISECONDS.sleep(500); 
          } catch (InterruptedException e) { 
            e.printStackTrace(); 
          } 
        }

```

1.  实现一个名为 `processData()` 的 `public static` 方法。它使当前线程休眠 2,000 毫秒：

```java
        public static void processData(){ 
          try { 
            TimeUnit.SECONDS.sleep(2); 
          } catch (InterruptedException e) { 
            e.printStackTrace(); 
          } 
        }

```

1.  实现一个名为 `Task1` 的类，并指定它实现 `Runnable` 接口：

```java
        public class Task1 implements Runnable {

```

1.  声明一个名为 `lock` 的私有 `Lock` 属性：

```java
        private final Lock lock;

```

1.  实现类的构造函数以初始化其属性：

```java
        public Task1 (Lock lock) { 
          this.lock=lock; 
        }

```

1.  实现 `run()` 方法。获取锁，调用 `Operations` 类的三个操作，并释放锁：

```java
        @Override 
        public void run() { 
          lock.lock(); 
          Operations.readData(); 
          Operations.processData(); 
          Operations.writeData(); 
          lock.unlock(); 
        }

```

1.  实现一个名为 `Task2` 的类，并指定它实现 `Runnable` 接口：

```java
        public class Task2 implements Runnable {

```

1.  声明一个名为 `lock` 的私有 `Lock` 属性：

```java
        private final Lock lock;

```

1.  实现类的构造函数以初始化其属性：

```java
        public Task2 (Lock lock) { 
          this.lock=lock; 
        }

```

1.  实现 `run()` 方法。获取锁，调用 `readData()` 操作，并释放锁。然后，调用 `processData()` 方法，获取锁，调用 `writeData()` 操作，并释放锁：

```java
        @Override 
        public void run() { 
          lock.lock(); 
          Operations.readData(); 
          lock.unlock(); 
          Operations.processData(); 
          lock.lock(); 
          Operations.writeData(); 
          lock.unlock(); 
        }

```

1.  通过创建一个名为 `Main` 的类并添加 `main()` 方法来实现示例的主类：

```java
        public class Main { 

          public static void main(String[] args) {

```

1.  创建一个名为 `lock` 的 `Lock` 对象，一个名为 `task1` 的 `Task1` 对象，一个名为 `task2` 的 `Task2` 对象，以及一个包含 10 个线程的数组：

```java
        Lock lock=new ReentrantLock(); 
        Task1 task1=new Task1(lock); 
        Task2 task2=new Task2(lock); 
        Thread threads[]=new Thread[10];

```

1.  通过控制执行时间来启动 10 个线程以执行第一个任务：

```java
        Date begin, end; 

        begin=new Date(); 
        for (int i=0; i<threads.length; i++) { 
          threads[i]=new Thread(task1); 
          threads[i].start(); 
        } 

        for (int i=0; i<threads.length; i++) { 
          try { 
            threads[i].join(); 
          } catch (InterruptedException e) { 
            e.printStackTrace(); 
          } 
        } 
        end=new Date(); 
        System.out.printf("Main: First Approach: %d\n",
                          (end.getTime()-begin.getTime()));

```

1.  通过控制执行时间来启动 10 个线程以执行第二个任务：

```java
        begin=new Date(); 
        for (int i=0; i<threads.length; i++) { 
          threads[i]=new Thread(task2); 
          threads[i].start(); 
        } 

        for (int i=0; i<threads.length; i++) { 
          try { 
            threads[i].join(); 
          } catch (InterruptedException e) { 
            e.printStackTrace(); 
          } 
        } 
        end=new Date(); 
        System.out.printf("Main: Second Approach: %d\n",
                          (end.getTime()-begin.getTime()));

```

# 它是如何工作的...

如果您执行示例，您将看到两种方法执行时间之间的巨大差异。所有操作都在临界区内的任务比其他任务花费的时间更长。

当您需要实现一个受锁保护的代码块时，仔细分析它，只包括必要的指令。将方法拆分为多个关键部分，并在必要时使用多个锁以获得最佳的应用程序性能。

# 参见

+   本章中关于*通过排序锁避免死锁*的配方

# 将线程管理委托给执行器

在 Java 5 之前，当我们想要实现一个并发应用程序时，我们必须自己管理线程。最初，我们通常实现`Runnable`接口或`Thread`类的扩展。然后，我们创建一个`thread`对象，并使用其`start()`方法启动其执行。我们还需要控制其状态，以了解线程是否已经完成执行或仍在运行。

在 Java 5 版本中，执行器作为提供执行线程池的提供者的概念出现。这种机制由`Executor`和`ExecutorService`接口以及`ThreadPoolExecutor`和`ScheduledThreadPoolExecutor`类实现，它允许您只关注任务的逻辑实现。您实现任务并将其发送到执行器。它有一个线程池，并且是这个池负责线程的创建、管理和最终化。在 Java 7 版本中，在 fork/join 框架中出现了执行器机制的另一种实现，专门用于可以分解为更小子问题的任务。这种方法具有许多优势，如下所述：

+   我们不需要为所有任务创建线程。当我们向执行器发送任务，并由池中的线程执行时，我们节省了创建新线程所需的时间。如果我们的应用程序必须执行大量任务，那么节省的总时间将会非常显著，并且应用程序的性能将会更好。

+   如果我们创建的线程较少，我们的应用程序也将使用更少的内存。这也可以从我们的应用程序中提取更好的性能。

+   我们可以通过实现`Runnable`或`Callable`接口来构建由执行器执行的任务。`Callable`接口允许我们实现返回结果的任务，这比传统任务提供了很大的优势。

+   当我们向执行器发送任务时，它返回一个`Future`对象，允许我们轻松了解任务的状态和返回的结果，无论它是否已经完成执行。

+   我们可以使用`ScheduledThreadPoolExecutor`类实现的特殊执行器来安排我们的任务并重复执行它们。

+   我们可以轻松控制执行器使用的资源。我们可以设置池中线程的最大数量，这样我们的执行器就不会同时运行超过这个数量的任务。

与直接使用线程相比，使用执行器有很多优势。在本配方中，您将实现一个示例，展示如何使用执行器而不是自己创建线程来获得更好的性能。

# 准备工作

本配方示例使用 Eclipse IDE 实现。如果您使用 Eclipse 或 NetBeans 等其他 IDE，请打开它并创建一个新的 Java 项目。

# 如何操作...

按照以下步骤实现示例：

1.  创建一个名为 `Task` 的类，并指定它实现 `Runnable` 接口：

```java
        public class Task implements Runnable {

```

1.  实现运行方法。创建一个包含 1,000,000 步的循环，并在每一步中，对一个整型变量进行一些数学运算：

```java
        @Override 
        public void run() { 
          int r; 
          for (int i=0; i<1000000; i++) { 
            r=0; 
            r++; 
            r++; 
            r*=r; 
          } 
        }

```

1.  通过创建一个名为 `Main` 的类并添加 `main()` 方法来实现示例中的主类：

```java
        public class Main { 

          public static void main(String[] args) {

```

1.  创建 1,000 个线程来执行 1,000 个任务对象并等待它们完成，控制总执行时间：

```java
        Thread threads[]=new Thread[1000]; 
        Date start,end; 

        start=new Date(); 
        for (int i=0; i<threads.length; i++) { 
          Task task=new Task(); 
          threads[i]=new Thread(task); 
          threads[i].start(); 
        } 

        for (int i=0; i<threads.length; i++) { 
          try { 
            threads[i].join(); 
          } catch (InterruptedException e) { 
            e.printStackTrace(); 
          } 
        } 
        end=new Date(); 
        System.out.printf("Main: Threads: %d\n",
                          (end.getTime()-start.getTime()));

```

1.  创建一个 `Executor` 对象，发送 1,000 个 `Task` 对象给它，并等待它们完成。测量总执行时间：

```java
        ThreadPoolExecutor executor=(ThreadPoolExecutor)Executors
                                                .newCachedThreadPool(); 

        start=new Date();

```

```java
        for (int i=0; i<threads.length; i++) { 
          Task task=new Task(); 
          executor.execute(task); 
        } 
        executor.shutdown(); 
        try { 
          executor.awaitTermination(1, TimeUnit.DAYS); 
        } catch (InterruptedException e) { 
          e.printStackTrace(); 
        } 
        end=new Date(); 
        System.out.printf("Main: Executor: %d\n",
                          (end.getTime()-start.getTime()));

```

# 它是如何工作的...

在整个示例执行过程中，我们总是获得了比直接创建线程更小的执行时间。如果你的应用程序必须执行大量任务，最好使用执行器。

# 参见

+   本章中的 *使用执行器而不是线程组* 和 *使用 fork/join 框架而不是执行器* 的食谱

# 使用并发数据结构而不是自己编程

数据结构是每个程序的基本组成部分。你必须始终管理存储在数据结构中的数据。数组、列表或树是常见的数据结构示例。Java API 提供了许多现成可用的数据结构，但在处理并发应用程序时，你必须小心，因为 Java API 提供的所有结构并不都是 **线程安全** 的。如果你选择了一个非线程安全的数据结构，你的应用程序中可能会有不一致的数据。

当你想要在你的并发应用程序中使用一种数据结构时，你必须审查实现该数据结构的类的文档，以检查它是否支持并发操作。Java 提供以下两种类型的并发数据结构：

+   **非阻塞数据结构**：这些数据结构提供的所有操作，无论是向数据结构中插入元素还是从中移除元素，如果当前无法执行（因为数据结构已满或为空），都会返回 null 值。

+   **阻塞数据结构**：这些数据结构提供了与非阻塞数据结构相同的操作。然而，它们还提供了插入和移除数据操作，如果未立即执行，将阻塞线程，直到你能够执行操作。

这些是 Java API 提供的一些数据结构，你可以在你的并发应用程序中使用：

+   `ConcurrentLinkedDeque`：这是一个基于链节点的非阻塞数据结构，允许你在结构的开始或末尾插入数据。

+   `LinkedBlockingDeque`：这是一个基于链节点的阻塞数据结构。它可以具有固定容量。你可以在结构的开始或末尾插入元素。它提供了操作，如果未立即执行，将阻塞线程，直到你能够执行操作。

+   `ConcurrentLinkedQueue`：这是一个非阻塞队列，允许您在队列末尾插入元素并从其开头取出元素。

+   `ArrayBlockingQueue`：这是一个固定大小的阻塞队列。您可以在队列的末尾插入元素，并从其开头取出元素。它提供了操作，如果因为队列已满或为空而没有执行，则将线程休眠，直到您能够执行操作。

+   `LinkedBlockingQueue`：这是一个允许您在队列末尾插入元素并从其开头取出元素的阻塞队列。它提供了操作，如果因为队列已满或为空而没有执行，则将线程休眠，直到您能够执行操作。

+   `DelayQueue`：这是一个带有延迟元素的`LinkedBlockingQueue`队列。每个插入到该队列的元素都必须实现`Delayed`接口。一个元素不能从列表中移除，直到其延迟时间为 0。

+   `LinkedTransferQueue`：这是一个阻塞队列，提供在可以表示为生产者/消费者问题的场景中工作的操作。它提供了操作，如果因为队列已满或为空而没有执行，则将线程休眠，直到您能够执行操作。

+   `PriorityBlockingQueue`：这是一个基于优先级对元素进行排序的阻塞队列。所有插入到该队列的元素都必须实现`Comparable`接口。`compareTo()`方法返回的值将确定元素在队列中的位置。就像所有阻塞数据结构一样，它提供了操作，如果立即执行，则将线程休眠，直到您能够执行操作。

+   `SynchronousQueue`：这是一个阻塞队列，其中每个`insert`操作都必须等待另一个线程的`remove`操作。这两个操作必须同时进行。

+   `ConcurrentHashMap`：这是一个允许并发操作的自定义`HashMap`。它是一个非阻塞的数据结构。

+   `ConcurrentSkipListMap`：此数据结构将键与值关联。每个键只能有一个值。它以有序方式存储键并提供方法来查找元素和从映射中获取一些元素。它是一个非阻塞数据结构。

# 还有更多...

如果您需要在您的并发应用程序中使用数据结构，请查看 Java API 文档以找到最适合您需求的数据结构。实现您自己的并发数据结构，它存在以下问题：

+   它们具有复杂的内部结构

+   您必须考虑许多不同的情况

+   您必须设计大量的测试来确保其正确性

如果您找不到完全符合您需求的数据结构，请尝试扩展现有的并发数据结构之一，以适当地实现您的问题。

# 参见

+   第七章中的食谱，*并发集合*

# 使用懒加载时的注意事项

**懒加载**是一种常见的编程技术，它将对象创建推迟到第一次需要时。这通常会导致对象初始化是在操作的实现中而不是在类的构造函数中进行的。这种技术的优点是您可以节省内存。这是因为您只为应用程序的执行创建必需的对象。您可以在一个类中声明很多对象，但在程序的每次执行中，您并不使用每个对象；因此，您的应用程序不会使用在程序执行中不使用的对象的内存。这种优势对于在资源有限的环境中运行的应用程序非常有用。

相比之下，这种技术在第一次在操作中使用对象时创建对象，可能会在应用程序中引起性能问题。

如果您在并发应用程序中使用此技术，它也可能引发问题。由于一次可以有多个线程执行操作，它们可以在同一时间创建对象，这种情况可能会出现问题。这对于**单例**类尤为重要。应用程序只有一个这些类的对象，如前所述，并发应用程序可以创建多个对象。考虑以下代码：

```java
    public static DBConnection getConnection(){ 
      if (connection==null) { 
        connection=new DBConnection(); 
      } 
      return connection; 
    }

```

这是单例类中获取该类在应用程序中存在的唯一对象引用的典型方法，使用懒加载初始化。如果对象尚未创建，则创建该对象。最后，它总是返回它。

如果两个或多个线程同时执行第一句话的比较（`connection == null`），它们都会创建一个 `Connection` 对象。这不是一个理想的情况。

在这个菜谱中，你将实现一个优雅的懒初始化问题的解决方案。

# 准备工作

本菜谱的示例已使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE，例如 NetBeans，请打开它并创建一个新的 Java 项目。

# 如何做到这一点...

按照以下步骤实现示例：

1.  创建一个名为 `DBConnectionOK` 的类：

```java
        public class DBConnectionOK {

```

1.  声明一个 `private` 构造函数。写上执行它的线程名称：

```java
        private DBConnectionOK() { 
          System.out.printf("%s: Connection created.\n",
                            Thread.currentThread().getName()); 
        }

```

1.  声明一个名为 `LazyDBConnectionOK` 的 `private static` 类。它有一个名为 `INSTANCE` 的 `private static final DBConnectionOK` 实例：

```java
        private static class LazyDBConnection { 
          private static final DBConnectionOK INSTANCE = new
                                                   DBConnectionOK(); 
        }

```

1.  实现 `getConnection()` 方法。它不接受任何参数，并返回一个 `DBConnectionOK` 对象。它返回 `INSTANCE` 对象：

```java
        public static DBConnectionOK getConnection() { 
          return LazyDBConnection.INSTANCE; 
        }

```

1.  创建一个名为 `Task` 的类，并指定它实现 `Runnable` 接口。实现 `run()` 方法。调用 `DBConnectionOK()` 方法的 `getConnection()` 方法：

```java
        public class Task implements Runnable { 

          @Override 
          public void run() { 

            System.out.printf("%s: Getting the connection...\n",
                              Thread.currentThread().getName()); 
            DBConnectionOK connection=DBConnectionOK.getConnection(); 
            System.out.printf("%s: End\n",
                              Thread.currentThread().getName()); 
          } 

        }

```

1.  通过创建一个名为 `Main` 的类并添加 `main()` 方法来实现示例的主类：

```java
        public class Main { 
          public static void main(String[] args) {

```

1.  创建 20 个 `Task` 对象和 20 个线程来执行它们：

```java
            for (int i=0; i<20; i++){ 
              Task task=new Task(); 
              Thread thread=new Thread(task); 
              thread.start(); 
            } 
          }

```

# 它是如何工作的...

示例的关键是 `getConnection()` 方法以及 `private static class LazyDBConnection` 实例。当第一个线程调用 `getConnection()` 方法时，`LazyDBConnection` 类通过调用 `DBConnection` 类的构造函数来初始化 `INSTANCE` 对象。这个对象由 `getConnection()` 方法返回。当其他线程调用 `getConnection()` 方法时，对象已经创建，所以所有线程都使用只创建一次的同一个对象。

当你运行示例时，你会看到 20 个任务的开始和结束消息，但只有一个创建消息。

# 使用 fork/join 框架而不是执行器

Executors 允许你避免创建和管理线程。你通过实现 `Runnable` 或 `Callable` 接口来执行任务，并将它们发送到执行器。它有一个线程池，并使用其中的一个来执行任务。

Java 7 提供了一种新的 executor，即 fork/join 框架。这个在 `ForkJoinPool` 类中实现的 executor，旨在解决可以使用划分和征服技术划分为更小部分的问题。当你为 fork/join 框架实现任务时，你必须检查你要解决的问题的大小。如果它大于预定义的大小，你将问题划分为两个或更多子类别，并创建与划分次数相等的子任务。任务使用 `fork()` 操作将这些子任务发送到 `ForkJoinPool` 类，并使用 `join()` 操作等待它们的最终化。

对于这类问题，fork/join 池比经典执行器有更好的性能。在这个菜谱中，你将实现一个示例，以检查这个点。

# 准备工作

这个菜谱的示例是使用 Eclipse IDE 实现的。如果你使用 Eclipse 或其他 IDE，例如 NetBeans，打开它并创建一个新的 Java 项目。

# 如何做...

按照以下步骤实现示例：

1.  创建一个名为 `TaskFJ` 的类，并指定它扩展 `RecursiveAction` 类：

```java
        public class TaskFJ extends RecursiveAction {

```

1.  声明一个名为 `array` 的私有 `int` 数组：

```java
        private final int array[];

```

1.  声明两个名为 `start` 和 `end` 的私有 `int` 属性：

```java
        private final int start, end;

```

1.  实现类的构造函数以初始化其属性：

```java
        public TaskFJ(int array[], int start, int end) { 
          this.array=array; 
          this.start=start; 
          this.end=end; 
        }

```

1.  实现 `compute()` 方法。如果这个任务需要处理超过 1,000 个元素的块（由 `start` 和 `end` 属性确定），创建两个 `TaskFJ` 对象，使用 `fork()` 方法将它们发送到 `ForkJoinPool` 类，并使用 `join()` 方法等待它们的最终化：

```java
        @Override 
        protected void compute() { 
          if (end-start>1000) { 
            int mid=(start+end)/2; 
            TaskFJ task1=new TaskFJ(array,start,mid); 
            TaskFJ task2=new TaskFJ(array,mid,end); 
            task1.fork(); 
            task2.fork(); 
            task1.join(); 
            task2.join();

```

1.  否则，增加这个任务需要处理的元素。在每次增加操作后，让线程休眠 1 毫秒：

```java
        } else { 
          for (int i=start; i<end; i++) { 
            array[i]++; 
            try { 
              TimeUnit.MILLISECONDS.sleep(1); 
            } catch (InterruptedException e) { 
              e.printStackTrace(); 
            } 
          } 
        }

```

1.  创建一个名为 `Task` 的类，并指定它实现 `Runnable` 接口：

```java
        public class Task implements Runnable {

```

1.  声明一个名为 `array` 的私有 `int` 数组：

```java
        private final int array[];

```

1.  实现类的构造函数以初始化其属性：

```java
        public Task(int array[]) { 
          this.array=array; 
        }

```

1.  实现一个`run()`方法。增加数组中所有元素。在每次增加操作后，让线程休眠 1 毫秒：

```java
        @Override 
        public void run() { 
          for (int i=0; i<array.length; i++ ){ 
            array[i]++; 
            try { 
              TimeUnit.MILLISECONDS.sleep(1); 
            } catch (InterruptedException e) { 
              e.printStackTrace(); 
            } 
          } 
        }

```

1.  通过创建一个名为`Main`的类并添加`main()`方法来实现示例的主类：

```java
        public class Main { 

          public static void main(String[] args) {

```

1.  创建一个包含 100,000 个元素的`int`数组：

```java
        int array[]=new int[100000];

```

1.  创建一个`Task`对象和一个`ThreadPoolExecutor`对象并执行它们。通过控制任务运行的时间来执行任务：

```java
        Task task=new Task(array); 
        ExecutorService executor=Executors.newCachedThreadPool(); 

        Date start,end; 
        start=new Date(); 
        executor.execute(task); 
        executor.shutdown(); 
        try { 
          executor.awaitTermination(1, TimeUnit.DAYS); 
        } catch (InterruptedException e) { 
          e.printStackTrace(); 
        } 
        end=new Date(); 
        System.out.printf("Main: Executor: %d\n",
                          (end.getTime()-start.getTime()));

```

1.  创建一个`TaskFJ`对象和一个`ForkJoinPool`对象并执行它们。通过控制任务运行的时间来执行任务：

```java
          TaskFJ taskFJ=new TaskFJ(array,1,100000); 
          ForkJoinPool pool=new ForkJoinPool(); 
          start=new Date(); 
          pool.execute(taskFJ); 
          pool.shutdown(); 
          try { 
            pool.awaitTermination(1, TimeUnit.DAYS); 
          } catch (InterruptedException e) { 
            e.printStackTrace(); 
          } 
          end=new Date(); 
          System.out.printf("Core: Fork/Join: %d\n",
                            (end.getTime()-start.getTime())); 
        }

```

# 它是如何工作的...

当你执行示例时，你会看到`ForkJoinPool`和`TaskFJ`类比`ThreadPoolExecutor`和`Task`类有更好的性能。

如果你必须解决一个可以使用分而治之技术分割的问题，请使用`ForkJoinPool`类而不是`ThreadPoolExecutor`类。你将获得更好的性能。

# 参见

+   本章的*将线程管理委托给执行器*食谱

# 避免在锁内部使用阻塞操作

**阻塞操作**是那些在事件发生之前阻止当前线程执行的操作。典型的阻塞操作包括与控制台、文件或网络的输入或输出操作。

如果你在一个锁的临界区内部使用阻塞操作，你会降低应用程序的性能。当一个线程正在等待完成阻塞操作的事件时，应用程序的其余部分可能也在等待相同的事件；然而，其他线程将无法访问临界区并执行其代码（临界区的代码）。

在本食谱中，你将实现这种情况的一个示例。线程在临界区内部从控制台读取一行。这条指令使得应用程序的其他线程将被阻塞，直到用户输入该行。

# 准备工作

本食谱的示例已使用 Eclipse IDE 实现。如果你使用 Eclipse 或 NetBeans 等其他 IDE，请打开它并创建一个新的 Java 项目。

# 如何操作...

按照以下步骤实现示例：

1.  创建一个名为`Task`的类并指定它实现`Runnable`接口：

```java
        public class Task implements Runnable {

```

1.  声明一个名为`lock`的私有`Lock`属性：

```java
        private final Lock lock;

```

1.  实现类的构造函数以初始化其属性：

```java
        public Task (Lock lock) { 
          this.lock=lock; 
        }

```

1.  实现一个`run()`方法：

```java
        @Override 
        public void run() { 
          System.out.printf("%s: Starting\n",
                            Thread.currentThread().getName());

```

1.  使用`lock()`方法获取锁：

```java
        lock.lock();

```

1.  调用`criticalSection()`方法：

```java
        try { 
          criticalSection();

```

1.  从控制台读取一行：

```java
          System.out.printf("%s: Press a key to continue: \n",
                            Thread.currentThread().getName()); 
          InputStreamReader converter = new InputStreamReader
                                                    (System.in); 
          BufferedReader in = new BufferedReader(converter); 
          String line=in.readLine(); 
        } catch (IOException e) { 
          e.printStackTrace();

```

1.  使用`finally`部分中的`unlock()`方法释放锁：

```java
          } finally {          
            lock.unlock(); 
          } 
        }

```

1.  实现一个`criticalSection()`方法。等待一个随机的时间段：

```java
        private void criticalSection() { 
          Random random=new Random(); 
          int wait=random.nextInt(10); 
          System.out.printf("%s: Wait for %d seconds\n",
                            Thread.currentThread().getName(),wait); 
          try { 
            TimeUnit.SECONDS.sleep(wait); 
          } catch (InterruptedException e) { 
            e.printStackTrace(); 
          } 
        }

```

1.  通过创建一个名为`Main`的类并添加`main()`方法来实现应用程序的主类：

```java
        public class Main { 
          public static void main(String[] args) {

```

1.  创建一个名为`lock`的新`ReentrantLock`对象。创建 10 个`Task`对象和 10 个线程来执行它们：

```java
        ReentrantLock lock=new ReentrantLock(); 
        for (int i=0; i<10; i++) { 
          Task task=new Task(lock); 
          Thread thread=new Thread(task); 
          thread.start(); 
        }

```

# 它是如何工作的...

当您执行此示例时，10 个线程开始执行，但只有一个进入临界区，该临界区在`run()`方法中实现。由于每个任务在释放锁之前都会从控制台读取一行文本，因此所有应用程序都将被阻塞，直到您在控制台中输入文本。

# 参见

+   *尽可能短地持有锁*的技巧

# 避免使用已弃用的方法

Java 并发 API 也有一些已弃用的操作。这些是包含在 API 的第一个版本中的操作，但现在您不应使用它们。它们已被其他操作所取代，这些操作实现了比原始操作更好的实践。

最关键的已弃用操作是那些由`Thread`类提供的。这些操作包括：

+   `destroy()`: 在过去，此方法销毁线程。实际上，它抛出`NoSuchMethodError`异常。

+   `suspend()`: 此方法将线程的执行挂起，直到它被恢复。

+   `stop()`: 此方法强制线程完成其执行。

+   `resume()`: 此方法恢复线程的执行。

`ThreadGroup`类也有一些已弃用的方法，如下所示：

+   `suspend()`: 此方法将此线程组中属于此线程的所有线程的执行挂起

+   `stop()`: 此方法强制此线程组中所有线程的执行完成

+   `resume()`: 此方法恢复此线程组中所有线程的执行

`stop()`操作已被弃用，因为它可能引发不一致的错误。因为它强制线程完成其执行，您可能会遇到线程在操作完成之前完成其执行，并可能使数据处于不一致状态的情况。例如，如果您有一个正在修改银行账户的线程，并且在完成之前被停止，那么银行账户可能包含错误数据。

`stop()`操作也可能导致死锁情况。如果在线程执行由同步机制（例如，锁）保护的临界区时调用此操作，则此同步机制将继续阻塞，并且没有线程能够进入临界区。这就是为什么`suspend()`和`resume()`操作已被弃用的原因。

如果您需要这些操作的替代方案，可以使用一个内部属性来存储线程的状态。此属性必须使用同步访问进行保护，或者使用原子变量。您必须检查此属性值并根据它采取行动。请注意，您必须避免数据不一致和死锁情况，以确保应用程序的正确运行。

# 使用执行器而不是线程组

`ThreadGroup`类提供了一个机制，可以将线程组织成层次结构，这样你就可以通过一个调用对属于线程组的所有线程执行操作。默认情况下，所有线程都属于同一个组，但当你创建线程时可以指定不同的组。

无论如何，线程组不提供任何使其使用有趣的功能：

+   你必须创建线程并管理它们的状态

+   控制线程组所有线程状态的这些方法已被弃用，并建议不要使用。

如果你需要将线程分组到一个共同的架构下，最好使用`Executor`实现，例如`ThreadPoolExecutor`。它提供了更多功能，具体如下：

+   你不必担心线程的管理。执行器创建并重用线程以节省执行资源。

+   你可以通过实现`Runnable`或`Callable`接口来实施你的并发任务。`Callable`接口允许你实现返回结果的任务，这比传统任务提供了很大的优势。

+   当你向执行器发送任务时，它返回一个`Future`对象，允许你轻松地知道任务的状态以及如果它已经完成执行，则返回的结果。

+   你可以使用`ScheduledThreadPoolExecutor`类实现的特殊执行器来安排任务并重复执行它们。

+   你可以轻松控制执行器使用的资源。你可以设置池中线程的最大数量，这样你的执行器一次就不会运行超过那么多任务。

由于这些原因，最好你不使用线程组而使用执行器。

# 参见

+   本章中关于*将线程管理委托给执行器*的配方

# 使用流处理大数据集

`Stream`接口是一系列可以按顺序或并行过滤和转换以获得最终结果的元素序列。这个最终结果可以是原始数据类型（整数、长整型等）、对象或数据结构。这些是更好地定义`Stream`的特征：

+   流是一系列数据，而不是数据结构。

+   你可以从不同的源创建流，如集合（列表、数组等）、文件、字符串或提供流元素的类。

+   你不能访问流中的单个元素。

+   你不能修改流的源。

+   流定义了两种类型的操作：中间操作，它产生一个新的`Stream`接口，允许你转换、过滤、映射或排序流中的元素，以及终端操作，它生成操作的最终结果。流管道由零个或多个中间操作和一个最终操作组成。

+   中间操作是懒执行的。它们不会在终端操作开始执行之前执行。如果 Java 检测到中间操作不会影响操作最终结果，它可以避免对流中的元素或元素集合执行中间操作。

当你需要以并发方式实现处理大量数据的操作时，你可以使用**Java 并发 API**的不同元素来实现它。你可以将 Java 线程分配给**fork/join 框架**或**Executor 框架**，但我认为并行流是最佳选择。在这个菜谱中，我们将实现一个示例来解释使用并行流提供的优势。

# 准备工作

本菜谱的示例已使用 Eclipse IDE 实现。如果你使用 Eclipse 或 NetBeans 等其他 IDE，请打开它并创建一个新的 Java 项目。

# 如何做到这一点...

按照以下步骤实现示例：

1.  创建一个名为`Person`的类。这个类将有六个属性来定义一个人的基本特征。我们将实现获取和设置属性值的方法，但它们将不包括在这里：

```java
        public class Person { 
          private int id; 
          private String firstName; 
          private String lastName; 
          private Date birthDate; 
          private int salary; 
          private double coeficient;

```

1.  现在，实现一个名为`PersonGenerator`的类。这个类将只有一个名为`generatedPersonList()`的方法，用于生成具有指定参数大小的随机`Person`对象列表。这是该类的源代码：

```java
        public class PersonGenerator { 
          public static List<Person> generatePersonList (int size) { 
            List<Person> ret = new ArrayList<>(); 

            String firstNames[] = {"Mary","Patricia","Linda",
                                   "Barbara","Elizabeth","James",
                                   "John","Robert","Michael","William"}; 
            String lastNames[] = {"Smith","Jones","Taylor",
                                  "Williams","Brown","Davies",
                                  "Evans","Wilson","Thomas","Roberts"}; 

            Random randomGenerator=new Random(); 
            for (int i=0; i<size; i++) { 
              Person person=new Person(); 
              person.setId(i); 
              person.setFirstName(firstNames
                                       [randomGenerator.nextInt(10)]); 
              person.setLastName(lastNames
                                     [randomGenerator.nextInt(10)]); 
              person.setSalary(randomGenerator.nextInt(100000)); 
              person.setCoeficient(randomGenerator.nextDouble()*10); 
              Calendar calendar=Calendar.getInstance(); 
              calendar.add(Calendar.YEAR, -randomGenerator
                                                     .nextInt(30)); 
              Date birthDate=calendar.getTime(); 
              person.setBirthDate(birthDate); 

              ret.add(person); 
            } 
            return ret; 
          } 
        }

```

1.  现在，实现一个名为`PersonMapTask`的任务。这个任务的主要目的是将人员列表转换为映射，其中键将是人员的姓名，值将是具有与键相同名称的`Person`对象的列表。我们将使用 fork/join 框架来实现这种转换，因此`PersonMapTask`将扩展`RecursiveAction`类：

```java
        public class PersonMapTask extends RecursiveAction {

```

1.  `PersonMapTask`类将有两个私有属性：要处理的`Person`对象列表和用于存储结果的`ConcurrentHashMap`。我们将使用类的构造函数来初始化这两个属性：

```java
        private List<Person> persons; 
        private ConcurrentHashMap<String, ConcurrentLinkedDeque
                                                <Person>> personMap; 

        public PersonMapTask(List<Person> persons, ConcurrentHashMap
                   <String, ConcurrentLinkedDeque<Person>> personMap) { 
          this.persons = persons; 
          this.personMap = personMap; 
        }

```

1.  现在是时候实现`compute()`方法了。如果列表少于 1,000 个元素，我们将处理元素并将它们插入到`ConcurrentHashMap`中。我们将使用`computeIfAbsent()`方法获取与键关联的`List`或如果键不存在于映射中，则生成一个新的`ConcurrentMapedDeque`对象：

```java
        protected void compute() { 

          if (persons.size() < 1000) { 

            for (Person person: persons) { 
              ConcurrentLinkedDeque<Person> personList=personMap
                     .computeIfAbsent(person.getFirstName(), name -> { 
              return new ConcurrentLinkedDeque<>(); 
              }); 

```

```java
              personList.add(person); 
            } 
            return; 
          }

```

1.  如果`List`有超过 1,000 个元素，我们将创建两个子任务并将列表的一部分处理过程委托给它们：

```java
          PersonMapTask child1, child2; 

          child1=new PersonMapTask(persons.subList(0,persons.size()/2),
                                   personMap); 
          child2=new PersonMapTask(persons.subList(persons.size()/2,
                                                   persons.size()),
                                   personMap); 

            invokeAll(child1,child2);   
          } 
        }

```

1.  最后，实现带有`main()`方法的`Main`类。首先，生成一个包含 100,000 个随机`Person`对象的列表：

```java
        public class Main { 

          public static void main (String[] args) { 
            List<Person> persons=PersonGenerator
                                        .generatePersonList(100000);

```

1.  然后，比较两种方法来生成以名称作为键、`Person`作为值的`Map`。列表将使用并行`Stream`函数和`collect()`方法使用`groupingByConcurrent()`收集器：

```java
        Date start, end; 

        start =  new Date(); 
        Map<String, List<Person>> personsByName = persons
                                                  .parallelStream() 
        .collect(Collectors.groupingByConcurrent(p -> p
                                                   .getFirstName())); 
        end = new Date(); 
        System.out.printf("Collect: %d - %d\n", personsByName.size(),
                          end.getTime()-start.getTime());

```

1.  第二种选择是使用 fork/join 框架和`PersonMapTask`类：

```java
            start = new Date(); 
            ConcurrentHashMap<String, ConcurrentLinkedDeque<Person>>
                          forkJoinMap=new ConcurrentHashMap<>(); 
            PersonMapTask personMapTask=new PersonMapTask
                                            (persons,forkJoinMap); 
            ForkJoinPool.commonPool().invoke(personMapTask); 
            end = new Date(); 

            System.out.printf("Collect ForkJoinPool: %d - %d\n",
                              forkJoinMap.size(),
                              end.getTime()-start.getTime()); 
          } 
        }

```

# 它是如何工作的...

在这个菜谱中，我们实现了从`List`到`Map`的同一算法的两个不同版本。如果你执行它，你会得到相同的结果和相似的执行时间（至少在我用四核计算机执行示例时，后者是正确的）。我们使用流获得的最大优势是解决方案的简单性和其开发时间。我们只用一行代码就实现了解决方案。而在另一种情况下，我们使用并发数据结构实现了一个新的类（`PersonMapTask`），然后在 fork/join 框架中执行它。

使用流，你可以将你的算法分解成简单的步骤，这些步骤可以用优雅的方式表达，易于编程和理解。

# 参见

+   第六章中的*从不同来源创建流*、*减少流元素*和*排序流元素*的菜谱，*并行和反应流*

# 其他技巧和窍门

在这个最后的菜谱中，我们包括了本章其他菜谱中没有包含的其他技巧和窍门：

+   在可能的情况下，使用并发设计模式：在软件工程中，设计模式是解决常见问题的方案。它们在软件开发和并发应用中普遍使用，并不例外。如信号量、 rendezvous 和互斥锁等模式定义了如何在具体情况下实现并发应用程序，并且它们已被用于实现并发工具。

+   在尽可能高的级别上实现并发：丰富的线程 API，如 Java 并发 API，为你提供了不同的类来实现应用程序中的并发。尽量使用那些提供更高抽象级别的类。这将使你更容易实现你的算法，并且它们被优化以提供比直接使用线程更好的性能。因此，性能不会成为问题。

+   考虑可伸缩性：当你实现一个并发算法时，主要目标之一是利用你计算机的所有资源，特别是处理器或核心的数量。但这个数字可能会随时间变化。当你设计一个并发算法时，不要预设你的应用程序将要执行的核心或处理器的数量。动态获取系统信息。例如，在 Java 中，你可以使用`Runtime.getRuntime().availableProcessors()`方法来获取它，并让你的算法使用这些信息来计算它将要执行的任务数量。

+   在可能的情况下，优先使用局部线程变量而不是静态和共享变量：线程局部变量是一种特殊的变量。每个任务都将为这个变量有一个独立的值，因此你不需要任何同步机制来保护对它的访问。

# 参见

+   本章所有菜谱
