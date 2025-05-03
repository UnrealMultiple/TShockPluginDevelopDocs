# Part 6.5.1 数据包参考表格\(1.4.4.9\) \(by @xuyuwtu\)

## 数据包结构

| 偏移量(Offset) | 大小(Size) | 描述(Description)          | 类型(Type)       | 说明(Note)                               |
|:---------------:|:-----------:|----------------------------|-------------------|-----------------------------------|
|               0 |           2 | PacketLength(数据包长度)（字节）          | ushort            | -                             |
|               2 |           1 | PacketType(数据包类型)             | byte              | 使用`TSAPI.PacketTypes.PacketName` 枚举值 |
|               3 |           ? | Data(数据内容)                   | ?                 | 具体结构参考后续字段定义           |


### ConnectRequest \[1\]
#### Client -> Server
客户端向服务器发起连接请求
#### 结构
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| ? | Version(客户端版本号) | String | "Terraria" + `Main.curRelease` |
#### GetData
```csharp
using BinaryReader binaryReader = new(new MemoryStream(args.Msg.readBuffer, args.Index, args.Length));
int version = binaryReader.ReadString() //客户端版本
```
#### SendData
| PacketTypes | Text | number | number2 | number3 | number4 | number5 |
| ----------- | ---- | ------------ | ------- | ------- | ------- | ------- |
|    ConnectRequest     |  无   |    无     |    无   |    无   |    无      |  无     |

### Disconnect \[2\]
#### Server -> Client
服务器踢出客户端
#### 结构
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| ? | Reason(踢出理由) | NetworkText | \- |
#### GetData
```csharp
using BinaryReader binaryReader = new(new MemoryStream(args.Msg.readBuffer, args.Index, args.Length));
string kickReasion = NetworkText.Deserialize(binaryReader).ToString(); //踢出理由
```
#### SendData
| PacketTypes | Text | number | number2 | number3 | number4 | number5 |
| ----------- | ---- | ------------ | ------- | ------- | ------- | ------- |
|    Disconnect     |  Reason(踢出理由)   |    无     |    无   |    无   |    无      |  无     |

### ContinueConnecting \[3\]
#### Server -> Client
服务器请求客户端将指定玩家角色的完整数据(包括物品栏、装备、银行等所有物品状态)同步到服务器
#### 结构
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 1 | PlayerID(玩家索引) | Byte | \- |
| 1 | ServerWantsToRunCheckBytesInClientLoopThread(服务端是否要在客户端循环线程中运行CheckBytes) | Boolean | NetPlay.Connection.ServerSpecialFlags[2]、RemoteServer.ServerWantsToRunCheckBytesInClientLoopThread |

#### GetData
```csharp
using BinaryReader binaryReader = new(new MemoryStream(args.Msg.readBuffer, args.Index, args.Length));
byte index = binaryReader.ReadByte(); //玩家索引
bool serverWantsToRunCheckBytesInClientLoopThread = binaryReader.ReadBoolean(); //服务端是否要在客户端循环线程中运行CheckBytes()
```

#### SendData
| PacketTypes | Text | number | number2 | number3 | number4 | number5 |
| ----------- | ---- | ------------ | ------- | ------- | ------- | ------- |
|    ContinueConnecting     |  无   |    无     |    无   |    无   |    无      |  无     |


### PlayerInfo \[4\]
#### Server <-> Client (Sync)
服务器和客户端同步玩家角色信息
#### 结构
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
|------|------|------|------|
| 1 | PlayerID(玩家索引) | Byte | 目标玩家索引 (0-255) |
| 1 | SkinVariant(人物风格) | Byte | 角色基础外观变体 (0-PlayerVariantID.Max) |
| 1 | Hair(发型) | Byte | 发型ID (0-164，超出自动重置) |
| ? | Name(角色名称) | String | 自动Trim()的玩家名称 |
| 1 | HairDye(染发剂) | Byte | 头发染色类型ID |
| 2 | AccessoryVisibility(饰品可见性) | UInt16 | 按位控制饰品显示状态 |
| 1 | HideMisc(杂项隐藏) | Byte | 控制宠物/坐骑等额外显示 |
| 3 | HairColor(发色) | Color | 红(R)绿(G)蓝(B)值 |
| 3 | SkinColor(肤色) | Color | - |
| 3 | EyeColor(眼睛颜色) | Color | - |
| 3 | ShirtColor(上衣颜色) | Color | - |
| 3 | UnderShirtColor(内衣颜色) | Color | - |
| 3 | PantsColor(裤子颜色) | Color | - |
| 3 | ShoeColor(鞋子颜色) | Color | - |
| 1 | DifficultyFlags(难度标记) | Byte | BitFlags(位标记):<br/> 0=Mediumcore(中核)<br/> 1=Hardcore(硬核)<br/> 2=ExtraAccessory(额外饰品栏)<br/> 3=Creative(旅行模式) |
| 1 | TorchFlags(火把标记) | Byte | BitFlags(位标记):<br/> 0=UsingBiomeTorches(启用火把神的恩宠)<br>1=HappyFunTorchTime(火把神事件)<br>2=UnlockedBiomeTorches(解锁火把神的恩宠)<br>3=UnlockedSuperCart(解锁矿车升级包)<br>4=EnabledSuperCart(启用矿车升级包) |
| 1 | ConsumableFlags(消耗品标记) | Byte | BitFlags(位标记):<br/>0=UsedAegisCrystal(活力水晶)<br>1=UsedAegisFruit(神盾果)<br>2=UsedArcaneCrystal(奥术水晶)<br>3=UsedGalaxyPearl(星系珍珠)<br>4=UsedGummyWorm(黏性蠕虫)<br>5=UsedAmbrosia(仙馔密酒)<br>6=AteArtisanBread(工匠面包) |

