# Part 6.1 读取数据包 (超难理解)

本章你将学到：  

- 如何解包数据包
- 如何使用`ServerApi.Hooks.NetGetData`


## 解析数据包结构
### 数据包文档
你可以在[Part6.5.1 数据包参考表格](Part6.5.1.md)找到相关数据包结构(目前表格未完善)

### 反编译`MessageBuffer.GetData`
假设我们要写个插件，全服广播打开上锁箱子的玩家，并且显示箱子的坐标和物品
1. 在随便一个方法里面加一句`MessageBuffer.GetData`  
2. 右键GetData，选择`转到`，点击`前往声明或用法` (当然你可以直接按`Ctrl+B`)  
  ![PixPin_2025-05-02_21-55-55](https://github.com/user-attachments/assets/3a804874-be89-4f96-839f-542283ca81e5)  

3. 找到你需要的数据包，你可以在数据包文档找，也可以在`PacketTypes`里面找
   例如：我如果要处理箱子解锁相关的，我就在文档或者`PacketTypes`找相关的，最后发现有个数据包`ChestUnlock = 52`，ChestUnlock意味解锁箱子，说明52号数据包就是解锁箱子相关的。
4. 在`MessageBuffer.GetData`定位相关的处理逻辑
   1. 首先我们要找到`switch (packetId)`，然后下面有一堆的case
   2. 找到我们需要的case，例如：`case 52`
   ![PixPin_2025-05-02_22-28-29](https://github.com/user-attachments/assets/b5ce9abd-08a7-462c-bafb-96072f2ce29e)

5. 分析代码
   #### Reader
   - 在分析数据包之前，简单介绍一下读取数据包的工具Reader
   - Reader意为阅读器，可以帮助我们读取数据包的每个字段。`MessageBuffer.GetData`中的`this.reader`在跑到下面这段代码时已经处理过包头的`长度`字段和`包号`字段了，也就意味着我们从这里开始读取的就是`数据内容`。
   - Reader会顺序读取每一个字段。例如我们在上一章提到的`TileGetSection(客户端请求获取区块) [8]`，结构差不多是这样的：
     ```csharp
      public struct TileGetSection
      {
        public int TileX;
        public int TileY;
      }
     ```
     我们使用下面的代码来读取：
     ```csharp
      int x = this.reader.ReadInt32(); //先读取第一个int，即TileX
      int y = this.reader.ReadInt32(); //读取第二个int，即TileY
     ```
      > 每次使用reader.Read...()读取数据后,`this.reader.BaseStream.Position`都会移动到这个字段之后，即下一个字段的开头，你可以把`this.reader.BaseStream.Position`理解为reader的指针，每次读完字段指针就会移动到下一个字段的开头(前提是你读取的字段类型是正确的)
   #### 初步分析
   ```csharp
    case 52: 
          int number2_7 = (int) this.reader.ReadByte(); //这里读取了个byte - number2_7 啥？
          int num65 = (int) this.reader.ReadInt16(); //这里读取了个short - num65 啥啥？
          int num66 = (int) this.reader.ReadInt16(); //这里读取了个short - num66 啥啥啥？
          if (number2_7 == 1) //难以理解...
          {
            Chest.Unlock(num65, num66);
            if (Main.netMode == 2)
            {
              NetMessage.TrySendData(52, ignoreClient: this.whoAmI, number2: (float) number2_7, number3: (float) num65, number4: (float) num66);
              NetMessage.SendTileSquare(-1, num65, num66, 2);
            }
          }
          if (number2_7 == 2)
          {
            WorldGen.UnlockDoor(num65, num66);
            if (Main.netMode == 2)
            {
              NetMessage.TrySendData(52, ignoreClient: this.whoAmI, number2: (float) number2_7, number3: (float) num65, number4: (float) num66);
              NetMessage.SendTileSquare(-1, num65, num66, 2);
            }
          }
          if (number2_7 != 3)
            break;
          Chest.Lock(num65, num66);
          if (Main.netMode != 2)
            break;
          NetMessage.TrySendData(52, ignoreClient: this.whoAmI, number2: (float) number2_7, number3: (float) num65, number4: (float) num66);
          NetMessage.SendTileSquare(-1, num65, num66, 2);
          break;
   ```
   `ChestUnlock`并不像`TileGetSection`那样有清晰的结构，我们不知道`number2_7`，`num65`，`num66`表示什么，如果你完全是入门小白，你遇到这样的反编译代码绝对会一脸懵逼，这时我们就需要`NetMessage.orig_SendData`辅助判断(不一定有用)
   
### 反编译NetMessage.orig_SendData
1. 在随便一个方法里面加一句`NetMessage.orig_SendData`  
2. 右键GetData，选择`转到`，点击`前往声明或用法` (当然你可以直接按`Ctrl+B`)  
  ![PixPin_2025-05-02_22-35-11](https://github.com/user-attachments/assets/c4a39d2c-029b-4228-bb91-c4d3ffcd5801)
3. 找到你需要的数据包，例如我们需要的52号
4. 在`NetMessage.orig_SendData`定位相关的处理逻辑
   1. 首先我们要找到`switch (msgType)`，然后下面有一堆的case
   2. 找到我们需要的case，例如：`case 52`
   ![PixPin_2025-05-02_22-37-36](https://github.com/user-attachments/assets/829e4485-6b8b-4900-ac7e-59fb3afd6ff2)
5. 初步分析
   ```csharp
   case 52:
        packetWriter.Write((byte) number2); //写入number2
        packetWriter.Write((short) number3); //写入number3
        packetWriter.Write((short) number4); //写入number4
        break;
   ```
   呃，貌似还是得不到有用的信息
### 偷看文档
这里有两个可选的文档
1. [Part6.5.1 数据包参考表格](Part6.5.1.md)
 - 优点: 大多是最新的结构
 - 缺点: 覆盖不完整(等待填充)
 例如: 没有收录52
3. TShock官方文档：[https://tshock.readme.io/docs/multiplayer-packet-structureC](https://tshock.readme.io/docs/multiplayer-packet-structure)
 - 优点: 覆盖全
 - 缺点: 大多过期
 例如:  
 ![image](https://github.com/user-attachments/assets/995f037b-874c-45a9-ba2c-bc0f958c98c0)


   
### 终极の大招
我们直接将`MessageBuffer.GetData`得到的代码喂给DeepSeek解析
Q: ...(代码) 重命名上述变量，并且附上每行的注释
A:
```csharp
case 52:  // 处理箱子/门锁操作的数据包
    int lockActionType = (int)this.reader.ReadByte();  // 读取操作类型：1-解锁箱子，2-解锁门，3-上锁箱子
    int tileX = (int)this.reader.ReadInt16();  // 读取操作位置的X坐标
    int tileY = (int)this.reader.ReadInt16();  // 读取操作位置的Y坐标
    
    if (lockActionType == 1)  // 如果是解锁箱子操作
    {
        Chest.Unlock(tileX, tileY);  // 执行解锁箱子
        if (Main.netMode == 2)  // 如果是服务器模式
        {
            // 向所有客户端广播解锁箱子消息
            NetMessage.TrySendData(52, ignoreClient: this.whoAmI, number2: (float)lockActionType, number3: (float)tileX, number4: (float)tileY);
            // 更新周围2格范围的方块状态
            NetMessage.SendTileSquare(-1, tileX, tileY, 2);
        }
    }
    
    if (lockActionType == 2)  // 如果是解锁门操作
    {
        WorldGen.UnlockDoor(tileX, tileY);  // 执行解锁门
        if (Main.netMode == 2)  // 如果是服务器模式
        {
            // 向所有客户端广播解锁门消息
            NetMessage.TrySendData(52, ignoreClient: this.whoAmI, number2: (float)lockActionType, number3: (float)tileX, number4: (float)tileY);
            // 更新周围2格范围的方块状态
            NetMessage.SendTileSquare(-1, tileX, tileY, 2);
        }
    }
    
    if (lockActionType != 3)  // 如果不是上锁箱子操作则退出
        break;
        
    Chest.Lock(tileX, tileY);  // 执行上锁箱子
    if (Main.netMode != 2)  // 如果不是服务器模式则退出
        break;
        
    // 向所有客户端广播上锁箱子消息
    NetMessage.TrySendData(52, ignoreClient: this.whoAmI, number2: (float)lockActionType, number3: (float)tileX, number4: (float)tileY);
    // 更新周围2格范围的方块状态
    NetMessage.SendTileSquare(-1, tileX, tileY, 2);
    break;
```
> [!NOTE]
> 关于Main.netMode, 你可以查看文档: [网络模式](Part6.5.2.md#网络模式)

可以看到解析结果几乎都正确，当然如果你是TMODL或者插件的老开发者不需要借助DeepSeek也能看出来这是干啥的.jpg。  
一套连招之后，我们可以知道数据包的结构了 (过程是最重要的，所有数据包解包思路都是这样的)
```csharp
int lockActionType = (int)this.reader.ReadByte();  // 操作类型：1-解锁箱子，2-解锁门，3-上锁箱子
int tileX = (int)this.reader.ReadInt16();  // 目标的X坐标
int tileY = (int)this.reader.ReadInt16();  // 目标的Y坐标
```

## `ServerApi.Hooks.NetGetData`
### 注册、注销、处理三件套 (不多说了)
```csharp
//插件加载时执行的代码
public override void Initialize()
{
    ServerApi.Hooks.NetGetData.Register(this, OnGetData);
}

private void OnGetData(GetDataEventArgs args)
{
}

//插件卸载时执行的代码
protected override void Dispose(bool disposing)
{
    if (disposing)
    {
        ServerApi.Hooks.NetGetData.Deregister(this, OnGetData);
    }
    base.Dispose(disposing);
}
```
### 处理方法
```csharp
private void OnGetData(GetDataEventArgs args)
{
    if (args.MsgID == PacketTypes.ChestUnlock)
    {
        using BinaryReader binaryReader = new(new MemoryStream(args.Msg.readBuffer, args.Index, args.Length)); //新建一个binaryReader
        var plr = TShock.Players[args.Msg.whoAmI]; //获取玩家实例
        if (plr == null) //判断玩家是否为空 (超重要，别忘了哦)
        {
            return;
        }
        
        int lockActionType = binaryReader.ReadByte();  //操作类型：1-解锁箱子，2-解锁门，3-上锁箱子
        
        if (lockActionType != 1) //我们只处理解锁箱子
        {
            return;
        }


        int tileX = binaryReader.ReadInt16();  //目标的X坐标
        int tileY = binaryReader.ReadInt16();  //目标的Y坐标
        
        var chestId = Chest.FindChest(tileX, tileY); //获取箱子索引
        if (chestId == -1) //当找不到箱子则直接结束方法
        {
            return;
        }

        var chest = Main.chest[chestId]; //获取箱子实例
        
        var itemTags = new List<string>(); //定义一个列表存放物品Tag (eg. [i:4322])
        foreach (var i in chest.item)
        {
            if (i.netID == 0) //跳过ID为0的空格子
            {
                continue; 
            }
            itemTags.Add(TShock.Utils.ItemTag(i)); //将物品Tag加到列表
        }
        //发送广播
        TShock.Utils.Broadcast($"玩家{plr.Name}解锁了一个箱子在({tileX},{tileY}): {string.Join(',',itemTags)}",255,255,255); 
    }
}
```
这样我们的`ServerApi.Hooks.NetGetData`钩子就大功告成了！
效果:  
![image](https://github.com/user-attachments/assets/5a479b6e-dc76-44df-bab7-823814a9cc36)

### 处理多个数据包
我们可以使用`switch-case`处理
```csharp
private void OnGetData(GetDataEventArgs args)
{
    switch (args.MsgID)
    {
        case PacketTypes.ChestOpen:
            ...
            break;
        case PacketTypes.ChestItem:
            ...
            break;
        case PacketTypes.ChestName:
            ...
            break;
    }
}
```
## 小结
通过以上的例子，我们需要学会如何在插件开发中去利用反编译来获取数据包的结构，这对于新人来说很难，但这对插件开发非常重要。
> [!IMPORTANT]
> 如果你需要的数据包有GetDataHandlers，那么请使用`GetDataHandlers`提供的对应钩子，不要造轮子遭罪...
