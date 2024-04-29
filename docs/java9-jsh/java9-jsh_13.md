# 第十三章。Java 9 中的模块化

在本章中，我们将利用 Java 9 添加的新功能之一，使我们能够将源代码模块化并轻松管理依赖关系。我们将：

+   重构现有代码以利用面向对象编程

+   在 Java 9 中使用新的模块化组织面向对象的代码

+   在 Java 9 中创建模块化源代码

+   使用 Java 9 编译器编译多个模块

+   使用 Java 9 运行模块化代码

# 重构现有代码以利用面向对象编程

如果我们从头开始编写面向对象的代码，我们可以利用我们在前几章学到的一切以及 Java 9 中包含的所有功能。随着需求的演变，我们将不得不对接口和类进行更改，进一步泛化或专门化它们，编辑它们，并创建新的接口和类。我们以面向对象的方式开始项目的事实将使我们能够轻松地对代码进行必要的调整。

有时，我们非常幸运，有机会在启动项目时就遵循最佳实践。然而，很多时候我们并不那么幸运，不得不处理未遵循最佳实践的项目。在这些情况下，我们可以利用我们喜爱的 IDE 提供的功能和额外的辅助工具来重构现有代码，生成促进代码重用并允许我们减少维护工作的面向对象代码，而不是遵循生成容易出错、重复且难以维护的代码的相同不良实践。

例如，假设我们需要开发一个 Web 服务，允许我们处理 3D 模型并在具有特定分辨率的 2D 图像上渲染它们。需求指定我们将使用我们的 Web 服务渲染的前两个 3D 模型是一个球体和一个立方体。Web 服务必须允许我们更改透视摄像机的以下参数，以便我们可以在 2D 屏幕上看到渲染的 3D 世界的特定部分：

+   位置（*X*、*Y*和*Z*值）

+   方向（*X*、*Y*和*Z*值）

+   上向量（*X*、*Y*和*Z*值）

+   透视视野（以度为单位）

+   近裁剪平面

+   远裁剪平面

假设其他开发人员开始在项目上工作，并生成了一个包含声明两个静态方法的类包装器的单个 Java 文件。其中一个方法渲染一个立方体，另一个方法渲染一个球体。这些方法接收渲染每个 3D 图形所需的所有参数，包括确定 3D 图形位置和大小以及配置透视摄像机和定向光的所有必要参数。

以下几行展示了一个名为`Renderer`的类的声明示例，其中包含两个静态方法：`renderCube`和`renderSphere`。第一个方法设置并渲染一个立方体，第二个方法设置并渲染一个球体。非常重要的是要理解，示例代码并不遵循最佳实践，我们将对其进行重构。请注意，这两个静态方法有很多共同的代码。示例的代码文件包含在`java_9_oop_chapter_13_01`文件夹中的`example13_01.java`文件中。

```java
// The following code doesn't follow best practices
// Please, do not use this code as a baseline
// We will refactor it to generate object-oriented code
public class Renderer {
    public static void renderCube(int x, int y, int z, int edgeLength,
        int cameraX, int cameraY, int cameraZ,
        int cameraDirectionX, int cameraDirectionY, int cameraDirectionZ,
        int cameraVectorX, int cameraVectorY, int cameraVectorZ,
        int cameraPerspectiveFieldOfView,
        int cameraNearClippingPlane,
        int cameraFarClippingPlane,
        int directionalLightX, int directionalLightY, int directionalLightZ,
        String directionalLightColor) {
            System.out.println(
                String.format("Created camera at (x:%d, y:%d, z:%d)",
                    cameraX, cameraY, cameraZ));
            System.out.println(
                String.format("Set camera direction to (x:%d, y:%d, z:%d)",
                    cameraDirectionX, cameraDirectionY, cameraDirectionZ));
            System.out.println(
                String.format("Set camera vector to (x:%d, y:%d, z:%d)",
                    cameraVectorX, cameraVectorY, cameraVectorZ));
            System.out.println(
                String.format("Set camera perspective field of view to: %d",
                    cameraPerspectiveFieldOfView));
            System.out.println(
                String.format("Set camera near clipping plane to: %d", 
                    cameraNearClippingPlane));
            System.out.println(
                String.format("Set camera far clipping plane to: %d",
                    cameraFarClippingPlane));
            System.out.println(
                String.format("Created directional light at (x:%d, y:%d, z:%d)",
                    directionalLightX, directionalLightY, directionalLightZ));
            System.out.println(
                String.format("Set light color to %s",
                    directionalLightColor));
            System.out.println(
                String.format("Drew cube at (x:%d, y:%d, z:%d) with edge length equal to %d" +
                    "considering light at (x:%d, y:%d, z:%d) " +
                    "and light's color equal to %s", 
                    x, y, z, edgeLength,
                    directionalLightX, directionalLightY, directionalLightZ,
                    directionalLightColor));
    }

    public static void renderSphere(int x, int y, int z, int radius,
        int cameraX, int cameraY, int cameraZ,
        int cameraDirectionX, int cameraDirectionY, 
        int cameraDirectionZ,
        int cameraVectorX, int cameraVectorY, int cameraVectorZ,
        int cameraPerspectiveFieldOfView,
        int cameraNearClippingPlane,
        int cameraFarClippingPlane,
        int directionalLightX, int directionalLightY, 
        int directionalLightZ,
        String directionalLightColor) {
            System.out.println(
                String.format("Created camera at (x:%d, y:%d, z:%d)",
                    cameraX, cameraY, cameraZ));
            System.out.println(
                String.format("Set camera direction to (x:%d, y:%d, z:%d)",
                    cameraDirectionX, cameraDirectionY, cameraDirectionZ));
            System.out.println(
                String.format("Set camera vector to (x:%d, y:%d, z:%d)",
                    cameraVectorX, cameraVectorY, cameraVectorZ));
            System.out.println(
                String.format("Set camera perspective field of view to: %d",
                    cameraPerspectiveFieldOfView));
            System.out.println(
                String.format("Set camera near clipping plane to: %d", 
                    cameraNearClippingPlane));
            System.out.println(
                String.format("Set camera far clipping plane to: %d",
                    cameraFarClippingPlane));
            System.out.println(
                String.format("Created directional light at (x:%d, y:%d, z:%d)",
                    directionalLightX, directionalLightY, directionalLightZ));
            System.out.println(
                String.format("Set light color to %s",
                    directionalLightColor));
            // Render the sphere
            System.out.println(
                String.format("Drew sphere at (x:%d, y:%d z:%d) with radius equal to %d",
                    x, y, z, radius));
            System.out.println(
                String.format("considering light at (x:%d, y:%d, z:%d)",
                    directionalLightX, directionalLightY, directionalLightZ));
            System.out.println(
                String.format("and the light's color equal to %s",
                    directionalLightColor));
    }
}
```