> [!NOTE]
> 1. 所有标记字段均为1字节(8位)长度  
> 2. 每个位标记对应一个布尔值状态  

#### GetData
```csharp
using BinaryReader binaryReader = new(new MemoryStream(args.Msg.readBuffer, args.Index, args.Length));

// 基础玩家信息
byte playerID = binaryReader.ReadByte(); //玩家索引
byte skinVariant = binaryReader.ReadByte(); //人物风格
byte hair = binaryReader.ReadByte(); //发型
string name = binaryReader.ReadString().Trim(); //角色名称
byte hairDye = binaryReader.ReadByte(); //染发剂

// 可见性标记
ushort accessoryVisibility = binaryReader.ReadUInt16(); //饰品可见性
byte hideMisc = binaryReader.ReadByte(); //杂项隐藏

// 颜色信息
Color hairColor = binaryReader.ReadRGB(); //发色
Color skinColor = binaryReader.ReadRGB(); //肤色
Color eyeColor = binaryReader.ReadRGB(); //眼睛颜色
Color shirtColor = binaryReader.ReadRGB(); //上衣颜色
Color underShirtColor = binaryReader.ReadRGB(); //内衣颜色
Color pantsColor = binaryReader.ReadRGB(); //裤子颜色
Color shoeColor = binaryReader.ReadRGB(); //鞋子颜色

// 难度和功能标记
BitsByte difficultyFlags = binaryReader.ReadByte(); //难度标记
byte difficulty = 0; // 0=软核
if (difficultyFlags[0]) difficulty = 1; //中核
if (difficultyFlags[1]) difficulty = 2; //硬核
if (difficultyFlags[3]) difficulty = 3; //旅行
bool extraAccessory = difficultyFlags[2]; //额外饰品栏

// 火把系统标记
BitsByte torchFlags = binaryReader.ReadByte(); //火把标记
bool usingBiomeTorches = torchFlags[0]; //启用火把神的恩宠
bool happyFunTorchTime = torchFlags[1]; //火把神事件
bool unlockedBiomeTorches = torchFlags[2]; //已解锁火把神的恩宠
bool unlockedSuperCart = torchFlags[3]; //解锁矿车升级包
bool enabledSuperCart = torchFlags[4]; //启用矿车升级包

// 消耗品标记
BitsByte consumableFlags = binaryReader.ReadByte(); //消耗品标记
bool usedAegisCrystal = consumableFlags[0]; //使用活力水晶
bool usedAegisFruit = consumableFlags[1]; //使用神盾果
bool usedArcaneCrystal = consumableFlags[2]; //使用奥术水晶
bool usedGalaxyPearl = consumableFlags[3]; //使用星系珍珠
bool usedGummyWorm = consumableFlags[4]; //使用黏性蠕虫
bool usedAmbrosia = consumableFlags[5]; //使用仙馔密酒
bool ateArtisanBread = consumableFlags[6]; //食用工匠面包
```

#### SendData
| PacketTypes | Text | number | number2 | number3 | number4 | number5 |
| ----------- | ---- | ------------ | ------- | ------- | ------- | ------- |
|    PlayerInfo     |  无   |    Index(玩家索引)     |    无   |    无   |    无      |  无     |


### PlayerSlot \[5\]
#### Server <-> Client (Sync)
服务器和客户端同步玩家的库存(包括背包、猪猪储钱罐、保险箱等)的每格物品的信息
#### 结构
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 1 | PlayerID(玩家索引) | Byte | \- |
| 2 | SlotID(格位ID) | Int16 | \- |
| 2 | Stack(数量) | Int16 | \- |
| 1 | Prefix(前缀ID) | Byte | \- |
| 2 | Type(物品ID) | Int16 | \- |

#### GetData
```csharp
using BinaryReader binaryReader = new(new MemoryStream(args.Msg.readBuffer, args.Index, args.Length));
short index = this.reader.ReadInt16(); //玩家索引
short slot = (int) this.reader.ReadInt16(); //格位ID
byte prefix = (int) this.reader.ReadByte(); //物品前缀
short type = (int) this.reader.ReadInt16(); //物品ID
```

