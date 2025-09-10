# 第四章 发布工件

在前面的章节中，我们学习了如何在项目中定义和使用依赖关系。然而，我们在项目中编写的代码也可以成为另一个项目的依赖。为了使另一个项目能够将我们的代码作为依赖使用，我们应该将我们的代码发布为依赖项工件，以便其他项目可以使用它。

在本章中，你将学习如何在你的项目中定义工件。这些工件需要发布以便其他人可以使用它们。我们首先使用文件系统发布它们，这样工件就可以在同一台计算机上使用，甚至如果我们使用内网上的网络共享。在后面的章节中，我们将看到如何将我们的工件发布到 Maven 仓库、Ivy 仓库和 Bintray。

# 定义工件配置

一个 Gradle 项目可以包含我们想要发布的工件。一个工件可以是 ZIP 或 JAR 存档文件或任何其他文件。我们可以在一个项目中定义一个或多个工件。因此，如果我们想要从同一个源树中获取两个不同的工件，我们不需要创建两个不同的项目。

在 Gradle 中，我们使用配置来分组工件。我们曾使用配置来定义项目的依赖关系，但现在我们将使用配置来分组我们的工件，这些工件可以是其他项目的依赖项。因此，一个配置可以包含依赖项和工件。如果我们将 Java 插件应用到我们的项目中，我们得到一个名为`archives`的配置，它包含项目的默认 JAR 工件。

在以下示例 Gradle 构建文件中，我们使用了 Java 插件。我们添加了一个任务来显示属于`archives`配置的工件文件名。以下代码展示了这一点：

```java
apply plugin: 'java'

// Set the archivesBaseName property,
// to change the name of the
// default project artifact.
archivesBaseName = 'java_lib'

task artifactsInfo << {
  // Find archives configuration
  // and display file name(s)
  // for artifacts belonging
  // to the configuration.
  configurations
    .findByName('archives')
    .allArtifacts
    .each { artifact ->
      println artifact.file.name
    }
}
```

当我们从命令行运行`artifactsInfo`任务时，我们在输出中看到`java_lib.jar`文件名。以下代码展示了这一点：

```java
$ gradle artifactsInfo
:artifactsInfo
java_lib.jar

BUILD SUCCESSFUL

Total time: 1.088 secs

```

对于我们项目中的每个配置，Gradle 都会为项目添加两个任务：`build<ConfigurationName>`和`upload<ConfigurationName>`。`build<ConfigurationName>`任务为给定的配置名称创建工件。`upload<ConfigurationName>`任务为给定的配置名称创建和上传工件。`upload<ConfigurationName>`任务需要额外的配置来知道上传工件的位置。我们将在本章后面看到如何配置这个任务。

在我们的示例项目中，我们有`buildArchives`和`uploadArchives`任务。让我们运行我们的示例项目的`buildArchives`任务，看看哪些任务被执行：

```java
$ gradle buildArchives
:compileJava
:processResources
:classes
:jar
:buildArchives

BUILD SUCCESSFUL

Total time: 1.209 secs
$ ls build/libs
java_lib.jar

```

在这里，我们可以看到首先在我们的 Java 项目中准备了一切以创建 JAR 工件。然后，JAR 工件被添加到`artifacts`配置中。创建的`java_lib.jar` JAR 文件可以在`build/libs`目录中找到。

如果我们为我们的项目设置 `version` 属性，那么它将被用于我们创建的工件名称中。在下一个示例构建文件中，我们将设置 `version` 属性并查看为工件创建的名称：

```java
apply plugin: 'java'

archivesBaseName = 'java_lib'

// Set project version,
// which is then used in the 
// artifact name.
version = '2.3'

task artifactsInfo << {
configurations
    .findByName('archives')
    .allArtifacts
    .each { artifact ->
      println artifact.file.name
    }
}
```

让我们运行 `artifactsInfo` 任务来查看我们工件的名称：

```java
$ gradle artifactsInfo
:artifactsInfo
java_lib-2.3.jar

BUILD SUCCESSFUL

Total time: 2.831 secs

```

# 定义工件

在前面的章节中，你了解到 Java 插件添加了一个 `archives` 配置，用于从项目中分组工件。就像我们在项目中创建依赖项配置一样，我们也可以为其工件创建自己的配置。要将归档或文件分配给此配置，我们必须在我们的构建脚本中使用 `artifacts` 配置块。在配置闭包内部，我们使用配置名称后跟工件名称。我们还可以在 `artifacts` 块内部进一步自定义工件定义。

