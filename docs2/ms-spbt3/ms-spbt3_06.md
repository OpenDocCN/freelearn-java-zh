

# 第六章：高级测试策略

本章将进一步引导我们进入复杂测试方法的领域，并提供清晰的指南，以确保我们的软件可靠且健壮。它将涵盖一系列主题：从考虑**测试驱动开发**（**TDD**）的基本原理到具体内容，如考虑安全因素的单元测试网络控制器、应用程序不同部分的集成以及反应式环境测试的独特挑战。这将使你成为一个更好的开发者，能够编写覆盖所有代码的测试。这些技术为确保质量改进提供了坚实的基础，并且这些改进适用于各种软件架构 – 从经典网络应用程序到现代软件架构的反应式系统。

换句话说，通过学习这些测试策略，你不仅是为了捕捉错误或防止错误，而是为了应对现代软件开发的需求。本章概述了创建高性能、可扩展和可维护应用程序所需的一切。随着我们继续阅读本章，你将了解何时以及如何自信地应用这些测试技术，无论你的应用程序或其架构的复杂性如何。本章将为你提供必要的信息和工具，以成功地穿越不断变化的软件管理世界。

在本章中，我们将涵盖以下内容：

+   Spring Boot 中的 TDD

+   带有安全层的控制器单元测试

+   集成测试 – 将组件连接起来

+   测试反应式组件

让我们开始这段学习之旅，了解如何使你的 Spring Boot 应用程序安全且健壮！

# 技术要求

对于本章，我们需要在我们的本地机器上设置一些配置：

+   **Java 17 开发工具包**（**JDK 17**）

+   一个现代**集成开发环境**（**IDE**） – 我推荐 IntelliJ IDEA