#### SendData
| PacketTypes | Text | number | number2 | number3 | number4 | number5 |
| ----------- | ---- | ------------ | ------- | ------- | ------- | ------- |
|    PlayerSlot     |  无     |  PlayerID(玩家索引)   |    SlotID(格位ID)     |    Prefix(前缀ID)   |    无   |    无      |


### ContinueConnecting 2 \[6\]
#### Client -> Server
客户端向服务器请求世界信息，此数据包不携带如何数据，仅表示一个信号
#### 结构
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| - | - | - | - |

#### GetData
```csharp
没有可以读取的数据
```

#### SendData
| PacketTypes | Text | number | number2 | number3 | number4 | number5 |
| ----------- | ---- | ------------ | ------- | ------- | ------- | ------- |
|    ContinueConnecting2     |  无     |  无   |    无    |   无   |    无   |    无      |


### WorldInfo \[7\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 4 | Time | Int32 | \- |
| 1 | Flag1 | Byte | BitFlags:<br/> 1 = Day Time<br/> 2 = Blood Moon<br/> 4 = Eclipse |
| 1 | Moon Phase | Byte | \- |
| 2 | Max Tile X | Int16 | \- |
| 2 | Max Tile X | Int16 | \- |
| 2 | Spawn Tile X | Int16 | \- |
| 2 | Spawn Tile Y | Int16 | \- |
| 2 | World Surface | Int16 | \- |
| 2 | Rock Layer | Int16 | \- |
| 4 | Wolrd ID | Int32 | \- |
| ? | Wolrd Name | String | \- |
| 16 | Unique ID | Byte\[16\] | \- |
| 1 | Game Mode | Byte | \- |
| 8 | World Generator Version | UInt64 | \- |
| 1 | Moon Type | Byte | \- |
| 1 | Tree Background 1 | Byte | \- |
| 1 | Tree Background 2 | Byte | \- |
| 1 | Tree Background 3 | Byte | \- |
| 1 | Tree Background 4 | Byte | \- |
| 1 | Corrupt Background | Byte | \- |
| 1 | Jungle Background | Byte | \- |
| 1 | Snow Background | Byte | \- |
| 1 | Hallowd Background | Byte | \- |
| 1 | Cirmson Background | Byte | \- |
| 1 | Desert Background | Byte | \- |
| 1 | Ocean Background | Byte | \- |
| 1 | Mushroom Background | Byte | \- |
| 1 | Underworld Background | Byte | \- |
| 1 | Ice Background Style | Byte | \- |
| 1 | Jungle Background Style | Byte | \- |
| 1 | Hell Background Style | Byte | \- |
| 4 | Wind Speed | Single | \- |
| 1 | Cloud Number | Byte | \- |
| 12 | Tree X | Int32\[3\] | \- |
| 4 | Tree Style X | Byte\[4\] | \- |
| 12 | Cave Back X | Int32\[3\] | \- |
| 4 | Cave Back Style | Byte\[4\] | \- |
| 13 | Tree Tops | Byte\[13\] | Forest1<br/> Forest2<br/> Forest3<br/> Forest4<br/> Corruption<br/> Jungle<br/> Snow<br/> Hallow<br/> Crimson<br/> Desert<br/> Ocean<br/> Glowing Mushroom<br/> Underworld |
| 4 | Rain | Single | \- |
| 1 | Flag2 | Byte | BitFlags:<br/> 1 = Shadow Orb Smashed<br/> 2 = Downed Boss 1<br/> 4 = Downed Boss 2<br/> 8 = Downed Boss 3<br/> 16 = Hard Mode<br/> 32 = Downed Clown<br/> 64 = Server Side Character<br/> 128 = Downed Plant Boss |
| 1 | Flag3 | Byte | BitFlags:<br/> 1 = Mech Boss Downed 1<br/> 2 = Mech Boss Downed 2<br/> 4 = Mech Boss Downed 3<br/> 8 = Mech Boss Any Downed<br/> 16 = Cloud BG Active<br/> 32 = Crimson<br/> 64 = Pumpkin Moon<br/> 128 = Snow Moon |
| 1 | Flag4 | Byte | BitFlags:<br/> 1 = None<br/> 2 = FastForwardTime<br/> 4 = Slime Rain<br/> 8 = Downed Slime King<br/> 16 = Downed Queen Bee<br/> 32 = Downed Fishron<br/> 64 = Downed Martians<br/> 128 = Downed Ancient Cultist |
| 1 | Flag5 | Byte | BitFlags:<br/> 1 = Downed Moon Lord<br/> 2 = Downed Pumking<br/> 4 = Downed Mourning Wood<br/> 8 = Downed Ice Queen<br/> 16 = Downed Santank<br/> 32 = Downed Everscream<br/> 64 = Downed Golem<br/> 128 = Birthday Party |
| 1 | Flag6 | Byte | BitFlags:<br/> 1 = Downed Pirates<br/> 2 = Downed Frost Legion<br/> 4 = Downed Goblins<br/> 8 = Sandstorm<br/> 16 = DD2 Event<br/> 32 = Downed DD2 Tier 1<br/> 64 = Downed DD2 Tier 2<br/> 128 = Downed DD2 Tier 3 |
| 1 | Flag7 | Byte | BitFlags:<br/> 1 = Combat Book Used<br/> 2 = Manual Lanterns<br/> 4 = Downed Solar Tower<br/> 8 = Downed Vortex Tower<br/> 16 = Downed Tower Nebula<br/> 32 = Downed Stardust Tower<br/> 64 = Force Halloween (day)<br/> 128 = Force XMas (day) |
| 1 | Flag8 | Byte | BitFlags:<br/> 1 = Bought Cat<br/> 2 = Bought Dog<br/> 4 = Bought Bunny<br/> 8 = Free Cake<br/> 16 = Drunk World<br/> 32 = Downed Empress of Light<br/> 64 = Downed Queen Slime<br/> 128 = GetGoodWorld |
| 1 | Flag9 | Byte | BitFlags:<br/> 1 = Tenth Anniversar World<br/> 2 = Dont Starve World<br/> 4 = Downed Deerclops<br/> 8 = Not The Bees World<br/> 16 = Remix World<br/> 32 = Unlocked Slime Blue Spawn<br/> 64 = Combat Book Volume Two Was Used<br/> 128 = Peddlers Satchel Was Used |
| 1 | Flag10 | Byte | BitFlgs:<br/> 1 = Unlocked Slime Green Spawn<br/> 2 = Unlocked Slime Old Spawn<br/> 4 = Unlocked Slime Purple Spawn<br/> 8 = Unlocked Slime Rainbow Spawn<br/> 16 = Unlocked Slime Red Spawn<br/> 32 = Unlocked Slime Yellow Spawn<br/> 64 = Unlocked SlimeCopper Spawn<br/> 128 = Fast Forward Time To Dusk |
| 1 | Flag11 | Byte | 1 = NoTrapsWorld<br/> 2 = ZenithWorld<br/> 4 = Unlocked Truffle Spawn<br/> |
| 2 | Copper Ore Tier | Int16 | \- |
| 2 | Iron Ore Tier | Int16 | \- |
| 2 | Silver Ore Tier | Int16 | \- |
| 2 | Gold Ore Tier | Int16 | \- |
| 2 | Cobalt Ore Tier | Int16 | \- |
| 2 | Mythril Ore Tier | Int16 | \- |
| 2 | Adamantite Ore Tier | Int16 | \- |
| 1 | Invasion Type | SByte | \- |
| 8 | Lobby ID | UInt64	| \- |
| 4 | Sandstorm Severity | Single | \- |
#### Definition
```csharp
[ClientGetOnly]
public unsafe struct WorldInfo
{
    public int Time;
    public BitsByte Flag1;
    public byte MoonPhase;
    public short MaxTileX;
    public short MaxTileY;
    public short SpawnTileX;
    public short SpawnTileY;
    public short WorldSurface;
    public short RockLayer;
    public int WolrID;
    public string WorldName;
    public byte GameMode;
    public fixed byte UniqueID[16];
    public ulong WorldGeneratorVersion;
    public byte MoonType;
    public byte TreeBG1;
    public byte TreeBG2;
    public byte TreeBG3;
    public byte TreeBG4;
    public byte CorruptBG;
    public byte JungleBG;
    public byte SnowBG;
    public byte HallowBG;
    public byte CirmsonBG;
    public byte DesertBG;
    public byte OceanBG;
    public byte MushroomBG;
    public byte UnderworldBG;
    public byte IceBackStyle;
    public byte JungleBackStyle;
    public byte HellBackStyle;
    public float WindSpeedTarget;
    public byte NumClouds;
    public fixed int TreeX[3];
    public fixed byte TreeStyle[4];
    public fixed int CaveBackX[3];
    public fixed byte CaveBackStyle[4];
    public fixed byte TreeTops[13];
    public float MaxRaining;
    public BitsByte Flag2;
    public BitsByte Flag3;
    public BitsByte Flag4;
    public BitsByte Flag5;
    public BitsByte Flag6;
    public BitsByte Flag7;
    public BitsByte Flag8;
    public BitsByte Flag9;
    public BitsByte Flag10;
    public BitsByte Flag11;
    public byte SundialCooldown;
    public byte MoondialCooldown;
    public short CopperTier;
    public short IronTier;
    public short SilverTier;
    public short GoldTier;
    public short CobaltTier;
    public short MythrilTier;
    public short AdamantiteTier;
    public sbyte InvasionType;
    public ulong LobbyID;
    public float IntendedServerity;
}
```
#### SendData
```csharp
NetMessage.SendData(7);
```

