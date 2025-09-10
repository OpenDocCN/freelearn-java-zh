# 第六章：玩家权限

玩家权限是几乎每个 Bukkit 服务器管理员都希望在他们的服务器上拥有的一个功能。在原版的 Minecraft 中，你要么是**OP**（**操作员**），要么只是一个普通玩家。有了权限，你可以在两者之间创建无限数量的等级。有几个权限插件可以在 Bukkit 或 Spigot 网站上找到。在过去，开发者必须编写自己的代码来支持一个或多个这些权限系统。幸运的是，Bukkit API 现在为玩家权限提供了一个基础，这使得我们的工作变得容易。我们不再需要为每个存在的权限插件学习一个新的 API。我们只需要支持 Bukkit 的通用权限系统，我们可以确信它不会在任何时候发生剧烈变化。在本章中，你将做这件事，并安装一个权限插件，帮助你组织每个玩家的权限。到本章结束时，你将能够以确保不受信任的玩家不会破坏其他人的乐趣的方式来控制你的服务器。在本章中，我们将涵盖以下主题：

+   在你的服务器和插件中使用权限的好处

+   权限节点是什么以及开发者和服务器管理员如何使用它

+   将权限节点添加到`plugin.yml`文件中

+   将权限节点分配给你的插件的一个命令

+   在游戏中测试玩家权限

+   安装和配置第三方权限插件

+   在你的插件中使用权限节点

# 权限的好处

**权限**让你对你的服务器上的玩家有更多的控制。它们允许你防止不受信任的玩家滥用。有了权限，你可以根据玩家在服务器中的角色和他们的可信度给予每个玩家一个特定的等级。比如说，你想要给某个特定的玩家赋予将其他玩家位置传送到自己位置的能力。有了权限，你可以这样做，而不必给那个玩家赋予生成物品、踢出/封禁其他玩家，甚至完全停止服务器的能力！一个有用的权限的最简单例子就是不给新玩家建造权限。这阻止了有人登录你的服务器，仅仅是为了破坏世界。他们将无法破坏你或其他玩家的建筑。

当编程插件时，你可以将某些权限分配给特定的命令或操作。这允许你只将插件的好处给予有特权的人。例如，你可能只想让你的好朋友和你自己有使用`enchant`命令对物品进行附魔的选项。完成这一步骤的第一步是了解权限节点是什么以及它们是如何工作的。

# 理解权限节点

**权限节点**是一个包含多个单词并由点号分隔的`string`。这些权限节点被赋予玩家，以在服务器上给予他们特殊的特权。一个例子是`minecraft.command.give`，这是执行`give`命令所需的权限节点。如您所见，它可以分解为三个部分，即创建者（Minecraft）、类别（命令）和特定特权（`give`命令）。您会发现大多数权限节点都是这样结构的。对于插件，其权限节点以插件的名称开头。这有助于防止节点冲突。如果两个插件使用相同的权限节点，那么管理员无法限制对一个节点的访问而不限制另一个节点。您还会发现许多插件的权限节点只有两个单词长。这是在插件没有很多权限时进行的。因此，不需要类别。另一方面，对于大型插件，您可能希望包含许多嵌套类别。

为了帮助您理解权限节点，我们将为`Enchanter`插件创建一个权限节点。权限节点的第一个单词将是插件的名称，而第二个单词将是命令的名称。如果权限节点直接与特定命令相关，那么在权限节点中使用命令名称是明智的。这将使您的权限易于理解和记忆。`enchant`命令的权限节点将是`enchanter.enchant`。如果我们预计这个插件有多个权限，那么我们可以使用`enchanter.command.enchant`。这两个权限节点都可以，但我们将使用前者作为示例。请注意，大多数开发者倾向于将权限节点保持为小写。这是可选的，但通常可以防止在稍后输入节点时出错。一旦我们确定了权限节点，我们必须将其添加到`plugin.yml`文件中，以便与插件一起使用。

# 将权限节点添加到 plugin.yml