+   **GitHub 仓库**：您可以从这里克隆与*第六章*相关的所有仓库：[`github.com/PacktPublishing/Mastering-Spring-Boot-3.0`](https://github.com/PacktPublishing/Mastering-Spring-Boot-3.0)

+   Docker Desktop

# Spring Boot 中的 TDD

当我第一次接触到 TDD 的概念时，我承认我相当怀疑。我觉得在编写代码之前先写单元测试的想法似乎很荒谬，或者说，疯狂。我和其他人一样，觉得这只是一个额外的过程，会减缓已经非常繁忙的开发周期。但现在，在探索了使用 Spring Boot 3.0 进行应用程序开发中的 TDD 之后，我知道情况并非如此。

Spring Boot 3.0 是一个非常出色的平台，可以用于基于 TDD 的工作。我刚刚接手了一个新项目，并开始基于 TDD 的概念前进。这个过程本身至少是尴尬的。它就像通过为不存在的代码编写测试来预先判断未来。然而，我继续这样做。单元测试实际上以我以前从未见过的方式推动了代码的编写。每个测试都有一个明确和定义的目的，以及编写满足它的代码，这使得每个相关的发展方法都变得专注和经过深思熟虑。

其目的是捕捉早期错误，并使代码库有组织且易于维护。以这种方式编写测试形成了一个通过**红**（编写失败的测试）、**绿**（使测试通过）和**重构**（清理代码）的循环。它推动着项目前进。在编写测试时投入的时间通过减少后续的调试和修正错误代码而得到回报。

现在，让我们实际操作，将理论应用到我们的书店应用中。在本节中，我们将练习 TDD，构建应用中的一个功能。你将学习如何首先编写测试，然后根据功能通过测试。所有这些实践经验旨在为你提供一个坚实的基础，而不仅仅是理论上的知识，在 Spring Boot 3.0 的应用开发中。

这当然是为了让你在使用 TDD 时感到舒适和自信。让我们开始吧！

## 实施 TDD

在本节中，我们将开始一个新的 TDD 之旅。作为这项任务的一部分，我们对书店应用有一个新的需求：在控制器和`Author`流程的存储库之间引入一个额外的服务层。控制器有两个 GET 方法，一个 PUT 方法，一个 POST 方法和一个 DELETE 方法。要求是创建一个名为`AuthorService.java`的类，提供控制器类需要的方法，并在`DELETE`过程中在数据库中找不到`Author`时抛出`EntityNotFound`异常。让我们通过使用 TDD 方法来完成这个任务：

1.  在`src/test/java`文件夹下的`AuthorServiceTest.java`。首先，我们将为潜在的`getAuthor`方法编写第一个测试。然而，当我们开始编写测试方法时，我们会看到我们还没有服务类，我们将创建一个名为`AuthorService.java`的空服务类。但当我们尝试自动完成`getAuthor`方法时，我们会看到没有这样的方法。因此，我们将在服务类中创建一个新的方法，如下所示：

    ```java
    public Optional<Author> getAuthor(Long id) {
    return Optional.empty();
    }
    ```

    如你所见，这种方法几乎是空的。然而，我们知道我们将有一个名为`getAuthor`的方法，接受`Id`作为参数，并返回`Optional<Author>`。在所有测试中，我们需要为这个测试准备环境，例如创建所需的数据。因此，我们将`authorRepository`类注入到服务和测试类中。现在，我们可以编写第一个测试用例：

    ```java
    @Mock
    private AuthorRepository authorRepository;
    @Test
    void givenExistingAuthorId_whenGetAuthor_thenReturnAuthor() {
    Publisher defaultPublisher = Publisher.builder().name("Packt Publishing").build();
    Author savedAuthor = Author.builder()
                    .id(1L)
                    .name("Author Name")
                    .publisher(defaultPublisher)
                    .biography("Biography of Author")
                    .build();
        when(authorRepository.findById(1L)).thenReturn(Optional.of(savedAuthor));
        Optional<Author> author = authorService.getAuthor(1L);
        assertTrue(author.isPresent(), "Author should be found");
        assertEquals(1L, author.get().getId(), "Author ID should match");
    }
    @Mock: This mocks the class and allows us to manipulate all its methods.
    ```

1.  `when`：这有助于我们操作模拟方法的返回对象。在我们的示例中，它模拟了当调用 `authorRepository.findById()` 时，它将始终返回 `savedAuthor`。

1.  `assertTrue`：这个断言断言方法返回 true。

1.  `assertEquals`：这个断言断言提供的两个值是相等的。

当我们运行 `givenExistingAuthorId_whenGetAuthor_thenReturnAuthor` 方法时，测试将失败，因为我们的方法始终返回 `Optional.empty()`。所以，我们手头有一个失败的测试；让我们去修复它。

1.  **绿色（使测试通过）**：为了通过这个测试，我们需要在我们的方法中使用仓库类：

    ```java
    public Optional<Author> getAuthor(long id) {
           return authorRepository.findById(id);
    }
    ```

    现在，它已经完成了。当我们再次运行测试时，我们会看到它会通过。

1.  **重构（清理代码）**：在这个步骤中，我们需要检查测试和源类，看看它们是否需要重构。在我们的例子中，在源类中我们不需要任何重构，但在测试方面，我们可能需要稍微整理一下。我们可以从测试类中移除对象创建，使测试用例更易于阅读。此外，我们可以在未来的其他测试类中重用该对象：

    ```java
    private Author savedAuthor;
    @BeforeEach
    void setup() {
     Publisher defaultPublisher = Publisher.builder().name("Packt Publishing").build();
     savedAuthor = Author.builder()
             .id(1L)
             .name("Author Name")
             .publisher(defaultPublisher)
             .biography("Biography of Author")
             .build();
    }
    ```

    我们引入了一个 `setup` 方法来设置公共变量，以减少测试类中的代码重复。

    在引入 `setup` 方法之后，我们的测试方法变得更加清晰，代码行数更少：

    ```java
    @Test
    void givenExistingAuthorId_whenGetAuthor_thenReturnAuthor() {
    when(authorRepository.findById(1L)).thenReturn(Optional.of(savedAuthor));
        Optional<Author> author = authorService.getAuthor(1L);
        assertTrue(author.isPresent(), "Author should be found");
        assertEquals(1L, author.get().getId(), "Author ID should match");
    }
    ```

    我们已经完成了对作者服务的 TDD 迭代。我们需要对所有方法进行相同的迭代，直到我们有一个成熟的 `AuthorService`，可以在控制器类中使用。

    在这些过程结束时，我们将拥有 GitHub 仓库中的 `AuthorServiceTest`。然而，我们还将有一些新的单元测试术语，我们将在 *讨论单元测试术语* 部分中进行讨论。

1.  我们已经留下了一步最后的工作：将 `AuthorController` 类更新为使用这个新的 (`AuthorService`) 服务，而不是使用仓库。

    我们需要将 `AuthorService` 类注入到控制器类中，并使用 `AuthorService` 中的方法，而不是使用仓库中的方法。

    你可以在 GitHub 仓库中看到更新的 `AuthorController`（[`github.com/PacktPublishing/Mastering-Spring-Boot-3.0/blob/main/Chapter-6-unit-integration-test/src/main/java/com/packt/ahmeric/bookstore/controller/AuthorController.java`](https://github.com/PacktPublishing/Mastering-Spring-Boot-3.0/blob/main/Chapter-6-unit-integration-test/src/main/java/com/packt/ahmeric/bookstore/controller/AuthorController.java)）。我想在这里提到 `delete` 方法：

    ```java
        @DeleteMapping("/{id}")
        public ResponseEntity<Object> deleteAuthor(@PathVariable Long id) {
            try {
                authorService.deleteAuthor(id);
                return ResponseEntity.ok().build();
            } catch (EntityNotFoundException e) {
                return ResponseEntity.notFound().build();
            }
        }
    }
    ```

    如你所见，我们已经替换了方法，在 `delete` 函数中，我们添加了一个异常处理程序来覆盖 `EntityNotFoundException`。

在本节中，我们学习了 TDD（测试驱动开发）以及如何在现实世界的示例中实现它。这需要一些时间，并且需要一些耐心，使其成为开发周期中的习惯。然而，一旦你学会了如何进行 TDD，你将会有更少的错误和更易于维护的代码。

## 讨论单元测试术语

让我们讨论一些单元测试中的基本术语：

+   `assertThrows()`: 这是一个在 JUnit 测试中使用的用于断言在执行代码片段期间抛出特定类型的异常的方法。当您想测试您的代码是否正确处理错误条件时，它特别有用。在我们的测试类中，您可以看到如下：

    ```java
    assertThrows(EntityNotFoundException.class, () -> authorService.deleteAuthor(1L));
    ```

    此方法接受两个主要参数：预期的异常类型和一个功能接口（通常是 lambda 表达式），其中包含预期抛出异常的代码。如果指定的异常被抛出，则测试通过；否则，测试失败。

+   `verify()`: Mockito 是我们始终在单元测试中使用的库。它有一些非常有用的方法，使我们的测试更加可靠和可读。`verify()`是其中之一；它用于检查与模拟对象发生的某些交互。它可以验证方法是否以特定的参数、特定次数调用，甚至从未被调用。这对于测试您的代码是否按预期与依赖项交互至关重要。在我们的测试类中，您可以看到如下：

    ```java
    verify(authorRepository).delete(savedAuthor);
    verify(authorRepository, times(1)).delete(savedAuthor);
    ```

+   `@InjectMocks`: Mockito 中的`@InjectMocks`注解用于创建类的实例并将模拟字段注入其中。这在您有一个依赖于其他组件或服务的类时特别有用，您想通过使用其依赖项的模拟版本来单独测试该类。以下代码片段展示了示例用法：

    ```java
    @InjectMocks
    private AuthorService authorService;
    ```

+   `@ExtendWith(MockitoExtension.class)`: 此注解与 JUnit 5 一起使用，以在测试中启用 Mockito 支持。通过在类级别声明`@ExtendWith(MockitoExtension.class)`，您允许 Mockito 在测试运行之前初始化模拟并注入它们。这使得编写更干净的测试代码并减少样板代码变得更容易。您可以在我们的测试类中看到它：

    ```java
    @ExtendWith(MockitoExtension.class)
    public class AuthorServiceTest
    {}
    ```

+   `@BeforeEach`: 在 JUnit 5 中，`@BeforeEach`注解用于方法上，指定它应该在当前测试类中的每个测试方法之前执行。它通常用于所有测试都通用的设置任务，确保每个测试从一个新鲜的状态开始。它与方法一起使用，正如您在以下代码中可以看到的：

    ```java
    @BeforeEach
    void setUp() {
       // common setup code
    }
    ```

既然我们已经了解了单元测试中的新术语，在下一节中，我们将利用我们的单元测试知识，通过学习如何测试控制器类，特别是带有安全层的控制器类来提高它。

# 带有安全层的控制器单元测试

在这个新章节中，我们将处理测试控制器类。为什么我们为控制器类采用不同的测试方法？请求和响应可以表示为 JSON 对象，类似于真实的请求和响应，而不是我们项目中的对象。这将帮助我们检查是否一切正常以接受请求，并在它们符合要求时断言 JSON 响应。我们将讨论一些新的注解，接下来，我们将逐步关注如何为`AuthorControllerTest`实现这些新注解。

## Spring MVC 控制器测试的关键注解

Spring 的`@WebMvcTest`、`@Import`、`@WithMockUser`和`@MockBean`是 Spring MVC 控制器测试中的关键角色。这些注解帮助建立测试框架，确保我们的控制器无论在独立测试还是与 Spring 的 Web 上下文和安全组件集成时都能按预期执行。让我们来看看它们：

+   `@WebMvcTest(AuthorController.class)`：`@WebMvcTest`注解用于以更集中的方式对 Spring MVC 应用程序进行单元测试。它应用于需要测试 Spring MVC 控制器的测试类。使用`@WebMvcTest`与特定的控制器类，如`AuthorController.class`，告诉 Spring Boot 仅实例化给定的控制器及其所需依赖项，而不是整个上下文。这使得测试运行更快，并且严格关注 MVC 组件。此注解会自动为测试配置 Spring MVC 基础设施。

+   `@Import(SecurityConfig.class)`：`@Import`注解允许您将额外的配置类导入 Spring 测试上下文。在控制器测试中，尤其是在与`@WebMvcTest`一起使用时，通常需要包含特定的配置类，这些配置类不是由`@WebMvcTest`自动获取的。通过指定`@Import(SecurityConfig.class)`，您明确告诉 Spring 加载您的`SecurityConfig`类。此类包含安全配置（如身份验证和授权设置），这对于测试在接近您应用程序安全设置的环境中运行是必要的。

+   `@MockBean`：Spring 应用程序上下文使用诸如 Service 和 Repository 之类的 bean，在我们的测试上下文中，我们需要模拟这些 bean。`@MockBean`将模拟对象添加到 Spring 应用程序上下文中，这些模拟对象将用于替代真实的 Service 和 Repository 对象。这对于注入模拟实现服务、存储库或控制器所依赖的任何其他组件非常有用，而无需从真实的应用程序上下文中实际加载这些 bean。

+   `@WithMockUser`：这个注解用于 Spring Security 测试，用于模拟使用模拟认证用户运行测试。此注解允许您指定模拟用户的具体细节，例如用户名、角色和权限，而无需与实际的安全环境或认证机制交互。它特别适用于控制器测试，您希望测试端点在不同认证或授权场景下的行为。通过使用 `@WithMockUser`，您可以轻松模拟不同的用户上下文，测试应用程序如何响应不同的访问级别，并确保安全约束得到正确执行。这使得它成为全面测试 Spring Boot 应用程序中受保护端点的必备工具。

对于控制器测试，尤其是在有安全层的情况下，这些注解在确保您的测试专注且快速，尽可能反映应用程序的实际运行条件方面发挥着关键作用。在下一节中，我们将将这些注解实现到测试类中。

## 使用 Spring 注解构建控制器测试

当我们开始为控制器创建测试时，使用 Spring 注解非常重要。这些注解，例如 `@WebMvcTest`、`@Import`、`@WithMockUser` 和 `@MockBean`，对于建立与我们的应用程序的 Web 层和安全设置相匹配的测试环境至关重要。本节重点介绍如何利用这些注解来为我们的控制器开发有针对性的测试。通过整合这些工具，我们的目标是平衡测试的速度和准确性，以确保我们的控制器在 Web 环境中有效运行。让我们探讨如何实际应用这些注解来模拟现实场景，并验证我们的 Spring MVC 控制器在某些情况下的功能。

在 Spring Boot 应用程序中为 `AuthorController` 创建一个全面的测试套件涉及多个步骤，从使用特定注解设置初始测试环境到为不同的用户角色和操作编写详细的测试用例。以下是实现 `AuthorControllerTest.java` 最终状态的逐步指南。

### 第 1 步 - 设置您的测试环境

要设置您的环境，请按照以下步骤操作：

1.  创建一个名为 `AuthorControllerTest.java` 的测试类文件。使用 `@WebMvcTest(AuthorController.class)` 注解该类，以专注于测试 `AuthorController`。这告诉 Spring Boot 仅配置测试所需的 MVC 组件，而不使用完整的应用程序上下文。

1.  使用 `@Import(SecurityConfig.class)` 将您的自定义安全配置包含在测试上下文中。这在测试期间准确模拟安全行为至关重要。

1.  声明所需的字段：

    +   `ApplicationContext` 用于设置 `MockMvc` 对象

    +   一个用于执行和断言 HTTP 请求的 `MockMvc` 对象

    +   使用 `@MockBean` 注解模拟控制器所依赖的任何服务或组件，例如 `AuthorService` 和 `JwtDecoder`。

    +   用于测试中 JSON 序列化和反序列化的 `ObjectMapper`

这是这一步的代码更改：

```java
@WebMvcTest(AuthorController.class)
@Import(SecurityConfig.class)
class AuthorControllerTest {
    @Autowired
    private WebApplicationContext context;
    private MockMvc mockMvc;
    @MockBean
    private AuthorService authorService;
    @MockBean
    private JwtDecoder jwtDecoder;
    private final ObjectMapper objectMapper = new ObjectMapper();
}
```

在这里，我们已经通过模拟 `AuthorService` 和 `JwtDecoder` 来设置测试类。我们将在下一节中需要时能够操纵它们。

### 第 2 步 – 初始化测试框架

实现一个带有 `@BeforeEach` 注解的设置方法，以便在每个测试之前初始化 `MockMvc` 对象：

```java
    @BeforeEach
    public void setup() {
        mockMvc = MockMvcBuilders
                .webAppContextSetup(context)
                .apply(springSecurity())
                .build();
    }
```

此方法使用 `MockMvcBuilders` 工具构建带有 Web 应用程序上下文和 Spring Security 集成的 `MockMvc` 对象。

### 第 3 步 – 编写测试用例

在设置我们的测试类之后，我们可以开始逐步编写我们的测试用例：

1.  编写针对添加和获取不同角色作者的参数化测试用例。使用 `@ParameterizedTest` 和 `@MethodSource` 来提供角色和预期的 HTTP 状态。在这些测试中，您将模拟具有不同用户角色的请求并断言预期的结果。

    +   对于添加作者，模拟 `AuthorService` 的响应并执行 POST 请求，根据角色断言状态。

    +   对于通过 ID 获取作者，模拟 `AuthorService` 的响应并执行 GET 请求，根据角色断言状态和内容。

1.  编写获取所有作者、更新作者和删除作者的测试用例。利用 `@Test` 和 `@WithMockUser` 注解指定用户详细信息。这些测试将执行以下操作：

    +   模拟服务层响应

    +   执行相关的 HTTP 请求（GET 用于获取所有作者，PUT 用于更新，DELETE 用于删除）

    +   断言预期的结果，包括状态码和，当适用时，响应体内容

您可以在控制器测试类中看到这些步骤是如何实现的。您可以在 GitHub 仓库中看到五个测试方法，这些方法验证了本节中提到的测试用例，链接为 [`github.com/PacktPublishing/Mastering-Spring-Boot-3.0/blob/main/Chapter-6-unit-integration-test/src/test/java/com/packt/ahmeric/bookstore/controller/AuthorControllerTest.java`](https://github.com/PacktPublishing/Mastering-Spring-Boot-3.0/blob/main/Chapter-6-unit-integration-test/src/test/java/com/packt/ahmeric/bookstore/controller/AuthorControllerTest.java)。我们将在下一节讨论如何测试异常处理。

### 第 4 步 – 处理异常情况

编写一个处理要删除的作者未找到场景的测试用例。模拟服务在尝试删除不存在的作者时抛出 `EntityNotFoundException`，并断言控制器正确地响应了 `404` 状态：

```java
    @Test
    @WithMockUser(username="testUser", authorities={"ROLE_ADMIN"})
    void testDeleteAuthorNotFoundWithAdminRole() throws Exception {
        Long id = 1L;
        doThrow(new EntityNotFoundException("Author not found with id: " + id))
                .when(authorService).deleteAuthor(id);
        mockMvc.perform(delete("/authors/" + id))
                .andExpect(status().isNotFound());
    }
```

在这个测试方法中，我们通过使用 `doThrow()` 方法操纵 `authorService.deleteAuthor` 方法来抛出异常，并期望得到一个“未找到”的状态响应。

### 第 5 步 – 运行测试

运行你的测试以验证所有测试都通过，并且你的控制器在各种场景和用户角色下表现如预期。

这种全面的测试方法不仅验证了`AuthorController`的功能方面，还确保了安全约束得到尊重，从而在应用程序的行为和安全性方面提供了信心。

在本节中，我们学习了如何使用具有安全配置的 Spring MVC 控制器编写控制器测试。我们已经为断言、功能和安全编写了测试。通过这些测试，我们确信我们的控制器和安全过滤器按预期工作。

在完成单元测试之旅后，我们将在下一节中关注集成测试。我们将探讨如何测试我们项目中的不同组件之间的集成/交互，包括数据库、网络服务和外部 API。

# 集成测试 – 将组件连接起来

一旦你理解了单元测试的概念，尤其是在处理棘手的网络安全层时，你可以将你的视野扩展到集成测试。在编写单元测试时，你可以将质量检查想象成对砖块进行的检查，以确保用这些砖块建造的墙能让你和家人免受雨、雪以及其他一切天气的影响。集成测试评估我们应用程序的不同部分如何协同工作。这就是完整的应用程序测试在全面运行的地方：模块交互、数据库、网络服务和与其他所有外部系统的交互都会被交叉检验，以确保一切顺利。

为什么需要集成测试？单元测试不足以证明我们的应用程序是健壮的吗？简短的答案是：不。在单元测试中，我们只是在证明方法按预期工作，但在现实生活中，这些方法并不是独立工作的。它们与其他组件交互。因此，集成测试对于查看任何组件是否受到你的更改的影响是至关重要的。

因此，系好安全带，因为我们现在要开始一段旅程。为了确保万无一失，一个大型集成系统将满足所有要求和可能的场景——这实际上让你的应用程序为现实世界和满足用户需求的最终测试做好了更好的准备。我们将通过清晰、实用的示例引导你，最终提供简洁的集成测试，确保你的高质量软件产品最终能够交付。

## 设置测试环境

集成测试的主要目的是识别和解决与应用程序不同部分之间交互相关的问题。这可能包括几个服务层、数据库或外部 API 之间的设计交互，所有这些共同的目标是让它们有目的地以相同的方式协同工作，就像函数应该执行一样。与单元测试不同，集成测试确保隔离的功能是正确的，因为它研究系统的行为。总的来说，应该从接口级别部分测试应用程序本身，以确保质量和功能完整，避免在缺陷、性能瓶颈和其他集成问题中留下接口缺失，这些问题可能来自单元测试。

在我们对作者控制器端点的集成测试策略中，我们利用这两个主要类：

+   `AbstractIntegrationTest`：这个类作为我们集成测试的基础，提供跨多个测试类共享的常见配置和设置例程。它是一个抽象类，不直接运行测试；相反，它设置测试环境。这包括为数据库配置测试容器，初始化 WireMock 以模拟外部服务，以及设置 Spring 的应用程序上下文，包括集成测试所需的必要配置文件和配置。WireMock 是我们用于模拟 REST 和**简单对象访问协议**（**SOAP**）服务等的库。通过这种模拟能力，我们可以隔离我们的组件，避免与外部连接以及这些服务的潜在故障。由于所有集成测试也需要相同的设置，我们可以使用这个抽象类。使用抽象基类有助于我们维护一个干净且**不要重复自己**（**DRY**）的测试代码库。

+   `AuthorControllerIntegrationTest`：从`AbstractIntegrationTest`扩展而来，这个类专门用于测试作者控制器端点。它从`AbstractIntegrationTest`继承了常见的测试环境设置，并添加了覆盖作者控制器功能的测试，例如创建、读取、更新和删除作者。`AuthorControllerIntegrationTest`类利用 Spring 的 MockMvc 来模拟 HTTP 请求并断言响应，确保当与其他应用程序组件（如安全层和数据库）集成时，作者控制器表现如预期。

通过以这种方式构建我们的集成测试，我们实现了一种分层测试方法，使我们能够在仍然利用测试之间共享的公共设置的同时，隔离特定组件（如作者控制器）的测试。这种组织结构使我们的测试更加高效，更容易维护，并确保我们全面测试对应用程序整体性能和可靠性至关重要的交互和集成。

## 配置集成测试的应用程序属性

在集成测试中，我们的应用程序将需要以它在真实环境中的方式运行。因此，设置应用程序属性文件至关重要。然而，我们还需要将集成测试环境与其他测试环境隔离开来。这就是我们启动一个新的`application-integration-test.properties`文件的原因。通过这种隔离，我们确保集成测试环境中的配置仅针对该环境，并且不会影响其他测试和开发环境。

我们正在添加与当前源代码中使用的相同的属性。这是因为以下参数将在我们的应用程序以集成测试配置运行时被需要：

```java
spring.security.oauth2.client.registration.keycloak.client-id=bookstore-client
spring.security.oauth2.client.registration.keycloak.client-secret=secret-client
spring.security.oauth2.client.registration.keycloak.client-name=Keycloak
spring.security.oauth2.client.registration.keycloak.provider=keycloak
spring.security.oauth2.client.registration.keycloak.scope=openid,profile,email
spring.security.oauth2.client.registration.keycloak.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.keycloak.redirect-uri={baseUrl}/login/oauth2/code/keycloak
spring.security.oauth2.client.provider.keycloak.issuer-uri=http://localhost:8180/auth/realms/BookStoreRealm
spring.security.oauth2.resourceserver.jwt.issuer-uri=http://localhost:8180/auth/realms/BookStoreRealm
```

通过配置这些属性，我们创建了一个受控、可预测和隔离的环境，使我们能够彻底和准确地测试应用程序的集成点。这种设置对于评估应用程序在模拟生产环境中的行为至关重要，确保所有组件协同工作。

接下来，我们将深入了解这些配置的实际应用，为稳健、真实环境测试做好准备。

## 使用 Testcontainers 初始化数据库

`Testcontainers`是一个为 JUnit 和系统测试设计的 Java 库。它通常在运行时提供轻量级、可丢弃的实例方式，用于共享数据库和 Selenium 网络浏览器或任何可以在 Docker 容器内运行的东西。在底层，`Testcontainers`使用 Docker 来帮助完成实际的数据库实例的设置和，特别是，拆卸，这些实例是隔离的、短暂的，并且完全受控。借助像`Testcontainers`这样的工具，可以在不增加太多开销的情况下，准确测试满足业务需求的数据库交互和持久性，无需数据库安装的复杂性或某些共享测试实例的设置。

我们现在将通过在`AbstractIntegrationTest`类中初始化 PostgreSQL 和 MongoDB 数据库的`Testcontainers`来配置 PostgreSQL 和 MongoDB 容器。以下是操作方法：

+   使用`Testcontainers`库。此方法指定要使用的 Docker 镜像（`postgres:latest`），以及数据库特定的配置，如数据库名称、用户名和密码。一旦初始化，容器就会被启动，并且**Java 数据库连接**（**JDBC**）URL、用户名和密码将被动态注入到 Spring 应用程序上下文中。这允许集成测试与一个真实的 PostgreSQL 数据库实例交互，该实例与生产中使用的实例相同。

+   通过指定 Docker 镜像（`mongo:4.4.6`）使用`Testcontainers`库。在启动 MongoDB 容器时，连接 URI 被注入到 Spring 应用程序上下文中，使测试能够与真实的 MongoDB 实例通信。

要初始化数据库，请按照以下步骤操作：

1.  首先，我们将定义所需的参数，例如数据库镜像版本和数据库名称：

    ```java
       private static final String POSTGRES_IMAGE = "postgres:latest";
        private static final String MONGO_IMAGE = "mongo:4.4.6";
        private static final String DATABASE_NAME = "bookstore";
        private static final String DATABASE_USER = "postgres";
        private static final String DATABASE_PASSWORD = "yourpassword";
        private static final int WIREMOCK_PORT = 8180;
        private static WireMockServer wireMockServer;
    ```

1.  当我们初始化我们的测试数据库容器时，我们将使用这些参数：

    ```java
        @Container
        static final PostgreSQLContainer<?> postgresqlContainer = initPostgresqlContainer();
        @Container
        static final MongoDBContainer mongoDBContainer = initMongoDBContainer();
        private static PostgreSQLContainer<?> initPostgresqlContainer() {
            PostgreSQLContainer<?> container = new PostgreSQLContainer<>(POSTGRES_IMAGE)
                    .withDatabaseName(DATABASE_NAME)
                    .withUsername(DATABASE_USER)
                    .withPassword(DATABASE_PASSWORD);
            container.start();
            return container;
        }
        private static MongoDBContainer initMongoDBContainer() {
            MongoDBContainer container = new MongoDBContainer(DockerImageName.parse(MONGO_IMAGE));
            container.start();
            return container;
        }
    ```

    在此代码块中，首先，我们定义容器并在其中调用容器初始化函数。容器初始化函数消耗在前面代码块中定义的*步骤 1*中的参数。

1.  我们需要一些动态属性，这些属性将在容器被触发后定义。通过以下代码，我们可以让应用程序知道将使用哪个数据源 URL 来连接到数据库：

    ```java
        @DynamicPropertySource
        static void properties(DynamicPropertyRegistry registry) {
            registry.add("spring.datasource.url", postgresqlContainer::getJdbcUrl);
            registry.add("spring.datasource.username", postgresqlContainer::getUsername);
            registry.add("spring.datasource.password", postgresqlContainer::getPassword);
            registry.add("spring.data.mongodb.uri", mongoDBContainer::getReplicaSetUrl);
        }
    ```

    每个容器在测试前后都会自动启动、准备和拆除，确保每个测试套件都在一个干净、隔离的数据库环境中运行。这个自动化过程提高了集成测试的可靠性和可重复性，并简化了数据库交互的设置和故障排除。

在数据库已经准备好并在容器化的沙盒中设置好之后，下一节将展示如何做到这一点：模拟外部 API 的响应，以便我们可以深入到我们的代码库中，提供一个全面、无保留的测试策略。

## 使用 WireMock 模拟外部服务

在集成测试的上下文中，模拟外部服务的能力很重要，因为你不能在集成测试环境中运行所有外部设备。即使你可以运行它们，集成测试的目的也是测试组件中的代码。外部系统的问题与应用程序代码的质量无关。WireMock 为这个挑战提供了一个强大的解决方案。通过创建模拟这些外部服务行为的可编程 HTTP 服务器，WireMock 允许开发者生成可靠、一致和快速的测试。模拟外部服务确保测试不仅从应用程序控制之外的因素中隔离出来，而且可以在任何环境中运行，无需实际服务连接。

为了有效地模拟与 OpenID Connect 提供者的交互，WireMock 可以被配置为以预定义的响应来响应身份验证和令牌请求。我们需要这个设置来测试受保护的端点，而无需与真实的身份验证服务交互。以下是实现此目的的方法：

1.  在`AbstractIntegrationTest`类中，设置一个 WireMock 服务器在特定端口上运行。这个服务器充当你的模拟 OpenID Connect 提供者。

1.  **模拟 OpenID 配置**：配置 WireMock 为 OpenID Connect 发现文档和其他相关端点提供服务。我们需要模拟端点以返回提供者元数据，包括授权、令牌、用户信息和**JSON Web Key Set**（**JWKS**）URI 的 URL。这确保了当你的应用程序尝试发现 OpenID Connect 提供者的配置时，它会从 WireMock 接收到一致和受控的响应。

1.  **模拟令牌和授权响应**：进一步配置 WireMock 以模拟响应令牌和授权请求。这些响应应模仿来自 OpenID Connect 提供者的真实响应结构，包括必要的访问令牌、ID 令牌和刷新令牌。

请参阅 GitHub 仓库中的相关抽象类，了解我们如何在[`github.com/PacktPublishing/Mastering-Spring-Boot-3.0/blob/main/Chapter-6-unit-integration-test/src/test/java/integrationtests/AbstractIntegrationTest.java`](https://github.com/PacktPublishing/Mastering-Spring-Boot-3.0/blob/main/Chapter-6-unit-integration-test/src/test/java/integrationtests/AbstractIntegrationTest.java)中模拟 key-cloak 服务器。每当我们的应用程序需要与 key-cloak 通信时，我们的模拟服务器将按预期响应我们的应用程序。

通过这种方式模拟 OpenID Connect 提供者，您可以准确且一致地测试您的应用程序的认证和授权流程，确保您的安全机制按预期工作，而不依赖于外部系统。

在为数据库交互和外部服务依赖项建立了受控环境之后，我们现在已准备好在下一节开始编写集成测试。

## 为作者控制器编写集成测试

在深入测试本身之前，确保每个测试运行都有一个干净的页面至关重要。`@BeforeEach`方法在这个过程中起着至关重要的作用，它允许我们在每次测试之前将我们的数据库重置到已知状态：

```java
@BeforeEach
void clearData() { authorRepository.deleteAll(); }
```

通过调用如`authorRepository.deleteAll()`这样的方法，我们可以清除所有数据，防止跨测试污染并确保每个测试独立运行。

### 使用@WithMockUser 保护测试

由于我们的应用程序有一个安全层，我们需要考虑这个层来编写我们的测试，即使我们有模拟的第三方安全依赖项。我们的应用程序仍然检查安全过滤器中的请求角色。我们有一个非常有用的注解：`@WithMockUser`注解允许我们模拟具有特定角色的认证用户请求，确保我们的测试准确反映了应用程序的安全约束。这样，我们可以确认我们的安全配置正在有效工作。

### 测试端点

现在，我们已准备好为每个端点编写测试。我们拥有运行中的测试数据库和模拟的第三方依赖项。现在，我们只需向`/authors`端点发送请求并断言响应。这部分与控制器单元测试非常相似，但不同之处在于我们不会模拟服务——我们将使用服务本身。所有测试都将从头到尾运行。因此，我们将确保我们的应用程序及其所有组件按预期运行。

在下面的代码块中，我们将为`Get /authors/{authorId}`端点编写一个测试用例：

```java
@Test
@WithMockUser(username="testUser", authorities={"ROLE_ADMIN"})
void testGetAuthor() throws Exception {
    Author author = Author.builder().name("Author Name").build();
    authorRepository.save(author);
    mockMvc.perform(get("/authors/" + author.getId()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name", is(author.getName())));
}
```

在这里，我们向应用程序发送一个带有管理员角色的模拟用户 GET 请求，并通过插入一个样本`Author`对象来准备数据库。我们还期望从应用程序获得有效的响应。正如你所看到的，我们没有模拟仓库类或服务类，因此当应用程序开始工作时，我们发起一个`GET`请求，所有相关的类和方法都真的像真实应用程序一样工作。

对于其余的测试用例，你可以在我们的 GitHub 仓库[`github.com/PacktPublishing/Mastering-Spring-Boot-3.0/blob/main/Chapter-6-unit-integration-test/src/test/java/integrationtests/AuthorControllerIntegrationTest.java`](https://github.com/PacktPublishing/Mastering-Spring-Boot-3.0/blob/main/Chapter-6-unit-integration-test/src/test/java/integrationtests/AuthorControllerIntegrationTest.java)中查看。

当我们运行我们的测试类时，它将测试所有端点的端到端。集成测试在单元测试和端到端测试之间架起了一座桥梁，专注于应用程序不同部分之间的交互。它验证了应用程序组件按预期协同工作，并识别出在单独测试组件时可能看不到的问题。通过使用`Testcontainers`和 WireMock 等工具，我们已经看到了如何模拟真实世界的环境和依赖关系，从而实现全面和可靠的测试。

总结来说，我们可以看到在软件开发周期中集成测试是多么重要。它为我们应用程序的整体功能提供了一个全面的测试。我总是想象这些集成测试就像本地开发测试一样。当你更改代码库时，你可以依赖集成测试来确保你的更改不会破坏其他流程。在下一节中，我们将处理异步环境：响应式组件。

# 测试响应式组件

在本节中，我们将深入研究测试响应式组件，重点关注我们示例响应式 Spring Boot 应用程序中的`UserController`端点。测试响应式组件与传统应用程序略有不同，因为响应式编程提供了一种非阻塞、事件驱动的处理数据流和变化传播的方法。我们将使用 Spring WebFlux 和`WebTestClient`来测试响应式 HTTP 请求。

## 设置测试环境

正如我们在*第三章*中学到的，Spring 中的响应式编程，由 Spring WebFlux 提供支持，引入了一种非阻塞、事件驱动的模型，该模型能够有效地处理异步数据流。这就是为什么我们需要一个稍微不同的策略来测试这些响应式组件，以确保异步和非阻塞行为被准确考虑。响应式测试环境必须能够处理随时间变化的数据流和序列，因此理解如何有效地设置和使用正确的工具至关重要。

我们将使用`WebTestClient`来测试反应式组件，而不是像在非反应式应用测试中使用的那样使用`MockMVC`。在配置了`@WebFluxTest(controllers = UserController.class)`的`UserControllerTest`类中，`WebTestClient`被自动注入，以便能够直接与`UserController`端点进行交互。这个注解帮助我们隔离控制器，确保测试轻量级且具有针对性，从而显著加快测试过程：

```java
@WebFluxTest(controllers = UserController.class,
        excludeAutoConfiguration = {ReactiveSecurityAutoConfiguration.class, ReactiveOAuth2ClientAutoConfiguration.class})
class UserControllerTest {
    @Autowired
    private WebTestClient webTestClient;
}
```

`@WebFluxTest`还为我们的测试环境设置了`WebTestClient`，使其准备好发送模拟的 HTTP 请求并断言响应。

`WebTestClient`有助于模拟请求和响应的行为，就像它们在一个实时、反应式网络环境中发生一样。它还再次展示了 Spring 如何无缝支持反应式端点的测试。在接下来的理论信息之后，我们将深入到下一节中模拟组件的部分。

## 准备模拟组件

模拟在测试期间防止实际数据库操作中起着关键作用，这有几个重要原因。首先，它确保了测试隔离，允许每个测试独立运行，而不会受到共享数据的影响。您已经在上一章中知道了这一点。因此，我们直接进入代码片段：

```java
@MockBean
private UserRepository userRepository;
private User testUser;
@MockBean
private SecurityWebFilterChain securityWebFilterChain;
@BeforeEach
void setUp() {
    testUser = new User(1L, "Test User", "test@example.com");
}
```

在每个测试之前，我们已经模拟了依赖项并初始化了测试数据，这使得我们能够深入到测试策略的核心，检查`UserController`中的每个端点是如何被测试的。接下来，我们将开始编写测试用例，我们将分解每个操作的测试过程，确保`UserController`在各种条件下都能按预期行为。

## 编写测试用例

现在，我们准备为每个方法编写测试。我们已经模拟了依赖项；我们只需要编写单元测试并查看它们是否返回预期的结果。与非反应式组件测试的唯一区别是，我们将使用`webTestClient`而不是`mockMVC`。让我们开始吧：

```java
@Test
void getAllUsersTest() {
    when(userRepository.findAll()).thenReturn(Flux.just(testUser));
    webTestClient.get().uri("/users")
            .exchange()
            .expectStatus().isOk()
            .expectBodyList(User.class).hasSize(1);
}
```

在这个代码块中，我们编写了一个单元测试来获取所有用户端点。首先，我们操作了`userRepositry.findAll()`方法，使其返回一个`Flux` `testUser`对象，并期望得到一个成功的响应。

对于其他端点的测试方法，请参阅 GitHub 仓库[`github.com/PacktPublishing/Mastering-Spring-Boot-3.0/blob/main/Chapter-6-reactive-test/src/test/java/com/packt/ahmeric/reactivesample/controller/UserControllerTest.java`](https://github.com/PacktPublishing/Mastering-Spring-Boot-3.0/blob/main/Chapter-6-reactive-test/src/test/java/com/packt/ahmeric/reactivesample/controller/UserControllerTest.java)。

在我们结束对 Spring Boot 应用程序中测试反应性组件的动手实践之前，回顾一下是有必要的。转向反应性编程迫使开发者改变他们构建应用程序和沟通的方式，本质上主要关注在压力下的非阻塞、异步和可扩展交互。然而，巨大的权力伴随着巨大的责任，代码的测试正是这一原则的顶峰。测试这些非常反应性组件带来的主要测试挑战是确保处理数据流、保证非阻塞性质和处理背压。

在本节中讨论了多种测试目标的方法，从通过`@WebFluxTest`设置测试环境到指定依赖关系，以及使用`WebTestClient`测试异步结果，一个人就可以配备上实现反应性 Spring Boot 应用程序质量、可扩展性和可维护性的所需工具。这些策略确保无论运行时实现什么条件，应用程序都能表现良好，并交付预期的功能和性能。

测试的第三个良好实践是“反应性”。当应用程序变得更加复杂和规模更大时，有效地测试这些反应性组件的能力成为成功开发生命周期的基石。从另一个角度来看，实践这些测试方法的开发者可以在问题甚至上升到令人烦恼的程度之前捕捉到它们，并找到一种方法来培养质量和弹性的文化。

换句话说，测试反应性部分是动态变化的 Web 开发景观的一个特色，它总是在最佳开发和最佳测试实践中遇到创新。通过这次探索的见解和技术，我们向未来构建更稳健、负责任和用户友好的应用程序迈进。

# 摘要

随着我们结束对非反应性和反应性 Spring Boot 应用程序的高级测试策略的全面探索，很明显，这次旅程既具有启发性又具有赋权性。我们学习了测试对于开发生命周期的重要性，以及它如何利用 Springboot 的能力变得简单。通过实际示例和动手指导，本章为您提供了在当今快速发展的软件开发领域中至关重要的基本技能和见解。以下是我们所涵盖内容的总结：

+   **TDD 的基础原则**：我们学习了 TDD 的基础原则及其对软件质量和可靠性的影响。

+   **单元测试控制器**：我们探索了使用安全层进行单元测试控制器的技术，确保我们的应用程序不仅功能齐全，而且安全。

+   **集成测试的重要性**：我们学习了集成测试的重要性，确保我们应用程序的不同部分能够无缝协作。

+   **测试响应式组件**：我们探讨了测试响应式组件的策略，解决了响应式编程范式带来的独特挑战。

这些技能将使你的应用程序经过测试，更加可靠、可扩展和易于维护。掌握 Spring Boot 中的这些测试技术，将使你作为一个开发者脱颖而出。

展望未来，软件开发之旅持续演变，带来新的挑战和机遇。在下一章中，我们将深入探讨容器化和编排的世界。这一即将到来的章节将揭示如何使 Spring Boot 应用程序准备好容器化，以及如何使用 Kubernetes 进行编排以增强可扩展性和可管理性。

# 第四部分：部署、可扩展性和生产力

在这部分，我们将把重点转向有效地部署和扩展应用程序，同时提高生产力。*第七章* 探讨了最新的 Spring Boot 3.0 功能，特别是那些增强容器化和编排以简化部署流程的功能。*第八章* 深入探讨了使用 Kafka 构建事件驱动系统，这对于考虑可扩展性管理高吞吐量数据至关重要。最后，*第九章* 介绍了提高生产力和简化开发策略，确保随着项目的增长，你可以保持快速高效的流程。这一部分对于掌握软件开发的操作方面至关重要，为你轻松处理大规模部署做好准备。

本部分包含以下章节：

+   *第七章*，*Spring Boot 3.0 的容器化和编排功能*

+   *第八章*，*使用 Kafka 探索事件驱动系统*

+   *第九章*，*提高生产力和开发简化*