### TileGetSection \[8\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 4 | X | Int32 | Player Spawn X |
| 4 | Y | Int32 | Player Spawn Y |
#### Definition
```csharp
[ServerGetOnly]
public struct TileGetSection
{
    public int TileX;
    public int TileY;
}
```
#### SendData
```csharp
NetMessage.SendData(8, -1, -1, null, tileX, tileY)
```

### Status \[9\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 4 | Status Max | Int32 | \- |
| ? | Status Text | NetworkText | \- |
| 1 | Status Text Flags | Byte | BitFlags:<br> 1 = HideStatusTextPercent<br> 2 = StatusTextHasShadows |
#### Definition
```csharp
[ClientGetOnly]
public struct Status
{
    public int StatusMax;
    public Localization.NetworkText Text;
    public BitsByte Flag;
}
```
#### SendData
```csharp
NetMessage.SendData(9, -1, -1, null, statusMax, flag)
```

### TileSendSection \[10\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 1 | Compressed | Boolean | \- |
| 4 | X Start | Int32 | \- |
| 4 | Y Start | Int32 | \- |
| 2 | Width | Int16 | \- |
| 2 | Height | Int16 | \- |
| ? | Tiles | \- |  |
| 2 | Chest Count | Int16 | \- |
| ? | Chests | \- |  |
| 2 | Sign Count | Int16 | \- |
| ? | Signs | \- |  |
| 2 | TileEntity Count | Int16 | \- |
| ? | TileEntities | \- | \- |
#### Definition
```csharp
[ClientGetOnly]
public struct TileSendSection
{
    public bool Compressed;
    public int XStart;
    public int YStart;
    public short Width;
    public short Height;
    public byte[] TilesData;
    public short ChestCount;
    public ChestData[] Chests;
    public short SignCount;
    public SignData[] Sings;
    public short TileEntityCount;
    public byte[] TileEntitys;
    public struct ChestData
    {
        public short ChestID;
        public short X;
        public short Y;
        public string Name;
    }
    public struct SignData
    {
        public short SignID;
        public short X;
        public short Y;
        public string Text;
    }
}
```
#### SendData
```csharp
NetMessage.SendData(10, -1, -1, null, xStart, yStart, width, height)
```

