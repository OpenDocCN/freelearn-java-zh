

# 第三章：掌握 Java 中的并行性

开始一段激动人心的旅程，深入 Java 并行编程的核心，这是一个利用多个线程的合力将复杂、耗时任务转化为高效、流畅操作的区域。

想象一下：一群忙碌厨房中的厨师或一个乐队中的音乐家，每个人都在创造和谐杰作中扮演着至关重要的角色。在本章中，我们深入探讨 Fork/Join 框架，它是线程艺术中的指挥家，巧妙地编排众多线程以无缝协作。

随着我们探索并行编程的复杂性，你会发现它在提升速度和效率方面的显著优势，就像一个协调良好的团队能够实现比各部分总和更多的成果。然而，权力越大，责任越大。你将遇到独特的挑战，例如线程竞争和竞态条件，我们将为你提供克服这些障碍所需的战略和洞察力。

本章不仅是一次探索，更是一个工具箱。你将学习如何有效地使用 Fork/Join 框架，将艰巨的任务分解成可管理的子任务，就像主厨将复杂菜谱的各个部分委派给助手一样。我们将深入研究`RecursiveTask`和`RecursiveAction`的细微差别，了解这些元素如何协同工作以优化并行处理。此外，你还将获得性能优化技术和最佳实践的见解，确保你的 Java 应用程序不仅功能齐全，而且像一台运转良好的机器一样在顶峰表现。

到本章结束时，你将不仅拥有知识，还将具备在 Java 应用程序中有效实施并行编程的实用技能。你将准备好增强功能、优化性能，并直面并发计算带来的挑战。

那么，让我们开始这段激动人心的冒险，进入 Java 并行能力的动态世界。我们将一起打开高效、并发计算的大门，为你搭建一个能够制作出在现代计算世界中脱颖而出的高性能应用程序的舞台。

# 技术要求