我们可以使用以下三种类型来定义工件：

| 类型 | 描述 |
| --- | --- |
| `AbstractArchiveTask` | 工件的信息是从归档任务中提取的。工件是 `org.gradle.api.artifact` 包中的 `PublishArtifact` 的一个实例。 |
| `File` | 工件的信息是从文件名中提取的。工件是扩展 `PublishArtifact` 的 `ConfigurablePublishArtifact` 的一个实例。 |
| `Map` | 这是定义文件工件的另一种方式。该映射必须包含一个 `file` 键，其他属性用于进一步配置工件。 |

## 使用归档任务

在下一个示例构建文件中，我们将使用归档任务来定义项目的工件。记住将 Gradle 基础插件应用到项目中非常重要，因为基础插件为 `build<ConfigurationName>` 和 `upload<ConfigurationName>` 添加了任务规则。以下代码展示了这一点：

```java
// The base plugin adds the
// build<ConfigurationName> and
// upload<ConfigurationName> tasks
// to our project.
apply plugin: 'base'

// Add archive task that will
// create a ZIP file with some
// contents we want to be published.
task manual(type: Zip) {
  baseName = 'manual'

  from 'src/manual'
}

// Create a new artifacts configuration
// with the name manualDistribution.
configurations {
  manualDistribution
}

// Use the manual archive task
// to define the artifact for the
// manualDistribution configuration.
// Syntax:
// configurationName archiveTask
artifacts {
  manualDistribution manual
}
```

当我们使用归档任务来定义配置的工件时，Gradle 也会为该工件添加一个任务依赖。这意味着，如果我们调用 `buildManualDistribution` 任务，Gradle 也会调用生成工件配置归档的 `manual` 任务。当我们从命令行执行任务时，我们会看到这一点。以下命令展示了这一点：

```java
$ gradle buildManualDistribution
:manual
:buildManualDistribution

BUILD SUCCESSFUL

Total time: 1.368 s
ecs

```

## 使用工件文件

除了归档任务之外，我们还可以使用文件作为工件。Gradle 将使用文件属性来定义工件名称、类型和扩展名。在以下示例构建文件中，我们使用文件作为工件：

```java
apply plugin: 'base'

configurations {
  readmeDistribution
}

artifacts {
  // Use a file as artifact.
  // Name and extension are extracted
  // from the actual file.
  readmeDistribution file('src/files/README.txt')
}
```

当我们使用文件工件表示法时，我们可以添加一个额外的配置闭包。在闭包中，我们可以设置名称、类型、扩展和分类器属性。以下代码展示了这一点：

```java
apply plugin: 'base'

configurations {
  readmeDistribution
}

artifacts {
  // Define file artifact, but we also
  // customize the file artifact
  // name, extension and classifier.
  readmeDistribution file('src/files/README.txt'), {
    name 'PLEASE_READ_THIS'
    extension ''
    classifier 'docs'
  }
}
```

在文件艺术品配置闭包中，我们可以使用的一个有趣的方法是 `builtBy` 方法。此方法接受一个或多个负责构建艺术品的任务名称。如果我们使用此方法，Gradle 可以确定当我们运行 `build<ConfigurationName>` 或 `upload<ConfigurationName>` 任务时需要执行的任务。

在下一个示例构建文件中，我们将使用 `builtBy` 方法：

```java
apply plugin: 'base'

configurations {
  readmeDistribution
}

// New task that copies
// a file to the build directory.
task docFiles(type:Copy) {
  from 'src/files'
  into "${buildDir}/docs"
  include 'README.txt'
}

artifacts {
  // Define file artifact.
  readmeDistribution(file("${buildDir}/docs/README.txt")) {
    // Define which task is responsible
    // for creating the file, so a
    // task dependency is added for
    // the buildReadmeDistribution and
    // uploadReadmeDistribution tasks.
    builtBy docFiles
  }
}
```

为了确保 `docFiles` 任务被添加为任务依赖项，我们从命令行运行 `buildReadmeDistribution`。以下命令展示了这一点：

```java
$ gradle buildReadmeDistribution
:docFiles
:buildReadmeDistribution

BUILD SUCCESSFUL

Total time: 0.864 secs

```