### SectionTileFrame \[11\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 2 | Start X | Int16 | - |
| 2 | Start Y | Int16 | - |
| 2 | End X | Int16 | - |
| 2 | End Y | Int16 | - |
#### Definition
```csharp
[ClientGetOnly]
public struct SectionTileFrame
{
    public short StartX;
    public short StartY;
    public short EndX;
    public short EndY;
}
```
#### SendData
```csharp
NetMessage.SendData(11, -1, -1, null, startX, startY, endX, endY)
```

### PlayerSpawn \[12\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 1 | PlayerID | Byte | - |
| 2 | Spawn X | Int16 | - |
| 2 | Spawn Y | Int16 | - |
| 4 | Respawn Time | Int32 | - |
| 2 | Number Of Deaths PVE | Int16 | - |
| 2 | Number Of Deaths PVP | Int16 | - |
| 1 | Player Spawn Context | Byte | 0 = ReviveFromDeath<br/> 1 = SpawningIntoWorld<br/> 2 = RecallFromItem |
```csharp
[ClientGetOnly]
public struct PlayerSpawn
{
    public byte PlayerID;
    public short SpawnX;
    public short SpawnY;
    public int RespownTimer;
    public short NumberOfDeathsPVE;
    public short NumberOfDeathsPVP;
    public byte PlayerSpawnContext;
}
```
#### SendData
```csharp
NetMessage.SendData(12, -1, -1, null, playerID, playerSpawnContext)
```

### PlayerUpdate \[13\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 1 | PlayerID | Byte | - |
| 1 | Flag1 | BitsByte | - |
| 1 | Flag2 | BitsByte | - |
| 1 | Flag3 | BitsByte | - |
| 1 | Flag4 | BitsByte | - |
| 1 | SelectedItem | Byte | - |
| 8 | Position | Vector2 | - |
| 8 | Velocity | Vector2 | - |
| 8 | PotionOfReturnOriginalUsePosition | Vector2 | - |
| 8 | PotionOfReturnHomePosition | Vector2 | - |
#### Definition
```csharp
[ServerForward]
public struct PlayerUpdate
{
    public byte PlayerID;
    public BitsByte Flag1;
    public BitsByte Flag2;
    public BitsByte Flag3;
    public BitsByte Flag4;
    public byte SelectedItem;
    public Vector2 Position;
    public Vector2 Velocity;
    public Vector2 PotionOfReturnOriginalUsePosition;
    public Vector2 PotionOfReturnHomePosition; 
}    
```
#### SendData
```csharp
NetMessage.SendData(13, -1, -1, null, playerID);
```

