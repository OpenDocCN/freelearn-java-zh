# 第三章：从执行者中获得最大效益

在第二章中，*管理大量线程-执行者*，我们介绍了执行者的基本特性，作为改进执行大量并发任务的并发应用程序性能的一种方式。在本章中，我们将进一步解释执行者的高级特性，使它们成为您并发应用程序的强大工具。在本章中，我们将涵盖以下内容：

+   执行者的高级特性

+   第一个例子-高级服务器应用程序

+   第二个例子-执行周期性任务

+   有关执行者的其他信息

# 执行者的高级特性

执行者是一个允许程序员执行并发任务而不必担心线程的创建和管理的类。程序员创建`Runnable`对象并将它们发送到执行者，执行者创建和管理必要的线程来执行这些任务。在第二章中，*管理大量线程-执行者*，我们介绍了执行者框架的基本特性：

+   如何创建执行者以及我们创建执行者时的不同选项

+   如何将并发任务发送到执行者

+   如何控制执行者使用的资源

+   执行者在内部如何使用线程池来优化应用程序的性能

但是，执行者可以为您提供更多选项，使其成为并发应用程序中的强大机制。

## 取消任务

您可以在将任务发送到执行者后取消任务的执行。使用`submit()`方法将`Runnable`对象发送到执行者时，它返回`Future`接口的实现。这个类允许您控制任务的执行。它具有`cancel()`方法，尝试取消任务的执行。它接收一个布尔值作为参数。如果它采用`true`值并且执行者正在执行此任务，则将中断执行任务的线程。

以下是您希望取消的任务无法取消的情况：

+   任务已经被取消

+   任务已经完成执行

+   任务正在运行，并且您向`cancel()`方法提供了`false`作为参数

+   API 文档中未指定的其他原因

`cancel()`方法返回一个布尔值，指示任务是否已取消。

## 安排任务的执行

`ThreadPoolExecutor`类是`Executor`和`ExecutorService`接口的基本实现。但是，Java 并发 API 提供了这个类的扩展，以允许执行计划任务。这是`ScheduledThreadPoolExeuctor`类，您可以：

+   在延迟后执行任务

+   定期执行任务；这包括以固定速率或固定延迟执行任务

## 重写执行者方法

执行者框架是一个非常灵活的机制。您可以实现自己的执行者，扩展现有类（`ThreadPoolExecutor`或`ScheduledThreadPoolExecutor`）以获得所需的行为。这些类包括使更改执行者工作方式变得容易的方法。如果您重写`ThreadPoolExecutor`，可以重写以下方法：

+   `beforeExecute()`：此方法在执行者中的并发任务执行之前调用。它接收将要执行的`Runnable`对象和将执行它们的`Thread`对象。此方法接收的`Runnable`对象是`FutureTask`类的实例，而不是使用`submit()`方法将`Runnable`对象发送到执行者的`Runnable`对象。

+   `afterExecute()`: 这个方法在执行器中的并发任务执行后被调用。它接收到已执行的`Runnable`对象和一个存储可能在任务内部抛出的异常的`Throwable`对象。与`beforeExecute()`方法一样，`Runnable`对象是`FutureTask`类的一个实例。

+   `newTaskFor()`: 这个方法创建将要执行`submit()`方法发送的`Runnable`对象的任务。它必须返回`RunnableFuture`接口的一个实现。默认情况下，Open JDK 8 和 Oracle JDK 8 返回`FutureTask`类的一个实例，但这种情况在将来的实现中可能会改变。

如果您扩展了`ScheduledThreadPoolExecutor`类，可以重写`decorateTask()`方法。这个方法类似于用于计划任务的`newTaskFor()`方法。它允许您重写执行器执行的任务。

## 更改一些初始化参数

您还可以通过更改创建时的一些参数来更改执行器的行为。最有用的是：

+   `BlockingQueue<Runnable>`: 每个执行器都使用内部的`BlockingQueue`来存储等待执行的任务。您可以将此接口的任何实现作为参数传递。例如，您可以更改执行任务的默认顺序。

+   `ThreadFactory`: 您可以指定`ThreadFactory`接口的一个实现，执行器将使用该工厂来创建执行任务的线程。例如，您可以使用`ThreadFactory`接口来创建`Thread`类的扩展，该扩展保存有关任务执行时间的日志信息。

+   `RejectedExecutionHandler`: 在调用`shutdown()`或`shutdownNow()`方法之后，发送到执行器的所有任务都将被拒绝。您可以指定`RejectedExecutionHandler`接口的一个实现来管理这种情况。

# 第一个示例 - 高级服务器应用程序

在第二章中，*管理大量线程 - 执行器*，我们介绍了一个客户端/服务器应用程序的示例。我们实现了一个服务器来搜索世界银行的世界发展指标数据，并且一个客户端对该服务器进行多次调用以测试执行器的性能。

在本节中，我们将扩展该示例以添加以下特性：

+   您可以使用新的取消查询取消服务器上的查询执行。

+   您可以使用优先级参数控制查询的执行顺序。具有更高优先级的任务将首先执行。

+   服务器将计算使用服务器的不同用户使用的任务数量和总执行时间。

为了实现这些新特性，我们对服务器进行了以下更改：

+   我们为每个查询添加了两个参数。第一个是发送查询的用户的名称，另一个是查询的优先级。查询的新格式如下：

+   **查询**: `q;username;priority;codCountry;codIndicator;year`，其中`username`是用户的名称，`priority`是查询的优先级，`codCountry`是国家代码，`codIndicator`是指标代码，`year`是一个可选参数，用于查询的年份。

+   **报告**: `r;username;priority;codIndicator`，其中`username`是用户的名称，`priority`是查询的优先级，`codIndicator`是您要报告的指标代码。

+   **状态**: `s;username;priority`，其中`username`是用户的名称，`priority`是查询的优先级。

+   **停止**: `z;username;priority`，其中`username`是用户的名称，`priority`是查询的优先级。

+   我们已经实现了一个新的查询：

+   **取消**：`c;username;priority`，其中`username`是用户的名称，`priority`是查询的优先级。

+   我们实现了自己的执行器来：

+   计算每个用户的服务器使用情况

+   按优先级执行任务

+   控制任务的拒绝

