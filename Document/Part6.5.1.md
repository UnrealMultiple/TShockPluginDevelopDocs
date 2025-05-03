# Part 6.5.1 数据包参考表格\(1.4.4.9\) \(by [@xuyuwtu](https://github.com/xuyuwtu) & [@ACaiCat](https://github.com/ACaiCat)\)

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
#### Server -> Client (Sync)  
服务器向客户端同步世界信息  
#### 结构  
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |  
|------|------|------|------|  
| 4 | Time(游戏时间) | Int32 | 当前世界时间（tick数） |  
| 1 | WorldFlags1(世界标记1) | Byte | BitFlags(位标记):<br>0=IsDayTime(白天)<br>1=IsBloodMoon(血月)<br>2=IsEclipse(日食) |  
| 1 | MoonPhase(月相) | Byte | 当前月相 (0-7) |  
| 2 | MaxTilesX(最大宽度) | Int16 | 世界最大X坐标（格数） |  
| 2 | MaxTilesY(最大高度) | Int16 | 世界最大Y坐标（格数） |  
| 2 | SpawnX(出生点X) | Int16 | 世界出生点X坐标 |  
| 2 | SpawnY(出生点Y) | Int16 | 世界出生点Y坐标 |  
| 2 | WorldSurface(地表高度) | Int16 | 地表层高度 |  
| 2 | RockLayer(岩石层高度) | Int16 | 岩石层高度 |  
| 4 | WorldID(世界ID) | Int32 | 世界唯一标识符 |  
| ? | WorldName(世界名称) | String | 世界名称字符串 |  
| 1 | GameMode(游戏模式) | Byte | 0=经典,1=专家,2=大师,3=旅行 |  
| 16 | UniqueID(唯一ID) | Byte[16] | 世界文件唯一标识 |  
| 8 | WorldVersion(世界版本) | UInt64 | 世界生成器版本号 |  
| 1 | MoonType(月亮类型) | Byte | 月亮外观类型 |  
| 1 | TreeBG1(树木背景1) | Byte | 第一类树木背景样式 |  
| 1 | TreeBG2(树木背景2) | Byte | 第二类树木背景样式 |  
| 1 | TreeBG3(树木背景3) | Byte | 第三类树木背景样式 |  
| 1 | TreeBG4(树木背景4) | Byte | 第四类树木背景样式 |  
| 1 | CorruptBG(腐化背景) | Byte | 腐化之地背景样式 |  
| 1 | JungleBG(丛林背景) | Byte | 丛林背景样式 |  
| 1 | SnowBG(雪地背景) | Byte | 雪地背景样式 |  
| 1 | HallowBG(神圣背景) | Byte | 神圣之地背景样式 |  
| 1 | CrimsonBG(猩红背景) | Byte | 猩红之地背景样式 |  
| 1 | DesertBG(沙漠背景) | Byte | 沙漠背景样式 |  
| 1 | OceanBG(海洋背景) | Byte | 海洋背景样式 |  
| 1 | MushroomBG(蘑菇背景) | Byte | 蘑菇地背景样式 |  
| 1 | UnderworldBG(地狱背景) | Byte | 地狱背景样式 |  
| 1 | IceBackStyle(冰雪背景) | Byte | 冰雪背景样式 |  
| 1 | JungleBackStyle(丛林背景) | Byte | 丛林背景样式 |  
| 1 | HellBackStyle(地狱背景) | Byte | 地狱背景样式 |  
| 4 | WindSpeed(风速) | Single | 当前风速 |  
| 1 | CloudCount(云层数量) | Byte | 当前云层数量 |  
| 12 | TreeX(特殊树木X坐标) | Int32[3] | 3棵特殊树木的X坐标 |  
| 4 | TreeStyle(树木样式) | Byte[4] | 4种树木样式 |  
| 12 | CaveBackX(洞穴背景X坐标) | Int32[3] | 洞穴背景X坐标 |  
| 4 | CaveBackStyle(洞穴背景样式) | Byte[4] | 洞穴背景样式 |  
| 13 | TreeTops(树冠样式) | Byte[13] | 13种树冠样式标记 |  
| 4 | MaxRaining(降雨强度) | Single | 当前降雨强度(0.0-1.0) |  
| 1 | WorldFlags2(世界标记2) | Byte | BitFlags(位标记):<br>0=ShadowOrbSmashed(暗影珠被破坏)<br>1=DownedEyeOfCthulhu(克苏鲁之眼)<br>2=DownedEaterOfWorlds(世界吞噬者)<br>3=DownedSkeletron(骷髅王)<br>4=HardMode(困难模式)<br>5=DownedClown(小丑)<br>6=ServerSideCharacter(服务器角色)<br>7=DownedPlantera(世纪之花) |  
| 1 | WorldFlags3(世界标记3) | Byte | BitFlags(位标记):<br>0=DownedMechBoss1(毁灭者)<br>1=DownedMechBoss2(双子魔眼)<br>2=DownedMechBoss3(机械骷髅王)<br>3=DownedAnyMechBoss(任意机械Boss)<br>4=CloudBGActive(云层背景激活)<br>5=IsCrimson(猩红世界)<br>6=IsPumpkinMoon(南瓜月事件)<br>7=IsSnowMoon(霜月事件) |  
| 1 | WorldFlags4(世界标记4) | Byte | BitFlags(位标记):<br>1=FastForwardTime(时间加速)<br>2=IsSlimeRain(史莱姆雨)<br>3=DownedKingSlime(史莱姆王)<br>4=DownedQueenBee(蜂后)<br>5=DownedDukeFishron(猪龙鱼公爵)<br>6=DownedMartians(火星人入侵)<br>7=DownedAncientCultist(远古教徒) |  
| 1 | WorldFlags5(世界标记5) | Byte | BitFlags(位标记):<br>0=DownedMoonLord(月亮领主)<br>1=DownedPumpking(南瓜王)<br>2=DownedMourningWood(哀木)<br>3=DownedIceQueen(冰雪女王)<br>4=DownedSantank(圣诞坦克)<br>5=DownedEverscream(常绿尖叫怪)<br>6=DownedGolem(石巨人)<br>7=IsBirthdayParty(生日派对) |  
| 1 | WorldFlags6(世界标记6) | Byte | BitFlags(位标记):<br>0=DownedPirates(海盗入侵)<br>1=DownedFrostLegion(霜月军团)<br>2=DownedGoblins(哥布林入侵)<br>3=IsSandstorm(沙尘暴)<br>4=IsDD2Event(撒旦军队事件)<br>5=DownedDD2T1(撒旦军队T1)<br>6=DownedDD2T2(撒旦军队T2)<br>7=DownedDD2T3(撒旦军队T3) |  
| 1 | WorldFlags7(世界标记7) | Byte | BitFlags(位标记):<br>0=CombatBookUsed(战斗手册使用)<br>1=IsLanternNight(灯笼夜)<br>2=DownedSolarTower(日耀塔)<br>3=DownedVortexTower(星璇塔)<br>4=DownedNebulaTower(星云塔)<br>5=DownedStardustTower(星尘塔)<br>6=ForceHalloween(强制万圣节)<br>7=ForceChristmas(强制圣诞节) |  
| 1 | WorldFlags8(世界标记8) | Byte | BitFlags(位标记):<br>0=BoughtCat(购买猫)<br>1=BoughtDog(购买狗)<br>2=BoughtBunny(购买兔)<br>3=FreeCake(免费蛋糕)<br>4=IsDrunkWorld(醉酒世界)<br>5=DownedEmpressOfLight(光之女皇)<br>6=DownedQueenSlime(史莱姆皇后)<br>7=IsGetGoodWorld(终极世界种子) |  
| 1 | WorldFlags9(世界标记9) | Byte | BitFlags(位标记):<br>0=IsTenthAnniversary(十周年世界)<br>1=IsDontStarve(饥荒世界)<br>2=DownedDeerclops(鹿角怪)<br>3=IsNotTheBees(Not the Bees世界)<br>4=IsRemixWorld(混搭世界)<br>5=UnlockedBlueSlime(解锁蓝史莱姆)<br>6=CombatBookVolumeTwo(战斗手册第二卷)<br>7=PeddlersSatchel(小贩的背包) |  
| 1 | WorldFlags10(世界标记10) | Byte | BitFlags(位标记):<br>0=UnlockedGreenSlime(解锁绿史莱姆)<br>1=UnlockedOldSlime(解锁老史莱姆)<br>2=UnlockedPurpleSlime(解锁紫史莱姆)<br>3=UnlockedRainbowSlime(解锁彩虹史莱姆)<br>4=UnlockedRedSlime(解锁红史莱姆)<br>5=UnlockedYellowSlime(解锁黄史莱姆)<br>6=UnlockedCopperSlime(解锁铜史莱姆)<br>7=FastForwardToDusk(加速至黄昏) |  
| 1 | WorldFlags11(世界标记11) | Byte | BitFlags(位标记):<br>0=IsNoTrapsWorld(无陷阱世界)<br>1=IsZenithWorld(天顶世界)<br>2=UnlockedTruffle(解锁松露人) |  
| 1 | SundialCooldown(日晷冷却) | Byte | 日晷剩余冷却时间 |  
| 1 | MoondialCooldown(月晷冷却) | Byte | 月晷剩余冷却时间 |  
| 2 | CopperTier(铜矿层级) | Int16 | 铜矿替代矿类型 |  
| 2 | IronTier(铁矿层级) | Int16 | 铁矿替代矿类型 |  
| 2 | SilverTier(银矿层级) | Int16 | 银矿替代矿类型 |  
| 2 | GoldTier(金矿层级) | Int16 | 金矿替代矿类型 |  
| 2 | CobaltTier(钴矿层级) | Int16 | 钴矿替代矿类型 |  
| 2 | MythrilTier(秘银层级) | Int16 | 秘银替代矿类型 |  
| 2 | AdamantiteTier(精金层级) | Int16 | 精金替代矿类型 |  
| 1 | InvasionType(入侵类型) | SByte | 当前入侵事件类型 |  
| 8 | LobbyID(大厅ID) | UInt64 | 多人游戏大厅ID |  
| 4 | SandstormSeverity(沙尘暴强度) | Single | 当前沙尘暴强度 |

