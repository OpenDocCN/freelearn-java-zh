# 第十四章：栈和队列

本章涵盖了涉及栈和队列的最受欢迎的面试编码挑战。主要是，您将学习如何从头开始提供栈/队列实现，以及如何通过 Java 的内置实现来解决编码挑战，例如`Stack`类和`Queue`接口实现，特别是`ArrayDeque`。通常，此类别的编码挑战将要求您构建栈/队列，或者要求您使用 Java 的内置实现解决特定问题。根据问题的不同，它可能明确禁止您调用某些内置方法，这将导致您找到一个简单的解决方案。

通过本章结束时，您将深入了解栈和队列，能够利用它们的功能，并且能够识别和编写依赖于栈和队列的解决方案。

在本章中，您将学习以下主题：

+   概述栈

+   概述队列

+   编码挑战

让我们首先简要介绍栈的数据结构。

# 技术要求

本章中提供的所有代码文件都可以在 GitHub 上找到，网址为[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter12`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter12)。

# 概述栈

栈是一种使用**后进先出**（**LIFO**）原则的线性数据结构。想象一堆需要清洗的盘子。您从顶部取出第一个盘子（最后添加的盘子），然后清洗它。然后，您从顶部取出下一个盘子，依此类推。这正是现实生活中的栈（例如，一堆盘子，一堆书，一堆 CD 等）。

因此，从技术上讲，在栈中，元素只能从一端添加（称为**push**操作）和移除（称为**pop**操作）（称为**top**）。

在栈中执行的最常见操作如下：

+   `push(E e)`: 将元素添加到栈的顶部

+   `E pop()`: 移除栈顶的元素

+   `E peek()`: 返回（但不移除）栈顶的元素

+   `boolean isEmpty()`: 如果栈为空则返回`true`

+   `int size()`: 返回栈的大小

+   `boolean isFull()`: 如果栈已满则返回`true`

与数组不同，栈不以常数时间提供对第 n 个元素的访问。但是，它确实提供了添加和移除元素的常数时间。栈可以基于数组甚至基于链表实现。这里使用的实现是基于数组的，并命名为`MyStack`。该实现的存根如下所示：

```java
public final class MyStack<E> {
  private static final int DEFAULT_CAPACITY = 10;
  private int top;
  private E[] stack;
  MyStack() {
    stack = (E[]) Array.newInstance(
             Object[].class.getComponentType(), 
             DEFAULT_CAPACITY);
    top = 0; // the initial size is 0
  }
  public void push(E e) {}
  public E pop() {}
  public E peek() {}
  public int size() {}
  public boolean isEmpty() {}
  public boolean isFull() {}
  private void ensureCapacity() {}
}
```

将元素推入栈意味着将该元素添加到基础数组的末尾。在推入元素之前，我们必须确保栈不是满的。如果满了，我们可以通过消息/异常来表示这一点，或者我们可以增加其容量，如下所示：

```java
// add an element 'e' in the stack
public void push(E e) {
  // if the stack is full, we double its capacity
  if (isFull()) {
    ensureCapacity();
  }
  // adding the element at the top of the stack
  stack[top++] = e;
}
// used internally for doubling the stack capacity
private void ensureCapacity() {
  int newSize = stack.length * 2;
  stack = Arrays.copyOf(stack, newSize);
}
```

如您所见，每当我们达到栈的容量时，我们都会将其大小加倍。从栈中弹出一个元素意味着我们返回最后添加到基础数组中的元素。通过将最后一个索引置空来从基础数组中移除该元素，如下所示：

```java
// pop top element from the stack
public E pop() {
  // if the stack is empty then just throw an exception
  if (isEmpty()) {
    throw new EmptyStackException();
  }
  // extract the top element from the stack                
  E e = stack[--top];
  // avoid memory leaks
  stack[top] = null;
  return e;
}
```

从栈中查看元素意味着返回最后添加到基础数组中的元素，但不从该数组中移除它：

```java
// return but not remove the top element in the stack
public E peek() {
  // if the stack is empty then just throw an exception
  if (isEmpty()) {
    throw new EmptyStackException();
  }
  return stack[top - 1];
}
```

由于此实现可能代表您在面试中可能遇到的编码挑战，建议您花时间分析其代码。完整的应用程序称为*MyStack*。

# 概述队列

队列是一种使用**先进先出**（**FIFO**）原则的线性数据结构。想象排队购物的人。您还可以想象成蚂蚁在队列中行走。

因此，从技术上讲，元素的移除顺序与它们添加的顺序相同。在队列中，添加到一端的元素称为后端（这个操作称为入队操作），从另一端移除的元素称为前端（这个操作称为出队或轮询操作）。

队列中的常见操作如下：

+   `enqueue(E e)`: 将元素添加到队列的末尾

+   `E dequeue()`: 删除并返回队列前面的元素

+   `E peek()`: 返回（但不删除）队列前面的元素

+   `boolean isEmpty()`: 如果队列为空则返回`true`

+   `int size()`: 返回队列的大小

+   `boolean isFull()`：如果队列已满则返回`true`

与数组不同，队列不提供以常量时间访问第 n 个元素的功能。但是，它确实提供了添加和删除元素的常量时间。队列可以基于数组实现，甚至可以基于链表或堆栈（堆栈是基于数组或链表构建的）实现。这里使用的实现是基于数组的，并且命名为`MyQueue`。这个实现的存根在这里列出：

```java
public final class MyQueue<E> {
  private static final int DEFAULT_CAPACITY = 10;
  private int front;
  private int rear;
  private int count;
  private int capacity;
  private E[] queue;
  MyQueue() {
    queue = (E[]) Array.newInstance(
                Object[].class.getComponentType(), 
                DEFAULT_CAPACITY);
  count = 0; // the initial size is 0
  front = 0;
  rear = -1;
  capacity = DEFAULT_CAPACITY;
  }
  public void enqueue(E e) {}
  public E dequeue() {}
  public E peek() {}
  public int size() {}
  public boolean isEmpty() {}
  public boolean isFull() {}
  private void ensureCapacity() {}
} 
```

将元素加入队列意味着将该元素添加到底层数组的末尾。在将元素加入队列之前，我们必须确保队列不是满的。如果满了，我们可以通过消息/异常来表示，或者我们可以增加其容量，如下所示：

```java
// add an element 'e' in the queue
public void enqueue(E e) {
  // if the queue is full, we double its capacity
  if (isFull()) {
    ensureCapacity();
  }
  // adding the element in the rear of the queue
  rear = (rear + 1) % capacity;
  queue[rear] = e;
  // update the size of the queue
  count++;
}
// used internally for doubling the queue capacity
private void ensureCapacity() {       
  int newSize = queue.length * 2;
  queue = Arrays.copyOf(queue, newSize);
  // setting the new capacity
  capacity = newSize;
}
```

从队列中出列一个元素意味着从底层数组的开头返回下一个元素。该元素从数组中删除：

```java
// remove and return the front element from the queue
public E dequeue() {
  // if the queue is empty we just throw an exception
  if (isEmpty()) {
    throw new EmptyStackException();
  }
  // extract the element from the front
  E e = queue[front];
  queue[front] = null;
  // set the new front
  front = (front + 1) % capacity;
  // decrease the size of the queue
  count--;
  return e;
}
```

从队列中窥视一个元素意味着从底层数组的开头返回下一个元素，而不将其从数组中删除：

```java
// return but not remove the front element in the queue
public E peek() {
  // if the queue is empty we just throw an exception
  if (isEmpty()) {
    throw new EmptyStackException();
  }
  return queue[front];
}
```

由于这个实现可以代表你在面试中可能遇到的编码挑战，建议你花时间来分析它的代码。完整的应用程序称为*MyQueue*。

# 编码挑战

在接下来的 11 个编码挑战中，我们将涵盖在过去几年中出现在面试中的涉及栈和队列的最流行问题，这些问题涉及到各种雇佣 Java 开发人员的公司。其中最常见的问题之一，*使用一个数组实现三个栈*，在*第十章**，数组和字符串*中有所涉及。

以下编码挑战的解决方案依赖于 Java 内置的`Stack`和`ArrayDeque`API。所以，让我们开始吧！

## 编码挑战 1 - 反转字符串

**问题**：假设你有一个字符串。使用堆栈将其反转。

**解决方案**：使用堆栈反转字符串可以按以下方式完成：

1.  从左到右循环字符串，并将每个字符推入堆栈。

1.  循环堆栈并逐个弹出字符。每个弹出的字符都放回字符串中。

基于这两个步骤的代码如下：

```java
public static String reverse(String str) {
  Stack<Character> stack = new Stack();
  // push characters of the string into the stack
  char[] chars = str.toCharArray();
  for (char c : chars) {
    stack.push(c);
  }
  // pop all characters from the stack and
  // put them back to the input string
  for (int i = 0; i < str.length(); i++) {
    chars[i] = stack.pop();
  }
  // return the string
  return new String(chars);
}
```

完整的应用程序称为*StackReverseString*。

## 编码挑战 2 - 大括号堆栈

**亚马逊**，**谷歌**，**Adobe**，**微软**，**Flipkart**

包含大括号的字符串。编写一小段代码，如果有匹配的大括号对，则返回`true`。如果我们可以找到适当顺序的闭合大括号来匹配开放的大括号，那么我们可以说有一个匹配的对。例如，包含匹配对的字符串如下：{{{}}}{}{{}}。

`false`。其次，如果它们的数量相等，则它们必须按适当的顺序；否则，我们返回`false`。按适当的顺序，我们理解最后打开的大括号是第一个关闭的，倒数第二个是第二个关闭的，依此类推。如果我们依赖于堆栈，那么我们可以详细说明以下算法：

1.  对于给定字符串的每个字符，做出以下决定之一：

a. 如果字符是一个开放的大括号，{，那么将其放入堆栈。

b. 如果字符是闭合大括号，}，则执行以下操作：

i. 检查堆栈顶部，如果是{，则弹出并将其移动到下一个字符。

ii. 如果不是{，则返回`false`。

1.  如果堆栈为空，则返回`true`（我们找到了所有配对）；否则返回`false`（堆栈包含不匹配的大括号）。

将这些步骤转化为代码，结果如下：

```java
public static boolean bracesMatching(String bracesStr) {
  Stack<Character> stackBraces = new Stack<>();
  int len = bracesStr.length();
  for (int i = 0; i < len; i++) {
    switch (bracesStr.charAt(i)) {
      case '{':
        stackBraces.push(bracesStr.charAt(i));
        break;
      case '}':
        if (stackBraces.isEmpty()) { // we found a mismatch
          return false;
        }
        // for every match we pop the corresponding '{'
        stackBraces.pop(); 
        break;
      default:
        return false;
    }
  }
  return stackBraces.empty();
}
```

完整的应用程序称为*StackBraces*。通过实现类似的问题，但是对于多种类型的括号（例如，在相同的给定字符串中允许(){}[]），来挑战自己。

## 编程挑战 3 - 堆叠盘

**亚马逊**，**谷歌**，**Adobe**，**微软**，**Flipkart**

`push()`和`pop()`方法将像单个堆栈一样工作。另外，编写一个`popAt(int stackIndex)`方法，它会从堆栈中弹出一个值，如`stackIndex`所示。

**解决方案**：我们知道如何处理单个堆栈，但是如何将多个堆栈链接在一起呢？嗯，既然我们需要*链接*，那么链表怎么样？如果链表中每个节点都包含一个堆栈，那么节点的下一个指针将指向下一个堆栈。以下图表可视化了这个解决方案：

![图 12.1 - 堆栈的链表](img/Figure_12.1_B15403.jpg)

图 12.1 - 堆栈的链表

每当当前堆栈容量超过时，我们就创建一个新节点并将其附加到链表中。Java 的内置链表（`LinkedList`）通过`getLast()`方法使我们可以访问最后一个节点。换句话说，通过`LinkedList#getLast()`，我们可以轻松操作当前堆栈（例如，我们可以推送或弹出一个元素）。通过`LinkedList#add()`方法很容易添加一个新的堆栈。基于这些语句，我们可以实现`push()`方法，如下所示：

```java
private static final int STACK_SIZE = 3;
private final LinkedList<Stack<Integer>> stacks 
  = new LinkedList<>();
public void push(int value) {
  // if there is no stack or the last stack is full
  if (stacks.isEmpty() || stacks.getLast().size()
      >= STACK_SIZE) {
    // create a new stack and push the value into it
    Stack<Integer> stack = new Stack<>();
    stack.push(value);
    // add the new stack into the list of stacks
    stacks.add(stack);
  } else {
    // add the value in the last stack
    stacks.getLast().push(value);
  }
}
```

如果我们想要弹出一个元素，那么我们必须从最后一个堆栈中这样做，所以`LinkedList#getLast()`在这里非常方便。这里的特殊情况是当我们从最后一个堆栈中弹出最后一个元素时。当这种情况发生时，我们必须删除最后一个堆栈，在这种情况下，倒数第二个（如果有的话）将成为最后一个。以下代码说明了这一点：

```java
public Integer pop() {
  // find the last stack
  Stack<Integer> lastStack = stacks.getLast();
  // pop the value from the last stack
  int value = lastStack.pop();
  // if last stack is empty, remove it from the list of stacks
  removeStackIfEmpty();
  return value;
}
private void removeStackIfEmpty() {
  if (stacks.getLast().isEmpty()) {
      stacks.removeLast();
  }
}
```

最后，让我们专注于实现`popAt(int stackIndex)`方法。我们可以通过简单调用`stacks.get(stackIndex).pop()`从`stackIndex`堆栈中弹出。一旦我们弹出一个元素，我们必须移动剩余的元素。下一个堆栈的底部元素将成为由`stackIndex`指向的堆栈的顶部元素，依此类推。如果最后一个堆栈包含单个元素，则移动其他元素将消除最后一个堆栈，并且其前面的堆栈将成为最后一个堆栈。让我们通过代码来看一下：

```java
public Integer popAt(int stackIndex) {
  // get the value from the correspondind stack
  int value = stacks.get(stackIndex).pop();
  // pop an element -> must shift the remaining elements        
  shift(stackIndex);
  // if last stack is empty, remove it from the list of stacks
  removeStackIfEmpty();
  return value;
}
private void shift(int index) {
  for (int i = index; i<stacks.size() - 1; ++i) {
    Stack<Integer> currentStack = stacks.get(i);
    Stack<Integer> nextStack = stacks.get(i + 1);
    currentStack.push(nextStack.remove(0));
  }
}
```

完整的应用程序称为*StackOfPlates*。

## 编程挑战 4 - 股票跨度

**亚马逊**，**谷歌**，**Adobe**，**微软**，**Flipkart**

**问题**：假设你已经获得了一个单一股票连续多天的价格数组。股票跨度由前几天（今天）的股票价格小于或等于当前天（今天）的股票价格的天数表示。例如，考虑股票价格覆盖 10 天的情况；即{55, 34, 22, 23, 27, 88, 70, 42, 51, 100}。结果的股票跨度是{1, 1, 1, 2, 3, 6, 1, 1, 2, 10}。注意，对于第一天，股票跨度始终为 1。编写一小段代码，计算给定价格列表的股票跨度。

**解决方案**：我们可以从给定的示例开始，尝试将其可视化，如下所示：

![图 12.2 - 10 天的股票跨度](img/Figure_12.2_B15403.jpg)

图 12.2 - 10 天的股票跨度

从前面的图表中，我们可以观察到以下内容：

+   对于第一天，跨度始终为 1。

+   对于第 2 天，价格为 34。由于 34 小于前一天（55）的价格，第 2 天的股票跨度也是 1。

+   对于第 3 天，价格为 22。由于 22 小于前一天（34）的价格，第 3 天的股票跨度也是 1。第 7 天和第 8 天也属于同样的情况。

+   对于第 4 天，价格是 23。由于 23 大于前一天的价格（22），但小于第 2 天的价格，所以股票跨度为 2。第 9 天与第 4 天类似。

+   对于第 5 天，价格是 27。由于这个价格大于第 3 天和第 4 天的价格，但小于第 2 天的价格，所以股票跨度为 3。

+   对于第 6 天，价格是 88。这是迄今为止最高的价格，所以股票跨度是 6。

+   对于第 10 天，价格是 100。这是迄今为止最高的价格，所以股票跨度是 10。

注意，我们计算当前天的股票跨度，是当前天的索引与对应于最后一个最大股价的那一天的索引之间的差。在追踪这种情况之后，我们可能会有这样的第一个想法：对于每一天，扫描它之前的所有天，直到股价大于当前天。换句话说，我们使用了蛮力方法。正如我在本书中早些时候提到的，蛮力方法应该在面试中作为最后的手段使用，因为它的性能较差，面试官不会感到印象深刻。在这种情况下，蛮力方法的时间复杂度为 O(n2)。

然而，让我们换个角度来思考。对于每一天，我们想找到一个之前的一天，它的价格比当前天的价格高。换句话说，我们要找到最后一个价格比当前天的价格高的那一天。

在这里，我们应该选择一个后进先出的数据结构，它允许我们按降序推入价格并弹出最后推入的价格。一旦我们做到这一点，我们可以遍历每一天，并将栈顶的价格与当前天的价格进行比较。直到栈顶的价格小于当前天的价格，我们可以从栈中弹出。但是如果栈顶的价格大于当前天的价格，那么我们计算当前天的股票跨度，就是当前天和栈顶价格对应的那一天之间的天数差。如果我们按降序将价格推入栈中，这将起作用 - 最大的价格在栈顶。然而，由于我们可以将股票跨度计算为当前天的索引与对应于最后一个最大股价的那一天的索引之间的差（我们用`i`表示），我们可以简单地将`i`索引存储在栈中；`stackPrices[i]`（我们将价格数组表示为`stackPrices`）将返回第*i*天的股票价格。

这可以通过以下算法实现：

1.  第一天的股票跨度为 1，索引为 0 - 我们将这个索引推入栈中（我们将其表示为`dayStack`；因此，`dayStack.push(0)`）。

1.  我们循环剩余的天数（第 2 天的索引为 1，第 3 天的索引为 2，依此类推）并执行以下操作：

a. 当`stockPrices[i] > stockPrices[dayStack.peek()]`并且`!dayStack.empty()`时，我们从栈中弹出（`dayStack.pop()`）。

1.  如果`dayStack.empty()`，那么`i+1`的股票跨度。

1.  如果`stockPrices[i] <= stockPrices[dayStack.peek()]`，那么股票跨度就是`i - dayStack.peek()`。

1.  将当前天的索引`i`推入栈中（`dayStack`）。

让我们看看这个算法如何适用于我们的测试案例：

1.  第一天的股票跨度为 1，索引为 0 - 我们将这个索引推入栈中，`dayStack.push(0)`。

1.  对于第 2 天，`stockPrices[1]=34`，`stockPrices[0]=55`。由于 34 < 55，第 2 天的股票跨度为`i - dayStack.peek()` = 1 - 0 = 1。我们将 1 推入栈中，`dayStack.push(1)`。

1.  对于第三天，`stockPrices[2]`=22，`stockPrices[1]`=34。由于 22 < 34，第 3 天的股票跨度为 2 - 1 = 1。我们将 1 推入栈中，`dayStack.push(2)`。

1.  对于第 4 天，`stockPrices[3]`=23，`stockPrices[2]`=22。由于 23 > 22 并且栈不为空，我们弹出栈顶，所以我们弹出值 2。由于 23 < 34（`stockPrices[1]`），第 4 天的股票跨度为 3 - 1 = 2。我们将 3 推入栈中，`dayStack.push(3)`。

1.  对于第五天，`stockPrices[4]`=27 和 `stockPrices[3]`=23。由于 27 > 23 并且栈不为空，我们弹出栈顶，所以我们弹出值 3。接下来，27 < 34（记住我们在上一步弹出了值 2，所以下一个栈顶的值为 1），第 5 天的股票跨度为 4 - 1 = 3。我们在栈中推入 4，`dayStack.push(4)`。

1.  对于第六天，`stockPrices[5]`=88 和 `stockPrices[4]`=27。由于 88 > 27 并且栈不为空，我们弹出栈顶，所以我们弹出值 4。接下来，88 > 34 并且栈不为空，所以我们弹出值 1。接下来，88 > 55 并且栈不为空，所以我们弹出值 0。接下来，栈为空，第 6 天的股票跨度为 5 + 1 = 6。

好了，我想你已经明白了，现在，挑战自己，继续到第 10 天。目前，我们有足够的信息将这个算法转化为代码：

```java
public static int[] stockSpan(int[] stockPrices) {
  Stack<Integer> dayStack = new Stack();
  int[] spanResult = new int[stockPrices.length];
  spanResult[0] = 1; // first day has span 1
  dayStack.push(0);
  for (int i = 1; i < stockPrices.length; i++) {
    // pop until we find a price on stack which is 
    // greater than the current day's price or there 
    // are no more days left
    while (!dayStack.empty() 
      && stockPrices[i] > stockPrices[dayStack.peek()]) {
      dayStack.pop();
    }
    // if there is no price greater than the current 
    // day's price then the stock span is the numbers of days
    if (dayStack.empty()) {
        spanResult[i] = i + 1;
    } else {
      // if there is a price greater than the current 
      // day's price then the stock span is the 
      // difference between the current day and that day
        spanResult[i] = i - dayStack.peek();
    }
    // push current day onto top of stack
     dayStack.push(i);
  }
  return spanResult;
}
```

完整的应用程序称为 *StockSpan*。

## 编码挑战 5 – 栈最小值

**亚马逊**，**谷歌**，**Adobe**，**微软**，**Flipkart**

`push()`、`pop()` 和 `min()` 方法应在 O(1) 时间内运行。

`push()` 和 `pop()` 在 O(1) 时间内运行。

符合问题约束的解决方案需要一个额外的栈来跟踪最小值。主要是，当推送的值小于当前最小值时，我们将这个值添加到辅助栈（我们将其表示为 `stackOfMin`）和原始栈中。如果从原始栈中弹出的值是 `stackOfMin` 的栈顶，则我们也从 `stackOfMin` 中弹出它。在代码方面，我们有以下内容：

```java
public class MyStack extends Stack<Integer> {
  Stack<Integer> stackOfMin;
  public MyStack() {
    stackOfMin = new Stack<>();
  }
  public Integer push(int value) {
    if (value <= min()) {
       stackOfMin.push(value);
    }
    return super.push(value);
  }
  @Override
  public Integer pop() {
    int value = super.pop();
    if (value == min()) {
       stackOfMin.pop();
    }
    return value;
  }
  public int min() {
   if (stackOfMin.isEmpty()) {
      return Integer.MAX_VALUE;
    } else {
      return stackOfMin.peek();
    }
  }
}
```

完成！我们的解决方案以 O(1) 复杂度时间运行。完整的应用程序称为 *MinStackConstantTime*。与此相关的一个问题要求您在常数时间和空间内实现相同的功能。这个问题的解决方案施加了几个限制，如下：

+   `pop()` 方法返回 `void`，以避免返回不正确的值。

+   给定值乘以 2 不应超出 `int` 数据类型的范围。

简而言之，这些限制是由解决方案本身造成的。我们不能使用额外的空间；因此，我们将使用初始值栈来存储最小值。此外，我们需要将给定值乘以 2，因此我们应确保不超出 `int` 范围。为什么我们需要将给定值乘以 2？

让我们来解释一下这个问题！假设我们需要将一个值推入一个具有特定最小值的栈中。如果这个值大于或等于当前最小值，那么我们可以简单地将它推入栈中。但是如果它小于最小值，那么我们推入 2**值-最小值*，这应该小于值本身。然后，我们将当前最小值更新为值。

当我们弹出一个值时，我们必须考虑两个方面。如果弹出的值大于或等于最小值，那么这是之前推送的真实值。否则，弹出的值不是推送的值。真正推送的值存储在最小值中。在我们弹出栈顶（最小值）之后，我们必须恢复先前的最小值。先前的最小值可以通过 2**最小值 - 栈顶* 获得。换句话说，由于当前栈顶是 2**值 - 先前的最小值*，而值是当前最小值，先前的最小值是 2**当前最小值 - 栈顶*。以下代码说明了这个算法：

```java
public class MyStack {
  private int min;
  private final Stack<Integer> stack = new Stack<>();
  public void push(int value) {
    // we don't allow values that overflow int/2 range
    int r = Math.addExact(value, value);
    if (stack.empty()) {
      stack.push(value);
      min = value;
    } else if (value > min) {
      stack.push(value);
    } else {
      stack.push(r - min);
      min = value;
    }
  }
  // pop() doesn't return the value since this may be a wrong   
  // value (a value that was not pushed by the client)!
  public void pop() {
    if (stack.empty()) {
      throw new EmptyStackException();
    }
    int top = stack.peek();
    if (top < min) {
      min = 2 * min - top;
    }
    stack.pop();
  }
  public int min() {
    return min;
  }
}
```

完整的应用程序称为 *MinStackConstantTimeAndSpace*。

## 编码挑战 6 – 通过栈实现队列

**谷歌**，**Adobe**，**微软**，**Flipkart**

**问题**：通过两个栈设计一个队列。

**解决方案**：为了找到这个问题的合适解决方案，我们必须从队列和栈之间的主要区别开始。我们知道队列按照先进先出的原则工作，而栈按照后进先出的原则工作。接下来，我们必须考虑主要的操作（推入、弹出和查看）并确定它们之间的区别。

它们都以相同的方式推送新元素。当我们将一个元素推入队列时，我们是从一端（队列的后端）推入的。当我们将一个元素推入栈时，我们是从栈的新顶部推入的，这可以被视为与队列的后端相同。

当我们从栈中弹出或查看一个值时，我们是从顶部这样做的。然而，当我们在队列上执行相同的操作时，我们是从前面这样做的。这意味着，当弹出或查看一个元素时，一个反转的栈将充当队列。以下图表说明了这一点：

![图 12.3 - 通过两个栈实现队列](img/Figure_12.3_B15403.jpg)

图 12.3 - 通过两个栈实现队列

因此，每个新元素都被推入*enqueue stack*作为新的顶部。当我们需要弹出或查看一个值时，我们使用*dequeue*栈，这是*enqueue stack*的反转版本。请注意，我们不必在每次弹出/查看操作时都反转*enqueue stack*。我们可以让元素停留在*dequeue stack*中，直到我们绝对必须反转元素。换句话说，对于每个弹出/查看操作，我们可以检查*dequeue stack*是否为空。只要*dequeue stack*不为空，我们就不需要反转*enqueue stack*，因为我们至少有一个元素可以弹出/查看。

让我们用代码来看一下：

```java
public class MyQueueViaStack<E> {
  private final Stack<E> stackEnqueue;
  private final Stack<E> stackDequeue;
  public MyQueueViaStack() {
    stackEnqueue = new Stack<>();
    stackDequeue = new Stack<>();
  }
  public void enqueue(E e) {
    stackEnqueue.push(e);
  }
  public E dequeue() {
    reverseStackEnqueue();
    return stackDequeue.pop();
  }
  public E peek() {
    reverseStackEnqueue();
    return stackDequeue.peek();
  }
  public int size() {
    return stackEnqueue.size() + stackDequeue.size();
  }
  private void reverseStackEnqueue() {
    if (stackDequeue.isEmpty()) {
      while (!stackEnqueue.isEmpty()) {
        stackDequeue.push(stackEnqueue.pop());
      }
    }
  }
}
```

完整的应用程序称为*QueueViaStack*。

## 编码挑战 7 - 通过队列实现栈

**Google**，**Adobe**，**Microsoft**

**问题**：设计一个通过两个队列实现的栈。

**解决方案**：为了找到这个问题的合适解决方案，我们必须从栈和队列之间的主要区别开始。我们知道栈是后进先出，而队列是先进先出。接下来，我们必须考虑主要操作（推入、弹出和查看）并确定它们之间的区别。

它们都以相同的方式推送新元素。当我们将一个元素推入栈时，我们是从栈的新顶部推入的。当我们将一个元素推入队列时，我们是从一端（队列的后端）推入的。队列的后端就像栈的顶部。

当我们从队列中弹出或查看一个值时，我们是从前面这样做的。然而，当我们在栈上执行相同的操作时，我们是从顶部这样做的。这意味着，当我们从充当栈的队列中弹出或查看一个元素时，我们需要轮询除最后一个元素之外的所有元素。最后一个元素就是我们弹出/查看的元素。以下图表说明了这一点：

![图 12.4 - 通过两个队列实现栈](img/Figure_12.4_B15403.jpg)

图 12.4 - 通过两个队列实现栈

正如前面的图表左侧所显示的，将一个元素推入栈和队列是一个简单的操作。前面图表的右侧显示了当我们想要从充当栈的队列中弹出/查看一个元素时会出现问题。主要是，在弹出/查看元素之前，我们必须将队列（在前面的图表中标记为*queue1*）中的元素（*rear*-1 和*front*之间）移动到另一个队列（在前面的图表中标记为*queue2*）。在前面的图表中，右侧，我们从*queue1*中轮询元素 2、5、3 和 1，并将它们添加到*queue2*中。接下来，我们从*queue1*中弹出/查看最后一个元素。如果我们弹出元素 6，那么*queue1*就会保持为空。如果我们查看元素 6，那么*queue1*就会保留这个元素。

现在，剩下的元素都在*queue2*中，所以为了执行另一个操作（推入、查看或弹出），我们有两个选项：

+   将*queue2*中剩余的元素移回*queue1*，恢复*queue1*。

+   使用*queue2*就像它是*queue1*一样，这意味着交替使用*queue1*和*queue2*。

在第二个选项中，我们避免了将*queue2*中的元素移回*queue1*的开销，目的是在*queue1*上执行下一个操作。虽然你可以挑战自己来实现第一个选项，但让我们更多地关注第二个选项。

如果我们考虑到我们应该使用的下一个操作的队列是不空的，那么可以交替使用*queue1*和*queue2*。由于我们在这两个队列之间移动元素，其中一个始终为空。因此，当我们查看一个元素时，会出现问题，因为查看操作不会移除元素，因此其中一个队列仍然保留该元素。由于没有一个队列是空的，我们不知道下一个操作应该使用哪个队列。解决方案非常简单：我们弹出最后一个元素，即使是对于查看操作，我们也将其存储为实例变量。随后的查看操作将返回此实例变量。推送操作将在推送给定值之前将此实例变量推回队列，并将此实例变量设置为`null`。弹出操作将检查此实例变量是否为`null`。如果不是`null`，那么这就是要弹出的元素。

让我们看看代码：

```java
public class MyStackViaQueue<E> {
  private final Queue<E> queue1;
  private final Queue<E> queue2;
  private E peek;
  private int size;
  public MyStackViaQueue() {
    queue1 = new ArrayDeque<>();
    queue2 = new ArrayDeque<>();
  }
  public void push(E e) {
    if (!queue1.isEmpty()) {
      if (peek != null) {
        queue1.add(peek);
      }
      queue1.add(e);
    } else {
      if (peek != null) {
        queue2.add(peek);
      }
      queue2.add(e);
    }
    size++;
    peek = null;
  }
  public E pop() {
    if (size() == 0) {
      throw new EmptyStackException();
    }
    if (peek != null) {
      E e = peek;
      peek = null;
      size--;
      return e;
    }
    E e;
    if (!queue1.isEmpty()) {
      e = switchQueue(queue1, queue2);
    } else {
      e = switchQueue(queue2, queue1);
    }
    size--;
    return e;
  }
  public E peek() {
    if (size() == 0) {
      throw new EmptyStackException();
    }
    if (peek == null) {
      if (!queue1.isEmpty()) {
        peek = switchQueue(queue1, queue2);
      } else {
        peek = switchQueue(queue2, queue1);
      }
    }
    return peek;
  }
  public int size() {
    return size;
  }
  private E switchQueue(Queue from, Queue to) {
    while (from.size() > 1) {
      to.add(from.poll());
    }
    return (E) from.poll();
  }
}
```

完整的应用程序称为*StackViaQueue*。

## 编码挑战 8 - 最大直方图面积

**亚马逊**，**谷歌**，**Adobe**，**微软**，**Flipkart**

**问题**：假设你已经得到了下图中显示的直方图：

![图 12.5 - 直方图，类间隔等于 1](img/Figure_12.5_B15403.jpg)

图 12.5 - 直方图，类间隔等于 1

我们将直方图定义为一个矩形条的图表，其中面积与某个变量的频率成比例。条的宽度称为直方图类间隔。例如，前面图像中的直方图的类间隔等于 1。有六个宽度均为 1，高度分别为 4、2、8、6、5 和 3 的条。

假设你已经得到了这些高度作为整数数组（这是问题的输入）。编写一小段代码，使用栈来计算直方图中最大的矩形区域。为了更好地理解这一点，下图突出显示了几个（不是全部）可以形成的矩形：

![图 12.6 - 直方图的矩形](img/Figure_12.6_B15403.jpg)

图 12.6 - 直方图的矩形

在前面的图像中，最大的矩形区域（即最大的矩形）是中间的一个，3 x 5 = 15。

**解决方案**：这个问题比起初看起来要困难得多。首先，我们需要分析给定的图像并制定几个声明。例如，非常重要的是要注意，只有当某个条的高度小于或等于该区域的高度时，该条才能成为矩形区域的一部分。此外，对于每个条，我们可以说，所有左侧高于当前条的条都可以与当前条形成一个矩形区域。同样，所有右侧高于当前条的条都可以与当前条形成一个矩形区域。

这意味着每个矩形区域由*左*和*右*边界限定，而(*右 - 左*) ** current_bar*给出了这个区域的值。我们应该计算所有可能的区域，并选择最高的区域作为我们实现的输出。以下图像突出显示了 3 x 5 矩形的左右边界：

![图 12.7 - 左右边界](img/Figure_12.7_B15403.jpg)

图 12.7 - 左右边界

请记住，我们必须使用栈来解决这个问题。现在我们有了一些可以引导我们解决问题的声明，是时候把栈引入讨论了。主要是，我们可以使用栈来计算左右边界。

我们从第一个条开始，将其索引（索引 0）推入栈中。我们继续处理剩下的条，并执行以下操作：

1.  重复*步骤 1a*、*1b*和*1c*，直到当前条小于栈顶部并且栈不为空：

a.我们弹出栈顶部。

b.我们计算左边界。

c. 我们计算可以在计算的左边界条和当前条之间形成的矩形区域的宽度。

d. 我们计算面积为计算的宽度乘以我们在*步骤 1a*中弹出的条的高度。

e. 如果这个区域比以前的大，那么我们存储这个区域。

1.  将当前条的索引推入栈中。

1.  重复从*步骤 1*直到每个条都被处理。

让我们看看代码方面的情况：

```java
public static int maxAreaUsingStack(int[] histogram) {
  Stack<Integer> stack = new Stack<>();
  int maxArea = 0;
  for (int bar = 0; bar <= histogram.length; bar++) {
    int barHeight;
    if (bar == histogram.length) {
      barHeight = 0; // take into account last bar
    } else {
      barHeight = histogram[bar];
    }
    while (!stack.empty() 
          && barHeight < histogram[stack.peek()]) {
      // we found a bar smaller than the one from the stack                
      int top = stack.pop();
      // find left boundary
      int left = stack.isEmpty() ? -1 : stack.peek();
      // find the width of the rectangular area 
      int areaRectWidth = bar - left - 1;
      // compute area of the current rectangle
      int area = areaRectWidth * histogram[top];
      maxArea = Integer.max(area, maxArea);
    }
    // add current bar (index) into the stack
    stack.push(bar);
  }        
  return maxArea;
}
```

这段代码的时间复杂度是 O(n)。此外，额外的空间复杂度是 O(n)。完整的应用程序称为*StackHistogramArea*。

## 编码挑战 9 - 最小数字

**问题**：考虑到你已经得到一个表示*n*位数的字符串。编写一小段代码，删除给定的*k*位数后打印出最小可能的数字。

**解决方案**：让我们假设给定的数字是*n*=4514327 和*k*=4。在这种情况下，删除四位数字后的最小数字是 127。如果*n*=2222222，那么最小数字是 222。

解决方案可以通过`Stack`和以下算法轻松实现：

1.  从左到右迭代给定的数字，逐位数字。

a. 只要给定的*k*大于 0，栈不为空，并且栈中的顶部元素大于当前遍历的数字：

i. 从栈中弹出顶部元素。

ii. 将*k*减 1。

b. 将当前数字推入栈中。

1.  当给定的*k*大于 0 时，执行以下操作（处理特殊情况，如 222222）：

a. 从栈中弹出元素。

b. 将*k*减 1。

在代码方面，我们有以下内容：

```java
public static void smallestAfterRemove(String nr, int k) {
  int i = 0;
  Stack<Character> stack = new Stack<>();
  while (i < nr.length()) {
    // if the current digit is less than the previous 
    // digit then discard the previous one
    while (k > 0 && !stack.isEmpty()
          && stack.peek() > nr.charAt(i)) {
      stack.pop();
      k--;
    }
    stack.push(nr.charAt(i));
    i++;
  }
  // cover corner cases such as '2222'
  while (k > 0) {
    stack.pop();
    k--;
  }
  System.out.println("The number is (as a printed stack; "
      + "ignore leading 0s (if any)): " + stack);
  }
}
```

完整的应用程序称为*SmallestNumber*。

## 编码挑战 10 - 岛屿

**亚马逊**，**Adobe**

**问题**：考虑到你已经得到一个包含只有 0 和 1 的*m*x*n*矩阵。按照惯例，1 表示陆地，0 表示水。编写一小段代码来计算岛屿的数量。岛屿被定义为由 0 包围的 1 组成的区域。

**解决方案**：让我们想象一个测试案例。以下是一个包含 6 个岛屿的 10x10 矩阵，分别标记为 1、2、3、4、5 和 6：

![图 12.8 - 10x10 矩阵中的岛屿](img/Figure_12.8_B15403.jpg)

图 12.8 - 10x10 矩阵中的岛屿

为了找到岛屿，我们必须遍历矩阵。换句话说，我们必须遍历矩阵的每个单元格。由于一个单元格由行（我们将其表示为*r*）和列（我们将其表示为*c*）来表示，我们观察到，从一个单元格(*r, c*)，我们可以朝八个方向移动：(*r-*1*, c-*1), (*r-*1*, c*), (*r-*1*, c+*1), (*r, c-*1), (*r, c+*1), (*r+*1*, c-*1), (*r+*1*, c*), 和 (*r+*1*, c+*1)。这意味着从当前单元格(*r, c*)，我们可以移动到(*r+ROW*[*k*]*, c+COL*[*k*])，只要`ROW`和`COL`是下面的数组，且 0 ≤ *k* ≤ 7：

```java
// top, right, bottom, left and 4 diagonal moves
private static final int[] ROW = {-1, -1, -1, 0, 1, 0, 1, 1};
private static final int[] COL = {-1, 1, 0, -1, -1, 1, 0, 1};
```

只要我们做到以下几点，移动到一个单元格就是有效的：

+   不要从网格上掉下来。

+   踩在代表陆地的单元格上（一个 1 的单元格）。

+   没有在该单元格之前。

为了确保我们不多次访问同一个单元格，我们使用一个布尔矩阵表示为`flagged[][]`。最初，这个矩阵只包含`false`的值，每次我们访问一个单元格(`r`, `c`)时，我们将相应的`flagged[r][c]`翻转为`true`。

以下是代码形式中的前三个要点：

```java
private static booleanisValid(int[][] matrix, 
      int r, int c, boolean[][] flagged) {
  return (r >= 0) && (r < flagged.length)
    && (c >= 0) && (c < flagged[0].length)
    && (matrix[r][c] == 1 && !flagged[r][c]);
}
```

到目前为止，我们知道如何决定从当前单元格移动到另一个单元格（从八个可能的移动中）。此外，我们必须定义一个算法来确定移动模式。我们知道从一个单元格(*r, c*)，我们可以在相邻单元格中的八个方向移动。因此，最方便的算法是尝试从当前单元格移动到所有有效的邻居，如下所示：

1.  从一个空队列开始。

1.  移动到一个有效的单元格(*r, c*)，将其入队，并标记为已访问 - 起始点应该是单元格(0, 0)。

1.  出队当前单元并解决其周围的八个相邻单元 - 解决单元意味着如果有效则将其入队并标记为已访问。

1.  重复*步骤 3*直到队列为空。当队列为空时，这意味着我们找到了一个岛屿。

1.  重复从*步骤 2*直到没有更多有效单元格。

在代码方面，我们有以下内容：

```java
private static class Cell {
  int r, c;
  public Cell(int r, int c) {
    this.r = r;
    this.c = c;
  }
}
// there are 8 possible movements from a cell    
private static final int POSSIBLE_MOVEMENTS = 8;
// top, right, bottom, left and 4 diagonal moves
private static final int[] ROW = {-1, -1, -1, 0, 1, 0, 1, 1};
private static final int[] COL = {-1, 1, 0, -1, -1, 1, 0, 1};
public static int islands(int[][] matrix) {
  int m = matrix.length;
  int n = matrix[0].length;
  // stores if a cell is flagged or not
  boolean[][] flagged = new boolean[m][n];
  int island = 0;
  for (int i = 0; i < m; i++) {
    for (int j = 0; j < n; j++) {
      if (matrix[i][j] == 1 && !flagged[i][j]) {
        resolve(matrix, flagged, i, j);
        island++;
      }
    }
  }
  return island;
}
private static void resolve(int[][] matrix, 
        boolean[][] flagged, int i, int j) {
  Queue<Cell> queue = new ArrayDeque<>();
  queue.add(new Cell(i, j));
  // flag source node
  flagged[i][j] = true;
  while (!queue.isEmpty()) {
    int r = queue.peek().r;
    int c = queue.peek().c;
    queue.poll();
    // check for all 8 possible movements from current 
    // cell and enqueue each valid movement
    for (int k = 0; k < POSSIBLE_MOVEMENTS; k++) {
      // skip this cell if the location is invalid
      if (isValid(matrix, r + ROW[k], c + COL[k], flagged)) {
        flagged[r + ROW[k]][c + COL[k]] = true;
        queue.add(new Cell(r + ROW[k], c + COL[k]));
      }
    }
  }
}
```

完整的应用程序称为*QueueIslands*。

## 编码挑战 11-最短路径

**亚马逊**，**谷歌**，**Adobe**

**问题**：假设给定一个只包含 0 和 1 的矩阵*m* x *n*。按照惯例，1 表示安全土地，而 0 表示不安全的土地。更准确地说，0 表示不应该被激活的传感器。此外，所有八个相邻的单元格都可以激活传感器。编写一小段代码，计算从第一列的任何单元格到最后一列的任何单元格的最短路径。您只能一次移动一步；向左、向右、向上或向下。结果路径（如果存在）应只包含值为 1 的单元格。

**解决方案**：让我们想象一个测试案例。以下是一个 10 x 10 的矩阵。

在下图的左侧，您可以看到给定的矩阵。请注意，值为 0 表示不应该被激活的传感器。在右侧，您可以看到应用程序使用的矩阵和可能的解决方案。这个矩阵是通过扩展传感器的覆盖区域从给定的矩阵中获得的。请记住，传感器的八个相邻单元格也可以激活传感器。解决方案从第一列（单元格（4,0））开始，以最后一列（单元格（9,9））结束，并包含 15 个步骤（从 0 到 14）。您可以在下图中看到这些步骤：

![图 12.9 - 给定矩阵（左侧）和解析矩阵（右侧）](img/Figure_12.9_B15403.jpg)

图 12.9 - 给定矩阵（左侧）和解析矩阵（右侧）

从坐标（*r，c*）的安全单元格，我们可以朝四个安全方向移动：（*r*-1*，c*），（*r，c*-1），（*r*+1*，c*）和（*r，c*+1）。如果我们将可能的移动视为方向（边）并将单元格视为顶点，则可以在图的上下文中可视化这个问题。边是可能的移动，而顶点是我们可以到达的可能单元格。每次移动都保持从当前单元格到起始单元格的距离（起始单元格是第一列的单元格）。对于每次移动，距离增加 1。因此，在图的上下文中，问题可以简化为在图中找到最短路径。因此，我们可以使用**广度优先搜索（BFS）**方法来解决这个问题。在*第十三章**，树和图*中，您已经了解了 BFS 算法的描述，并且另一个问题也是以与此处解决的问题相同的方式解决的- *国际象棋骑士*问题。

现在，根据前面问题提供的经验，我们可以详细说明这个算法：

1.  从一个空队列开始。

1.  将第一列的所有安全单元格入队，并将它们的距离设置为 0（这里，0 表示每个单元格到自身的距离）。此外，这些单元格被标记为已访问或标记。

1.  只要队列不为空，执行以下操作：

a. 弹出表示队列顶部的单元格。

b. 如果弹出的单元格是目的地单元格（即在最后一列），则简单地返回其距离（从目的地单元格到第一列源单元格的距离）。

c. 如果弹出的单元格不是目的地，则对该单元格的四个相邻单元格中的每一个，将每个有效单元格（安全且未访问）入队到队列中，并标记为已访问。

d. 如果我们处理了队列中的所有单元格但没有到达目的地，则没有解决方案。返回-1。

由于我们依赖 BFS 算法，我们知道所有最短路径为 1 的单元格首先被访问。接下来，被访问的单元格是具有最短路径为 1+1=2 等的相邻单元格。因此，具有最短路径的单元格等于其父级的最短路径+1。这意味着当我们第一次遍历目标单元格时，它给出了我们的最终结果。这就是最短路径。让我们看看代码中最相关的部分：

```java
private static int findShortestPath(int[][] board) {
  // stores if cell is visited or not
  boolean[][] visited = new boolean[M][N];
  Queue<Cell> queue = new ArrayDeque<>();
  // process every cell of first column
  for (int r1 = 0; r1 < M; r1++) {
    // if the cell is safe, mark it as visited and
    // enqueue it by assigning it distance as 0 from itself
    if (board[r1][0] == 1) {
      queue.add(new Cell(r1, 0, 0));
      visited[r1][0] = true;
    }
  }
  while (!queue.isEmpty()) {
    // pop the front node from queue and process it
    int rIdx = queue.peek().r;
    int cIdx = queue.peek().c;
    int dist = queue.peek().distance;
    queue.poll();
    // if destination is found then return minimum distance
    if (cIdx == N - 1) {
      return (dist + 1);
    }
    // check for all 4 possible movements from 
    // current cell and enqueue each valid movement
    for (int k = 0; k < 4; k++) {
      if (isValid(rIdx + ROW_4[k], cIdx + COL_4[k])
            && isSafe(board, visited, rIdx + ROW_4[k], 
                cIdx + COL_4[k])) {
        // mark it as visited and push it into 
        // queue with (+1) distance
        visited[rIdx + ROW_4[k]][cIdx + COL_4[k]] = true;
        queue.add(new Cell(rIdx + ROW_4[k], 
          cIdx + COL_4[k], dist + 1));
      }
    }
  }
  return -1;
}
```

完整的应用程序称为*ShortestSafeRoute*。

# 中缀、后缀和前缀表达式

前缀、后缀和中缀表达式在当今并不是一个非常常见的面试话题，但它可以被认为是任何开发人员至少应该涵盖一次的一个话题。以下是一个快速概述：

+   **前缀表达式**：这是一种表示法（代数表达式），用于编写算术表达式，其中操作数在其运算符之后列出。

+   **后缀表达式**：这是一种表示法（代数表达式），用于编写算术表达式，其中操作数在其运算符之前列出。

+   **中缀表达式**：这是一种表示法（代数表达式），通常用于算术公式或语句中，其中运算符写在其操作数之间。

如果我们有三个运算符 a、b 和 c，我们可以写出下图中显示的表达式：

![图 12.10 - 中缀、后缀和前缀](img/Figure_12.10_B15403.jpg)

图 12.10 - 中缀、后缀和前缀

最常见的问题涉及评估前缀和后缀表达式以及在前缀、中缀和后缀表达式之间进行转换。所有这些问题都有依赖于堆栈（或二叉树）的解决方案，并且在任何专门致力于基本算法的严肃书籍中都有涵盖。花些时间，收集一些关于这个主题的资源，以便熟悉它。由于这个主题在专门的书籍中得到了广泛的涵盖，并且在面试中并不常见，我们将不在这里进行涵盖。

# 摘要

本章涵盖了任何准备进行 Java 开发人员技术面试的候选人必须了解的堆栈和队列问题。堆栈和队列出现在许多实际应用中，因此掌握它们是面试官将测试您的顶级技能之一。

在下一章《树、Trie 和图形》中，您将看到堆栈和队列经常用于解决涉及树和图形的问题，这意味着它们也值得您的关注。
