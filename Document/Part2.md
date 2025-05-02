# Part 2.初步了解TShock插件模板​

本章你将学到：  

- TShock插件模板各部分的作用
- TShock插件的初始化顺序

## 文件结构
```
📂MyFirstPlugin
├── 📂MyFirstPlugin //项目文件夹
│   ├── 📂bin //构建产物
│   │   └── 📂Debug //Debug编译模式下的构建产物
│   │       └── 📂net6.0 //目标.NET版本，
│   │           ├── 📜Myfirstplugin.deps.json //依赖清单的文件，用不到
│   │           ├── 📜Myfirstplugin.dll //插件本体
│   │           └── 📜Myfirstplugin.pdb //和插件一起安装，报错后可以显示相关方法在源码中的行数
|   |       📂Release //Release编译模式下的构建产物
│   │       └── 📂net6.0
│   │           ├── 📜Myfirstplugin.deps.json //同上
│   │           ├── 📜Myfirstplugin.dll //同上
│   │           └── 📜Myfirstplugin.pdb //同上
│   ├── 📂obj //用于存放编译的临时文件，
│   ├── 📜Myfirstplugin.csproj //项目配置文件，这个文件的名字决定插件程序集的名字
│   ├── 📜Plugin.cs //插件本体源码文件
│   ├── 📜Config.cs //插件配置相关源码文件，按需添加
│   └── 📜Database.cs //插件数据库相关源码文件，按需添加
└── 📜MyFirstPlugin.sln //项目的解决方案，用来存放项目组的配置
```

## 插件模板​
以下是一个常用的TShock插件模板
```csharp
using System.Reflection;
using Terraria;
using TerrariaApi.Server;
using TShockAPI;

namespace MyFirstPlugin;

[ApiVersion(2, 1)]
public class Plugin : TerrariaPlugin 
{
    
    //插件的名称
    public override string Name => "MyFirstPlugin";
    
    //插件作者名字
    public override string Author => "Cai";
    
    //插件功能的一句话描述
    public override string Description => "我也不知道这个有什么用"; 

    //插件的版本, 可以采用日期来命名, eg: 2024年1月4日第1个版本 -> new(2024, 1,4,1)
    public override Version Version => new(2024, 1,4,1);
    
    //插件的构造器
    public Plugin(Main game) : base(game) 
    {
    }
    
    //插件加载时执行的代码
    public override void Initialize()
    {
        
    }
        
    //插件卸载时执行的代码
    protected override void Dispose(bool disposing)
    {
        if (disposing)
        {
        }
        
        base.Dispose(disposing);
    }
}
```


