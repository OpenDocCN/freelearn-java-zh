# 第二章 与仓库协同工作

在上一章中，你学习了如何为你的项目定义依赖项。这些依赖项大多存储在某个仓库或目录结构中。仓库通常有一个结构来支持同一依赖项的不同版本。此外，一些元数据，如模块的其他依赖项，也保存在仓库中。

在我们的构建文件中，我们必须定义依赖项仓库的位置。我们可以混合不同类型的仓库，例如 Maven 和 Ivy。我们甚至可以使用本地文件系统作为仓库。我们将看到如何在构建文件中定义和配置仓库。

此外，Gradle 还提供了配置仓库布局的选项，如果仓库使用自定义布局。我们将学习如何为使用基本身份验证的仓库提供凭据。

# 声明仓库

如果我们想在 Gradle 构建文件中添加来自仓库的依赖项，我们必须显式添加`repositories`配置块。在配置块内，我们定义仓库的位置，也许还有一些额外的配置。在以下构建文件的示例中，我们定义了一个具有自定义位置的 Maven 仓库：

```java
// Repositories configuration block,
// must be present to define and
// configure repositories to get
// dependencies in our build script.
repositories {

  // Sample Maven repository with a
  // custom location.
  maven {
    url 'http://mycompany.net/maven'
  }

}
```

我们可以在构建文件中包含多个仓库。我们甚至可以混合仓库类型，例如，包括 Ivy 仓库和本地文件系统。Gradle 支持以下类型的仓库：

| 类型 | 描述 |
| --- | --- |
| Maven JCenter 仓库 | 这是一个为 Bintray JCenter 预配置的仓库 |
| Maven 中央仓库 | 这是一个为 Maven Central 预配置的仓库 |
| Maven 本地仓库 | 这是一个为本地 Maven 仓库预配置的仓库 |
| Maven 仓库 | 这是一个待配置的 Maven 仓库，它有一个自定义位置 |
| Ivy 仓库 | 这是一个待配置的 Ivy 仓库，它有一个位置和布局 |
| 平坦目录仓库 | 这是一个本地文件系统仓库 |

我们将在稍后看到如何在构建文件中使用这些仓库。认识到 Gradle 将尝试从依赖项中下载所有工件，从找到依赖项模块描述符文件的同一仓库，这是很好的。因此，如果我们已经在构建脚本中定义了多个仓库，Gradle 仍然会使用找到模块描述符文件的第一个仓库来下载工件。

## 使用 Maven JCenter 仓库