每个静态方法都需要大量的参数。现在，让我们想象一下我们对我们的 Web 服务有新的要求。我们必须添加代码来渲染额外的形状，并添加不同类型的摄像机和灯光。此外，我们必须在一个**IoT**（**物联网**）项目中工作，在这个项目中，我们必须在计算机视觉应用程序中重用形状，因此，我们希望利用我们为 Web 服务编写的代码，并与这个新项目共享代码库。此外，我们必须在另一个项目上工作，这个项目将在一块强大的 IoT 板上运行，具体来说，是英特尔 Joule 系列的一员，它将运行一个渲染服务，并利用其 4K 视频输出功能来显示生成的图形。我们将使用这块板载的强大四核 CPU 来运行本地渲染服务，在这种情况下，我们不会调用 Web 服务。

许多应用程序必须共享许多代码片段，我们的代码必须为新的形状、摄像机和灯光做好准备。代码很容易变得非常混乱、重复，并且难以维护。当然，先前显示的代码已经很难维护了。因此，我们将重构现有的代码，并创建许多接口和类来创建一个面向对象的版本，我们将能够根据新的要求进行扩展，并在不同的应用程序中重用。

到目前为止，我们一直在使用 JShell 来运行我们的代码示例。这一次，我们将为每个接口或类创建一个 Java 源代码文件。此外，我们将把这些文件组织到 Java 9 中引入的新模块中。最后，我们将编译这些模块并运行一个控制台应用程序。您可以使用您喜欢的编辑器或 IDE 来创建不同的代码文件。请记住，您可以下载指定的代码文件，而不必输入任何代码。

我们将创建以下公共接口、抽象类和具体类：

+   `Vector3d`：这个具体类表示一个可变的 3D 向量，具有`x`、`y`和`z`的`int`值。

+   `可渲染`：这个接口指定了具有位置并且可以被渲染的元素的要求。

+   `场景元素`：这个抽象类实现了`可渲染`接口，表示任何具有位置并且可以被渲染的元素。所有的场景元素都将继承自这个抽象类。

+   `灯光`：这个抽象类继承自`场景元素`，表示场景中的灯光，必须提供其属性的描述。

+   `定向光`：这个具体类继承自`灯光`，表示具有特定颜色的定向光。

+   `摄像机`：这个抽象类继承自`场景元素`，表示场景中的摄像机。

+   `透视摄像机`：这个具体类继承自`摄像机`，表示具有方向、上向量、视野、近裁剪平面和远裁剪平面的透视摄像机。

+   `形状`：这个抽象类继承自`场景元素`，表示场景中可以使用活动摄像机渲染并接收多个灯光的形状。

+   `球体`：这个具体类继承自`形状`，表示一个球体。

+   `立方体`：这个具体类继承自`形状`，表示一个立方体。

+   `场景`：这个具体类表示具有活动摄像机、形状和灯光的场景。我们可以使用这个类的实例来组合一个场景并渲染它。

+   `示例 01`：这个具体类将声明一个主静态方法，该方法将使用`透视摄像机`、`球体`、`立方体`和`定向光`来创建一个`场景`实例并调用其渲染方法。

我们将在一个扩展名为`.java`的文件中声明之前列举的每个接口、抽象类和具体类，并且文件名与我们声明的类型相同。例如，我们将在名为`Vector3d.java`的文件中声明`Vector3d`类，也就是 Java 源文件。

### 提示

在 Java 源文件中，声明与类型相同名称的单个公共接口或类是一种良好的实践和常见约定。如果我们在 Java 源文件中声明了多个公共类型，Java 编译器将生成错误。

# 使用 Java 9 中的新模块化组织面向对象的代码

当我们只有一些接口和类时，数百行面向对象的代码很容易组织和维护。然而，随着类型和代码行数的增加，有必要遵循一些规则来组织代码并使其易于维护。

一个非常好的面向对象的代码如果没有以有效的方式组织，就会产生维护上的头疼。我们不应该忘记，一个良好编写的面向对象的代码促进了代码重用。

在我们的示例中，我们只会有一些接口、抽象类和具体类。然而，我们必须想象我们将有大量额外的类型来支持额外的需求。因此，我们最终将拥有数十个与渲染场景组成元素所需的数学运算相关的类，额外类型的灯光，新类型的摄像机，与这些新灯光和摄像机相关的类，以及数十个额外的形状及其相关的类。

我们将创建许多模块，以便我们可以创建具有名称、需要其他模块并导出其他模块可用和可访问的公共 API 的软件单元。当一个模块需要其他模块时，这意味着该模块依赖于列出的模块。每个模块的名称将遵循我们通常在 Java 中使用的包的相同约定。

### 提示

其他模块只能访问模块导出的公共类型。如果我们在模块内声明了一个公共类型，但没有将其包含在导出的 API 中，那么我们将无法在模块外部访问它。在创建模块依赖关系时，我们必须避免循环依赖。

我们将创建以下八个模块：

+   `com.renderer.math`

+   `com.renderer.sceneelements`

+   `com.renderer.lights`

+   `com.renderer.cameras`

+   `com.renderer.shapes`

+   `com.renderer.shapes.curvededges`

+   `com.renderer.shapes.polyhedrons`

+   `com.renderer`

现在，每当我们需要处理灯光时，我们将探索`com.renderer.lights`模块中声明的类型。每当我们需要处理具有曲边的 3D 形状时，我们将探索`com.renderer.shapes.curvededges`模块中声明的类型。

每个模块将在与模块名称相同的包中声明类和接口。例如，`com.renderer.cameras`模块将在`com.renderer.cameras`包中声明类。**包**是相关类型的分组。每个包生成一个声明范围的命名空间。因此，我们将与模块结合使用包。

以下表格总结了我们将创建的模块，以及我们将在每个模块中声明的接口、抽象类和具体接口。此外，表格还指定了每个模块所需的模块列表。