最后，当我们定义文件艺术品时，我们可以使用映射符号。我们使用 `file` 属性来定义文件。我们还可以使用 `name`、`extension`、`type`、`classifier` 和 `builtBy` 键来定义。在以下示例构建文件中，我们使用了映射符号：

```java
apply plugin: 'base'

configurations {
  readmeDistribution
}

task docFiles(type:Copy) {
  from 'src/files'
  into "${buildDir}/docs"
  include 'README.txt'
}

artifacts {
  // Define file artifact.
  readmeDistribution(
    file: "${buildDir}/docs/README.txt",
    name: 'DO_READ',
    extension: 'me',
    type: 'text',
    classifier: 'docs'
    builtBy: docFiles
  )
}
```

# 创建艺术品

我们看到了如何定义艺术品，但我们也需要在我们的构建文件中创建艺术品。我们可以使用归档任务来创建艺术品，或者文件本身可以是一个艺术品。大多数时候，当我们使用 Gradle 进行 Java 项目开发时，我们会构建一个包含编译类和资源的归档。实际上，Java 插件会为我们项目添加一个 `jar` 任务，它只会做这件事。创建的 JAR 文件随后被添加到 `archives` 配置中。

在下一个示例构建文件中，我们将使用 Java 插件并简单地依赖默认的艺术品配置和任务。以下代码展示了这一点：

```java
apply plugin: 'java'

// Define project properties.
group = 'com.mrhaki.sample'
version = '2.1'
archivesBaseName = 'sample'

// Extra task to check the artifacts.
task artifactsInfo << {
  configurations
    .findByName('archives')
    .allArtifacts
    .each { artifact ->
      println artifact.file.name
    }
}
```

我们现在可以运行 `buildArchives` 任务，并从命令行使用 `artifactsInfo` 任务检查艺术品：

```java
$ gradle buildArchives artifactsInfo
:compileJava
:processResources
:classes
:jar
:buildArchives
:artifactsInfo
sample-2.1.jar

BUILD SUCCESSFUL

Total time: 7.643 secs
$

```

在这种情况下，我们有一个单一的艺术品；然而，当我们使用 Gradle 时，在同一个项目中我们可以有多个艺术品。例如，我们可能想要将我们的源打包到一个 JAR 文件中，以及我们的生成文档。这两个 JAR 文件都应该包含在 `archives` 配置中，这样当我们执行 `buildArchives` 任务时，所有创建这些 JAR 文件所需的任务都会执行。

我们扩展了之前的构建文件示例，添加了创建两个额外 JAR 文件的代码，并将它们添加到 `archives` 艺术品配置中。以下代码展示了这一点：

```java
apply plugin: 'java'

// Define project properties.
group = 'com.mrhaki.sample'
version = '2.1'
archivesBaseName = 'sample'

// Create a JAR file with the
// Java source files.
task sourcesJar(type: Jar) {
  classifier = 'sources'

  from sourceSets.main.allJava
}

// Create a JAR file with the output
// of the javadoc task.
task javadocJar(type: Jar) {
  classifier = 'javadoc'

  from javadoc
}

artifacts {
  // Add the new archive tasks
  // to the artifacts configuration.
  archives sourcesJar, javadocJar
}

// Extra task to check the artifacts.
task artifactsInfo << {
  configurations
    .findByName('archives')
    .allArtifacts
    .each { artifact ->
      println artifact.file.name
    }
}
```

我们现在将执行 `buildArchives` 和 `artifactsInfo` 任务。我们在输出中看到我们的两个新任务 `sourcesJar` 和 `javadocJar` 已执行。生成的艺术品文件是 `sample-2.1.jar`、`sample-2.1-sources.jar` 和 `sample-2.1-javadoc.jar`。以下命令展示了这一点：

```java
$ gradle buildArchives artifactsInfo
:compileJava
:processResources
:classes
:jar
:javadoc
:javadocJar
:sourcesJar
:buildArchives
:artifactsInfo
sample-2.1.jar
sample-2.1-sources.jar
sample-2.1-javadoc.jar

BUILD SUCCESSFUL

Total time: 2.945 secs
$

```

在前面的示例中，我们有一个 Java 项目，并且从同一个源集中，我们想要创建两个不同的归档文件。源集包含一些 API 类和实现类。我们想要有一个包含 API 类的 JAR 文件，以及一个包含实现类的 JAR 文件。以下代码展示了这一点：