#### GetData
```csharp
using BinaryReader binaryReader = new(new MemoryStream(args.Msg.readBuffer, args.Index, args.Length));

// 基础世界信息
int time = binaryReader.ReadInt32(); // 游戏时间(tick)
BitsByte worldFlags1 = binaryReader.ReadByte(); // 世界标记1
bool isDayTime = worldFlags1[0]; // 是否白天
bool isBloodMoon = worldFlags1[1]; // 是否血月
bool isEclipse = worldFlags1[2]; // 是否日食
byte moonPhase = binaryReader.ReadByte(); // 月相(0-7)

// 世界尺寸
short maxTilesX = binaryReader.ReadInt16(); // 世界最大宽度
short maxTilesY = binaryReader.ReadInt16(); // 世界最大高度

// 出生点坐标
short spawnX = binaryReader.ReadInt16(); // 出生点X
short spawnY = binaryReader.ReadInt16(); // 出生点Y

// 世界层高度
short worldSurface = binaryReader.ReadInt16(); // 地表高度
short rockLayer = binaryReader.ReadInt16(); // 岩石层高度

// 世界标识
int worldID = binaryReader.ReadInt32(); // 世界ID
string worldName = binaryReader.ReadString(); // 世界名称
byte gameMode = binaryReader.ReadByte(); // 游戏模式(0-3)

// 世界唯一标识
Guid worldGuid = new Guid(binaryReader.ReadBytes(16)); // 世界唯一ID
ulong worldVersion = binaryReader.ReadUInt64(); // 世界版本

// 天体信息
byte moonType = binaryReader.ReadByte(); // 月亮类型

// 背景样式
byte[] bgStyles = new byte[13];
for(int i=0; i<13; i++){
    bgStyles[i] = binaryReader.ReadByte(); // 13种背景样式
}

// 风速和云层
float windSpeed = binaryReader.ReadSingle(); // 风速
byte cloudCount = binaryReader.ReadByte(); // 云层数量

// 特殊坐标点
int[] treeX = new int[3];
for(int i=0; i<3; i++){
    treeX[i] = binaryReader.ReadInt32(); // 特殊树木坐标
}

// 树木样式
byte[] treeStyle = new byte[4];
for(int i=0; i<4; i++){
    treeStyle[i] = binaryReader.ReadByte(); // 4种树木样式
}

// 降雨信息
float rainIntensity = binaryReader.ReadSingle(); // 降雨强度
bool isRaining = rainIntensity > 0f; // 是否正在下雨

// 读取所有世界状态标记(共13个字节)
BitsByte[] worldStateFlags = new BitsByte[13];
for(int i=0; i<13; i++){
    worldStateFlags[i] = binaryReader.ReadByte();
}

// 其他世界设置
byte sundialCooldown = binaryReader.ReadByte(); // 日晷冷却
byte moondialCooldown = binaryReader.ReadByte(); // 月晷冷却

// 矿石层级
short[] oreTiers = new short[7];
for(int i=0; i<7; i++){
    oreTiers[i] = binaryReader.ReadInt16(); // 7种矿石类型
}

// 多人游戏信息
sbyte invasionType = binaryReader.ReadSByte(); // 入侵事件类型
ulong lobbyID = binaryReader.ReadUInt64(); // 大厅ID
float sandstormSeverity = binaryReader.ReadSingle(); // 沙尘暴强度
```