| 模块名称 | 声明的公共类型 | 模块要求 |
| --- | --- | --- |
| `com.renderer.math` | `Vector3d` | `-` |
| `com.renderer.sceneelements` | `Rendereable``SceneElement` | `com.renderer.math` |
| `com.renderer.lights` | `Light``DirectionalLight` | `com.renderer.math``com.renderer.sceneelements` |
| `com.renderer.cameras` | `Camera``PerspectiveCamera` | `com.renderer.math``com.renderer.sceneelements` |
| `com.renderer.shapes` | `Shape` | `com.renderer.math``com.renderer.sceneelements``com.renderer.lights``com.renderer.cameras` |
| `com.renderer.shapes.curvededges` | `Sphere` | `com.renderer.math``com.renderer.lights``co` `m.renderer.shapes` |
| `com.renderer.shapes.polyhedrons` | `Cube` | `com.renderer.math``com.renderer.lights``com.renderer.shapes` |
| `com.renderer` | `Scene``Example01` | `com.renderer.math``com.renderer.cameras``com.renderer.lights``com.renderer.shapes``com.renderer.shapes.curvededges``com.renderer.shapes.polyhedrons` |

非常重要的是要注意，所有模块还需要`java.base`模块，该模块导出所有平台的核心包，如`java.io`、`java.lang`、`java.math`、`java.net`和`java.util`等。然而，每个模块都隐式依赖于`java.base`模块，因此，在声明新模块并指定其所需模块时，无需将其包含在依赖列表中。

下一个图表显示了模块图，其中模块是节点，一个模块对另一个模块的依赖是一个有向边。我们不在模块图中包括`java.lang`。

![使用 Java 9 中的新模块化组织面向对象的代码](img/00101.jpeg)

我们不会使用任何特定的 IDE 来创建所有模块。这样，我们将了解目录结构和所有必需的文件。然后，我们可以利用我们喜欢的 IDE 中包含的功能轻松创建新模块及其必需的目录结构。

有一个约定规定，模块的源代码必须位于与模块名称相同的目录中。例如，名为`com.renderer.math`的模块必须位于名为`com.renderer.math`的目录中。我们必须为每个所需的模块创建一个模块描述符，即在模块的根文件夹中创建一个名为`module-info.java`的源代码文件。该文件指定了模块名称、所需的模块和模块导出的包。导出的包将被需要该模块的模块看到。

然后，需要为模块名称中由点（`.`）分隔的每个名称创建子目录。例如，我们将在`com.renderer.math`目录中创建`com/renderer/math`目录（在 Windows 中为`com\renderer\math`子文件夹）。声明每个模块的接口、抽象类和具体类的 Java 源文件将位于这些子文件夹中。

我们将创建一个名为`Renderer`的基本目录，其中包含一个名为`src`的子文件夹，其中包含我们所有模块的源代码。因此，我们将`Renderer/src`（在 Windows 中为`Renderer\src`）作为我们的源代码基本目录。然后，我们将为每个模块创建一个文件夹，其中包含`module-info.java`文件和 Java 源代码文件的子文件夹。以下目录结构显示了我们将在`Renderer/src`（在 Windows 中为`Renderer\src`）目录中拥有的最终内容。文件名已突出显示。

```java
├───com.renderer
│   │   module-info.java
│   │
│   └───com
│       └───renderer
│               Example01.java
│               Scene.java
│
├───com.renderer.cameras
│   │   module-info.java
│   │
│   └───com
│       └───renderer
│           └───cameras
│                   Camera.java
│                   PerspectiveCamera.java
│
├───com.renderer.lights
│   │   module-info.java
│   │
│   └───com
│       └───renderer
│           └───lights
│                   DirectionalLight.java
│                   Light.java
│
├───com.renderer.math
│   │   module-info.java
│   │
│   └───com
│       └───renderer
│           └───math
│                   Vector3d.java
│
├───com.renderer.sceneelements
│   │   module-info.java
│   │
│   └───com
│       └───renderer
│           └───sceneelements
│                   Rendereable.java
│                   SceneElement.java
│
├───com.renderer.shapes
│   │   module-info.java
│   │
│   └───com
│       └───renderer
│           └───shapes
│                   Shape.java
│
├───com.renderer.shapes.curvededges
│   │   module-info.java
│   │
│   └───com
│       └───renderer
│           └───shapes
│               └───curvededges
│                       Sphere.java
│
└───com.renderer.shapes.polyhedrons
 │   module-info.java
    │
    └───com
        └───renderer
            └───shapes
                └───polyhedrons
 Cube.java

```

# 创建模块化源代码。

现在是时候开始创建必要的目录结构，并为每个模块编写`module-info.java`文件和源 Java 文件的代码了。我们将创建`com.renderer.math`模块。

创建一个名为`Renderer`的目录和一个`src`子目录。我们将使用`Renderer/src`（在 Windows 中为`Renderer\src`）作为我们的源代码基本目录。但是，请注意，如果您下载源代码，则无需创建任何文件夹。

现在在`Renderer/src`（在 Windows 中为`Renderer\src`）中创建`com.renderer.math`目录。将以下行添加到名为`module-info.java`的文件中，该文件位于最近创建的子文件夹中。下面的行组成了名为`com.renderer.math`的模块描述符。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.math`子文件夹中的`module-info.java`文件中。

```java
module com.renderer.math {
    exports com.renderer.math;
}
```

`module`关键字后跟模块名称`com.renderer.math`开始模块声明。花括号中包含的行指定了模块主体。`exports`关键字后跟包名`com.renderer.math`表示该模块导出`com.renderer.math`包中声明的所有公共类型。

在`Renderer/src`（在 Windows 中为`Renderer\src`）中创建`com/renderer/math`（在 Windows 中为`com\renderer\math`）文件夹。将以下行添加到名为`Vector3d.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行声明了公共`Vector3d`具体类，作为`com.renderer.math`包的成员。我们将使用`Vector3d`类，而不是使用`x`、`y`和`z`的单独值。`package`关键字后面跟着包名，表示类将被包含在其中的包。

示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.math/com/renderer/math`子文件夹中，名为`Vector3d.java`。

```java
package com.renderer.math;

public class Vector3d {
    public int x;
    public int y;
    public int z;

    public Vector3d(int x, 
        int y, 
        int z) {
        this.x = x;
        this.y = y;
        this.z = z;
    }

    public Vector3d(int valueForXYZ) {
        this(valueForXYZ, valueForXYZ, valueForXYZ);
    }

    public Vector3d() {
        this(0);
    }

    public void absolute() {
        x = Math.abs(x);
        y = Math.abs(y);
        z = Math.abs(z);
    }

    public void negate() {
        x = -x;
        y = -y;
        z = -z;
    }

