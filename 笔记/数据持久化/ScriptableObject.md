在编辑模式下，ScriptableObject 具有数据持久化特性；在发布运行时，ScriptableObject 并不具有数据持久化特性
### 数据文件的创建
#### 方法一：自定义ScriptableObject数据容器
1. 继承ScriptableObject类
2. 在该类中声明成员（变量、方法等）
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Video;

[CreateAssetMenu(fileName ="MrTangData", menuName ="ScriptableObject/我的数据", order = 0)]
public class MyData : ScriptableObject
{
    public int i;
    public float f;
    public bool b;

    public GameObject obj;
    public Material m;
    public AudioClip audioClip;
    public VideoClip videoClip;
}
```
效果：`menuName ` , `order = 0`:菜单项会出现在菜单的最前面。 ![image.png](https://s2.loli.net/2024/08/13/x6IAlHdfkRJSBew.png) 
`fileName`
![image.png](https://s2.loli.net/2024/08/13/mi6523aHLhRpQUu.png)
#### 方法二：根据自定义的ScriptableObject数据容器创建数据文件
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEditor;
using UnityEngine;

public class ScriptableObjectTool
{
    [MenuItem("ScriptableObject/CreateMyData")]
    public static void CreateMyData()
    {
        //书写创建数据资源文件的代码
        MyData asset = ScriptableObject.CreateInstance<MyData>();
        //通过编辑器API 根据数据创建一个数据资源文件
        AssetDatabase.CreateAsset(asset, "Assets/Resources/MyDataTest.asset");
        //保存创建的资源
        AssetDatabase.SaveAssets();
        //刷新界面
        AssetDatabase.Refresh();
    }
}
```
效果：
![image.png](https://s2.loli.net/2024/08/13/1XSgDvKtwLdF7zI.png)

---
### 数据文件的使用与生命周期
#### 通过 Inspector 中的 public 变量进行关联
1. 创建一个数据文件
2. 在继承 MonoBehaviour 类中申明数据容器类型的成员
#### 通过资源加载的信息关联
```cs
data = Resources.Load<MyData>("MyDataTest");
```
---
### ScriptableObject 非持久化数据
#### 生成 ScriptableObject 非持久化数据
- 利用 ScriptableObject 中的静态方法 `CreateInstance<>()`
- 该方法可以在运行时创建出指定继承 ScriptableObject 的对象
- 该对象只存在于内存当中，可以被 GC
- 调用一次就创建一次
- 通过这种方式创建出来的数据对象，它里面的默认值不会受到脚本中设置的影响（全是初始化类时的默认值）
```cs
    //data = ScriptableObject.CreateInstance("MyData") as MyData;
    data = ScriptableObject.CreateInstance<MyData>();
```
#### 非持久化数据的意义
1. 只希望在运行时能有一组唯一的数据可以使用，但是这个数据不希望保存为数据资源文件浪费硬盘空间
2. 特点：只在运行时使用，在编辑器模式下也不会保存在本地
---
### 真正意义上的持久化
*ScriptableObject并不适合用来做数据持久化功能，但是我们可以利用我们学过的数据持久化方案（PlayerPrefs、XML、Json、2 进制等）让其持久化。有些画蛇添足*
```cs
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEngine;

public class Lesson4 : MonoBehaviour
{
    void Start()
    {
    	MyData data = ScriptableObject.CreateInstance<MyData>();
    	
        #region 利用Json结合ScriptableObject存储数据
        data.PrintInfo();

        data.i = 9999;
        data.f = 6.6f;
        data.b = true;
        //将数据对象 序列化为 json字符串
        string str = JsonUtility.ToJson(data);
        print(str);
        //把数据序列化后的结果 存入指定路径当中
        File.WriteAllText(Application.persistentDataPath + "/testJson.json", str);
        print(Application.persistentDataPath);
        #endregion

        #region 利用Json结合ScriptableObject读取数据
        //从本地读取 Json字符串
        string str = File.ReadAllText(Application.persistentDataPath + "/testJson.json");
        //根据json字符串反序列化出数据 将内容覆盖到数据对象中
        JsonUtility.FromJsonOverwrite(str, data);
        data.PrintInfo();
        #endregion
    }
}
```

---
### 配置数据
#### ScriptableObject 数据文件非常适合用来做配置文件
1. 配置文件的数据在游戏发布之前定规则
2. 配置文件的数据在游戏运行时只会读出来使用，不会改变内容
3. 在 Unity 的 Inspector 窗口进行配置更加的方便
#### 制作举例
数据存储 `RoleInfo` 类包含一个 `List<RoleData>` 类型的字段 `roleList`，用于存储多个角色的数据
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu(fileName ="RoleInfo", menuName = "ScriptableObject/角色信息")]
public class RoleInfo : SingleScriptableObject<RoleInfo>
{
	//想在Inspector编辑自定义类 需要序列化
    [System.Serializable]
    public class RoleData
    {
        public int id;
        public string res;
        public int atk;
        public string tips;
        public int lockMoney;
        public int type;
        public string hitEff;
        