```java
apply plugin: 'java'

// Define project properties.
group = 'com.mrhaki.sample'
version = '2.1'
archivesBaseName = 'sample'

// We create a new source set
// api, which contains the
// Java sources. This means
// Gradle will search for the
// directory src/api/java.
sourceSets {
  api
}

task apiJar(type: Jar) {
  appendix = 'api'

  // We use the output of the
  // compilation of the api
  // source set, to be the
  // contents of this JAR file.
  from sourceSets.api.output
}

artifacts {
  // Assign apiJar archive task to the
  // archives configuration.
  archives apiJar
}

// Extra task to check the artifacts.
task artifactsInfo << {
  configurations
    .findByName('archives')
    .allArtifacts
    .each { artifact ->
      println artifact.file.name
    }
}
```

我们现在将运行 `buildArchives` 任务，并查看创建包含 `api` 源集类别的 JAR 文件所需的所有任务是否都已执行：

```java
$ gradle buildArchives artifactsInfo
:compileApiJava
:processApiResources
:apiClasses
:apiJar
:compileJava
:processResources
:classes
:jar
:buildArchives
:artifactsInfo
sample-2.1.jar
sample-api-2.1.jar

BUILD SUCCESSFUL

Total time: 2.
095 secs
$

```

# 将工件发布到本地目录

我们现在知道了如何创建一个或多个工件，以及如何使用工件配置来分组它们。在本节中，我们将了解如何将我们的工件复制到本地目录或网络共享。请记住，对于每个工件的配置，Gradle 都会添加一个`build<ConfigurationName>`任务和一个`upload<ConfigurationName>`任务。现在是时候更多地了解`upload<ConfigurationName>`任务，以便我们可以复制我们的工件。在接下来的章节中，我们还将学习如何部署到 Maven 仓库、Ivy 仓库以及 Bintray。

对于每个`upload<ConfigurationName>`任务，我们必须配置一个仓库定义。仓库定义基本上是我们上传或发布工件时的目的地。在本节中，我们使用本地目录，因此我们使用`flatDir`方法定义一个仓库。我们指定一个名称和目录，以便 Gradle 知道`upload<ConfigurationName>`任务的输出需要放在哪里。在我们应用了 Java 插件的 Gradle 项目中，我们已经有`archives`工件配置和`uploadArchives`任务。我们必须配置`uploadArchives`任务并定义需要使用的仓库。在下一个示例构建文件中，我们将使用`lib-repo`本地目录作为仓库目录：

```java
apply plugin: 'java'

// Define project properties.
group = 'com.mrhaki.sample'
version = '2.1'
archivesBaseName = 'sample'

// Configure the uploadArchives task.
uploadArchives {
  // Define a local directory as the
  // upload repository. The artifacts
  // must be 'published' in this
  // directory.
  repositories {
    flatDir(
      name: 'upload-repository',
      dirs: "${projectDir}/lib-repo")
  }
}
```

让我们看看执行`uploadArchives`任务时的输出，并检查`lib-repo`目录中的文件：

```java
$ gradle uploadArchives
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:jar UP-TO-DATE
:uploadArchives

BUILD SUCCESSFUL

Total time: 3.424 secs
$ ls -1 lib-repo
ivy-2.1.xml
ivy-2.1.xml.sha1
sample-2.1.jar
sample-2.1.jar.sha1
$

```

在我们的`lib-repo`目录中，对于我们的工件，我们有一个名为`ivy-2.1.xml`的 Ivy 描述符文件，以及一个名为`ivy-2.1.xml.sha1`的校验和文件。我们还看到了我们的`sample-2.1.jar`工件及其`sample-2.1.jar.sha1`校验和文件。Ivy 描述符文件包含有关我们的工件的基本信息。以下代码展示了这一点：

```java
<?xml version="1.0" encoding="UTF-8"?>
<ivy-module version="2.0" >
  <info organisation="com.mrhaki.sample" module="java" revision="2.1" status="integration" publication="20141126060840">
    <description/>
  </info>
  <configurations>
    <conf name="archives" visibility="public" description="Configuration for archive artifacts."/>
    <conf name="compile" visibility="private" description="Compile classpath for source set 'main'."/>
    <conf name="default" visibility="public" description="Configuration for default artifacts." extends="runtime"/>
    <conf name="runtime" visibility="private" description="Runtime classpath for source set 'main'." extends="compile"/>
    <conf name="testCompile" visibility="private" description="Compile classpath for source set 'test'." extends="compile"/>
    <conf name="testRuntime" visibility="private" description="Runtime classpath for source set 'test'." extends="runtime,testCompile"/>
  </configurations>
  <publications>
    <artifact name="sample" type="jar" ext="jar" conf="archives,runtime"/>
  </publications>
</ivy-module>
```

