# Part 1.编写你的第一个插件​

本章你将学到：  

- 安装TShock插件模板
- 使用`Rider`创建插件项目
- 初步了解编写一个插件的流程

## 1.安装TShock插件模板 (by @xuyuwtu)

1. 打开一个`PowerShell`  
![image](https://github.com/user-attachments/assets/661cb09b-1095-42d9-b6b1-3318f7d3a9a9)
2. 复制并且运行以下命令
```
dotnet new install TShock.Plugin.ProjectTemplates  
```
看到如下提示即安装成功  
![image](https://github.com/user-attachments/assets/afa2727e-e754-4443-8e94-d55ea86c7cd8)
## 2.新建TShock插件项目
1. 回到Rider，点击`新建解决方案`
![image](https://github.com/user-attachments/assets/026d0090-d774-4cbf-90ca-f823ae29cdf7)
2. 按照下图填写，然后点击`创建`
![image](https://github.com/user-attachments/assets/d608695b-30b1-49f9-a185-e4169c35681e)

> [!NOTE]
> 一般来讲，项目名只由字母(A-Z,a-z)，数字(0-9)，下划线(_)组成，并且项目名具有实际意义  
> 
> 不要使用中文！不要使用中文！不要使用中文！(引自棱镜)​

> [!TIP]
> 解决方案中可以包含多个插件项目。使用同一个解决方案存放多个插件项目可以方便管理多个插件代码，并且方便项目之间互相引用。
> 
> 注: 如果提示没有按照目标框架，点击提示安装即可。


## 2.进入项目项目​

### 资源管理器(右上角)
它是一个非常重要的工具窗口,它可以用来管理、操作你的项目中的各个文件。其中Plugin.cs就是你插件的源码文件，我们双击就可以打开它。
![image](https://github.com/user-attachments/assets/b009c220-28ef-47a5-aa23-998c05fa3249)

> [!TIP]
> 如果哪一天资源管理器不见了，你可以在主菜单(左上角Rider图标旁的四条横线)-视图-工具窗口-资源管理器找到它
> ![image](https://github.com/user-attachments/assets/3ada86d5-e3dc-47ad-9af5-0f34b6e68a1e)

> [!TIP]
> 如果你觉得文件名字不顺眼，那么你可以从资源管理器重命名这个文件
> 当然同样要遵循命名规范，不随意使用中文
> ![image](https://github.com/user-attachments/assets/00dbda0b-0def-4a3f-b28a-239fa34c4d74)


## 3.开始编写代码​
这里有个简单的TShock插件示例，你可以复制替换到Plugin.cs
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
    public override string Author => "You Name";
    
    //插件功能的一句话描述
    public override string Description => "Plugin Description"; 

    //插件的版本, 可以采用日期来命名, eg: 2024年1月4日第1个版本 -> new(2024, 1,4,1)
    public override Version Version => new(2024, 1,4,1);
    
    //插件的构造器
    public Plugin(Main game) : base(game) 
    {
    }

    //插件加载时执行的代码
    public override void Initialize()
    {
        /*
        添加一个TShock命令,下面这句的意思是: (看不懂没关系，下面有专门的章节讲解命令)
        添加一个命令: 权限为`plugin.perm`,执行这条命令时调用`HelloWorldCommand`方法,命令的名字是`helloworld`,别名是`hw`
        */
        Commands.ChatCommands.Add(new Command("plugin.perm" , HelloWorldCommand, "helloworld", "hw"));
    }
        
    //插件卸载时执行的代码
    protected override void Dispose(bool disposing)
    {
        if (disposing)
        {
            //移除所有由本插件添加的所有指令
            var asm = Assembly.GetExecutingAssembly();
            Commands.ChatCommands.RemoveAll(c => c.CommandDelegate.Method?.DeclaringType?.Assembly == asm);
        }
        base.Dispose(disposing);
    }

    //执行指令时对指令进行处理的方法
    private void HelloWorldCommand(CommandArgs args)
    {
        //向触发事件的玩家发送一条绿色的消息 "Hello world!"
        args.Player.SendSuccessMessage("Hello world!");
    }
}

```
  
粘贴代码后，左下角的错误数是`0`，恭喜你可以开始构建插件了  
> [!NOTE]
> 如果有错误存在，可能是你复制漏了某些地方
> 也可能是你的环境有问题



## 5.构建你的插件​

点击工具栏处(右上角)的`锤子`即可开始构建
![image](https://github.com/user-attachments/assets/820e4e90-4d00-48cb-96c3-3f8f9cec76f7)

## 6.找到并安装你的插件​

1. 在资源管理器右键`当前项目(Myfirstplugin)`,选择`打开于-资源管理器` 
![image](https://github.com/user-attachments/assets/910e1136-e93e-4767-84b8-fdd95980b392)

  
2. 此时会弹出你的项目文件夹，顺着`bin/Debug/net6.0`就能找到你的插件啦~

![image](https://github.com/user-attachments/assets/97830e0c-d7dd-4950-914d-a172630a9887)


> [!NOTE]
> 文件的作用:  
> .dll: 插件的本体  
> .pdb: 调试文件, 放在`ServerPlugins`文件夹和插件一起加载, 报错时会显示报错方法对应源代码的具体位置  

3. 然后把插件的`.dll`文件和`.pdb`文件复制到TShock的`ServerPlugins`文件夹就可以开始调试啦~
> [!TIP]
> 可以像Cai把`TShockPlugins`文件夹固定在快速访问，这样就可以很快的把插件替换上去啦~
> ![image](https://github.com/user-attachments/assets/d8d8cf34-cab3-4a4a-9ba5-3f410123caf6)


3. 为什么要安装PDB (看不懂可以先忽略)  
我们下面有段代码,会在插件加载时报错  
```csharp
29   public void KoKoLoveYou()
30   {
        //抛出错误，模拟报错 :(
31      throw new Exception("别做梦了Cai");
32   }
33   
34   //插件加载时执行的代码
35   public override void Initialize()
36   {
37      if ("Cai"=="SB") //Cai不是SB, 所以KoKoLoveYou不会在这里被执行
38      {
39          KoKoLoveYou();
40      }
41  
42      if (1 + 1 == 2) //1+1=2 所以KoKoLoveYou在这里被执行
43      {
44          KoKoLoveYou();
45      }
46  }
```
- 如果你没有安装PDB那么你看到的报错是这样的
  ![image](https://github.com/user-attachments/assets/3e8d6d00-46d9-4cd3-84fd-267691a9b8cb)

  从报错我们可以知道，在调用插件的`Initialize`方法时，`KoKoLoveYou`方法报错。此时我们回到源代码，发现`Initialize`方法中有两个`KoKoLoveYou`，所以就会出现找不到具体报错位置的情况(实际情况肯定不是向上面例子一样你一眼就能看出来哪里有问题的)
- 如果我们安装了PDB
  ![image](https://github.com/user-attachments/assets/2c3af694-38e7-404e-a5f3-2afbc5de5948)
  此时，报错的调用栈就会告诉我们报错方法对应代码的具体位置，这里我们看到`Initialize`在`第44行`报错，对应我们代码的第二个`KoKoLoveYou`方法，这样报错的位置就被轻松找到了
  
## 7.测试你的插件​

1. 启动TShock, 可以看到你的第一个插件已经加载成功了
  ![image](https://github.com/user-attachments/assets/a75f20ec-fe35-4789-bbae-ddd767304689)

2. 使用/help命令查看命令列表，可以发现插件添加的命令已经在命令列表中显示 (这里装了`Help Plus`)
   ![image](https://github.com/user-attachments/assets/8907fd09-205e-4b46-a75d-27efc10899ac)

4. 执行`/helloworld`(或者`/hw`)，控制台打印出绿色`Hello world!`说明插件功能正常  
   ![image](https://github.com/user-attachments/assets/c00c9fe1-8874-4f0b-b1ca-263aed108fc5)

## 8.使用`VBY.PluginLoader`热重载插件 (by @xuyuwtu)
当我们修改代码后，我们需要把新构建的插件丢进`ServerPlugins`里，同时还需要重启TShock来重新加载我们的插件，TShock重启的速度很慢，这样很浪费时间，所以我们就可以使用`VBY.PluginLoader`热重载插件
| 命令 | 作用 |
| :-: | :-: |
| /load load | 加载所有PluginLoader所管理的插件 |
| /load unload | 卸载PluginLoader所管理的插件 |
| /load reload| 重载PluginLoader所管理的插件 |
| /load plugins| 显示PluginLoader已加载的插件 |
1. 使用前你必须删除`ServerPlugins`中你需要热重载的插件  
   ![image](https://github.com/user-attachments/assets/b2e73076-150d-41f2-a014-f5a0eaf30094)  
2. 安装`VBY.PluginLoader`，你可以在[TShock中文插件库](https://github.com/UnrealMultiple/TShockPlugin)找到这个插件  
   ![image](https://github.com/user-attachments/assets/3e6770b2-ed53-44b5-9a19-cacbcf1dff9b)  
3. 安装并加载后你的`TShock`根目录下会多出一个`PluginLoader`文件夹, 我们把插件丢进去  
   ![image](https://github.com/user-attachments/assets/9c7cc86c-dd9b-4f32-8e2f-62056fd6416b)  
   ![image](https://github.com/user-attachments/assets/36b7e577-3e83-4a43-a728-ce3fcd766501)   
4. 此时使用命令`/hw`,发现插件已经被`VBY.PluginLoader`加载。如果弹出两个, 说明你没有删除原插件文件夹的插件  
   ![image](https://github.com/user-attachments/assets/53127f9b-a009-4f0f-95ec-51411674dbdf)  
5. 我们这时候修改一下我们的代码，然后重新构建  
    ```csharp
    //执行指令时对指令进行处理的方法
    private void HelloWorldCommand(CommandArgs args)
    {
        //显示 “Hello 玩家的名字!”
        args.Player.SendSuccessMessage($"Hello {args.Player.Name}!");
    }
    ```
6. 把文件替换到`PluginLoader`文件夹  
   ![image](https://github.com/user-attachments/assets/c4898124-a116-44f8-9cc0-26f7563e112c)  
7. 执行命令`/load reload`  
   ![image](https://github.com/user-attachments/assets/ab562fd6-65aa-4e84-a811-912dd05c7077)  
8. 此时我们再执行`/hw`,就会出现我们修改过的结果啦~  
   ![image](https://github.com/user-attachments/assets/a775c220-9f23-4ba9-ad78-84facd913dcf)  

> [!NOTE]
> 控制台也算是玩家`(TSPlayer.Server)`，它的名字是`Server`
   


  

