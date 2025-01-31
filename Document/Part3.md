# Part 3.添加新命令​

本章你将学到：  

- 如何在TShock插件中添加新命令
- 学会处理CommandArgs参数
- 学会卸载插件添加的命令

## 命令的组成
### 名字(Name)
顾名思义，就是这个命令的名字，例如: 如果命令名字就是`cai`，那么对应的命令就是`/cai`
> [!NOTE]  
> TShock允许命令重名，例如A插件定义了命令`/back`，那么B插件同样也可以定义命令`/back`  
> 如果玩家执行命令`/back`那么就会先后执行A、B两个插件的命令逻辑，执行顺序取决于命令被添加的先后(主要看插件的加载顺序)

### 别名(Alias)
命令的其他名字，比如：关闭服务器的命令名是`off`, 别名是`exit`、`stop`，那么你执行`/exit`、`/stop`也可以像执行`/off`一样关闭服务器
> [!NOTE]  
> 命令只有一个名字，但是可以有无数个别名

### 权限(Permissions)
执行命令所需的权限，一个命令可以有多个权限，也可以没有权限。如果玩家所在组没有对应的权限，就会被提示无权执行此命令   
权限的命名: `插件名.简略描述`或`插件名.分类.简单描述` (字母全小写)  
例如: `myplugin.use`、`myplugin.admin`、`myplugin.player.slap`
### 处理方法(CommandDelegate)
处理命令的方法，这个方法用来处理这个命令
### 命令说明(HelpText)
对命令的简单说明，使用`/help <命令>`时会显示
### 允许非玩家执行(AllowServer)
允许服务器控制台或者REST用户执行此命令，默认为`true`  
如果改为`false`,服务器控制台和REST用户将无法使用此命令
### 记录参数日志(DoLog)
默认为`true`，改为`false`后将不会记录此命令的参数日志  
![image](https://github.com/user-attachments/assets/721f3838-e32d-4d24-9c5c-aac95b650df9)  
## 命令在哪？
在TShock中，命令以`Command实例`的形式被存放在`Commands.ChatCommands`这个列表中
## 命令如何被执行？
首先，明确一点，命令是聊天的一种。TShock通过命令标识符(`CommandSpecifier`)(默认`/`)和静默命令标识符(`CommandSilentSpecifier`)(默认`.`)来区分聊天和命令  
当有玩家发送聊天时TShock会检查这个聊天内容的开头是否是命令标识符或者静默命令标识符, 如果是就执行命令，不是就当作聊天处理,例如: `KoKo真可爱`会被视为聊天，而`/KoKo真可爱`会被视为命令
然后TShock会去寻找是否有对应的命令，然后检查权限，最后执行命令的处理方法

## 1.添加新的命令​
在TShock中，我们通常在初始化方法`Initialize`中添加的命令，具体代码如下:
```csharp
public override void Initialize()
{
    Commands.ChatCommands.Add(new Command("myplugin.use", MyCmdHandler, "mycmd", "mc"));
    //Command()中第一个参数是权限名，第二个参数是处理方法，第三个参数及之后是命令名和别名
}
```

1. 我们复制之后，发现会报错，找不到`MyCmdHandler`这个方法
![image](https://github.com/user-attachments/assets/c6ae1887-277d-4e92-a4e0-5b1b680a80e1)  
> [!NOTE]
> 如果你报错是找不到Commands  
> ![image](https://github.com/user-attachments/assets/47b1213f-3902-42c9-b9ee-988766aaabaa)
> 你就需要先导入TShockAPI命名空间，具体方法如下：  
> ```csharp
> //文件顶部加上对应的using语句
> using TShockAPI;
> ```  
> 当然你也可以用Rider的快速操作
> ![PixPin_2025-01-18_12-41-27](https://github.com/user-attachments/assets/d8043374-c5df-4a9a-8e01-bcce8768884f)
  
2. 此时我们右键我们需要创建的方法名，然后使用Rider的快速操作构造我们的处理方法
![PixPin_2025-01-18_12-43-16](https://github.com/user-attachments/assets/ff5e8b03-2654-46b0-842c-d2dc6bcf0421)  
3. 现在我们就可以开始编写具体的命令逻辑啦~  
![image](https://github.com/user-attachments/assets/02e56c53-0c19-4134-9383-c3abd2efb787)  


## 2.命令的权限​



```csharp
public override void Initialize()
{
    Commands.ChatCommands.Add(new Command("mycommand", Cmd, "cmd2"));
}


private void Cmd(CommandArgs args)
{
    args.Player.SendSuccessMessage("你使用了cmd2命令!");
}
```

- 上面的示例代码中，当玩家输入"/cmd2"命令并且拥有"mycommand"权限时，就会向玩家发送一条绿色的消息，内容为"你使用了cmd2命令\!"
- 若玩家没有相应权限则会提示玩家无权执行该命令，并且日志会记录玩家试图执行命令的行为并发送给打开接受日志的玩家

![image](https://github.com/user-attachments/assets/f7d902ee-3595-4712-94ef-207925cbb685)


![image](https://github.com/user-attachments/assets/8dd1eed2-d75c-4d6e-972e-2c3337a5e802)


  
- 权限也可以设为无或者多个权限，如下代码可以添加3条分别为/cmd1\(无权限\)、/cmd2\(单权限\)、/cmd3\(多权限\)的命令  



```csharp
public override void Initialize()
{
    Commands.ChatCommands.Add(new Command(Cmd, "cmd1")); //不设置权限
    Commands.ChatCommands.Add(new Command("mycommand", Cmd, "cmd2")); //设置一个权限
    Commands.ChatCommands.Add(new Command(new List<string> { "mycommand","mycommand2" }, Cmd, "cmd3")); //设置多个权限
}
```

- 执行上面的命令，都会调用"Cmd"函数，他们的区别在于权限，权限区别请见下表:  

| 命令 | 权限 |
| --- | --- |
| /cmd1 | 无权限限制 |
| /cmd2 | 玩家需要拥有mycommand权限 |
| /cmd3 | 玩家需要拥有mycommand或者mycommand2权限 |


>注: 像/cmd3类似的多权限命令，只要玩家拥有命令中设置的任何一个权限，玩家就可以执行对应的命令  

- 权限的命名  
权限的命名是需要遵循一定的命名规范的并不能像上面代码中随便命名
- 通常插件权限名字格式应为: 插件名.功能名\(例如: si.use\)  
- 当然为了方便分类也可以使用: 插件名.功能.功能细分 \(例如: uban.playerban.admin\)  

## 3.命令的别名​

- 有时候一个命令可以由两个名字对应，例如：/off和/exit他们属于同一个命令，/exit就是/off的别名  
我们可以通过下面的代码实现命令别名  



```csharp
public override void Initialize()
{
    Commands.ChatCommands.Add(new Command("cancanneed", Vme, "vme50", "v我50"));
}
private void Vme(CommandArgs args)
{
    args.Player.GiveItem(74, 50, 0); //给玩家50个铂金币
}
```

- 上述代码中，当玩家使用/vme50或者/v我50且有"cancanneed"权限，就会给予这个玩家50铂金币  
- 此时"/v我50"就是"/vme50"的别名  
- 同样即使你使用别名，你仍然可以使用那几个权限的重载，就像如下示例:  



```csharp
public override void Initialize()
{
    Commands.ChatCommands.Add(new Command(Vme, "giveme50", "送我50"));
    Commands.ChatCommands.Add(new Command("cancanneed", Vme, "vme50", "V我50"));
    Commands.ChatCommands.Add(new Command(new List<string> { "乞丐" , "beggar" } , Vme, "begme50", "乞讨50"));
}
```

### 小结: 你可以把记作回调函数\(例如Vme\)就像一条分界线，回调函数左侧的是权限\(就像"cancanneed"\)，回调函数右侧的是名字\(就像"vme50"、"V我50"\)

![image](https://github.com/user-attachments/assets/e093d57a-51ae-4e42-8787-bf428a273c00)


## 4.命令的其他属性​

- TShock的命令还有4个属性，这些属性是AllowServer、DoLog、HelpDesc、HelpText，你可以使用new Command\(\)\{ 属性 = 值 \}来修改他们，也可以用如下方法一次修改多个属性  



```csharp
public override void Initialize()
{
    Commands.ChatCommands.Add(new Command(Vme, "giveme50", "送我50")
    { AllowServer = false,DoLog=true, HelpDesc = new string[]{ "帮助文档","这是第一行"}, HelpText= "帮助文本"});
}
```

### AllowServer​

- 默认为true，AllowServer被设置为false时，非真实玩家\(服务器后台、REST远程命令\)无法使用这个命令  

![image](https://github.com/user-attachments/assets/156ddc58-10b3-4fd3-9fb1-effb3f67973e)


  

### DoLog​

- 默认为true，当你设置为false时，服务器日志不会记录执行该命令的具体参数，只会记录命令名\(例如：使用命令/login 123456，只会记录/login\)  

![image](https://github.com/user-attachments/assets/b2f90e7c-9009-468d-b187-46a8ef6d7a47)


  

### HelpDesc和HelpText​

- HelpDesc的默认值为null，HelpText的默认值为No help available.\(在当前设置的语言有效时会被设为对应语言的翻译\)  
- 这两者基本一样，都是命令帮助文档，使用/help 命令名可以查看  
- HelpDesc的优先级会更高，当命令的HelpDesc不为null时只会显示HelpDesc  

![image](https://github.com/user-attachments/assets/9c3cdc88-b0e5-4ef5-af34-1fa95057b0cd)

### 5.CommandArgs参数​

- 当玩家执行我们添加的命令时，服务器会把一些相关的信息"打包"传给我们的回调函数，这些信息就是CommandArgs  

![image](https://github.com/user-attachments/assets/dd4820fb-6a47-4e14-8bab-898eba1fa46f)


  
- **它包含Message、Player\(重要\)、TPlayer、Silent、Parameters\(重要\)**

![image](https://github.com/user-attachments/assets/b59692bd-a44d-4ea7-a581-3a813155995a)



### Parameters​

- 当你输入含有参数的命令时，TShock会把命令分割成Parameters  

### Parameters是命令的参数，类型为字符串列表​



![image](https://github.com/user-attachments/assets/263c7307-baa5-41df-888e-efd02370254a)


  
>你可以使用switch-case结构实现子命令功能，示例如下：  



```csharp
//代码来自Cai的SSC管理器

private void CommandHandler(CommandArgs args)
{
    if (args.Parameters.Count == 0)
    {
        args.Player.SendErrorMessage("无效的SSC管理器子命令!有效命令如下:\n" +
                "ssc save [ID]--保存SSC\n" +
                "ssc del [ID]--删除SSC\n" +
                "ssc list --列出SSC\n" +
                "ssc restore --复原SSC");
        return;
    }
    int sscid = -1;
    switch (args.Parameters[0].ToLower())
    {
        case "保存":
        case "save":
            if (args.Parameters.Count != 2 || !int.TryParse(args.Parameters[1], out sscid))
            {
                args.Player.SendErrorMessage("用法错误!正确用法:ssc save [ID]");
                return;
            }


            if (SSCDB.InsertPlayerData(args.Player, sscid))
            {
                args.Player.SendSuccessMessage("保存成功!");
                return;
            }
            args.Player.SendErrorMessage("保存失败!");
            break;
        case "删除":
        case "del":
            if (args.Parameters.Count != 2 || !int.TryParse(args.Parameters[1], out sscid))
            {
                args.Player.SendErrorMessage("用法错误!正确用法:ssc del [ID]");
                return;
            }
            if (!SSCDB.GetPlayerData(sscid).exists)
            {
                args.Player.SendErrorMessage("你输入的SSC背包ID不存在!");
                return;
            }
            if (SSCDB.DeletePlayerData(sscid))
            {
                args.Player.SendSuccessMessage("删除成功!");
                return;
            }
            args.Player.SendErrorMessage("删除失败!");
            break;
        case "列出":
        case "list":
            args.Player.SendSuccessMessage("有效的SSC背包列表:" + string.Join(',', SSCDB.GetAllSSCId()));
            break;
        case "还原背包":
        case "restore":
            if (args.Parameters.Count != 2 || !int.TryParse(args.Parameters[1], out sscid))
            {
                args.Player.SendErrorMessage("用法错误!正确用法:ssc restore [ID]");
                return;
            }
            if (!SSCDB.GetPlayerData(sscid).exists)
            {
                args.Player.SendErrorMessage("你输入的SSC背包ID不存在!");
                return;
            }
            args.Player.PlayerData = SSCDB.GetPlayerData(sscid);
            args.Player.PlayerData.RestoreCharacter(args.Player);
            args.Player.Heal(args.Player.PlayerData.maxHealth);
            args.Player.SendSuccessMessage("背包已还原!");
            break;
        default:
            args.Player.SendErrorMessage("无效的SSC管理器子命令!有效命令如下:\n" +
                "ssc save [ID]--保存SSC\n" +
                "ssc del [ID]--删除SSC\n" +
                "ssc list --列出SSC\n" +
                "ssc restore --复原SSC");
            break;
    }
}
```







### Player​

- 执行命令的玩家的TSPlayer对象，通常可以对其进行以下操作：  



```csharp
args.Player.SendData(PacketTypes.WorldInfo); //发送数据包,根据NetMessage发送数据包
args.Player.SendRawData(new byte[] { }); //发送原始数据，即手搓数据包
args.Player.SendErrorMessage("这是一个错误信息"); //发送错误信息(红色)
args.Player.SendInfoMessage("这是一个信息"); //发送信息(黄色)
args.Player.SendSuccessMessage("这是一个成功信息"); //发送成功信息(绿色)
args.Player.SendWarningMessage("这是一个警告信息"); //发送警告信息(橙色)
args.Player.Kick("这是一个踢出信息",true,true,"Cai",false);
//踢出玩家(踢出理由，强制踢出，不发送广播，踢出用户名，保存玩家的存档)
args.Player.Ban("这是一个封禁信息","Cai"); //封禁玩家(封禁理由，封禁用户名)
args.Player.SetTeam(1); //设置玩家队伍，无队伍0，红队1，绿队2，蓝队3，黄队4，粉队5
args.Player.SetBuff(1, 1); //添加玩家Buff (BuffID,Buff持续时间(单位1/60s))
args.Player.Teleport(1, 1); //传送玩家到坐标(X,Y)
args.Player.Disconnect("这是一个断开信息"); //断开玩家的连接(断开理由)(也算是踢出玩家的一种)
```


### TPlayer​

- Player对象，即Terraria原版玩家对象，此处不讲解  

### Silent​

- 是否为静默执行
- 静默执行取决于命令是否写了静默执行的逻辑，如果没有，那么静默执行和正常执行没有任何区别
> - 当玩家使用CommandSpecifier\(默认为/\)为命令起始符时Silent为false  
>- 当玩家使用CommandSilentSpecifier\(默认为.\)为命令起始符时Silent为true  

### Message​

- 原消息，例如：你执行了/kick Cai 傻逼，那么Message的值就是"kick Cai 傻逼"  
由于Parameters的存在
  

### 5.卸载你添加的命令​

- 使用下面的代码就可以卸载本插件添加的所有命令  



```csharp
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
```
