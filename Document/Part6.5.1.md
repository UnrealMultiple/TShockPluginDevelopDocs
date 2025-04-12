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
  | ID | 名字 | 含义 |
  |:----:|------|------|
  |-1 | AwaitingPassword | 服务器受密码保护，连接处于等待状态，直到客户端发送密码。 |
  | 0 | AwaitingVersionCheck | TCP连接已建立，客户端必须验证其版本。 |
  | 1 | AssigningPlayerSlot | 服务器已接受客户端的连接密码/已验证客户端的版本字符串正确。客户端正在被分配玩家位置。 |
  | 2 | AwaitingPlayerInfo | 客户端已收到玩家位置，服务器正在等待玩家信息。 |
  | 3 | RequestingWorldData | 已收到玩家信息，客户端正在请求世界数据。 |
  | 4 | ReceivingWorldData | 世界数据正在发送给客户端。 |
  | 5 | FinalizingWorldLoad | 已接收世界数据，客户端正在完成加载。 |
  | 6 | RequestingTileData | 客户端正在请求图块数据。 |
  |10 | Complete | 连接过程完成（玩家已生成），客户端已完全加入游戏。 |

- 在Terraria中负责解析数据包的`MessageBuffer.GetData`的前几个数据包就有用来判断玩家连接状态的代码
  ```csharp
  case 3:
    if (Main.netMode != 1) //属于客户端的逻辑
      break;
    if (Netplay.Connection.State == 1) //如果客户端连接状态为`AssigningPlayerSlot(1)`
      Netplay.Connection.State = 2; //将客户端的连接状态设为`AwaitingPlayerInfo(2)`
  ```
## 连接流程 (连接状态都是指服务器的客户端连接状态)
1. 建立TCP连接  
   客户端向服务器发起TCP连接，经过三次握手，成功创建TCP连接，此时连接状态均为AwaitingVersionCheck(0)
2. 客户端 -> 服务端: `ConnectRequest(1)`, 
   这个数据包包含了客户端的版本信息。收到此数据包后若服务器设置了密码，服务器将会发送
   