    public void add(Vector3d vector) {
        x += vector.x;
        y += vector.y;
        z += vector.z;
    }

    public void sub(Vector3d vector) {
        x -= vector.x;
        y -= vector.y;
        z -= vector.z;
    }

    public String toString() {
        return String.format(
            "(x: %d, y: %d, z: %d)",
            x,
            y,
            z);
    }
}
```

现在在`Renderer/src`（在 Windows 中为`Renderer\src`）中创建`com.renderer.sceneelements`目录。将以下行添加到名为`module-info.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行组成了名为`com.renderer.sceneelements`的模块描述符。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.sceneelements`子文件夹中，名为`module-info.java`。

```java
module com.renderer.sceneelements {
    requires com.renderer.math;
    exports com.renderer.sceneelements;
}
```

`module`关键字后面跟着模块名`com.renderer.sceneelements`开始模块声明。花括号内包含的行指定了模块主体。`requires`关键字后面跟着模块名`com.renderer.math`，表示该模块需要先前声明的`com.renderer.math`模块中导出的类型。`exports`关键字后面跟着包名`com.renderer.sceneelements`，表示该模块导出`com.renderer.sceneelements`包中声明的所有公共类型。

在`Renderer/src`（在 Windows 中为`Renderer\src`）中创建`com/renderer/sceneelements`（在 Windows 中为`com\renderer\sceneelements`）文件夹。将以下行添加到名为`Rendereable.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行声明了公共`Rendereable`接口，作为`com.renderer.sceneelements`包的成员。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.sceneeelements/com/renderer/sceneelements`子文件夹中，名为`Rendereable.java`。

```java
package com.renderer.sceneelements;

import com.renderer.math.Vector3d;

public interface Rendereable {
    Vector3d getLocation();
    void setLocation(Vector3d newLocation);
    void render();
}
```

将以下行添加到名为`SceneElement.java`的文件中。接下来的行声明了公共`SceneElement`抽象类，作为`com.renderer.sceneelements`包的成员。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.sceneelements/com/renderer/sceneelements`子文件夹中，名为`SceneElement.java`。

```java
package com.renderer.sceneelements;

import com.renderer.math.Vector3d;

public abstract class SceneElement implements Rendereable {
    protected Vector3d location;

    public SceneElement(Vector3d location) {
        this.location = location;
    }

    public Vector3d getLocation() {
        return location;
    }

    public void setLocation(Vector3d newLocation) {
        location = newLocation;
    }
}
```

`SceneElement`抽象类实现了先前定义的`Rendereable`接口。该类表示场景中的 3D 元素，并具有使用`Vector3d`指定的位置。该类是所有需要在 3D 空间中具有位置的场景元素的基类。

现在在`Renderer/src`（在 Windows 中为`Renderer\src`）中创建`com.renderer.lights`目录。将以下行添加到名为`module-info.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行组成了名为`com.renderer.lights`的模块描述符。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.lights`子文件夹中，名为`module-info.java`。

```java
module com.renderer.lights {
    requires com.renderer.math;
    requires com.renderer.sceneelements;
    exports com.renderer.lights;
}
```

前面的行声明了`com.renderer.lights`模块，并指定该模块需要两个模块：`com.renderer.math`和`com.renderer.sceneelements`。`exports`关键字后面跟着包名`com.renderer.lights`，表示该模块导出`com.renderer.lights`包中声明的所有公共类型。

在`Renderer/src`中创建`com/renderer/lights`（在 Windows 中为`com\renderer\lights`）文件夹。将以下行添加到名为`Light.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行声明了公共`Light`抽象类作为`com.renderer.lights`包的成员。该类继承自`SceneElement`类，并声明了一个必须返回`String`类型的描述所有灯光属性的抽象`getPropertiesDescription`方法。从`Light`类继承的具体类将负责为此方法提供实现。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.lights/com/renderer/lights`子文件夹中的`Light.java`文件中。

```java
package com.renderer.lights;

import com.renderer.sceneelements.SceneElement;
import com.renderer.math.Vector3d;

public abstract class Light extends SceneElement {
    public Light(Vector3d location) {
        super(location);
    }

    public abstract String getPropertiesDescription();
}
```

将以下行添加到名为`DirectionalLight.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行声明了公共`DirectionalLight`具体类作为`com.renderer.lights`包的成员。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.lights/com/renderer/lights`子文件夹中的`DirectionalLight.java`文件中。

```java
package com.renderer.lights;

import com.renderer.math.Vector3d;

public class DirectionalLight extends Light {
    public final String color;

    public DirectionalLight(Vector3d location, 
        String color) {
        super(location);
        this.color = color;
    }

    @Override
    public void render() {
        System.out.println(
            String.format("Created directional light at %s",
                location));
        System.out.println(
            String.format("Set light color to %s",
                color));
    }

    @Override
    public String getPropertiesDescription() {
        return String.format(
            "light's color equal to %s",
            color);
    }
}
```

`DirectionalLight`具体类继承自先前定义的`Light`抽象类。`DirectionalLight`类表示定向光，并为`render`和`getPropertiesDescription`方法提供实现。

现在在`Renderer/src`中创建`com.renderer.cameras`目录（在 Windows 中为`Renderer\src`）。将以下行添加到名为`module-info.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行组成了名为`com.renderer.cameras`的模块描述符。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.cameras`子文件夹中的`module-info.java`文件中。

```java
module com.renderer.cameras {
    requires com.renderer.math;
    requires com.renderer.sceneelements;
    exports com.renderer.cameras;
}
```

前面的行声明了`com.renderer.cameras`模块，并指定该模块需要两个模块：`com.renderer.math`和`com.renderer.sceneelements`。`exports`关键字后跟包名`com.renderer.cameras`，表示该模块导出`com.renderer.cameras`包中声明的所有公共类型。

在`Renderer/src`中创建`com/renderer/cameras`（在 Windows 中为`com\renderer\cameras`）文件夹。将以下行添加到名为`Camera.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行声明了公共`Camera`抽象类作为`com.renderer.cameras`包的成员。该类继承自`SceneElement`类。该类表示 3D 相机。这是所有相机的基类。在这种情况下，类声明为空，我们只声明它是因为我们知道将会有许多类型的相机。此外，我们希望能够在将来概括所有类型相机的共同要求，就像我们为灯光做的那样。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.cameras/com/renderer/cameras`子文件夹中的`Camera.java`文件中。