+   我们已经调整了`ConcurrentServer`和`RequestTask`以考虑服务器的新元素

服务器的其余元素（缓存系统、日志系统和`DAO`类）都是相同的，因此不会再次描述。

## ServerExecutor 类

正如我们之前提到的，我们实现了自己的执行器来执行服务器的任务。我们还实现了一些额外但必要的类来提供所有功能。让我们描述这些类。

### 统计对象

我们的服务器将计算每个用户在其上执行的任务数量以及这些任务的总执行时间。为了存储这些数据，我们实现了`ExecutorStatistics`类。它有两个属性来存储信息：

```java
public class ExecutorStatistics {
    private AtomicLong executionTime = new AtomicLong(0L);
    private AtomicInteger numTasks = new AtomicInteger(0);
```

这些属性是`AtomicVariables`，支持对单个变量的原子操作。这允许您在不使用任何同步机制的情况下在不同的线程中使用这些变量。然后，它有两种方法来增加任务数量和执行时间：

```java
    public void addExecutionTime(long time) {
        executionTime.addAndGet(time);
    }
    public void addTask() {
        numTasks.incrementAndGet();
    }
```

最后，我们添加了获取这两个属性值的方法，并重写了`toString()`方法以便以可读的方式获取信息：

```java
    @Override
    public String toString() {
        return "Executed Tasks: "+getNumTasks()+". Execution Time: "+getExecutionTime();
    }
```

### 被拒绝的任务控制器

当您创建一个执行器时，可以指定一个类来管理其被拒绝的任务。当您在执行器中调用`shutdown()`或`shutdownNow()`方法后提交任务时，执行器会拒绝该任务。

为了控制这种情况，我们实现了`RejectedTaskController`类。这个类实现了`RejectedExecutionHandler`接口，并实现了`rejectedExecution()`方法：

```java
public class RejectedTaskController implements RejectedExecutionHandler {

    @Override
    public void rejectedExecution(Runnable task, ThreadPoolExecutor executor) {
        ConcurrentCommand command=(ConcurrentCommand)task;
        Socket clientSocket=command.getSocket();
        try {
            PrintWriter out = new PrintWriter(clientSocket.getOutputStream(),true);

            String message="The server is shutting down."
                +" Your request can not be served."
                +" Shutting Down: "
                +String.valueOf(executor.isShutdown())
                +". Terminated: "
                +String.valueOf(executor.isTerminated())
                +". Terminating: "
                +String.valueOf(executor.isTerminating());
            out.println(message);
            out.close();
            clientSocket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

`rejectedExecution()`方法每拒绝一个任务调用一次，并接收被拒绝的任务和拒绝任务的执行器作为参数。

### 执行器任务

当您向执行器提交一个`Runnable`对象时，它不会直接执行该`Runnable`对象。它会创建一个新对象，即`FutureTask`类的实例，正是这个任务由执行器的工作线程执行。

在我们的情况下，为了测量任务的执行时间，我们在`ServerTask`类中实现了我们自己的`FutureTask`实现。它扩展了`FutureTask`类，并实现了`Comparable`接口，如下所示：

```java
public class ServerTask<V> extends FutureTask<V> implements Comparable<ServerTask<V>>{
```

在内部，它将要执行的查询存储为`ConcurrentCommand`对象：

```java
    private ConcurrentCommand command;
```

在构造函数中，它使用`FutureTask`类的构造函数并存储`ConcurrentCommand`对象：

```java
    public ServerTask(ConcurrentCommand command) {
        super(command, null);
        this.command=command;
    }

    public ConcurrentCommand getCommand() {
        return command;
    }

    public void setCommand(ConcurrentCommand command) {
        this.command = command;
    }
```

最后，它实现了`compareTo()`操作，比较两个`ServerTask`实例存储的命令。这可以在以下代码中看到：

```java
    @Override
    public int compareTo(ServerTask<V> other) {
        return command.compareTo(other.getCommand());
    }
```

### 执行器

现在我们有了执行器的辅助类，我们必须实现执行器本身。我们实现了`ServerExecutor`类来实现这个目的。它扩展了`ThreadPoolExecutor`类，并具有一些内部属性，如下所示：

+   `startTimes`：这是一个`ConcurrentHashMap`，用于存储每个任务的开始日期。类的键将是`ServerTask`对象（一个`Runnable`对象），值将是一个`Date`对象。

+   `executionStatistics`：这是一个`ConcurrentHashMap`，用于存储每个用户的使用统计。键将是用户名，值将是一个`ExecutorStatistics`对象。

+   `CORE_POOL_SIZE`，`MAXIMUM_POOL_SIZE`和`KEEP_ALIVE_TIME`：这些是用于定义执行器特性的常量。

+   `REJECTED_TASK_CONTROLLER`: 这是一个`RejectedTaskController`类的属性，用于控制执行器拒绝的任务。

这可以通过以下代码来解释：

```java
public class ServerExecutor extends ThreadPoolExecutor {
    private ConcurrentHashMap<Runnable, Date> startTimes;
    private ConcurrentHashMap<String, ExecutorStatistics> executionStatistics;
    private static int CORE_POOL_SIZE = Runtime.getRuntime().availableProcessors();
    private static int MAXIMUM_POOL_SIZE = Runtime.getRuntime().availableProcessors();
    private static long KEEP_ALIVE_TIME = 10;

    private static RejectedTaskController REJECTED_TASK_CONTROLLER = new RejectedTaskController();

    public ServerExecutor() {
        super(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_TIME, TimeUnit.SECONDS, new PriorityBlockingQueue<>(), REJECTED_TASK_CONTROLLER);

        startTimes = new ConcurrentHashMap<>();
        executionStatistics = new ConcurrentHashMap<>();
    }
```

该类的构造函数调用父类构造函数，创建一个`PriorityBlockingQueue`类来存储将在执行器中执行的任务。该类根据`compareTo()`方法的执行结果对元素进行排序（因此存储在其中的元素必须实现`Comparable`接口）。使用此类将允许我们按优先级执行任务。

然后，我们重写了`ThreadPoolExecutor`类的一些方法。首先是`beforeExecute()`方法。该方法在每个任务执行之前执行。它接收`ServerTask`对象作为参数，以及将要执行任务的线程。在我们的情况下，我们使用`ConcurrentHashMap`存储每个任务的开始日期：

```java
    protected void beforeExecute(Thread t, Runnable r) {
        super.beforeExecute(t, r);
        startTimes.put(r, new Date());
    }
```

下一个方法是`afterExecute()`方法。该方法在执行器中每个任务执行后执行，并接收已执行的`ServerTask`对象作为参数和一个`Throwable`对象。只有在任务执行过程中抛出异常时，最后一个参数才会有值。在我们的情况下，我们将使用此方法来：

+   计算任务的执行时间。

+   以以下方式更新用户的统计信息：

```java
    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        super.afterExecute(r, t);
        ServerTask<?> task=(ServerTask<?>)r;
        ConcurrentCommand command=task.getCommand();