我们已经在`uploadArchives`任务配置中配置了仓库。然而，我们也可以通过使用`repositories`配置块来引用项目中配置的现有仓库定义。这是一个好习惯，因为我们只需要定义一次仓库，就可以在构建文件中的多个任务中重用它。让我们重写之前的示例构建文件，在`repositories`配置块中定义仓库，并在`uploadArchives`任务中引用它。以下代码展示了这一点：

```java
apply plugin: 'java'

// Define project properties.
group = 'com.mrhaki.sample'
version = '2.1'
archivesBaseName = 'sample'

// Define upload repository.
repositories {
  flatDir(
    name: 'upload-repository',
    dirs: "${projectDir}/repo")
}

// Configure the uploadArchives task.
uploadArchives {
  // Refer to repository with the
  // name 'upload-repository' as the
  // repository for uploading artifacts.
  repositories.add(
    project.repositories.'upload-repository')
}
```

## 排除描述符文件

默认情况下，Ivy 描述符文件会被添加到上传位置。如果我们不希望它被添加，我们可以为`Upload`任务设置`uploadDescriptor`属性。

在下面的示例构建文件中，我们在`uploadArchives`任务中将`uploadDescriptor`属性设置为`false`：

```java
apply plugin: 'java'

// Define project properties.
group = 'com.mrhaki.sample'
version = '2.1'
archivesBaseName = 'sample'

// Define upload repository.
repositories {
  flatDir(
    name: 'upload-repository',
    dirs: "${projectDir}/lib-repo")
}

uploadArchives {
  // Exclude the descriptor file.
  uploadDescriptor = false

  repositories.add(
    project.repositories.'upload-repository')
}
```

当我们执行任务并查看`lib-repo`目录中的文件时，我们看到描述符文件没有被添加。以下代码展示了这一点：

```java
$ gradle uploadArchives
:compileJava
:processResources
:classes
:jar
:uploadArchives

BUILD SUCCESSFUL

Total time: 1.463 secs
$ ls -1 lib-repo
sample-2.1.jar
sam
ple-2.1.jar.sha1
$

```

# 签署工件

我们可以使用 Gradle 的签名插件对工件进行数字签名。该插件支持生成 **Pretty Good Privacy** (**PGP**) 签名。这种签名格式也是发布到 Maven Central 仓库所必需的。要创建 PGP 签名，我们必须在我们的计算机上安装一些 PGP 工具。这些工具的安装方式因操作系统而异。在类 Unix 系统上，软件可能通过包管理器提供。使用 PGP 软件，我们需要创建一个密钥对，我们可以使用它来签名工件。

要对工件进行签名，我们必须将签名插件应用到我们的项目中。然后我们必须使用 `signing` 配置块来配置插件。我们需要至少添加有关我们的 PGP 密钥对的信息。我们需要公钥的十六进制表示，包含我们的私钥的秘密密钥环文件的路径，以及用于保护私钥的密码短语。我们将这些信息分配给签名插件配置的 `keyId`、`secretKeyRingFile` 和 `password` 属性。这些值不应包含在 Gradle 构建文件中，因为它们是秘密的，所以最好将它们存储在 `gradle.properties` 文件中，并对该文件应用安全的文件权限。此外，我们不应将此文件添加到我们的版本控制系统中。

在以下示例 `gradle.properties` 文件中，我们设置了属性。这些值是示例值，并且对每个用户都不同：

```java
signing.keyId = 8B00165A
signing.secretKeyRingFile = /Users/current/.gnupg/secring.gpg
signing.password = secret
```

## 使用配置进行签名

我们已经准备好对工件进行签名。我们需要使用 `signing` 配置块来配置我们想要签名的工件。我们必须指定包含要签名的工件的工件配置名称。

当我们将 Java 插件应用到我们的项目中时，我们会得到 `archives` 工件配置。我们希望对这个配置分配的工件进行签名。在下一个示例构建文件中，我们应用了 Java 和签名插件。在 `signing` 配置块中，我们定义了我们要对属于 `archives` 配置的工件进行签名：