```java
package com.renderer.cameras;

import com.renderer.math.Vector3d;
import com.renderer.sceneelements.SceneElement;

public abstract class Camera extends SceneElement {
    public Camera(Vector3d location) {
        super(location);
    }
}
```

将以下行添加到名为`PerspectiveCamera.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行声明了公共`PerspectiveCamera`具体类作为`com.renderer.cameras`包的成员。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.cameras/com/renderer/cameras`子文件夹中的`PerspectiveCamera.java`文件中。

```java
package com.renderer.cameras;

import com.renderer.math.Vector3d;

public class PerspectiveCamera extends Camera {
    protected Vector3d direction;
    protected Vector3d vector;
    protected int fieldOfView;
    protected int nearClippingPlane;
    protected int farClippingPlane;

    public Vector3d getDirection() {
        return direction;
    }

    public void setDirection(Vector3d newDirection) {
        direction = newDirection;
    }

    public Vector3d getVector() {
        return vector;
    }

    public void setVector(Vector3d newVector) {
        vector = newVector;
    }

    public int getFieldOfView() {
        return fieldOfView;
    }

    public void setFieldOfView(int newFieldOfView) {
        fieldOfView = newFieldOfView;
    }

    public int nearClippingPlane() {
        return nearClippingPlane;
    }

    public void setNearClippingPlane(int newNearClippingPlane) {
        this.nearClippingPlane = newNearClippingPlane;
    }

    public int farClippingPlane() {
        return farClippingPlane;
    }

    public void setFarClippingPlane(int newFarClippingPlane) {
        this.farClippingPlane = newFarClippingPlane;
    }

    public PerspectiveCamera(Vector3d location, 
        Vector3d direction, 
        Vector3d vector, 
        int fieldOfView, 
        int nearClippingPlane, 
        int farClippingPlane) {
        super(location);
        this.direction = direction;
        this.vector = vector;
        this.fieldOfView = fieldOfView;
        this.nearClippingPlane = nearClippingPlane;
        this.farClippingPlane = farClippingPlane;
    }

    @Override
    public void render() {
        System.out.println(
            String.format("Created camera at %s",
                location));
        System.out.println(
            String.format("Set camera direction to %s",
                direction));
        System.out.println(
            String.format("Set camera vector to %s",
                vector));
        System.out.println(
            String.format("Set camera perspective field of view to: %d",
                fieldOfView));
        System.out.println(
            String.format("Set camera near clipping plane to: %d", 
                nearClippingPlane));
        System.out.println(
            String.format("Set camera far clipping plane to: %d",
                farClippingPlane));
    }
}
```

`PerspectiveCamera`具体类继承自先前定义的`Camera`抽象类。`PerspectiveCamera`类表示具有许多获取器和设置器方法的透视相机的实现。该类为`render`方法提供了一个显示所创建相机的所有细节和其不同属性值的实现。

现在在`Renderer/src`（Windows 中为`Renderer\src`）中创建`com.renderer.shapes`目录。将以下行添加到名为`module-info.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行组成了名为`com.renderer.shapes`的模块描述符。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.shapes`子文件夹中的`module-info.java`文件中。

```java
module com.renderer.shapes {
    requires com.renderer.math;
    requires com.renderer.sceneelements;
    requires com.renderer.lights;
    requires com.renderer.cameras;
    exports com.renderer.shapes;
}
```

前面的行声明了`com.renderer.shapes`模块，并指定该模块需要四个模块：`com.renderer.math`、`com.renderer.sceneelements`、`com.renderer.lights`和`com.renderer.cameras`。`exports`关键字后跟包名`com.renderer.shapes`，表示该模块导出了`com.renderer.shapes`包中声明的所有公共类型。

在`Renderer/src`（Windows 中为`Renderer\src`）中创建`com/renderer/shapes`（Windows 中为`com\renderer\shapes`）文件夹。将以下行添加到名为`Shape.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行声明了公共`Shape`抽象类作为`com.renderer.shapes`包的成员。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.shapes/com/renderer/shapes`子文件夹中的`Shape.java`文件中。

```java
package com.renderer.shapes;

import com.renderer.math.Vector3d;
import com.renderer.sceneelements.SceneElement;
import com.renderer.lights.Light;
import com.renderer.cameras.Camera;
import java.util.*;
import java.util.stream.Collectors;

public abstract class Shape extends SceneElement {
    protected Camera activeCamera;
    protected List<Light> lights;

    public Shape(Vector3d location) {
        super(location);
        lights = new ArrayList<>();
    }

    public void setActiveCamera(Camera activeCamera) {
        this.activeCamera = activeCamera;
    }

    public void setLights(List<Light> lights) {
        this.lights = lights;
    }

    protected boolean isValidForRender() {
        return !((activeCamera == null) && lights.isEmpty());
    }

    protected String generateConsideringLights() {
        return lights.stream()
            .map(light -> String.format(
                "considering light at %s\nand %s",
                    light.getLocation(), 
                    light.getPropertiesDescription()))
            .collect(Collectors.joining());
    }
}
```

`Shape`类继承自`SceneElement`类。该类表示一个 3D 形状，是所有 3D 形状的基类。该类定义了以下方法：

+   `setActiveCamera`：这个公共方法接收一个`Camera`实例并将其保存为活动摄像机。

+   `setLights`：这个公共方法接收一个`List<Light>`并将其保存为必须考虑以渲染形状的灯光列表。

+   `isValidForRender`：这个受保护的方法返回一个`boolean`值，指示形状是否具有活动摄像机和至少一个灯光。否则，该形状不适合被渲染。

+   `generateConsideringLights`：这个受保护的方法返回一个带有正在考虑渲染形状的灯光、它们的位置和属性描述的`String`。

`Shape`类的每个子类，代表特定的 3D 形状，将为`render`方法提供实现。我们将在另外两个模块中编写这些子类。

现在在`Renderer/src`（Windows 中为`Renderer\src`）中创建`com.renderer.shapes.curvededges`目录。将以下行添加到名为`module-info.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行组成了名为`com.renderer.curvededges`的模块描述符。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.curvededges`子文件夹中的`module-info.java`文件中。

```java
module com.renderer.shapes.curvededges {
    requires com.renderer.math;
    requires com.renderer.lights;
    requires com.renderer.shapes;
    exports com.renderer.shapes.curvededges;
}
```

前面的行声明了`com.renderer.shapes`模块，并指定该模块需要三个模块：`com.renderer.math`、`com.renderer.lights`和`com.renderer.shapes`。`exports`关键字后跟包名`com.renderer.shapes.curvededges`，表示该模块导出了`com.renderer.shapes.curvededges`包中声明的所有公共类型。

