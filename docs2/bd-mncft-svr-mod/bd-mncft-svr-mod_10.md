# 第十章. Bukkit 调度器

Bukkit 调度器是一个非常强大的工具，学习如何使用它也很容易。它允许你创建重复的任务，例如保存数据。它还允许你延迟执行代码块的时间。Bukkit 调度器还可以用于异步计算长时间的任务。这意味着像将数据写入文件或下载文件到服务器这样的任务可以调度在单独的线程上运行，以防止主线程，从而防止游戏卡顿。在本章中，你将通过继续在 `Warper` 传送插件上工作，以及创建一个名为 `AlwaysDay` 的新插件来学习如何完成这些任务。这个新插件将通过反复将时间设置为中午来确保服务器始终是白天。本章将涵盖以下主题：

+   创建 `BukkitRunnable` 类

+   理解同步和异步任务及其使用时机

+   从 `BukkitRunnable` 类运行任务

+   从 `BukkitRunnable` 类调度延迟任务

+   从 `BukkitRunnable` 类调度重复任务

+   编写一个名为 `AlwaysDay` 的插件，该插件使用重复任务

+   将延迟任务添加到 `Warper` 插件中

+   异步执行代码

# 创建 BukkitRunnable 类

我们将首先创建 `AlwaysDay` 插件。我们将为这个插件编写的代码将放在 `onEnable` 方法中。创建调度任务的第一步是创建一个 `BukkitRunnable` 类。这个类将包含非常少的代码行。因此，没有必要为它创建一个全新的 Java 文件。因此，我们将在 `onEnable` 方法中创建一个类。这可以通过以下代码行完成：

```java
BukkitRunnable runnable = new BukkitRunnable();
```

通常，这段代码是有效的，因为你正在构造一个新类的实例。然而，`BukkitRunnable`是一个**抽象类**，这意味着它不能被实例化。抽象类的目的是提供一些基础代码，其他类可以**扩展**并在此基础上构建。一个例子是`JavaPlugin`类。对于你创建的每个插件，你都是从扩展`JavaPlugin`的类开始的。这允许你覆盖方法，例如`onEnable`，同时保留其他方法，如`getConfig`的当前代码。这与实现接口，如 Listener 类似。抽象类和接口之间的区别在于其目的。如前所述，抽象类是其他类扩展的基础。接口更像是一个框架，类可以在其中实现。接口不包含任何方法中的代码，因此，接口中的所有方法都必须有实现。对于抽象类，只有定义为`abstract`的方法必须被覆盖，因为它们不包含代码。因此，因为`BukkitRunnable`是一个抽象类，你会收到一个警告，要求你实现所有抽象方法。NetBeans 可以自动为你添加所需的方法。为你添加的新方法是`run`。当调度器运行你的任务时，将调用此方法。对于新的`AlwaysDay`插件，我们希望任务将每个世界的时间设置为中午，如下所示：

```java
BukkitRunnable runnable = new BukkitRunnable() {
  @Override
  public void run() {
    for (World world : Bukkit.getWorlds()) {
      //Set the time to noon
      world.setTime(6000);
    }
  }
};
```

### 注意

记住，在 Minecraft 服务器上，时间是以 tick 来衡量的。20 tick 等于 1 秒。tick 的测量方法如下：

0 ticks: 黎明

6,000 ticks: 中午

12,000 ticks: 黄昏

18,000 ticks: 午夜

查看关于`BukkitRunnable`类的 API 文档[`hub.spigotmc.org/javadocs/spigot/org/bukkit/scheduler/BukkitRunnable.html`](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/scheduler/BukkitRunnable.html)。注意，有六种运行此任务的方法，如下所示：

+   runTask

+   runTaskAsynchronously

+   runTaskLater

+   runTaskLaterAsynchronously

+   runTaskTimer

+   runTaskTimerAsynchronously

# 同步任务与异步任务

一个任务可以是同步执行或异步执行。简单来说，当一个同步任务执行时，它必须在服务器继续正常运行之前完成。异步任务可以在服务器继续运行的同时在后台运行。如果一个任务以任何方式访问 Bukkit API，那么它应该以同步方式运行。因此，你很少会以异步方式运行任务。异步任务的一个优点是它可以在不导致服务器卡顿的情况下完成。例如，将数据写入保存文件可以异步执行。在本章的后面，我们将修改`Warper`插件以异步保存其数据。至于`AlwaysDay`插件，我们必须以同步方式运行任务，因为它访问了 Minecraft 服务器。

## 从`BukkitRunnable`类运行任务

在`BukkitRunnable`类上调用`runTask`或`runTaskAsynchronously`会导致任务立即运行。你可能会使用这种情况是在从异步上下文运行同步任务或反之亦然。

## 从`BukkitRunnable`类稍后运行任务

在`BukkitRunnable`类上调用`runTaskLater`或`runTaskLaterAsynchronously`将延迟任务执行特定的时间。这个时间是以刻度为单位的。记住，每秒有 20 个刻度。在`Warper`插件中，我们将添加一个传送延迟，使玩家在运行传送命令后 5 秒被传送。我们将通过稍后运行任务来实现这一点。

