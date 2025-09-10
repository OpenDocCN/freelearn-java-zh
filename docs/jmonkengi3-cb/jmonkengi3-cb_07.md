# 第七章. 使用 SpiderMonkey 进行网络编程

本章将全部关于使用 jMonkeyEngine 的网络引擎 SpiderMonkey，将我们的游戏从我们自己的计算机的孤立状态带到互联网上。如果你对网络不太熟悉，不用担心，我们会从最基本的地方开始。

本章包含以下食谱：

+   设置服务器和客户端

+   处理基本消息

+   制作一个联网游戏 – 舰队大战

+   为 FPS 实现网络代码

+   加载关卡

+   在玩家位置之间进行插值

+   在网络上发射

+   优化带宽并避免作弊

# 简介

在网络上发送的数据组织在数据包中，协议以不同的方式处理它们。根据协议，数据包可能看起来不同，但它们包含数据本身以及控制信息，如地址和格式化信息。

SpiderMonkey 支持 TCP 和 UDP。在 SpiderMonkey 中，TCP 被称为可靠的。TCP 是可靠的，因为它验证发送的每个网络数据包，最小化由于数据包丢失和其他错误引起的问题。TCP 保证所有内容都能安全到达（如果可能的话）。为什么还要使用其他东西呢？因为速度。可靠性意味着 TCP 可能会慢。在某些情况下，我们并不依赖于每个数据包都到达目的地。UDP 更适合流媒体和低延迟应用，但应用程序必须准备好补偿不可靠性。这意味着当 FPS 中的数据包丢失时，游戏需要知道该怎么做。它会突然停止，还是会卡住？如果一个角色在移动，而游戏可以预测消息到达之间的移动，它将创建一个更平滑的体验。

学习如何使用 API 相对简单，但我们也会看到，网络不是你添加到游戏中的东西；游戏需要从规划阶段开始就适应它。

# 设置服务器和客户端

在这个食谱中，我们将查看绝对最小化，以便让服务器和客户端启动并运行，并能相互通信。

这只需几行代码就能完成。

服务器和客户端将共享一些共同的数据，我们将将其存储在一个`properties`文件中，以便于访问和外部修改。首先，客户端必须知道服务器的地址，而服务器和客户端都需要知道要监听和连接的端口号。这些很可能会在游戏中进行编辑。

## 如何做到这一点...

执行以下步骤来设置服务器和客户端：

1.  在服务器类的构造函数中，我们首先加载属性文件。一旦完成，我们可以使用以下代码初始化服务器：

    ```java
    server = Network.createServer(Integer.parseInt(prop.getProperty("server.port")));
    server.start();
    ```

    在静态块中，我们还需要确保服务器不会立即关闭。

1.  客户端以类似的方式设置，如下所示：

    ```java
    client = Network.connectToServer(prop.getProperty("server.address"), Integer.parseInt(prop.getProperty("server.port")));
    client.start();
    ```

1.  为了验证连接是否已经建立，我们可以在服务器中添加`ConnectionListener`，如下所示：

    ```java
    public void connectionAdded(Server server, HostedConnection conn) {
      System.out.println("Player connected: " + conn.getAddress());
    }
    ```

1.  如果我们再次连接到服务器，我们应该在服务器的输出窗口中看到打印的消息。

## 它是如何工作的...

`Network` 类是在设置和连接我们的组件时使用的主体类。这个特定的方法是以最简单的方式创建服务器，只需指定要监听的端口。让我们为 TCP 和 UDP 设置不同的端口，并供应服务器的名称和版本。

`connectToServer` 方法创建一个客户端并将其连接到指定的地址和端口。就像在服务器的情况下，`Network` 中还有其他方便的方法，允许我们指定更多参数，如果我们想的话。

实际上这就是所有需要的东西。当并行运行两个程序时，我们应该看到客户端连接到服务器。然而，并没有验证任何事情已经发生。这就是为什么我们在最后添加了 `ConnectionListener`。它是一个具有两个方法的接口：`connectionAdded` 和 `connectionRemoved`。每当客户端连接或断开连接时，这些方法都会被调用。这些方法给了服务器一种与我们通信的方式，表明已经发生了连接。这些方法将成为更高级食谱中事件链的来源。

一旦服务器启动，它就开始在指定的端口上监听传入的连接。如果网络地址被认为是街道名称，端口将是打开并通行的门。到目前为止，服务器和客户端之间只在门口进行了简单的握手。

# 处理基本消息

到目前为止，我们已经学习了设置服务器和连接客户端的基础知识。然而，它们并没有做很多事情，所以让我们看看它们相互通信需要什么。

## 准备工作

在 SpiderMonkey 中，通信是通过消息和消息接口处理的。当服务器发送消息时，它使用 `broadcast()` 方法，而客户端使用 `send()`。应该接收消息的一方必须有一个合适的 `MessageListener` 类。为了尝试所有这些，让我们让我们的服务器通过发送消息来问候连接的玩家，该消息在收到后将显示出来。

## 如何做到这一点...

执行以下步骤以连接和处理基本消息：

1.  我们首先定义我们的消息。它是一个简单的可序列化豆类，只有一个字段，如下面的代码片段所示：

    ```java
    @Serializable()
    public class ServerMessage extends AbstractMessage{
        private String message;

        public String getMessage() {
            return message;
        }

        public void setMessage(String message) {
            this.message = message;
        }
    }
    ```

1.  接下来，我们创建一个实现 `MessageListener` 的类。这是一个非常简单的类，当收到消息时会在控制台打印出来，如下所示：

    ```java
    public class ServerMessageHandler implements MessageListener<Client>{

        public void messageReceived(Client source, Message m) {
            ServerMessage message = (ServerMessage) m;
            System.out.println("Server message: " + message.getMessage());
        }
    }
    ```

1.  我们实例化 `ServerMessageHandler` 并将其添加到客户端，告诉它只监听 `ServerMessages`，如下所示：

    ```java
    ServerMessageHandler serverMessageHandler = new ServerMessageHandler();
    client.addMessageListener(serverMessageHandler, ServerMessage.class);
    ```

    也可以使用以下代码行让 `ServerMessageHandler` 处理所有传入的消息：

    ```java
    client.addMessageListener(serverMessageHandler);
    ```

1.  我们现在告诉服务器在有人连接时创建一个消息并发送给所有玩家：

    ```java
    ServerMessage connMessage = new ServerMessage();
    String message = "Player connected from: " + conn.getAddress();
    connMessage.setMessage(message);
    server.broadcast(connMessage);
    ```

1.  我们还需要做一件事。所有使用的消息类在使用之前都需要注册。我们在应用程序启动之前做这件事，如下所示：

    ```java
    public static void main(String[] args ) throws Exception {
      Serializer.registerClass(ServerMessage.class);
    ```

## 它是如何工作的...

花时间定义消息应包含的内容是掌握项目的好方法，因为很多架构都将围绕它们展开。在这个食谱中创建的消息被称为 `ServerMessage`，因为它用于从服务器向客户端发送大量信息。

我们创建的下一个类是 `MessageListener`。它在接收到消息时所做的唯一事情是将它打印到控制台。我们将其添加到客户端，并指出它应该专门监听 `ServerMessages`。

默认情况下，调用 `broadcast` 将会将消息发送给所有已连接的客户端。在这种情况下，我们只想向特定的客户端或一组客户端（如一个团队）发送消息。使用 `Filter` 也可以调用广播。它还可以向特定的频道发送消息，该频道可能分配给一个团队或一组玩家。

# 制作网络游戏 – 舰队大战