        public void Print()
        {
            Debug.Log(id);
            Debug.Log(res);
            Debug.Log(atk);
            Debug.Log(tips);
            Debug.Log(lockMoney);
            Debug.Log(type);
            Debug.Log(hitEff);
        }
    }

    public List<RoleData> roleList;
}
```

---
### 复用数据
对于只用不变的数据使用 prefab 可能有内存浪费问题 (不同对象，相同数据时)
`BulletInfo.cs`
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
[CreateAssetMenu()]
public class BulletInfo : ScriptableObject
{
    public float speed;
    public int atk;
}
```
`New Bullet Info` 共享同一内存空间数据，修改一处数据，其余都改变。
```cs
public class Move : MonoBehaviour
{
	public BulletInfo info;
	void Update()
	{
	    if (Input.GetKeyDown(KeyCode.Space))
	        info.speed += 1;
	}
}
```

---
### 数据带来的多态行为
#### 什么是数据带来的多态行为
某些行为的变化是因为数据的不同带来的，我们可以利用面向对象的特性和原则，以及设计模式相关知识点，结合ScriptableObject做出更加方便的功能，比如随机音效，物品拾取，AI等：
- 随机音效（里氏替换原则和依赖倒转原则）：播放音乐时，可能会随机播放多个音效当中的一种
- 物品拾取（里氏替换原则和依赖倒转原则）：比如拾取一个物品，物品给玩家带来不同的效果
- AI：不同数据带来的不同行为模式
#### 随机音效
`AudioPlayBase.cs`:  `AudioPlayBase` 是一个抽象类，包含一个抽象方法 `Play`这个方法没有实现，必须在派生类中实现。
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public abstract class AuidoPlayBase : ScriptableObject
{
    public abstract void Play(AudioSource source);
}
```
`RandomPlayAudio.cs`: 1. **继承抽象类**：`RandomPlayAudio` 继承了 `AudioPlayBase` 并实现了`Play`方法。在这个方法中，随机选择一个音效文件并播放。
```cs
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu()]
public class RandomPlayAudio : AudioPlayBase
{
    // 希望随机播放的音效文件
    public List<AudioClip> clips;
    // 实现抽象方法
    public override void Play(AudioSource source)
    {
        if (clips.Count == 0)
            return;
        // 设置音效切片文件
        source.clip = clips[Random.Range(0, clips.Count)];
        source.Play();
    }
}
```
挂载脚本 `AudioManager.cs`
```cs
public class AudioManager : MonoBehaviour
{
	public AudioPlayBase audioPlay;
	void Start()
	{
		audioPlay.Play(this.GetComponent<AudioSource>());
	}
}
```
#### 物品拾取
创建基类，提供抽象方法
```cs
using UnityEngine;
public abstract class ItemEffect : ScriptableObject
{
    public abstract void AddEffect(GameObject obj);
}
```
加血效果类
```cs
using UnityEngine;
[CreateAssetMenu(menuName = "ItemEffects/HealthEffect")]
public class HealthEffect : ItemEffect
{
    public int healthAmount;
    public override void AddEffect(GameObject obj)
    {
        // 假设玩家有一个 PlayerHealth 组件来管理健康值
        PlayerHealth playerHealth = obj.GetComponent<PlayerHealth>();
        if (playerHealth != null)
        {
            playerHealth.IncreaseHealth(healthAmount);
        }
    }
}
```
加攻击力效果
```cs
using UnityEngine;
[CreateAssetMenu]
public class AddAtkItemEffect : ItemEffect
{
    public int atk;
    public override void AddEffect(GameObject obj)
    {
       //具体加多少攻击力的逻辑
    }
}
```
挂载到物体上，检测触发
```cs
using UnityEngine;
public class ItemObj : MonoBehaviour
{
    public ItemEffect eff;
    private void OnTriggerEnter(Collider other)
    {
        //为和物品产生碰撞的对象加效果
        eff.AddEffect(other.gameObject);
    }
}
```
效果：
![屏幕截图 2024-08-13 175326.png](https://s2.loli.net/2024/08/13/Gluep7aQOUV91z3.png)

---
### 单例模式获取数据
#### 为什么要单例模式化的获取数据？
- 对于只用不变并且要复用的数据——比如配置文件中的数据，我们往往需要在很多地方获取他们，可能我们会直接通过在脚本中 public关联，或者动态加载
- 如果在多处使用，会存在很多重复代码，效率较低
- 如果我们将此类数据通过单例模式化的去获取，可以提升效率，减少代码量

**使用 where T : ScriptableObject：**
- **类型安全**：通过指定 `where T : ScriptableObject`，你确保了类型参数 `T` 必须是 `ScriptableObject` 或其子类。这意味着在泛型类或方法中可以安全地访问 `ScriptableObject` 的属性和方法，避免了类型不匹配的问题。
- **访问特定功能**：`ScriptableObject` 是 Unity 中一种特殊的类，用于保存数据。在泛型类或方法中使用这个约束，可以确保 `T` 能够使用 `ScriptableObject` 提供的功能，比如通过 `Resources.Load` 动态加载资源等。
- **避免运行时错误**：这个约束在编译时就会检查 `T` 是否满足 `ScriptableObject` 的要求，避免了在运行时因为类型不匹配而导致的错误。

```cs
using UnityEngine;
public class SingleScriptableObject<T> :ScriptableObject where T:ScriptableObject
{
    private static T instance;
    public static T Instance
    {
        get
        {
            //如果为空 首先应该去资源路径下加载 对应的 数据资源文件
            if (instance == null)
            {
                //我们定两个规则
                //1.所有的 数据资源文件都放在 Resources文件夹下的ScriptableObject中
                //2.需要复用的 唯一的数据资源文件名 我们定一个规则：和类名是一样的
                instance = Resources.Load<T>("ScriptableObject/" + typeof(T).Name);
            }
            //如果没有这个文件 为了安全起见 我们可以在这直接创建一个数据
            if(instance==null)
            {
                instance = CreateInstance<T>();
            }
            //可以在这里从json当中读取数据，但不建议用ScriptableObject来做数据持久化
            return instance;
        }
    }
}
```
#### 实现单例模式化获取数据
[[#制作举例]]
```cs
using UnityEngine;
public class xxx : MonoBehaviour
{
	void Start()
	{
		//通过单例快速获取数据
		print(RoleInfo.Instance.roleList[0].id);
		print(RoleInfo.Instance.roleList[1].tips);
	}
}
```

---
### 附案例：使用 ScriptableObject 制作存储游戏设置信息的数据
##### 音乐音效开关 &音乐音效大小
```cs
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEngine;