## 插件基本信息​
以下代码可以设置插件的一些基本信息，插件名建议按照功能命名，不要起奇怪名字(例如"炸裂"), Author(插件作者)、Name(插件名字)、Version(插件版本)将会在插件初始化完成时显示  
![image](https://github.com/user-attachments/assets/be546207-ea31-43c9-b26e-4114c8d9ed51)
```csharp
//插件的名称
public override string Name => "MyFirstPlugin";

//插件作者名字
public override string Author => "Cai";

//插件功能的一句话描述
public override string Description => "我也不知道这个有什么用"; 

//插件的版本, 可以采用日期来命名, eg: 2024年1月4日第1个版本 -> new(2024, 1,4,1)
public override Version Version => new(2024, 1,4,1);
```


> [!NOTE]
> 使用`new Version()`将会显示此方法参数为版本号  
> 而使用`Assembly.GetExecutingAssembly().GetName().Version`获取的是程序集的版本号, 程序集版本号需要在项目属性中修改  
> ```csharp
> //插件的版本
> public override Version Version => new Version(2024, 1,4,1); //代码中设置版本 (推荐)
> public override Version Version =>
> Assembly.GetExecutingAssembly().GetName().Version; //项目中设置版本 (麻烦)
> ```


<details>
  <summary>修改程序集版本</summary>
    1. 右键资源管理器中的项目(这里是Myfirstplugin),然后点击属性
        <img src="https://github.com/user-attachments/assets/41c63f6d-92a9-4ec8-a7b5-4f4723b38feb" alt="img"><br>
    2. 修改程序集版本即可
        <img src="https://github.com/user-attachments/assets/e8fa0c71-122e-454e-bfa1-bb9007301105" alt="img"><br>
      
</details>



## 插件的构造器​

插件的构造器，也叫构造函数。当TShock创造插件对象时，构造函数就会被调用，构造函数中的代码会比`Initialize`方法更先执行。服务端会先按照读取到插件的顺序(取决于插件的文件名)执行所有插件的构造函数，再按照`Order`由小到大执行插件的`Initialize`方法 

```csharp
//插件的构造器
public Plugin(Main game) : base(game)
{
    /*
    插件加载顺序，默认为1，数字越小越先加载
    其中int.MaxValue最晚加载, int.MinValue最先加载
    TShockAPI的Order为0, 所以Order < 0会比TShock更优先加载, Order > 0,则会比TShock更晚加载
    如果你不是指Order, 那么Order的默认值为1,会比TShock晚加载
    */
    base.Order = 1;
}
```
> [!CAUTION]
> 通常情况下，构造函数的执行会在TShock初始化之前
> 如果在构造函数中编写代码(例如建表)，可能会导致十分玄学的问题，所以没有特殊需要下一般插件的构造函数都会直接放空。
> 如果没有特殊需要也不要修改`Order`

## 插件的初始化方法

初始化方法是TShock插件中非常重要的部分，这个函数主要负责插件的`初始化`，可以用来`注册钩子`、`添加命令`、`注册REST API命令`、`初始化配置文件`、`初始化数据库`等  

```csharp
//插件加载时执行的代码
public override void Initialize()
{
    ServerApi.Hooks.ServerChat.Register(this, this.OnChat); //注册钩子
    Commands.ChatCommands.Add(new Command("xsb.fishrank", Fish, "钓鱼排行")); //添加命令
    TShock.RestApi.Register(new SecureRestCommand("/XSB/GetMap", GetMap, "rest.xsb.admin")); //注册REST API命令
    Config.Instance.Read(); //初始化配置文件
    DB.Init(); //初始化数据库
}
```

## 插件的卸载方法
插件的卸载方法在插件卸载(通常是TShock关闭时)被调用。如果你使用`PluginLoader`, 那么请你务必写好卸载方法, 否则会导致插件的功能卸载不完全, 重载出现异常。
```csharp
//插件卸载时执行的代码
protected override void Dispose(bool disposing)
{
     if (disposing)
     {
         ServerApi.Hooks.ServerChat.Deregister(this, this.OnChat);  //卸载钩子
         Commands.ChatCommands.RemoveAll(c => c.CommandDelegate.Method?.DeclaringType?.Assembly == Assembly.GetExecutingAssembly()); //移除插件添加的命令 (这个可以卸载所有命令,推荐每个插件都使用)
     }
     base.Dispose(disposing);
}
```
> [!IMPORTANT]  
> 注册、注销有始有终  
> ```csharp
> //注册 Initialize
> Commands.ChatCommands.Add(new Command("xsb.fishrank", Fish, "钓鱼排行"));
> ServerApi.Hooks.ServerChat.Register(this, this.OnChat); 
> GetDataHandlers.KillMe.Register(OnKillMe);
> PlayerHooks.PlayerPermission+= PlayerHooksOnPlayerPermission;
> ...
> //注销 Dispose
> Commands.ChatCommands.RemoveAll(c => c.CommandDelegate.Method?.DeclaringType?.Assembly == Assembly.GetExecutingAssembly());
> ServerApi.Hooks.ServerChat.Deregister(this, this.OnChat); 
> GetDataHandlers.KillMe.UnRegister(OnKillMe);
> PlayerHooks.PlayerPermission-= PlayerHooksOnPlayerPermission;
> ```
