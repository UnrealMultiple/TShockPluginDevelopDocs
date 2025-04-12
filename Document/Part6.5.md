# Part 6.5.1 数据包参考表格\(1.4.4.9\) \(by @xuyuwtu\)

### ConnectRequest \[1\]
| Size | Description | Type | Notes |
| ---- | ----------- | ---- | ----- |
| ? | Version | String | "Terraria" + Main.curRelease |
#### Definition
```csharp
[ServerGetOnly]
public struct ConnectRequest
{
    public string Version;
}
```
#### SendData
```csharp
NetMessage.SendData(1);
```

### Disconnect \[2\]
| Size | Description | Type | Notes |
| ---- | ----------- | ---- | ----- |
| ? | Reason | NetworkText | \- |
#### Definition
```csharp
[ClientGetOnly]
public struct Disconnect
{
    public Localization.NetworkText Text;
}
```
#### SendData
```csharp
NetMessage.SendData(2, -1, -1, text);
```

### ContinueConnecting \[3\]
| Size | Description | Type | Notes |
| ---- | ----------- | ---- | ----- |
| 1 | Player ID | Byte | \- |
| 1 | ServerWantsToRunCheckBytesInClientLoopThread | Boolean | NetPlay.Connection.ServerSpecialFlags[2] |
#### Definition
```csharp
[ClientGetOnly]
public struct ContinueConnecting
{
    public byte PlayerID;
    public bool ServerWantsToRunCheckBytesInClientLoopThread = false;
}
```
#### SendData
```csharp
NetMessage.SendData(3, remoteClient);
```

### PlayerInfo \[4\]
| Size | Description | Type | Notes |
| ---- | ----------- | ---- | ----- |
| 1 | Player ID | Byte | \- |
| 1 | Skin Variant | Byte | \- |
| 1 | Hair | Byte | \- |
| ? | Name | String | \- |
| 1 | Hair Dye | Byte | \- |
| 2 | Accessory Visibility | UInt16 | \- |
| 1 | Hide Misc | Byte | \- |
| 3 | Hair Color | RGBColor | \- |
| 3 | Skin Color | RGBColor | \- |
| 3 | Eye Color | RGBColor | \- |
| 3 | Shirt Color | RGBColor | \- |
| 3 | Under Shirt Color | RGBColor | \- |
| 3 | Pants Color | RGBColor | \- |
| 3 | Shoe Color | RGBColor | \- |
| 1 | Flag1 | Byte | BitFlags:<br/> 0 = Softcore<br/> 1 = Mediumcore<br/> 2 = Hardcore<br/> 4 = ExtraAccessory<br/> 8 = Creative |
| 1 | Flag2 | Byte | BitFlags:<br/> 1 = UsingBiomeTorches<br/> 2 = HappyFunTorchTime<br/> 4 = UnlockedBiomeTorches<br/> 8 = UnlockedSuperCart<br/> 16 = EnabledSuperCart |
| 1 | Flag3 | Byte | BitFlags:<br/> 1 = UsedAegisCrystal<br/> 2 = UsedAegisFruit<br/> 4 = UsedArcaneCrystal<br/> 8 = UsedGalaxyPearl<br/> 16 = UsedGummyWorm<br/> 32 = UsedAmbrosia<br/> 64 = AteArtisanBread |
#### Definition
```csharp
[ServerForward]
public struct PlayerInfo
{
    public byte PlayerID;
    public byte SkinVariant;
    public byte Hair;
    public string Name;
    public byte HairDye;
    public ushort AccessoryVisibility;
    public RGBColor HairColor;
    public RGBColor SkinColor; 
    public RGBColor EyeColor;
    public RGBColor ShirtColor;
    public RGBColor UnderShirtColor;
    public RGBColor RantsColor;
    public RGBColor ShoeColor;
    public BFlag1 Flag1;
    public BFlag2 Flag2;
    public BFlag3 Flag3;
    public struct BFlag1
    {
        public BitsByte Data;
        public bool IsSoftcore => Data[0];
        public bool IsMediumcore => Data[1];
        public bool ExtraAccessor => Data[2];
        public bool IsCreative => Data[3];
    }
    public struct BFlag2
    {
        public BitsByte Data;
        public bool UsingBiomeTorches => Data[0];
        public bool HappyFunTorchTime => Data[1];
        public bool UnlockedBiomeTorches => Data[2];
        public bool UnlockedSuperCart => Data[3];
        public bool EnabledSuperCart => Data[4];
    }
    public struct BFlag3
    {
        public BitsByte Data;
        public bool UsedAegisCrystal => Data[0];
        public bool UsedAegisFruit => Data[1];
        public bool UsedArcaneCrystal => Data[2];
        public bool UsedGalaxPearl => Data[3];
        public bool UsedGummyWorm => Data[4];
        public bool UsedAmbrosia => Data[5];
        public bool AteArtisanBread => Data[6];
    }
}
```
#### SendData
```csharp
NetMessage.SendData(4, -1, -1, null, playerID);
```

### PlayerSlot \[5\]
| Size | Description | Type | Notes |
| ---- | ----------- | ---- | ----- |
| 1 | Player ID | Byte | \- |
| 2 | Slot ID | Int16 | \- |
| 2 | Stack | Int16 | \- |
| 1 | Prefix | Byte | \- |
| 2 | Type | Int16 | \- |
#### Definition
```csharp
[ServerForward]
public struct PlayerSlot
{
    public byte PlayerID;
    public short SlotID;
    public short Stack;
    public byte Prefix;
    public short Type;
}
```
#### SendData
```csharp
NetMessage.SendData(5, -1, -1, null, playerID, slotID, prefix);
```

### ContinueConnecting 2 \[6\]
| Size | Description | Type | Notes |
| ---- | ----------- | ---- | ----- |
| - | - | - | - |
#### Definition
```csharp
[ServerGetOnly]
public struct ContinueConnecting2
{
}
```
#### SendData
```csharp
NetMessage.SendData(6);
```

### WorldInfo \[7\]
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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
| Size | Description | Type | Notes |
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

### RGBColor
```csharp
public struct RGBColor
{
    public byte R;
    public byte G;
    public byte B;
}
```
