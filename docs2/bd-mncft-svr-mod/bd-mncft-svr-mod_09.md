# 第九章：保存你的数据

Bukkit 插件有很多种类型。其中一些需要你保存数据。通过保存数据，我指的是将信息保存到系统的硬盘上。如果信息必须在服务器重启后保持完整，则需要这样做。到目前为止，我们创建的插件中还没有这个要求。以下是一些将保存数据的插件示例：

+   经济插件必须保存每个玩家有多少钱的信息

+   土地保护插件必须保存有关哪些地块已被声明及其所有者的信息

+   任务插件必须存储每个任务的所有信息，例如谁完成了它

在服务器关闭时，保存数据有无数用途。在本章中，我们将创建一个传送插件，将各种传送位置保存到文件中。同样，我们将这些位置保存到文件中，这样在服务器关闭后我们就不需要再次创建它们。你已经熟悉 YAML 文件格式。因此，我们将利用 YAML 配置来保存和加载数据。在本章中，我们将涵盖以下主题：

+   你可以保存的数据类型

+   值得保存的插件数据和保存频率

+   扩展预先编写的传送插件

+   创建和使用 `ConfigurationSerializable` 对象

+   在 YAML 配置中保存数据

+   从 YAML 配置文件中加载已保存的数据

# 可以保存的数据类型

你可能还记得，在上一章中讨论过，只有某些数据类型可以存储在 YAML 文件中。这些包括原始类型，如 `int` 和 `boolean`，字符串、列表以及实现 `ConfigurationSerializable` 的类型，如 `ItemStack`。

因此，我们只能保存这些特定类型的数据。

你可能希望保存其他类型的数据，例如 `Player` 对象，或者在传送插件的情况下，一个 `Location` 对象。这些可能不能直接存储，但通常可以分解以保存在以后加载时所需的重要值。例如，你不能保存 `Player` 对象，但你可以保存玩家的 **UUID**（**通用唯一标识符**），它可以被转换成字符串。每个 `Player` 都有一个 UUID。因此，这是我们后来能够引用该特定玩家的唯一信息。

### 小贴士

仅存储玩家名称不是一个充分的解决方案，因为提供的 Minecraft 账户名称可以更改。

`Location` 对象也不能直接存储，但它可以被分解为其世界、`x`、`y` 和 `z` 坐标、`yaw` 和 `pitch`。`x`、`y`、`z`、`yaw` 和 `pitch` 的值仅仅是可以存储的数字。至于世界，它也有一个永远不会改变的 UUID。因此，一个位置被分解为一个字符串（`world uuid`）、三个双精度浮点数（`x`、`y`、`z`）和两个浮点数（`yaw` 和 `pitch`）。

当你创建自己的插件时，你可能希望将某些类存储在文件中，例如一个 `BankAccount` 对象。如前所述，我们可以使用任何实现了 `ConfigurationSerializable` 的类来做到这一点。`ConfigurationSerializable` 意味着对象可以被转换成可以存储在配置中的形式。然后，这个配置可以被写入文件。在传送插件中，我们将创建一个 `location` 对象，它正好可以做到这一点。

# 保存哪些数据以及何时保存

我们知道可以保存到文件中的内容，但我们应该保存什么？将数据写入文件会占用磁盘空间。因此，我们只想保存我们需要的内容。最好是思考，“服务器关闭后，我想保留哪些信息？”例如，一个银行插件将希望保留每个账户的余额。作为另一个例子，一个 **PvP** 场地插件可能不会关心有关场地比赛的信息。在服务器关闭时，比赛很可能简单地被取消。在考虑传送插件时，我们希望服务器关闭后仍然保留每个传送门的位置。

我们接下来的关注点是何时保存这些信息。如果数据量很大，将数据写入文件可能会使服务器 **延迟**。如果你不熟悉“延迟”这个术语，它是一个用来表示服务器运行缓慢的短语。你知道这种情况发生时，因为游戏变得非常卡顿，玩家和怪物似乎四处移动。这对每个人来说都是一种不愉快的体验。因此，你只想在你必须的时候保存数据。保存数据频率有三个典型的选项：

+   每次数据被修改时

+   定期，例如每小时一次

+   当服务器/插件关闭时

这些选项按照它们的安全性排序。例如，如果你的数据仅在服务器关闭时保存，那么如果服务器崩溃，你将面临丢失未保存数据的风险。如果数据每小时保存一次，那么在最坏的情况下，你将丢失一小时的资料。因此，在可能的情况下，应始终使用第一个选项。第二个和第三个选项只有在插件处理大量数据且/或数据修改非常频繁时（例如每分钟修改几次）才应考虑。传送插件的数据只有在有人创建/删除传送门或设置他们的家传送门位置时才会被修改。因此，每次数据被修改时，我们将调用 `save` 方法。

