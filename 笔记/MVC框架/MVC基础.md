**MVC 概念：**
1. **Model：** 创建一个类来存储游戏的数据，例如玩家的健康、分数等，并提供一些方法来操作这些数据。
2. **View：** 使用 Unity 的 UI 系统来创建显示数据的界面，如文本框、进度条等。View 监听 Model 的变化来更新界面。
3. **Controller：** 创建控制器类来处理用户的输入事件，并调用 Model 的方法来更新数据。同时，控制器会通知 View 来更新显示。
___
## 传统思想与 MVC 思想对比：
### 传统思想：
- **直接操作**：通常直接在脚本中通过代码获取UI控件（如按钮、文本框等），并直接对这些控件进行操作（如设置文本、改变颜色等）。这种方式简单直接，但在大型项目中容易导致代码混乱，难以维护。
- **耦合度高**：UI逻辑与业务逻辑紧密耦合在一起，修改UI时可能需要同时修改多个相关脚本，增加了维护成本。
- **复用性差**：UI控件和逻辑紧密绑定，使得UI组件难以在不同的场景或项目中复用。
### MVC 思想：
- **分离关注点**：MVC（Model-View-Controller）将应用程序分为三个核心部分：模型（Model）、视图（View）和控制器（Controller）。模型负责业务逻辑和数据管理，视图负责显示数据，控制器负责接收用户输入并调用模型和视图以完成用户请求。这种分离使得各部分可以独立开发和测试，提高了代码的可维护性和复用性。
- **低耦合**：MVC通过明确的职责划分降低了组件之间的耦合度。模型、视图和控制器之间通过接口或事件进行通信，而不是直接调用对方的方法或访问对方的属性。
- **易于扩展和维护**：由于MVC的组件化设计，当需要添加新功能或修改现有功能时，可以更容易地找到需要修改的部分，并且修改的影响范围更小。同时，由于模型和视图的分离，可以更容易地实现多视图共享同一模型数据的功能。
![屏幕截图 2024-08-15 231729.png](https://s2.loli.net/2024/08/15/UNTLFdl8qf91gRb.png)

![mvcduibi](https://s2.loli.net/2024/08/16/VCNFUs9mwYeHT6t.png)

---
## 使用 MVC 思想制作 UI 逻辑
### Model
#### 1. **数据管理（Model）**
`PlayerModel` 类保存了多个玩家属性，如 `playerName`、`lev`、`money`、`hp` 等。这些字段是私有的，只有通过公共属性或方法才能访问和修改。这确保了数据的封装性，外部无法直接操作这些数据，防止了不必要的修改。
```cs
using System.Collections;
using System.Collections.Generic;
using System.Linq.Expressions;
using System.Runtime.CompilerServices;
using UnityEngine;
using UnityEngine.Events;

public class PlayerModel
{
    private string playerName;
    //为每个私有变量声明公共的属性：外部只能得到，不能改变
    public string PlayerName
    {
        get
        {
            return playerName;
        }
    }
    private int lev;
    public int Lev
    {
        get
        {
            return lev;
        }
    }
    private int money;
    public int Money
    {
        get
        {
            return money;
        }
    }
    private int gem;
    public int Gem
    {
        get
        {
            return gem;
        }
    }
    private int power;
    public int Power
    {
        get
        {
            return power;
        }
    }
    private int hp;
    public int HP
    {
        get
        {
            return hp;
        }
    }
    private int atk;
    public int Atk
    {
        get
        {
            return atk;
        }
    }
    private int def;
    public int Def
    {
        get
        {
            return def;
        }
    }
    private int crit;
    public int Crit
    {
        get
        {
            return crit;
        }
    }
    private int miss;
    public int Miss
    {
        get
        {
            return miss;
        }
    }
    private int luck;
    public int Luck
    {
        get
        {
            return luck;
        }
    }
}
```
#### 2. **数据的唯一性（Singleton 模式）**

`PlayerModel` 类使用了单例模式来确保只有一个数据模型实例存在。通过 `Data` 属性来访问唯一的 `PlayerModel` 实例，如果实例不存在，则会创建一个新的实例并初始化数据。
```cs
//作为一个唯一的数据模型 一般情况 要不自己是个单例模式对象
//要不自己存在在一个单例模式对象中
private static PlayerModel data = null;
public static PlayerModel Data
{
    get
    {
        if( data == null )
        {
            data = new PlayerModel();
            data.Init();
        }
        return data;
    }
}
```
#### 3. **数据初始化**
`Init` 方法从本地存储中加载玩家数据（使用 `PlayerPrefs`），这些数据会在游戏启动或加载时初始化。如果找不到相应的数据，则使用默认值。
```cs
public void Init()
{
    playerName = PlayerPrefs.GetString("PlayerName", "唐老狮");
    lev = PlayerPrefs.GetInt("PlayerLev", 1);
    money = PlayerPrefs.GetInt("PlayerMoney", 9999);
    gem = PlayerPrefs.GetInt("PlayerGem", 8888);
    power = PlayerPrefs.GetInt("PlayerPower", 99);

    hp = PlayerPrefs.GetInt("PlayerHp", 100);
    atk = PlayerPrefs.GetInt("PlayerAtk", 20);
    def = PlayerPrefs.GetInt("PlayerDef", 10);
    crit = PlayerPrefs.GetInt("PlayerCrit", 20);
    miss = PlayerPrefs.GetInt("PlayerMiss", 10);
    luck = PlayerPrefs.GetInt("PlayerLuck", 40);
}
```
#### 4. **数据更新和保存**

当数据需要更新时，例如玩家升级时，`LevUp` 方法会修改相关属性，然后调用 `SaveData` 方法将新的数据保存到本地，并触发更新事件通知外部（View 和 Controller）。
```cs
public void LevUp()
{
    lev += 1;

    hp += lev;
    atk += lev;
    def += lev;
    crit += lev;
    miss += lev;
    luck += lev;

    SaveData();
}
public void SaveData()
{
    //把这些数据内容 存储到本地
    PlayerPrefs.SetString("PlayerName", playerName);
    PlayerPrefs.SetInt("PlayerLev", lev);
    PlayerPrefs.SetInt("PlayerMoney", money);
    PlayerPrefs.SetInt("PlayerGem", gem);
    PlayerPrefs.SetInt("PlayerPower", power);

    PlayerPrefs.SetInt("PlayerHp", hp);
    PlayerPrefs.SetInt("PlayerAtk", atk);
    PlayerPrefs.SetInt("PlayerDef", def);
    PlayerPrefs.SetInt("PlayerCrit", crit);
    PlayerPrefs.SetInt("PlayerMiss", miss);
    PlayerPrefs.SetInt("PlayerLuck", luck);

    UpdateInfo();
}
```
#### 5. **事件通知（Observer 模式）**
`PlayerModel` 类使用事件系统通知外部数据的变化。通过 `AddEventListener` 和 `RemoveEventListener` 方法，其他对象可以订阅或取消订阅数据更新事件。
```cs
//一般写在文件开头
private event UnityAction<PlayerModel> updateEvent;

public void AddEventListener(UnityAction<PlayerModel> function)
{
    updateEvent += function;
}

public void RemoveEventListener(UnityAction<PlayerModel> function)
{
    updateEvent -= function;
}

private void UpdateInfo()
{
    if( updateEvent != null )
    {
        updateEvent(this);
    }

    EventCenter.GetInstance().EventTrigger<PlayerModel>("玩家数据", this);
}
```
`UpdateInfo` 方法会触发事件，将当前的 `PlayerModel` 实例传递给订阅者（通常是 View 或 Controller），从而使得它们可以根据最新的数据更新界面或逻辑。
### View
`MainView.cs`:主要用于显示玩家的一些基础信息，如名字、等级、金钱、宝石和战斗力。它通常位于游戏主界面，提供玩家在游戏中的基本状态信息。
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class MainView : MonoBehaviour
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
    //使用 Model 的数据来更新 UI 元素的显示内容
    public void UpdateInfo( PlayerModel data )
    {
        txtName.text = data.PlayerName;
        txtLev.text = "LV." + data.Lev;
        txtMoney.text = data.Money.ToString();
        txtGem.text = data.Gem.ToString();
        txtPower.text = data.Power.ToString();
    }
}
```
`RoleView.cs`:专门用于显示玩家角色的详细属性，如生命值、攻击力、防御力、暴击率等。这个视图用于角色界面，玩家可以在这里查看和操作角色的具体属性。
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class RoleView : MonoBehaviour
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
    //使用 Model 的数据来更新 UI 元素的显示内容
    public void UpdateInfo(PlayerModel data)
    {
        txtLev.text = "LV." + data.Lev;
        txtHp.text = data.HP.ToString();
        txtAtk.text = data.Atk.ToString();
        txtDef.text = data.Def.ToString();
        txtCrit.text = data.Crit.ToString();
        txtMiss.text = data.Miss.ToString();
        txtLuck.text = data.Luck.ToString();
    }
}
```
### Controller
`MainController.cs`：通过 `PlayerModel` 作为数据源，并与 `MainView` 交互来更新界面显示
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MainController : MonoBehaviour
{
    //能够在Controller中得到界面才行
    private MainView mainView;


    private static MainController controller = null;

    public static MainController Controller
    {
        get
        {
            return controller;
        }
    }
    //1.界面的显影
    public static void ShowMe()
    {
        if (controller == null)
        {
            //实例化面板对象
            GameObject res = Resources.Load<GameObject>("UI/MainPanel");
            GameObject obj = Instantiate(res);
            //设置它的父对象 为Canvas
            obj.transform.SetParent(GameObject.Find("Canvas").transform, false);

            controller = obj.GetComponent<MainController>();
        }
        controller.gameObject.SetActive(true);
    }
    public static void HideMe()
    {
        if( controller != null )
        {
            controller.gameObject.SetActive(false);
        }
    }

    private void Start()
    {
        //获取同样挂载在一个对象上的 view脚本
        mainView = this.GetComponent<MainView>();
        //第一次的 界面更新
        mainView.UpdateInfo(PlayerModel.Data);

        //2.界面 事件的监听 来处理对应的业务逻辑
        mainView.btnRole.onClick.AddListener(ClickRoleBtn);
        //告知数据模块 当更新时 通知哪个函数做处理
        PlayerModel.Data.AddEventListener(UpdateInfo);
    }

    private void ClickRoleBtn()
    {
        Debug.Log("点击按钮显示角色面板");
        //通过Controller去显示 角色面板
        RoleController.ShowMe();
    }

    //3.界面的更新
    private void UpdateInfo( PlayerModel data )
    {
        if( mainView != null )
        {
            mainView.UpdateInfo(data);
        }
    }

    private void OnDestroy()
    {
        PlayerModel.Data.RemoveEventListener(UpdateInfo);
    }
}
```
#### 1. Controller 单例模式
`MainController` 采用了单例模式来保证控制器实例的唯一性。这种设计模式在管理全局状态或资源时非常有用。通过 `MainController. Controller` 可以全局访问唯一的控制器实例。
```cs
Private static MainController controller = null;
Public static MainController Controller
{
    Get
    {
        Return controller;
    }
}
```
#### 2. 显示与隐藏界面
`ShowMe` 和 `HideMe` 方法用于控制 `MainView` 的显隐。通过加载预制件并将其实例化为 MainController 对象，确保了当需要显示主界面时，它始终存在，并且与全局 Canvas 关联。
```cs
Public static void ShowMe ()
{
    If (controller == null)
    {
        GameObject res = Resources. Load<GameObject>("UI/MainPanel");
        GameObject obj = Instantiate (res);
        Obj.Transform.SetParent (GameObject.Find ("Canvas"). Transform, false);

        controller = obj. GetComponent<MainController>();
    }
    Controller.GameObject.SetActive (true);
}

Public static void HideMe ()
{
    If (controller != null)
    {
        Controller.GameObject.SetActive (false);
    }
}
```
#### 3. 初始化和事件绑定
在 Start 方法中，MainController 完成了对 View 的初始化，并绑定了按钮点击事件和数据更新事件。初始化过程中，MainView 与 Model (PlayerModel) 关联起来，通过调用 `UpdateInfo` 方法确保界面在启动时显示最新的数据。
```cs
Private void Start ()
{
    mainView = this. GetComponent<MainView>();
    MainView.UpdateInfo (PlayerModel. Data);

    MainView.BtnRole.OnClick.AddListener (ClickRoleBtn);
    PlayerModel.Data.AddEventListener (UpdateInfo);
}
```
#### 4. 按钮点击处理
`ClickRoleBtn` 方法是 `btnRole` 按钮的点击事件处理函数，它负责在用户点击按钮时显示角色面板。通过调用 `RoleController.ShowMe ()`，实现控制不同视图之间的切换。
```cs
Private void ClickRoleBtn ()
{
    Debug.Log ("点击按钮显示角色面板");
    RoleController.ShowMe ();
}
```
#### 5. 更新界面
`UpdateInfo` 方法用于处理当 PlayerModel 数据发生变化时的界面更新。这是 MVC 模式中 Controller 的核心职责之一：监听 Model 的变化，并通知 View 更新显示。
```cs
Private void UpdateInfo (PlayerModel data)
{
    If (mainView != null)
    {
        MainView.UpdateInfo (data);
    }
}
```
#### 6. 清理资源
在 OnDestroy 方法中，MainController 确保在销毁时取消对 PlayerModel 数据更新事件的监听，避免可能的内存泄漏或异常。
```cs
Private void OnDestroy ()
{
    PlayerModel.Data.RemoveEventListener (UpdateInfo);
}
```

`RoleController.cs`
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
public class RoleController : MonoBehaviour
{
    private RoleView roleView;

    private static RoleController controller = null;

    public static RoleController Controller
    {
        get
        {
            return controller;
        }
    }

    public static void ShowMe()
    {
        if (controller == null)
        {
            //实例化面板对象
            GameObject res = Resources.Load<GameObject>("UI/RolePanel");
            GameObject obj = Instantiate(res);
            //设置它的父对象 为Canvas
            obj.transform.SetParent(GameObject.Find("Canvas").transform, false);

            controller = obj.GetComponent<RoleController>();
        }
        //如果是隐藏的形式hide 在这要显示
        controller.gameObject.SetActive(true);

    }

    public static void HideMe()
    {
        if (controller != null)
        {
            //方式一 直接删
            //Destroy(panel.gameObject);
            //panel = null;
            //方式二 设置可见为隐藏
            controller.gameObject.SetActive(false);
        }
    }

    void Start()
    {
        roleView = this.GetComponent<RoleView>();
        //第一次更新面板
        roleView.UpdateInfo(PlayerModel.Data);

        roleView.btnClose.onClick.AddListener(ClickCloseBtn);
        roleView.btnLevUp.onClick.AddListener(ClickLevUpBtn);

        //告知数据模块 当更新时 通知哪个函数做处理
        PlayerModel.Data.AddEventListener(UpdateInfo);
    }


    private void ClickCloseBtn()
    {
        HideMe();
    }

    private void ClickLevUpBtn()
    {
        //通过数据模块 进行升级 达到数据改变
        PlayerModel.Data.LevUp();
    }

    private void UpdateInfo( PlayerModel data )
    {
        if( roleView != null )
        {
            roleView.UpdateInfo(data);
        }
    }

    private void OnDestroy()
    {
        PlayerModel.Data.RemoveEventListener(UpdateInfo);
    }
}
```
