免费开源的轻量级应用框架 https://puremvc.org
## PureMVC 基本结构
![屏幕截图 2024-08-16 124426.png](https://s2.loli.net/2024/08/16/TeOzRif6Wh92kS3.png)
Model (数据模型):
	关联 Proxy (代理) 对象, 负责处理数据
View (界面):
	关联 Mediator (中介) 对象, 负责处理界面
Controller (业务控制):
	管理 Command (命令) 对象, 负责处理业务逻辑
Facade (外观):
	是 MVC 三者的经纪人, 统管全局
	可以获取代理、中介、命令
Notification:
	通知, 负责传递信息
## PureMVC 的创建
- 默认打开的是多核版本，但推荐使用单核版本。
- 单核版本通过点击“Standard”链接获取。
### 方法一
1. 打开pureMVC.sin项目，右键生成PureMVC.dll包
2. 将dll包导入Unity工程./Assets/Plugins目录中
![image.png](https://s2.loli.net/2024/08/16/JZC9pubIloDrAh1.png)
![image.png](https://s2.loli.net/2024/08/16/dxCJSGqzsFHpYRy.png)
### 方法二
导入`Core` `Interfaces` `Patterns`文件夹至Unity的Scripts/PureMVC目录
![image.png](https://s2.loli.net/2024/08/16/6zPbVMl2f4seEXr.png)
优点：便于查看源码
### 通知名类
定义所有通知名称为静态常量，作为消息传递的"契约"，确保发送和接收通知使用相同的名称。便于统一管理和使用通知。
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
/// 这个是pureMVC中的 通知名类
/// 主要是用来申明各个通知的 名字 
/// 方便使用和管理
public class PureNotification 
{
	// 启动通知
    public const string START_UP = "startUp";
	// 显示面板通知
    public const string SHOW_PANEL = "showPanel";
    //隐藏面板通知
    public const string HIDE_PANEL = "hidePanel";
    // 代表玩家数据更新的通知名
    public const string UPDATE_PLAYER_INFO = "updatePlayerInfo";
	// 升级通知
    public const string LEV_UP = "levUp";
}
```
## Model和proxy
- Model: 定义数据结构(如PlayerDataObj)
- Proxy:
    - 继承PureMVC的Proxy类
    - 在构造函数中初始化数据
    - 提供数据操作方法(如LevUp、SaveData等)
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
// 玩家数据结构 
public class PlayerDataObj
{
    //申明一堆玩家属性相关的变量
    public string playerName;
    public int lev;
    public int money;
    public int gem;
    public int power;

    public int hp;
    public int atk;
    public int def;
    public int crit;
    public int miss;
    public int luck;
}
```
`PlayerProxy.cs`
```cs
using PureMVC.Patterns.Proxy;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
// 玩家数据代理对象
// 主要处理 玩家数据更新相关的逻辑
public class PlayerProxy : Proxy
{
    public new const string NAME = "PlayerProxy";
    //1.继承Proxy父类
    //2.写构造函数

    //写构造函数重要点
    //1.代理的名字
    //2.代理相关的数据
    public PlayerProxy():base(PlayerProxy.NAME)
    {
        //在构造函数中 初始化一个数据 进行关联
        PlayerDataObj data = new PlayerDataObj();

        //初始化
        data.playerName = PlayerPrefs.GetString("PlayerName", "唐老狮");
        data.lev = PlayerPrefs.GetInt("PlayerLev", 1);
        data.money = PlayerPrefs.GetInt("PlayerMoney", 9999);
        data.gem = PlayerPrefs.GetInt("PlayerGem", 8888);
        data.power = PlayerPrefs.GetInt("PlayerPower", 99);

        data.hp = PlayerPrefs.GetInt("PlayerHp", 100);
        data.atk = PlayerPrefs.GetInt("PlayerAtk", 20);
        data.def = PlayerPrefs.GetInt("PlayerDef", 10);
        data.crit = PlayerPrefs.GetInt("PlayerCrit", 20);
        data.miss = PlayerPrefs.GetInt("PlayerMiss", 10);
        data.luck = PlayerPrefs.GetInt("PlayerLuck", 40);

        //赋值给自己的Data进行关联
        Data = data;
    }

    public void LevUp()
    {
        PlayerDataObj data = Data as PlayerDataObj;
        //升级 改变内容
        data.lev += 1;

        data.hp += data.lev;
        data.atk += data.lev;
        data.def += data.lev;
        data.crit += data.lev;
        data.miss += data.lev;
        data.luck += data.lev;
    }

    public void SaveData()
    {
        PlayerDataObj data = Data as PlayerDataObj;
        //把这些数据内容 存储到本地
        PlayerPrefs.SetString("PlayerName", data.playerName);
        PlayerPrefs.SetInt("PlayerLev", data.lev);
        PlayerPrefs.SetInt("PlayerMoney", data.money);
        PlayerPrefs.SetInt("PlayerGem", data.gem);
        PlayerPrefs.SetInt("PlayerPower", data.power);

        PlayerPrefs.SetInt("PlayerHp", data.hp);
        PlayerPrefs.SetInt("PlayerAtk", data.atk);
        PlayerPrefs.SetInt("PlayerDef", data.def);
        PlayerPrefs.SetInt("PlayerCrit", data.crit);
        PlayerPrefs.SetInt("PlayerMiss", data.miss);
        PlayerPrefs.SetInt("PlayerLuck", data.luck);
    }
}
```
## View和Mediator
- View:
    - 定义UI元素
    - 提供更新UI的方法
- Mediator:
    - 继承PureMVC的Mediator类
    - 关联对应的View
    - 处理View相关的业务逻辑
    - 监听和处理通知
View:
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class NewMainView : MonoBehaviour
{
    //1.找控件
    public Button btnRole;
    public Button btnSill;

    public Text txtName;
    public Text txtLev;
    public Text txtMoney;
    public Text txtGem;
    public Text txtPower;
    
    //2.提供面板更新的相关方法给外部
    //按照MVC的思想 可以直接在这里提供 更新的方法
    public void UpdateInfo(PlayerDataObj data)
    {
        txtName.text = data.playerName;
        txtLev.text = "LV." + data.lev;
        txtMoney.text = data.money.ToString();
        txtGem.text = data.gem.ToString();
        txtPower.text = data.power.ToString();
    }
}
```
角色面板View：
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class NewRoleView : MonoBehaviour
{
    //1.找控件
    public Button btnClose;
    public Button btnLevUp;

    public Text txtLev;
    public Text txtHp;
    public Text txtAtk;
    public Text txtDef;
    public Text txtCrit;
    public Text txtMiss;
    public Text txtLuck;
    //2.提供面板更新的相关方法给外部
    public void UpdateInfo(PlayerDataObj data)
    {
        txtLev.text = "LV." + data.lev;
        txtHp.text = data.hp.ToString();
        txtAtk.text = data.atk.ToString();
        txtDef.text = data.def.ToString();
        txtCrit.text = data.crit.ToString();
        txtMiss.text = data.miss.ToString();
        txtLuck.text = data.luck.ToString();
    }
}
```

Mediator:
在 PureMVC 框架中，每个 `Mediator` 都有自己的 `NAME` 字段来标识其名称。由于 `NAME` 是静态的，在子类中重定义 `NAME` 字段时使用 `new` 关键字，可以确保这个字段是属于子类自己的标识符，而不是与基类的 `NAME` 混淆。
```cs
using PureMVC.Interfaces;
using PureMVC.Patterns.Mediator;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class NewMainViewMediator : Mediator
{
    public static new  string NAME = "NewMainViewMediator";
    //套路写法
    //1.继承PureMVC中的Mediator脚本 
    //2.写构造函数
    public NewMainViewMediator():base(NAME)
    {
        //这里面可以去创建界面预设体等等的逻辑
        //但是界面显示应该是触发的控制的
        //而且创建界面的代码 重复性比较高
    }

    public void SetView(NewMainView view)
    {
        ViewComponent = view;
        view.btnRole.onClick.AddListener(()=>
        {
            SendNotification(PureNotification.SHOW_PANEL, "RolePanel");
        });
    }

    //3.重写监听通知的方法*
    public override string[] ListNotificationInterests()
    {
        //这是一个PureMVC的规则
        //就是你需要监听哪些通知 那就在这里把通知们通过字符串数组的形式返回出去
        //PureMVC就会帮助我们监听这些通知 
        // 类似于 通过事件名 注册事件监听
        return new string[]{
            PureNotification.UPDATE_PLAYER_INFO,
        };
    }

    //4.重写处理通知的方法*
    public override void HandleNotification(INotification notification)
    {
        //INotification 对象 里面包含两个队我们来说 重要的参数
        //1.通知名 我们根据这个名字 来做对应的处理
        //2.通知包含的信息 
        switch (notification.Name)
        {
	        //与监听通知配对
            case PureNotification.UPDATE_PLAYER_INFO:
                //收到 更新通知的时候 做处理
                if(ViewComponent != null)
                {
                    (ViewComponent as NewMainView).UpdateInfo(notification.Body as PlayerDataObj);
                }
                break;
        }
    }

    //5.可选：重写注册时的方法
    public override void OnRegister()
    {
        base.OnRegister();
        //初始化一些内容
    }
}
```
角色模板mediator：
```cs
using PureMVC.Interfaces;
using PureMVC.Patterns.Mediator;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class NewRoleViewMediator : Mediator
{
    public static new string NAME = "NewRoleViewMediator";
    //套路写法
    //1.继承PureMVC中的Mediator脚本 
    //2.写构造函数
    public NewRoleViewMediator():base(NAME)
    {

    }

    public void SetView(NewRoleView view)
    {
        ViewComponent = view;
        //关闭按钮 事件监听
        view.btnClose.onClick.AddListener(() =>
        {
            SendNotification(PureNotification.HIDE_PANEL, this);
        });

        //升级按钮监听
        view.btnLevUp.onClick.AddListener(()=>
        {
            //去升级
            //去通知升级
            SendNotification(PureNotification.LEV_UP);
        });
    }

    //3.重写监听通知的方法
    public override string[] ListNotificationInterests()
    {
        return new string[] {
            PureNotification.UPDATE_PLAYER_INFO,
            //以后你还关心别的通知 就在这后面通过逗号连接 加起来就行了
        };
    }


    //4.重写处理通知的方法
    public override void HandleNotification(INotification notification)
    {
        //INotification 对象 里面包含两个队我们来说 重要的参数
        //1.通知名 我们根据这个名字 来做对应的处理
        //2.通知包含的信息 
        switch (notification.Name)
        {
            case PureNotification.UPDATE_PLAYER_INFO:
                //玩家数据更新 逻辑处理
                if(ViewComponent != null)
                {
                    (ViewComponent as NewRoleView).UpdateInfo(notification.Body as PlayerDataObj);
                }
                break;
        }
    }
}
```
## Facade和Command
- Facade:
    - 单例模式，初始化Controller,注册Command
    - 提供启动方法
- Command:
    - 继承SimpleCommand类
    - 重写Execute方法处理具体业务逻辑
Facade:
```cs
using PureMVC.Interfaces;
using PureMVC.Patterns.Facade;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GameFacade : Facade
{
    //1.继承PureMVC中Facade脚本
    
    //2.为了方便我们使用Facade 需要自己写一个单例模式的属性
    public static GameFacade Instance
    {
        get
        {
            if( instance == null )
            {
                instance = new GameFacade();
            }
            return instance as GameFacade;
        }
    }
    
    //3.初始化 控制层相关的内容
    protected override void InitializeController()
    {
        base.InitializeController();
        //这里面要写一些 关于 命令和通知 绑定的逻辑
        RegisterCommand(PureNotification.START_UP, () =>
        {
            return new StartUpCommand();
        });

        RegisterCommand(PureNotification.SHOW_PANEL, () =>
        {
            return new ShowPanelCommand();
        });

        RegisterCommand(PureNotification.HIDE_PANEL, () =>
        {
            return new HidePanelCommand();
        });

        RegisterCommand(PureNotification.LEV_UP, () =>
        {
            return new LevUpCommand();
        });
    }

    //4.一定是有一个启动函数的
    public void StartUp()
    {
        //发送通知
        SendNotification(PureNotification.START_UP);

        //SendNotification(PureNotification.SHOW_PANEL, "MainPanel");
    }
}
```
Command:
```cs
using PureMVC.Interfaces;
using PureMVC.Patterns.Command;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class StartUpCommand : SimpleCommand
{
    //1.继承Command相关的脚本
    //2.重写里面的执行函数
    public override void Execute(INotification notification)
    {
        base.Execute(notification);
        //当命令被执行时 就会调用该方法
        //启动命令中 往往是做一些初始化操作

        //没有这个数据代理 才注册 有了就别注册
        if( !Facade.HasProxy(PlayerProxy.NAME) )
        {
            Facade.RegisterProxy(new PlayerProxy());
        }
    }
}
```
### 显隐面板
显示面板：
```cs
public class ShowPanelCommand : SimpleCommand
{
	public override void Execute(INotification notification)
	{
		base.Execute(notification);
		string panelName = notification.Body.ToString();
		switch(panelName)
		{
			case "MainPanel":
				//显示面板相关内容
                //如果要使用Mediator 一定也要在Facade中去注册
                //command、proxy都是一样的 要用 就要注册
                //可以在命令 中 直接使用Facade代表的就是唯一的 Facade

                //判断如果没有mediator就去new一个 
                if( !Facade.HasMediator(NewMainViewMediator.NAME) )
                {
                    Facade.RegisterMediator(new NewMainViewMediator());
                }
                //有mediator了 下一步 就是去创建界面 创建预设体 

                //Facade 得到Mediator的方法
                NewMainViewMediator mm = Facade.RetrieveMediator(NewMainViewMediator.NAME) as NewMainViewMediator;
                //实例化面板对象
                if (mm.ViewComponent == null)
                {
                    GameObject res = Resources.Load<GameObject>("UI/MainPanel");
                    GameObject obj = GameObject.Instantiate(res);
                    //设置它的父对象 为Canvas
                    obj.transform.SetParent(GameObject.Find("Canvas").transform, false);
                    //得到预设体上的 view脚本 关联到 mediator上
                    mm.SetView(obj.GetComponent<NewMainView>());
                }
                //往往现实了面板后 需要在这里进行第一次更新
                //需要把 数据一起通过参数 传出去
                SendNotification(PureNotification.UPDATE_PLAYER_INFO, Facade.RetrieveProxy(PlayerProxy.NAME).Data);

				break;
			case "RolePanel":
				if (!Facade.HasMediator(NewRoleViewMediator.NAME))
				{
					Facade.RegisterMediator(new NewRoleViewMediator());
				}
				NewRoleViewMediator rm = Facade.RetrieveMediator(NewRoleViewMediator.NAME) as NewRoleViewMediator;
				if(rm.ViewComponent == null)
				{
					GameObject res = Resources.Load<GameObject>("UI/RolePanel");
					GameObject obj = GameObject.Instantiate(res);
					obj.transform.SetParent(GameObject.Find("Canvas").transform, false);
					rm.SetView(obj.GetComponent<NewRoleView>());
				}
				SendNotification(PureNotification.UPDATE_PLAYER_INFO, Facade.RetrieveProxy(PlayerProxy.NAME).Data);
				break;
		}
	}

}
```
隐藏面板：
```cs
using PureMVC.Interfaces;
using PureMVC.Patterns.Command;
using PureMVC.Patterns.Mediator;
using UnityEngine;

public class HidePanelCommand : SimpleCommand
{
    public override void Execute(INotification notification)
    {
        base.Execute(notification);
        //隐藏的目的
        //得到 mediator 再得到 mediator中的 view 然后去要不删除 要不 设置显隐
        //得到传入的 mediator
        Mediator m = notification.Body as Mediator;

        if( m != null && m.ViewComponent != null )
        {
            //直接删除场景上的面板对象
            GameObject.Destroy((m.ViewComponent as MonoBehaviour).gameObject);
            //删了后 一定要置空
            m.ViewComponent = null;
        }
    }
}
```
挂载Main函数：
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Main : MonoBehaviour
{
    void Start()
    {
        GameFacade.Instance.StartUp();   
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.M))
        {
            //显示主面板
		    GameFacade.Instance.SendNotification(PureNotification.SHOW_PANEL, "MainPanel");
        }
        else if (Input.GetKeyDown(KeyCode.N))
        {
            //隐藏主面板
            GameFacade.Instance.SendNotification(PureNotification.HIDE_PANEL, GameFacade.Instance.RetrieveMediator(NewMainViewMediator.NAME));
        }
    }
}
```
### 升级命令
```cs
using PureMVC.Interfaces;
using PureMVC.Patterns.Command
public class LevUpCommand : SimpleCommand
{
    public override void Execute(INotification notification)
    {
        base.Execute(notification);

        //得到数据代理 调用升级 升级完成后通知别人 更新数据
        PlayerProxy playerProxy = Facade.RetrieveProxy(PlayerProxy.NAME) as PlayerProxy;

        if( playerProxy != null )
        {
            //升级
            playerProxy.LevUp();
            playerProxy.SaveData();
            //通知更新
            SendNotification(PureNotification.UPDATE_PLAYER_INFO, playerProxy.Data);
        }
    }
}
```