### PlayerActive \[14\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 1 | PlayerID | Byte | - |
| 1 | Active | Boolean | - |
#### Definition
```csharp
[ClientGetOnly]
public struct PlayerActive
{
    public byte PlayerID;
    public byte Active;
}    
```
#### SendData
```csharp
NetMessage.SendData(14, -1, -1, null, playerID, active);
```

### PlayerHp \[16\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 1 | PlayerID | Byte | - |
| 2 | StatLife | Int16 | - |
| 2 | StatLifeMax | Int16 | - |
#### Definition
```csharp
[ServerForward]
public struct PlayerHp
{
    public byte PlayerID;
    public short StatLife;
    public short StatLifeMax;
}    
```
#### SendData
```csharp
NetMessage.SendData(16, -1, -1, null, playerID);
```

### Tile \[17\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 1 | Action | Byte | - |
| 2 | TileX | Int16 | - |
| 2 | TileY | Int16 | - |
| 2 | Flag1 | Int16 | - |
| 1 | Flag2 | Byte | - |
#### Definition
```csharp
[ServerForward]
public struct Tile
{
    public byte Action;
    public short TileX;
    public short TileY;
    public short Flag1;
    public byte Flag2;
}    
```
#### SendData
```csharp
NetMessage.SendData(17, -1, -1, null, action, tileX, tileY, flag1, flag2);
```

### TimeSet \[18\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 1 | DayTime | Boolean | - |
| 4 | Time | Int32 | - |
| 2 | SunModY | Int16 | - |
| 2 | MoonModY | Int16 | - |
#### Definition
```csharp
[ClientGetOnly]
public struct TimeSet
{
    public bool DayTime;
    public int Time;
    public short SunModY;
    public short MoonModY;
}    
```
#### SendData
```csharp
NetMessage.SendData(18);
```

### DoorUse \[19\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 1 | Action | Byte | - |
| 2 | TileX | Int16 | - |
| 2 | TileY | Int16 | - |
| 1 | Direction | Byte | - |
#### Definition
```csharp
[ServerForward]
public struct DoorUse
{
    public byte Action;
    public short TileX;
    public short TileY;
    public byte Direction;
}    
```
#### SendData
```csharp
NetMessage.SendData(19, -1, -1, null, action, tileX, tileY, direction);
```

### TileSendSquare \[20\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 2 | TileX | Int16 | - |
| 2 | TileY | Int16 | - |
| 1 | Width | Byte | - |
| 1 | Height | Byte | - |
| 1 | TileChangeType | Byte | - |
| ? | Data | Byte[] | - |
#### Definition
```csharp
[ServerForward]
public struct TileSendSquare
{
    public short TileX;
    public short TileY;
    public byte Width;
    public byte Height;
    public byte TileChangeType;
    public byte[] Data;
}    
```
#### SendData
```csharp
NetMessage.SendData(20, -1, -1, null, tileX, tileY, width, height, tileChangeType);
```

### ItemDrop \[21\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ItemDrop
{
    public short ItemID;
    public Vector2 Position;
    public Vector2 Velocity;
    public short Stack;
    public byte Prefix;
    public byte OwnIgnore;
    public short NetID;
}    
```
#### SendData
```csharp
NetMessage.SendData(21, -1, -1, null, itemID, ownIgnore);
```

### ItemOwner \[22\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ItemOwner
{
    public short NPCID;
    public byte PlayerIndex;
}    
```
#### SendData
```csharp
NetMessage.SendData(22, -1, -1, null, npcID);
```

### NpcUpdate \[23\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct NpcUpdate
{
    public short NPCID;
    public Vector2 Position;
    public Vector2 Velocity;
    public ushort Target;
    public BitsFlag Flag1;
    public BitsFlag Flag2;
    public float[] AI;
    public short NetID;
    public byte PlayerCountForMultiplayerDifficultyOverride;
    public float StrengthMultiplier;
    public byte LifeSize;
    public int Life; //int or short or sbyte
    public byte ReleaseOwner;
}    
```
#### SendData
```csharp
NetMessage.SendData(23, -1, -1, null, npcID);
```

### NpcItemStrike \[24\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct NpcItemStrike
{
    public short NPCID;
    public byte PlayerID;
}    
```
#### SendData
```csharp
NetMessage.SendData(24, -1, -1, null, npcID, playerID);
```

### ProjectileNew \[27\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ProjectileNew
{
    public short Identiry;
    public Vector2 Position;
    public Vector2 Velocity;
    public byte Owner;
    public short Type;
    public BitsByte Flag1;
    public BitsByte Flag2;
    public float AI0;
    public float AI1;
    public ushort BannerIdToRespondTo;
    public short Damage;
    public float KnockBack;
    public short originalDamage;
    public short ProjUUID;
    public float AI2;
}    
```
#### SendData
```csharp
NetMessage.SendData(27);
```

### NpcStrike \[28\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct NpcStrike
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(28);
```

### ProjectileDestroy \[29\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ProjectileDestroy
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(29);
```

### TogglePvp \[30\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct TogglePvp
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(30);
```