#### SendData
| PacketTypes | Text | number | number2 | number3 | number4 | number5 |
| ----------- | ---- | ------------ | ------- | ------- | ------- | ------- |
|  WorldInfo     |  无     |  无   |    无    |   无   |    无   |    无      |

### TileGetSection \[8\]
#### Client -> Server
玩家加入服务器，客户端向服务器请求区块，服务器会发送玩家生成位置的区块，图鉴信息，旅行模式能力设置，晶塔信息等数据
> [!NOTE]
> - 无论X,Y的值，服务器一定会发送世界出生点的区块
> - 当玩家在服务器中已设置出生点时，服务器还会发送玩家出生点的区块信息
> - 当玩家在服务器中没有设置出生点时，X=-1且Y=-1，此时服务器只会发送世界出生点的区块

#### 结构  
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
| 4 | X(玩家出生点TileX坐标) | Int32 | - |
| 4 | Y(玩家出生点TileY坐标) | Int32 | - |

#### GetData
```csharp
using BinaryReader binaryReader = new(new MemoryStream(args.Msg.readBuffer, args.Index, args.Length));
int x = this.reader.ReadInt32(); //玩家出生点TileX坐标
int y = this.reader.ReadInt32(); //玩家出生点TileY坐标
```

#### SendData
| PacketTypes | Text | number | number2 | number3 | number4 | number5 |
| ----------- | ---- | ------------ | ------- | ------- | ------- | ------- |
|  TileGetSection     | 无 | PlayerSpawnX(玩家出生点TileX坐标)     |  PlayerSpawnY(玩家出生点TileY坐标)   |   无   |    无   |    无      |