在之前的食谱中，我们探讨了如何设置服务器，连接和处理基本消息。在这个食谱中，我们将通过添加服务器验证并将其应用于实际游戏来加强和扩展这些知识。

基于回合制的棋盘游戏可能不是你通常使用 3D 游戏 SDK 开发的，但它是一个很好的学习网络的好游戏。舰队大战游戏是一个很好的例子，不仅因为规则简单且众所周知，而且还因为它有一个隐藏元素，这将帮助我们理解服务器验证的概念。

### 注意

如果你不太熟悉舰队大战游戏，请访问 [`www.wikipedia.org/wiki/Batt`](http://www.wikipedia.org/wiki/Batt)。

由于我们主要对游戏的网络方面感兴趣，我们将跳过一些通常需要的验证，例如查找重叠的船只。我们也不会编写任何图形界面，并使用命令提示符来获取输入。再次强调，为了专注于网络 API，一些游戏规则的简单 Java 逻辑将不会解释。

游戏将有一个客户端和服务器类。每个类都将有一个 `MessageListener` 实现，并共享消息和游戏对象。

## 准备工作

如果你还没有熟悉该章节中前面的食谱内容，强烈建议你熟悉一下。

与之前的食谱相比，消息的数量将大大增加。由于服务器和客户端都需要跟踪相同的消息，并且它们需要按相同的顺序注册，我们可以创建一个 `GameUtil` 类。它有一个名为 `initialize()` 的静态方法。对于每个新创建的消息类型，我们添加一行如下：

```java
Serializer.registerClass(WelcomeMessage.class);
```

游戏围绕着我们将在进入网络方面之前定义的一些对象。

我们需要一个`Ship`类。对于这个实现，它只需要`name`和`segments`字段。我们添加方法，以便一旦击中包含`Ship`的瓷砖，我们可以减少段数。当段数达到零时，它就沉没了。同样，`Player`也可以是一个简单的类，只需要一个 ID，用于与服务器进行识别，以及仍然存活的船只数量。如果船只数量达到零，玩家就输了。

许多消息类型扩展了一个名为`GameMessage`的类。这个类反过来扩展`AbstractMessage`，需要包含游戏的 ID，以及消息应该可靠的状态，因此使用 TCP 协议。

## 如何做到这一点...

我们首先设置一个`Game`类。这将由以下六个步骤组成：

1.  首先，`Game`类需要一个 ID。这被服务器用来跟踪哪些游戏消息与之相关（因为它同时支持多个游戏），也将被用作其他事物的参考。

1.  `Game`类需要两个`Player`对象，`player1`和`player2`，以及当前轮到哪个玩家的 ID。我们可以称之为`currentPlayerId`。

1.  `Game`类需要两个板；一个用于每个玩家。这些板将由 2D `Ship`数组组成。每个有船段所在的瓷砖都有一个指向`Ship`对象的引用；其他都是 null。

1.  一个整数`status`字段让我们知道游戏当前处于什么状态，这对于消息过滤很有用。我们还可以添加不同的状态常量，并设置一个默认状态，如下所示：

    ```java
    public final static int GAME_WAITING = 0;
    public final static int GAME_STARTED = 1;
    public final static int GAME_ENDED = 2;
    private int status = GAME_WAITING;
    ```

1.  现在，我们添加一个`placeShip`方法。在这个实现中，方法被简化了，只包含验证船是否在板内的验证，如下所示：

    ```java
    public void placeShip(int playerId, int shipId, int x, int y, boolean horizontal){
      Ship s = GameUtil.getShip(shipId);
      Ship[][] board;
      if(playerId == playerOne.getId()){
        board = boardOne;
        playerOne.increaseShips();
      } else {
        board = boardTwo;
        playerTwo.increaseShips();
      }
      for(int i = 0;i < s.getSegments(); i++){
        [verify segment is inside board bounds]
      }
    }
    ```

1.  在`Game`类中执行一些工作的另一个方法是`applyMove`。这个方法接受`FireActionMessage`作为输入，检查提供的瓷砖是否在该位置有船。然后检查假设的船是否沉没，以及玩家是否还有船只。如果击中船只，它将返回`Ship`对象，如下所示：

    ```java
    public Ship applyMove(FireActionMessage action){
      int x = action.getX();
      int y = action.getY();
      Ship ship = null;
      if(action.getPlayerId() == playerOne.getId()){
        ship = boardTwo[x][y];
        if(ship != null){
          ship.hit();
          if(ship.isSunk()){
            playerTwo.decreaseShips();
          }
        }
      } else {
          [replicate for playerTwo]
    }
      if(playerTwo.getShips() < 1 || playerOne.getShips() < 1){
        status = GAME_ENDED;
      }
      if(action.getPlayerId() == playerTwo.getId()){
        turn++;
      }
      return ship;
    }
    ```

现在，让我们来看看服务器端的事情。在前几章中，我们探讨了如何连接客户端，但一个完整的游戏需要更多的通信来设置一切，正如我们将看到的。本节将包含以下八个步骤：

1.  由于服务器旨在同时处理多个游戏实例，我们将定义几个`HashMaps`来跟踪游戏对象。对于每个我们创建的游戏，我们将`Game`对象放入`games`映射中，ID 作为键：

    ```java
    private HashMap<Integer, Game> games = new HashMap<Integer, Game>();
    ```

1.  我们还将使用`Filters`仅向相关游戏中的玩家发送消息。为此，我们存储一个`HostedConnections`列表，其中每个都是一个指向客户端的地址，游戏 ID 作为键：

    ```java
    private HashMap<Integer, List<HostedConnection>> connectionFilters = new HashMap<Integer, List<HostedConnection>>();
    ```

1.  由于我们不断地分配新的玩家 ID 并增加游戏 ID 的值，因此我们也将为此设置两个字段：`nextGameId`和`nextPlayerId`。

1.  一切从连接的客户端开始。就像在*设置服务器和客户端*食谱中一样，我们使用`ConnectionListener`来处理这个问题。该方法要么将玩家添加到现有的游戏中，要么如果没有可用，则创建一个新的游戏。无论是否创建了新游戏，之后都会调用`addPlayer`方法，如下面的代码片段所示：

    ```java
    public void connectionAdded(Server server, HostedConnection conn) {
      Game game = null;
      if(games.isEmpty() || games.get(nextGameId - 1).getPlayerTwo() != null){
        game = createGame();
      } else {
        game = games.get(nextGameId - 1);
      }
      addPlayer(game, conn);
    }
    ```

1.  `createGame`方法创建一个新的`game`对象并设置正确的 ID。将其放入`games`映射后，它创建一个新的`List<HostedConnection>`，称为`connsForGame`，并将其添加到`connectionFilters`映射中。`connsForGame`列表目前为空，但将在玩家连接时被填充：

    ```java
    private Game createGame(){
      Game game = new Game();
      game.setId(nextGameId++);
      games.put(game.getId(), game);
      List<HostedConnection> connsForGame = new ArrayList<HostedConnection>();
      connectionFilters.put(game.getId(), connsForGame);
      return game;
    }
    ```

1.  `addPlayer`方法首先创建一个新的`Player`对象，然后设置其 ID。我们使用`WelcomeMessage`将 ID 发送回玩家：

    ```java
    private void addPlayer(Game game, HostedConnection conn){
      Player player = new Player();
      player.setId(nextPlayerId++);
    ```

1.  服务器使用客户端的连接作为过滤器广播这条消息，确保它是唯一的接收者，如下所示：

    ```java
      WelcomeMessage welcomeMessage = new WelcomeMessage();
      welcomeMessage.setMyPlayerId(player.getId());
      server.broadcast(Filters.in(conn), welcomeMessage);
    ```

1.  接着，它决定玩家是第一个还是第二个连接到游戏的，并将玩家的`HostedConnection`实例添加到与该游戏关联的连接列表中，如下面的代码片段所示：

    ```java
      if(game.getPlayerOne() == null){
        game.setPlayerOne(player);
      } else {
        game.setPlayerTwo(player);
      }
    List<HostedConnection> connsForGame = connectionFilters.get(game.getId());
    connsForGame.add(conn);
    ```

1.  然后创建一个`GameStatusMessage`对象，让所有游戏中的玩家知道当前的状态（即`WAITING`）以及可能拥有的任何玩家信息，如下面的代码片段所示：

    ```java
      GameStatusMessage waitMessage = new GameStatusMessage();
      waitMessage.setGameId(game.getId());
      waitMessage.setGameStatus(Game.GAME_WAITING);
      waitMessage.setPlayerOneId(game.getPlayerOne() != null ? game.getPlayerOne().getId() : 0);
      waitMessage.setPlayerTwoId(game.getPlayerTwo() != null ? game.getPlayerTwo().getId() : 0);
      server.broadcast(Filters.in(connsForGame), waitMessage);
    }
    ```

我们将探讨客户端的消息处理，并查看其`MessageListener`接口如何处理传入的`WelcomeMessages`和游戏更新：

1.  我们创建了一个名为`ClientMessageHandler`的类，该类实现了`MessageListener`接口。首先，我们将遍历处理游戏开始的部分。

1.  `thisPlayer`对象已经在客户端实例化了，所以当接收到`WelcomeMessage`时，我们只需要设置玩家的 ID。此外，我们可以向玩家显示一些信息，让他们知道连接已经建立：

    ```java
    public void messageReceived(Client source, Message m) {
      if(m instanceof WelcomeMessage){
        WelcomeMessage welcomeMess = ((WelcomeMessage)m);
        Player p = gameClient.getThisPlayer();
        p.setId(welcomeMessage.getMyPlayerId());
    }
    ```

1.  当接收到`GameStatusMessage`时，我们需要完成三件事情。首先，设置游戏的 ID。在这个实现中，客户端知道游戏的 ID 不是必需的，但与服务器通信时可能很有用：

    ```java
    else if(m instanceof GameStatusMessage){
      int status = ((GameStatusMessage)m).getGameStatus();
      switch(status){
      case Game.GAME_WAITING:
        if(game.getId() == 0 && ((GameStatusMessage)m).getGameId() > 0){
             game.setId(((GameStatusMessage)m).getGameId());
          }
    ```

1.  然后，我们通过简单地检查它们是否之前已经设置来设置`playerOne`和`playerTwo`字段。我们还需要通过比较消息中玩家的 ID 与与此客户端关联的 ID 来识别玩家。一旦找到，我们让他或她开始放置船只，如下所示：

    ```java
    if(game.getPlayerOne() == null && ((GameStatusMessage)m).getPlayerOneId() > 0){
      int playerOneId = ((GameStatusMessage)m).getPlayerOneId();
      if(gameClient.getThisPlayer().getId() == playerOneId){
        game.setPlayerOne(gameClient.getThisPlayer());
        gameClient.placeShips();
           } else {
             Player otherPlayer = new Player();
                 otherPlayer.setId(playerOneId);
        game.setPlayerOne(otherPlayer);
      }

    }
    game.setStatus(status);
    ```

1.  当接收到`TurnMessage`时，我们应该从其中提取`activePlayer`并将其设置在游戏上。如果`activePlayer`与`gameClient`的`thisPlayer`相同，则在`gameClient`上设置`myTurn`为`true`。

1.  类最后要处理的消息是`FiringResult`消息。这将在`game`对象上调用`applyMove`。应该将某种输出与这条消息关联起来，告诉玩家发生了什么。这个示例游戏使用`System.out.println`来传达这一点。

1.  最后，在客户端类的构造函数中初始化我们的`ClientMessageHandler`对象，如下所示：

    ```java
    ClientMessageHandler messageHandler = new ClientMessageHandler(this, game);
    client.addMessageListener(messageHandler);
    ```

处理完接收到的消息后，我们可以查看客户端的逻辑以及它发送的消息。这非常有限，因为大多数游戏功能都是由服务器处理的。

以下步骤展示了如何实现客户端游戏逻辑：

1.  `placeShip`方法可以以多种不同的方式编写。通常，你会有一个图形界面。然而，对于这个食谱，我们使用命令提示符，它将输入分解为 *x* 和 *y* 坐标以及船只是否水平或垂直放置。最后，它应该向服务器发送五个`PlaceShipMessages`实例。对于每增加的船只，我们还调用`thisPlayer.increaseShips()`。

1.  我们还需要一个名为`setMyTurn`的方法。这个方法使用命令提示符接收要射击的 *x* 和 *y* 坐标。之后，它填充`FireActionMessage`，并将其发送到服务器。

1.  对于`PlaceShipMessage`，创建一个新的类，并让它扩展`GameMessage`。

1.  这个类需要包含放置船只的玩家的 ID、坐标和船只的方向。船只的 ID 指的是以下数组中的位置：

    ```java
    private static Ship[] ships = new Ship[]{new Ship("PatrolBoat", 2), new Ship("Destroyer", 3), new Ship("Submarine", 3), new Ship("Battleship", 4), new Ship("Carrier", 5)};
    ```

1.  我们创建另一个名为`FireActionMessage`的类，它也扩展`GameMessage`。

1.  这有一个指向开火的玩家的引用，以及 *x* 和 *y* 坐标。

服务器上的消息处理与客户端类似。我们有一个实现`MessageListener`接口的`ServerMessageHandler`类。这个类必须处理接收来自放置船只和开火的玩家的消息。

1.  在`messageReceived`方法内部，捕获所有`PlaceShipMessages`。使用提供的`gameId`，我们从服务器的`getGame`方法获取游戏实例并调用`placeShip`方法。一旦完成，我们检查是否两个玩家都已经放置了所有他们的船只。如果是这样，是时候开始游戏了：

    ```java
    public void messageReceived(HostedConnection conn, Message m) {
      if (m instanceof PlaceShipMessage){
        PlaceShipMessage shipMessage = (PlaceShipMessage) m;
        int gameId = shipMessage.getGameId();
        Game game = gameServer.getGame(gameId);
        game.placeShip( … );
        if(game.getPlayerOne().getShips() == 5 && game.getPlayerTwo() != null&& game.getPlayerTwo().getShips() == 5){
          gameServer.startGame(gameId);
        }
    ```

1.  在`startGame`方法中，我们首先需要发送一条消息，让玩家知道游戏现在已经开始。我们知道要向哪些客户端发送消息，如下从`connectionFilters`映射中获取连接列表：

    ```java
    public Game startGame(int gameId){
      Game game = games.get(gameId);
      List<HostedConnection> connsForGame = connectionFilters.get(gameId);
      GameStatusMessage startMessage = new GameStatusMessage();
      startMessage.setGameId(game.getId());
      startMessage.setGameStatus(Game.GAME_STARTED);
      server.broadcast(Filters.in(connsForGame), startMessage);
    ```

1.  之后，我们决定哪个玩家将先走一步，并将`TurnMessage`发送给玩家，如下所示：

    ```java
      int startingPlayer = FastMath.nextRandomInt(1, 2);
      TurnMessage turnMessage = new TurnMessage();

      server.broadcast(Filters.in(connsForGame), turnMessage);
      return game;
    }
    ```

1.  现在，我们需要定义`TurnMessage`。这是一个简单的消息，只包含当前轮到哪个玩家的 ID，并扩展`GameMessage`。

1.  回到`ServerMessageListener`，我们使其准备好接收来自玩家的`FireActionMessage`。我们首先验证传入消息的`playerId`是否与服务器端的当前玩家匹配。它可以如下实现：

    ```java
    if(m instanceof FireActionMessage){
      FireActionMessage fireAction = (FireActionMessage) m;
      int gameId = fireAction.getGameId();
      Game game = gameServer.getGame(gameId);
      if(game.getCurrentPlayerId() == fireAction.getPlayerId()){
    ```

1.  然后，我们在游戏上调用`applyMove`，让它决定是否命中。如果是命中，船只将被返回。这可以通过输入以下代码实现：

    ```java
        Ship hitShip = game.applyMove(fireAction);
    ```

1.  我们继续创建一个`FiringResult`消息。这是`FireActionMessage`的扩展，包含关于（可能的）被击中的船只的额外字段。它应该广播给两位玩家，让他们知道该动作是否击中。

1.  最后，我们切换活动玩家，并向两位玩家发送另一个`TurnMessage`，如下所示：

    ```java
                    TurnMessage turnMessage = new TurnMessage();
                    turnMessage.setGameId(game.getId());
                    game.setCurrentPlayerId(game.getCurrentPlayerId() == 1 ? 2 : 1);
                    turnMessage.setActivePlayer(game.getCurrentPlayerId());
                    gameServer.sendMessage(turnMessage);
                }
    ```

1.  这种流程将持续进行，直到其中一位玩家用完船只。然后，我们应该简单地发送包含`END`状态的`GameStatusMessage`给玩家，并将他们断开连接。

## 它是如何工作的...

当玩家启动客户端时，它将自动连接到在属性文件中定义的服务器。

服务器将确认这一点，为玩家分配一个用户 ID，并发送包含 ID 的`WelcomeMessage`。`WelcomeMessage`的职责是确认与客户端的连接，并让客户端知道其分配的 ID。在此实现中，它用于客户端未来的通信。另一种过滤传入消息的方法是使用`HostedConnection`实例，因为它持有客户端的唯一地址。

当第一位玩家连接时，将创建一个新的游戏。游戏处于`WAITING`状态，直到两位玩家都连接，并且都放置了他们的船只。对于每个连接的玩家，它创建一个`GameStatusMessage`，让所有游戏中的玩家知道当前状态（即`WAITING`）以及任何可能拥有的玩家信息。第一位玩家`PlayerOne`将收到两次消息（再次当`PlayerTwo`连接时），但这没关系，因为游戏将在两位玩家都放置了船只之前保持`WAITING`状态。

`placeShip`方法被简化了，它不包含在完整游戏中通常所拥有的所有验证。确保服务器检查船只是否在棋盘外，或者重叠，并确保它是正确的类型、长度等，如果错误则发送回消息。此方法仅检查船只是否在边界内，如果不在此范围内则跳过。验证也可以在客户端进行，但为了限制滥用，它也必须在服务器上进行。

起始玩家将被随机选择，并在`TurnMessage`中发送给两位玩家，说明谁开始。玩家被要求输入一组坐标进行射击，并发送`FireActionMessage`到服务器。

服务器验证玩家并将其应用于棋盘。然后，它向所有玩家广播一个包含关于动作信息以及是否有船只被击中的`FireResult`消息。如果被攻击的玩家仍有船只剩余，那么轮到该玩家进行射击。

一旦一位玩家用完船只，游戏结束。服务器向所有客户端广播消息，并将他们断开连接。

客户端对其他玩家的信息非常有限。这种做法的好处是它使得作弊变得更加困难。

# 实现 FPS 的网络代码

网络 FPS 游戏是一种永远不会失去人气的游戏类型。在本菜谱中，我们将查看基本设置，以启动服务器和多个客户端。我们将模拟一个具有持久环境的服务器，玩家可以随时连接和断开连接。

我们可以利用之前章节中生成的一些代码。我们将使用的代码需要对网络游戏进行一些修改以适应，但它将再次展示使用 jMonkeyEngine 的 `Control` 和 `AppState` 类的好处。

## 准备工作

在此之前，阅读本章中之前的菜谱（特别是 *制作网络游戏 – 舰队战斗*，该架构高度依赖于它）以及第二章中的 *创建可重用角色控制* 菜谱（第二章，*相机和游戏控制*），因为我们将在这里为我们的 `NetworkedPlayerControl` 实现使用类似的模式。为了避免重复，本菜谱将不会展示或解释所有常规游戏代码。

## 如何实现...

我们首先定义一些将在服务器和客户端之间共同使用的类：

1.  首先，我们定义一个名为 `NetworkedPlayerControl` 的类，它扩展了 `AbstractControl`。我们将使用这个类作为玩家对象的标识符，以及作为玩家空间表示的控制。

1.  这个类将在后续的菜谱中扩展，但到目前为止，它应该跟踪一个名为 `ID` 的整数。

1.  它还需要一个名为 `onMessageReceived` 的抽象方法，该方法接受 `PlayerMessage` 作为输入。这是我们消息处理器将调用来应用更改的方法。在 `ServerPlayerControl` 中，消息将包含玩家的实际输入，而 `ClientPlayerControl` 简单地复制服务器上发生的事情。

1.  现在，我们定义一个名为 `Game` 的类，它将由客户端和服务器共享。

1.  我们添加一个名为 `players` 的 `HashMap` 对象，其中 `playerId` 是键，`NetworkedPlayerControl` 是值。它跟踪玩家。

我们需要为这个例子添加一些新的消息。所有消息都假设是按照 bean 模式编写的，具有 getter 和 setter 方法。我们按照以下步骤定义消息：

1.  我们创建一个用于玩家相关信息的基消息，并将其命名为 `PlayerMessage`，它扩展了 `AbstractMessage`。这个消息只需要一个名为 `playerId` 的整数。

1.  我们创建第一个扩展 `PlayerMessage` 的消息，它被称为 `PlayerActionMessage`，用于处理玩家输入。这个消息应该设置为可靠的，因为我们不希望错过任何玩家的输入。

1.  由于玩家输入可以是按键或鼠标点击，因此它需要有一个名为 `pressed` 的布尔值和一个名为 `floatValue` 的浮点值。

1.  此外，我们还需要添加一个名为 `action` 的字符串值。

1.  我们在另一个名为`PlayerUpdateMessage`的类中扩展了`PlayerMessage`。这将用于从服务器向客户端分发玩家位置信息。这不应该太可靠，以避免不必要的延迟。

1.  它有一个名为`position`的`Vector3f`字段和一个名为`lookDirection`的`Quaternion`字段。

定义了消息后，让我们看看服务器代码的样子：

1.  我们定义了一个名为`FPSServer`的新类，它扩展了`SimpleApplication`。

1.  它需要跟踪以下字段。除了`Server`字段外，它还跟踪要分配给连接玩家的下一个 ID、一个游戏和一个所有当前连接玩家的映射，其中连接作为键：

    ```java
    private Server server;
    private int nextPlayerId = 1;
    private Game game;
    private HashMap<HostedConnection, ServerPlayerControl> playerMap = new HashMap<HostedConnection, ServerPlayerControl>();
    ```

1.  就像在先前的配方中一样，我们使用一个名为`GameUtil`的类来注册我们所有的消息类。我们还设置`frameRate`为`30 fps`。这可能会根据游戏类型而有所不同。最后，我们以无头模式启动应用程序，以节省资源，如下所示：

    ```java
    public static void main(String[] args ) throws Exception{
      GameUtil.initialize();
      FPSServer gameServer = new FPSServer();
      AppSettings settings = new AppSettings(true);
      settings.setFrameRate(30);
      gameServer.setSettings(settings);
      gameServer.start(JmeContext.Type.Headless);
    }
    ```

1.  我们初始化服务器，就像在*制作网络游戏 - 舰队大战*配方中一样，并创建一个`ConnectionListener`实例来寻找连接和断开连接的玩家。当玩家连接或断开时，它将分别调用`addPlayer`和`removePlayer`。

1.  在`addPlayer`方法中，我们创建一个新的`ServerPlayerControl`实例，这是`NetworkedPlayerControl`的服务器端实现，并为其分配一个 ID 以便更容易引用，如下所示：

    ```java
    private void addPlayer(Game game, HostedConnection conn){
      ServerPlayerControl player = new ServerPlayerControl();
      player.setId(nextPlayerId++);
      playerMap.put(conn, player);
      game.addPlayer(player);
    ```

1.  然后，我们为它创建一个空间，以便它在场景图中有一个引用（因此，它将自动更新）。这不仅是为了视觉表示，我们还依赖于它来更新我们的方法，如下所示：

    ```java
      Node s = new Node("");
      s.addControl(player);
      rootNode.attachChild(s);
    ```

1.  对于与服务器未来的任何通信，客户端将在所有消息中提供其`playerId`，因此服务器在`WelcomeMessage`中将分配的 ID 发送回客户端。它使用客户端的连接作为过滤器广播消息，如下所示：

    ```java
      WelcomeMessage welcomeMessage = new WelcomeMessage();
      welcomeMessage.setMyPlayerId(player.getId());
      server.broadcast(Filters.in(conn), welcomeMessage);
    ```

1.  然后，我们向加入的玩家发送关于所有其他玩家的信息，如下所示：

    ```java
      Collection<NetworkedPlayerControl> players = game.getPlayers().values();
      for(NetworkedPlayerControl p: players){
        PlayerJoinMessage joinMessage = new PlayerJoinMessage();
        joinMessage.setPlayerId(p.getId());
        server.broadcast(Filters.in(conn), joinMessage);
      }
    ```

1.  最后，服务器向所有其他玩家发送关于新玩家的消息，如下所示：

    ```java
      PlayerJoinMessage joinMessage = new PlayerJoinMessage();
      joinMessage.setPlayerId(player.getId());
      server.broadcast(joinMessage);
    }
    ```

1.  `removePlayer`方法的工作方式类似，但它只需向当前连接的每个玩家发送关于断开连接玩家的消息。它也使用`PlayerJoinMessage`，但将`leaving`布尔值设置为`true`，以指示玩家正在离开，而不是加入游戏。

1.  然后，服务器将连续向所有玩家发送位置和旋转（方向）更新。由于我们将`fps`设置为`30`，它将尝试每 33 毫秒这样做，如下所示：

    ```java
    public void simpleUpdate(float tpf) {
      super.simpleUpdate(tpf);
      Collection<NetworkedPlayerControl> players = game.getPlayers().values();
      for(NetworkedPlayerControl p: players){
        p.update(tpf);
        PlayerUpdateMessage updateMessage = new PlayerUpdateMessage();
        updateMessage.setPlayerId(p.getId());
    updateMessage.setLookDirection(p.getSpatial().getLocalRotation());
    updateMessage.setPosition(p.getSpatial().getLocalTranslation());
        updateMessage.setYaw(p.getYaw());
        server.broadcast(updateMessage);
      }
    }
    ```

1.  我们还创建了一个`ServerMessageHandler`类，它实现了`MessageListener`。在这个例子中，这是一个简短的类，它将只监听扩展`PlayerMessage`的消息，并将其传递给正确的`NetworkedPlayerControl`类以更新它。在这个配方中，这意味着来自玩家的输入，如下所示：

    ```java
    public void messageReceived(HostedConnection source, Message m) {
      if(m instanceof PlayerMessage){
        PlayerMessage message = (PlayerMessage)m;
        NetworkedPlayerControl p = game.getPlayer(message.getPlayerId());
        p.onMessageReceived(message);
      }
    }
    ```

1.  对于`NetworkedPlayerControl`类的服务器端实现，我们将其扩展到一个新的类，称为`ServerPlayerControl`。

1.  与第二章中的`GameCharacterControl`类类似，我们将使用一组布尔值来跟踪输入，如下所示：

    ```java
    boolean forward = false, backward = false, leftRotate = false, rightRotate = false, leftStrafe = false, rightStrafe = false;
    ```

1.  在实现的`onMessageReceived`方法中，监听`PlayerMessages`。我们不知道它将包含布尔值还是浮点值，所以我们寻找两者，如下所示：

    ```java
    public void onMessageReceived(PlayerMessage message) {
      if(message instanceof PlayerActionMessage){
        String action = ((PlayerActionMessage) message).getAction();
        boolean value = ((PlayerActionMessage) message).isPressed();
        float floatValue = ((PlayerActionMessage) message).getFloatValue();
    ```

1.  然后，我们应用以下代码片段中的值：

    ```java
    if (action.equals("StrafeLeft")) {
      leftStrafe = value;
    } else if (action.equals("StrafeRight")) {
      rightStrafe = value;
    }
    ...
    else if (action.equals("RotateLeft")) {
      rotate(floatValue);
    } else if (action.equals("RotateRight")) {
      rotate(-floatValue);
     }
    ```

1.  在重写的`controlUpdate`方法中，我们根据输入修改空间的位置和旋转，就像我们在第二章中*创建可重用的角色控制*食谱中所做的那样。

客户端在很多方面都很简单，因为它基本上只做两件事。它接收玩家的输入，将其发送到服务器，接收来自服务器的更新，并按以下方式应用它们：

1.  我们首先创建一个名为`FPSClient`的新类，它扩展了`SimpleApplication`。

1.  在构造函数中，我们读取网络属性文件并连接到服务器，如下所示：

    ```java
    Properties prop = new Properties();   prop.load(getClass().getClassLoader().getResourceAsStream("network/resources/network.properties"));
            client = Network.connectToServer(prop.getProperty("server.name"), Integer.parseInt(prop.getProperty("server.version")), prop.getProperty("server.address"), Integer.parseInt(prop.getProperty("server.port")));
    ```

1.  就像服务器一样，在启动应用程序之前，我们注册所有消息类。

1.  应用程序应该有一个指向名为`playerModel`的`Node`类的引用，它将是游戏中玩家的视觉表示。还应该有一个名为`thisPlayer`的`ClientPlayerControl`类。

1.  在`simpleInitApp`方法中，我们附加了`InputAppState`。这与第二章中*创建一个输入 AppState 对象*食谱中的功能相同，即*相机和游戏控制*。唯一的区别是它将受益于有一个直接的方式到达客户端发送消息：

    ```java
    public void simpleInitApp() {
      InputAppState inputAppState = new InputAppState();
      inputAppState.setClient(this);
      stateManager.attach(inputAppState);
    ```

1.  接下来，我们创建`playerGeometry`，用于本例中的所有玩家，如下所示：

    ```java
      Material playerMaterial  = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
      playerGeometry = new Geometry("Player", new Box(1f,1f,1f));
      playerGeometry.setMaterial(playerMaterial);
    ```

1.  我们还关闭了应用程序的`flyByCamera`实例并创建了一个新的`game`对象，当从服务器接收到信息时，我们将填充它，如下所示：

    ```java
      getFlyByCamera().setEnabled(false);
      game = new Game();
    ```

1.  最后，我们创建一个新的`ClientMessageListener`对象并将其添加到客户端，如下所示：

    ```java
    ClientMessageHandler messageHandler = new ClientMessageHandler(this, game);
    client.addMessageListener(messageHandler);
    ```

1.  在`createPlayer`方法中，我们创建一个新的`ClientPlayerControl`实例，还创建了一个`Node`实例，并将其附加到场景图中，如下所示：

    ```java
    ClientPlayerControl player = new ClientPlayerControl();
    player.setId(id);
    final Node playerNode = new Node("Player Node");
            playerNode.attachChild(assetManager.loadModel("Models/Jaime/Jaime.j3o"));//
    playerNode.addControl(player);
    ```

1.  由于我们不知道这个方法将在何时被调用，我们确保以线程安全的方式附加空间。这可以如下实现：

    ```java
    enqueue(new Callable(){
      public Object call() throws Exception {
        rootNode.attachChild(playerNode);
        return null;
      }
    });
    ```

1.  最后，我们将创建的`ClientPlayerControl`实例返回给调用方法。

1.  我们添加了一个名为`setThisPlayer`的新方法。当接收到玩家的`WelcomeMessage`时，将调用此方法。在这个方法内部，我们创建`CameraNode`，它将被附加到玩家，如下所示：

    ```java
    public void setThisPlayer(ClientPlayerControl player){
      this.thisPlayer = player;
      CameraNode camNode = new CameraNode("CamNode", cam);
      camNode.setControlDir(CameraControl.ControlDirection.SpatialToCamera);
      ((Node)player.getSpatial()).attachChild(camNode);
    }
    ```

1.  我们还必须重写 `destroy` 方法，以确保在客户端关闭时关闭与服务器的连接。这可以按以下方式实现：

    ```java
    public void destroy() {
      super.destroy();
      client.close();
    }
    ```

1.  现在，我们需要创建 `NetworkedPlayerControl` 的客户端表示，并在一个名为 `ClientPlayerControl` 的类中扩展它。

1.  它有一个名为 `tempLocation` 的 `Vector3f` 字段和一个名为 `tempRotation` 的 `Quaternion` 字段。这些用于存储从服务器接收到的更新。它还可以有一个名为 `yaw` 的 `float` 字段，用于头部移动。

1.  在 `onMessageReceived` 方法中，我们只查找 `PlayerUpdateMessages` 并使用消息中接收到的值设置 `tempLocation` 和 `tempRotation`，如下所示：

    ```java
    public void onMessageReceived(PlayerMessage message) {
      if(message instanceof PlayerUpdateMessage){
        PlayerUpdateMessage updateMessage = (PlayerUpdateMessage) message;
      tempRotation.set(updateMessage.getLookDirection());
      tempLocation.set(updateMessage.getPosition());
    tempYaw = updateMessage.getYaw();
      }
    }
    ```

1.  然后，我们将 `temp` 变量的值应用于 `controlUpdate` 方法：

    ```java
    spatial.setLocalTranslation(tempLocation);
    spatial.setLocalRotation(tempRotation);
    yaw = tempYaw;
    ```

就像在服务器端一样，我们需要一个消息处理程序来监听传入的消息。为此，执行以下步骤：

1.  我们创建了一个名为 `ClientMessageHandler` 的新类，该类实现了 `MessageListener<Client>`。

1.  `ClientMessageHandler` 类应该在名为 `gameClient` 的字段中有一个对 `FPSClient` 的引用，在另一个名为 `game` 的字段中有一个对 `Game` 的引用。

1.  在 `messageReceived` 方法中，我们需要处理许多消息。`WelcomeMessage` 最有可能首先到达。当这种情况发生时，我们创建一个玩家对象和空间，并将其分配给这个客户端的玩家，如下所示：

    ```java
    public void messageReceived(Client source, Message m) {
      if(m instanceof WelcomeMessage){
        ClientPlayerControl p = gameClient.createPlayer(((WelcomeMessage)m).getMyPlayerId());
        gameClient.setThisPlayer(p);
        game.addPlayer(gameClient.getThisPlayer());
    ```

1.  当玩家加入或离开游戏时都会接收到 `PlayerJoinMessage`。使其与众不同的因素是 `leaving` 布尔值。根据玩家是加入还是离开，我们调用 `game` 和 `gameClient` 方法，如下面的代码片段所示：

    ```java
    PlayerJoinMessage joinMessage = (PlayerJoinMessage) m;
    int playerId = joinMessage.getPlayerId();
    if(joinMessage.isLeaving()){
       gameClient.removePlayer((ClientPlayerControl)   game.getPlayer(playerId));
      game.removePlayer(playerId);
    } else if(game.getPlayer(playerId) == null){
      ClientPlayerControl p = gameClient.createPlayer(joinMessage.getPlayerId());
      game.addPlayer(p);
    }
    ```

1.  当接收到 `PlayerUpdateMessage` 时，我们首先找到相应的 `ClientPlayerControl` 类，并将消息传递给它，如下所示：

    ```java
      } else if (m instanceof PlayerUpdateMessage){
        PlayerUpdateMessage updateMessage = (PlayerUpdateMessage) m;
        int playerId = updateMessage.getPlayerId();
        ClientPlayerControl p = (ClientPlayerControl) game.getPlayer(playerId);
        if(p != null){
          p.onMessageReceived(updateMessage);
        }
    ```

## 它是如何工作的...

服务器以无头模式运行，这意味着它不会进行任何渲染，也不会有图形输出，但我们仍然可以访问完整的 jMonkeyEngine 应用程序。在这个菜谱中，一个服务器实例一次只能有一个游戏活动。

我们在名为 `GameUtil` 的类中实例化所有网络消息，因为它们在客户端和服务器上必须相同（并且以相同的顺序序列化）。

客户端将在启动时立即尝试连接到服务器。一旦连接，它将通过 `WelcomeMessage` 从服务器接收 `playerId`，以及所有已连接的其他玩家的 `PlayerJoinMessages`。同样，所有其他玩家都将收到包含新玩家 ID 的 `PlayerJoinMessage`。

客户端使用 `PlayerActionMessage` 将玩家执行的所有操作发送到服务器，并将其应用于其游戏实例。运行在 30 fps 的服务器将使用 `PlayerUpdateMessages` 将每个玩家的位置和方向发送给所有其他玩家。

客户端的`InputAppState`类与第二章中的类似，*相机和游戏控制*。唯一的区别是，它不是直接更新一个`Control`实例，而是创建一个消息并发送到服务器。在`onAction`类中，我们设置消息的布尔值，而在`onAnalog`（用于查看和旋转）中，将使用`floatValue`，如下面的代码片段所示：

```java
public void onAction(String name, boolean isPressed, float tpf) {
  InputMapping input = InputMapping.valueOf(name);
  PlayerActionMessage action = new PlayerActionMessage();
  action.setAction(name);
  action.setPressed(isPressed);
  action.setPlayerId(client.getThisPlayer().getId());
  client.send(action);
}
```

在玩家离开游戏的情况下，`PlayerJoinMessages`将被发送给其他玩家，并将`leaving`设置为`true`。

`NetworkedPlayerControl`类是一个抽象类，它本身并不做很多事情。您可能已经认识到了`GameCharacterControl`中`ServerPlayerControl`的实现，并且它们的功能相似，但`ServerPlayerControl`不是直接从用户那里接收输入，而是通过网络消息接收。

客户端和服务器对`NetworkedPlayerControl`的实现都使用`tempRotation`和`tempLocation`字段，并将任何传入的更改应用于这些字段。这样做是为了我们不在主循环之外修改实际的空间变换。

我们不应该被这个配方的相对简单性所迷惑。它仅仅展示了实时网络化环境的基础。制作一个完整的游戏会带来更多的复杂性。

## 参考信息

+   如果您想看到一个完整实时游戏的示例，请查看 MonkeyZone 的完整源代码，网址为[`hub.jmonkeyengine.org/wiki/doku.php/jme3:advanced:monkey_zone`](http://hub.jmonkeyengine.org/wiki/doku.php/jme3:advanced:monkey_zone)。它不仅有人类玩家，还有网络化 AI。

# 加载一个级别

无论我们制作的是 FPS、RTS 还是驾驶游戏，我们都希望能够为玩家加载不同类型的游戏环境，让他们在其中漫游。我们如何轻松地做到这一点？

在这个配方中，我们将向本章先前概述的网络化 FPS 游戏添加功能。这个原则适用于任何类型的已经网络化的游戏，尽管它可能取决于游戏如何实现级别。在这里，我们假设它使用 jMonkeyEngine 场景或`.j3o`场景。

## 如何实现...

执行以下步骤以加载一个级别：

1.  我们首先定义一个新的消息类：`LoadLevelMessage`。它扩展了`GameMessage`，因为可能需要知道`gameId`。除此之外，它还有一个字段`levelName`。

1.  我们将在我们的`Game`类中添加相同的字段，以便它可以跟踪它正在运行哪个级别。

1.  接下来，让我们在我们的服务器上创建一个`levelNode`字段，我们可以将级别加载到其中，如下所示：

    ```java
    private Node loadLevel(String levelName){
      return (Node) assetManager.loadModel("Scenes/"+levelName + ".j3o");
    }
    ```

1.  然后，我们创建一个小的方法，从预定义的路径加载级别，如下所示：

    ```java
    levelNode = loadLevel("TestScene");
    rootNode.attachChild(levelNode);
    game.setLevelName("TestScene");
    ```

1.  在`simpleInitApp`方法内部，我们将告诉应用程序从第一章，*SDK 游戏开发中心*加载`TestScene`：

    ```java
    LoadLevelMessage levelMessage = new LoadLevelMessage();
    levelMessage.setLevelName(game.getLevelName());
    server.broadcast(Filters.in(conn), levelMessage);
    ```

1.  最后，在`addPlayer`方法内部，我们需要创建并发送消息给连接的客户端。这就是服务器端的所有事情。

1.  在客户端，我们创建一个`levelNode`字段和一个`loadLevel`方法，但略有不同：

    ```java
    public void loadLevel(final String levelName){
      enqueue(new Callable(){
        public Object call() throws Exception {
          if(rootNode.hasChild(levelNode)){
            rootNode.detachChild(levelNode);
            }
            levelNode = (Node) assetManager.loadModel("Scenes/"+levelName + ".j3o");
            rootNode.attachChild(levelNode);
            return null;
        }
      });
    }
    ```

1.  我们需要确保我们在正确的时间点操纵场景图，以便我们可以在`enqueue`块内分离和附加节点。

1.  最后，我们确保`MessageListener`能够接收到`LoadLevelMessage`，如下所示：

    ```java
    else if (m instanceof LoadLevelMessage){
      gameClient.loadLevel(((LoadLevelMessage)m).getLevelName());
      game.setLevelName(((LoadLevelMessage)m).getLevelName());
    }
    ```

1.  就这样！当我们再次连接到服务器时，我们应该看到一个熟悉的场景。

## 它是如何工作的...

当客户端加入时，服务器创建一个`LoadLevelMessage`类，并用当前加载的关卡名称填充它。服务器不提供关卡本身，但客户端必须之前提供这些关卡。《LoadLevelMessage`类在这种情况下只提供一个名称，这在许多情况下可能已经足够。对于某些游戏，在加载关卡时支持自定义路径是一个好主意，因为它允许有更多的定制选项。

# 在玩家位置之间进行插值

如果我们只在一个局域网环境中运行我们的游戏，我们可能永远不会期望低延迟或任何显著的丢包。虽然现在许多人都有良好的互联网连接，但时不时地，问题仍然会发生。尝试减轻这些问题的技巧之一是在客户端使用插值来处理实体。

这意味着客户端将不会只是应用从服务器获取的位置和旋转，而是会逐步移动到目标位置和旋转。

## 如何操作...

执行以下步骤以在玩家位置之间进行插值：

1.  为了模拟一些网络问题，将服务器的`framerate`设置为`10`。

1.  如果你现在连接到服务器，移动将会显得明显地跳跃。

1.  我们用以下行替换了`ClientPlayerControl`的`controlUpdate`方法的内 容，以应用插值：

    ```java
    float factor = tpf / 0.03f; spatial.setLocalTranslation(spatial.getLocalTranslation().interpolateLocal(tempLocation, factor)); spatial.setLocalRotation(spatial.getLocalRotation().slerp(spatial.getLocalRotation(), tempRotation, factor));
    ```

1.  当我们再次连接并比较体验时，它将会更加平滑。

## 它是如何工作的...

为了模拟存在如丢包等问题的情况，我们将服务器的 FPS 更改为 10。它不会像之前每秒发送 30 个更新那样，而是每十分之一秒发送一个更新。这并不等同于 100 毫秒的延迟，因为它没有说明往返时间。这更像是每三个更新中有两个在途中丢失，即 66%的丢包率。

之前，客户端只是简单地将其从服务器获取的值应用到本地玩家上。使用插值，玩家的位置和旋转将每更新一次逐步移动到最新的实际位置和旋转。

我们通过首先确定插值因子来实现插值。这是通过将`tpf`除以我们希望插值所花费的时间（大致上，以秒为单位）来完成的。实际时间会更长，因为每次更新步骤会变得更短。

然后，我们输入这个值，并使用`Vector3f`的插值方法和`Quaternion`的`slerp`方法将它们移动到实际值。

这是通过使用`update`方法中提供的`tpf`值作为系数来实现的。通过这样做，插值时间将与帧率无关。我们应该意识到，在现实中这变成了延迟，即动作和外观之间的延迟，因为我们已经稍微延迟了玩家到达实际位置的时间。

# 在网络上射击

一个 FPS 游戏如果没有实际射击功能，那就不能算作射击游戏。我们将通过一个带有可见、非瞬间子弹的例子来看一下。为此，我们将能够重用第二章中的代码，*相机和游戏控制*。配方不会描述实际的碰撞，因为这已经在那一章中描述过了。

## 如何做到这一点...

在网络上射击的步骤如下：

1.  首先，我们创建一个新的消息，称为`BulletUpdateMessage`，用于发送子弹位置更新。它只需要两个字段：一个用于位置的`Vector3f`字段和一个表示是否存活的双精度布尔字段。

1.  我们将在`ServerMessageHandler`的`messageReceived`方法中添加一个检查，以查看是否有玩家在射击。我们想要进行的任何动作验证都应该在这个之前发生：

    ```java
    if(message.getAction().equals("Fire") && message.isPressed()){
      server.onFire(p);
    }
    ```

1.  我们找出玩家面向的方向并创建一个新的`ServerBullet`实例。它被分配了下一个可用的对象 ID，并添加到`bullets`列表中，如下所示：

    ```java
    public void onFire(NetworkedPlayerControl player){
      Vector3f direction = player.getSpatial().getWorldRotation().getRotationColumn(2);
      direction.setY(-player.getYaw());
      ServerBullet bullet = new ServerBullet(player.getSpatial().getWorldTranslation().add(0, 1, 0), direction);
      bullet.setId(nextObjectId++);
      bullets.add(bullet);
    }
    ```

1.  现在，我们需要在`simpleUpdate`方法中添加另一个代码块来维护子弹并发送消息，如下所示：

    ```java
    int nrOfBullets = bullets.size();
    for(int i = 0; i < nrOfBullets; i++){
      ServerBullet bullet = bullets.get(i);
      bullet.update(tpf);
      BulletUpdateMessage update = new BulletUpdateMessage();
      update.setId(bullet.getId());
      update.setPosition(bullet.getWorldPosition());
      update.setAlive(bullet.isAlive());
      server.broadcast(update);
      if(!bullet.isAlive()){
        bullets.remove(bullet);
        nrOfBullets--;
        i--;
      }
    }
    ```

1.  在一个`for`循环中，我们首先更新子弹，然后创建一个新的`BulletUpdateMessage`，将其发送给所有玩家。如果子弹超出范围，它将从列表中移除。这如下实现：

    ```java
    if (m instanceof BulletUpdateMessage){
      BulletUpdateMessage update = (BulletUpdateMessage) m;
      ClientBullet bullet = gameClient.getBullet(update.getId());
      if(bullet == null){
        bullet = gameClient.createBullet(update.getId());
      }
      bullet.setPosition(update.getPosition());
      if(!update.isAlive()){
        gameClient.removeBullet(update.getId(), bullet.getSpatial());
      }
    }
    ```

1.  在客户端，我们编写一个新的方法，一旦从服务器接收到信息，就创建一个新的子弹：

    ```java
    public ClientBullet createBullet(int id){
      final ClientBullet bulletControl = new ClientBullet();
      final Spatial g = assetManager.loadModel("Models/Banana/banana.j3o");
      g.rotate(FastMath.nextRandomFloat(), FastMath.nextRandomFloat(), FastMath.nextRandomFloat());
      g.addControl(bulletControl);
      bullets.put(id, bulletControl);
      rootNode.attachChild(g);
      return bulletControl;
    }
    ```

1.  然后，一旦我们从服务器接收到信息，我们需要一个`removeBullet`方法。

## 它是如何工作的...

与之前的配方一样，控制权在服务器手中。客户端只是表示想要射击，任何检查都在服务器端进行（尽管在客户端模拟验证以节省带宽是可以的）。配方中不包含任何特定的验证（玩家可以随时射击），但这在第二章，*相机和游戏控制*中有更详细的解释。

与第二章，*相机和游戏控制*不同，我们不能使用相机作为输入；相反，我们使用射击玩家的方向，并应用偏航来调整上下倾斜。

服务器和客户端上的子弹是不同的。在服务器上，它们仅仅是逻辑对象。就像第二章中“发射非即时子弹”菜谱的非即时子弹一样，它们像慢射线一样穿过世界，直到击中某个物体或超出范围。

在客户端，子弹与服务器端略有不同，并基于控制模式。客户端在接收到第一条更新时通过`ClientMessageHandler`了解子弹。它检查`ClientBullet`是否存在，如果不存在，它将创建一个新的子弹。然后`ClientBullet`所做的只是更新`controlUpdate`方法中的位置。

不是实际的射击消息创建了子弹，而是在客户端接收到第一条`BulletUpdateMessage`时。客户端将像玩家位置一样持续更新子弹的位置，直到收到一条消息表示它不再存活。此时，它将被移除。

当前菜谱向所有玩家发送所有子弹。与玩家一样，这可能（并且可能应该）基于需要知道的基础来避免欺骗（和过度使用带宽）。

# 优化带宽和避免欺骗

可以总结如下：客户拥有的信息越少，利用这些信息进行欺骗的机会就越小。同样，客户需要的信息越少，所需的带宽就越少。

以前，我们慷慨地发送了关于每个玩家、每个更新周期的信息。在这个菜谱中，我们将改变这一点，让服务器检查哪些玩家可以被其他人看到，并且只发送这些信息。

我们将在“实现 FPS 网络代码”菜谱的基础上构建这个功能。

我们需要在服务器应用程序的`simpleUpdate`方法中添加一些复杂性。因此，我们不需要向每个人发送所有玩家的信息，而需要检查谁应该接收什么。

## 如何做到这一点...

执行以下步骤以优化带宽：

1.  首先，我们将向我们的`PlayerUpdateMessage`添加一个可见字段。这样做是为了让客户端知道何时有玩家从视图中消失。

1.  在服务器端，我们需要更改两个类。首先，我们的`ServerPlayerControl`需要维护一个当前看到的玩家 ID 列表。

1.  在我们进行检查之前，我们需要确保所有玩家都已更新：

    ```java
    Collection<NetworkedPlayerControl> players = game.getPlayers().values();
      for(NetworkedPlayerControl p: players){
        p.update(tpf);
      }
    ```

1.  接下来，我们遍历我们的`playerMap`对象。在这里，我们添加一个简单的范围检查，以查看玩家是否可见，最后将信息广播给相关玩家，如下所示：

    ```java
    Iterator<HostedConnection> it = playerMap.keySet().iterator();
    while(it.hasNext()){
      HostedConnection conn = it.next();
      ServerPlayerControl player = playerMap.get(conn);
      for(NetworkedPlayerControl otherPlayer: players){
        float distance = player.getSpatial().getWorldTranslation().distance(otherPlayer.getSpatial().getWorldTranslation());
      PlayerUpdateMessage updateMessage = null;
      if(distance < 50){
        updateMessage = createUpdateMessage(otherPlayer);
        player.addVisiblePlayer(otherPlayer.getId());
      } else if (player.removeVisiblePlayer(otherPlayer.getId())){
        updateMessage = createUpdateMessage(otherPlayer);
        updateMessage.setVisible(false);
      }
      if(updateMessage != null){
        server.broadcast(Filters.in(conn), updateMessage);
      }
    }
    ```

1.  服务器端的操作到此结束。在客户端，我们需要向`ClientPlayerControl`添加一个可见字段。

1.  我们做的第二个更改是在`ClientMessageHandler`中。我们检查玩家是否应该被看到，以及它是否连接到场景图：

    ```java
    if(p.isVisible() && p.getSpatial().getParent() == null){
      gameClient.getRootNode().attachChild(p.getSpatial());
    } else if (!p.isVisible() && p.getSpatial().getParent() != null){
      gameClient.getRootNode().detachChild(p.getSpatial());
    }
    ```

## 它是如何工作的...

通过使用这个原则，每个客户端将只接收其他相关玩家的更新。然而，我们不能仅仅停止发送某些玩家的更新，而不让客户端知道原因，否则他们就会停留在最后已知的位置。这就是为什么服务器发送给玩家的最后一条消息是`visible`设置为`false`。然而，为了做到这一点，服务器必须跟踪玩家何时消失，而不仅仅是他们何时不可见。这就是为什么每个`ServerPlayerControl`类都需要跟踪它在`visibleList`中最后一次更新的玩家。

这个配方专注于可见性的网络方面，以及何时以及如何发送更新。一个合适的游戏（至少是一个第一人称射击游戏）还需要跟踪被遮挡的玩家，而不仅仅是他们距离有多远。

优化可以通过不同的方式进行，这都取决于应用。例如，一个大型多人在线游戏可能并不像其他游戏那样依赖于频繁的更新。在这样的游戏中，如果玩家距离较远，网络更新可以不那么频繁，而是依靠良好的插值来避免跳跃感。

如果我们使用插值而不是绝对更新，那么当可见性从假变为真时，我们也应该关闭插值，以避免玩家可能滑到新位置。我们也可以在可见性为假时关闭更新。

## 参见

+   第五章中的 *感知-视觉* 配方，在《人工智能》一书中，它提供了一个在服务器上实现视觉的方法的想法