## 从`BukkitRunnable`类运行任务计时器

在`BukkitRunnable`类上调用`runTaskTimer`或`runTaskTimerAsynchronously`会使任务每隔指定的时间间隔重复执行。任务会一直重复，直到被取消。任务计时器也可以延迟启动，以调整任务的初始运行。任务计时器可以用来定期保存数据，但到目前为止，我们将使用这种重复任务来完成`AlwaysDay`插件。

# 为插件编写重复任务

我们已经有了`BukkitRunnable`类。因此，为了运行任务计时器，我们只需要确定任务的延迟和延迟周期。我们希望延迟为 0。这样，如果插件启用时是夜晚，时间会立即设置为中午。至于周期，如果我们想保持太阳始终在正上方，我们可以每秒重复任务。任务只包含一行简单的代码。频繁重复它不会导致服务器卡顿。然而，每分钟重复任务仍然可以防止世界永远变暗，并且对计算机的压力会更小。因此，我们将任务延迟 0 个刻度，并每 1,200 个刻度重复一次。这导致以下代码行：

```java
runnable.runTaskTimer(this, 0, 1200);
```

通过这种方式，我们启动了一个重复任务。当插件禁用时取消重复任务是良好的实践。为了完成这个任务，我们将`BukkitTask`存储为类变量，这样我们就可以稍后访问它来禁用它。一旦你在`onDisable`方法中取消了任务，以下代码给出了整个`AlwaysDay`插件：

```java
package com.codisimus.alwaysday;

import org.bukkit.Bukkit;
import org.bukkit.World;
import org.bukkit.plugin.java.JavaPlugin;
import org.bukkit.scheduler.BukkitRunnable;
import org.bukkit.scheduler.BukkitTask;

public class AlwaysDay extends JavaPlugin {
    BukkitTask dayLightTask;

    @Override
    public void onDisable() {

        dayLightTask.cancel();

    }

  @Override
  public void onEnable() {
    BukkitRunnable bRunnable = new BukkitRunnable() {
      @Override
      public void run() {
        for (World world : Bukkit.getWorlds()) {
          //Set the time to noon
          world.setTime(6000);
        }
      }
    };

    //Repeat task every 1200 ticks (1 minute)
    dayLightTask = bRunnable.runTaskTimer(this, 0, 1200);
  }
}
```

# 向插件中添加延迟任务

现在，我们将向`Warper`插件添加一个延迟。这将要求玩家在运行传送或回家命令后保持静止。如果他们移动太多，传送任务将被取消，他们不会传送。这将防止玩家在有人攻击他们或他们正在坠落时传送。

如果你还没有做，请在`main`类中添加一个`warpDelay`变量。这将在以下代码行中给出：

```java
static int warpDelay = 5; //in seconds
```

这个时间将以秒为单位。我们将在代码中稍后将其乘以 20，以计算我们希望延迟任务的刻度数。

我们还需要跟踪正在传送过程中的玩家，以便我们可以检查他们是否在移动。为当前的`warpers`添加另一个变量。这将是一个`HashMap`，这样我们就可以跟踪哪些玩家正在传送以及将要运行的传送任务。这样，如果特定玩家移动，我们可以获取他们的任务并取消它。这可以在以下代码中看到：

```java
private static HashMap<String, BukkitTask>warpers = new HashMap<>();//Player UUID -> Warp Task
```

代码包含三个新方法，必须将它们添加到`main`类中，以便安排传送任务、检查玩家是否有传送任务以及取消玩家的传送任务。代码如下：

```java
/**
 * Schedules a Player to be teleported after the delay time
 *
 * @param player The Player being teleported
 * @param loc The location of the destination
 */
public static void scheduleWarp(final Player player, final Location loc) {
  //Inform the player that they will be teleported
  player.sendMessage("You will be teleported in "+ warpDelay + " seconds");

  //Create a task to teleport the player
  BukkitRunnable bRunnable = new BukkitRunnable() {
    @Override
    public void run() {
      player.teleport(loc); 

      //Remove the player as a warper because they have already beenteleported
      warpers.remove(player.getName());
    }
  };

  //Schedule the task to run later
  BukkitTask task = bRunnable.runTaskLater(plugin, 20L * warpDelay);

  //Keep track of the player and their warp task
  warpers.put(player.getUniqueId().toString(), task);
}

/**
 * Returns true if the player is waiting to be teleported
 *
 * @param player The UUID of the Player in question
 * @return true if the player is waiting to be warped
 */
public static boolean isWarping(String player) {
  return warpers.containsKey(player);
}

/**
 * Cancels the warp task for the given player
 *
 * @param player The UUID of the Player whose warp task will be canceled
 */
public static void cancelWarp(String player) {
  //Check if the player is warping
  if (isWarping(player)) {
    //Remove the player as a warper
    //Cancel the task so that the player is not teleported
    warpers.remove(player).cancel();
  }
}
```

