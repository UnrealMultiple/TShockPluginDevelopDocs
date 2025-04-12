# Part 6.5.2 客户端与服务端的连接 (不要求学会，仅做参考)

本章你将学到：  
- 客户端连接服务端的过程
- 对Terraria的数据协议有所了解

## 为什么学这个？
如果你想写个多世界代理，或者了解多世界代理是如何工作，又或者你想在客户端与服务端连接时做一些额外的事情。  
例如: CaiBotPlugin(现在叫Lite了)处理掉玩家发送PlayerInfo(4)来校验白名单, 跨版本插件处理ConnectRequest(1)数据包来实现忽略版本检测
## 几个概念
### 数据包
- 数据包是服务器和客户端通信的最小单位。其实客户端和服务器通信并不难理解，就像两个人互发短信一样，而服务器和客户端的短信就是数据包
- 数据包根据发送方向可分为3种，分别是`服务端数据包`、`客户端数据包`、`同步数据包`
  | 类型 | 方向 | 描述 |
  | :----: | ----------- | ---- |
  | 服务端数据包 | 服务器 -> 客户端 | 只有服务端发送有效，且只有客户端会处理 |
  | 客户端数据包 | 服务器 <- 客户端 | 只有客户端发送有效，且只有服务端会处理 |
  | 同步数据包 | 服务器 <-> 客户端 | 顾名思义，客户端和服务器用来同步状态的数据包，是双向的，服务器和客户端都会处理 |

### 网络模式

- 为什么会有`网络模式(NetMode)`?因为Terraria服务端和客户端都共用一套代码(基本上都一样), 所以在处理数据包时，就要知道本程序是服务器还是客户端，应该执行哪一段逻辑
- 网络模式也叫NetMode，对应的字段是`Main.netMode`，它表示程序在数据传输中的角色，在目前的Terraria有3种网络模式
  | ID | 模式 | 描述|
  | :----: | ----------- |---- |
  | 0 | 本地 | 单人模式客户端 |
  | 1 | 客户端 | 多人模式客户端 |
  | 2 | 服务端 | 多人模式服务端 |