### ChestGetContents \[31\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ChestGetContents
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(31);
```

### ChestItem \[32\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ChestItem
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(32);
```

### ChestOpen \[33\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ChestOpen
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(33);
```

### PlaceChest \[34\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlaceChest
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(34);
```

### EffectHeal \[35\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct EffectHeal
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(35);
```

### Zones \[36\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct Zones
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(36);
```

### PasswordRequired \[37\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PasswordRequired
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(37);
```

### PasswordSend \[38\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PasswordSend
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(38);
```

### RemoveItemOwner \[39\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct RemoveItemOwner
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(39);
```

### NpcTalk \[40\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct NpcTalk
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(40);
```

### PlayerAnimation \[41\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlayerAnimation
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(41);
```

### PlayerMana \[42\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlayerMana
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(42);
```

### EffectMana \[43\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct EffectMana
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(43);
```

### PlayerTeam \[45\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlayerTeam
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(45);
```

### SignRead \[46\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SignRead
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(46);
```

### SignNew \[47\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SignNew
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(47);
```

### LiquidSet \[48\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct LiquidSet
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(48);
```

### PlayerSpawnSelf \[49\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlayerSpawnSelf
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(49);
```

### PlayerBuff \[50\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlayerBuff
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(50);
```

### NpcSpecial \[51\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct NpcSpecial
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(51);
```

### ChestUnlock \[52\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ChestUnlock
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(52);
```

### NpcAddBuff \[53\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct NpcAddBuff
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(53);
```

### NpcUpdateBuff \[54\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct NpcUpdateBuff
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(54);
```

### PlayerAddBuff \[55\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlayerAddBuff
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(55);
```

### UpdateNPCName \[56\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct UpdateNPCName
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(56);
```

### UpdateGoodEvil \[57\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct UpdateGoodEvil
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(57);
```

### PlayHarp \[58\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlayHarp
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(58);
```

### HitSwitch \[59\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct HitSwitch
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(59);
```

### UpdateNPCHome \[60\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct UpdateNPCHome
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(60);
```

### SpawnBossorInvasion \[61\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SpawnBossorInvasion
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(61);
```

### PlayerDodge \[62\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlayerDodge
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(62);
```

### PaintTile \[63\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PaintTile
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(63);
```

### PaintWall \[64\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PaintWall
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(64);
```

### Teleport \[65\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct Teleport
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(65);
```

### PlayerHealOther \[66\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlayerHealOther
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(66);
```

### Placeholder \[67\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct Placeholder
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(67);
```

### ClientUUID \[68\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ClientUUID
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(68);
```

### ChestName \[69\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ChestName
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(69);
```

### CatchNPC \[70\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct CatchNPC
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(70);
```

### ReleaseNPC \[71\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ReleaseNPC
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(71);
```

### TravellingMerchantInventory \[72\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct TravellingMerchantInventory
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(72);
```

### TeleportationPotion \[73\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct TeleportationPotion
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(73);
```

### AnglerQuest \[74\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct AnglerQuest
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(74);
```

### CompleteAnglerQuest \[75\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct CompleteAnglerQuest
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(75);
```

### NumberOfAnglerQuestsCompleted \[76\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct NumberOfAnglerQuestsCompleted
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(76);
```

### CreateTemporaryAnimation \[77\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct CreateTemporaryAnimation
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(77);
```

### ReportInvasionProgress \[78\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ReportInvasionProgress
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(78);
```

### PlaceObject \[79\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlaceObject
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(79);
```

### SyncPlayerChestIndex \[80\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SyncPlayerChestIndex
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(80);
```

### CreateCombatText \[81\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct CreateCombatText
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(81);
```

### LoadNetModule \[82\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct LoadNetModule
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(82);
```

### NpcKillCount \[83\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct NpcKillCount
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(83);
```

### PlayerStealth \[84\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlayerStealth
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(84);
```

### ForceItemIntoNearestChest \[85\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ForceItemIntoNearestChest
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(85);
```

### UpdateTileEntity \[86\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct UpdateTileEntity
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(86);
```

### PlaceTileEntity \[87\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlaceTileEntity
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(87);
```

### TweakItem \[88\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct TweakItem
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(88);
```

### PlaceItemFrame \[89\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlaceItemFrame
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(89);
```

### UpdateItemDrop \[90\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct UpdateItemDrop
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(90);
```

### EmoteBubble \[91\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct EmoteBubble
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(91);
```

### SyncExtraValue \[92\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SyncExtraValue
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(92);
```

### SocialHandshake \[93\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SocialHandshake
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(93);
```

### Deprecated \[94\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct Deprecated
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(94);
```

### KillPortal \[95\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct KillPortal
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(95);
```

### PlayerTeleportPortal \[96\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlayerTeleportPortal
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(96);
```

### NotifyPlayerNpcKilled \[97\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct NotifyPlayerNpcKilled
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(97);
```

