## Unity3D 开发基于 Wii 模型的游戏<br><br>
> ### Java Jdk

### [Java Jdk](http://www.oracle.com/technetwork/java/javase/downloads/index.html)<br>
下载最新版
<hr>
> ### Android Sdk

### [Android Sdk](https://developer.android.com/studio/index.html)<br>
下载最新版，并更新`Packages`与最新Api的`SdkPlatform`
<hr>
> ### Unity 3D

### [Unity 3D](https://unity3d.com/unity/qa/patch-releases)<br>
下载最新Patch版，或 [正式发布版](https://store.unity.com/download?ref=personal)
<hr>
> ### 提取游戏模型与骨骼动画

`WiiScrubber`提取Wii镜像的文件<br>
基于生化危机安布雷拉历代记和暗黑编年史中的内容，需`chara`目录的文件<br>
`EZ00.arc`与`EZ0S.arc`用`BrresModelViewer\Convert`打开<br>
`3DModels(NW4R)`是模型，`AnmChr(NW4R)`是骨骼动画<br>
模型导出为`Psk`格式，骨骼动画导出为`Psa`格式，贴图将自动随模型一同导出
<hr>
> ### 转换模型与骨骼动画为Fbx

`DnUnPsaToolkit`依次添加`模型Psk`与`骨骼动画Psa`，并另存为新文件<br>

```
可将多份Psa合并保存为一个，但其中的每份骨骼动画在Unity3D内分割时会有问题
```
`Psk`与`新Psa`用`Noesis`转换导出<br>
`Main output type`选择`.fbx`<br>
`Adbanced options`填入`-rotate 90 0 90 -scale 10`

```
如果不在此将模型或骨骼动画旋转并放大，则需要在Unity3D内调整
```
批量转换操作可以附加`$inpath$\$inname$.$outext$`命令来指定输出文件名与目录
<hr>
> ### 导入Fbx至Unity3D

模型与骨骼动画的`Fbx`导入`Unity3D`，更改适合模型的`Materials`<br>
骨骼动画Fbx中的动画，`Ctrl+D`拷贝至`Project`内为独立的动画片段，删除Fbx<br>
动画片段属性中，勾选`Loop Time`与`Loop Pose`<br>
这将使骨骼动画绑定模型后，循环运动并忽略骨骼动画自身位移

```
这并非是对骨骼动画的最完美解决方案，此方法Unity3D有可能会提示一个空引用警告
而另一种方案则是直接在骼动画Fbx进行设置，像上章节提到的多份骨骼动画分割等操作
此方案可以使用骨骼动画中的自身位移：
---骨骼动画Fbx属性Rig，将Root node选择为骨骼动画的根节点（非骨骼树形结构根节点）
---将模型Fbx也同样设置
---模型拖入Hierarchy，在属性Animator勾选Apply Root Motion即可使用位移
---不勾选Apply Root Motion则不使用位移（骨骼动画绑定模型见下章节）
但此方案在游戏时，有可能会自动将模型水平旋转90或180度
```
<hr>
> ### 骨骼动画绑定至模型并自动运动

`Project`创建`Avatar Mask`<br>
属性`Transform`的`Use skeleton from`用模型Fbx的`Avator`赋予<br>
`Inport skeleton`导入骨骼树形结构

```
骨骼树形结构包含所有骨骼节点，有选择的勾选需要绑定相应骨骼的节点
如上下半身的动作由两份动画来定义，则需要两份Avatar Mask，分别绑定不同的骨骼节点
```
`Project`创建`Animator Controller`<br>
创建相应骨骼的`Layers`，属性`Weight`为`1`<br>
`Mask`用相应骨骼的`Avatar Mask`赋予<br>
`Layers`创建`State`，属性`Motion`用相应骨骼动画的`动画片段`赋予<br>
模型拖入`Hierarchy`，属性`Animator`的`Controller`用`Animator Controller`赋予

```
不同骨骼节点的骨骼动画需要创建相应的Layers
但多个相同骨骼节点的骨骼动画可以用同一个Layers，其定义多个State，并设定转换关系
如站立、走路、跑步都需要全身骨骼节点，但需要创建一个空State关联至Entry默认状态
```
<hr>
> ### 控制骨骼动画

`Animator Controller`的`Layers`创建多个`State`<br>
`Animator Controller`的`Parameters`创建参数用以转换多个`State`的关系<br>
将关联`Entry`默认状态的`State`在其它`State`上用`Make Transition`创建双向关联线<br>
点击关联线，属性`Conditions`添加`Parameters`的参数并选择生效条件<br>
`Project`创建`C# Script`：

```ruby
using UnityEngine;
public class NewBehaviourScript : MonoBehaviour
{
    private Animator _Anim;
    void Start()
    {
        _Anim = this.GetComponent<Animator>();
    }
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            _Anim.SetBool("Is_Walk", true);
        }
        AnimatorStateInfo _AnimSI = _Anim.GetCurrentAnimatorStateInfo(0);
        if (_AnimSI.shortNameHash == Animator.StringToHash("S_Walk"))
        {
            _Anim.SetBool("Is_Walk", false);
        }
    }
}
```
脚本赋予引擎`Hierarchy`的相应模型<br>
按下空格键，生效Is_Walk参数用以运动命名为S_Walk的`State`骨骼动画<br>
通过`Layers`索引获取当前播放动画信息来将参数改变停止骨骼动画运动<br>
S_Walk的`State`如果有反向关联`Entry`默认状态的`State`则恢复默认状态<br>
否则模型变为S_Walk骨骼动画最后一帧的样子

```
骨骼动画过度生硬，尝试将Animator Controller的Layers属性Blending改为Additive
```
<hr>
> ### 交互操作响应

与模型交互需要其有`Colliders`，套盒子或从模型生成`Mesh Colliders`<br>
`Project`创建`C# Script`：

```ruby
using UnityEngine;
public class NewBehaviourScript : MonoBehaviour
{
    void Start() { }
    void Update()
    {
        if (Input.GetMouseButton(0))
        {
            Ray _Ray = Camera.main.ScreenPointToRay(Input.mousePosition);
            RaycastHit _RayHit;
            if (Physics.Raycast(_Ray, out _RayHit))
            {
                Debug.DrawLine(_Ray.origin, _RayHit.point);
                GameObject _GameObj = _RayHit.collider.gameObject;
                Debug.Log(_GameObj.name);
            }
        }
    }
}
```
脚本赋予`Hierarchy`的主镜头<br>
按下鼠标左键，从镜头发出射线定位接触到的模型对象，产生逻辑操作
<hr>
> ### 编译并发布

编译Android版需要Unity3D的安卓支持包，且引擎为专业版<br>
还需Android Sdk用以提供编译Apk的支持<br>
一个简单的Demo已开发完成，其中部分步骤可以使用脚本批量自动化
<hr>