在`scheduleTeleportation`方法中，请注意`player`和`loc`变量都是`final`的。这是在`BukkitRunnable`类中使用变量所必需的。这样做是为了确保在任务运行之前，值不会改变。此外，请注意`runTaskLater`方法调用返回一个`BukkitTask`，这是我们保存在`HashMap`中的内容。你可以通过查看`cancelWarp`方法来了解原因，该方法在执行之前移除指定玩家的`BukkitTask`并对其调用`cancel`方法。

在`WarpCommand`和`HomeCommand`类中，我们都会传送玩家。我们希望移除那行代码，并用对`scheduleTeleportation`方法的调用来替换它。功能添加即将完成。剩下要做的就是当`warper`移动时调用`cancelWarp`方法。为此，添加一个事件监听器来监听`player move`事件。这可以在以下代码中看到：

```java
package com.codisimus.warper;

import org.bukkit.block.Block;
import org.bukkit.entity.Player;
import org.bukkit.event.EventHandler;
import org.bukkit.event.EventPriority;
import org.bukkit.event.Listener;
import org.bukkit.event.player.PlayerMoveEvent;

public class WarperPlayerListener implements Listener {
  @EventHandler (priority = EventPriority.MONITOR)
  public void onPlayerMove(PlayerMoveEvent event) {
    Player player = event.getPlayer();
    String playerUuid = player.getUniqueId().toString();

    //We only care about this event if the player is flagged aswarping
    if (Warper.isWarping(playerUuid)) {
      //Compare the block locations rather than the player locations
      //This allows a player to move their head without canceling thewarp
      Block blockFrom = event.getFrom().getBlock();
      Block blockTo = event.getTo().getBlock();

      //Cancel the warp if the player moves to a different block
      if (!blockFrom.equals(blockTo)) {
        Warper.cancelWarp(playerUuid);
        player.sendMessage("Warping canceled because you moved!");
      }
    }
  }
}
```

不要忘记在`onEnable`方法中注册事件。

# 异步执行代码

我们可以通过异步将数据写入文件来改进`Warper`插件。这将有助于保持服务器主线程的流畅运行，不会出现延迟。

看一下当前的`save`方法。我们将数据添加到`YamlConfiguration`文件中，然后将配置写入文件。整个方法不能异步运行。将数据添加到配置必须同步进行，以确保在添加过程中不会被修改。然而，对配置的`save`方法调用可以是异步的。我们将整个`try/catch`块放在一个新的`BukkitRunnable`类中。然后我们将它作为一个任务异步运行。这个任务将被保存在`Warper`类的静态变量中。这可以在以下代码中看到：

```java
BukkitRunnable saveRunnable = new BukkitRunnable() {
  @Override
  public void run() {
    try {
      //Write the configuration to our save file
      config.save(new File(plugin.getDataFolder(), "warps.yml"));
    } catch (Exception saveFailed) {
      plugin.getLogger().log(Level.SEVERE, "Save Failed!", saveFailed);
    }
  }
};

saveTask = saveRunnable.runTaskAsynchronously(plugin);
```

现在，在数据保存期间，服务器的其余部分可以继续运行。

然而，如果我们尝试在之前的写入尚未完成时再次保存文件怎么办？在这种情况下，我们不在乎之前的任务，因为它现在正在保存过时的数据。我们将在创建`BukkitRunnable`类之前首先取消任务，然后开始一个新的任务。这将在以下代码中完成：

```java
if (saveTask != null) {
  saveTask.cancel();
}
```

这完成了`Warper`的此版本。如第九章中提到的，*保存你的数据*，这个插件有很多功能添加的潜力。你现在有了添加这些功能所需的知识。

# 摘要

你现在熟悉了 Bukkit API 的大部分复杂方面。有了这些知识，你可以编写几乎任何类型的 Bukkit 插件。尝试通过创建一个新的插件来应用这些知识。你可能尝试编写一个公告插件，该插件将在服务器上循环播放需要广播的消息列表。考虑所有 Bukkit API 概念以及你如何使用它们来为插件添加新功能。例如，使用公告插件，你可以做以下事情：

+   添加命令，允许管理员添加需要宣布的消息

+   添加权限来控制谁可以添加消息，甚至谁可以看到宣布的消息

+   添加一个`EventHandler`方法来监听玩家登录时，以便可以向他们发送消息

+   添加一个`config.yml`文件来设置消息应宣布的频率

+   添加一个保存文件来保存和加载将要宣布的所有消息

+   使用 Bukkit 调度器在服务器运行时重复广播消息

对于你制作的任何插件，考虑 Bukkit API 的每个部分，以找出通过添加更多功能来改进插件的方法。这无疑会使你的插件和服务器脱颖而出。

本书没有讨论一些主题，但它们足够简单；你可以通过阅读 API 文档自学如何使用它们。一些可以美化 Bukkit 插件的有趣功能是`playSound`和`playEffect`方法，它们位于`World`和`Player`类中。我鼓励你阅读它们并尝试自己使用。

你已经知道如何编写插件命令、玩家权限、事件监听器、配置文件、数据的保存和加载以及计划任务。剩下的事情就是想象如何使用这些新技能为 Bukkit 服务器创建一个伟大且独特的插件。