[CreateAssetMenu(fileName = "SettingInfo", menuName = "ScriptableObject/音乐音效设置信息")]
public class SettingInfo : ScriptableObject
{
    //音乐和音效的开关
    public bool musicIsOpen;
    public bool soundIsOpen;

    //音乐和音效的 大小
    public float musicValue;
    public float soundValue;
}
```
##### UI 控件设置
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class SettingPanel : MonoBehaviour
{
    public Toggle musicTog;
    public Toggle soundTog;
    public Slider musicSlider;
    public Slider soundSlider;

    public SettingInfo info;

    void Start()
    {
        musicTog.isOn = info.musicIsOpen;
        soundTog.isOn = info.soundIsOpen;
        musicSlider.value = info.musicValue;
        soundSlider.value = info.soundValue;

        //当UI面板上 控件变化时 记录数据
        musicTog.onValueChanged.AddListener((value) =>
        {
            info.musicIsOpen = value;
        });

        soundTog.onValueChanged.AddListener((value) =>
        {
            info.soundIsOpen = value;
        });

        musicSlider.onValueChanged.AddListener((value) =>
        {
            info.musicValue = value;
        });

        soundSlider.onValueChanged.AddListener((value) =>
        {
            info.soundValue = value;
        });
    }
    
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
            info.Save();
    }
}
```
##### 修改为非持久化
```cs
void Start()
{
	//添加此行
    info = ScriptableObject.CreateInstance<SettingInfo>();
    
    musicTog.isOn = info.musicIsOpen;
    soundTog.isOn = info.soundIsOpen;
    musicSlider.value = info.musicValue;
    soundSlider.value = info.soundValue;
}
```
##### 修改为真正意义上的持久化
```cs
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEngine;

[CreateAssetMenu(fileName = "SettingInfo", menuName = "ScriptableObject/音乐音效设置信息")]
public class SettingInfo : ScriptableObject
{
    //音乐和音效的开关
    public bool musicIsOpen;
    public bool soundIsOpen;

    //音乐和音效的 大小
    public float musicValue;
    public float soundValue;

    private void Awake()
    {
        //判断是否存在持久化的数据文件
        if( File.Exists(Application.persistentDataPath + "/SettingInfo.json") )
        {
            string str = File.ReadAllText(Application.persistentDataPath + "/SettingInfo.json");
            JsonUtility.FromJsonOverwrite(str, this);
        }
    }

    public void Save()
    {
        string str = JsonUtility.ToJson(this);
        File.WriteAllText(Application.persistentDataPath + "/SettingInfo.json", str);
    }
}
```