# 一个示例传送插件

对于这个项目，你将得到一个不完整的传送插件。你已经知道如何编写这个项目的大部分代码。因此，我们只讨论以下三个主题：

+   创建一个实现 `ConfigurationSerializable` 的类

+   `save` 方法

+   `load` 方法

插件的其余部分已提供，可以从[www.packtpub.com](http://www.packtpub.com)下载，如前言中所述。你将工作的代码是插件 Warper 的 0.1 版本。通过插件并阅读注释，尝试理解它所做的一切。在这个项目中使用了`Maps`和`try/catch`块。如果你不知道它们是什么，那没关系。它们将在你需要使用它们的时候进行解释。请注意，`SerializableLocation`类是位置类，实现了`ConfigurationSerializable`；我们将在下面讨论这一点。

# 编写 ConfigurationSerializable 类

**序列化**是将数据或对象转换为可以写入文件的形式的过程。在 Warper 插件中，我们需要保存 Bukkit 位置信息。位置信息本身不能进行序列化。因此，我们将创建自己的类来保存 Bukkit `Location`对象的数据，并且能够将其转换为地图形式，以及从地图形式转换回来。如果你对地图还不熟悉，它们是一种非常有用的集合类型，我们将在整个项目中使用。地图有键和值。每个键指向一个特定的值。`Warper`插件是地图如何使用的良好示例。当进行传送时，玩家将通过名称选择一个特定的位置进行传送。如果所有传送位置都在一个列表中，我们就必须遍历列表，直到找到具有正确名称的传送位置。使用地图，我们可以将键（传送的名称）传递给地图，它将返回值（传送位置）。

创建一个名为`SerializableLocation`的新类，其中包含一个私有变量，用于保存 Bukkit `Location 对象`。第一个构造函数将需要一个`Location`对象。我们还将包括一个`getLocation`方法。以下是新类开头的外观代码：

```java
package com.codisimus.warper;

import org.bukkit.Location;

/**
 * A SerializableLocation represents a Bukkit Location object
 * This class is configuration serializable so that it may be
 * stored using Bukkit's configuration API
 */
public class SerializableLocation {
    private Location loc;

    public SerializableLocation(Location loc) {
        this.loc = loc;
    }

    /**
     * Returns the Location object in its full form
     *
     * @return The location of this object
     */
    public Location getLocation() {
        return loc;
    }
}
```

一旦你添加了`implements ConfigurationSerializable`，你的 IDE 应该会警告你关于实现所有抽象方法。你必须重写的方法是`serialize`。这个方法将返回你对象的地图表示形式。我们之前已经提到了我们需要的数据。现在，我们只需要给每份数据分配一个名称并将其放入地图中。要向地图添加数据，你可以调用`put`方法。这个方法需要两个参数，即键和键的值。键是数据的名称，它允许我们稍后引用它。值是可序列化的数据。要了解更多关于地图的信息，你可以阅读[`docs.oracle.com/javase/8/docs/api/java/util/Map.html`](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html)上的**Javadoc**。对于`serialize`方法，我们需要获取我们之前提到的所有数据并将其放入地图中，如下所示：

```java
/**
 * Returns a map representation of this object for use of serialization
 *
 * @return This location as a map of Strings to Objects
 */
@Override
public Map<String, Object> serialize() {
    Map map = new TreeMap();
    map.put("world", loc.getWorld().getUID().toString());
    map.put("x", loc.getX());
    map.put("y", loc.getY());
    map.put("z", loc.getZ());
    map.put("yaw", loc.getYaw());
    map.put("pitch", loc.getPitch());
    return map;
}
```

这处理了保存部分，但我们仍然需要处理加载部分。最简单的方法是添加一个接受地图作为参数的构造函数，如下所示：

```java
/**
 * This constructor is used by Bukkit to create this object
 *
 * @param map The map which matches the return value of the serialize() method
 */
public SerializableLocation(Map<String, Object> map) {
    //Check if the world for this location is loaded
    UUID uuid = UUID.fromString((String) map.get("world"));
    World world = Bukkit.getWorld(uuid);
    if (world != null) {
        //Each coordinate we cast to it's original type
        double x = (double) map.get("x");
        double y = (double) map.get("y");
        double z = (double) map.get("z");

        //Both yaw and pitch are loaded as type Double and then converted to float
        float yaw = ((Double) map.get("yaw")).floatValue();
        float pitch = ((Double) map.get("pitch")).floatValue();

        loc = new Location(world, x, y, z, yaw, pitch);
    } else {
        Warper.plugin.getLogger().severe("Invalid location, most likely due to missing world");
    }
}
```

加载基本上是保存的反面。我们从映射中提取每个值，然后使用它来创建 Bukkit 的`Location`对象。作为一个安全措施，我们首先会验证世界是否实际上已经加载。如果世界没有加载，位置将不存在。我们不希望插件因为这一点而崩溃。也没有必要尝试加载一个不存在世界的位置，因为无论如何没有人能够传送到那里。

从映射中获取的每个对象都必须转换为它的原始类型，这在之前的代码中已经完成。`float`值是一个特例。每个`float`值都将被读取为`double`值。`double`值类似于`float`，但更精确。因此，将`float`值作为`double`值加载，然后转换它们不会导致数据丢失。

这两种方法都将由 Bukkit 使用。作为一个程序员，您只需将此对象存储在 YAML 配置中即可。这可以通过简单地使用以下代码行来完成：

```java
config.set("location", serializableLoc);
```

然后，您可以使用以下代码检索数据：

```java
SerializableLocation loc = (SerializableLocation)config.get("location");
```

Bukkit 使用`serialize`方法和构造函数来处理其余部分。

类名和路径用于引用这个类。为了查看一个例子，请查看`MobEnhancer`插件的`config.yml`文件中的`ItemStack`对象。这个类的例子也已经提供：

```java
==: com.codisimus.warper.SerializableLocation
```

### 提示

路径当然会有您自己的命名空间，而不是`com.codisimus`。

这工作得很好，但可能会引起混淆，尤其是当路径名很长时。然而，有一种方法可以要求 Bukkit 通过别名引用这个类。执行以下两个步骤来完成：

1.  在类上方直接添加`@SerializableAs`注解，如下所示：

    ```java
    @SerializableAs("WarperLocation")
    public class SerializableLocation implements ConfigurationSerializable {
    ```

1.  在`ConfigurationSerialization 类`中注册您的班级：

    ```java
    ConfigurationSerialization.registerClass(SerializableLocation.class, "WarperLocation");
    ```

这可以在`onEnable`方法中完成。只需确保它在尝试加载数据之前执行。

### 提示

可序列化的名称必须是唯一的。因此，最好包含您的插件名称，而不仅仅是`Location`。这样，您可以为另一个插件提供一个可序列化的位置，而不会发生冲突。

# 将数据保存到 YAML 配置文件

现在，我们准备完成`save`方法。我们希望将数据保存到 YAML 文件中，就像我们在`config.yml`中做的那样。然而，我们不想将其保存到`config.yml`中，因为它有不同的用途。我们首先需要做的是创建一个新的 YAML 配置，如下所示：

```java
YamlConfiguration config = new YamlConfiguration();
```

接下来，我们将存储我们希望保存的所有信息。这是通过将对象设置到特定的路径来完成的，如下所示：

```java
config.set(String path, Object value);
```

在本章前面提到了`value`的可接受类型。在传送插件中，我们有地图，其中包含`SerializableLocation`方法。只要它们是字符串到`ConfigurationSerializable`对象的映射，就可以将地图添加到 YAML 配置中。**哈希表**在配置中的添加方式不同。你必须使用地图创建一个配置部分。

以下代码显示了我们将如何将传送数据添加到配置中：

```java
config.createSection("homes", homes);
config.createSection("warps", warps);
```

一旦所有数据都存储完毕，剩下的就是将配置写入`save`文件。这是通过在`config`上调用`save`方法并传递我们希望使用的文件来完成的。调用插件上的`getDataFolder`方法将给我们一个目录，我们应该在其中存储所有插件数据。这也是`config.yml`所在的位置。我们可以使用这个目录来引用我们将要保存数据的文件，如下所示：

```java
File file = new File(plugin.getDataFolder(), "warps.yml");
config.save(file);
```

我们将所有这些代码行放入一个`try`块中，以捕获可能发生的异常。如果你还不了解异常，它们是在出现某种错误或发生意外情况时抛出的。可以使用`try/catch`块来防止错误导致你的插件崩溃。在这种情况下，如果由于某种原因指定的文件无法写入，则会抛出异常。这种原因可能是用户权限不足或找不到文件位置。因此，带有`try`块的`save`方法如下：

```java
/**
 * Saves our HashMaps of warp locations so that they may be loaded
 */
private static void save() {
    try {
        //Create a new YAML configuration
        YamlConfiguration config = new YamlConfiguration();

        //Add each of our hashmaps to the config by creating sections
        config.createSection("homes", homes);
        config.createSection("warps", warps);

        //Write the configuration to our save file
        config.save(new File(plugin.getDataFolder(), "warps.yml"));
    } catch (Exception saveFailed) {
        plugin.getLogger().log(Level.SEVERE, "Save Failed!", saveFailed);
    }
}
```

以下是一个使用 Warper 插件创建的示例`warps.yml`文件：

```java
homes:
  18d6a045-cd24-451b-8e2e-b3fe09df46d3:
    ==: WarperLocation
    pitch: 6.1500483
    world: 89fd34ff-2c01-4d47-91c4-fa5d1e9fdb81
    x: -446.45572804715306
    y: 64.0
    yaw: 273.74963
    z: 224.9827566893271
warps:
  spawn:
    ==: WarperLocation
    pitch: 9.450012
    world: 89fd34ff-2c01-4d47-91c4-fa5d1e9fdb81
    x: -162.47507312961542
    y: 69.0
    yaw: -1.8000238
    z: 259.70096111857805
  Jungle:
    ==: WarperLocation
    pitch: 7.500037
    world: 35dafe89-3451-4c27-a626-3464e3856428
    x: -223.87850735096316
    y: 74.0
    yaw: 87.60001
    z: 382.482006630207
  frozen_lake:
    ==: WarperLocation
    pitch: 16.200054
    world: 53e7fab9-5f95-4e25-99d1-adce40d5447c
    x: -339.3448071127722
    y: 63.0
    yaw: 332.84973
    z: 257.9509874720554
```

# 从 YAML 配置加载数据

现在完成`save`方法后，我们准备编写`load`方法。你已经熟悉使用 Bukkit 配置 API 加载数据。我们现在要做的是类似于在上一章中讨论的从`config.yml`检索值。然而，我们必须首先手动使用以下代码加载配置，这将有所不同。我们只有在文件实际存在的情况下才这样做。因此，我们不希望在第一次使用插件时出现错误：

```java
File file = new File(plugin.getDataFolder(), "warps.yml");
if (file.exists()) {
    YamlConfiguration config = new YamlConfiguration();
    config.load(file);
```

现在我们已经加载了 YAML 配置，我们可以从中获取值。数据已放置在两个独特的配置部分中。我们将遍历这两个部分中的每个键，以加载所有位置。要从部分获取特定对象，我们只需要调用`get`方法并将其转换为正确的对象。你可以在完成的`load`方法中看到这是如何完成的：

```java
/**
 * Loads warp names/locations from warps.yml
 * 'warp' refers to both homes and public warps
 */
private static void load() {
    try {
        //Ensure that the file exists before attempting to load it
        File file = new File(plugin.getDataFolder(), "warps.yml");
        if (file.exists()) {
            //Load the file as a YAML Configuration
            YamlConfiguration config = new YamlConfiguration();
            config.load(file);

            //Get the homes section which is our saved hash map
            //Each key is the uuid of the Player
            //Each value is the location of their home
            ConfigurationSection section = config.getConfigurationSection("homes");
            for (String key: section.getKeys(false)) {
                //Get the location for each key
                SerializableLocation loc = (SerializableLocation)section.get(key);
                //Only add the warp location if it is valid
                if (loc.getLocation() != null) {
                    homes.put(key, loc);
                }
            }

            //Get the warps section which is our saved hash map
            //Each key is the name of the warp
            //Each value is the warp location
            section = config.getConfigurationSection("warps");
            for (String key: section.getKeys(false)) {
                //Get the location for each key
                SerializableLocation loc = (SerializableLocation) section.get(key);
                //Only add the warp location if it is valid
                if (loc.getLocation() != null) {
                    warps.put(key, loc);
                }
            }
        }
    } catch (Exception loadFailed) {
        plugin.getLogger().log(Level.SEVERE, "Load Failed!",loadFailed);
    }
}
```

现在插件已经完成，你可以在服务器上对其进行测试。设置一个家位置以及一些传送位置，然后查看保存的文件。停止并重新启动服务器以验证插件确实加载了正确的数据。

# 摘要

本章中我们创建的插件在数据被修改时都会调用`save`方法。在下一章中，你将学习如何定期保存数据。如果你希望在服务器关闭时保存数据，只需从插件的`main`类的`onDisable`方法中调用`save`方法。你可以利用你的编程技能来扩展这个插件。你可以添加权限节点，只需将它们添加到`plugin.yml`文件中即可。你还可以添加一个`config.yml`文件来修改消息或可能需要设置的即将到来的扭曲延迟的时间。如果你想要集成监听器，你可以监听`PlayerRespawnEvent`事件。然后，你可以设置玩家的重生位置为他们的家。有无数种方法可以自定义这个插件以满足你的需求。许多传送插件使用扭曲延迟来防止玩家在战斗中传送离开。在下一章中，我们将通过添加使用 Bukkit 调度器的扭曲延迟来扩展这个项目。
