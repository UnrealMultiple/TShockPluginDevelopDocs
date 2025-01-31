# Part 4.挂钩钩~

本章你将学到：  

- 学习如何注册、卸载钩子
- 学会使用常用钩子修改游戏事件

## 什么是钩子
- 钩子一种事件处理的机制。当游戏中发生某些事件前后，你可以通过钩子来获取这个事件，包括它的参数等等，然后通过钩子的处理方法，来实现一些你需要的功能，必要时你可以阻止这个事件发生。  
- 钩子并不是一直检测游戏有没有这个事件，而是游戏出现这个事件后再调用这个钩子。打个比方，就是老师发现你上课睡觉，然后打算一个黑板擦砸爆你，此时`老师发现你上课睡觉，然后打算一个黑板擦砸`就是个事件，你可以通过钩子，来获取这个事件还有他的参数(例如`黑板擦的速度`、`方位`等)，你可以通过编写钩子的`处理方法`来做加入自己需要的逻辑或者修改这个事件的某些东西，比如你为了不被打中，你可以将`黑板擦的速度`改为`0`，这样黑板擦就会停下，当然你也可以调用你的`Miss()`方法来躲开黑板擦，甚至你可以`取消这个事件`，阻止事件发生。
- 举个泰拉的例子就是，如果你想记录玩家发送的表情，那么你就要把钩子挂在`玩家发送表情`这个事件上，然后编写这个钩子的`处理方法`，去实现你需要的功能


## 举个例子：
假如你需要实现当玩家说“wdnmd”时把他踢出去，那么你就可以注册一个`ServerChat`钩子，并且调用`OnChatHandler`方法处理事件。你需要在`OnChatHandler`方法中编写代码，识别玩家的聊天是否包含`wdnmd`,最后别忘了注销你的钩子。
> [!NOTE]  
> 钩子并不一定是下面这种固定的形式，下面这个属于`ServerApi`钩子的一种，并不是所有的钩子都是下面这种形式，但是大同小异  
> 下面只是简单列举例子，看不懂真的没关系，直接复制就好

```csharp
//以下为示例代码，看不懂没事，我们慢慢学
using System.Reflection;
using Terraria;
using TerrariaApi.Server;
using TShockAPI;

namespace NoEggplant
{
    [ApiVersion(2, 1)]
    public class Plugin : TerrariaPlugin
    {
        //定义插件的作者名称
        public override string Author => "Cai";

        //插件的一句话描述
        public override string Description => "神秘示例插件";

        //插件的名称
        public override string Name => "No Eggplant";

        //插件的版本
        public override Version Version => new(2024,1,18,1);

        //插件的构造器
        public Plugin(Main game) : base(game)
        {
        }

        //插件加载时执行的代码
        public override void Initialize()
        {
            ServerApi.Hooks.ServerChat.Register(this, OnChatHandler); //注册聊天钩子
        }

        private void OnChatHandler(ServerChatEventArgs args)
        {
            TSPlayer plr = TShock.Players[args.Who]; //获取TShock玩家实例
            if (args.Text.Contains("wdnmd")) //检测聊天是否含有"wdnmd"
            {
                plr.Kick("不准说wdnmd", true); //踢出玩家
            }
        }

        //插件卸载时执行的代码
        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                ServerApi.Hooks.ServerChat.Deregister(this, OnChatHandler); //注销聊天钩子
            }
            base.Dispose(disposing);
        }

    }
}
```


## 代码解析：

### 注册钩子
注册钩子好比你设置一条定时命令一样，我们要设置`什么时候触发我们的处理方法`，我们的`处理方法的名字`。不同类型的钩子有不同的注册语句，`ServerApi.Hooks.ServerChat.Register(this, OnChatHandler);`是`ServerApi`钩子的注册语句，`ServerApi.Hooks`是钩子存放的位置，`ServerChat`则是我们注册钩子的事件(服务器聊天), `this`表示钩子在这个类里(不会不要管), `OnChatHandler`则是钩子的处理方法，这点和命令有点像。这句话完整的意思是注册一个`ServerApi钩子`，在发生事件`ServerChat(服务器聊天)`时调用处理方法`OnChatHandler`
> [!NOTE]  
> 处理方法需要开发者自己命名，一般命名为`On+钩子名`或者 `钩子名+Handler`  
> 例如: `ServerChat钩子` -> `OnServerChat` / `ServerChatHandler`  
> 当然，因为是开发者自己去名字，只要你乐意其实啥都行，但最好按照规范来
```csharp
//插件加载时执行的代码
public override void Initialize()
{
    ServerApi.Hooks.ServerChat.Register(this, OnChatHandler); //注册聊天钩子
}
```

