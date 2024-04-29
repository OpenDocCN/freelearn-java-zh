# 使用 MailFilter 进行电子邮件和垃圾邮件管理

在计算机科学中，我们有许多**定律**，其中最著名的可能是摩尔定律，它涉及计算机处理能力增加的速度。另一条定律，虽然不那么著名，当然也不那么严肃，被称为**Zawinski 定律**。杰米·扎温斯基，以其在网景和 Mozilla 的角色而闻名，曾指出“每个程序都试图扩展到可以读取邮件的程度。那些无法扩展到这一程度的程序将被可以的程序所取代。”尽管 Zawinski 定律并不像摩尔定律那样准确，但似乎确实有一定的真实性，不是吗？

本章将关注电子邮件，看看我们是否能解决困扰我们所有人的问题：电子邮件杂乱。从垃圾邮件到邮件列表的帖子，这些消息不断涌现，不断堆积。

我有几个电子邮件账户。作为家里的负责人和极客，我经常被委托管理我们的数字资产，即使他们没有意识到，而一小部分垃圾邮件可能看起来微不足道，但随着时间的推移，它可能成为一个真正的问题。在某个时候，处理起来似乎几乎不可能。

在本章中，我们将解决这个非常真实的问题，尽管可能有些夸张。这将给我们一个完美的借口来使用标准的 Java 电子邮件 API，适当地称为 JavaMail。

在本章中，我们将涵盖以下主题：

+   - JavaMail API

+   - 电子邮件协议

+   - 一些更多的 JavaFX 工作（当然）

+   - 使用 Quartz 在 Java 中创建作业计划

+   安装 Java 编写的特定于操作系统的服务

也许你已经很好地控制了你的电子邮件收件箱，如果是这样，恭喜你！然而，无论你的邮件客户端是多么整洁或令人不知所措，我们在本章中应该在探索小而强大的 JavaMail API 和电子邮件的美妙世界时玩得开心。

# 入门

在我们深入了解应用程序之前，让我们停下来快速看一下电子邮件涉及的内容。尽管电子邮件是如此普遍的工具，似乎对大多数人来说，甚至是技术上有心的人来说，它似乎是一个相当不透明的话题。如果我们要使用它，了解它将非常有帮助，即使只是一点点。如果你对协议的细节不感兴趣，那么可以跳到下一节。

# - 电子邮件协议的简要历史

像许多伟大的计算概念一样，**电子邮件**--**电子邮件**--最早是在 1960 年代引入的，尽管当时看起来大不相同。电子邮件的详细历史，虽然当然是一个很大的技术好奇心，但超出了我们在这里的目的范围，但我认为看一看今天仍然相关的一些电子邮件协议会很有帮助，其中包括用于发送邮件的 SMTP，以及用于（从您的电子邮件客户端的角度）接收邮件的 POP3 和 IMAP。（从技术上讲，电子邮件是通过 SMTP 由**邮件传输代理**（**MTA**）接收的，以将邮件从一个服务器传输到另一个服务器。我们非 MTA 作者从不以这种方式考虑，因此我们不需要过分担心这种区别）。