### NotifyPlayerOfEvent \[98\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct NotifyPlayerOfEvent
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(98);
```

### UpdateMinionTarget \[99\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct UpdateMinionTarget
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(99);
```

### NpcTeleportPortal \[100\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct NpcTeleportPortal
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(100);
```

### UpdateShieldStrengths \[101\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct UpdateShieldStrengths
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(101);
```

### NebulaLevelUp \[102\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct NebulaLevelUp
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(102);
```

### MoonLordCountdown \[103\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct MoonLordCountdown
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(103);
```

### NpcShopItem \[104\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct NpcShopItem
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(104);
```

### GemLockToggle \[105\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct GemLockToggle
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(105);
```

### PoofOfSmoke \[106\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PoofOfSmoke
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(106);
```

### SmartTextMessage \[107\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SmartTextMessage
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(107);
```

### WiredCannonShot \[108\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct WiredCannonShot
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(108);
```

### MassWireOperation \[109\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct MassWireOperation
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(109);
```

### MassWireOperationPay \[110\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct MassWireOperationPay
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(110);
```

### ToggleParty \[111\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ToggleParty
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(111);
```

### TreeGrowFX \[112\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct TreeGrowFX
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(112);
```

### CrystalInvasionStart \[113\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct CrystalInvasionStart
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(113);
```

### CrystalInvasionWipeAll \[114\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct CrystalInvasionWipeAll
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(114);
```

### MinionAttackTargetUpdate \[115\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct MinionAttackTargetUpdate
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(115);
```

### CrystalInvasionSendWaitTime \[116\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct CrystalInvasionSendWaitTime
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(116);
```

### PlayerHurtV2 \[117\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlayerHurtV2
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(117);
```

### PlayerDeathV2 \[118\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlayerDeathV2
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(118);
```

### CreateCombatTextExtended \[119\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct CreateCombatTextExtended
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(119);
```

### Emoji \[120\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct Emoji
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(120);
```

### TileEntityDisplayDollItemSync \[121\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct TileEntityDisplayDollItemSync
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(121);
```

### RequestTileEntityInteraction \[122\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct RequestTileEntityInteraction
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(122);
```

### WeaponsRackTryPlacing \[123\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct WeaponsRackTryPlacing
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(123);
```

### TileEntityHatRackItemSync \[124\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct TileEntityHatRackItemSync
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(124);
```

### SyncTilePicking \[125\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SyncTilePicking
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(125);
```

### SyncRevengeMarker \[126\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SyncRevengeMarker
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(126);
```

### RemoveRevengeMarker \[127\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct RemoveRevengeMarker
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(127);
```

### LandGolfBallInCup \[128\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct LandGolfBallInCup
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(128);
```

### FinishedConnectingToServer \[129\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct FinishedConnectingToServer
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(129);
```

### FishOutNPC \[130\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct FishOutNPC
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(130);
```

### TamperWithNPC \[131\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct TamperWithNPC
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(131);
```

### PlayLegacySound \[132\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct PlayLegacySound
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(132);
```

### FoodPlatterTryPlacing \[133\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct FoodPlatterTryPlacing
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(133);
```

### UpdatePlayerLuckFactors \[134\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct UpdatePlayerLuckFactors
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(134);
```

### DeadPlayer \[135\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct DeadPlayer
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(135);
```

### SyncCavernMonsterType \[136\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SyncCavernMonsterType
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(136);
```

### RequestNPCBuffRemoval \[137\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct RequestNPCBuffRemoval
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(137);
```

### ClientSyncedInventory \[138\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ClientSyncedInventory
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(138);
```

### SetCountsAsHostForGameplay \[139\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SetCountsAsHostForGameplay
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(139);
```

### SetMiscEventValues \[140\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SetMiscEventValues
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(140);
```

### RequestLucyPopup \[141\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct RequestLucyPopup
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(141);
```

### SyncProjectileTrackers \[142\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SyncProjectileTrackers
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(142);
```

### CrystalInvasionRequestedToSkipWaitTime \[143\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct CrystalInvasionRequestedToSkipWaitTime
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(143);
```

### RequestQuestEffect \[144\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct RequestQuestEffect
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(144);
```

### SyncItemsWithShimmer \[145\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SyncItemsWithShimmer
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(145);
```

### ShimmerActions \[146\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct ShimmerActions
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(146);
```

### SyncLoadout \[147\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SyncLoadout
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(147);
```

### SyncItemCannotBeTakenByEnemies \[148\]
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
#### Definition
```csharp
public struct SyncItemCannotBeTakenByEnemies
{
    
}    
```
#### SendData
```csharp
NetMessage.SendData(148);
```

### Color
```csharp
public struct Color
{
    public byte R;
    public byte G;
    public byte B;
}
```
