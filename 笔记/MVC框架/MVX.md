### 常见的 MVX 模式
1. **MVC (Model-View-Controller)**:
2. **MVVM (Model-View-ViewModel)**:
    - **Model**: 数据和业务逻辑。
    - **View**: 用户界面，负责显示数据。
    - **ViewModel**: 是 View 的抽象，充当 View 和 Model 之间的中介。它通过绑定机制将数据从 Model 传递给 View，并且通常包含 UI 逻辑，使得 View 更加简洁。
    MVVM 模式广泛应用于数据绑定框架（如 WPF 和 Xamarin）中，它使得 UI 和业务逻辑的分离更加彻底。
3. **MVP (Model-View-Presenter)**:
    - **Model**: 数据和业务逻辑。
    - **View**: 负责展示用户界面，通常包含很少的逻辑（如果有的话）。
    - **Presenter**: 负责处理所有的 UI 逻辑。它从 Model 获取数据并将其传递给 View，同时处理用户的输入并更新 Model。
    与 MVC 不同的是，MVP 的 View 通常通过接口与 Presenter 交互，View 的实现被完全解耦。
### MVP ![image.png](https://s2.loli.net/2024/08/16/XzI84EhfYdjtBJ6.png)
#### MVP 中的 V
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class MVP_MainView : MonoBehaviour
{
    //1.找控件 
    public Button btnRole;
    public Button btnSill;

    public Text txtName;
    public Text txtLev;
    public Text txtMoney;
    public Text txtGem;
    public Text txtPower;
    //2.更新数据
    //public void UpdateInfo( string name, int lev, int money, int gem, int power )
    //{
    //    txtName.text = name;
    //    txtLev.text = lev.ToString();
    //    txtMoney.text = money.ToString();
    //    txtGem.text = gem.ToString();
    //    txtPower.text = power.ToString();
    //}
}
```

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
public class MVP_RoleView : MonoBehaviour
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
    //方法可选 你到时候可以直接在P里面通过访问控件 去修改
}
```
#### MPV 中的 Presenter
`MainPresenter.cs`
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MainPresenter : MonoBehaviour
{
    //能够在Presenter中得到界面才行
    private MVP_MainView mainView;

    private static MainPresenter presenter = null;

    public static MainPresenter Presenter
    {
        get
        {
            return presenter;
        }
    }
    //1.界面的显影
    public static void ShowMe()
    {
        if (presenter == null)
        {
            //实例化面板对象
            GameObject res = Resources.Load<GameObject>("UI/MainPanel");
            GameObject obj = Instantiate(res);
            //设置它的父对象 为Canvas
            obj.transform.SetParent(GameObject.Find("Canvas").transform, false);

            presenter = obj.GetComponent<MainPresenter>();
        }
        presenter.gameObject.SetActive(true);
    }
    public static void HideMe()
    {
        if (presenter != null)
        {
            presenter.gameObject.SetActive(false);
        }
    }

    private void Start()
    {
        //获取同样挂载在一个对象上的 view脚本
        mainView = this.GetComponent<MVP_MainView>();
        //第一次的 界面更新
        //mainView.UpdateInfo(PlayerModel.Data);
        //通过P自己的更新方法 去更新 View
        UpdateInfo(PlayerModel.Data);

        //2.界面 事件的监听 来处理对应的业务逻辑
        mainView.btnRole.onClick.AddListener(ClickRoleBtn);
        //告知数据模块 当更新时 通知哪个函数做处理
        PlayerModel.Data.AddEventListener(UpdateInfo);
    }
    
    private void ClickRoleBtn()
    {
        Debug.Log("点击按钮显示角色面板");
        //通过Controller去显示 角色面板
        //RoleController.ShowMe();

        RolePresenter.ShowMe();
    }

    //3.界面的更新
    private void UpdateInfo(PlayerModel data)
    {
        if (mainView != null)
        {
            //mainView.UpdateInfo(data);
            //以前是把数据M传到V中去更新 现在全部由P来做
            mainView.txtName.text = data.PlayerName;
            mainView.txtLev.text = "LV." + data.Lev.ToString();
            mainView.txtMoney.text = data.Money.ToString();
            mainView.txtGem.text = data.Gem.ToString();
            mainView.txtPower.text = data.Power.ToString();
        }
    }

    private void OnDestroy()
    {
        PlayerModel.Data.RemoveEventListener(UpdateInfo);
    }
}
```
`RolePresenter.cs`
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class RolePresenter : MonoBehaviour
{
    private MVP_RoleView roleView;

    private static RolePresenter presenter = null;

    public static RolePresenter Presenter
    {
        get
        {
            return presenter;
        }
    }

    public static void ShowMe()
    {
        if (presenter == null)
        {
            //实例化面板对象
            GameObject res = Resources.Load<GameObject>("UI/RolePanel");
            GameObject obj = Instantiate(res);
            //设置它的父对象 为Canvas
            obj.transform.SetParent(GameObject.Find("Canvas").transform, false);

            presenter = obj.GetComponent<RolePresenter>();
        }
        //如果是隐藏的形式hide 在这要显示
        presenter.gameObject.SetActive(true);

    }

    public static void HideMe()
    {
        if (presenter != null)
        {
            //方式一 直接删
            //Destroy(panel.gameObject);
            //panel = null;
            //方式二 设置可见为隐藏
            presenter.gameObject.SetActive(false);
        }
    }

    void Start()
    {
        roleView = this.GetComponent<MVP_RoleView>();
        //第一次更新面板
        //roleView.UpdateInfo(PlayerModel.Data);
        UpdateInfo(PlayerModel.Data);

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

    private void UpdateInfo(PlayerModel data)
    {
        if (roleView != null)
        {
            //roleView.UpdateInfo(data);
            //直接在P中得到V界面的控件 进行修改 断开V和M的联系
            roleView.txtLev.text = "LV." + data.Lev;
            roleView.txtHp.text = data.HP.ToString();
            roleView.txtAtk.text = data.Atk.ToString();
            roleView.txtDef.text = data.Def.ToString();
            roleView.txtCrit.text = data.Crit.ToString();
            roleView.txtMiss.text = data.Miss.ToString();
            roleView.txtLuck.text = data.Luck.ToString();
        }
    }

    private void OnDestroy()
    {
        PlayerModel.Data.RemoveEventListener(UpdateInfo);
    }
}
```
### MVVM (MP)
![image.png](https://s2.loli.net/2024/08/16/Dk4X8RN2wBodfSc.png)
**MVVM 在 Unity 中水土不服：**
1. View 对象始终由我们来书写, 并没有 UI 配置文件 (如 WPF 中的 XAML) 的存在
2. 硬要在 Unity 中实现 MVVM, 需要写三模块, 并且还要对 V 和 VM 进行数据绑定, 工作量大, 好处也不够明显
**什么是数据绑定：**
将 UI 元素（如 Unity 中的 `Text` 组件）直接绑定到 ViewModel 中的数据属性的能力。这样可以实现 UI 和底层数据之间的自动同步：
- **双向绑定:** UI 元素的更改会更新 ViewModel 的属性，反之亦然。
- **单向绑定:** 只将 ViewModel 的更改更新到 UI。
#### MVVM 的粗暴变式：MP
![image.png](https://s2.loli.net/2024/08/16/asMgyn5G3XIiTVv.png)
**MP**（Model-Presenter）将 Presenter（或 MVVM 中的 ViewModel）和 View 合并为一个脚本。既减少了手动数据绑定的需求，又保留了关注点分离的好处。
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class MP_MainPanel : BasePanel
{
    //1.找控件 通过集成小框架中的 UI基类实现了  
    //2.逻辑处理 
    //3.数据更新
    
    void Start()
    {
        //第一次打开时更新
        UpdateInfo(PlayerModel.Data)
        //PlayerModel.Data.AddEventListener(UpdateInfo);
        //使用MVE思想（EventCenter）
        EventCenter.GetInstance().AddEventListener<PlayerModel>("玩家数据", UpdateInfo);
    }

    protected override void OnClick(string btnName)
    {
        base.OnClick(btnName);
        switch (btnName)
        {
            case "btnRole":
                //处理角色面板打开
                UIManager.GetInstance().ShowPanel<MP_RolePanel>("RolePanel");
                break;
        }
    }

    public void UpdateInfo(PlayerModel data)
    {
        //直接在这获取控件 进行更新
        GetControl<Text>("txtName").text = data.PlayerName;
        GetControl<Text>("txtLev").text = "LV." +  data.Lev;

        GetControl<Text>("txtMoney").text = data.Money.ToString();
        GetControl<Text>("txtGem").text = data.Gem.ToString();
        GetControl<Text>("txtPower").text = data.Power.ToString();
    }

    private void OnDestroy()
    {
        //PlayerModel.Data.RemoveEventListener(UpdateInfo);
        EventCenter.GetInstance().RemoveEventListener<PlayerModel>("玩家数据", UpdateInfo);
    }
}
```

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class MP_RolePanel : BasePanel
{
    //1.找控件
    //2.处理逻辑
    //3.面板更新
    
    void Start()
    {
        //第一次显示时 更新面板
        UpdateInfo(PlayerModel.Data);
        //PlayerModel.Data.AddEventListener(UpdateInfo);
        // 注册一个事件监听器，当玩家数据发生变化时调用 UpdateInfo 方法更新面板
        EventCenter.GetInstance().AddEventListener<PlayerModel>("玩家数据", UpdateInfo);
    }

    protected override void OnClick(string btnName)
    {
        base.OnClick(btnName);
        switch (btnName)
        {
            case "btnClose":
                UIManager.GetInstance().HidePanel("RolePanel");
                break;
            case "btnLevUp":
                PlayerModel.Data.LevUp();
                break;
        }
    }

    public void UpdateInfo(PlayerModel data)
    {
        GetControl<Text>("txtLev").text = "LV." + data.Lev;
        GetControl<Text>("txtHp").text = data.HP.ToString();
        GetControl<Text>("txtAtk").text = data.Atk.ToString();
        GetControl<Text>("txtDef").text = data.Def.ToString();
        GetControl<Text>("txtCrit").text = data.Crit.ToString();
        GetControl<Text>("txtMiss").text = data.Miss.ToString();
        GetControl<Text>("txtLuck").text = data.Luck.ToString();
    }

    private void OnDestroy()
    {
        //PlayerModel.Data.RemoveEventListener(UpdateInfo);
        EventCenter.GetInstance().RemoveEventListener<PlayerModel>("玩家数据", UpdateInfo);
    }
}
```
#### MVE (Model-View-EventCenter)
![屏幕截图 2024-08-16 122735.png](https://s2.loli.net/2024/08/16/LCSqZIe6oYykBUP.png)
1. View 第一次显示获取 Mode 数据用于更新自己, 并通知事件中心监听事件
2. 数据更新时 (玩家操作或者服务器更新) 通过告知事件中心触发并分发事件
3. 数据从事件中心流入 View 中进行更新