### 注销钩子
如果你要使用热重载，那么插件卸载就非常有必要了。不同类型的钩子有不同的卸载语句`ServerApi.Hooks.ServerChat.Deregister(this, OnChatHandlert);` 就是`ServerApi`钩子的卸载语句，钩子要在插件卸载时注销(一般在Dispose卸载)。`this`表示钩子在这个类里(不会不要管), `OnChatHandler`则是钩子的处理方法，这和注册是一样的
```csharp
//插件卸载时执行的代码
protected override void Dispose(bool disposing)
{
    if (disposing)
    {
        ServerApi.Hooks.ServerChat.Deregister(this, OnChatHandler); //注销聊天钩子
    }
    base.Dispose(disposing);
}
```
### 处理方法  
处理方法是触发钩子时执行的方法，创建方法类似命令的处理方法。写处理方法前，需要在`Initialize()`先注册钩子:
```csharp
using System.Reflection;
using Terraria;
using TerrariaApi.Server;
using TShockAPI;

namespace NoEggplant
{
    [ApiVersion(2, 1)]
    public class Plugin : TerrariaPlugin
    {
        public override string Author => "Cai";
        public override string Description => "神秘示例插件";
        public override string Name => "No Eggplant";
        public override Version Version => new(2024,1,18,1);

        public Plugin(Main game) : base(game)
        {
        }

        public override void Initialize()
        {
            ServerApi.Hooks.ServerChat.Register(this, OnChatHandler); //注册聊天钩子
        }

        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                ServerApi.Hooks.ServerChat.Deregister(this, OnChatHandler); //注销聊天钩子
            }
            base.Dispose(disposing);
        }

    }
}
```
生成处理方法，我们一般使用Rider的快速操作，而不是手搓。右键方法名，点击快速操作即可:  
![PixPin_2025-01-18_14-55-05](https://github.com/user-attachments/assets/5605244c-f845-4e40-b3ee-770e87ac67f0)  

## 常用钩子​

### ServerApi钩子  
`ServerApi`的钩子覆盖了服务器大部分可能需要挂钩的事件，同时值得注意的是`ServerApi`钩子中参数的命名和Terraria原版相似 
<details>
<summary> ServerApi钩子汇总</summary>
    
| 钩子名称                               | 参数含义                                                                                                                                                                                                                              | 钩子含义                   |
|------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------|
| **GameUpdate**                     | -                                                                                                                                                                                                                                 | 游戏主循环更新时触发             |
| **GamePostUpdate**                 | -                                                                                                                                                                                                                                 | 游戏主循环更新完成后触发           |
| **GameInitialize**                 | -                                                                                                                                                                                                                                 | 游戏初始化开始时触发             |
| **GamePostInitialize**             | -                                                                                                                                                                                                                                 | 游戏初始化完成后触发             |
| **GameHardmodeTileUpdate**         | `X` (图格的X坐标), `Y` (图格的Y坐标), `Type` (图格的类型)                                                                                                                                                                                        | 困难模式下世界更新或放置图格         |
| **GameStatueSpawn**                | `X` (生成位置的X坐标), `Y` (生成位置的Y坐标), `Type` (生成的NPC或物品类型), `Npc` (是否生成NPC)                                                                                                                                                             | 雕像生成NPC或物品时触发          |
| **NetSendData**                    | `MsgId` (数据包类型), `remoteClient` (目标客户端ID), `text` (文本消息数据), n个参数                                                                                                                                                                  | 服务器发送数据包前触发            |
| **NetSendNetData**                 | `netManager` (NetManager实例), `socket` (目标套接字), `packet` (数据包)                                                                                                                                                                     | NetManager发送数据包前触发     |
| **NetGetData**                     | `MsgID` (接收的数据包类型), `Msg` (数据缓冲区), `Index` (数据起始索引)等                                                                                                                                                                              | 服务器接收数据包时触发            |
| **NetSendBytes**                   | `Socket` (目标套接字), `Buffer` (发送的字节数据), `Offset` (数据偏移)等                                                                                                                                                                            | 发送原始字节数据前触发            |
| **NetNameCollision**               | `Who` (玩家索引), `Name` (冲突的玩家名称)                                                                                                                                                                                                    | 玩家名称冲突时触发              |
| **NpcKilled**                      | `npc` (被击杀的NPC实例)                                                                                                                                                                                                                 | NPC被击杀时触发              |
| **NpcStrike**                      | `Npc` (被攻击的NPC), `Damage` (伤害值), `KnockBack` (击退值), `Critical` (是否暴击), `FromNet` (来自玩家数据包), `NoEffect` (无作用),                                                                                                                     | NPC受到攻击时触发             |
| **NpcTransform**                   | `NpcId` (变换后的NPC索引)                                                                                                                                                                                                               | NPC发生微光转换时触发           |
| **NpcSpawn**                       | `NpcId` (生成的NPC索引)                                                                                                                                                                                                                | NPC生成前触发               |
| **NpcLootDrop**                    | `NpcId` (NPC类型ID),`NpcArrayIndex` (NPC索引),`NoGrabDelay` (是否禁用拾取延迟),`ItemId` (掉落的物品ID), `Stack` (掉落数量), `Prefix` (物品的前缀), `Position` (掉落位置),`Width` (掉落散布宽度), `Height` (掉落散布高度), `Broadcast` (不发包),`ReverseLookup` (从后向前查找可用掉落物索引) | NPC掉落物品时触发             |
| **DropBossBag**                    | `NpcId` (NPC类型ID),`NpcArrayIndex` (NPC索引),`NoGrabDelay` (是否禁用拾取延迟),`ItemId` (掉落的物品ID), `Stack` (掉落数量), `Prefix` (物品的前缀), `Position` (掉落位置),`Width` (掉落散布宽度), `Height` (掉落散布高度), `Broadcast` (不发包),`ReverseLookup` (从后向前查找可用掉落物索引) | Boss死亡时掉落Boss袋前触发      |
| **NpcTriggerPressurePlate**        | `Object` (触发压力板的NPC), `TileX` (压力板的X坐标), `TileY` (压力板的Y坐标)                                                                                                                                                                        | NPC触发压力板时触发            |
| **NpcAIUpdate**                    | `Npc` (更新AI的NPC)                                                                                                                                                                                                                  | NPC的AI逻辑更新时触发          |
| **ProjectileAIUpdate**             | `Projectile` (正在更新AI的弹幕)                                                                                                                                                                                                          | 弹幕的AI逻辑更新时触发           |
| **ItemForceIntoChest**             | `Chest` (目标箱子), `Item` (被堆叠的物品), `Player` (触发此操作的玩家)                                                                                                                                                                              | 物品被快速堆叠放入箱子时触发         |
| **ServerCommand**                  | `Command` (输入的服务器命令)                                                                                                                                                                                                              | 执行服务器控制台命令前触发          |
| **ServerConnect**                  | `Who` (玩家索引)                                                                                                                                                                                                                      | 玩家尝试连接服务器时触发           |
| **NetGreetPlayer**                 | `Who` (玩家索引)                                                                                                                                                                                                                      | 服务器发送motd和'xxx加入游戏'前触发 |
| **ServerJoin**                     | `Who` (玩家索引)                                                                                                                                                                                                                     | 玩家成功加入服务器时触发           |
| **ServerLeave**                    | `Who` (玩家索引)                                                                                                                                                                                                                      | 玩家离开服务器时触发             |
| **ServerChat**                     | `Who` (玩家索引), `Text` (聊天内容), `CommandId` (命令类型, 聊天为Text)                                                                                                                                                                          | 服务器处理聊天消息前触发           |
| **ServerBroadcast**                | `Message` (广播的消息内容), `Color` (消息颜色)                                                                                                                                                                                               | 服务器发送广播消息前触发           |
| **ServerSocketReset**              | `Socket` (被重置的客户端套接字)                                                                                                                                                                                                             | 客户端套接字被重置时触发           |
| **PlayerTriggerPressurePlate**     | `Object` (触发压力板的玩家), `TileX` (压力板的X坐标), `TileY` (压力板的Y坐标)                                                                                                                                                                         | 触发压力板的玩家               |
| **NpcTriggerPressurePlate**        | `Object` (触发压力板的NPC), `TileX` (压力板的X坐标), `TileY` (压力板的Y坐标)                                                                                                                                                                        | NPC触发压力板时触发            |
| **ProjectileTriggerPressurePlate** | `Object` (触发压力板的弹幕), `TileX` (压力板的X坐标), `TileY` (压力板的Y坐标)                                                                                                                                                                         | 弹幕触发压力板时触发             |
| **WorldSave**                      | `ResetTime` (是否重置时间到白天13500刻, 并且做一些清理)                                                                                                                                                                                            | 世界保存前触发                |
| **WorldStartHardMode**             | -                                                                                                                                                                                                                                 | 世界即将进入困难模式时触发          |
| **WorldMeteorDrop**                | `X` (陨石坠落的TileX坐标), `Y` (陨石坠落的TileY坐标)                                                                                                                                                                                            | 陨石坠落前触发                |
| **WorldChristmasCheck**            | `Xmas` (当前是否为圣诞节事件)                                                                                                                                                                                                               | 检查圣诞节事件状态时触发           |
| **WorldHalloweenCheck**            | `Halloween` (当前是否为万圣节事件)                                                                                                                                                                                                          | 检查万圣节事件状态时触发           |
| **WorldGrassSpread**               | `TileX` (草蔓延的方块X坐标), `TileY` (草蔓延的方块Y坐标), `Dirt` (泥土类型), `Grass` (草的类型)                                                                                                                                                           | 草/邪恶/神圣蔓延时触发           |
| **WireTriggerAnnouncementBox**     | `TileX` (广播盒的X坐标), `TileY` (广播盒的Y坐标), `Text` (广播盒的文本内容)                                                                                                                                                                           | 触发广播盒时触发               |
| **ItemSetDefaultsInt**             | `Info` (物品的ID), `Object` (物品实例)                                                                                                                                                                                                   | 通过ID设置物品默认值时触发         |
| **ItemSetDefaultsString**          | `Info` (物品的名称), `Object` (物品实例)                                                                                                                                                                                                   | 通过名称设置物品默认值时触发         |
| **ItemNetDefaults**                | `Info` (物品的网络ID), `Object` (物品实例)                                                                                                                                                                                                 | 设置物品的网络默认值时触发          |
| **NpcSetDefaultsInt**              | `Info` (NPC的ID), `Object` (NPC实例)                                                                                                                                                                                                 | 通过ID设置NPC默认值时触发        |
| **NpcSetDefaultsString**           | `Info` (NPC的名称), `Object` (NPC实例)                                                                                                                                                                                                 | 通过名称设置NPC默认值时触发        |
| **NpcNetDefaults**                 | `Info` (NPC的网络ID), `Object` (NPC实例)                                                                                                                                                                                               | 设置NPC的网络默认值时触发         |
| **ProjectileSetDefaults**          | `Info` (弹幕的ID), `Object` (弹幕实例)                                                                                                                                                                                                   | 设置弹幕默认值时触发             |  
| **GameWorldConnect**               | -                                                                                                                                                                                                                                 | 几乎无作用                  |
| **GameWorldDisconnect**            | -                                                                                                                                                                                                                                 | 几乎无作用                  |
| **PlayerUpdatePhysics**            | `Player` (玩家实例)                                                                                                                                                                                                                   | 玩家物理状态更新时触发, 已废弃       |


</details>

```csharp
//插件加载时执行的代码
public override void Initialize()
{
    ServerApi.Hooks.ServerChat.Register(this, OnChatHandler); //注册ServerApi钩子(玩家聊天)
}

//处理方法
private void OnChatHandler(ServerChatEventArgs args)
{
    ...           
}

//插件卸载时执行的代码
protected override void Dispose(bool disposing)
{
    if (disposing)
    {
        ServerApi.Hooks.ServerChat.Deregister(this, OnChat); //注销ServerApi钩子(玩家聊天)
    }
    base.Dispose(disposing);
}
```
其中`args`参数中不包含`TSPlayer`实例或`Player`实例取而代之的是一个名为`Who`的字段，有时候也叫`WhoAmI`或者`Index`, 这些都统称为索引
Terraria中的玩家、生物、弹幕等等都存放在`Terraria.Main`的几个数组中，有唯一的索引，也叫做下标。Player实例就放在`Main.player`中,如果你想获取索引为`0`的Player实例,就使用`Main.player[0]`即可，`TShock.Players`中的TSPlayer实例的获取方法也是如此。不过值得注意的是，`Main.player`和`TShock.Players`都是数组，它们的`Length`代表他们的长度，而非玩家人数，当你使用一个不存在的玩家的索引时，将会`Main.player[114]`或`TShock.Players[233]`将会获取到null值，超过数组长度还会报错。
```csharp
private void OnChat(ServerChatEventArgs args)
{
    //args.Who(有的地方是WhoAmI)是`玩家索引`的意思，也就是在玩家实例数组中的下标(索引)。Who和WhoAmI等同于TSPlayer中的Index，Who和WhoAmI也可能表示的是生物(NPC)或者弹幕(Projectile)等的索引，这要取决于你的钩子
    //TShock.Players是TShock玩家实例(TSPlayer)数组，TShock玩家实例中的绝大部分属性、变量、方法都为插件编写服务，大部分都可以在服务端使用
    //Main.player是Terraria玩家实例(Player)数组，Terraria玩家实例是Terraria客服端和服务端都使用的，所以其中的属性、变量、方法在服务器中可能不会生效
    TSPlayer plr = TShock.Players[args.Who]; //获取TShock玩家实例，说白点就是用下标获取数组元素
    Player player = Main.player[args.Who]; //获取TShock玩家实例
}
```

> [!NOTE]  
> `args.Handled`可以标记这个事件是否已经处理，如果将其设为true服务器可能会忽略这个事件  
> `ServerApi.Hooks.ServerChat.Deregister(this, OnChat, 整数);` 可以调整`ServerApi`钩子的优先级，和`Order`不同的是，钩子优先级越高越优先触发

### TShockAPI钩子  
这些钩子主要挂钩了TShock中的事件，例如：新建账号、玩家进入区域等，TShockAPI钩子分为如下几类:  

 类型 | 包含钩子 
-------|----------
 AccountHooks | AccountCreate(创建账号)、AccountDelete(删除账号) 
 PlayerHooks | PlayerChat(玩家聊天)、PlayerCommand(玩家执行命令)、PlayerLogout(玩家登出)、PlayerPreLogin(玩家登录前)、PlayerPostLogin(玩家登录后)、PlayerPermission(权限检查)、PlayerHasBuildPermission(玩家建筑权限检查)、PlayerProjbanPermission(玩家服务器忽略禁用弹幕权限检查)、  PlayerTilebanPermission(玩家服务器忽略禁用图格权限检查)、  PlayerItembanPermission(玩家服务器忽略违禁物品权限检查) 
 GeneralHooks | ReloadEvent(使用/reload命令重载配置) |
 RegionHooks | RegionCreated(区域创建)、RegionDeleted(区域删除)、RegionRenamed(区域重命名)、RegionEntered(玩家进入区域)、  RegionLeft(玩家离开区域) 

TShockAPI钩子使用方法:  

```csharp
//插件加载时执行的代码
public override void Initialize()
{
    TShockAPI.Hooks.AccountHooks.AccountCreate += AccountHooks_AccountCreate;//注册TShockAPI钩子(玩家创建账号)
}

//处理方法
private void AccountHooks_AccountCreate(TShockAPI.Hooks.AccountCreateEventArgs e)
{
    TShock.Utils.Broadcast($"[欢迎新人]{e.Account.Name}注册了新账号，快举小花欢迎吧~", 0, 255, 120); //发送全服广播
}

//插件卸载时执行的代码
protected override void Dispose(bool disposing)
{
    if (disposing)
    {
        TShockAPI.Hooks.AccountHooks.AccountCreate -= AccountHooks_AccountCreate; ; //注销TShockAPI钩子(玩家创建账号)
    }
    base.Dispose(disposing);
}
```

### GetDataHandlers数据包钩子  
服务器会收到客户端发送的数据包，而`GetDataHandlers`钩子会解析这些数据包，触发对应的数据包钩子，并且打包成参数传入处理方法，但是`GetDataHandlers`不包含所有数据包。  
```csharp
//插件加载时执行的代码
public override void Initialize()
{
    GetDataHandlers.KillMe.Register(OnKillPlayer); //注册GetDataHandlers数据包钩子(玩家死亡)
}
//处理方法
private void OnKillPlayer(object? sender, GetDataHandlers.KillMeEventArgs e)
{
    TShock.Utils.Broadcast($"[死亡]{e.Player.Name}挂掉啦，快来嘲笑他吧!", 0, 255, 120); //发送全服广播
    e.Handled = false; //是否处理这个事件，默认为false，如果你标记为true，这个事件将会被忽略，会导致TShock直接忽略这个玩家死亡(不进行重生倒计时)
}

//插件卸载时执行的代码
protected override void Dispose(bool disposing)
{
    if (disposing)
    {
        GetDataHandlers.KillMe.UnRegister(OnKillPlayer); //注销GetDataHandlers数据包钩子(玩家死亡)
    }
    base.Dispose(disposing);
}
```


> [!NOTE]  
> `e.Handled`可以标记这个事件是否已经处理，如果将其设为`true`服务器可能会忽略这个事件 
> `GetDataHandlers.KillMe.UnRegister(OnKillPlayer, HandlerPriority.优先级);` 可以调整`GetDataHandlers`钩子的优先级，有以下优先级可选：  
> ![image](https://github.com/user-attachments/assets/baf61235-716d-4b65-bdc9-83fcfc17d843)

### On钩子
填坑中...

### IL钩子
超级危险，需要会IL


## 钩子练习题​

1. 使用ServerApi钩子实现，在玩家加入服务器时(ServerJoin,ServerLeave)广播"欢迎玩家XXX加入服务器!"，在玩家离开服务器时广播"玩家XXX离开服务器!"  
> 提示: 可以使用如下语句获取TShock玩家对象  
```csharp
TSPlayer plr = TShock.Players[args.Who]; //获取TShock玩家对象，args.Who是玩家的索引
```

<details>
<summary>参考答案</summary>
    

    ```csharp
    using System.Reflection;
    using Terraria;
    using TerrariaApi.Server;
    using TShockAPI;

    namespace Plugin
    {
        [ApiVersion(2, 1)]
        public class Plugin : TerrariaPlugin
        {
            //定义插件的作者名称
            public override string Author => "Cai";

            //插件的一句话描述
            public override string Description => "钩子测试";

            //插件的名称
            public override string Name => "Hooks";

            //插件的版本
            public override Version Version => Assembly.GetExecutingAssembly().GetName().Version;

            //插件的构造器
            public Plugin(Main game) : base(game)
            {
            }

            //插件加载时执行的代码
            public override void Initialize()
            {
                ServerApi.Hooks.ServerJoin.Register(this, OnJoin); //注册钩子
                ServerApi.Hooks.ServerLeave.Register(this, OnLeave);
            }
            private void OnJoin(JoinEventArgs args)
            {
                TSPlayer plr = TShock.Players[args.Who]; //获取TShock玩家对象，args.Who是玩家的索引
                TShock.Utils.Broadcast($"欢迎玩家{plr.Name}加入服务器!", 255, 255, 255); //发送广播Broadcast(内容,R,G,B) R,G,B是对应的RGB颜色代码
            }
            private void OnLeave(LeaveEventArgs args)
            {
                TSPlayer plr = TShock.Players[args.Who]; //获取TShock玩家对象，args.Who是玩家的索引
                TShock.Utils.Broadcast($"玩家{plr.Name}离开服务器!", 255, 255, 255); //发送广播Broadcast(内容,R,G,B) R,G,B是对应的RGB颜色代码
            }

            //插件卸载时执行的代码
            protected override void Dispose(bool disposing)
            {
                if (disposing)
                {
                    ServerApi.Hooks.ServerJoin.Deregister(this, OnJoin); //注销钩子
                    ServerApi.Hooks.ServerLeave.Deregister(this, OnLeave);
                }
                base.Dispose(disposing);
            }
        }
    }
    ```
</details>



2. 使用ServerApi钩子实现，在服务器加载完成后(GamePostInitialize)在控制台打印"服务器已经加载完啦!"  
>提示: GameInitialize和GamePostInitialize的区别：  
 他们都是游戏初始化钩子，但是GameInitialize是开始初始化时触发，而GamePostInitialize是完成初始化时触发(即地图加载完毕，服务器处于可加入状态)  



```csharp
ServerApi.Hooks.GameInitialize.Register(this,(args) => { Console.WriteLine("触发GameInitialize"); });
ServerApi.Hooks.GamePostInitialize.Register(this,(args) => { Console.WriteLine("触发GamePostInitialize"); });
```

![image](https://github.com/user-attachments/assets/b6b5e8a8-bcf6-4f3c-a4ac-fc3fd4e0eb23)
  

<details>
<summary>参考答案</summary>

```csharp
using System.Reflection;
using Terraria;
using TerrariaApi.Server;
using TShockAPI;

namespace Plugin
{
    [ApiVersion(2, 1)]
    public class Plugin : TerrariaPlugin
    {
        //定义插件的作者名称
        public override string Author => "Cai";

        //插件的一句话描述
        public override string Description => "钩子测试";

        //插件的名称
        public override string Name => "Hooks";

        //插件的版本
        public override Version Version => Assembly.GetExecutingAssembly().GetName().Version;

        //插件的构造器
        public Plugin(Main game) : base(game)
        {
        }

        //插件加载时执行的代码
        public override void Initialize()
        {
            ServerApi.Hooks.GamePostInitialize.Register(this, OnGamePostInitialize); //注册钩子
        }

        private void OnGamePostInitialize(EventArgs args)
        {
            Console.WriteLine("服务器已经加载完啦!");
        }

        //插件卸载时执行的代码
        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                ServerApi.Hooks.GamePostInitialize.Deregister(this, OnGamePostInitialize); //注销钩子
            }
            base.Dispose(disposing);
        }
    }
}
```
</details>


3. 使用TShockAPI钩子实现，在玩家完成登录后(PlayerHooks.PlayerPostLogin) 给玩家发送"您登录的IP是：xxx.xxx.xxx.xxx"  
提示: 可以使用如下语句获取玩家IP  


```csharp
string ip = e.Player.IP;
```

<details>
<summary>参考答案</summary>

```csharp
using System.Reflection;
using Terraria;
using TerrariaApi.Server;
using TShockAPI;
using TShockAPI.Hooks;


namespace Plugin
{
    [ApiVersion(2, 1)]
    public class Plugin : TerrariaPlugin
    {
        //定义插件的作者名称
        public override string Author => "Cai";

        //插件的一句话描述
        public override string Description => "钩子测试";

        //插件的名称
        public override string Name => "Hooks";

        //插件的版本
        public override Version Version => Assembly.GetExecutingAssembly().GetName().Version;

        //插件的构造器
        public Plugin(Main game) : base(game)
        {
        }

        //插件加载时执行的代码
        public override void Initialize()
        {
            PlayerHooks.PlayerPostLogin += PlayerHooks_PlayerPostLogin;  //注册钩子
        }

        private void PlayerHooks_PlayerPostLogin(PlayerPostLoginEventArgs e)
        {
            e.Player.SendSuccessMessage($"您的登录IP是：{e.Player.IP}");
        }

        //插件卸载时执行的代码
        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                PlayerHooks.PlayerPostLogin -= PlayerHooks_PlayerPostLogin;  //注销钩子
            }
            base.Dispose(disposing);
        }
    }
}
```
</details>


4. 使用TShockAPI钩子实现，在服务器使用`/reload`命令(GeneralHooks.ReloadEvent)后显示"服务器正在重载!"  
> 重点: Reload钩子非常重要，可以让你用`/reload`统一重载插件  

<details>
<summary>参考答案</summary>

```csharp
using System.Reflection;
using Terraria;
using TerrariaApi.Server;
using TShockAPI;
using TShockAPI.Hooks;

namespace Plugin
{
    [ApiVersion(2, 1)]
    public class Plugin : TerrariaPlugin
    {
        //定义插件的作者名称
        public override string Author => "Cai";

        //插件的一句话描述
        public override string Description => "钩子测试";

        //插件的名称
        public override string Name => "Hooks";

        //插件的版本
        public override Version Version => Assembly.GetExecutingAssembly().GetName().Version;

        //插件的构造器
        public Plugin(Main game) : base(game)
        {
        }

        //插件加载时执行的代码
        public override void Initialize()
        {
            GeneralHooks.ReloadEvent += GeneralHooks_ReloadEvent;
        }

        private void GeneralHooks_ReloadEvent(ReloadEventArgs e)
        {
            //这个钩子非常常用，可以让你用/reload统一重载插件
            TSPlayer.All.SendWarningMessage("服务器正在重载!!!"); //这是发送全服广播的另一种方法，TSPlayer.All对象可以代表全服所有玩家
            //上面代码也可以换为
            //TShock.Utils.Broadcast("服务器正在重载!!!",255,135,0);
        }

        //插件卸载时执行的代码
        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                GeneralHooks.ReloadEvent -= GeneralHooks_ReloadEvent;
            }
            base.Dispose(disposing);
        }
    }
}
```
</details>

5. 使用`GetDataHandlers`实现，在玩家打开箱子时(`GetDataHandlers.ChestOpen`)广播"玩家XXX打开了箱子XXX位于\(X, Y\)"  
提示: 用以下代码可以获取箱子对象  



```csharp
int chestIndex = Chest.FindChest(e.X,e.Y); //找到箱子的索引
Chest chest = Main.chest[chestIndex]; //获取箱子对象
//箱子名称: chest.name, 箱子X坐标: chest.x, 箱子Y坐标: chest.y, 箱子物品数组: chest.item
```

<details>
<summary>参考答案</summary>

```csharp
using System.Reflection;
using Terraria;
using TerrariaApi.Server;
using TShockAPI;
using TShockAPI.Hooks;

namespace Plugin
{
    [ApiVersion(2, 1)]
    public class Plugin : TerrariaPlugin
    {
        //定义插件的作者名称
        public override string Author => "Cai";

        //插件的一句话描述
        public override string Description => "钩子测试";

        //插件的名称
        public override string Name => "Hooks";

        //插件的版本
        public override Version Version => Assembly.GetExecutingAssembly().GetName().Version;

        //插件的构造器
        public Plugin(Main game) : base(game)
        {
        }

        //插件加载时执行的代码
        public override void Initialize()
        {
            GetDataHandlers.ChestOpen.Register(ChestOpen); //注册钩子
        }

        private void ChestOpen(object? sender, GetDataHandlers.ChestOpenEventArgs e)
        {
            int chestIndex = Chest.FindChest(e.X,e.Y); //找到箱子的索引
            Chest chest = Main.chest[chestIndex]; //获取箱子对象
            TSPlayer.All.SendWarningMessage($"玩家{e.Player}打开了箱子{chest.name}位于({chest.x}, {chest.y})");
        }

        //插件卸载时执行的代码
        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                GetDataHandlers.ChestOpen.UnRegister(ChestOpen); //注销钩子
            }
            base.Dispose(disposing);
        }
    }
}
```

</details>


6. 使用GetDataHandlers实现，在玩家切换PVP模式时(GetDataHandlers.TogglePvp)发送"本服务器禁止切换PVP模式",并记录日子"玩家XXX试图开启/关闭PVP",并且阻止玩家切换  
提示1: 用以下代码可以记录日志  



```csharp
//只写日志
TShock.Log.Info("信息Log");
TShock.Log.Warn("警告Log");
TShock.Log.Error("错误Log");
TShock.Log.Debug("调试Log，要在Config打开Debug才会生效哦");
//写日志并打印到控制台
TShock.Log.ConsoleInfo("信息Log，会打印在控制台上哦");
TShock.Log.ConsoleWarn("警告Log，会打印在控制台上哦");
TShock.Log.ConsoleError("错误Log，会打印在控制台上哦");
TShock.Log.ConsoleDebug("调试Log，会打印在控制台上哦，要在Config打开Debug才会生效哦");
```

> 提示2: 用以下代码获取玩家想要切换的PVP状态  


```csharp
bool pvp = e.Pvp //切换前状态为!e.pvp
```

> 提示3: 用以下代码切换PVP模式  



```csharp
e.Player.SetPvP(bool);
```

  

<details>
<summary>参考答案</summary>

```csharp
using System.Reflection;
using Terraria;
using TerrariaApi.Server;
using TShockAPI;
using TShockAPI.Hooks;

namespace Plugin
{
    [ApiVersion(2, 1)]
    public class Plugin : TerrariaPlugin
    {
        //定义插件的作者名称
        public override string Author => "Cai";

        //插件的一句话描述
        public override string Description => "钩子测试";

        //插件的名称
        public override string Name => "Hooks";

        //插件的版本
        public override Version Version => Assembly.GetExecutingAssembly().GetName().Version;

        //插件的构造器
        public Plugin(Main game) : base(game)
        {
        }

        //插件加载时执行的代码
        public override void Initialize()
        {
            GetDataHandlers.TogglePvp.Register(TogglePvp); //注册钩子
        }

        private void TogglePvp(object? sender, GetDataHandlers.TogglePvpEventArgs e)
        {
            e.Player.SendErrorMessage("本服务器禁止切换PVP模式"); //给玩家发送消息
            TShock.Log.ConsoleInfo($"玩家{e.Player.Name}试图{(e.Pvp?"打开":"关闭")}PVP"); //输出日志
            e.Player.SetPvP(!e.Pvp); //设置PVP(和e.Pvp相反)
            e.Handled = true; //标记为已处理，即忽略这个事件
        }

        //插件卸载时执行的代码
        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                GetDataHandlers.TogglePvp.UnRegister(TogglePvp); //注销钩子
            }
            base.Dispose(disposing);
        }
    }
}
```
</details>