在`Enchanter`项目中打开`plugin.yml`文件。添加权限节点的方式与添加命令类似。在新的一行中，添加`permissions:`。确保这一行没有任何缩进。在随后的行中，添加我们插件将使用的每个权限节点，后面跟一个冒号。接下来的几行将提供权限的属性，例如其描述。以下代码是添加了`enchant`权限节点后的`plugin.yml`文件示例。请确保缩进相似。请注意，版本属性也应更新，以表明这是一个新的、改进的`Enchanter`插件版本：

```java
name: Enchanter
version: 0.2
main: com.codisimus.enchanter.Enchanter
description: Used to quickly put enchantments on an item
commands:
  enchant:
    aliases: e
    description: Adds enchantments to the item in your hand
    usage: Hold the item you wish to enchant and type /enchant
permissions:
  enchanter.enchant:
    description: Needed to use the enchant command
    default: op
```

默认属性可以设置为`true`、`false`、`op`或`not op`。这决定了谁将拥有这个权限；`true`表示每个人都将拥有这个权限，`false`表示没有人将拥有它，`op`表示只有操作员将拥有它，而`not op`表示除了操作员之外的所有人将拥有它。可以通过使用权限插件进一步修改拥有这个权限的人，这将在本章后面讨论。

就像命令一样，你可以给插件分配多个权限。有关`plugin.yml`文件的更多信息，请访问[`wiki.bukkit.org/Plugin_YAML`](http://wiki.bukkit.org/Plugin_YAML)。

# 将权限节点分配给插件命令

现在我们已经创建了权限节点，我们希望阻止没有`enchanter.enchant`节点的玩家使用`enchant`命令。这个过程很简单，因为它只需要在`plugin.yml`文件中添加几行。

对于`enchant`命令，我们将添加两个属性，即`permission`和`permission-message`。`permission`属性简单地表示执行命令所需的权限节点。`permission-message`属性是一个消息，如果玩家没有必要的权限，他们将看到这个消息。在这些添加之后，`plugin.yml`文件将看起来像这样：

```java
name: Enchanter
version: 0.2
main: com.codisimus.enchanter.Enchanter
description: Used to quickly put enchantments on an item
commands:
  enchant:
    aliases: [e]
    description: Adds enchantments to the item in your hand
    usage: Hold the item you wish to enchant and type /enchant
    permission: enchanter.enchant
    permission-message: You do not have permission to enchant items
permissions:
  enchanter.enchant:
    description: Needed to use the enchant command
    default: op
```

您可能想给权限消息添加颜色。这可以通过使用`§`符号来完成。这是 Minecraft 用来指示颜色代码的字符。通过按住*Alt*键同时按下*2*然后*1*可以轻松地输入这个符号。所有颜色及其对应代码的列表可以在[`www.minecraftwiki.net/wiki/Formatting_codes`](http://www.minecraftwiki.net/wiki/Formatting_codes)找到。带有颜色支持的`permissions-message`行的示例如下：

```java
permission-message: §4You do not have permission to §6enchant items
```

![将权限节点分配给插件命令](img/00037.jpeg)

# 测试玩家权限

您可以通过构建`jar`文件并在您的服务器上安装它来测试插件的新增功能，如第四章所述，在 Spigot 服务器上测试。确保您重新加载或重启服务器，以便使用插件的新版本。请记住，当插件启用时，版本号会在控制台上打印出来。

通过在您的服务器上进行测试，您会发现您可以通过插件来附魔物品。由于您是 OP，您默认拥有`enchanter.enchant`节点。通过以下控制台命令来*取消 OP*自己：

```java
>deop Codisimus
```

现在，您将无法再使用`/enchant`命令。

# 使用第三方权限插件

你很可能会在服务器上有一些值得信赖的玩家，你希望与他们分享`/enchant`命令的使用。然而，这些玩家还没有足够信任到可以成为 OP。为了共享这个命令的使用，你需要使用权限插件。权限插件将允许你创建多个玩家组。每个组将分配不同的权限。然后，每个在服务器上玩游戏的玩家都可以被分配到特定的组。例如，你可以有四个权限组，即*default*、*trusted*、*mod*和*admin*。*default*组将拥有基本权限。新加入服务器的玩家将被放入*default*组。*trusted*组将拥有更多一些的特权。他们将能够访问特定的命令，例如设置服务器世界的白天时间以及传送玩家。*mod*组代表“管理员”，它将能够访问许多其他命令，例如踢出或禁止玩家。最后，`admin`组代表“管理员”，它将拥有`/give`命令和`/enchant`命令。

在[dev.bukkit.org](http://dev.bukkit.org)上可以找到几个权限插件。每个权限插件都是由不同的开发者创建的。它们具有各种功能，这取决于开发者如何编程。今天使用的许多流行权限插件实际上是在权限添加到 API 之前创建的。因此，它们可能无法利用 Bukkit 的所有功能。它们还包括一些不再需要的附加功能，例如权限组。我们将使用的插件是我自己开发的，名为`CodsPerms`。`CodsPerms`是一个简单且基本的权限插件。因为`CodsPerms`遵循 Bukkit API 的规则，所以在本章中你将学习的组配置也可以用于其他权限插件。有关如何下载`CodsPerms`的说明可以在[`codisimus.com/codsperms`](http://codisimus.com/codsperms)找到。

一旦你有了插件的`jar`文件，就像安装你自己的插件一样在你的服务器上安装它。安装插件后，`permission`命令将可供你使用。执行`/perm`命令将告诉你现在可供使用的各种命令。

### 小贴士

你需要拥有`permissions.manage`节点才能使用权限命令。在我们完全设置权限插件之前，你可以要么从控制台运行这些命令，要么给自己分配 OP 状态。

你会发现有一些命令可以用来给玩家分配权限节点以及移除它们。如果你只想添加单个节点，比如给自己分配`permissions.manage`节点，这将很有用，但你可能不希望为所有加入你服务器的玩家使用这些命令。为了解决这个问题，我们将配置之前提到的组。

这些组将被创建为一个包含几个其他子权限节点的权限节点。这将允许我们给一个玩家一个单独的组节点，然后他们将继承其所有子节点。我们可以在位于 `root` 目录（即你放置 `spigot.jar` 的同一个文件夹）中的 `permissions.yml` 文件内创建这些父节点。`permissions.yml` 文件是一个 `YAML` 文件，就像 `plugin.yml`。因此，你应该熟悉其格式。你可以使用文本编辑器编辑此文件。如果你希望使用 NetBeans，你可以通过导航到 **文件** | **打开文件…** 或通过将文件拖放到 NetBeans 窗口中来打开文件。

### 小贴士

错误地编辑 `YAML` 文件会导致它无法完全加载。你很可能会遇到的问题是在你的文档中有一个 *制表符* 而不是 *空格*。这会导致你的文件无法正确加载。

以下代码是创建之前指定的组后 `permissions.yml` 可能看起来的一个示例：

```java
group.default:
  description: New Players who may have joined for the first time
  default: true
  children:
    minecraft.command.kill: true
    minecraft.command.list: true
group.trusted:
  description: Players who often play on the server
  default: false
  children:
    group.default: true
    minecraft.command.weather: true
    minecraft.command.time: true
    minecraft.command.teleport: true
group.mod:
  description: Players who moderate the server
  default: false
  children:
    group.trusted: true
    minecraft.command.ban: true
    minecraft.command.pardon: true
    minecraft.command.kick: true
group.admin:
  description: Players who administer on the server
  default: false
  children:
    group.mod: true
    minecraft.command.ban-ip: true
    minecraft.command.pardon-ip: true
    minecraft.command.gamerule: true
    minecraft.command.give: true
    minecraft.command.say: true
    permissions.manage: true
    enchanter.enchant: true
```

每个组都可以通过简单地将该组权限节点添加为其子节点之一来继承另一个组的权限节点。在这个例子中，`admin` 组继承了 `mod` 组的所有权限，`mod` 组继承了 `trusted` 组的所有权限，而 `trusted` 组继承了 `default` 组的所有权限。因此，`admin` 组也通过父级继承了 `default` 组的权限。在这个示例文件中，我们将 `group.default` 父节点设置为 `true`。这意味着服务器上的每个玩家都将自动拥有 `group.default` 权限节点。由于子节点，每个玩家也将拥有 `minecraft.command.kill` 和 `minecraft.command.list`。将权限添加到默认组将消除向每个加入你服务器的玩家分配权限的需要。

如你所见，之前的权限节点包括了某些 Minecraft 命令的权限以及 `Enchanter` 插件的权限。还有更多权限尚未列出。这些是一些常用的权限。Minecraft 和 Bukkit 命令的其余权限可以在 [wiki.bukkit.org/CraftBukkit_commands](http://wiki.bukkit.org/CraftBukkit_commands) 找到。

一旦你填充了权限 `YAML` 文件，你必须重新加载服务器才能使更改生效。现在，你可以将玩家分配到不同的组。使用以下命令并替换为你自己的用户名来将自己添加到受信任的组：

```java
>perm give Codisimus group.trusted
```

你将在`permissions.yml`文件中的`group.trusted`定义权限。尝试将自己放入不同的组中，并使用`/enchant`命令以及其他各种命令。确保你不是 OP，因为这会给你所有权限，无论你在哪个组中。此外，请注意，你必须手动从组中移除自己。如果一个`admin`组的玩家被添加到`trusted`组，他们仍然会保留管理员权限，直到他们从管理员组中移除。

# 在你的插件中使用权限节点

在某些情况下，你可能想在代码中检查玩家是否有特定的权限。随着 Bukkit 中通用权限系统的添加，这非常简单，无论你使用的是哪个权限插件。查看 Bukkit API 文档，你会看到`Player`对象包含一个`hasPermission`方法，它返回一个布尔响应。该方法需要一个`string`值，即正在检查的权限节点。我们可以将此方法放在一个`if`语句中，如下面的代码所示：

```java
if (player.hasPermission("enchanter.enchant")) {
  //Add a level 10 Knockback enchantment
  Enchantment enchant = Enchantment.KNOCKBACK;
  hand.addUnsafeEnchantment(enchant, 10);
  player.sendMessage("Your item has been enchanted!");
} else {
  player.sendMessage("You do not have permission to enchant");
}
```

这段代码对于插件来说是不必要的，因为 Bukkit 可以自动处理命令的玩家权限。为了了解如何正确使用，让我们回到`MyFirstBukkitPlugin`并添加一个权限检查。以下代码是修改后的`onEnable`方法，它只会向具有必要权限的玩家说`Hello`：

```java
@Override
public void onEnable() {
  if (Bukkit.getOnlinePlayers().size() >= 1) {
    for (Player player : Bukkit.getOnlinePlayers()) {
      //Only say 'Hello' to each player that has permission
      if (player.hasPermission("myfirstbukkitplugin.greeting")) {
        player.sendMessage("Hello " + player.getName());
      }
    }
  } else {
    //Say 'Hello' to the Minecraft World
    broadcastToServer("Hello World!");
  }
}
```

记住，你还需要修改`plugin.yml`来将权限节点添加到你的插件中。

你还可以向只有具有特定权限节点的玩家广播消息。有关此内容的文档可以在[`hub.spigotmc.org/javadocs/spigot/org/bukkit/Bukkit.html#broadcast(java.lang.String,%20java.lang.String)`](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/Bukkit.html#broadcast(java.lang.String,%20java.lang.String))找到。

尝试将一些权限节点添加到之前章节中创建的其他项目中。例如，将`creeperhiss.scare`权限节点添加到具有`/scare <player>`命令的插件中。作为一个额外的挑战，添加一个选项，允许玩家输入`/scare all`来吓唬服务器上的所有玩家。在这种情况下，你可以检查每个玩家的`creeperhiss.hear`权限节点。这样，只有那些玩家会听到声音。这是一个很好的例子，说明权限节点应该默认设置为`not op`。

# 摘要

现有的插件经过修改后，在权限插件的辅助下变得更加灵活。当你的服务器运行`CodsPerms`时，你可以为玩家设置多个组。你可以创建为特定玩家提供特权命令的插件，同时这些玩家将无法使用可能被滥用的命令。这种关于 Bukkit 权限的新知识将使你能够更好地控制你的插件和服务器。现在你已经学会了如何编程命令和权限，你就可以深入探索 Bukkit API 中更具挑战性和趣味性的部分了。在下一章中，你将学习如何通过使用 Bukkit 事件系统来自动化和定制你的服务器。