在`Renderer/src`（Windows 中为`Renderer\src`）中创建`com/renderer/shapes/curvededges`（Windows 中为`com\renderer\shapes\curvededges`）文件夹。将以下行添加到名为`Sphere.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行声明了公共`Sphere`具体类作为`com.renderer.shapes.curvededges`包的成员。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.shapes.curvededges/com/renderer/shapes/curvededges`子文件夹中的`Sphere.java`文件中。

```java
package com.renderer.shapes.curvededges;

import com.renderer.math.Vector3d;
import com.renderer.shapes.Shape;
import com.renderer.lights.Light;

public class Sphere extends Shape {
    protected int radius;

    public Sphere(Vector3d location, int radius) {
        super(location);
        this.radius = radius;
    }

    public int getRadius() {
        return radius;
    }

    public void setRadius(int newRadius) { 
        radius = newRadius;
    }

    @Override
    public void render() {
        if (!isValidForRender()) {
            System.out.println(
                "Setup wasn't completed to render the sphere.");
            return;
        }
        StringBuilder sb = new StringBuilder();
        sb.append(String.format(
            "Drew sphere at %s with radius equal to %d\n",
            location, 
            radius));
        String consideringLights = 
            generateConsideringLights();
        sb.append(consideringLights);
        System.out.println(sb.toString());
    }
}
```

`Sphere`类继承自`Shape`类，并在构造函数中需要一个半径值，除了指定球体位置的`Vector3d`实例。该类提供了`render`方法的实现，该方法检查`isValidForRender`方法返回的值。如果该方法返回`true`，则球体可以被渲染，并且代码将使用球体半径、位置以及在渲染球体时考虑的灯光构建消息。代码调用`generateConsideringLights`方法来构建消息。

现在在`Renderer/src`（Windows 中为`Renderer\src`）中创建`com.renderer.shapes.polyhedrons`目录。将以下行添加到名为`module-info.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行组成了名为`com.renderer.polyhedrons`的模块描述符。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.polyhedrons`子文件夹中的`module-info.java`文件中。

```java
module com.renderer.shapes.polyhedrons {
    requires com.renderer.math;
    requires com.renderer.lights;
    requires com.renderer.shapes;
    exports com.renderer.shapes.polyhedrons;
}
```

前面的行声明了`com.renderer.polyhedrons`模块，并指定该模块需要三个模块：`com.renderer.math`、`com.renderer.lights`和`com.renderer.shapes`。`exports`关键字后跟包名`com.renderer.shapes.polyhedrons`，表示该模块导出`com.renderer.shapes.polyhedrons`包中声明的所有公共类型。

在`Renderer/src`（Windows 中为`Renderer\src`）中创建`com/renderer/shapes/polyhedrons`（Windows 中为`com\renderer\shapes\polyhedrons`）文件夹。将以下行添加到名为`Cube.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行声明了公共`Cube`具体类作为`com.renderer.shapes.polyhedrons`包的成员。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer.shapes.polyhedrons/com/renderer/shapes/polyhedrons`子文件夹中的`Cube.java`文件中。

```java
package com.renderer.shapes.polyhedrons;

import com.renderer.math.Vector3d;
import com.renderer.shapes.Shape;
import com.renderer.lights.Light;
import java.util.stream.Collectors;

public class Cube extends Shape {
    protected int edgeLength;

    public Cube(Vector3d location, int edgeLength) {
        super(location);
        this.edgeLength = edgeLength;
    }

    public int getEdgeLength() {
        return edgeLength;
    }

    public void setEdgeLength(int newEdgeLength) { 
        edgeLength = newEdgeLength;
    }

    @Override
    public void render() {
        if (!isValidForRender()) {
            System.out.println(
                "Setup wasn't completed to render the cube.");
            return;
        }
        StringBuilder sb = new StringBuilder();
        sb.append(String.format(
            "Drew cube at %s with edge length equal to %d\n",
            location,
            edgeLength));
        String consideringLights = 
            generateConsideringLights();
        sb.append(consideringLights);
        System.out.println(sb.toString());
    }
}
```

`Cube`类继承自`Shape`类，并在构造函数中需要一个`edgeLength`值，除了指定立方体位置的`Vector3d`。该类提供了`render`方法的实现，该方法检查`isValidForRender`方法返回的值。如果该方法返回`true`，则立方体可以被渲染，并且代码将使用立方体的边长、位置以及在渲染立方体时考虑的灯光构建消息。代码调用`generateConsideringLights`方法来构建消息。

现在在`Renderer/src`（Windows 中为`Renderer\src`）中创建`com.renderer`目录。将以下行添加到名为`module-info.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行组成了名为`com.renderer`的模块描述符。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer`子文件夹中的`module-info.java`文件中。

```java
module com.renderer {
    exports com.renderer;
    requires com.renderer.math;
    requires com.renderer.cameras;
    requires com.renderer.lights;
    requires com.renderer.shapes;
    requires com.renderer.shapes.curvededges;
    requires com.renderer.shapes.polyhedrons;
}
```

前面的行声明了`com.renderer`模块，并指定该模块需要六个模块：`com.renderer.math`、`com.renderer.cameras`、`com.renderer.lights`、`com.renderer.shapes`、`com.renderer.shapes.curvededges`和`com.renderer.shapes.polyhedrons`。`exports`关键字后跟包名`com.renderer`，表示该模块导出`com.renderer`包中声明的所有公共类型。

在`Renderer/src`（Windows 中为`Renderer\src`）中创建`com/renderer`（Windows 中为`com\renderer`）文件夹。将以下行添加到名为`Scene.java`的文件中，该文件位于最近创建的子文件夹中。接下来的行声明了公共`Scene`具体类作为`com.renderer`包的成员。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer/com/renderer`子文件夹中的`Scene.java`文件中。

```java
package com.renderer;

import com.renderer.math.Vector3d;
import com.renderer.cameras.Camera;
import com.renderer.lights.Light;
import com.renderer.shapes.Shape;
import java.util.*;

public class Scene {
    protected List<Light> lights;
    protected List<Shape> shapes;
    protected Camera activeCamera;

    public Scene(Camera activeCamera) {
        this.activeCamera = activeCamera;
        this.lights = new ArrayList<>();
        this.shapes = new ArrayList<>();
    }

    public void addLight(Light light) {
        this.lights.add(light);
    }