```java
apply plugin: 'java'
apply plugin: 'signing'

group = 'com.mrhaki.sample'
version = '2.1'
archivesBaseName = 'sample'

// Configure signing plugin.
signing {
  // Define that we want to
  // sign the artifacts belonging
  // to the archives configuration.
  sign configurations.archives
}

uploadArchives {
  repositories {
    flatDir(
      name: 'local-repo',
      dirs: "${projectDir}/repo")
  }
}
```

签名插件还为我们项目添加了一个新的任务规则——`sign<ConfigurationName>`。配置的名称是我们定义在 `signing` 配置块中的名称。我们定义了 `archives` 配置，因此，在我们的项目中，我们现在可以执行 `signArchives` 任务。该任务也被添加为 `assemble` 任务的依赖项；因此，每次我们调用 `assemble` 任务时，Gradle 都会确保 `signArchives` 任务也被调用。

在这里，我们运行 `uploadArchives` 任务以查看哪些文件被放入了仓库目录：

```java
$ gradle uploadArchives
:compileJava
:processResources
:classes
:jar
:signArchives
:uploadArchives

BUILD SUCCESSFUL

Total time: 4.305 secs
$ ls -1 repo
ivy-2.1.xml
ivy-2.1.xml.sha1
sample-2.1.asc
sample-2.1.asc.sha1
sample-2.1.jar
sample-2.1.jar.sha1
$

```

我们注意到，与签名文件 `sample-2.1.asc` 一起创建了一个用于签名文件的校验和文件 `sample-2.1.asc.sha1`。

## 使用存档任务进行签名

要对不属于工件配置的工件进行签名，我们必须对签名插件进行不同的配置。在`signing`配置块中，我们在上一节中分配了一个配置，但我们也可以使用归档任务。当我们调用`sign<TaskName>`任务规则时，这个归档任务的输出将被签名。

在下一个示例构建文件中，我们将使用`manualZip`任务创建一个 ZIP 文件。我们将为`manualZip`任务配置签名插件，以便这个 ZIP 文件被签名：

```java
apply plugin: 'signing'

version = '1.0'

// New archive task to create
// a ZIP file from some files.
task manualZip(type: Zip) {
  archivesBaseName = 'manual'
  from 'src/docroot'
}

// Configure signing plugin to
// sign the output of the
// manualZip task.
signing {
  sign manualZip
}

// Create new configuration for
// ZIP and signed ZIP artifacts.
configurations {
  manualDistribution
}

// Set artifacts to manualDistribution
// configuration.
artifacts {
  manualDistribution(
    manualZip,
    signManualZip.singleSignature.file)
}

// Configure upload task for
// manualDistribution configuration.
uploadManualDistribution {
  repositories {
    flatDir {
      dirs "${projectDir}/repo"
    }
  }
}
// Add task dependency so signing of
// ZIP file is done before upload.
uploadManualDistribution.dependsOn signManualZip
```

所有`sign<TaskName>`任务自动具有对归档任务标识符的任务依赖，通过`<TaskName>`。因此，我们现在可以简单地调用`uploadManualDistribution`任务，ZIP 文件将被创建、签名并上传到`repo`目录。以下代码展示了这一点：

```java
$ gradle uploadManualDistribution
:manualZip
:signManualZip
:uploadManualDistribution

BUILD SUCCESSFUL

Total time: 1.695 secs
$ ls -1 repo
ivy-1.0.xml
ivy-1.0.xml.sha1
manual-1.0.zip
manual-1.0.zip-1.0.asc
manual-1.0.zip-1.0.asc.sha1
manual-1.0.zip.sha1
$

```

# 摘要

在前面的章节中，你学习了如何使用外部依赖。在本章中，你学习了如何定义工件配置以分配你自己的工件。这些工件可以是其他项目和其他应用中其他开发者的依赖。

你还学习了在使用 Java 插件时如何创建默认工件。接下来，我们看到了如何从一个项目中创建多个工件。

你随后学习了如何配置一个`上传`任务，以便你可以将你的工件上传到本地目录。这个目录也可以是其他开发团队可访问的网络共享。

最后，你学习了如何使用签名插件对你的工件进行签名。当你希望向使用工件的人提供额外信心时，这可能很有用。

在接下来的章节中，你将看到如何将你的工件上传到 Maven 仓库、Ivy 仓库和 Bintray。