### Status \[9\]
#### Server -> Client
主要用于服务器向客户端同步长时间任务的执行进度和状态提示(例如进服时接收区块)，也可以用来做计分板
#### 结构  
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |  
|------|------|------|------|  
| 4 | StatusMax(总进度) | Int32 | 任务总进度值（如区块同步总数） |  
| ? | StatusText(状态文本) | NetworkText | 显示的状态消息（如"Receiving tile data"） |  
| 1 | StatusFlags(状态标记) | Byte | BitFlags(位标记):<br> 0=HidePercent(隐藏百分比)<br> 1=TextShadow(文本阴影) |  

#### GetData
```csharp
using BinaryReader binaryReader = new(new MemoryStream(args.Msg.readBuffer, args.Index, args.Length));

// 读取进度数据
int statusMax = binaryReader.ReadInt32();             // StatusMax(总进度值)
string statusText = NetworkText.Deserialize(binaryReader).ToString(); // StatusText(状态文本)

// 读取状态标记位
BitsByte statusFlags = binaryReader.ReadByte();       // StatusFlags(状态标记)
bool hidePercent = statusFlags[0];                    // 位0: 是否隐藏百分比
bool hasTextShadow = statusFlags[1];                  // 位1: 是否显示文本阴影
```

#### SendData
| PacketTypes | Text | number | number2 | number3 | number4 | number5 |
| ----------- | ---- | ------------ | ------- | ------- | ------- | ------- |
|  Status     | StatusText(状态文本) | StatusMax(总进度)     |  StatusFlags(状态标记)   |   无   |    无   |    无      |