你将需要**Visual Studio Code**（**VS Code**），你可以从这里下载：[`code.visualstudio.com/download`](https://code.visualstudio.com/download)。

VS Code 提供了比其他可用选项更轻量级和可定制的替代方案。它是那些更喜欢资源消耗较少的**集成开发环境**（**IDE**）并希望安装针对其特定需求的扩展的开发者的绝佳选择。然而，与更成熟的 Java IDE 相比，它可能没有所有开箱即用的功能。

此外，本章中的代码可以在 GitHub 上找到：

[`github.com/PacktPublishing/Java-Concurrency-and-Parallelism`](https://github.com/PacktPublishing/Java-Concurrency-and-Parallelism)

# 解放并行动力之源——Fork/Join 框架

**Fork/Join 框架**释放了并行处理的力量，将你的 Java 任务转变为协作线程的交响乐。深入探索其秘密，如工作窃取算法、递归征服和优化策略，以提升性能，让顺序烹饪成为过去式！

## 揭秘 Fork/Join – 并行编程中的烹饪冒险

想象一下踏入 Java 并行计算的宏伟厨房。这就是 Fork/Join 框架发挥作用的地方，它将编程艺术转变为一个充满熟练厨师的繁忙厨房。这不仅仅是增加更多的厨师；这是关于用技巧和策略来编排他们。

在这个繁忙的厨房中心，是 Fork/Join 框架，这是 Java 工具箱中的大师级工具，它自动将复杂任务分解成更小、更易于管理的部分。想象一位主厨将复杂的食谱分解成更简单的任务，并委托给副厨师。每位厨师专注于餐点的一部分，确保没有人无所事事，也没有任务压倒性。这种效率类似于框架的秘密成分——工作窃取算法，其中提前完成的厨师会帮助仍在忙碌的厨师，确保烹饪过程和谐高效。

在这个烹饪管弦乐队中，`ForkJoinPool` 扮演着一位熟练的指挥。这是一个专门针对 Fork/Join 任务的线程池，它扩展了在 *第二章*，*Java 并发基础：线程、进程及其他* 中引入的 `Executor` 和 `ExecutorService` 接口。`Executor` 接口提供了一种将任务提交与每个任务如何运行的具体机制解耦的方法，包括线程使用、调度等细节。`ExecutorService` 接口通过生命周期管理和跟踪一个或多个异步任务进度的方法来补充这一点。

基于这些基础构建的 `ForkJoinPool`，旨在处理可以递归分解成更小部分的工作。它采用了一种称为工作窃取的技术，其中空闲线程可以从其他忙碌线程中“窃取”工作，从而最小化空闲时间并最大化 CPU 利用率。

就像管理得当的厨房一样，`ForkJoinPool` 管理任务的执行，将它们分解成子食谱，并确保没有厨师——或线程——是空闲的。当任务完成时，就像副厨师展示他们的菜肴一样，`ForkJoinPool` 精湛地将这些个别努力结合起来，完成最终的杰作。这种分解任务和组合结果的过程是 Fork/Join 模型的基础，使 `ForkJoinPool` 成为并发工具箱中的必备工具。

Fork/Join 框架围绕 `ForkJoinTask` 抽象类展开，它代表一个可以分解成更小的子任务并在 `ForkJoinPool` 中并行执行的任务。它提供了用于分解任务（fork）、等待子任务完成（join）以及计算结果的函数。

`ForkJoinTask` 的两个具体实现是 `RecursiveTask`，用于返回结果的任务，而 `RecursiveAction` 用于不返回值的任务。

两者都允许你将任务分解成更小的部分以并行执行。你需要实现计算方法来定义基本情况以及将任务分解成子任务的逻辑。框架负责在 `ForkJoinPool` 中的线程间分配子任务以及聚合结果。

`RecursiveTask` 和 `RecursiveAction` 之间的关键区别在于它们的目的和返回类型。`RecursiveTask` 用于计算并返回一个结果，而 `RecursiveAction` 则执行一个操作而不返回任何值。

为了说明 `RecursiveTask` 和 `RecursiveAction` 在 Fork/Join 框架中的使用，考虑以下代码示例。`SumTask` 展示了如何对数据数组求和，而 `ActionTask` 展示了如何处理数据而不返回结果：

```java
import java.util.concurrent.RecursiveTask;
import java.util.concurrent.RecursiveAction;
import java.util.ArrayList;
import java.util.concurrent.ForkJoinPool;
public class DataProcessor{
    public static void main(String[] args) {
        // Example dataset
        int DATASET_SIZE = 500;
        ArrayList<Integer> data = new ArrayList<Integer> (
DATASET_SIZE);
        ForkJoinPool pool = new ForkJoinPool();
        // RecursiveAction for generating large dataset
        ActionTask actionTask = new ActionTask(data, 0, DATASET_SIZE);
        pool.invoke(actionTask);
        // RecursiveTask for summing large dataset
        SumTask sumTask = new SumTask(data,0,DATASET_SIZE);
        int result = pool.invoke(sumTask);
        System.out.println("Total sum: " + result);
        pool.shutdown();
        pool.close();
    }
// Splitting task for parallel execution
    static class SumTask extends RecursiveTask<Integer> {
        private final ArrayList<Integer> data;
        private final int start, end;
        private static final int THRESHOLD = 50;
        SumTask(ArrayList<Integer> data,int start,int end){
            this.data = data;
            this.start = start;
            this.end = end;
        }
        @Override
        protected Integer compute() {
            int length = end - start;
            System.out.println(String.format("RecursiveTask.compute()             called for %d elements from index %d to %d", length,             start, end));
            if (length <= THRESHOLD) {
                // Simple computation
                System.out.println(String.format("Calculating sum of                 %d elements from index %d to %d", length, start,                 end));
                int sum = 0;
                for (int i = start; i < end; i++) {
                    sum += data.get(i);
                }
                return sum;
            } else {
                // Split task
                int mid = start + (length / 2);
                SumTask left = new SumTask(data,start,mid);
                SumTask right = new SumTask(data,mid,end);
                left.fork();
                right.fork();
                return right.join() + left.join();
            }
        }
    }
    static class ActionTask extends RecursiveAction {
        private final ArrayList<Integer> data;
        private final int start, end;
        private static final int THRESHOLD = 50;
        ActionTask(ArrayList<Integer> data,int start,
            int end){
                this.data = data;
                this.start = start;
                this.end = end;
            }
        @Override
        protected void compute() {
            int length = end - start;
            System.out.println(String.format("RecursiveAction.            compute() called for %d elements from index %d to %d",             length, start, end));
            if (length <= THRESHOLD) {
                // Simple processing
                for (int i = start; i < end; i++) {
                    this.data.add((int) Math.round(
                        Math.random() * 100));
                }
            } else {
                // Split task
                int mid = start + (length / 2);
                ActionTask left = new ActionTask(data,
                    start, mid);
                ActionTask right = new ActionTask(data,
                    mid, end);
                invokeAll(left, right);
            }
        }
    }
}
```

下面是对代码及其功能的分解：

+   `SumTask` 扩展了 `RecursiveTask<Integer>` 并用于计算数组的一部分，返回总和。

+   在 `SumTask` 类中，当数据长度超过阈值时，任务会被分解，展示了分而治之的方法。这类似于主厨将大型食谱任务分配给副厨师。

+   `ActionTask` 扩展了 `RecursiveAction` 并用于处理数组的一部分而不返回结果。

+   `fork()` 方法启动子任务的并行执行，而 `join()` 等待这些任务的完成，合并它们的结果。`compute()` 方法包含直接执行任务或进一步分解它的逻辑。

+   当数据集大小超过阈值时，这两个类都会分解任务，展示了分而治之的方法。

+   `ForkJoinPool` 执行这两个任务，说明了 `RecursiveTask` 和 `RecursiveAction` 如何在并行处理场景中使用。

此示例展示了 Fork/Join 框架在并行处理大型数据集方面的实际应用，正如之前所讨论的。它们展示了复杂任务如何被分解并并行执行以提升应用性能。想象一下使用 `SumTask` 快速处理大型金融数据集或使用 `ActionTask` 在实时分析应用的数据清洗操作中进行并行处理。

在下一节中，我们将探讨如何处理具有依赖关系的任务，并导航复杂任务图的复杂性。

## 超越递归——利用依赖关系克服复杂性

我们已经见证了递归任务在解决较小、独立挑战时的美妙之处。但在现实场景中，任务具有复杂依赖关系，如多道菜式中一道菜依赖于另一道菜完成的情况，又该如何呢？这正是`ForkJoinPool.invokeAll()`大放异彩的地方，它是协调具有复杂关系的并行任务的有力工具。

## ForkJoinPool.invokeAll() –错综复杂任务的指挥家

想象一个熙熙攘攘的厨房，厨师们正在制作各种菜肴。有些任务，如切菜，可以独立完成。但其他任务，如制作酱料，则依赖于已经准备好的食材。这就是主厨`ForkJoinPool`介入的地方。通过`invokeAll()`，他们分配任务，确保依赖任务在开始之前等待其前驱任务完成。

## 在厨房交响乐中管理依赖关系——效率的秘方

正如厨师精心协调不同烹饪时间的菜肴一样，并行处理需要细致地管理任务依赖关系。让我们通过厨房的视角来探讨这种艺术，我们的目标是高效地准备多道菜式。

以下是一些并行处理的关键策略：

+   **任务分解**：将工作流程分解成更小、更易于管理的任务，并确保有明确的依赖关系。在我们的厨房交响乐中，我们将创建准备蔬菜、制作酱料和烹饪蛋白质的任务，每个任务都有其自身的先决条件。

+   **依赖分析**：识别任务依赖并定义执行顺序。如烹饪蛋白质这样的任务必须等待准备好的蔬菜和酱料，以确保菜肴的完美协调。

+   **粒度控制**：选择适当的任务大小以平衡效率和开销。过多的细粒度任务会增加管理开销，而大任务可能会限制并行性。

+   **数据共享和同步**：确保共享数据的正确访问和同步，以避免不一致性。如果多个厨师使用共享的食材，我们需要一个系统来避免冲突并保持厨房的和谐。

让我们通过`PrepVeggiesTask`类来可视化依赖管理：

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;
public class PrepVeggiesDemo {
    static interface KitchenTask {
        int getTaskId();
        String performTask();
    }
    static class PrepVeggiesTask implements KitchenTask {
        protected int taskId;
        public PrepVeggiesTask(int taskId) {
            this.taskId = taskId;
        }
        public String performTask() {
            String message = String.format(
                "[Task-%d] Prepped Veggies", this.taskId);
            System.out.println(message);
            return message;
        }
        public int getTaskId() {return this.taskId; }
    }
    static class CookVeggiesTask implements KitchenTask {
        protected int taskId;
        public CookVeggiesTask(int taskId) {
            this.taskId = taskId;
        }
        public String performTask() {
            String message = String.format(
                "[Task-%d] Cooked Veggies", this.taskId);
            System.out.println(message);
            return message;
        }
        public int getTaskId() {return this.taskId; }
    }
    static class ChefTask extends RecursiveTask<String> {
        protected KitchenTask task;
        protected List<ChefTask> dependencies;
        public ChefTask(
            KitchenTask task,List<ChefTask> dependencies) {
                this.task = task;
                this.dependencies = dependencies;
            }
        // Method to wait for dependencies to complete
        protected void awaitDependencies() {
            if (dependencies == null || dependencies.isEmpty())             return;
            ChefTask.invokeAll(dependencies);
        }
        @Override
        protected String compute() {
            awaitDependencies(); // Ensure all prerequisites are met
            return task.performTask(); // Carry out the specific task
        }
    }
    public static void main(String[] args) {
        // Example dataset
        int DEPENDENCY_SIZE = 10;
        ArrayList<ChefTask> dependencies = new ArrayList<ChefTask>();
        for (int i = 0; i < DEPENDENCY_SIZE; i++) {
            dependencies.add(new ChefTask(
                new PrepVeggiesTask(i), null));
        }
        ForkJoinPool pool = new ForkJoinPool();
        ChefTask cookTask = new ChefTask(
            new CookVeggiesTask(100), dependencies);
        pool.invoke(cookTask);
        pool.shutdown();
        pool.close();
    }
}
```

提供的代码展示了在 Java 中使用 Fork/Join 框架处理具有依赖关系的任务的用法。它定义了两个接口：`KitchenTask`用于通用任务和`ChefTask`用于返回 String 结果的任务。

下面是一些关键点：

+   `PrepVeggiesTask`和`CookVeggiesTask`实现了`KitchenTask`，代表厨房中的具体任务。`ChefTask`类是 Fork/Join 实现的核心，包含实际任务（`task`）及其依赖（`dependencies`）。

+   `awaitDependencies()`方法等待所有依赖完成后再执行当前任务。`compute()`方法是 Fork/Join 框架的主要入口点，确保满足先决条件并执行实际任务。

+   在主方法中，使用 `PrepVeggiesTask` 对象作为依赖项创建了一个示例数据集。使用 `ForkJoinPool` 来管理任务的执行。通过 `pool.invoke(cookTask)` 将具有依赖关系的 `CookVeggiesTask` 提交到池中，触发任务的执行及其依赖项。

+   `ChefTask` 作为具有依赖关系的任务的蓝图。

+   `awaitDependencies()` 等待先决条件完成。

+   `PrepVeggiesTask` 和 `CookVeggiesTask` 代表特定的任务。

+   `performTask()` 包含实际的任务逻辑。

代码演示了如何使用 Fork/Join 框架来处理具有依赖关系的任务，确保在执行任务之前完成先决条件。`ForkJoinPool` 管理任务的执行，而 `ChefTask` 类提供了一种结构化的方式来定义和执行具有依赖关系的任务。

让我们结合一个现实世界的场景，以巩固并行处理中依赖关系管理概念。

想象一下：你正在构建一个下一代图像渲染应用程序，该应用程序需要处理复杂的 3D 场景。为了有效地管理工作负载，你将渲染过程分解为以下并行任务：

+   **任务 1**：下载纹理和模型数据

+   **任务 2**：从下载的数据中构建几何原语。

+   **任务 3**：对场景应用光照和阴影

+   **任务 4**：渲染最终图像

正是这里，依赖关系开始发挥作用：

+   任务 2 在开始之前必须等待任务 1 下载必要的数据完成。

+   任务 3 在应用光照和阴影之前需要任务 2 构建的几何原语。

+   最后，任务 4 依赖于任务 3 完成的场景来生成最终图像。

通过精心管理这些依赖关系并利用并行处理技术，你可以显著加快渲染过程，提供流畅且视觉上令人惊叹的 3D 体验。

这个现实世界的例子展示了依赖关系管理对于在各个领域（从图像渲染到科学模拟等）发挥并行处理真正力量的关键作用。

记住，就像指挥厨房交响乐或渲染复杂的 3D 场景一样，掌握并行处理在于细致的计划、执行和高效的依赖关系管理。有了正确的工具和技术，你可以将并行处理努力转变为和谐且高性能的任务交响乐。

现在，让我们继续探讨在性能优化技术主题中调整这些交响乐的艺术。

# 调整并行交响乐——性能优化的旅程

在并行编程的动态世界中，实现最佳性能就像指挥一个大型管弦乐队。每个元素都扮演着至关重要的角色，而精细调整它们对于创作和谐的交响乐是必不可少的。让我们开始一段旅程，了解 Java 并行计算中性能优化的关键策略。

## 粒度控制的技艺

正如厨师平衡配料以制作完美佳肴一样，在并行编程中的粒度控制是关于找到理想的任务大小。较小的任务，就像有更多的厨师，可以增强并行处理，但会引入依赖和管理开销。相反，较大的任务简化了管理，但限制了并行处理，就像少数厨师处理一切。关键是评估任务复杂性，权衡开销与收益，并避免过于精细的任务，这些任务可能会使流程变得混乱。

## 调整并行级别

设置正确的并行级别就像指挥我们的厨师，确保每个人都有适量的工作——既不过于忙碌也不过于闲散。这是在利用可用资源与避免过多活跃线程带来的过度开销之间的一种微妙平衡。考虑你任务的特性以及可用的硬件。记住，较大的线程池可能并不总是像较小的、更专注的群体那样高效地受益于工作窃取。

## 稳定性能的最佳实践

在我们的并行厨房中，最佳实践是成功的秘诀。限制线程间的数据共享可以防止对共享资源的冲突，就像厨师在各自的岗位上工作一样。选择智能的、线程安全的如`ConcurrentHashMap`这样的数据结构可以确保对共享数据的安全访问。定期监控性能并准备好调整任务大小和线程数量可以使你的并行应用程序运行得更加顺畅和高效。

通过掌握这些技术——粒度控制、调整并行级别以及遵循最佳实践——我们可以将我们的并行计算提升到新的效率和性能高度。这不仅仅是运行并行任务；这是关于精确和深入地指挥它们，确保每个线程在这个复杂的并行处理交响乐中扮演其角色。

性能优化是高效并行处理的基础。现在，我们进入了一个由 Java 的并行流带来的精致优雅的世界，通过并发执行实现闪电般的数据处理。

## 在 Java 中使用并行流简化并行处理

微调并行处理的交响乐就像指挥一场大型管弦乐。每个元素都扮演着至关重要的角色，掌握它们可以解锁峰值性能。通过关键策略，如粒度控制和并行级别，这一旅程确保了 Java 并行计算中的和谐执行。

现在，我们进入了一个由 Java 的并行流带来的精致优雅的世界。想象一下，将一个只有一个厨师的单人厨房转变为一个同步的团队，利用多个核心进行闪电般的数据处理。记住，高效的并行处理在于选择正确的任务。

并行流之所以出色，有以下原因：

+   **更快地执行**：特别是对于大数据集，它们显著加速数据操作

+   **处理大量数据**：它们的优势在于高效地处理大量数据

+   **易于使用**：从顺序流切换到并行流通常很简单

然而，考虑以下挑战：

+   **额外资源管理**：线程管理会产生开销，使得小任务不太理想

+   **任务独立性**：当任务独立且没有顺序依赖时，并行流表现得尤为出色

+   **小心共享数据**：对共享数据的并发访问需要谨慎同步，以避免竞态条件

让我们了解如何无缝集成并行流，以利用其性能优势同时应对潜在挑战：

+   **识别合适的任务**：首先，确定代码中计算成本高昂的操作，这些操作独立于数据元素，例如图像调整大小、排序大型列表或执行复杂计算。这些任务是并行化的理想候选。

+   使用 `parallelStream()` 方法而不是 `stream()`。这种微妙的变化释放了多核处理的能力。

    例如，考虑这样一个场景，你需要调整大量照片的大小。顺序方法 `photos.stream().map(photo -> resize(photo))` 会逐个处理每张照片。通过切换到 `photos.parallelStream().map(photo -> resize(photo))`，你释放了多核的潜力，它们协同工作同时调整照片大小，通常能带来显著的性能提升。

记住，有效的并行流集成需要仔细考虑任务适用性、资源管理和数据安全，以确保最佳结果并避免潜在陷阱。

接下来，我们将进行对比分析，探讨不同的并行处理工具，帮助你为你的编程交响乐选择完美的乐器。

## 选择你的武器——Java 中的并行处理对决

精通 Fork/Join 框架本身就是一项烹饪壮举，但在这个更广泛的 Java 并行处理工具领域中导航才是真正展现专长的领域。为了帮助你为你的并行处理菜肴选择完美的配料，让我们探讨 Fork/Join 与其他选项如何相提并论：

+   另一方面，`ThreadPoolExecutor` 是一个更通用的厨房经理，处理大量独立且不可分割的任务，例如为宴会准备单独的菜肴。它非常适合简单的并行需求，其中副厨师不需要进一步分解他们的配料。

+   **Fork/Join 与并行流**：并行流就像预先清洗和切好的蔬菜，可以直接放入处理锅中。它们通过在幕后自动并行化操作来简化集合上的数据处理，将 Fork/Join 作为其秘密武器。对于简单的数据处理，它们是一个快速方便的选择。然而，对于具有自定义处理逻辑的复杂任务，Fork/Join 提供了像经验丰富的厨师一样的精细控制和灵活性，允许你为最佳结果定制食谱。

+   `CompletableFuture`就像一个多任务处理的副厨师，擅长处理异步操作。它允许你编写非阻塞代码并将多个异步任务链接在一起，确保厨房即使在其他菜肴慢炖时也能平稳运行。想象一下，你可以在不耽误主菜的情况下准备多个配菜。

+   `Executors.newCachedThreadPool()`就像雇佣临时厨师，他们可以根据需要随时加入或退出。这对于短暂、异步的任务，如取食材，非常合适。然而，对于长时间运行、CPU 密集型任务，Fork/Join 的工作窃取算法再次大放异彩，确保每位厨师都处于最佳忙碌状态，在整个烹饪过程中最大化效率。

通过理解每个工具的优点和缺点，你可以为你的并行处理需求选择最合适的一个。记住，Fork/Join 是大规模可并行任务的专家，而其他工具则满足特定的需求，如独立作业、简单的数据处理、异步工作流，甚至是临时协助。

在探讨了 Fork/Join 框架与其他 Java 并行处理方法的比较分析之后，我们现在转向一个更专业的话题。接下来，我们将深入探讨如何使用自定义 Spliterator 释放大数据的威力，我们将揭示优化并行流处理的先进技术，重点关注自定义 Spliterator 实现和计算开销的有效管理。

# 使用自定义 Spliterator 释放大数据的威力

Java 的**可分割迭代器**（**Spliterator**）接口为将数据分割成更小的块以进行并行处理提供了强大的工具。但对于大型数据集，例如在云平台如**亚马逊网络服务**（**AWS**）上找到的数据集，一个自定义 Spliterator 可以成为游戏规则的改变者。

例如，想象一下 AWS **简单存储服务**（**S3**）中一个巨大的文件桶。为这项任务专门设计的自定义 Spliterator 可以智能地将数据分割成最佳大小，考虑因素包括文件类型和访问模式。这允许你更有效地将任务分配到 CPU 核心，从而显著提高性能并减少资源利用率。

现在，假设你有一个 AWS S3 存储桶中有很多文件，并想同时使用 Java Streams 处理它们。以下是如何为这些 AWS S3 对象设置自定义 Spliterator 的方法：

```java
// Assume s3Client is an initialized AmazonS3 client
public class S3CustomSpliteratorExample {
    public static void main(String[] args) {
        String bucketName = "your-bucket-name";
        ListObjectsV2Result result = s3Client.        listObjectsV2(bucketName);
        List<S3ObjectSummary> objects = result.getObjectSummaries();
        Spliterator<S3ObjectSummary> spliterator = new         S3ObjectSpliterator(objects);
        StreamSupport.stream(spliterator, true)
                .forEach(S3CustomSpliteratorExample::processS3Object);
    }
    private static class S3ObjectSpliterator implements     Spliterator<S3ObjectSummary> {
        private final List<S3ObjectSummary> s3Objects;
        private int current = 0;
        S3ObjectSpliterator(List<S3ObjectSummary> s3Objects) {
            this.s3Objects = s3Objects;
        }
        @Override
        public boolean tryAdvance(Consumer<? super S3ObjectSummary>         action) {
            if (current < s3Objects.size()) {
                action.accept(s3Objects.get(current++));
                return true;
            }
            return false;
        }
        @Override
        public Spliterator<S3ObjectSummary> trySplit() {
            int remaining = s3Objects.size() - current;
            int splitSize = remaining / 2;
            if (splitSize <= 1) {
                return null;
            }
            List<S3ObjectSummary> splitPart = s3Objects.            subList(current, current + splitSize);
            current += splitSize;
            return new S3ObjectSpliterator(splitPart);
        }
        @Override
        public long estimateSize() {
            return s3Objects.size() - current;
        }
        @Override
        public int characteristics() {
            return IMMUTABLE | SIZED | SUBSIZED;
        }
    }
    private static void processS3Object(S3ObjectSummary objectSummary) {
        // Processing logic for each S3 object
    }
}
```

展示的 Java 代码展示了如何利用自定义 Spliterator 实现 S3 对象的效率并行处理。让我们深入了解其关键元素：

1.  `S3ObjectSpliterator`用于将列表分割以进行并行处理

1.  使用 Spliterator 启动并行流，对每个对象应用`processS3Object`方法

1.  `S3ObjectSpliterator`类实现了`Spliterator<S3ObjectSummary>`接口，使得针对并行流的定制数据分割成为可能。其他关键方法如下：

    +   `tryAdvance`：处理当前对象并移动游标。

    +   `trySplit`：将列表分割成更小的块以进行并行执行，并为分割的部分返回一个新的 Spliterator。

    +   `estimateSize`：提供剩余对象的估计，有助于流优化。

    +   `characteristics`：指定 Spliterator 特性（`IMMUTABLE`、`SIZED`或`SUBSIZED`），以实现高效的流操作。

1.  `processS3Object`方法封装了在每个 S3 对象上执行的具体处理步骤。实现细节未显示，但此方法可能涉及下载对象内容、应用转换或提取元数据等任务。

以下为自定义 Spliterator 方法的优势：

+   **细粒度控制**：自定义 Spliterator 允许对数据分割进行精确控制，根据任务需求和硬件能力，实现并行处理的最优块大小。

+   `trySplit`方法有效地将工作负载分割给多核处理器，从而可能导致性能提升。

+   **灵活处理多种数据类型**：自定义 Spliterator 可以适应处理不同的 S3 对象类型或访问模式，针对特定用例定制处理策略。

从本质上讲，这段代码展示了自定义 Spliterator 如何使 Java 开发者能够控制 S3 对象的并行处理，解锁云环境中各种数据密集型任务的增强性能和灵活性。

除了自定义 Spliterator 之外，Java 还提供了一系列高级技术，用于微调流并行性和解锁卓越的性能。让我们通过一个代码示例来看看三种强大的策略：自定义线程池、组合流操作和并行友好型数据结构。

让我们通过以下代码来探索这些 Java 类：

```java
import java.util.List;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ConcurrentHashMap;
import java.util.stream.Collectors;
public class StreamOptimizationDemo {
    public static void main(String[] args) {
        // Example data
        List<Integer> data = List.of(
            1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        // Custom Thread Pool for parallel streams
        ForkJoinPool customThreadPool = new ForkJoinPool(4);         // Customizing the number of threads
        try {
            List<Integer> processedData = customThreadPool.submit(()             ->
                data.parallelStream()
                    .filter(n -> n % 2 == 0)
// Filtering even numbers
                    .map(n -> n * n) // Squaring them
                    .collect(Collectors.toList())
// Collecting results
            ).get();
            System.out.println(
                "Processed Data: " + processedData);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            customThreadPool.shutdown();
// Always shutdown your thread pool!
        }
// Using ConcurrentHashMap for better performance in parallel streams
        ConcurrentHashMap<Integer, Integer> map = new         ConcurrentHashMap<>();
        data.parallelStream().forEach(n -> map.put(
            n, n * n));
        System.out.println("ConcurrentHashMap contents: " + map);
    }
}
```

在此代码中，我们使用了以下技术：

+   `ForkJoinPool`具有指定的线程数（在本例中为`4`）。这个自定义线程池用于执行我们的并行流，比使用通用池提供了更好的资源分配。

+   `filter`（选择偶数）和`map`（平方数字）流操作被组合成一个单一的流管道。这减少了数据迭代次数。

+   `ConcurrentHashMap`用于存储并行流操作的结果。这种数据结构旨在支持并发访问，因此是并行流使用的良好选择。

这个类展示了如何结合这些高级技术，在 Java 中实现更高效和优化的并行流处理。

自定义 Spliterator 为并行处理提供了一个强大的配方，但它是否总是最美味的菜肴？在下一节中，我们将加入一些现实检查，探讨并行化的潜在优势和隐藏成本。

# 并行化的优势和陷阱

并行处理不仅提供了显著的速度优势，还伴随着诸如线程竞争和数据依赖问题等挑战。本节重点在于理解何时有效地使用并行处理。它概述了好处和潜在问题，提供了在并行和顺序处理之间选择时的指导。

并行处理在以下关键场景中优于顺序方法：

+   **计算密集型任务**：想象一下处理数字、处理图像或分析大量数据集。这些都是并行处理的游乐场。

+   **独立操作**：当任务独立时，并行性蓬勃发展，这意味着它们不依赖于彼此的结果。想想在列表中过滤项目或调整多个图像的大小。每个操作都可以由一个单独的线程并发处理，从而提高效率，而不会造成复杂的依赖关系。

+   **输入/输出（I/O）绑定操作**：等待从磁盘或网络获取数据的任务非常适合并行处理。当一条线程等待数据时，其他线程可以处理其他独立任务，最大化资源利用，并保持你的代码流畅运行。

+   **实时应用**：无论是渲染动态视觉效果还是处理用户交互，响应性在实时应用中至关重要。并行处理可以是你的秘密武器，通过分割工作负载并保持**用户界面**（**UI**）即使在重负载下也能保持响应，确保流畅、无延迟的体验。

除了这些特定场景之外，并行处理可能带来的性能提升是巨大的。从加速视频编码到支持实时模拟，它释放多个核心的能力可以显著提高应用程序的效率和响应速度。

我们见证了并行处理的激动人心的潜力，但现在有一个关键问题：它有多快？我们如何量化并行处理的性能提升？

测量并行处理效率的最常见指标是加速比。它简单地比较了顺序执行任务的时间和并行执行时间。公式很简单：

*加速比 = 顺序执行时间 / 并行* *执行时间*

加速比为`2`意味着并行版本的时间是顺序版本的一半。

然而，并行处理不仅仅是关于原始速度；它还关乎资源利用率和效率。以下是一些需要考虑的额外指标：

+   **效率**：并行程序占用的 CPU 时间百分比。理想情况下，你希望看到效率接近 100%，这表明所有核心都在努力工作。

+   **Amdahl 定律**：Gene Amdahl 在 20 世纪 60 年代提出的原则，它为并行处理设定了限制。Amdahl 定律指出，增加处理器不会神奇地加快一切。首先关注瓶颈，然后明智地并行化。为什么？仅加速任务的一部分只有在其余部分也很快的情况下才有帮助。因此，随着任务变得更加并行，增加更多处理器带来的好处越来越少。首先优化最慢的部分！即使是高度并行的任务也有*不可并行化部分*，这限制了整体速度的提升。

+   **可伸缩性**：随着核心数量的增加，并行程序的性能如何？理想情况下，我们希望看到随着核心数量的增加，速度几乎线性提升。

这里是一些在云环境和 Java 框架中进行性能调优的知名工具：

+   **分析器**：识别代码中的热点和瓶颈：

    +   **云**：

        +   **Amazon CodeGuru Profiler**：在 AWS 环境中识别性能瓶颈和优化机会

        +   **Azure Application Insights**：为在 Azure 中运行的.NET 应用程序提供分析洞察

        +   **Google Cloud Profiler**：分析在**Google Cloud 平台**（**GCP**）上运行的 Java 和 Go 应用程序的性能

    +   **Java 框架**：

        +   **JProfiler**：用于详细分析 CPU、内存和线程使用的商业分析器

        +   **YourKit Java Profiler**：另一个具有全面分析功能的商业选项

        +   **Java VisualVM**：包含在 JDK 中的免费工具，提供基本的分析和监控功能

        +   **Java 飞行记录器**（**JFR**）：用于低开销分析和诊断的内置工具，在生产环境中特别有用

+   **基准测试**：比较同一任务的不同实现性能：

    +   **云**：

        +   **AWS Lambda 性能调优**：优化 Lambda 函数的内存和并发设置

        +   **Azure 性能基准测试**：为 Azure 中各种虚拟机类型和工作负载提供参考分数

        +   **Google Cloud 基准测试**：提供 GCP 上不同计算选项的性能数据

    +   **Java 框架**：

        +   **Java 微基准测试工具**（**JMH**）：用于创建可靠和精确微基准测试的框架

        +   **Caliper**：来自 Google 的另一个微基准测试框架

        +   **SPECjvm2008**：用于衡量 Java 应用程序性能的标准基准测试套件

+   **监控工具**：持续跟踪和评估各种资源的性能和健康状况，如 CPU、磁盘和网络使用情况以及应用程序性能指标：

    +   **云**：

        +   **Amazon CloudWatch**：监控 AWS 服务中的各种指标，包括 CPU、内存、磁盘和网络使用情况

        +   **Azure Monitor**：为 Azure 资源提供全面的监控，包括应用程序性能指标

        +   **Google Cloud Monitoring**：为 GCP 资源提供监控和日志记录功能

    +   **Java 框架**：

        +   **Java 管理扩展**（**JMX**）：Java 应用程序暴露管理和监控信息的内置 API

        +   **Micrometer**：用于收集和导出指标到不同监控系统的框架（例如 Prometheus 和 Graphite）。

        +   **Spring Boot Actuator**：提供用于监控 Spring Boot 应用程序的现成端点。

通过掌握这些工具和指标，您可以从蒙眼的速度恶魔转变为数据驱动的指挥家，自信地运用并行处理的力量，同时确保最佳性能和效率。

在下一节中，我们将探讨并行主义的另一面：并行性的潜在陷阱。我们将深入研究线程竞争、竞态条件和您可能遇到的其它挑战。

## 并行处理中的挑战与解决方案

并行处理加速了计算，但伴随着线程竞争、竞态条件和调试复杂性等挑战。理解和解决这些问题对于高效的并行计算至关重要。让我们深入了解这些问题：

+   **线程竞争**：当多个线程竞争相同资源时发生，导致性能问题，如增加的等待时间、资源饥饿和死锁。

+   **竞态条件**：当多个线程不可预测地访问共享数据时，就会发生这种情况，导致数据损坏和不可靠的程序行为。

+   **调试复杂性**：在多线程环境中调试具有挑战性，因为存在非确定性行为和隐藏的依赖关系，例如共享状态依赖和执行顺序依赖。这些依赖关系通常源于代码中未明确表示的线程交互，但可能影响程序的行为。

虽然这些挑战可能看起来令人畏惧，但它们并非不可逾越。让我们深入了解缓解这些陷阱的实际策略：

+   `ConcurrentHashMap`或`ConcurrentLinkedQueue`在处理共享数据时，防止并发访问问题和数据损坏。

+   **采用无锁算法**：考虑使用无锁算法，如**比较并交换**（**CAS**）操作，这些操作避免了与传统锁相关的开销，可以提高性能同时减轻竞争。

+   `AtomicInteger`，它保证了底层值的线程安全更新。*   **精通** **并行调试**：

    +   **使用具有线程视图的可视化调试器**：例如 Eclipse 或 IntelliJ IDEA 这样的调试器提供了用于可视化线程执行时间线、识别死锁和定位竞态条件的专用视图。

    +   **利用带时间戳的日志**：在多线程代码中策略性地添加时间戳到日志中，有助于您重建事件序列并识别导致问题的线程。

    +   **采用断言检查**：在代码的关键点放置断言检查，以检测可能表明竞态条件的不预期的数据值或执行路径。

    +   **考虑自动化测试工具**：具有并行执行能力的工具，如 JUnit，可以帮助你在开发早期阶段发现并发相关的问题

这里有一些如何在 AWS 中避免这些问题的实际例子：

+   **Amazon SQS – 消息队列的并行处理**：

    +   **用例**：使用 Amazon Simple Queue Service（SQS）的批量操作实现消息队列处理的并行处理

    +   **场景**：一个系统需要高效地处理大量传入的消息

    +   **实现**：而不是逐个处理消息，系统使用 Amazon SQS 的批量操作并行处理多个消息。

    +   **优势**：这种方法最小化了线程竞争，因为多个消息是批量读取和写入的，而不是为单个消息处理而竞争

+   **Amazon DynamoDB – 原子更新和条件写入**：

    +   **用例**：利用 DynamoDB 的原子更新和条件写入进行安全的并行数据访问和修改。

    +   **场景**：一个在线商店在 DynamoDB 中跟踪产品库存，并在多个购买同时发生时需要安全地更新库存水平。

    +   **实现**：在处理购买时，系统使用 DynamoDB 的原子更新来调整库存水平。条件写入确保只有在库存水平足够的情况下才会进行更新，从而防止竞争条件。

    +   **优势**：这确保了即使在并发购买交易的情况下，库存水平也能被准确维护。

+   **AWS Lambda – 无状态函数和资源管理**：

    +   **用例**：设计 AWS Lambda 函数为无状态，避免共享资源，以实现更简单、更安全的并发执行。

    +   **场景**：一个 Web 应用程序使用 Lambda 函数来处理用户请求，例如检索用户数据或处理交易。

    +   **实现**：每个 Lambda 函数都被设计为无状态的，这意味着它不依赖于或改变共享资源。任何所需的数据都通过其请求传递给函数。

    +   **优势**：这种无状态设计简化了 Lambda 执行，并减少了当同一函数因不同用户并发调用而出现数据不一致或冲突的风险。

在这些情况下，目标是通过利用 AWS 的内置功能来有效地处理并发，确保应用程序保持健壮、可扩展和错误-free。通过采用这些最佳实践和实际解决方案，你可以自信地应对并行处理的复杂性。记住，掌握并发需要仔细平衡速度、效率和可靠性。

在下一节中，我们将探讨并行处理的权衡，帮助你做出明智的决定，何时利用其力量，何时坚持使用经过验证的顺序方法。

## 评估软件设计中的并行性 – 平衡性能和复杂性

在软件设计中实现并行处理涉及在提高性能的潜力与它带来的额外复杂性之间的关键权衡。进行仔细的评估是确定并行化是否合理的必要条件。

这里是并行化的考虑因素：

+   **任务适宜性**：评估任务是否适合并行化，以及预期的性能提升是否足以证明增加的复杂性是合理的。

+   **资源可用性**：评估有效并行执行所需的硬件能力，如 CPU 核心和内存。

+   **开发限制**：考虑开发和维护并行化系统所需的时间、预算和专业知识。

+   **专业知识要求**：确保你的团队拥有并行编程所需的技能。

并行处理的方法应该从简单、模块化的设计开始，以便更容易过渡到并行化。基准测试对于衡量潜在的性能改进至关重要。选择增量重构，并在每一步都进行全面的测试，以确保并行过程的顺利集成。

从所有这些讨论中，我们得出结论，并行处理可以显著提高性能，但成功的实施需要平衡的方法，考虑到任务适宜性、资源可用性和开发团队的专长。这是一个强大的工具，当谨慎使用并清晰设计时，可以导致高效且易于维护的代码。记住，尽管并行处理很强大，但它不是万能的解决方案，应该有策略地使用。

# 摘要

本章是邀请您进入这个迷人的并行处理世界的邀请，我们探索了您可用的工具。首先是 Fork/Join 框架。您的总厨师，擅长将艰巨的任务分解成小份的子食谱，确保每个人都有一份角色扮演。但效率是关键，这就是工作窃取算法介入的地方。想象一下，厨师们互相看了看对方的肩膀，如果有人落后，就跳进去帮忙，并保持厨房像一台运转良好的机器一样嗡嗡作响。

然而，并非所有任务都是平等的。这就是`RecursiveTask`和`RecursiveAction`介入的地方。它们就像专注于不同课程的厨师，一个细致地切菜，另一个搅拌着慢炖的酱汁，每个人都专注于他们自己的烹饪难题的一部分。

现在，让我们谈谈效率。并行流就像预先清洗和切好的食材，准备好被扔进处理锅中。我们看到它们如何简化集合上的数据处理，使用 Fork/Join 框架作为他们的秘密武器来提高速度，特别是对于那些处理大量数据的人来说。

然而，选择正确的工具至关重要。这就是为什么我们深入到并行处理对决中，将 Fork/Join 与其他方法如`ThreadPoolExecutor`和`CompletableFuture`进行对比。这帮助你理解它们的优缺点，并使你能够做出明智的决定。

然而，复杂性潜伏在阴影中。因此，我们也处理了处理具有依赖关系的任务的技巧，学习了如何分解它们，并保持数据同步。这确保了你的烹饪杰作不会变成混乱的杂烩。

谁不喜欢一点优化？因此，我们探索了微调你的并行处理策略，并学习了如何平衡任务大小和并行级别以获得最有效的性能，就像厨师调整热量和调味料以达到完美一样。

最后，我们深入到自定义 Spliterator 的高级领域，赋予你根据特定需求定制并行流处理的能力。

每道菜都有自己的权衡，我们讨论了性能提升和复杂度之间的平衡，引导你做出明智的软件设计决策，让你感到满意，而不是疲惫不堪。

在本章中，我们编排了一部并行处理的交响曲，但当你的烹饪作品冲突，锅开始沸腾时会发生什么？这就是第四章介入的地方，我们将深入探讨 Java 并发工具和测试，这是你处理多线程微妙舞蹈的必备工具包。

# 问题

1.  Fork/Join 框架在 Java 中的主要目的是什么？

    1.  为 Java 应用程序提供 GUI 界面

    1.  通过递归拆分和执行任务来增强并行处理

    1.  简化 Java 应用程序中的数据库连接

    1.  在 Java 应用程序中管理网络连接

1.  `RecursiveTask`和`RecursiveAction`在 Fork/Join 框架中的区别是什么？

    1.  `RecursiveTask`返回一个值，而`RecursiveAction`不返回

    1.  `RecursiveAction`返回一个值，而`RecursiveTask`不返回

    1.  两者都返回值，但`RecursiveAction`是异步返回的

    1.  没有区别；它们可以互换

1.  工作窃取算法在 Fork/Join 框架中扮演什么角色？

    1.  它加密数据以进行安全处理

    1.  它允许空闲线程接管忙碌线程的任务

    1.  它根据复杂度优先执行任务

    1.  它减少了应用程序的内存占用

1.  以下哪项是优化 Java 中并行处理性能的最佳实践？

    1.  增加共享数据的使用

    1.  平衡任务粒度和并行级别

    1.  避免使用线程安全的数据结构

    1.  持续使用最高可能的并行级别

1.  在软件设计中实现并行处理时，应考虑哪些因素？

    1.  色彩方案和 UI 设计

    1.  任务的本质、资源可用性和团队的专业知识

    1.  正在使用的硬件品牌

    1.  编程语言的流行度
