# 附录 A. 运行示例

以下是下载和运行本书中开发的示例源代码所需的步骤：

1.  从本书的网站([`www.packtpub.com/support`](http://www.packtpub.com/support))下载包含我们示例源代码的 ZIP 文件。将它们解压到硬盘上。解压文件时应该会创建两个目录——`Samples`和`Widgets`。这两个目录包含了本书中开发的应用程序的源代码。

1.  启动 Eclipse 3.2。创建一个名为`GWT_HOME`的新类路径变量。转到**窗口** | **首选项** | **Java** | **构建路径** | **类路径变量**。添加一个名为`GWT_HOME`的新变量条目，并将其设置为您解压 GWT 分发的目录，例如：`C:\gwt-windows-1.3.1`。这样可以确保 GWT JAR 文件对示例项目可用。

1.  逐个将这两个项目导入到您的 Eclipse 工作区。您可以通过转到**文件** | **导入** | **导入现有项目到工作区**，然后选择项目的根目录来将现有项目导入 Eclipse。`Widgets`项目用于创建打包在 JAR 文件中并被`Samples`项目使用的两个小部件。因此，它不定义入口点。您只需要运行/调试`Samples`项目。

1.  你可以在 Eclipse 内部运行`Samples`项目。转到**运行** | **运行**并选择`Samples`。这将启动熟悉的 GWT shell 并启动带有`Samples`应用程序的托管浏览器。

1.  您可以在 Eclipse 内部调试`Samples`项目。转到**调试** | **调试**并选择**Samples**。

1.  如果您安装了 Apache Ant，您可以使用`Samples.ant.xml`文件来构建应用程序并创建一个 WAR 文件，该文件可用于部署到诸如 Tomcat 之类的 Servlet 容器。

1.  您还可以运行`Samples-compile.cmd`来编译应用程序，以及运行`Samples-shell.cmd`来在 Windows 控制台上运行应用程序。