| StatusFlags(状态标记)  | 位标记状态 (`[HidePercent, TextShadow]`) | 说明 |
|-----------------------|-----------------------------------------|------|
| `0`    | `[false, false]` | 显示百分比 + 无阴影 |
| `1`    | `[true, false]`  | 隐藏百分比 + 无阴影 |
| `2`    | `[false, true]`  | 显示百分比 + 有阴影 |
| `3`    | `[true, true]`   | 隐藏百分比 + 有阴影 |


### TileSendSection \[10\]
#### Server -> Client
服务器向客户端同步区块数据
#### 结构  
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |  
|------|------|------|------|  
| 1 | IsCompressed(是否压缩) | Boolean | 数据是否使用压缩格式 |  
| 4 | XStart(起始TileX坐标) | Int32 | 区块左上角世界TileX坐标 |  
| 4 | YStart(起始TileY坐标) | Int32 | 区块左上角世界TileY坐标 |  
| 2 | Width(宽度) | Int16 | 区块横向格数 |  
| 2 | Height(高度) | Int16 | 区块纵向格数 |  
| ? | Tiles(图格数据) | Byte[] | (压缩)图格数组，每个图格包含:<br> 类型(Type)<br> 样式(Style)<br> 液体(Liquid)<br> 电线(Wire) |  
| 2 | ChestCount(箱子数量) | Int16 | 区块内包含的箱子数 |  
| ? | Chests(箱子数据) | Byte[] | 每个箱子包含:<br> X/Y坐标<br> 物品列表 |  
| 2 | SignCount(标牌数量) | Int16 | 区块内包含的标牌数 |  
| ? | Signs(标牌数据) | Byte[] | 每个标牌包含:<br> X/Y坐标<br> 文本内容 |  
| 2 | TileEntityCount(实体数量) | Int16 | 区块内特殊实体数 |  
| ? | TileEntities(实体数据) | Byte[] | 包含:<br> 逻辑传感器<br> 物品框架<br> 训练假人等 |  

#### GetData
不写详细的解包逻辑，想看请反编译`NetMessage.DecompressTileBlock`
```csharp
using BinaryReader binaryReader = new(new MemoryStream(args.Msg.readBuffer, args.Index, args.Length));
NetMessage.DecompressTileBlock(binaryReader.BaseStream); //读取、解压区块数据并且同步到游戏中
```
#### SendData
| PacketTypes | Text | number | number2 | number3 | number4 | number5 |
| ----------- | ---- | ------------ | ------- | ------- | ------- | ------- |
|  TileSendSection| 无 | xStart(区块起始TileX坐标)     |  yStart(区块起始TileY坐标)   |   width(区块宽度)   |    height(无区块高度)   |    无      |

> [!NOTE]
> - 泰拉瑞亚的图格坐标起点是地图左上角(0,0)
> - 区块由起始位置向右下角延申



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
#### Client -> Server
客户端同步解锁或上锁箱子、解锁丛林门
#### 结构
| 大小(Size) | 描述(Description) | 类型(Type) | 说明(Note) |
| ---- | ----------- | ---- | ----- |
|1	|Type(操作类型) | Byte|	Value(值): <br> 1 = ChestUnlock(解锁箱子)<br> 2 = DoorUnlock(解锁神庙大门)<br> 3 = ChestLock(给箱子上锁) |
|2	|X(目标TileX坐标)	|Int16|	-|
|2	|Y(目标TileY坐标)	|Int16|	-|

#### GetData
```csharp
using BinaryReader binaryReader = new(new MemoryStream(args.Msg.readBuffer, args.Index, args.Length));
byte lockActionType = binaryReader.ReadByte();  //操作类型：1-解锁箱子，2-解锁门，3-上锁箱子
short tileX = binaryReader.ReadInt16();  //目标的TileX坐标
short tileY = binaryReader.ReadInt16();  //目标的TileY坐标
```
#### SendData
| PacketTypes | Text | number | number2 | number3 | number4 | number5 |
| ----------- | ---- | ------------ | ------- | ------- | ------- | ------- |
|   ChestUnlock  |  无   |    无     |   Type(操作类型)    |    X(目标TileX坐标)   |    Y(目标TileY坐标)      |  无     |

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
