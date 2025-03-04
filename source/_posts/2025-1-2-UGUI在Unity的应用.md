---
title: Unity UGUI 在项目中的应用与优化
date: 2025-03-09 10:00
update: 2025-03-09 11:00
cover: https://pic-bed-5pz.pages.dev/img/14.jpg
description: 深入探讨 UGUI 在 Unity 项目中的实际应用
swiper_index: 3 #置顶轮播图顺序，非负整数，数字越大越靠前
---

## Unity教程
在这篇文章中，我们将深入探讨 Unity 的 UGUI 系统在我的项目中的应用，包括动态生成 UI、事件处理、性能优化等内容。UGUI 是 Unity 内置的强大 UI 工具，灵活且易用，下面通过我的代码和一些额外技巧，带你看看如何用好它。

### UGUI 在项目中的核心应用

我的项目大量使用了 UGUI，比如登录界面、服务器选择面板等，涉及动态布局、交互逻辑和资源管理。以下是几个关键实现：

#### 动态生成 UI 元素
在 `ChooseServerPanel` 中，我通过代码动态生成服务器按钮，适应不同的服务器数量：
```csharp
GameObject rightbut = Instantiate(Resources.Load<GameObject>("UI/Right_But"));
rightbut.transform.SetParent(Right_Scroll.content, false);
Right_But RigBut = rightbut.GetComponent<Right_But>();
RigBut.RightButInfo(nowInfo);
Right_Button.Add(rightbut);
```
-**实现思路**：从 Resources 加载预制体，实例化后设置父对象为 ScrollRect 的 content，避免手动拖拽。

**好处**：服务器数量变化时，UI 自动调整，代码驱动比手调更灵活。
事件绑定与交互
UGUI 的事件系统非常强大，我的 Right_But 类用它实现了服务器选择：
```csharp
Rightbut.onClick.AddListener(() => 
{
    LoginMager.Instance.loginData1.ServerID = Nowinfos.id;
    UImag1.Instance.Hideme<ChooseServerPanel>();
    UImag1.Instance.ShowPanel<SeverPanel>();
});
```

**Lambda 表达式**：简洁绑定点击事件，减少冗余代码。
**面板切换**：通过 UImag1 的封装方法，轻松实现界面跳转。
**ScrollRect** 实现滚动视图
服务器列表使用了 ScrollRect，左右分开显示

**配置要点**：确保 content 的大小适配子物体数量，设置 Horizontal 或 Vertical 属性控制滚动方向。
**实际效果**：用户可以滑动浏览大量服务器，体验流畅。
 ### Toggle 与 InputField 的应用
在 LoginPanel 中，我用 Toggle 和 InputField 实现了记住密码和输入框功能：
```csharp
RememberTog.onValueChanged.AddListener((isOn) => 
{
    if (!isOn) AutoLoginTog.isOn = false;
});
User_Name.text = loginData.UserName;
User_PassWord.text = loginData.Password;
```

### UGUI 的进阶技巧
除了项目中的基础实现，UGUI 还有很多值得探索的技巧，下面补充一些我在开发中常用的方法。

**1. Canvas 的层级管理**
UGUI 的 Canvas 是所有 UI 的根节点，我的项目用单个 Canvas（通过 UImag1 的 canvastran）管理所有面板：
```csharp
canvastran = GameObject.Find("Canvas").transform;
```
**进阶建议**：如果 UI 复杂，可以用多个 Canvas，按功能分层（如 HUD、弹窗）。设置 Sorting Order 控制显示顺序，避免重叠问题。

**动态调整 RectTransform**
在动态生成 UI 时，手动调整位置和大小很实用。例如，假设我想让按钮均匀排列：
```csharp
RectTransform rect = rightbut.GetComponent<RectTransform>();
rect.anchoredPosition = new Vector2(10 + i * 110, 0); // 横向排列，每隔 110 单位
rect.sizeDelta = new Vector2(100, 50); // 设置宽高
```
### UI 性能优化
UGUI 的性能优化直接影响游戏流畅度，我的项目中有几处实践：

**使用 SpriteAtlas**
在 Right_But 中用 SpriteAtlas 加载服务器状态图标：
```csharp
SpriteAtlas stateicon = Resources.Load<SpriteAtlas>("StateIcon");
ButtonState.sprite = stateicon.GetSprite("ui_DL_liuchang_01");
好处：减少 Draw Call，提升渲染效率。
```

**批量销毁旧元素**
在更新服务器列表时，先清理旧按钮：
```csharp

for (int i = 0; i < Right_Button.Count; i++) { Destroy(Right_Button[i]); }
Right_Button.Clear();
```
**好处**：避免内存泄漏，保持帧率稳定。
**禁用不必要的 UI**
未显示的面板通过 SetActive(false) 隐藏，而不是销毁，节省加载时间。
### 自定义 UI 动画
UGUI 配合代码可以轻松实现动画效果。例如，淡入淡出：

```csharp

public void FadeIn(GameObject ui, float duration)
{
    CanvasGroup group = ui.GetComponent<CanvasGroup>() ?? ui.AddComponent<CanvasGroup>();
    StartCoroutine(FadeCoroutine(group, 0f, 1f, duration));
}

IEnumerator FadeCoroutine(CanvasGroup group, float start, float end, float duration)
{
    float time = 0f;
    while (time < duration)
    {
        time += Time.deltaTime;
        group.alpha = Mathf.Lerp(start, end, time / duration);
        yield return null;
    }
    group.alpha = end;
}
```
**应用**：在 Hideme<T> 中调用，提升切换体验。
项目中的优化心得

结合项目代码，我总结了几点 UGUI 使用心得：

**预制体规范化**：UI 预制体的命名和结构要统一（如 UI/Right_But），方便动态加载。
**避免过度嵌套**：UI 层级太多会增加 Draw Call，尽量扁平化。
**调试技巧**：用 Canvas 的 Override Sorting 检查层级问题，快速定位显示异常。
