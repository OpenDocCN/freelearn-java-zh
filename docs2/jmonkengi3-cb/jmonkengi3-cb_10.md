# 附录 A. 信息片段

# 简介

本附录包含一些过于通用而无法出现在常规章节中的代码和过程。它们跨越几个章节使用，但在此处展示以避免重复。

本附录涵盖了以下主题：

+   下载插件

+   启用每夜构建

+   向应用程序添加 Bullet 物理效果

+   Jaime 动画帧用于音素

+   `AnimationEvent` 补丁

+   `ImageGenerator` 类

+   `CellUtil` 类

# 下载插件

前往 **工具** 菜单并选择 **插件**。在 **可用插件** 选项卡中，查找您想要安装的插件，勾选旁边的复选框，然后选择 **安装**。

# 启用每夜构建

每夜构建将为您提供来自 jMonkeyEngine 仓库的最新更新，但应指出这些是不稳定的。有时功能可能会损坏，并且根本无法保证它们可以构建。要启用每夜构建，请执行以下步骤：

1.  在 SDK 中，前往 **工具** 菜单并选择 **插件**。

1.  前往 **设置** 选项卡并勾选 **jMonkeyEngine SDK Nightly (Breaks!)** 复选框。

1.  要查找更新，请前往 **帮助** 菜单并选择 **检查更新**。

# 向应用程序添加 Bullet 物理效果

本节提供了向应用程序添加子弹物理的基本步骤的描述。

在应用程序的 `simple InitApp` 方法中，添加以下代码行：

```java
BulletAppState bulletAppState = new BulletAppState();
stateManager.attach(bulletAppState);
```

要获得一个基本的地面和一些可以玩耍的项目，请添加以下代码：

```java
PhysicsTestHelper.createPhysicsTestWorldSoccer(rootNode, assetManager, bulletAppState.getPhysicsSpace());
```

需要物理效果的物体应同时附加到场景图中，并具有一个 `RigidBodyControl` 对象，该对象添加到 `bulletAppState` 的 `physicsSpace` 中。

# Jaime 动画帧用于音素

以下列表提供了在 Jaime 的动画中找到的多个帧，这些帧适合用作音素。它们对希望构建库的人来说可能很有用：

+   要找到 AAAH 音素，请使用 Punches 动画的第 30 帧。

+   要找到 EEH 音素，请使用 Wave 动画的第 4 帧。

+   要找到 I 音素，请使用 Wave 动画的第 9 帧。

+   要找到 OH 音素，请使用 Taunt 动画的第 22 帧。

+   要找到 OOOH 音素，请使用 Wave 动画的第 15 帧。

+   要找到 FUH 音素，请使用 Taunt 动画的第 7 帧。

+   要找到 MMM 音素，请使用 Wave 动画的第 1 帧。

+   要找到 LUH 音素，请使用 Punches 动画的第 21 帧。

+   要找到 EES 音素，请使用 Wave 动画的第 22 帧。

+   要找到 RESET 音素，请使用 Wave 动画的第 0 帧。

# 动画事件补丁

以下代码片段显示了 第四章 中 *掌握角色动画* 的 *唇同步和面部表情* 菜单所需的补丁。将其应用到您的项目文件中。如果您手动应用，则必须将构造函数添加到 `AnimationEvent`，并且以下代码行必须在 `cinematic.putEventData(MODEL_CHANNELS, model, s);` 之后放入 `initEvent` 方法中：

```java
int numChannels = model.getControl(AnimControl.class).getNumChannels();
for(int i = 0; i < numChannels; i++){
  ((HashMap<Integer, AnimChannel>)s).put(i, model.getControl(AnimControl.class).getChannel(i));
}
```

完整的补丁是：

```java
Index: AnimationEvent.java
===================================================================
— AnimationEvent.java    (revision 11001)
+++ AnimationEvent.java    (working copy)
@@ -221,6 +221,24 @@
 initialDuration = model.getControl(AnimControl.class).getAnimationLength(animationName);
 this.channelIndex = channelIndex;
}
+
+/**
+ * creates an animation event
+ *
+ * @param model the model on which the animation will be played
+ * @param animationName the name of the animation to play
+ * @param channelIndex the index of the channel default is 0\. Events on the
+ * @param blendTime the time during the animation are going to be blended
+ * same channelIndex will use the same channel.
+ */
+public AnimationEvent(Spatial model, String animationName, LoopMode loopMode, int channelIndex, float blendTime) {
+this.model = model;
+this.animationName = animationName;
+this.loopMode = loopMode;
+initialDuration = model.getControl(AnimControl.class).getAnimationLength(animationName);
+this.channelIndex = channelIndex;
+this.blendTime = blendTime;
+}

/**
* creates an animation event
@@ -264,11 +282,16 @@
Object s = cinematic.getEventData(MODEL_CHANNELS, model);
if (s == null) {
s = new HashMap<integer , AnimChannel>();
+int numChannels = model.getControl(AnimControl.class).getNumChannels();
+for(int i = 0; i < numChannels; i++){
+ ((HashMap<Integer, AnimChannel>)s).put(i, model.getControl(AnimControl.class).getChannel(i));
+}
cinematic.putEventData(MODEL_CHANNELS, model, s);
 }

Map</integer><integer , AnimChannel> map = (Map</integer><integer , AnimChannel>) s;
this.channel = map.get(channelIndex);
+
if (this.channel == null) {
if (model == null) {
                     //the model is null we try to find it according to the name
```

# `ImageGenerator` 类

`ImageGenerator` 类用于 第三章 的 *使用噪声生成地形* 菜谱，*世界构建*。创建此类的代码如下：

```java
public class ImageGenerator {

  public static void generateImage(float[][] terrain){
    int size = terrain.length;
    int grey;

    BufferedImage img = new BufferedImage(size, size, BufferedImage.TYPE_INT_RGB);
    for(int y = 0; y < size; y++){
      for(int x = 0; x < size; x++){
        double result = terrain[x][y];

        grey = (int) (result * 255);
        int color = (grey << 16) | (grey << 8) | grey;
        img.setRGB(x, y, color);

      }
    }

    try {
      ImageIO.write(img, "png", new File("assets/Textures/heightmap.png"));
    } catch (IOException ex) {
          Logger.getLogger(NoiseMapGenerator.class.getName()).log(Level.SEVERE, null, ex);
      }
    }
}
```

# `CellUtil` 类

`CellUtil` 类用于 第三章 的 *使用细胞自动机流动水* 菜谱，*世界构建*。创建此类的代码如下：

```java
public class CellUtil {

  private static int[][] directions = new int[][]{{0,-1},{1,-1},{1,0},{1,1},{0,1},{-1,1},{-1,0},{-1,-1}};
  public static int getDirection(int x, int y){
    witch(x){
      case 1:
      switch(y){
        case -1:
        return 1;
        case 0:
        return 2;
        case 1:
        return 3;
      }
      break;
      case -1:
      switch(y){
        case -1:
        return 7;
        case 0:
        return 6;
        case 1:
        return 5;
      }
      break;
      case 0:
      switch(y){
        case -1:
        return 0;
        case 0:
        return -1;
        case 1:
        return 4;
      }
      break;
    }
    return -1;
  }

  public static int[] getDirection(int dir){
   return directions[dir];
  }
}
```