我们将从发送电子邮件开始，因为本章的重点将更多地放在文件夹管理上。SMTP（简单邮件传输协议）于 1982 年创建，最后更新于 1998 年，是发送电子邮件的主要协议。通常，在 SSL 和 TLS 安全连接的时代，客户端通过端口 587 连接到 SMTP 服务器。服务器和客户端之间的对话，通常称为对话，可能看起来像这样（摘自 SMTP RFC [`tools.ietf.org/html/rfc5321`](https://tools.ietf.org/html/rfc5321)）：

```java
    S: 220 foo.com Simple Mail Transfer Service Ready
    C: EHLO bar.com
    S: 250-foo.com greets bar.com
    S: 250-8BITMIME
    S: 250-SIZE
    S: 250-DSN
    S: 250 HELP
    C: MAIL FROM:<Smith@bar.com>
    S: 250 OK
    C: RCPT TO:<Jones@foo.com>
    S: 250 OK
    C: RCPT TO:<Green@foo.com>
    S: 550 No such user here
    C: RCPT TO:<Brown@foo.com>
    S: 250 OK
    C: DATA
    S: 354 Start mail input; end with <CRLF>.<CRLF>
    C: Blah blah blah...
    C: ...etc. etc. etc.
    C: .
    S: 250 OK
    C: QUIT
    S: 221 foo.com Service closing transmission channel

```

在这个简单的例子中，客户端与服务器握手，然后告诉邮件是从谁那里来的，发给谁。请注意，电子邮件地址列出了两次，但只有这些第一次出现的地方（`MAIL FROM`和`RCPT TO`，后者为每个收件人重复）才重要。第二组只是用于电子邮件的格式和显示。注意到这个特殊之处，实际的电子邮件在`DATA`行之后，这应该是相当容易理解的。一行上的孤立句号标志着消息的结束，此时服务器确认收到消息，我们通过说`QUIT`来结束。这个例子看起来非常简单，而且确实如此，但当消息有附件（如图像或办公文档）或者电子邮件以 HTML 格式进行格式化时，情况会变得更加复杂。

SMTP 用于发送邮件，而 POP3 协议用于检索邮件。POP，或者说是邮局协议，最早是在 1984 年引入的。当前标准的大部分 POP3 是在 1988 年引入的，并在 1996 年发布了更新。POP3 服务器旨在接收或下载客户端（如 Mozilla Thunderbird）的邮件。如果服务器允许，客户端可以在端口 110 上进行未加密连接，通常在端口 995 上进行安全连接。

POP3 曾经是用户下载邮件的主要协议。它快速高效，一度是我们唯一的选择。文件夹管理是必须在客户端上完成的，因为 POP3 将邮箱视为一个大存储区，没有文件夹的概念（POP4 旨在添加一些文件夹的概念，但在几年内没有对拟议的 RFC 取得任何进展）。POP3（RC 1939，位于[`tools.ietf.org/html/rfc1939`](https://tools.ietf.org/html/rfc1939)）给出了这个示例对话：

```java
    S: <wait for connection on TCP port 110>
    C: <open connection>
    S:    +OK POP3 server ready <1896.697170952@dbc.mtview.ca.us>
    C:    APOP mrose c4c9334bac560ecc979e58001b3e22fb
    S:    +OK mrose's maildrop has 2 messages (320 octets)
    C:    STAT
    S:    +OK 2 320
    C:    LIST
    S:    +OK 2 messages (320 octets)
    S:    1 120
    S:    2 200
    S:    .
    C:    RETR 1
    S:    +OK 120 octets
    S:    <the POP3 server sends message 1>
    S:    .
    C:    DELE 1
    S:    +OK message 1 deleted
    C:    RETR 2
    S:    +OK 200 octets
    S:    <the POP3 server sends message 2>
    S:    .
    C:    DELE 2
    S:    +OK message 2 deleted
    C:    QUIT
    S:    +OK dewey POP3 server signing off (maildrop empty)
    C:  <close connection>
    S:  <wait for next connection>

```

请注意，客户端发送`RETR`命令来检索消息，然后发送`DELE`命令来从服务器中删除它。这似乎是大多数 POP3 客户端的标准/默认配置。

尽管如此，许多客户端可以配置为在服务器上保留邮件一定数量的天数，或者永久保留，可能在本地删除邮件时从服务器中删除邮件。如果你以这种方式管理你的邮件，你会亲眼看到这如何使电子邮件管理变得复杂。

例如，在没有笔记本电脑的时代，想象一下你在办公室有一台台式电脑，在家里也有一台。你希望能够在两个地方都阅读你的电子邮件，所以你在两台机器上都设置了 POP3 客户端。你在工作日里阅读、删除，也许还分类邮件。当你回家时，那些在工作中处理的 40 封邮件现在都在你的收件箱里，用粗体字标记为未读邮件。如果你希望保持两个客户端的状态相似，你现在必须在家里重复你的电子邮件管理任务。这是繁琐且容易出错的，这导致我们创建了 IMAP。

**IMAP**或**Internet Access Message Protocol**，创建于 1986 年，其设计目标之一是允许多个客户端完全管理邮箱、文件夹等。多年来，它经历了几次修订，IMAP 4 修订 1 是当前的标准。客户端通过端口 143 连接到 IMAP 服务器进行未加密连接，通过端口 993 连接到 SSL 到 TLS 的连接。

IMAP，因为它提供比 POP 更强大的功能，所以是一个更复杂的协议。从 RFC（[`tools.ietf.org/html/rfc3501`](https://tools.ietf.org/html/rfc3501)）中，我们可以看到以下示例对话：

```java
    S:   * OK IMAP4rev1 Service Ready 
    C:   a001 login mrc secret 
    S:   a001 OK LOGIN completed 
    C:   a002 select inbox 
    S:   * 18 EXISTS 
    S:   * FLAGS (\Answered \Flagged \Deleted \Seen \Draft) 
    S:   * 2 RECENT 
    S:   * OK [UNSEEN 17] Message 17 is the first unseen message 
    S:   * OK [UIDVALIDITY 3857529045] UIDs valid 
    S:   a002 OK [READ-WRITE] SELECT completed 
    C:   a003 fetch 12 full 
    S:   * 12 FETCH (FLAGS (\Seen) INTERNALDATE 
         "17-Jul-1996 02:44:25 -0700" 
      RFC822.SIZE 4286 ENVELOPE ("Wed,
         17 Jul 1996 02:23:25 -0700 (PDT)" 
      "IMAP4rev1 WG mtg summary and minutes" 
      (("Terry Gray" NIL "gray" "cac.washington.edu")) 
      (("Terry Gray" NIL "gray" "cac.washington.edu")) 
      (("Terry Gray" NIL "gray" "cac.washington.edu")) 
      ((NIL NIL "imap" "cac.washington.edu")) 
      ((NIL NIL "minutes" "CNRI.Reston.VA.US") 
      ("John Klensin" NIL "KLENSIN" "MIT.EDU")) NIL NIL 
      "<B27397-0100000@cac.washington.edu>") 
       BODY ("TEXT" "PLAIN" ("CHARSET" "US-ASCII") NIL NIL "7BIT" 3028 
       92)) 
    S:    a003 OK FETCH completed 
    C:    a004 fetch 12 body[header] 
    S:    * 12 FETCH (BODY[HEADER] {342} 
    S:    Date: Wed, 17 Jul 1996 02:23:25 -0700 (PDT) 
    S:    From: Terry Gray <gray@cac.washington.edu> 
    S:    Subject: IMAP4rev1 WG mtg summary and minutes 
    S:    To: imap@cac.washington.edu 
    S:    cc: minutes@CNRI.Reston.VA.US, John Klensin <KLENSIN@MIT.EDU> 
    S:    Message-Id: <B27397-0100000@cac.washington.edu> 
    S:    MIME-Version: 1.0 
    S:    Content-Type: TEXT/PLAIN; CHARSET=US-ASCII 
    S: 
    S:    ) 
    S:    a004 OK FETCH completed 
    C:    a005 store 12 +flags \deleted 
    S:    * 12 FETCH (FLAGS (\Seen \Deleted)) 
    S:    a005 OK +FLAGS completed 
    C:    a006 logout 
    S:    * BYE IMAP4rev1 server terminating connection 
    S:    a006 OK LOGOUT completed 

```

正如你所看到的，这里比我们的示例 POP3 对话中有更多的细节。这也应该突显出为什么我们使用像 JavaMail 这样的 API，而不是直接打开套接字并直接与服务器通信。说到 JavaMail，让我们把注意力转向这个标准 API，看看它能为我们做些什么。

# JavaMail，用于电子邮件的标准 Java API

JavaMail API 是一组抽象，提供了一种与电子邮件一起工作的协议和平台无关的方式。虽然它是**Java 企业版**（**Java EE**）的必需部分，但它是 Java SE 的附加库，这意味着你需要单独下载它，我们将通过我们的 POM 文件处理。

本章的应用程序主要关注消息管理，但我们将花一点时间来看看如何使用 API 发送电子邮件，这样你以后如果需要的话就有东西可以使用。

要开始发送邮件，我们需要获取 JavaMail `Session`。为此，我们需要设置一些属性如下：

```java
    Properties props = new Properties(); 
    props.put("mail.smtps.host", "smtp.gmail.com"); 
    props.put("mail.smtps.auth", "true"); 
    props.put("mail.smtps.port", "465"); 
    props.put("mail.smtps.ssl.trust", "*"); 

```

我们将通过 Gmail 的服务器发送电子邮件，并且我们将使用 SMTP over SSL。有了这个`Properties`实例，我们可以创建我们的`Session`实例如下：

```java
    Session session = Session.getInstance(props,  
      new javax.mail.Authenticator() { 
      @Override 
      protected PasswordAuthentication getPasswordAuthentication() { 
        return new PasswordAuthentication(userName, password); 
      } 
    }); 

```

要登录服务器，我们需要指定凭据，我们通过匿名的`PasswordAuthentication`实例来实现。一旦我们有了`Session`实例，我们需要创建一个`Transport`如下：

```java
    transport = session.getTransport("smtps"); 
      transport.connect(); 

```

请注意，对于协议参数，我们指定了`smtps`，这告诉 JavaMail 实现我们希望使用 SMTP over SSL/TLS。现在我们准备使用以下代码块构建我们的消息：

```java
    MimeMessage message = new MimeMessage(session); 
    message.setFrom("jason@steeplesoft.com"); 
    message.setRecipients(Message.RecipientType.TO, 
      "jason@steeplesoft.com"); 
    message.setSubject("JavaMail Example"); 

```

电子邮件消息使用`MimeMessage`类建模，所以我们使用我们的`Session`实例创建一个实例。我们设置了发件人和收件人地址，以及主题。为了使事情更有趣，我们将使用`MimeBodyPart`附加一个文件，如下所示：

```java
    MimeBodyPart text = new MimeBodyPart(); 
    text.setText("This is some sample text"); 

    MimeBodyPart attachment = new MimeBodyPart(); 
    attachment.attachFile("src/test/resources/rules.json"); 

    Multipart multipart = new MimeMultipart(); 
    multipart.addBodyPart(text); 
    multipart.addBodyPart(attachment); 
    message.setContent(multipart); 

```

我们的消息将有两个部分，使用`MimeBodyPart`建模，一个是消息的正文，是简单的文本，另一个是附件。在这种情况下，我们只是附加了一个数据文件，我们稍后会看到。一旦我们定义了这些部分，我们使用`MimeMultipart`将它们组合起来，然后将其设置为我们的消息的内容，现在我们可以使用`transport.sendMessage()`方法：

```java
    transport.sendMessage(message, new Address[] { 
      new InternetAddress("jason@steeplesoft.com")}); 
      if (transport != null) { 
        transport.close();   
      }  

```

仅仅几秒钟内，你应该会在收件箱中看到以下电子邮件出现：

![](img/4dd554f8-c9ec-4b1a-8d9f-b0c28215ea48.png)

如果你想发送带有文本替代的 HTML 电子邮件，可以使用以下代码：

```java
    MimeBodyPart text = new MimeBodyPart(); 
    text.setContent("This is some sample text", "text/plain");  
    MimeBodyPart html = new MimeBodyPart(); 
    html.setContent("<strong>This</strong> is some <em>sample</em>
      <span style=\"color: red\">text</span>", "text/html"); 
    Multipart multipart = new MimeMultipart("alternative"); 
    multipart.addBodyPart(text); 
    multipart.addBodyPart(html); 
    message.setContent(multipart); 
    transport.sendMessage(message, new Address[]{ 
      new InternetAddress("jason@example.com")});

```

请注意，我们在每个`MimeBodyPart`上设置了内容，指定了 mime 类型，当我们创建`Multipart`时，我们将 alternative 作为`subtype`参数传递。如果不这样做，将会导致电子邮件显示两个部分，一个接一个，这显然不是我们想要的。如果我们正确编写了应用程序，我们应该在我们的电子邮件客户端中看到以下内容：

![](img/a6a7a1c4-d024-48ea-8a82-d24e1dbfb7b7.png)

你当然看不到红色文本，在黑白打印中，但你可以看到粗体和斜体文本，这意味着显示的是 HTML 版本，而不是文本版本。任务完成！

发送电子邮件非常有趣，但我们在这里是为了学习文件夹和消息管理，所以让我们把注意力转向那里，并且我们将从设置我们的项目开始。

# 构建 CLI

这个项目，就像其他项目一样，将是一个多模块的 Maven 项目。我们将有一个模块用于所有核心代码，另一个模块用于我们将编写的 GUI 来帮助管理规则。

要创建项目，这次我们将做一些不同的事情。我们将使用 Maven 原型从命令行创建项目，可以将其粗略地视为项目模板，这样你就可以看到如何以这种方式完成：

```java
    $ mvn archetype:generate \ -DarchetypeGroupId=
      org.codehaus.mojo.archetypes \ -DarchetypeArtifactId=pom-root -
      DarchetypeVersion=RELEASE 
      ... 
    Define value for property 'groupId': com.steeplesoft.mailfilter 
    Define value for property 'artifactId': mailfilter-master 
    Define value for property 'version':  1.0-SNAPSHOT 
    Define value for property 'package':  com.steeplesoft.mailfilter 

```

一旦 Maven 处理完成，就切换到新项目的目录`mailfilter-master`。从这里，我们可以创建我们的第一个项目，CLI：

```java
    $ mvn archetype:generate \ -DarchetypeGroupId=
      org.apache.maven.archetypes \ -DarchetypeArtifactId=
      maven-archetype-quickstart \ -DarchetypeVersion=RELEASE 
    Define value for property 'groupId': com.steeplesoft.mailfilter 
    Define value for property 'artifactId': mailfilter-cli 
    Define value for property 'version':  1.0-SNAPSHOT 
    Define value for property 'package':  com.steeplesoft.mailfilter 

```

这将在`mailfilter-master`下创建一个名为`mailfilter-cli`的新项目。我们现在可以在 NetBeans 中打开`mailfilter-cli`并开始工作。

我们需要做的第一件事是规定我们希望这个工具如何工作。在高层次上，我们希望能够为一个帐户指定任意数量的规则。这些规则将允许我们根据某些标准移动或删除电子邮件，例如发件人或电子邮件的年龄。为了保持简单，我们将所有规则范围限定为特定帐户，并将操作限制为移动和删除。

让我们首先看一下帐户可能是什么样子：

```java
    public class Account { 
      @NotBlank(message="A value must be specified for serverName") 
      private String serverName; 
      @NotNull(message = "A value must be specified for serverPort") 
      @Min(value = 0L, message = "The value must be positive") 
      private Integer serverPort = 0; 
      private boolean useSsl = true; 
      @NotBlank(message = "A value must be specified for userName") 
      private String userName; 
      @NotBlank(message = "A value must be specified for password") 
      private String password; 
      private List<Rule> rules; 

```

这基本上是一个非常简单的**POJO**（**Plain Old Java Object**），有六个属性：`serverName`，`serverPort`，`useSsl`，`userName`，`password`和`rules`。那些注释是什么呢？那些来自一个名为 Bean Validation 的库，它提供了一些注释和支持代码，允许我们以声明方式表达对值的约束，变量可以保存。这里是我们正在使用的注释及其含义：

+   `@NotBlank`：这告诉系统该值不能为空，也不能是空字符串（实际上，`string != null && !string.trim() .equals("")`）

+   `@NotNull`：这告诉系统该值不能为空

+   `@Min`：描述最小有效值

当然，还有许多其他的方法，系统定义了一种方法让您定义自己的方法，因此这是一个非常简单但非常强大的框架，用于验证输入，这带来了一个重要的观点：这些约束只有在要求 Bean Validation 框架进行验证时才会被验证。我们可以轻松地构建一个大量的`Account`实例集合，其中每个字段都保存着无效数据，JVM 对此也会非常满意。应用 Bean Validation 约束的唯一方法是要求它检查我们提供的实例。简而言之，是 API 而不是 JVM 强制执行这些约束。这似乎是显而易见的，但有时明确说明是值得的。

在我们进一步进行之前，我们需要将 Bean Validation 添加到我们的项目中。我们将使用参考实现：`Hibernate Validator`。我们还需要在我们的项目中添加表达式语言 API 和一个实现。我们通过将以下依赖项添加到`pom.xml`中来获得所有这些依赖项：

```java
    <dependency> 
      <groupId>org.hibernate</groupId> 
      <artifactId>hibernate-validator</artifactId> 
      <version>5.3.4.Final</version> 
    </dependency> 
    <dependency> 
      <groupId>javax.el</groupId> 
      <artifactId>javax.el-api</artifactId> 
      <version>2.2.4</version> 
    </dependency> 
    <dependency> 
      <groupId>org.glassfish.web</groupId> 
      <artifactId>javax.el</artifactId> 
      <version>2.2.4</version> 
    </dependency> 

```

回到我们的模型，当然有一些 getter 和 setter，但这些并不是很有趣。但有趣的是`equals()`和`hashCode()`的实现。Josh Bloch 在他的重要作品《Effective Java》中说：

当你重写`equals`时，总是要重写`hashCode`。

他的主要观点是，不这样做违反了`equals()`合同，该合同规定相等的对象必须具有相等的哈希值，这可能导致类在任何基于哈希的集合中使用时出现不正确和/或不可预测的行为，例如`HashMap`。 Bloch 然后列出了一些创建良好的`hashCode`实现以及良好的`equals`实现的规则，但这是我的建议：让 IDE 为您完成这项工作，这就是我们在以下代码块中为`equals()`所做的。

```java
    public boolean equals(Object obj) { 
      if (this == obj) { 
        return true; 
      } 
      if (obj == null) { 
        return false; 
      } 
      if (getClass() != obj.getClass()) { 
        return false; 
      } 
      final Account other = (Account) obj; 
      if (this.useSsl != other.useSsl) { 
        return false; 
      } 
      if (!Objects.equals(this.serverName, other.serverName)) { 
        return false; 
      } 
      if (!Objects.equals(this.userName, other.userName)) { 
        return false; 
      } 
      if (!Objects.equals(this.password, other.password)) { 
        return false; 
      } 
      if (!Objects.equals(this.serverPort, other.serverPort)) { 
        return false; 
      } 
      if (!Objects.equals(this.rules, other.rules)) { 
         return false; 
      } 
      return true; 
    } 

```

我们在这里也对`hashCode()`做了同样的事情：

```java
    public int hashCode() { 
      int hash = 5; 
      hash = 59 * hash + Objects.hashCode(this.serverName); 
      hash = 59 * hash + Objects.hashCode(this.serverPort); 
      hash = 59 * hash + (this.useSsl ? 1 : 0); 
      hash = 59 * hash + Objects.hashCode(this.userName); 
      hash = 59 * hash + Objects.hashCode(this.password); 
      hash = 59 * hash + Objects.hashCode(this.rules); 
      return hash; 
    } 

```

请注意，`equals()`中测试的每个方法也在`hashCode()`中使用。您的实现必须遵循这个规则，否则您最终会得到不像应该那样工作的方法。您的 IDE 可能会在生成方法时帮助您，但您必须确保您确实使用相同的字段列表，当然，如果您修改了其中一个方法，另一个方法必须相应地更新。

现在我们有了`Account`，那么`Rule`是什么样子呢？让我们看一下以下代码片段：

```java
    @ValidRule 
    public class Rule { 
      @NotNull 
      private RuleType type = RuleType.MOVE; 
      @NotBlank(message = "Rules must specify a source folder.") 
      private String sourceFolder = "INBOX"; 
      private String destFolder; 
      private Set<String> fields = new HashSet<>(); 
      private String matchingText; 
      @Min(value = 1L, message = "The age must be greater than 0.") 
      private Integer olderThan; 

```

这个类的验证是双重的。首先，我们可以看到与`Account`上看到的相同的字段级约束：`type`不能为空，`sourceFolder`不能为空，`olderThan`必须至少为 1。虽然您可能不会认识它是什么，但我们在`@ValidRule`中也有一个类级别的约束。

字段级别的约束只能看到它们所应用的字段。这意味着如果字段的有效值取决于某个其他字段的值，这种类型的约束是不合适的。然而，类级别的规则允许我们在验证时查看整个对象，因此我们可以在验证另一个字段时查看一个字段的值。这也意味着我们需要更多的代码，所以我们将从以下注解开始：

```java
    @Target({ElementType.TYPE, ElementType.ANNOTATION_TYPE}) 
    @Retention(RetentionPolicy.RUNTIME) 
    @Constraint(validatedBy = ValidRuleValidator.class) 
    @Documented 
    public @interface ValidRule { 
      String message() default "Validation errors"; 
      Class<?>[] groups() default {}; 
      Class<? extends Payload>[] payload() default {}; 
    } 

```

如果你以前从未见过注解的源代码，这是一个相当典型的例子。与其声明对象的类型为`class`或`interface`，我们使用了`@interface`，这是一个细微但重要的区别。注解的字段也有点不同，因为没有可见性修饰符，类型也不能是原始类型。注意使用了`default`关键字。

注解本身也有注解，如下所示：

+   `@Target`：这限制了这个注解可以应用的元素类型；在这种情况下，是类型和其他注解。

+   `@Retention`：这指示编译器是否应该将注解写入类文件，并在运行时可用。

+   `@Constraint`：这是一个 Bean 验证注解，标识我们的注解作为一个新的约束类型。这个注解的值告诉系统哪个`ConstraintValidator`处理这个约束的验证逻辑。

+   `@Documented`：这表明在任何类型上存在这个注解应该被视为该类型的公共 API 的一部分。

我们的`ConstraintValidator`实现来处理这个新的约束有点复杂。我们声明了这个类如下：

```java
    public class ValidRuleValidator implements  
      ConstraintValidator<ValidRule, Object> { 

```

Bean 验证为约束验证提供了一个参数化接口，该接口接受约束的类型和验证逻辑适用的对象类型。这允许您为不同的对象类型编写给定约束的不同验证器。在我们的情况下，我们可以指定`Rule`而不是`Object`。如果我们这样做，任何时候除了`Rule`之外的东西被注解为`@ValidRule`并且实例被验证，调用代码将看到一个异常被抛出。相反，我们所做的是验证被注解的类型，特别是在需要时添加约束违规。

接口要求我们也实现这个方法，但是我们这里没有工作要做，所以它有一个空的方法体，如下所示：

```java
    @Override 
    public void initialize(ValidRule constraintAnnotation) { 
    } 

```

有趣的方法叫做`isValid()`。它有点长，所以让我们一步一步地来看：

```java
    public boolean isValid(Object value,  
      ConstraintValidatorContext ctx) { 
        if (value == null) { 
          return true; 
        } 

```

第一步是确保`value`不为空。我们有两种选择：如果它是空的，返回`true`，表示没有问题，或者返回`false`，表示有问题。我们的选择取决于我们希望应用程序的行为。对于任何一种方法都可以提出合理的论点，但似乎认为将空的`Rule`视为无效是有道理的，所以让我们将这个部分的主体改为这样：

```java
    ctx.disableDefaultConstraintViolation(); 
    ctx.buildConstraintViolationWithTemplate( 
      "Null values are not considered valid Rules") 
      .addConstraintViolation(); 
    return false; 

```

我们使用指定的消息构建`ConstraintViolation`，将其添加到`ConstraintValidatorContext`，`ctx`，并返回`false`以指示失败。

接下来，我们要确保我们正在处理一个`Rule`的实例：

```java
    if (!(value instanceof Rule)) { 
      ctx.disableDefaultConstraintViolation(); 
      ctx.buildConstraintViolationWithTemplate( 
        "Constraint valid only on instances of Rule.") 
      .addConstraintViolation(); 
      return false; 
    } 

```

一旦我们确定我们有一个非空的`Rule`实例，我们就可以进入我们的验证逻辑的核心：

```java
    boolean valid = true; 
    Rule rule = (Rule) value; 
    if (rule.getType() == RuleType.MOVE) { 
      valid &= validateNotBlank(ctx, rule, rule.getDestFolder(),  
      "A destination folder must be specified."); 
    } 

```

我们想要能够收集所有的违规行为，所以我们创建一个`boolean`变量来保存当前状态，然后我们将值转换为`Rule`，以使处理实例更加自然。在我们的第一个测试中，我们确保，如果`Rule`的类型是`RuleType. MOVE`，它有一个指定的目标文件夹。我们使用这个私有方法来做到这一点：

```java
    private boolean validateNotBlank(ConstraintValidatorContext ctx,  
      String value, String message) { 
      if (isBlank(value)) { 
        ctx.disableDefaultConstraintViolation(); 
        ctx.buildConstraintViolationWithTemplate(message) 
        .addConstraintViolation(); 
        return false; 
      } 
      return true; 
   } 

```

如果`value`为空，我们添加`ConstraintViolation`，就像我们已经看到的那样，使用指定的消息，并返回`false`。如果不为空，我们返回`true`。然后这个值与`valid`进行 AND 运算，以更新`Rule`验证的当前状态。

`isBlank()`方法非常简单：

```java
    private boolean isBlank(String value) { 
      return (value == null || (value.trim().isEmpty())); 
    } 

```

这是一个非常常见的检查，实际上在逻辑上与 Bean Validation 的`@NotBlank`背后的验证器是相同的。

我们的下两个测试是相关的。逻辑是这样的：规则必须指定要匹配的文本，或者最大的天数。测试看起来像这样：

```java
     if (!isBlank(rule.getMatchingText())) { 
       valid &= validateFields(ctx, rule); 
     } else if (rule.getOlderThan() == null) { 
       ctx.disableDefaultConstraintViolation(); 
       ctx.buildConstraintViolationWithTemplate( 
         "Either matchingText or olderThan must be specified.") 
       .addConstraintViolation(); 
       valid = false; 
     } 

```

如果`Rule`指定了`matchingText`，那么我们验证`fields`是否已正确设置。如果既没有设置`matchingText`也没有设置`olderThan`，那么我们会添加一个`ConstraintViolation`，并设置`valid`为 false。我们的`fields`验证如下：

```java
    private boolean validateFields(ConstraintValidatorContext ctx, Rule rule) { 
      if (rule.getFields() == null || rule.getFields().isEmpty()) { 
        ctx.disableDefaultConstraintViolation(); 
        ctx.buildConstraintViolationWithTemplate( 
          "Rules which specify a matching text must specify the field(s)
            to match on.") 
          .addConstraintViolation(); 
        return false; 
      } 
      return true; 
    } 

```

我们确保`fields`既不是 null 也不是空。我们在这里不对`Set`字段的实际内容进行任何验证，尽管我们当然可以。

我们可能已经编写了我们的第一个自定义验证。你的反应可能是：“哇！这对于一个‘简单’的验证来说是相当多的代码”，你是对的。在你把孩子和洗澡水一起扔掉之前，想一想：Bean Validation 的价值在于你可以将潜在复杂的验证逻辑隐藏在一个非常小的注解后面。然后，你可以通过在适当的位置放置你的约束注解来简单地重用这个逻辑。逻辑在一个地方表达，一个地方维护，但在许多地方使用，非常整洁和简洁。

所以，是的，这是相当多的代码，但你只需要写一次，约束的使用者永远不需要看到它。实际上，与通常写的代码相比，这并没有太多额外的工作，但这取决于你是否认为这额外的工作值得一试。

现在我们已经快速浏览了自定义 Bean Validation 约束，让我们回到我们的数据模型。最后要展示的是`RuleType`枚举：

```java
    public enum RuleType { 
      DELETE, MOVE; 
      public static RuleType getRuleType(String type) { 
        switch(type.toLowerCase()) { 
          case "delete" : return DELETE; 
          case "move" : return MOVE; 
          default : throw new IllegalArgumentException( 
            "Invalid rule type specified: " + type); 
        } 
      } 
    } 

```

这是一个基本的 Java `enum`，有两个可能的值，`DELETE`和`MOVE`，但我们还添加了一个辅助方法，以返回给定字符串表示的适当的`RuleType`实例。这将在我们从 JSON 中解组`Rule`时帮助我们。

有了我们定义的数据模型，我们准备开始编写实用程序本身的代码。虽然 Maven 模块被称为`mailfilter-cli`，但我们在这里不会关心一个健壮的命令行界面，就像我们在前几章中看到的那样。相反，我们将提供一个与命令行的非常基本的交互，将 OS 服务留作首选的使用方式，我们稍后会看到。

在这一点上，我们将开始使用 JavaMail API，所以我们需要确保我们的项目设置正确，因此我们在`pom.xml`中添加以下代码：

```java
    <dependency> 
      <groupId>com.sun.mail</groupId> 
      <artifactId>javax.mail</artifactId> 
      <version>1.5.6</version> 
    </dependency> 

```

在我们的 IDE 中，我们创建一个新的类`MailFilter`，并创建如下的熟悉的`public static void main`方法：

```java
    public static void main(String... args) { 
      try { 
        final MailFilter mailFilter =  
          new MailFilter(args.length > 0 ? args[1] : null); 
        mailFilter.run(); 
        System.out.println("\tDeleted count: "  
          + mailFilter.getDeleted()); 
        System.out.println("\tMove count:    "  
          + mailFilter.getMoved()); 
      } catch (Exception e) { 
        System.err.println(e.getLocalizedMessage()); 
      } 
    } 

```

NetBeans 支持许多代码模板。这里感兴趣的模板是`psvm`，它将创建一个`public static void main`方法。要使用它，请确保你在类定义的空行上（以避免奇怪的格式问题），然后输入`psvm`并按 tab 键。NetBeans 会为你创建方法，并将光标放在空方法的第一行上，准备让你开始编码。你可以通过导航到工具 | 选项 | 编辑器 | 代码模板找到其他几十个有用的代码模板。你甚至可以定义自己的模板。

在我们的`main()`方法中，我们创建一个`MainFilter`的实例，传入可能在命令行中指定的任何规则定义文件，并调用`run()`：

```java
    public void run() { 
      try { 
        AccountService service = new AccountService(fileName); 

        for (Account account : service.getAccounts()) { 
          AccountProcessor processor =  
            new AccountProcessor(account); 
          processor.process(); 
          deleted += processor.getDeleteCount(); 
          moved += processor.getMoveCount(); 
        } 
      } catch (MessagingException ex) { 
        Logger.getLogger(MailFilter.class.getName()) 
        .log(Level.SEVERE, null, ex); 
      } 
    } 

```

我们首先创建一个`AccountService`的实例，它封装了读取和写入`Rules`文件的细节。对于指定文件中的每个帐户，我们创建一个`AccountProcessor`，它封装了规则处理逻辑。

`AccountService`实例可能听起来并不令人兴奋，但在这个公共接口的背后隐藏着一些非常有趣的技术细节。我们看到了 Bean Validation 约束是如何实际检查的，我们还看到了使用 Jackson JSON 库来读取和写入`Rules`文件。在我们可以开始使用 Jackson 之前，我们需要将其添加到我们的项目中，我们通过添加这个`pom.xml`来实现：

```java
    <dependency> 
      <groupId>com.fasterxml.jackson.core</groupId> 
      <artifactId>jackson-databind</artifactId> 
      <version>2.8.5</version> 
    </dependency> 

```

您应该始终确保您使用的是库的最新版本。

这不是一个很大的类，但这里只有三种方法是有趣的。我们将从最基本的方法开始，如下所示：

```java
    private File getRulesFile(final String fileName) { 
      final File file = new File(fileName != null ? fileName 
        : System.getProperty("user.home") + File.separatorChar 
        + ".mailfilter" + File.separatorChar + "rules.json"); 
      if (!file.exists()) { 
        throw new IllegalArgumentException( 
          "The rules file does not exist: " + rulesFile); 
      } 
      return file; 
    } 

```

我在这里包含的唯一原因是，从用户的主目录中读取文件是我发现自己经常做的事情，您可能也是如此。这个示例向您展示了如何做到这一点，如果用户没有明确指定文件，则尝试在`~/.mailfilter/rules.json`中找到规则文件。生成或指定，如果找不到规则文件，我们会抛出异常。

也许最有趣的方法是`getAccounts()`方法。我们将慢慢地逐步进行：

```java
    public List<Account> getAccounts() { 
      final Validator validator = Validation 
        .buildDefaultValidatorFactory().getValidator(); 
      final ObjectMapper mapper = new ObjectMapper() 
        .configure(DeserializationFeature. 
        ACCEPT_SINGLE_VALUE_AS_ARRAY, true); 
      List<Account> accounts = null; 

```

这三个语句正在设置一些处理账户所需的对象。首先是`Validator`，它是 Bean Validation 类，是我们应用和检查我们在数据模型上描述的约束的入口点。接下来是`ObjectMapper`，这是一个 Jackson 类，它将把 JSON 数据结构映射到我们的 Java 数据模型上。我们需要指定`ACCEPT_SINGLE_VALUE_AS_ARRAY`以确保 Jackson 正确处理我们模型中的任何列表。最后，我们创建`List`来保存我们的`Account`实例。

使用 Jackson 将规则文件读入内存并将其作为我们数据模型的实例非常容易：

```java
    accounts = mapper.readValue(rulesFile,  
      new TypeReference<List<Account>>() {}); 

```

由于我们 Java 类中的属性名称与我们的 JSON 文件中使用的键匹配，`ObjectMapper`可以轻松地从 JSON 文件中读取数据，并仅使用这一行构建我们的内存模型。请注意`TypeReference`实例。我们希望 Jackson 返回一个`List<Account>`实例，但由于 JVM 中的一些设计决策，直接访问运行时参数化类型是不可能的。然而，`TypeReference`类有助于捕获这些信息，Jackson 然后使用它来创建数据模型。如果我们传递`List.class`，我们将在运行时获得类型转换失败。

现在我们有了我们的`Account`实例，我们准备开始验证：

```java
    accounts.forEach((account) -> { 
      final Set<ConstraintViolation<Account>> violations =  
        validator.validate(account); 
      if (violations.size() > 0) { 
        System.out.println( 
          "The rule file has validation errors:"); 
        violations.forEach(a -> System.out.println("  \"" + a)); 
        throw new RuntimeException("Rule validation errors"); 
      } 
      account.getRules().sort((o1, o2) ->  
        o1.getType().compareTo(o2.getType())); 
    }); 

```

使用`List.forEach()`，我们遍历`List`中的每个账户（这里没有显示空值检查）。对于每个`Account`，我们调用`validator.validate()`，这是实际验证约束的时候。到目前为止，它们只是存储在类中的注释，JVM 很高兴地将它们一起携带，但不做其他任何事情。正如我们之前讨论的那样，Bean Validation 是注释描述的约束的执行者，在这里我们看到了手动 API 调用。

当对“验证器”进行调用时返回，我们需要查看是否有任何`ConstraintViolations`。如果有，我们会相当天真地将每个失败的详细信息打印到标准输出。如果规则有多个违规行为，由于我们编写的验证器，我们将一次看到它们所有，因此用户可以在不必多次尝试处理规则的情况下修复它们。将这些打印到控制台并不一定是最佳方法，因为我们无法以编程方式处理它们，但目前对我们的需求来说已经足够了。

Bean Validation 真正闪耀的是在代表您集成它的框架中。例如，JAX-RS，用于构建 REST 资源的标准 Java API，提供了这种类型的集成。我们在此示例 REST 资源方法中看到了功能的使用：

`@GET`

`public Response getSomething (`

`@QueryParam("foo") @NotNull Integer bar) {`

当一个请求被路由到这个方法时，JAX-RS 确保查询参数`foo`被转换为`Integer`（如果可能的话），并且它不是`null`，所以在你的代码中，你可以假设你有一个有效的`Integer`引用。

在这个类中我们要看的最后一个方法是`saveAccounts()`，这个方法保存了指定的`Account`实例到规则文件中。

```java
    public void saveAccounts(List<Account> accounts) { 
      try { 
        final ObjectMapper mapper =  
          new ObjectMapper().configure(DeserializationFeature. 
          ACCEPT_SINGLE_VALUE_AS_ARRAY, true); 
        mapper.writeValue(rulesFile, accounts); 
      } catch (IOException ex) { 
        // ... 
      } 
    } 

```

就像读取文件一样，写入文件也非常简单，只要你的 Java 类和 JSON 结构匹配。如果名称不同（例如，Java 类可能具有`accountName`属性，而 JSON 文件使用`account_name`），Jackson 提供了一些注解，可以应用于 POJO，以解释如何正确映射字段。你可以在 Jackson 的网站上找到这些完整的细节（[`github.com/FasterXML/jackson`](https://github.com/FasterXML/jackson)）。

当我们的`Account`实例加载到内存中并验证正确后，我们现在需要处理它们。入口点是`process()`方法：

```java
    public void process() throws MessagingException { 
      try { 
        getImapSession(); 

        for (Map.Entry<String, List<Rule>> entry :  
          getRulesByFolder(account.getRules()).entrySet()) { 
          processFolder(entry.getKey(), entry.getValue()); 
        } 
      } catch (Exception e) { 
        throw new RuntimeException(e); 
      } finally { 
        closeFolders(); 
        if (store != null) { 
          store.close(); 
        } 
      } 
    } 

```

需要注意的三行是对`getImapSession()`、`getRulesByFolder()`和`processFolder()`的调用，我们现在将详细讨论它们：

```java
    private void getImapSession()  
      throws MessagingException, NoSuchProviderException { 
      Properties props = new Properties(); 
      props.put("mail.imap.ssl.trust", "*"); 
      props.put("mail.imaps.ssl.trust", "*"); 
      props.setProperty("mail.imap.starttls.enable",  
        Boolean.toString(account.isUseSsl())); 
      Session session = Session.getInstance(props, null); 
      store = session.getStore(account.isUseSsl() ?  
        "imaps" : "imap"); 
      store.connect(account.getServerName(), account.getUserName(),  
        account.getPassword()); 
    } 

```

要获得 IMAP`Session`，就像我们在本章前面看到的那样，我们创建一个`Properties`实例并设置一些重要的属性。我们使用用户在规则文件中指定的协议来获取`Store`引用：对于非 SSL 连接使用`imap`，对于 SSL 连接使用`imaps`。

一旦我们有了我们的会话，我们就会遍历我们的规则，按源文件夹对它们进行分组：

```java
    private Map<String, List<Rule>> getRulesByFolder(List<Rule> rules) { 
      return rules.stream().collect( 
        Collectors.groupingBy(r -> r.getSourceFolder(), 
        Collectors.toList())); 
    } 

```

现在我们可以按照以下方式处理文件夹：

```java
    private void processFolder(String folder, List<Rule> rules)  
      throws MessagingException { 
      Arrays.stream(getFolder(folder, Folder.READ_WRITE) 
        .getMessages()).forEach(message -> 
        rules.stream().filter(rule ->  
        rule.getSearchTerm().match(message)) 
        .forEach(rule -> { 
          switch (rule.getType()) { 
            case MOVE: 
              moveMessage(message, getFolder( 
                rule.getDestFolder(),  
                Folder.READ_WRITE)); 
            break; 
            case DELETE: 
              deleteMessage(message); 
            break; 
          } 
      })); 
    } 

```

使用`Stream`，我们遍历源文件夹中的每条消息，过滤出只匹配`SearchTerm`的消息，但那是什么，它从哪里来？

`Rule`类上还有一些我们还没有看过的额外项目：

```java
    private SearchTerm term; 
    @JsonIgnore 
    public SearchTerm getSearchTerm() { 
      if (term == null) { 
        if (matchingText != null) { 
          List<SearchTerm> terms = fields.stream() 
          .map(f -> createFieldSearchTerm(f)) 
          .collect(Collectors.toList()); 
          term = new OrTerm(terms.toArray(new SearchTerm[0])); 
        } else if (olderThan != null) { 
          LocalDateTime day = LocalDateTime.now() 
          .minusDays(olderThan); 
          term = new SentDateTerm(ComparisonTerm.LE, 
            Date.from(day.toLocalDate().atStartOfDay() 
            .atZone(ZoneId.systemDefault()).toInstant())); 
        } 
      } 
      return term; 
    } 

```

我们添加了一个私有字段来缓存`SearchTerm`，这样我们就不必多次创建它。这是一个小的优化，但我们希望避免在大型文件夹上为每条消息重新创建`SearchTerm`而导致不必要的性能损失。如果规则设置了`matchingText`，我们将根据指定的字段创建一个`List<SearchTerm>`。一旦我们有了这个列表，我们就将它包装在`OrTerm`中，这将指示 JavaMail 在*任何*指定的字段与文本匹配时匹配消息。

如果设置了`olderThan`，那么我们创建`SentDateTerm`来匹配至少`olderThan`天前发送的任何消息。我们将`SearchTerm`引用保存在我们的私有实例变量中，然后返回它。

请注意，该方法具有`@JsonIgnore`注解。我们使用这个注解来确保 Jackson 不会尝试将此 getter 返回的值编组到 JSON 文件中。

对于好奇的人，`createFieldSearchTerm()`看起来像这样：

```java
    private SearchTerm createFieldSearchTerm(String f) { 
      switch (f.toLowerCase()) { 
        case "from": 
          return new FromStringTerm(matchingText); 
        case "cc": 
          return new RecipientStringTerm( 
            Message.RecipientType.CC, matchingText); 
        case "to": 
          return new RecipientStringTerm( 
            Message.RecipientType.TO, matchingText); 
        case "body": 
          return new BodyTerm(matchingText); 
        case "subject": 
          return new SubjectTerm(matchingText); 
        default: 
            return null; 
      } 
    } 

```

那么，消息实际上是如何移动或删除的呢？当然，JavaMail API 有一个用于此目的的 API，其使用可能看起来像这样：

```java
    private static final Flags FLAGS_DELETED =  
      new Flags(Flags.Flag.DELETED); 
    private void deleteMessage(Message toDelete) { 
      if (toDelete != null) { 
        try { 
          final Folder source = toDelete.getFolder(); 
          source.setFlags(new Message[]{toDelete},  
            FLAGS_DELETED, true); 
          deleteCount++; 
        } catch (MessagingException ex) { 
          throw new RuntimeException(ex); 
        } 
      } 
    } 

```

我们进行了一个快速的空值检查，然后我们获取了消息`Folder`的引用。有了这个引用，我们指示 JavaMail 在文件夹中的消息上设置一个`FLAGS_DELETED`标志。JavaMail API 更多地使用`Message`（`Message[]`）数组，所以我们需要将`Message`包装在数组中，然后将其传递给`setFlags()`。在完成时，我们增加了我们的已删除消息计数器，这样我们就可以在完成时打印我们的报告。

移动`Message`非常类似：

```java
    private void moveMessage(Message toMove, Folder dest) { 
      if (toMove != null) { 
        try { 
          final Folder source = toMove.getFolder(); 
          final Message[] messages = new Message[]{toMove}; 
          source.setFlags(messages, FLAGS_DELETED, true); 
          source.copyMessages(messages, dest); 
          moveCount++; 
        } catch (MessagingException ex) { 
          throw new RuntimeException(ex); 
        } 
      } 
    } 

```

这个方法的大部分看起来就像`deleteMessage()`，但有一个细微的区别。JavaMail 没有`moveMessages()`API。相反，我们需要调用`copyMessages()`来在目标文件夹中创建消息的副本，然后从源文件夹中删除消息。我们增加了移动计数器并返回。

感兴趣的最后两个方法处理文件夹。首先，我们需要获取文件夹，我们在这里这样做：

```java
    final private Map<String, Folder> folders = new HashMap<>(); 
    private Folder getFolder(String folderName, int mode) { 
      Folder source = null; 
      try { 
        if (folders.containsKey(folderName)) { 
          source = folders.get(folderName); 
        } else { 
          source = store.getFolder(folderName); 
          if (source == null || !source.exists()) { 
            throw new IllegalArgumentException( 
             "Invalid folder: " + folderName); 
          } 
          folders.put(folderName, source); 
        } 
        if (!source.isOpen()) { 
          source.open(mode); 
        } 
      } catch (MessagingException ex) { 
        //... 
      } 
      return source; 
    } 

```

出于性能原因，我们将每个“文件夹”实例缓存在`Map`中，以文件夹名称为键。如果我们在`Map`中找到“文件夹”，我们就使用它。如果没有，那么我们向 IMAP“存储”请求对所需的“文件夹”的引用，并将其缓存在`Map`中。最后，我们确保“文件夹”是打开的，否则我们的移动和删除命令将抛出异常。

当我们完成时，我们还需要确保关闭“文件夹”：

```java
    private void closeFolders() { 
      folders.values().stream() 
      .filter(f -> f.isOpen()) 
      .forEachOrdered(f -> { 
        try { 
          f.close(true); 
        } catch (MessagingException e) { 
        } 
      }); 
    } 

```

我们过滤我们的`Folder`流，只选择那些是打开的，然后调用`folder.close()`，忽略可能发生的任何失败。在处理的这一点上，没有太多可以做的。

我们的邮件过滤现在在技术上已经完成，但它并不像它本应该的那样可用。我们需要一种定期运行的方式，并且能够在 GUI 中查看和编辑规则将会非常好，所以我们将构建这两者。由于如果我们没有要运行的内容，安排某事就没有意义，所以我们将从 GUI 开始。

# 构建 GUI

由于我们希望尽可能地使其易于使用，我们现在将构建一个 GUI 来帮助管理这些规则。为了创建项目，我们将使用与创建 CLI 时相同的 Maven 原型：

```java
$ mvn archetype:generate \ -DarchetypeGroupId=org.apache.maven.archetypes \ -DarchetypeArtifactId=maven-archetype-quickstart \ -DarchetypeVersion=RELEASE 
Define value for property 'groupId': com.steeplesoft.mailfilter 
Define value for property 'artifactId': mailfilter-gui 
Define value for property 'version':  1.0-SNAPSHOT 
Define value for property 'package':  com.steeplesoft.mailfilter.gui 

```

一旦 POM 被创建，我们需要稍微编辑它。我们需要通过向`pom.xml`添加此元素来设置父级：

```java
    <parent> 
      <groupId>com.steeplesoft.j9bp.mailfilter</groupId> 
      <artifactId>mailfilter-master</artifactId> 
      <version>1.0-SNAPSHOT</version> 
    </parent> 

```

我们还将添加对 CLI 模块的依赖，如下所示：

```java
    <dependencies> 
      <dependency> 
        <groupId>${project.groupId}</groupId> 
        <artifactId>mailfilter-cli</artifactId> 
        <version>${project.version}</version> 
      </dependency> 
    </dependencies> 

```

由于我们不依赖 NetBeans 为我们生成 JavaFX 项目，我们还需要手动创建一些基本工件。让我们从应用程序的入口点开始：

```java
    public class MailFilter extends Application { 
      @Override 
      public void start(Stage stage) throws Exception { 
        Parent root = FXMLLoader.load(getClass() 
        .getResource("/fxml/mailfilter.fxml")); 
        Scene scene = new Scene(root); 
        stage.setTitle("MailFilter"); 
        stage.setScene(scene); 
        stage.show(); 
      } 

      public static void main(String[] args) { 
        launch(args); 
      } 
    } 

```

这是一个非常典型的 JavaFX 主类，所以我们将直接跳到 FXML 文件。现在，我们将使用以下代码创建一个存根：

```java
    <?xml version="1.0" encoding="UTF-8"?> 
    <?import java.lang.*?> 
    <?import java.util.*?> 
    <?import javafx.scene.*?> 
    <?import javafx.scene.control.*?> 
    <?import javafx.scene.layout.*?> 

    <AnchorPane id="AnchorPane" prefHeight="200" prefWidth="320"  

      fx:controller= 
        "com.steeplesoft.mailfilter.gui.Controller"> 
      <children> 
        <Button layoutX="126" layoutY="90" text="Click Me!"  
          fx:id="button" /> 
        <Label layoutX="126" layoutY="120" minHeight="16"  
          minWidth="69" fx:id="label" /> 
      </children> 
    </AnchorPane> 

```

最后，我们创建控制器：

```java
    public class Controller implements Initializable { 
      @Override 
      public void initialize(URL url, ResourceBundle rb) { 
      } 
    } 

```

这给了我们一个可以启动和运行的 JavaFX 应用程序，但没有做其他太多事情。在之前的章节中，我们已经详细介绍了构建 JavaFX 应用程序，所以我们不会在这里再次重复，但是在这个应用程序中有一些有趣的挑战值得一看。

为了让您了解我们正在努力的方向，这是最终用户界面的屏幕截图：

![](img/e6217bcf-fb27-4bc2-b84c-dbba3296e5d9.png)

在左侧，我们有`ListView`来显示规则文件中配置的`Account`。在`ListView`下方，我们有一些控件来编辑当前选定的`Account`。在右侧，我们有`TableView`来显示`Rule`，以及其下方类似的区域来编辑`Rule`。

当用户点击`Account`或`Rule`时，我们希望下方的表单区域填充相关信息。当用户修改数据时，`Account`/`Rule`以及`ListView`/`TableView`应该被更新。

通常，这是 JavaFX 真正擅长的领域之一，即属性绑定。我们已经在`ObservableList`中看到了一小部分：我们可以向`List`中添加项目，它会自动添加到已绑定的 UI 组件中。但是，我们现在所处的情况有点不同，因为我们的模型是一个 POJO，它不使用任何 JavaFX API，所以我们不会轻易获得该功能。让我们看看将这些东西连接在一起需要做些什么。

首先，让我们看一下`Account`列表。我们有`ObservableList`：

```java
    private final ObservableList<Account> accounts =  
      FXCollections.observableArrayList(); 

```

我们将我们的账户添加到这个`ObservableList`中，如下所示：

```java
    private void configureAccountsListView() { 
      accountService = new AccountService(); 
      accounts.addAll(accountService.getAccounts()); 

```

然后，我们绑定`List`和`ListView`，如下所示：

```java
    accountsListView.setItems(accounts); 

```

这里有一点变化。为了封装我们的 POJO 绑定设置，我们将创建一个名为`AccountProperty`的新类，我们很快会看到。尽管，让我们首先添加以下代码片段来处理`ListView`的点击：

```java
    accountProperty = new AccountProperty(); 
    accountsListView.setOnMouseClicked(e -> { 
      final Account account = accountsListView.getSelectionModel() 
      .getSelectedItem(); 
      if (account != null) { 
        accountProperty.set(account); 
      } 
    }); 

```

当用户点击`ListView`时，我们在`AccountProperty`实例上设置`Account`。在离开这个方法并查看`AccountProperty`之前，我们需要设置最后一个项目：

```java
    final ChangeListener<String> accountChangeListener =  
      (observable, oldValue, newValue) ->  
      accountsListView.refresh(); 
    serverName.textProperty().addListener(accountChangeListener); 
    userName.textProperty().addListener(accountChangeListener); 

```

我们定义了`ChangeListener`，它简单地调用`accountsListView.refresh()`，这指示`ListView`重新绘制自身。当模型本身更新时，我们希望它这样做，这是`ObservableList`不会向`ListView`冒泡的变化。接下来的两行将`Listener`添加到`serverName`和`userName`的`TextField`。这两个控件编辑`Account`上同名的属性，并且是用于生成`ListView`显示字符串的唯一两个控件，这里我们不展示。

`AccountProperty`是一个自定义的 JavaFX 属性，所以我们扩展`ObjectPropertyBase`如下：

```java
    private class AccountProperty extends ObjectPropertyBase<Account> { 

```

这提供了绑定解决方案的一部分，但繁重的工作由 JFXtras 项目的一个类`BeanPathAdapter`处理：

```java
    private final BeanPathAdapter<Account> pathAdapter; 

```

截至撰写本书时，JFXtras 库尚不兼容 Java 9。我们只需要这个库的一个类，所以我暂时将该类的源代码从 JFXtras 存储库复制到了这个项目中。一旦 JFXtras 在 Java 9 下运行，我们就可以删除这个副本。

文档将这个类描述为一个“适配器，它接受一个 POJO bean，并在内部和递归地将其字段绑定/解绑到其他`Property`组件”。这是一个非常强大的类，我们无法在这里完全覆盖它，所以我们将直接跳到我们的特定用法，如下所示：

```java
    public AccountProperty() { 
        pathAdapter = new BeanPathAdapter<>(new Account()); 
        pathAdapter.bindBidirectional("serverName",  
            serverName.textProperty()); 
        pathAdapter.bindBidirectional("serverPort",  
            serverPort.textProperty()); 
        pathAdapter.bindBidirectional("useSsl",  
            useSsl.selectedProperty(), Boolean.class); 
        pathAdapter.bindBidirectional("userName",  
            userName.textProperty()); 
        pathAdapter.bindBidirectional("password",  
            password.textProperty()); 
        addListener((observable, oldValue, newValue) -> { 
            rules.setAll(newValue.getRules()); 
        }); 
    } 

```

`BeanPathAdapter`允许我们将 JavaFX`Property`绑定到 POJO 上的属性，这些属性可以嵌套到任意深度，并使用点分隔路径表示。在我们的情况下，这些属性是`Account`对象上的顶级属性，因此路径是简短而简单的。在我们将控件绑定到属性之后，我们添加了一个`Listener`来使用`Rule`更新`ObservableList`规则。

在前面的代码中，当`ListView`中的`Account`选择发生变化时调用的`set()`方法非常简单：

```java
    @Override 
    public void set(Account newValue) { 
      pathAdapter.setBean(newValue); 
      super.set(newValue); 
    } 

```

有了这些部分，`Account`对象在我们在各种控件中输入时得到更新，`ListView`标签在编辑`serverName`和/或`userName`字段时得到更新。

现在我们需要为将显示用户配置的每个`Rule`的`TableView`做同样的事情。设置几乎相同：

```java
    private void configureRuleFields() { 
        ruleProperty = new RuleProperty(); 
        fields.getCheckModel().getCheckedItems().addListener( 
          new RuleFieldChangeListener()); 
        final ChangeListener<Object> ruleChangeListener =  
            (observable, oldValue, newValue) ->  
                rulesTableView.refresh(); 
        sourceFolder.textProperty() 
           .addListener(ruleChangeListener); 
        destFolder.textProperty().addListener(ruleChangeListener); 
        matchingText.textProperty() 
            .addListener(ruleChangeListener); 
        age.textProperty().addListener(ruleChangeListener); 
        type.getSelectionModel().selectedIndexProperty() 
            .addListener(ruleChangeListener); 
    } 

```

在这里，我们看到了相同的基本结构：实例化`RuleProperty`，创建`ChangeListener`来请求`TableView`刷新自身，并将该监听器添加到相关的表单字段。

`RuleProperty`也类似于`AccountProperty`：

```java
    private class RuleProperty extends ObjectPropertyBase<Rule> { 
      private final BeanPathAdapter<Rule> pathAdapter; 

      public RuleProperty() { 
        pathAdapter = new BeanPathAdapter<>(new Rule()); 
        pathAdapter.bindBidirectional("sourceFolder",  
          sourceFolder.textProperty()); 
        pathAdapter.bindBidirectional("destFolder",  
          destFolder.textProperty()); 
        pathAdapter.bindBidirectional("olderThan",  
          age.textProperty()); 
        pathAdapter.bindBidirectional("matchingText",  
          matchingText.textProperty()); 
        pathAdapter.bindBidirectional("type",  
          type.valueProperty(), String.class); 
        addListener((observable, oldValue, newValue) -> { 
          isSelectingNewRule = true; 
          type.getSelectionModel().select(type.getItems() 
          .indexOf(newValue.getType().name())); 

          IndexedCheckModel checkModel = fields.getCheckModel(); 
          checkModel.clearChecks(); 
          newValue.getFields().forEach((field) -> { 
            checkModel.check(checkModel.getItemIndex(field)); 
          }); 
          isSelectingNewRule = false; 
      }); 
    } 

```

这里最大的区别是创建的`Listener`。考虑到使用了来自 ControlsFX 项目的自定义控件`CheckListView`，值得注意的是逻辑：我们获取`IndexedCheckModel`，然后清除它，然后我们遍历每个字段，在`CheckModel`中找到其索引并进行检查。

我们通过`RuleFieldChangeListener`控制更新`Rule`上设置的字段值：

```java
    private class RuleFieldChangeListener implements ListChangeListener { 
      @Override 
      public void onChanged(ListChangeListener.Change c) { 
        if (!isSelectingNewRule && c.next()) { 
          final Rule bean = ruleProperty.getBean(); 
          bean.getFields().removeAll(c.getRemoved()); 
          bean.getFields().addAll(c.getAddedSubList()); 
        } 
      } 
    } 

```

`ListChangeListener`告诉我们移除了什么和添加了什么，所以我们相应地进行了处理。

GUI 还有其他几个移动部分，但我们在之前的章节中已经看到了它们的一个或另一个，所以我们在这里不再介绍它们。如果您对这些细节感兴趣，可以在本书的源代码存储库中找到它们。让我们把注意力转向我们项目的最后一部分：特定于操作系统的服务。

# 构建服务

这个项目的一个明确目标是能够定义规则来管理和过滤电子邮件，并且在大多数时间内运行，而不仅仅是在电子邮件客户端运行时。 （当然，我们无法控制运行此项目的机器被关闭，所以我们不能保证持续覆盖）。为了实现这一承诺的一部分，我们需要一些额外的部分。我们已经有了执行实际工作的系统部分，但我们还需要一种在计划中运行该部分的方法，还需要一个启动计划作业的部分。

对于调度方面，我们有许多选择，但我们将使用一个名为 Quartz 的库。Quartz 作业调度库是一个开源库，可以在 Java SE 和 Java EE 应用程序中使用。它提供了一个干净简单的 API，非常适合在这里使用。要将 Quartz 添加到我们的项目中，我们需要在`pom.xml`中进行如下操作：

```java
    <dependency> 
      <groupId>org.quartz-scheduler</groupId> 
      <artifactId>quartz</artifactId> 
      <version>2.2.3</version> 
    </dependency> 

```

API 有多简单呢？这是我们的`Job`定义：

```java
    public class MailFilterJob implements Job { 
      @Override 
      public void execute(JobExecutionContext jec)  
        throws JobExecutionException { 
        MailFilter filter = new MailFilter(); 
        filter.run(); 
      } 
    } 

```

我们扩展了`org.quartz.Job`，重写了`execute()`方法，在其中我们只是实例化了`MailFilter`并调用了`run()`。就是这么简单。定义了我们的任务之后，我们只需要安排它的执行，这将在`MailFilterService`中完成：

```java
    public class MailFilterService { 
      public static void main(String[] args) { 
        try { 
          final Scheduler scheduler =  
            StdSchedulerFactory.getDefaultScheduler(); 
          scheduler.start(); 

          final JobDetail job =  
            JobBuilder.newJob(MailFilterJob.class).build(); 
          final Trigger trigger = TriggerBuilder.newTrigger() 
          .startNow() 
          .withSchedule( 
             SimpleScheduleBuilder.simpleSchedule() 
             .withIntervalInMinutes(15) 
             .repeatForever()) 
          .build(); 
          scheduler.scheduleJob(job, trigger); 
        } catch (SchedulerException ex) { 
          Logger.getLogger(MailFilterService.class.getName()) 
          .log(Level.SEVERE, null, ex); 
        } 
      } 
    } 

```

我们首先获取对默认`Scheduler`的引用并启动它。接下来，我们使用`JobBuilder`创建一个新的任务，然后使用`TriggerBuilder`构建`Trigger`。我们告诉`Trigger`立即开始执行，但请注意，直到它实际构建并分配给`Scheduler`之前，它不会开始执行。一旦发生这种情况，`Job`将立即执行。最后，我们使用`SimpleScheduleBuilder`辅助类为`Trigger`定义`Schedule`，指定每 15 分钟运行一次，将永远运行。我们希望它在计算机关闭或服务停止之前一直运行。

如果现在运行/调试`MailFilterService`，我们可以观察`MailFilter`的运行。如果你这样做，而且你不是非常有耐心的话，我建议你将间隔时间降低到更合理的水平。

这让我们还有最后一件事：操作系统集成。简而言之，我们希望能够在操作系统启动时运行`MailFilterService`。理想情况下，我们希望不需要临时脚本来实现这一点。幸运的是，我们又有了许多选择。

我们将使用 Tanuki Software 的出色的 Java Service Wrapper 库（详情请参阅[`wrapper.tanukisoftware.com`](https://wrapper.tanukisoftware.com/)）。虽然我们可以手动构建服务工件，但我们更愿意让我们的构建工具为我们完成这项工作，当然，有一个名为`appassembler-maven-plugin`的 Maven 插件可以做到这一点。为了将它们整合到我们的项目中，我们需要修改 POM 文件的`build`部分，添加以下代码片段：

```java
    <build> 
      <plugins> 
        <plugin> 
          <groupId>org.codehaus.mojo</groupId> 
          <artifactId>appassembler-maven-plugin</artifactId> 
          <version>2.0.0</version> 

```

这个插件的传递依赖项将引入我们需要的一切 Java Service Wrapper，所以我们只需要配置我们的使用方式。我们首先添加一个执行，告诉 Maven 在打包项目时运行`generate-daemons`目标：

```java
    <executions> 
      <execution> 
        <id>generate-jsw-scripts</id> 
        <phase>package</phase> 
        <goals> 
          <goal>generate-daemons</goal> 
        </goals> 

```

接下来，我们需要配置插件，这可以通过`configuration`元素来实现：

```java
    <configuration> 
      <repositoryLayout>flat</repositoryLayout> 

```

`repositoryLayout`选项告诉插件构建一个**lib**风格的存储库，而不是 Maven 2 风格的布局，后者是一些嵌套目录。至少对于我们在这里的目的来说，这主要是一个样式问题，但我发现能够扫描生成的目录并一目了然地看到包含了什么是很有帮助的。

接下来，我们需要按照以下方式定义**守护进程**（来自 Unix 世界的另一个表示操作系统服务的术语，代表**磁盘和执行监视器**）：

```java
    <daemons> 
      <daemon> 
        <id>mailfilter-service</id> 
        <wrapperMainClass> 
          org.tanukisoftware.wrapper.WrapperSimpleApp 
        </wrapperMainClass> 
        <mainClass> 
         com.steeplesoft.mailfilter.service.MailFilterService 
        </mainClass> 
        <commandLineArguments> 
          <commandLineArgument>start</commandLineArgument> 
        </commandLineArguments> 

```

Java Service Wrapper 是一个非常灵活的系统，提供了多种包装 Java 项目的方式。我们的需求很简单，所以我们指示它使用`WrapperSimpleApp`，并指向主类`MailFilterService`。

该插件支持其他几种服务包装方法，但我们对 Java Service Wrapper 感兴趣，因此在这里我们使用`platform`元素来指定：

```java
        <platforms> 
          <platform>jsw</platform> 
        </platforms> 

```

最后，我们需要配置生成器，告诉它支持哪个操作系统：

```java
        <generatorConfigurations> 
          <generatorConfiguration> 
            <generator>jsw</generator> 
            <includes> 
              <include>linux-x86-64</include> 
              <include>macosx-universal-64</include> 
              <include>windows-x86-64</include> 
            </includes> 
          </generatorConfiguration> 
        </generatorConfigurations> 
      </daemon> 
    </daemons> 

```

每个操作系统定义都提供了一个 32 位选项，如果需要的话可以添加，但为了简洁起见，我在这里省略了它们。

现在构建应用程序，无论是通过`mvn package`还是`mvn install`，这个插件都会为我们的服务生成一个包装器，其中包含适用于配置的操作系统的二进制文件。好处是，它将为每个操作系统构建包装器，而不管实际运行构建的操作系统是什么。例如，这是在 Windows 机器上构建的输出（请注意 Linux 和 Mac 的二进制文件）：

![](img/fa36d256-0b22-4cf4-bf7f-7356b2448c3d.png)

包装器还可以做得更多，所以如果你感兴趣，可以在 Tanuki Software 的网站上阅读所有细节。

# 总结

就像这样，我们的应用程序又**完成**了。在本章中，我们涵盖了相当多的内容。我们首先学习了一些关于几种电子邮件协议（SMTP、POP3 和 IMAP4）的历史和技术细节，然后学习了如何使用 JavaMail API 与基于这些协议的服务进行交互。在这个过程中，我们发现了 Jackson JSON 解析器，并使用它来将 POJO 从磁盘转换为 POJO，并从磁盘转换为 POJO。我们使用了 ControlsFX 类`BeanPathAdapter`，将非 JavaFX 感知的 POJO 绑定到 JavaFX 控件，以及 Quartz 作业调度库来按计划执行代码。最后，我们使用 Java Service Wrapper 来创建安装工件，完成了我们的应用程序。

我希望我们留下的应用程序既有趣又有帮助。当然，如果你感到有动力，还有几种方法可以改进它。账户/规则数据结构可以扩展，以允许定义跨账户共享的全局规则。GUI 可以支持在账户的文件夹中查看电子邮件，并根据实时数据生成规则。构建可以扩展为创建应用程序的安装程序。你可能还能想到更多。随时随地查看代码并进行修改。如果你想到了有趣的东西，一定要分享出来，因为我很想看看你做了什么。

完成另一个项目（不是故意的），我们准备把注意力转向另一个项目。在下一章中，我们将在 GUI 中花费全部时间，构建一个照片管理系统。这将让我们有机会了解一些 JDK 的图像处理能力，包括新增的 TIFF 支持，这个功能应该会让图像爱好者非常高兴。翻页，让我们开始吧！