    public void addShape(Shape shape) {
        this.shapes.add(shape);
    }

    public void render() {
        activeCamera.render();
        lights.forEach(Light::render);
        shapes.forEach(shape -> {
            shape.setActiveCamera(activeCamera);
            shape.setLights(lights);
            shape.render();
        });
    }
}
```

`Scene`类表示要渲染的场景。该类声明了一个`activateCamera`受保护字段，其中包含一个`Camera`实例。`lights`受保护字段是`Light`实例的`List`，`shapes`受保护字段是组成场景的`Shape`实例的`List`。`addLight`方法将接收到的`Light`实例添加到`List<Light>lights`中。`addShape`方法将接收到的`Shape`实例添加到`List<Shape> shapes`中。

`render`方法调用活动摄像机和所有灯光的渲染方法。然后，代码对每个形状执行以下操作：设置其活动摄像机，设置灯光，并调用`render`方法。

最后，将以下行添加到名为`Example01.java`的文件中。接下来的行声明了公共`Example01`具体类作为`com.renderer`包的成员。示例的代码文件包含在`java_9_oop_chapter_13_01/Renderer/src/com.renderer/com/renderer`子文件夹中的`Example01.java`文件中。

```java
package com.renderer;

import com.renderer.math.Vector3d;
import com.renderer.cameras.PerspectiveCamera;
import com.renderer.lights.DirectionalLight;
import com.renderer.shapes.curvededges.Sphere;
import com.renderer.shapes.polyhedrons.Cube;

public class Example01 {
    public static void main(String[] args){
        PerspectiveCamera camera = new PerspectiveCamera(
            new Vector3d(30),
            new Vector3d(50, 0, 0),
            new Vector3d(4, 5, 2),
            90,
            20,
            40);
        Sphere sphere = new Sphere(new Vector3d(20), 8);
        Cube cube = new Cube(new Vector3d(10), 5);
        DirectionalLight light = new DirectionalLight(
            new Vector3d(2, 2, 5), "Cornflower blue");
        Scene scene = new Scene(camera);
        scene.addShape(sphere);
        scene.addShape(cube);
        scene.addLight(light);
        scene.render();
    }
}
```

`Example01`类是我们测试应用程序的主类。该类只声明了一个名为`main`的`static`方法，该方法接收一个名为`args`的`String`数组作为参数。当我们执行应用程序时，Java 将调用此方法，并将参数传递给`args`参数。在这种情况下，`main`方法中的代码不考虑任何指定的参数。

主要方法创建一个具有必要参数的`PerspectiveCamera`实例，然后创建一个名为`shape`和`cube`的`Shape`和`Cube`。然后，代码创建一个名为`light`的`DirectionalLight`实例。

下一行创建一个具有`camera`作为`activeCamera`参数值的`Scene`实例。然后，代码两次调用`scene.addShape`方法，参数分别为`sphere`和`cube`。最后，代码调用`scene.addLight`，参数为`light`，并调用`scene.render`方法来显示模拟渲染过程生成的消息。

# 使用 Java 9 编译器编译多个模块

在名为`Renderer`的基本目录中创建一个名为`mods`的子文件夹。这个新的子文件夹将复制我们在`Renderer/src`（Windows 中的`Renderer\src`）文件夹中创建的目录结构。我们将运行 Java 编译器为每个 Java 源文件生成一个 Java 类文件。Java 类文件将包含可以在**Java 虚拟机**上执行的 Java 字节码，也称为**JVM**。对于每个具有`.java`扩展名的 Java 源文件，包括模块描述符，我们将有一个具有`.class`扩展名的文件。例如，当我们成功使用 Java 编译器编译`Renderer/src/com.renderer.math/com/renderer/math/Vector3d.java`源文件时，编译器将生成一个`Renderer/mods/com.renderer.math/com/renderer/math/Vector3d.class`文件，其中包含 Java 字节码（称为 Java 类文件）。在 Windows 中，我们必须使用反斜杠（`\`）作为路径分隔符，而不是斜杠（`/`）。

现在，在 macOS 或 Linux 上打开一个终端窗口，或者在 Windows 上打开命令提示符，并转到`Renderer`文件夹。确保`javac`命令包含在路径中，并且它是 Java 9 的 Java 编译器，而不是之前版本的 Java 编译器，这些版本不兼容 Java 9 中引入的模块。

在 macOS 或 Linux 中，运行以下命令来编译我们最近创建的所有模块，并将生成的 Java 类文件放在`mods`文件夹中的目录结构中。`-d`选项指定了生成类文件的位置，`--module-source-path`选项指示了多个模块的输入源文件的位置。

```java
javac -d mods --module-source-path src src/com.renderer.math/module-info.java src/com.renderer.math/com/renderer/math/Vector3d.java src/com.renderer.sceneelements/module-info.java src/com.renderer.sceneelements/com/renderer/sceneelements/Rendereable.java src/com.renderer.sceneelements/com/renderer/sceneelements/SceneElement.java src/com.renderer.cameras/module-info.java src/com.renderer.cameras/com/renderer/cameras/Camera.java src/com.renderer.cameras/com/renderer/cameras/PerspectiveCamera.java src/com.renderer.lights/module-info.java src/com.renderer.lights/com/renderer/lights/DirectionalLight.java src/com.renderer.lights/com/renderer/lights/Light.java src/com.renderer.shapes/module-info.java src/com.renderer.shapes/com/renderer/shapes/Shape.java src/com.renderer.shapes.curvededges/module-info.java src/com.renderer.shapes.curvededges/com/renderer/shapes/curvededges/Sphere.java src/com.renderer.shapes.polyhedrons/module-info.java src/com.renderer.shapes.polyhedrons/com/renderer/shapes/polyhedrons/Cube.java src/com.renderer/module-info.java src/com.renderer/com/renderer/Example01.java src/com.renderer/com/renderer/Scene.java