        if (t==null) {
            if (!task.isCancelled()) {
                Date startDate = startTimes.remove(r);
                Date endDate=new Date();
                long executionTime= endDate.getTime() - startDate.getTime();
                            ;
                ExecutorStatistics statistics = executionStatistics.computeIfAbsent (command.getUsername(), n -> new ExecutorStatistics());
                statistics.addExecutionTime(executionTime);
                statistics.addTask();
                ConcurrentServer.finishTask (command.getUsername(), command);
            }
            else {

                String message="The task" + command.hashCode() + "of user" + command.getUsername() + "has been cancelled.";
                Logger.sendMessage(message);
            }

        } else {

            String message="The exception "
                    +t.getMessage()
                    +" has been thrown.";
            Logger.sendMessage(message);
        }
    }
```

最后，我们重写了`newTaskFor()`方法。该方法将被执行，将我们通过`submit()`方法发送到执行器的`Runnable`对象转换为由执行器执行的`FutureTask`实例。在我们的情况下，我们将默认的`FutureTask`类替换为我们的`ServerTask`对象：

```java
    @Override
    protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new ServerTask<T>(runnable);
    }
```

我们在执行器中包含了一个额外的方法，用于将执行器中存储的所有统计信息写入日志系统。此方法将在服务器执行结束时调用，稍后您将看到。我们有以下代码：

```java
    public void writeStatistics() {

        for(Entry<String, ExecutorStatistics> entry: executionStatistics.entrySet()) {
             String user = entry.getKey();
             ExecutorStatistics stats = entry.getValue(); Logger.sendMessage(user+":"+stats);
        }
    }
```

## 命令类

命令类执行您可以发送到服务器的不同查询。您可以向我们的服务器发送五种不同的查询：

+   **查询**：这是用于获取有关国家、指标和可选年份的信息的命令。由`ConcurrentQueryCommand`类实现。

+   **报告**：这是用于获取有关指标的信息的命令。由`ConcurrentReportCommand`类实现。

+   **状态**：这是用于获取服务器状态信息的命令。由`ConcurrentStatusCommand`类实现。

+   **取消**：这是用于取消用户任务执行的命令。由`ConcurrentCancelCommand`类实现。

+   **停止**：这是用于停止服务器执行的命令。由`ConcurrentStopCommand`类实现。

我们还有`ConcurrentErrorCommand`类，用于处理服务器接收到未知命令的情况，以及`ConcurrentCommand`类，它是所有命令的基类。

### ConcurrentCommand 类

这是每个命令的基类。它包括所有命令共有的行为，包括以下内容：

+   调用实现每个命令特定逻辑的方法

+   将结果写入客户端

+   关闭通信中使用的所有资源

该类扩展了`Command`类，并实现了`Comparable`和`Runnable`接口。在第二章的示例中，命令是简单的类，但在这个示例中，并发命令是将发送到执行器的`Runnable`对象：

```java
public abstract class ConcurrentCommand extends Command implements Comparable<ConcurrentCommand>, Runnable{
```

它有三个属性：

+   `username`：这是用于存储发送查询的用户的名称。

+   `priority`：这是用于存储查询的优先级。它将确定查询的执行顺序。

+   `socket`：这是与客户端通信中使用的套接字。

该类的构造函数初始化了这些属性：

```java
    private String username;
    private byte priority;
    private Socket socket;

    public ConcurrentCommand(Socket socket, String[] command) {
        super(command);
        username=command[1];
        priority=Byte.parseByte(command[2]);
        this.socket=socket;

    }
```

这个类的主要功能在抽象的`execute()`方法中，每个具体命令都将通过该方法来计算和返回查询的结果，并且在`run()`方法中。`run()`方法调用`execute()`方法，将结果存储在缓存中，将结果写入套接字，并关闭通信中使用的所有资源。我们有以下内容：

```java
    @Override
    public abstract String execute();