Bintray 的 JCenter 是一个相对较新的公共 Maven 仓库，其中存储了大量的 Maven 开源依赖项。它是 Maven Central 仓库的超集，并且还包含直接发布到 JCenter 的依赖项工件。访问仓库的 URL 是[`jcenter.bintray.com`](https://jcenter.bintray.com)。Gradle 为 JCenter 提供了一个快捷方式，因此我们不需要在`repositories`配置块中自己输入 URL。快捷方式是`jcenter()`。

在下面的构建文件示例中，我们使用`jcenter()`快捷方式定义了对 Bintray 的 JCenter 仓库的引用：

```java
repositories {
  // Define Bintray's JCenter
  // repository, to find
  // dependencies.
  jcenter()
}
```

自 Gradle 2.1 版本以来，JCenter 仓库 URL 的默认协议是`https`。如果我们想使用`http`协议，我们必须为仓库设置`url`属性。在下一个构建文件示例中，我们将重新定义`url`属性：

```java
repositories {
  jcenter {
    // By default https is used as protocol,
    // but we can change it with the url
    // property.
    url = 'http://jcenter.bintray.com'
  }
}
```

可选地，我们可以为仓库定义分配一个名称。这可以适用于所有 Maven 仓库，因为 JCenter 也是一个 Maven 仓库，我们可以设置`name`属性。在下面的构建文件示例中，我们定义了多个仓库并设置了`name`属性。我们添加了一个新的任务`repositoriesInfo`，它将显示每个仓库的`name`和`URL`属性：

```java
repositories {
  // Define multiple Maven repositories.
  jcenter()

  jcenter {
    name 'Bintray JCenter legacy'
    url = 'http://jcenter.bintray.com'
  }
}

task repositoriesInfo {
  description 'Display information about repositories'

  doFirst {
    // Access repositories as collection.
    project.repositories.each {
      // Display name and URL for each
      // repository.
      println "'${it.name}' uses URL ${it.url}"
    }
  }
}
```

当我们运行`repositoriesInfo`任务时，我们得到以下输出：

```java
$ gradle -q repositoriesInfo
'BintrayJCenter' uses URL https://jcenter.bintray.com/
'Bintray JCenter legacy' uses URL http://jcenter.bintray.com

```

## 使用 Maven 中央仓库

我们可以在`repositories`配置块中配置中央 Maven 2 仓库。Gradle 提供了快捷方式方法`mavenCentral`。这配置了具有 URL `https://repo1.maven.org/maven2/`的中央 Maven 仓库。

在下一个构建文件示例中，我们将定义我们的构建的中央 Maven 2 仓库：

```java
repositories {
  // Define central Maven repository
  // to use for dependencies.
  mavenCentral()
}
```

当我们使用`mavenCentral`方法时，Gradle 2.1 使用`https`协议。如果我们想使用`http`协议，我们可以重新定义`url`属性并使用`http://repo1.maven.org/maven2/`地址。在下一个构建文件示例中，我们将重新定义`url`属性：

```java
repositories {
  mavenCentral(
    // Use http protocol for the
    // central Maven repository.
    url: 'http://repo1.maven.org/maven2/'
  )
}
```

除了更改`url`属性外，我们还可以在调用`mavenCentral`方法时设置一个可选的`name`属性。在下面的构建脚本示例中，我们为`name`属性赋值。我们添加了一个新的任务`repositoriesInfo`，用于显示配置的仓库信息：

```java
repositories {
  // Define multiple Maven repositories.
  mavenCentral()

  mavenCentral(
    name: 'HTTP Maven Central',
    url:  'http://repo1.maven.org/maven2/'
  )
}

task repositoriesInfo {
  description 'Display information about repositories'

  doFirst {
    // Access repositories as collection.
    project.repositories.each {
      // Display name and URL for each
      // repository.
      println "'${it.name}' uses URL ${it.url}"
    }
  }
}
```

让我们调用`repositoriesInfo`任务来查看输出：

```java
$ gradle -q repositoriesInfo
'MavenRepo' uses URL https://repo1.maven.org/maven2/
'HTTP Maven Central' uses URL http://repo1.maven.org/maven2/

```

## 使用 Maven 本地仓库

如果我们之前在我们的本地计算机上使用过 Maven，那么我们有一个包含已下载组件的本地 Maven 缓存。我们可以使用这个本地缓存作为 Gradle 构建的仓库，使用`mavenLocal`快捷方式方法。尽管可以使用我们的本地 Maven 缓存，但这样做并不可取，因为它使得构建依赖于本地设置。如果我们在一个由更多开发者组成的大项目上工作，那么我们不能仅依赖每个开发者的计算机上的本地 Maven 缓存作为唯一的仓库。

在下面的构建文件示例中，我们使用`mavenLocal`快捷方式方法：

```java
repositories {
  // Define the local Maven cache as
  // a repository for dependencies.
  mavenLocal()
}
```

本地 Maven 缓存的定位方式与 Maven 相同。Gradle 将尝试在`USER_HOME/.m2`或`M2_HOME/conf`中找到`settings.xml`文件，其中前者优先于后者。如果找到`settings.xml`文件，则使用文件中定义的本地 Maven 仓库的位置。如果找不到`settings.xml`文件，或者本地 Maven 仓库的位置未定义，则默认位置为`USER_HOME/.` `m2/repository`。

## 使用 Maven 仓库

我们已经学习了定义 Maven 仓库的快捷方法。如果我们有自己的 Maven 仓库，例如 Nexus 或 Artifactory，我们可以在 `repositories` 配置块中使用 `maven` 方法。使用这种方法，我们可以定义 `url` 属性来访问仓库。我们可以在下面的示例构建脚本中看到这个方法的应用：

```java
repositories {

  // Define a custom Maven repository and
  // set the url property so Gradle can look
  // for the dependency module descripts
  // and artifacts.
  maven {
    url = 'http://ourcompany.com/maven'
    // Alternative syntax is to use
    // the url method:
    // url 'http://ourcompany.com/maven'
  }

}
```

当 Gradle 在 Maven 仓库中找到模块依赖描述符时，将在这个仓库中搜索这些工件。如果工件存储在其他位置，我们使用 `artifactUrls` 属性来指定位置。这样，Gradle 将在 `url` 属性指定的位置查找依赖模块描述符，并在 `artifactUrls` 属性指定的位置查找工件。

下一个示例构建脚本将定义一个自定义的 Maven 仓库，并为工件指定多个位置：

```java
repositories {
  maven {
    // At this location at the least the
    // dependency module descriptor files
    // must be stored.
    url 'http://ourcompany.com/maven'

    // Define extra locations where artifacts
    // can be found and downloaded.
    artifactUrls 'http://ourcompany.com/jars'
    artifactUrls 'http://ourcompany.com/lib'

    // Alternative syntax is to use the
    // artifactUrls property assignment:
    // artifactUrls = [
    //   'http://ourcompany.com/jars', 'http://ourcompany.com/lib'
    // ]
  }
}
```

如果我们已经配置了带有基本身份验证的自定义 Maven 仓库，我们必须提供用户名和密码来访问该仓库。在我们的 Gradle 构建文件中，我们在 `maven` 配置的 `credentials` 块中设置用户名和密码。让我们首先将用户名和密码添加到构建文件中，然后稍后看看我们如何将这些属性外部化。下一个示例构建文件将使用 `credentials` 配置块：

```java
repositories {
  maven {
    url 'http://ourcompany.com/maven'

    // Here we assign the username and
    // password to access the repository.
    credentials {
      username = 'developer'
      password = 'secret'

      // Alternate syntax is to use
      // the username and password
      // methods.
      // username 'developer'
      // password 'secret'
    }
  }
}
```

将用户名和密码添加到构建文件中不是一个好主意，因为这个文件是与参与我们项目的所有开发者共享的。我们使用项目属性来解决这个问题，而不是使用硬编码的用户名和密码。项目属性的值可以通过命令行使用 `-P` 或 `--project-prop` 选项来设置。或者，我们可以在项目目录中创建 `gradle.properties` 文件，包含项目属性的名称和值。`gradle.properties` 文件不应放入我们项目的版本控制系统中，这样值对开发者来说是私有的。

下面的示例构建文件使用 `mavenUsername` 项目属性和 `mavenPassword` 作为 Maven 仓库的凭证：

```java
repositories {
  maven {
    name = 'Company Maven Repository'

    url 'http://ourcompany.com/maven'

    // Check that properties mavenUsername
    // and mavenPassword are set when
    // we run the script.
    verifyProperty('mavenUsername')
    verifyProperty('mavenPassword')

    credentials {
      // Use project properties instead
      // of hard coded username and password.
      username mavenUsername
      password mavenPassword
    }
  }
}

/**
* Helper method to check if project property
* with the given name is set.
*
* @param propertyName Name of the property to check
* @throws GradleException When property is not set.
*/
void verifyProperty(final String propertyName) {
  if (!hasProperty(propertyName)) {
    throw new GradleException("Property $propertyName must be set")
  }
}
```

当我们为此脚本执行任何任务时，我们应该通过命令行提供项目属性的值：

```java
$ gradle -PmavenUsername=developer -PmavenPassword=secret

```

或者，我们可以在项目目录中创建 `gradle.properties` 文件，内容如下：

```java
mavenUsername = developer
mavenPassword = secret
```

如果我们有多个使用相同自定义 Maven 仓库的项目，我们还可以创建一个带有正确凭证的 Gradle init 脚本。Gradle init 脚本在构建开始之前运行。在脚本中，我们希望为具有特定名称的 Maven 仓库设置凭证。使用 init 脚本有几种方法：

+   我们可以直接从命令行使用带有 `-I` 或 `--init-script` 选项的 init 脚本。在这里，我们指定 init 脚本的名字。

+   我们将 `init.gradle` 文件放在 `USER_HOME/.gradle` 目录中。这个文件在我们计算机上每次执行 Gradle 构建之前都会运行。

+   我们在`USER_HOME/.gradle/init.d`目录中放置了一个以`.gradle`扩展名的文件。所有来自此目录的 Gradle 初始化脚本都会在每次构建之前运行。

+   我们在`GRADLE_HOME/init.d`目录中放置了一个以`.gradle`扩展名的文件。这样，我们可以打包一个带有始终需要执行的初始化脚本的自定义 Gradle 发行版。

让我们看看下一个示例初始化脚本文件中的初始化脚本内容：

```java
// Run for all projects.
allprojects {

  // After the project is evaluated, we can access
  // the repository by name.
  afterEvaluate { project ->

    // Check if project contains a repository
    // with the given name.
    if (project.repositories.any { it.name == 'Company Maven Repository' }) {

      // Set credentials for custom Maven repository
      // with the given name.
      configure(project.repositories['Company Maven Repository']) {
        credentials {
          username 'developer'
          password 'secret'
        }
      }

    }

  }

}
```

我们必须更改我们的项目 Gradle 构建文件，因为凭证现在是通过初始化脚本设置的。我们将从项目构建文件中删除凭证。在下一个示例构建文件中，我们将删除凭证和辅助方法，以设置凭证属性。凭证由初始化脚本设置。以下代码显示了这一点：

```java
repositories {
  maven {
    name = 'Company Maven Repository'
    url 'http://ourcompany.com/maven'

    // Credentials are set via init script.
  }
}
```

## 使用平面目录仓库

Gradle 还允许使用目录作为仓库来解决依赖项。我们可以使用`flatDir`方法指定一个或多个目录。可选地，我们可以为仓库指定一个名称。在下一个示例构建文件中，我们指定了`lib`和`jars`目录作为仓库：

```java
repositories {

  // Define the directories jars and lib
  // to be used as repositories.
  flatDir {
    name 'Local lib directory'
    dirs "${projectDir}/jars", "${projectDir}/lib"
  }

  // Alternate syntax is using a Map
  // with the flatDir method.
  // flatDir name: 'Local lib directory',
  //         dirs: ["${projectDir}/jars", "${projectDir}/lib"]

}
```

当我们使用平面目录仓库时，Gradle 根据工件名称和版本解析依赖项工件。依赖项的组部分被忽略。如果我们只在项目中使用平面目录仓库，我们甚至可以在配置依赖项时省略组部分。Gradle 使用以下规则来解析依赖项：

+   `[artifact]-[version].[ext]`

+   `[artifact]-[version]-[classifier].[ext]`

+   `[artifact].[ext]`

+   `[artifact]-[classifier].[ext]`

在下一个示例构建文件中，我们将定义一个平面目录仓库和一个单一依赖项：

```java
repositories {
  flatDir name: 'Local lib directory',
      dirs: ["${projectDir}/lib"]
}

dependencies {
  traffic group: 'com.traffic', name: 'pedestrian',
      version: '1.0', classifier: 'jdk16'
}
```

Gradle 将在`lib`目录中解析以下文件；使用第一个匹配的文件：

+   `pedestrian-1.0.jar`

+   `pedestrian-1.0-jdk16.jar`

+   `pedestr``ian.jar`

+   `pedestrian-jdk16.jar`

## 使用 Ivy 仓库

Ivy 仓库允许自定义和灵活的仓库布局模式。Gradle 支持 Ivy 仓库，我们可以在我们的 Gradle 构建脚本中配置仓库布局模式。要定义一个 Ivy 仓库，我们在`repositories`配置块中使用`ivy`方法。

在以下示例构建文件中，我们定义了一个标准的 Ivy 仓库，并且我们还为仓库设置了可选的`name`属性：

```java
repositories {

  // Define an Ivy repository with
  // a URL and optional name property.
  ivy {
    name 'Ivy Repository'
    url 'http://ourompany.com/repo'
  }

}
```

Ivy 仓库的布局定义了用于搜索模块依赖元数据和依赖项的模式的布局。我们可以在我们的构建脚本中使用一些预定义的布局。在先前的示例构建文件中，我们没有指定布局。然后 Gradle 将使用默认的`gradle`布局。下表显示了我们可以使用的不同布局名称，它们用于查找 Ivy 元数据 XML 文件和依赖项的工件：

| 名称 | Ivy 模式 | 工件模式 |
| --- | --- | --- |
| gradle | `[organisation]/[module]/[revision]/ivy-[revision].xml` | `[organisation]/[module]/[revision]/[artifact]-revision(.[ext])` |
| maven | `[organisation]/[module]/[revision]/ivy-[revision].xml` | `[organisation]/[module]/[revision]/[artifact]-revision(.[ext])` |
| ivy | `[organisation]/[module]/[revision]/[type]s/artifact` | `[organisation]/[module]/[revision]/[type]s/artifact` |

`organisation` 中的 `.` 被替换为 `/`。

要使用布局，我们在 `ivy` 配置中使用 `layout` 方法。例如，在下一个构建脚本中，我们使用 `maven` 和 `ivy` 布局：

```java
repositories {

  ivy {
    // Set layout to maven.
    layout 'maven'
    name 'Ivy repository Maven layout'
    url 'http://ourcompany.com/repo1'
  }

  ivy {
    // Set layout to ivy.
    layout 'ivy'
    name 'Ivy repository'
    url 'http://ourcompany.com/repo'
  }

}
```

要为 Ivy XML 文件和工件定义自定义模式，我们使用 `pattern` 布局。使用此布局，我们使用 Ivy 定义的令牌定义自己的模式。在以下表中，我们可以看到可以用来构建模式的令牌：

| 令牌 | 描述 |
| --- | --- |
| [organisation] | 这是组织名称。 |
| [orgPath] | 这是组织名称，其中 `.` 已被替换为 `/`。这可以用来配置类似 Maven2 的仓库。 |
| [module] | 这是模块名称。 |
| [branch] | 这是分支名称。 |
| [revision] | 这是修订名称。 |
| [artifact] | 这是工件名称（或 ID）。 |
| [type] | 这是工件类型。 |
| [ext] | 这是工件文件扩展名。 |
| [conf] | 这是配置名称。 |
| [originalname] | 这是原始工件名称（包括扩展名）。 |

指定一个可选的令牌时，我们将令牌用括号（`(` 和 `)`）括起来。如果括号中定义的令牌为空或为空字符串，则忽略该令牌。例如，`artifact.[ext]` 模式在未设置 `revision` 时将接受 `artifact.jar`，如果设置了 `revision`，则接受 `artifact-1.1.jar`。

我们通过指定 `pattern` 名称并在其中添加一个配置块来定义 Ivy XML 文件和工件的模式来在我们的构建脚本中定义自定义布局。如果我们没有为 Ivy XML 文件指定特殊模式，则使用工件模式。我们需要定义至少一个工件模式。这些模式附加到仓库的 `url` 属性。可选地，我们可以设置 `pattern` 布局属性 `m2compatible`。如果值为 `true`，则 `[organisation]` 令牌中的 `.` 被替换为 `/`。

在下一个示例构建脚本中，我们将定义两个具有自定义布局的新仓库：

```java
repositories {

  ivy {
    url 'http://ourcompany.com/repo'

    // Here we define a custom pattern
    // for the artifacts and Ivy XML files.
    layout('pattern') {
      // Define pattern with artifact method.
      // This pattern is used for both Ivy XML files
      // and artifacts.
      artifact '[module]/[type]/[artifact]-[revision].[ext]'
    }
  }

  ivy {
    url 'http://ourcompany.com/repo1'

    layout('pattern') {
      // We can define multiple patterns. 
      // The order of the definitions
      // defines search path.
      artifact 'binaries/[organisation]/[module]/[artifact]-[revision].[ext]'
      artifact 'signed-jars/[organisation]/[module]/[artifact]-[revision].[ext]'

      // Seperate definition for Ivy XML files 
      // with ivy method.
      ivy '[organisation]/[module]/metadata/ivy-[revision].xml'
    }
  }

}
```

定义自定义模式的另一种语法是在 `ivy` 配置块中使用 `artifactPattern` 和 `ivyPattern`。我们不需要使用 `layout` 方法进行此定义。如果我们没有指定 `ivyPattern`，则使用 `artifactPattern` 定义的模式来查找 Ivy XML 文件。在以下示例构建脚本中，我们重写了上一个示例构建文件中的仓库定义：

```java
repositories {

  ivy {
    url 'http://ourcompany.com/repo'

    // Define pattern with artifact method.
    // This pattern is used for both Ivy XML files
    // and artifacts.
    artifactPattern '[module]/[type]/[artifact]-[revision].[ext]'
  }

  ivy {
    url 'http://ourcompany.com/repo1'

    // We can define multiple patterns. The order of the definitions
    // defines search path.
    artifactPattern 'binaries/[organisation]/[module]/[artifact]-[revision].[ext]'
    artifactPattern 'signed-jars/[organisation]/[module]/[artifact]-[revision].[ext]'

    // Seperate definition for Ivy XML files with ivy method.
    ivyPattern '[organisation]/[module]/metadata/ivy-[revision].xml'
  }

}
```

要为具有基本身份验证的 Ivy 仓库指定用户名和密码，我们使用 `credentials` 方法，就像我们在 Maven 仓库中所做的那样。在下一个示例构建文件中，我们将设置访问 Ivy 仓库的凭证。查看有关 Maven 仓库的部分，了解我们如何外部化用户名和密码，以便它们不是构建脚本代码的一部分。以下代码显示了这一点：

```java
repositories {
  ivy {
    url 'http://ourcompany.com/repo'

    // Here we assign the username and
    // password to access the repository.
    credentials {
      username = 'developer'
      password = 'secret'

      // Alternate syntax is to use
      // the username and password
      // methods.
      // username 'developer'
      // password 'secret'
    }
  }
}
```

# 使用不同的协议

Maven 和 Ivy 仓库可以通过多种协议访问。我们已经了解到我们可以使用 `http` 和 `https` 协议。然而，我们也可以使用 `file` 和 `sftp` 协议。当我们使用 `sftp` 协议时，我们必须提供凭证。`file` 协议不支持身份验证。

下一个示例构建文件将使用 `file` 和 `sftp` 协议来定义 Maven 和 Ivy 仓库：

```java
repositories {
  ivy {
    // Use file protocol, for example an
    // accessible network share or local directory.
    url 'file://Volumes/shared/developers/repo'
  }

  maven {
    url 'sftp://ourcompany.com:22/repo'

    // With the sftp protocol we must provide
    // the username and password.
    credentials {
      username 'developer'
      password 'secret'
    }
  }
}
```

# 摘要

在本章中，你学习了如何在 Gradle 构建脚本中定义仓库。你看到了如何使用预定义的快捷方法：`jcenter`、`mavenCentral` 和 `mavenLocal`。要访问位于自定义位置的 Maven 仓库，我们可以使用 `url` 属性和 `maven` 方法。当我们配置 Ivy 仓库时，我们拥有最大的控制权。我们可以指定一个 URL，也可以指定仓库的布局模式。你了解到你还可以在你的构建脚本中使用平面目录仓库。

你看到了如何为具有基本身份验证的仓库提供凭证。现在你知道了如何将用户名和密码保存在构建脚本之外。最后，你学习了如何使用不同的传输协议来访问仓库。

在下一章中，我们将看到 Gradle 将如何解析依赖项。