```

在 Windows 中，运行以下命令以实现相同的目标：

```java
javac -d mods --module-source-path src src\com.renderer.math\module-info.java src\com.renderer.math\com\renderer\math\Vector3d.java src\com.renderer.sceneelements\module-info.java src\com.renderer.sceneelements\com\renderer\sceneelements\Rendereable.java src\com.renderer.sceneelements\com\renderer\sceneelements\SceneElement.java src\com.renderer.cameras\module-info.java src\com.renderer.cameras\com\renderer\cameras\Camera.java src\com.renderer.cameras\com\renderer\cameras\PerspectiveCamera.java src\com.renderer.lights\module-info.java src\com.renderer.lights\com\renderer\lights\DirectionalLight.java src\com.renderer.lights\com\renderer\lights\Light.java src\com.renderer.shapes\module-info.java src\com.renderer.shapes\com\renderer\shapes\Shape.java src\com.renderer.shapes.curvededges\module-info.java src\com.renderer.shapes.curvededges\com\renderer\shapes\curvededges\Sphere.java src\com.renderer.shapes.polyhedrons\module-info.java src\com.renderer.shapes.polyhedrons\com\renderer\shapes\polyhedrons\Cube.java src\com.renderer\module-info.java src\com.renderer\com\renderer\Example01.java src\com.renderer\com\renderer\Scene.java

```

以下目录结构显示了我们将在`Renderer/mods`（Windows 中的`Renderer\mods`）目录中拥有的最终内容。Java 编译器生成的 Java 类文件已经高亮显示。

```java
├───com.renderer
│   │   module-info.class
│   │
│   └───com
│       └───renderer
│               Example01.class
│               Scene.class
│
├───com.renderer.cameras
│   │   module-info.class
│   │
│   └───com
│       └───renderer
│           └───cameras
│                   Camera.class
│                   PerspectiveCamera.class
│
├───com.renderer.lights
│   │   module-info.class
│   │
│   └───com
│       └───renderer
│           └───lights
│                   DirectionalLight.class
│                   Light.class
│
├───com.renderer.math
│   │   module-info.class
│   │
│   └───com
│       └───renderer
│           └───math
│                   Vector3d.class
│
├───com.renderer.sceneelements
│   │   module-info.class
│   │
│   └───com
│       └───renderer
│           └───sceneelements
│                   Rendereable.class
│                   SceneElement.class
│
├───com.renderer.shapes
│   │   module-info.class
│   │
│   └───com
│       └───renderer
│           └───shapes
│                   Shape.class
│
├───com.renderer.shapes.curvededges
│   │   module-info.class
│   │
│   └───com
│       └───renderer
│           └───shapes
│               └───curvededges
│                       Sphere.class
│
└───com.renderer.shapes.polyhedrons
 │   module-info.class
    │
    └───com
        └───renderer
            └───shapes
                └───polyhedrons
 Cube.class

```

# 使用 Java 9 运行模块化代码

最后，我们可以使用 `java` 命令启动 Java 应用程序。返回 macOS 或 Linux 上的终端窗口，或者 Windows 上的命令提示符，并确保你在 `Renderer` 文件夹中。确保 `java` 命令包含在路径中，并且它是 Java 9 的 `java` 命令，而不是不兼容 Java 9 中引入的模块的先前 Java 版本的 `java` 命令。

在 macOS、Linux 或 Windows 中，运行以下命令来加载已编译的模块，解析 `com.renderer` 模块，并运行 `com.renderer` 包中声明的 `Example01` 类的 `main` 静态方法。`--module-path` 选项指定可以找到模块的目录。在这种情况下，我们只指定 `mods` 文件夹。但是，我们可以包括许多由分号 (`;`) 分隔的目录。`-m` 选项指定要解析的初始模块名称，后面跟着一个斜杠 (`/`) 和要执行的主类的名称。

```java
java --module-path mods -m com.renderer/com.renderer.Example01

```

以下行显示了执行先前命令后运行 `Example01` 类的 `main` 静态方法后生成的输出。

```java
Created camera at (x: 30, y: 30, z: 30)
Set camera direction to (x: 50, y: 0, z: 0)
Set camera vector to (x: 4, y: 5, z: 2)
Set camera perspective field of view to: 90
Set camera near clipping plane to: 20
Set camera far clipping plane to: 40
Created directional light at (x: 2, y: 2, z: 5)
Set light color to Cornflower blue
Drew sphere at (x: 20, y: 20, z: 20) with radius equal to 8
considering light at (x: 2, y: 2, z: 5)
and light's color equal to Cornflower blue
Drew cube at (x: 10, y: 10, z: 10) with edge length equal to 5
considering light at (x: 2, y: 2, z: 5)
and light's color equal to Cornflower blue

```

在以前的 Java 版本中，我们可以将许多 Java 类文件及其关联的元数据和资源聚合到一个名为 **JAR**（**Java 存档**）文件的压缩文件中。我们还可以将模块打包为包含 `module-info.class` 文件的模块化 JAR，该文件在顶层文件夹中的压缩文件中。

此外，我们可以使用 Java 链接工具 (`jlink`) 创建一个定制的运行时映像，其中只包括我们应用程序所需的模块。这样，我们可以利用整体程序优化，并生成一个在 JVM 之上运行的自定义运行时映像。

# 测试你的知识

1.  默认情况下，模块需要：

1.  `java.base` 模块。

1.  `java.lang` 模块。

1.  `java.util` 模块。

1.  有一个约定规定，Java 9 模块的源代码必须位于一个具有以下内容的目录中：

1.  与模块导出的主类相同的名称。

1.  与模块名称相同的名称。

1.  与模块导出的主类型相同的名称。

1.  以下哪个源代码文件是模块描述符：

1.  `module-def.java`

1.  `module-info.java`

1.  `module-data.java`

1.  以下是模块描述符中必须跟随模块名称的关键字：

1.  `name`

1.  `module-name`

1.  `module`

1.  模块描述符中的 `exports` 关键字后跟包名表示模块导出：

1.  包中声明的所有类。

1.  包中声明的所有类型。

1.  包中声明的所有公共类型。

# 总结

在本章中，我们学会了重构现有代码，充分利用 Java 9 的面向对象代码。我们已经为未来的需求准备好了代码，减少了维护成本，并最大程度地重用了代码。

我们学会了组织面向对象的代码。我们创建了许多 Java 源文件。我们在不同的 Java 源文件中声明了接口、抽象类和具体类。我们利用了 Java 9 中包含的新模块化特性，创建了许多具有对不同模块的依赖关系并导出特定类型的模块。我们学会了声明模块，将它们编译成 Java 字节码，并在 JShell 之外启动应用程序。

现在你已经学会在 Java 9 中编写面向对象的代码，你可以在真实的桌面应用程序、移动应用、企业应用程序、Web 服务和 Web 应用程序中使用你学到的一切。这些应用程序将最大程度地重用代码，简化维护，并且始终为未来的需求做好准备。你可以使用 JShell 轻松地原型化新的接口和类，这将提高你作为面向对象的 Java 9 开发人员的生产力。