    @Override
    public void run() {

        String message="Running a Task: Username: "
                +username
                +"; Priority: "
                +priority;
        Logger.sendMessage(message);

        String ret=execute();

        ParallelCache cache = ConcurrentServer.getCache();

        if (isCacheable()) {
            cache.put(String.join(";",command), ret);
        }

        try {
            PrintWriter out = new PrintWriter(socket.getOutputStream(),true);
            out.println(ret);
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        System.out.println(ret);
    }
```

最后，`compareTo()`方法使用优先级属性来确定任务的顺序。这将被`PriorityBlockingQueue`类用来对任务进行排序，因此具有更高优先级的任务将首先执行。请注意，当`getPriority()`方法返回较低的值时，任务的优先级更高。如果任务的`getPriority()`返回`1`，那么该任务的优先级将高于`getPriority()`方法返回`2`的任务：

```java
    @Override
    public int compareTo(ConcurrentCommand o) {
        return Byte.compare(o.getPriority(), this.getPriority());
    }
```

### 具体命令

我们对实现不同命令的类进行了微小的更改，并添加了一个由`ConcurrentCancelCommand`类实现的新命令。这些类的主要逻辑包含在`execute()`方法中，该方法计算查询的响应并将其作为字符串返回。

新的`ConcurrentCancelCommand`的`execute()`方法调用`ConcurrentServer`类的`cancelTasks()`方法。此方法将停止与作为参数传递的用户相关的所有待处理任务的执行：

```java
    @Override
    public String execute() {
        ConcurrentServer.cancelTasks(getUsername());

        String message = "Tasks of user "
                +getUsername()
                +" has been cancelled.";
        Logger.sendMessage(message);
        return message;
    }
```

`ConcurrentReportCommand`的`execute()`方法使用`WDIDAO`类的`query()`方法来获取用户请求的数据。在第二章中，*管理大量线程-执行者*，您可以找到此方法的实现。实现几乎相同。唯一的区别是命令数组索引如下：

```java
    @Override
    public String execute() {

        WDIDAO dao=WDIDAO.getDAO();

        if (command.length==5) {
            return dao.query(command[3], command[4]);
        } else if (command.length==6) {
            try {
                return dao.query(command[3], command[4], Short.parseShort(command[5]));
            } catch (NumberFormatException e) {
                return "ERROR;Bad Command";
            }
        } else {
            return "ERROR;Bad Command";
        }
    }
```

`ConcurrentQueryCommand`的`execute()`方法使用`WDIDAO`类的`report()`方法来获取数据。在第二章中，*管理大量线程-执行者*，您还可以找到此方法的实现。这里的实现几乎相同。唯一的区别是命令数组索引：

```java
    @Override
    public String execute() {

        WDIDAO dao=WDIDAO.getDAO();
        return dao.report(command[3]);
    }
```

`ConcurrentStatusCommand`在其构造函数中有一个额外的参数：`Executor`对象，它将执行命令。此命令使用此对象来获取有关执行程序的信息，并将其作为响应发送给用户。实现几乎与第二章中的相同。我们使用相同的方法来获取`Executor`对象的状态。

`ConcurrentStopCommand`和`ConcurrentErrorCommand`与第二章中的相同，因此我们没有包含它们的源代码。

## 服务器部分

服务器部分接收来自服务器客户端的查询，并创建执行查询的命令类，并将其发送到执行程序。由两个类实现：

+   `ConcurrentServer`类：它包括服务器的`main()`方法和取消任务以及完成系统执行的其他方法。

+   `RequestTask`类：此类创建命令并将其发送到执行程序

与第二章的示例*管理大量线程-执行器*的主要区别是`RequestTask`类的作用。在`SimpleServer`示例中，`ConcurrentServer`类为每个查询创建一个`RequestTask`对象并将其发送到执行器。在这个例子中，我们只会有一个`RequestTask`的实例，它将作为一个线程执行。当`ConcurrentServer`接收到一个连接时，它将把用于与客户端通信的套接字存储在一个并发的待处理连接列表中。`RequestTask`线程读取该套接字，处理客户端发送的数据，创建相应的命令，并将命令发送到执行器。

这种改变的主要原因是只在执行器中留下查询的代码，并将预处理的代码留在执行器之外。

### ConcurrentServer 类

`ConcurrentServer`类需要一些内部属性才能正常工作：

+   一个`ParallelCache`实例用于使用缓存系统。

+   一个`ServerSocket`实例用于接收来自客户端的连接。

+   一个`boolean`值用于知道何时停止执行。

+   一个`LinkedBlockingQueue`用于存储发送消息给服务器的客户端的套接字。这些套接字将由`RequestTask`类处理。

+   一个`ConcurrentHashMap`用于存储与执行器中的每个任务相关的`Future`对象。键将是发送查询的用户的用户名，值将是另一个`Map`，其键将是`ConcurrenCommand`对象，值将是与该任务相关联的`Future`实例。我们使用这些`Future`实例来取消任务的执行。

+   一个`RequestTask`实例用于创建命令并将其发送到执行器。

+   一个`Thread`对象来执行`RequestTask`对象。

这段代码如下：

```java
public class ConcurrentServer {
    private static ParallelCache cache;
    private static volatile boolean stopped=false;
    private static LinkedBlockingQueue<Socket> pendingConnections;
    private static ConcurrentMap<String, ConcurrentMap<ConcurrentCommand, ServerTask<?>>> taskController;
    private static Thread requestThread;
    private static RequestTask task;
```

这个类的`main()`方法初始化这些对象，并打开`ServerSocket`实例以监听来自客户端的连接。此外，它创建`RequestTask`对象并将其作为线程执行。它将循环执行，直到`shutdown()`方法改变了 stopped 属性的值。之后，它等待`Executor`对象的完成，使用`RequestTask`对象的`endTermination()`方法，并使用`finishServer()`方法关闭`Logger`系统和`RequestTask`对象：

```java
    public static void main(String[] args) {

        WDIDAO dao=WDIDAO.getDAO();
        cache=new ParallelCache();
        Logger.initializeLog();
        pendingConnections = new LinkedBlockingQueue<Socket>();
        taskController = new ConcurrentHashMap<String, ConcurrentHashMap<Integer, Future<?>>>();
        task=new RequestTask(pendingConnections, taskController);
        requestThread=new Thread(task);
        requestThread.start();

        System.out.println("Initialization completed.");

        serverSocket= new ServerSocket(Constants.CONCURRENT_PORT);
        do {
            try {
                Socket clientSocket = serverSocket.accept();
                pendingConnections.put(clientSocket);
            } catch (Exception e) {
                e.printStackTrace();
            }
        } while (!stopped);
        finishServer();
        System.out.println("Shutting down cache");
        cache.shutdown();
        System.out.println("Cache ok" + new Date());

    }
```

它包括两种方法来关闭服务器的执行器。`shutdown()`方法改变`stopped`变量的值，并关闭`serverSocket`实例。`finishServer()`方法停止执行器，中断执行`RequestTask`对象的线程，并关闭`Logger`系统。我们将这个过程分成两部分，以便在服务器的最后一条指令之前使用`Logger`系统：

```java
    public static void shutdown() {
        stopped=true;
        try {
            serverSocket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void finishServer() {
        System.out.println("Shutting down the server...");
        task.shutdown();
        System.out.println("Shutting down Request task");
        requestThread.interrupt();
        System.out.println("Request task ok");
        System.out.println("Closing socket");
        System.out.println("Shutting down logger");
        Logger.sendMessage("Shutting down the logger");
        Logger.shutdown();
        System.out.println("Logger ok");
        System.out.println("Main server thread ended");
    }
```

服务器包括取消与用户关联的任务的方法。正如我们之前提到的，`Server`类使用嵌套的`ConcurrentHashMap`来存储与用户关联的所有任务。首先，我们获取一个用户的所有任务的`Map`，然后我们处理这些任务的所有`Future`对象，调用`Future`对象的`cancel()`方法。我们将值`true`作为参数传递，因此如果执行器正在运行该用户的任务，它将被中断。我们已经包括了必要的代码来避免`ConcurrentCancelCommand`的取消：

```java
    public static void cancelTasks(String username) {

        ConcurrentMap<ConcurrentCommand, ServerTask<?>> userTasks = taskController.get(username);
        if (userTasks == null) {
            return;
        }
        int taskNumber = 0;

        Iterator<ServerTask<?>> it = userTasks.values().iterator();
        while(it.hasNext()) {
            ServerTask<?> task = it.next();
             ConcurrentCommand command = task.getCommand();
              if(!(command instanceof ConcurrentCancelCommand) && task.cancel(true)) {
                    taskNumber++;
                    Logger.sendMessage("Task with code "+command.hashCode()+"cancelled: "+command.getClass().getSimpleName());
                    it.remove();
              }
        }
        String message=taskNumber+" tasks has been cancelled.";
        Logger.sendMessage(message);
    }
```

最后，我们已经包括了一个方法，当任务正常执行完毕时，从我们的`ServerTask`对象的嵌套映射中消除与任务相关的`Future`对象。这就是`finishTask()`方法：

```java
    public static void finishTask(String username, ConcurrentCommand command) {

        ConcurrentMap<ConcurrentCommand, ServerTask<?>> userTasks = taskController.get(username);
        userTasks.remove(command);
        String message = "Task with code "+command.hashCode()+" has finished";
        Logger.sendMessage(message);

    }
```

### RequestTask 类

`RequestTask`类是`ConcurrentServer`类与客户端连接和`Executor`类执行并发任务之间的中介。它与客户端打开套接字，读取查询数据，创建适当的命令，并将其发送到执行器。

它使用一些内部属性：

+   `LinkedBlockingQueue`，`ConcurrentServer`类在其中存储客户端套接字

+   `ServerExecutor`用于执行命令作为并发任务。

+   使用`ConcurrentHashMap`存储与任务相关的`Future`对象

该类的构造函数初始化了所有这些对象：

```java
public class RequestTask implements Runnable {
    private LinkedBlockingQueue<Socket> pendingConnections;
    private ServerExecutor executor = new ServerExecutor();
    private ConcurrentMap<String, ConcurrentMap<ConcurrentCommand, ServerTask<?>>> taskController;
    public RequestTask(LinkedBlockingQueue<Socket> pendingConnections, ConcurrentHashMap<String, ConcurrentHashMap<Integer, Future<?>>> taskController) {
        this.pendingConnections = pendingConnections;
        this.taskController = taskController;
    }
```

该类的主要方法是`run()`方法。它执行一个循环，直到线程被中断，处理存储在`pendingConnections`对象中的套接字。在该对象中，`ConcurrentServer`类存储了与发送查询到服务器的不同客户端通信的套接字。它打开套接字，读取数据，并创建相应的命令。它还将命令发送到执行器，并将`Future`对象存储在与任务的`hashCode`和发送查询的用户相关联的双重`ConcurrentHashMap`中：

```java
    public void run() {
        try {
            while (!Thread.currentThread().interrupted()) {
                try {
                    Socket clientSocket = pendingConnections.take();
                    BufferedReader in = new BufferedReader(new InputStreamReader (clientSocket.getInputStream()));
                    String line = in.readLine();

                    Logger.sendMessage(line);

                    ConcurrentCommand command;

                    ParallelCache cache = ConcurrentServer.getCache();
                    String ret = cache.get(line);
                    if (ret == null) {
                        String[] commandData = line.split(";");
                        System.out.println("Command: " + commandData[0]);
                        switch (commandData[0]) {
                        case "q":
                            System.out.println("Query");
                            command = new ConcurrentQueryCommand(clientSocket, commandData);
                            break;
                        case "r":
                            System.out.println("Report");
                            command = new ConcurrentReportCommand (clientSocket, commandData);
                            break;
                        case "s":
                            System.out.println("Status");
                            command = new ConcurrentStatusCommand(executor, clientSocket, commandData);
                            break;
                        case "z":
                            System.out.println("Stop");
                            command = new ConcurrentStopCommand(clientSocket, commandData);
                            break;
                        case "c":
                            System.out.println("Cancel");
                            command = new ConcurrentCancelCommand (clientSocket, commandData);
                            break;
                        default:
                            System.out.println("Error");
                            command = new ConcurrentErrorCommand(clientSocket, commandData);
                            break;
                        }

                        ServerTask<?> controller = (ServerTask<?>)executor.submit(command);
                        storeContoller(command.getUsername(), controller, command);
                    } else {
                        PrintWriter out = new PrintWriter (clientSocket.getOutputStream(),true);
                        out.println(ret);
                        clientSocket.close();
                    }

                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        } catch (InterruptedException e) {
            // No Action Required
        }
    }
```

`storeController()`方法是将`Future`对象存储在双重`ConcurrentHashMap`中的方法：

```java
    private void storeContoller(String userName, ServerTask<?> controller, ConcurrentCommand command) {
        taskController.computeIfAbsent(userName, k -> new ConcurrentHashMap<>()).put(command, controller);
    }
```

最后，我们包含了两个方法来管理`Executor`类的执行，一个是调用`shutdown()`方法来关闭执行器，另一个是等待其完成。请记住，您必须显式调用`shutdown()`或`shutdownNow()`方法来结束执行器的执行。否则，程序将无法终止。请看下面的代码：

```java
    public void shutdown() {

        String message="Request Task: "
                +pendingConnections.size()
                +" pending connections.";
        Logger.sendMessage(message);
        executor.shutdown();
    }

    public void terminate() {
        try {
            executor.awaitTermination(1,TimeUnit.DAYS);
            executor.writeStatistics();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
```

## 客户端部分

现在是测试服务器的时候了。在这种情况下，我们不会太担心执行时间。我们测试的主要目标是检查新功能是否正常工作。

我们将客户端部分分为以下两个类：

+   **ConcurrentClient 类**：这实现了服务器的单个客户端。该类的每个实例都有不同的用户名。它进行了 100 次查询，其中 90 次是查询类型，10 次是报告类型。查询查询的优先级为 5，报告查询的优先级较低（10）。

+   **MultipleConcurrentClient 类**：这测试了多个并发客户端的行为。我们已经测试了具有一到五个并发客户端的服务器。该类还测试了取消和停止命令。

我们已经包含了一个执行器来执行对服务器的并发请求，以增加客户端的并发级别。

在下图中，您可以看到任务取消的结果：

![客户端部分](img/00008.jpeg)

在这种情况下，**USER_2**用户的四个任务已被取消。

以下图片显示了关于每个用户的任务数量和执行时间的最终统计数据：

![客户端部分](img/00009.jpeg)

# 第二个示例 - 执行周期性任务

在之前的执行器示例中，任务只执行一次，并且尽快执行。执行器框架包括其他执行器实现，使我们对任务的执行时间更加灵活。`ScheduledThreadPoolExecutor`类允许我们*周期性*执行任务，并在*延迟*后执行任务。

在本节中，您将学习如何执行周期性任务，实现**RSS 订阅**阅读器。这是一个简单的情况，您需要定期执行相同的任务（阅读 RSS 订阅的新闻）。我们的示例将具有以下特点：

+   将 RSS 源存储在文件中。我们选择了一些重要报纸的世界新闻，如纽约时报、每日新闻或卫报。

+   我们为每个 RSS 源向执行器发送一个`Runnable`对象。每次执行器运行该对象时，它会解析 RSS 源并将其转换为包含 RSS 内容的`CommonInformationItem`对象列表。

+   我们使用**生产者/消费者设计模式**将 RSS 新闻写入磁盘。生产者将是执行器的任务，它们将每个`CommonInformationItem`写入缓冲区。只有新项目将存储在缓冲区中。消费者将是一个独立的线程，它从缓冲区中读取新闻并将其写入磁盘。

+   任务执行结束和下一次执行之间的时间将是一分钟。

我们还实现了示例的高级版本，其中任务执行之间的时间可以变化。

## 共同部分

正如我们之前提到的，我们读取一个 RSS 源并将其转换为对象列表。为了解析 RSS 文件，我们将其视为 XML 文件，并在`RSSDataCapturer`类中实现了一个**SAX**（简单 XML API）解析器。它解析文件并创建一个`CommonInformationItem`列表。这个类为每个 RSS 项存储以下信息：

+   **标题**：RSS 项的标题。

+   **日期**：RSS 项的日期。

+   **链接**：RSS 项的链接。

+   **描述**：RSS 项的文本。

+   **ID**：RSS 项的 ID。如果该项不包含 ID，我们将计算它。

+   **来源**：RSS 来源的名称。

我们使用生产者/消费者设计模式将新闻存储到磁盘中，因此我们需要一个缓冲区来存储新闻和一个`Consumer`类，该类从缓冲区中读取新闻并将其存储到磁盘中。

我们在`NewsBuffer`类中实现了缓冲区。它有两个内部属性：

+   **LinkedBlockingQueue**：这是一个带有阻塞操作的并发数据结构。如果我们想从列表中获取一个项目，而它是空的，调用方法的线程将被阻塞，直到列表中有元素为止。我们将使用这个结构来存储`CommonInformationItems`。

+   **ConcurrentHashMap**：这是`HashMap`的并发实现。我们将使用它来在缓冲区中存储之前存储的新闻项的 ID。

我们只会将以前未插入的新闻插入到缓冲区中：

```java
public class NewsBuffer {
    private LinkedBlockingQueue<CommonInformationItem> buffer;
    private ConcurrentHashMap<String, String> storedItems;

    public NewsBuffer() {
        buffer=new LinkedBlockingQueue<>();
        storedItems=new ConcurrentHashMap<String, String>();
    }
```

在`NewsBuffer`类中有两个方法：一个用于将项目存储在缓冲区中，并检查该项目是否已经插入，另一个用于从缓冲区中获取下一个项目。我们使用`compute()`方法将元素插入`ConcurrentHashMap`中。这个方法接收一个 lambda 表达式作为参数，其中包含与该键关联的实际值（如果键没有关联的值，则为 null）。在我们的情况下，如果该项以前没有被处理过，我们将把该项添加到缓冲区中。我们使用`add()`和`take()`方法来向队列中插入、获取和删除元素：

```java
    public void add (CommonInformationItem item) {
        storedItems.compute(item.getId(), (id, oldSource) -> {
              if(oldSource == null) {
                buffer.add(item);
                return item.getSource();
              } else {
                System.out.println("Item "+item.getId()+" has been processed before");
                return oldSource;
              }
            });
    }

    public CommonInformationItem get() throws InterruptedException {
        return buffer.take();
    }
```

缓冲区的项目将由`NewsWriter`类写入磁盘，该类将作为一个独立的线程执行。它只有一个内部属性，指向应用程序中使用的`NewsBuffer`类：

```java
public class NewsWriter implements Runnable {
    private NewsBuffer buffer;
    public NewsWriter(NewsBuffer buffer) {
        this.buffer=buffer;
    }
```

这个`Runnable`对象的`run()`方法从缓冲区中获取`CommonInformationItem`实例并将它们保存到磁盘中。由于我们使用了阻塞方法`take`，如果缓冲区为空，这个线程将被阻塞，直到缓冲区中有元素为止。

```java
    public void run() {
        try {
            while (!Thread.currentThread().interrupted()) {
                CommonInformationItem item=buffer.get();

                Path path=Paths.get ("output\\"+item.getFileName());

                try (BufferedWriter fileWriter = Files.newBufferedWriter(path, StandardOpenOption.CREATE)) {
                    fileWriter.write(item.toString());
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        } catch (InterruptedException e) {
            //Normal execution
        }
    }
```

## 基本读取器

基本读取器将使用标准的`ScheduledThreadPoolExecutor`类来定期执行任务。我们将为每个 RSS 源执行一个任务，并且在一个任务执行的终止和下一个任务执行的开始之间将有一分钟的时间。这些并发任务在`NewsTask`类中实现。它有三个内部属性来存储 RSS 源的名称、其 URL 和存储新闻的`NewsBuffer`类：

```java
public class NewsTask implements Runnable {
    private String name;
    private String url;
    private NewsBuffer buffer;

    public NewsTask (String name, String url, NewsBuffer buffer) {
        this.name=name;
        this.url=url;
        this.buffer=buffer;
    }
```

这个`Runnable`对象的`run()`方法简单地解析 RSS 源，获取`CommonItemInterface`实例的列表，并将它们存储在缓冲区中。这个方法将定期执行。在每次执行中，`run()`方法将从头到尾执行：

```java
    @Override
    public void run() {
        System.out.println(name+": Running. " + new Date());
        RSSDataCapturer capturer=new RSSDataCapturer(name);
        List<CommonInformationItem> items=capturer.load(url);

        for (CommonInformationItem item: items) {
            buffer.add(item);
        }
    }
```

在这个例子中，我们还实现了另一个线程来实现执行器和任务的初始化以及等待执行的结束。我们将这个类命名为`NewsSystem`。它有三个内部属性，用于存储带有 RSS 源的文件路径，用于存储新闻的缓冲区，以及用于控制其执行结束的`CountDownLatch`对象。`CountDownLatch`类是一种同步机制，允许您使一个线程等待一个事件。我们将在第九章中详细介绍这个类的使用，*深入并发数据结构和同步工具*。我们有以下代码：

```java
public class NewsSystem implements Runnable {
    private String route;
    private ScheduledThreadPoolExecutor executor;
    private NewsBuffer buffer;
    private CountDownLatch latch=new CountDownLatch(1);

    public NewsSystem(String route) {
        this.route = route;
        executor = new ScheduledThreadPoolExecutor (Runtime.getRuntime().availableProcessors());
        buffer=new NewsBuffer();
    }
```

在`run()`方法中，我们读取所有的 RSS 源，为每一个创建一个`NewsTask`类，并将它们发送到我们的`ScheduledThreadPool`执行器。我们使用`Executors`类的`newScheduledThreadPool()`方法创建了执行器，并使用`scheduleAtFixedDelay()`方法将任务发送到执行器。我们还启动了`NewsWriter`实例作为一个线程。`run()`方法等待有人告诉它结束执行，使用`CountDownLatch`类的`await()`方法，并结束`NewsWriter`任务和`ScheduledExecutor`的执行。

```java
    @Override
    public void run() {
        Path file = Paths.get(route);
        NewsWriter newsWriter=new NewsWriter(buffer);
        Thread t=new Thread(newsWriter);
        t.start();

        try (InputStream in = Files.newInputStream(file);
                BufferedReader reader = new BufferedReader(
                        new InputStreamReader(in))) {
            String line = null;
            while ((line = reader.readLine()) != null) {
                String data[] = line.split(";");

                NewsTask task = new NewsTask(data[0], data[1], buffer);
                System.out.println("Task "+task.getName());
                executor.scheduleWithFixedDelay(task,0, 1, TimeUnit.MINUTES);
            }
        }  catch (Exception e) {
            e.printStackTrace();
        }

        synchronized (this) {
            try {
                latch.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println("Shutting down the executor.");
        executor.shutdown();
        t.interrupt();
        System.out.println("The system has finished.");

    }
```

我们还实现了`shutdown()`方法。这个方法将使用`CountDownLatch`类的`countDown()`方法通知`NewsSystem`类结束执行。这个方法将唤醒`run()`方法，因此它将关闭运行`NewsTask`对象的执行器：

```java
    public void shutdown() {
        latch.countDown();
    }
```

这个例子的最后一个类是实现了例子的`main()`方法的`Main`类。它启动了一个`NewsSystem`实例作为一个线程，等待 10 分钟，然后通知线程完成，从而结束整个系统的执行，如下所示：

```java
public class Main {

    public static void main(String[] args) {

        // Creates the System an execute it as a Thread
        NewsSystem system=new NewsSystem("data\\sources.txt");

        Thread t=new Thread(system);

        t.start();

        // Waits 10 minutes
        try {
            TimeUnit.MINUTES.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // Notifies the finalization of the System
         (
        system.shutdown();
    }
```

当您执行这个例子时，您会看到不同的任务是如何周期性地执行的，以及新闻项目是如何写入磁盘的，如下面的截图所示：

![基本阅读器](img/00010.jpeg)

## 高级读者

基本新闻阅读器是`ScheduledThreadPoolExecutor`类的一个使用示例，但我们可以更进一步。与`ThreadPoolExecutor`一样，我们可以实现自己的`ScheduledThreadPoolExecutor`以获得特定的行为。在我们的例子中，我们希望周期性任务的延迟时间根据一天中的时间而变化。在这一部分，您将学习如何实现这种行为。

第一步是实现一个告诉我们周期性任务两次执行之间延迟的类。我们将这个类命名为`Timer`类。它只有一个名为`getPeriod()`的静态方法，它返回一个执行结束和下一个开始之间的毫秒数。这是我们的实现，但您也可以自己制作：

```java
public class Timer {
    public static long getPeriod() {
        Calendar calendar = Calendar.getInstance();
        int hour = calendar.get(Calendar.HOUR_OF_DAY);

        if ((hour >= 6) && (hour <= 8)) {
            return TimeUnit.MILLISECONDS.convert(1, TimeUnit.MINUTES);
        }

        if ((hour >= 13) && (hour <= 14)) {
            return TimeUnit.MILLISECONDS.convert(1, TimeUnit.MINUTES);
        }

        if ((hour >= 20) && (hour <= 22)) {
            return TimeUnit.MILLISECONDS.convert(1, TimeUnit.MINUTES);
        }
        return TimeUnit.MILLISECONDS.convert(2, TimeUnit.MINUTES);
    }
}
```

接下来，我们必须实现执行器的内部任务。当您将一个`Runnable`对象发送到执行器时，从外部来看，您会将这个对象视为并发任务，但执行器会将这个对象转换为另一个任务，即`FutureTask`类的一个实例，其中包括`run()`方法来执行任务以及`Future`接口的方法来管理任务的执行。为了实现这个例子，我们必须实现一个扩展`FutureTask`类的类，并且，由于我们将在**计划执行器**中执行这些任务，它必须实现`RunnableScheduledFuture`接口。这个接口提供了`getDelay()`方法，返回到下一个任务执行的剩余时间。我们在`ExecutorTask`类中实现了这些内部任务。它有四个内部属性：

+   `ScheduledThreadPoolExecutor`类创建的原始`RunnableScheduledFuture`内部任务

+   将执行任务的计划执行器

+   任务的下一次执行的开始日期

+   RSS 订阅的名称

代码如下：

```java
public class ExecutorTask<V> extends FutureTask<V> implements RunnableScheduledFuture<V> {
    private RunnableScheduledFuture<V> task;

    private NewsExecutor executor;

    private long startDate;

    private String name;

    public ExecutorTask(Runnable runnable, V result, RunnableScheduledFuture<V> task, NewsExecutor executor) {
        super(runnable, result);
        this.task = task;
        this.executor = executor;
        this.name=((NewsTask)runnable).getName();
        this.startDate=new Date().getTime();
    }
```

在这个类中，我们重写或实现了不同的方法。首先是`getDelay()`方法，正如我们之前告诉过你的，它返回给定单位时间内任务下一次执行的剩余时间：

```java
    @Override
    public long getDelay(TimeUnit unit) {
        long delay;
        if (!isPeriodic()) {
            delay = task.getDelay(unit);
        } else {
            if (startDate == 0) {
                delay = task.getDelay(unit);
            } else {
                Date now = new Date();
                delay = startDate - now.getTime();
                delay = unit.convert(delay, TimeUnit.MILLISECONDS);
            }

        }

        return delay;
    }
```

接下来的是`compareTo()`方法，它比较两个任务，考虑到任务的下一次执行的开始日期：

```java
    @Override
    public int compareTo(Delayed object) {
        return Long.compare(this.getStartDate(), ((ExecutorTask<V>)object).getStartDate());
    }
```

然后，`isPeriodic()`方法返回`true`如果任务是周期性的，如果不是则返回`false`：

```java
    @Override
    public boolean isPeriodic() {
        return task.isPeriodic();
    }
```

最后，我们有`run()`方法，它实现了这个示例的最重要部分。首先，我们调用`FutureTask`类的`runAndReset()`方法。这个方法执行任务并重置它的状态，这样它就可以再次执行。然后，我们使用`Timer`类计算下一次执行的开始日期，最后，我们必须再次将任务插入`ScheduledThreadPoolExecutor`类的队列中。如果不执行这最后一步，任务将不会再次执行，如下所示：

```java
    @Override
    public void run() {
        if (isPeriodic() && (!executor.isShutdown())) {
            super.runAndReset();
            Date now=new Date();
            startDate=now.getTime()+Timer.getPeriod();
            executor.getQueue().add(this);
            System.out.println("Start Date: "+new Date(startDate));
        }
    }
```

一旦我们有了执行器的任务，我们就必须实现执行器。我们实现了`NewsExecutor`类，它扩展了`ScheduledThreadPoolExecutor`类。我们重写了`decorateTask()`方法。通过这个方法，你可以替换调度执行器使用的内部任务。默认情况下，它返回`RunnableScheduledFuture`接口的默认实现，但在我们的情况下，它将返回`ExecutorClass`实例的一个实例：

```java
public class NewsExecutor extends ScheduledThreadPoolExecutor { 
    public NewsExecutor(int corePoolSize) {
        super(corePoolSize);
    }

    @Override
    protected <V> RunnableScheduledFuture<V> decorateTask(Runnable runnable,
            RunnableScheduledFuture<V> task) {
        ExecutorTask<V> myTask = new ExecutorTask<>(runnable, null, task, this);
        return myTask;
    }
}
```

我们必须实现`NewsSystem`和`Main`类的其他版本来使用`NewsExecutor`。我们为此目的实现了`NewsAdvancedSystem`和`AdvancedMain`。

现在你可以运行高级新闻系统，看看执行之间的延迟时间如何改变。

# 有关执行器的附加信息

在本章中，我们扩展了`ThreadPoolExecutor`和`ScheduledThreadPoolExecutor`类，并重写了它们的一些方法。但是，如果需要更特定的行为，你可以重写更多的方法。以下是一些你可以重写的方法：

+   `shutdown()`: 你必须显式调用这个方法来结束执行器的执行。你可以重写它来添加一些代码，以释放你自己的执行器使用的额外资源。

+   `shutdownNow()`: `shutdown()`方法和`shutdownNow()`方法的区别在于，`shutdown()`方法等待所有等待在执行器中的任务的最终处理。

+   `submit()`, `invokeall()`, 或 `invokeany()`: 你可以调用这些方法将并发任务发送到执行器中。如果需要在任务插入执行器的任务队列之前或之后执行一些操作，可以重写它们。请注意，在任务入队之前或之后添加自定义操作与在任务执行之前或之后添加自定义操作是不同的，我们在重写`beforeExecute()`和`afterExecute()`方法时已经做过。

在新闻阅读器示例中，我们使用`scheduleWithFixedDelay()`方法将任务发送到执行器。但是`ScheduledThreadPoolExecutor`类还有其他方法来执行周期性任务或延迟任务：

+   `schedule()`: 这个方法在给定的延迟之后执行一次任务。

+   `scheduleAtFixedRate()`: 这个方法以给定的周期执行周期性任务。与`scheduleWithFixedDelay()`方法的区别在于，在后者中，两次执行之间的延迟从第一次执行结束到第二次执行开始，而在前者中，两次执行之间的延迟在两次执行的开始之间。

# 总结

在本章中，我们介绍了两个示例，探讨了执行器的高级特性。在第一个示例中，我们延续了第二章中的客户端/服务器示例，*管理大量线程 - 执行器*。我们实现了自己的执行器，扩展了`ThreadPoolExecutor`类，以按优先级执行任务，并测量每个用户任务的执行时间。我们还包括了一个新的命令，允许取消任务。

在第二个示例中，我们解释了如何使用`ScheduledThreadPoolExecutor`类来执行周期性任务。我们实现了两个版本的新闻阅读器。第一个版本展示了如何使用`ScheduledExecutorService`的基本功能，第二个版本展示了如何覆盖`ScheduledExecutorService`类的行为，例如，更改任务两次执行之间的延迟时间。

在下一章中，您将学习如何执行返回结果的`Executor`任务。如果您扩展`Thread`类或实现`Runnable`接口，`run()`方法不会返回任何结果，但执行器框架包括`Callable`接口，允许您实现返回结果的任务。