- 在Terraria中负责解析数据包的`MessageBuffer.GetData`就有许多代码用来区分客户端、服务器，但是没有本地 :(
  ```csharp
  case 2: //在协议中，Disconnect(2)数据包代表踢出客户端
    if (Main.netMode != 1) //若不是客户端模式则退出，说明这个数据包只有客户端才处理，是服务端数据包。
      break;
    Netplay.Disconnect = true; //将客户端状态设为断开连接
    Main.statusText = NetworkText.Deserialize(this.reader).ToString(); //显示踢出的理由
    break;
  ```

### 连接状态
- 连接状态表示客户端的连接状态，用来区分不同的连接阶段，在代码中对应的字段为`Netplay.Connection(客户端)`、`Netplay.Clients[Index].State(服务端)`、`TSPlayer.State(TShock, 与服务端的等价)`
- 连接状态也分为两种
  - 服务端的客户端连接状态 `Netplay.Clients[Index].State`
    表示服务器连接的每个客户端的连接状态
  - 客户端连接状态 `Netplay.Connection`
    表示本地客户端的连接状态
    
  | ID | 名字                  | 含义                                          |
  |:--:|----------------------|---------------------------------------------|
  | -1 | AwaitingPassword     | 服务器受密码保护，连接处于等待状态，直到客户端发送密码。                |
  | 0  | AwaitingVersionCheck | TCP连接已建立，客户端必须验证其版本。                        |
  | 1  | AssigningPlayerSlot  | 服务器已接受客户端的连接密码/已验证客户端的版本字符串正确。客户端正在被分配玩家位置。 |
  | 2  | AwaitingPlayerInfo   | 客户端已收到玩家位置，服务器正在等待玩家信息。                     |
  | 3  | RequestingWorldData  | 已收到玩家信息，客户端正在请求世界数据。                        |
  | 4  | ReceivingWorldData   | 世界数据正在发送给客户端。                               |
  | 5  | FinalizingWorldLoad  | 已接收世界数据，客户端正在完成加载。                          |
  | 6  | RequestingTileData   | 客户端正在请求图块数据。                                |
  | 10 | Complete             | 连接过程完成（玩家已生成），客户端已完全加入游戏。                   |

- 在Terraria中负责解析数据包的`MessageBuffer.GetData`的前几个数据包就有用来判断玩家连接状态的代码
  ```csharp
  case 3:
    if (Main.netMode != 1) //属于客户端的逻辑
      break;
    if (Netplay.Connection.State == 1) //如果客户端连接状态为`AssigningPlayerSlot(1)`
      Netplay.Connection.State = 2; //将客户端的连接状态设为`AwaitingPlayerInfo(2)`
  ```
## 连接流程 (注意: 服务器和客户端连接状态是不同的!!!)
1. 客户端(0) <-> 服务器: 建立TCP连接
   客户端向服务器发起TCP连接，经过三次握手，成功创建TCP连接  
   客户端: 将连接状态`AwaitingVersionCheck(0)`设为`AssigningPlayerSlot(1)`  
   服务器: 将连接状态设为`AwaitingVersionCheck(0)`  
3. 客户端(1) -> 服务器(0): `ConnectRequest(1)`,  
   这个数据包包含了客户端的版本信息。 
   服务器: 若服务器设置了密码，服务器将会将连接状态设为`AwaitingPassword(-1)`并发送数据包`PasswordRequired(37)`,若没有密码则直接将连接状态设为`AssigningPlayerSlot(1)`然后发送数据包`ContinueConnecting(3)`  
4. 服务器(-1) -> 客户端(1): `PasswordRequired(37)` (没有密码则跳过3,4步骤)  
   这个数据包不包含数据，只用来表示服务器请求客户端提交密码  
   客户端: 让用户输入服务器密码，并且发送数据包`PasswordSend(38)`  
5. 客户端(1) -> 服务器(-1): `PasswordSend(38)`  
   这个数据包传输客户端发送的密码  
   服务器: 若密码正确则将连接状态设为`AssigningPlayerSlot(1)`然后发送数据包`ContinueConnecting(3)`, 若密码不正确则发送数据包`Disconnect(2)`踢出玩家  
6. 服务器(1) -> 客户端(1): `ContinueConnecting(3)`   
   这个数据包表示连接继续  
   客户端: 将连接状态设为`AwaitingPlayerInfo(2)`, 并且发送数据包`PlayerInfo(4)`、`ClientUUID(68)`、`PlayerHp(16)`、`PlayerMana(42)`、 `PlayerBuff(50)`、`SyncLoadout(147)`、`PlayerSlot(5)`x背包格数(含装备、储罐等)、`ContinueConnecting2(6)`，发送完成后将连接状态设置为`RequestingWorldData(3)`  
7. 客户端(3) -> 服务器(1): `PlayerInfo(4)`    
   这个数据包包含玩家的一些基本信息，比如名字、皮肤等。  
   服务器: 若玩家名字合规、玩家难度与地图匹配则，服务器向其他客户端(除正在连接的客户端)发送数据包`PlayerInfo(4)`(同步给其他玩家)  
8. 客户端(3) -> 服务器(1): `ClientUUID(68)`   
   这个数据包包含玩家的客户端UUID  
   服务器: 只有TShock服务器才会处理UUID，并且将UUID储存在`Netplay.Clients[Index].ClientUUID  `
9. 客户端(3) -> 服务器(1): `ContinueConnecting2(6)`  
   这个数据包表面客户端已经将角色信息完全同步给客户端，客户端请求继续连接  
   服务器: 将连接状态设为`AwaitingPlayerInfo(2)`, 并且发送数据包`WorldInfo(7)`，并且向玩家同步入侵信息  
10. 服务器(2) -> 客户端(3): `WorldInfo(7)`  
   这个数据包包含世界的基本信息, 例如时间、出生点位置等，用来同步世界信息  
   客户端: 同步世界信息，并将连接状态设置为`ReceivingWorldData(4)`，然后设置连接状态为`FinalizingWorldLoad(5)`，接着完成一些初始化世界的操作，然后将连接状态设置为`RequestingTileData(6)`，最后发送数据包`TileGetSection(8)`  
11. 客户端(6) -> 服务器(2): `TileGetSection(8)`  
   这个数据包用来请求服务器的区块数据，用来同步地图  
   服务器: 发送数据包`WorldInfo(7)`同步世界信息，将连接状态设为`RequestingWorldData(3)`, 然后发送数据包`Status(9)`客户端显示正在接受图格信息, 并开始发送区块`SendSection`（发送多组数据包`TileSendSection(10)`和`NpcUpdate(23)`分别同步区块和生物，接着发送数据包`ItemDrop(21)`和`ItemOwner(22)`同步掉落物, 然后发送多个数据包`NpcKillCount(83)`来同步图鉴，然后发送数据包`UpdateGoodEvil(57)`更新邪恶和神圣，发送数据包`MoonLordCountdown(103)`同步月总倒计时，发送数据包`UpdateShieldStrengths(101)`同步四柱进度，发送数据包`SyncCavernMonsterType(136)`同步这个世界的洞穴怪物类型(蝾螈，卷壳怪，龙虾三选二)，最后发送数据包`PlayerSpawnSelf(49)`生成玩家
12. 服务器(3) -> 客户端(6): `PlayerSpawnSelf(49)`  
    这个数据包用来生成玩家，客户端收到此数据包后玩家会出生在出生点  
    客户端: 将连接状态设为`Complete(10)`完成连接, 并且生成玩家，并且发送数据包`PlayerSpawn(12)`以完成出生  
13. 客户端(10) -> 服务器(3): `PlayerSpawn(12)`  
    这个数据包用来同步告诉服务器玩家已经出生  
    服务器: 将连接状态设为`Complete(10)`完成连接，接着服务器会判断玩家是否为是Host玩家(直接使用多人模式开服), 接着向服务器向其他客户端(除正在连接的客户端)发送数据包`PlayerSpawn(12)`以同步玩家的出生，接着发送`PlaceObject(79)`，然后发送`FinishedConnectingToServer(129)`完成连接, 最后发送欢迎消息  
14. 服务器(10) -> 客户端(10): `FinishedConnectingToServer(129)`  
    这个数据包标志着连接完成，客户端会进行世界和UI的设置，玩家正式加入游戏。





