# 5.数据的存储与加载

假如现在需要新添加一个系统，为了保存数据，需要做哪些工作？

以成就系统为例，做一下数据的持久化。

### 数据的保存

1. 首先在成就类中，添加可序列化的数据

```csharp
[SerializeField]
public AchievementData data; // AchievementData 包括是否解锁，累计计数等等参数
```

2. 然后在SaveData中，添加字段

```csharp
[SerializeField]
public SerializableDictionary<int, AchievementData> achievements = new();
```

3. 在SaveData.GetDataToSave()中，添加代码

<pre class="language-csharp"><code class="lang-csharp">data.achievements = Achievement.achievementDatas; // 这个静态字段是为了方便存档特意设置的
<strong>// 在加载时初始化并保存
</strong><strong>// 也可以采取另一种方法：初始化时保存所有的实例类，在保存时遍历所有实例类中的数据
</strong></code></pre>



### 数据的加载

数据的加载用到了反射，将其实例存入字典之中

```csharp
public static void LoadAchievementByReflection()
{
    var assembly = typeof(Achievement).Assembly;
    
    var types = assembly.GetTypes();
    foreach (var type in types)
    {
        if (type.IsSubclassOf(typeof(Achievement)) && type != typeof(Achievement))
        {
            var achievement = Activator.CreateInstance(type) as Achievement;
            achievementDict.Add(achievement.Id, achievement);
        }
    }
}
```

然后从存档中读取数据

```csharp
public static void LoadAchievementFromSave()
{
    var saveData = SaveData.Load(PlayerConfigs.SaveName);
    if (saveData is null)
        return;

    achievementDatas = saveData.achievements;

    foreach (var (id, achievement) in achievementDict)
    {
        if(achievementDatas.entries.Any(entry => entry.key == id))
        {
            achievement.data = achievementDatas[id];
        }
        else
        {
            achievement.data = new AchievementData();
        }
        achievement.RegisterCondition();
    }
    
}
```

最后在sceneInitializer中添加这两个方法，就可以在场景初始化时从存档中加载数据了

### 其他考虑

这样实现的存档存在一个问题，即很难实现不同版本存档的数据互通，但是这个问题也不算太紧迫，可以在之后考虑解决。

目前想到了两种解决方案：

1. 在存档类中加入版本转换的功能，将存档强制转换为适合版本的存档
2. 加载存档时开启一个本地服务器进程，返回适合版本的存档
