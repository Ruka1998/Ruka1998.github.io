---
title: Unity的地形开发管线
tags: 场景 工具 材质
---

由于这段时间泡在场景组这边，接到了他们提出的许多需求，其中地形相关的占了许多，零零散散做下来，发现已经集成了一套地形的小小管线，因此做一总结。

# 工具

首先需要提到的是Unity自带的Preview Package：**Terrain Tools**，虽说有许多地形相关的功能，但个人感觉最为便利的是多出来的笔刷（可旋转）、Mesh Stamp、splatmaps的导入导出。问题在于，是预览版所以不太稳定。

## 样条线铺路

![样条线](/assets/images/unity-terrain-tools/spline.png)

虽然没有用过虚幻，但是听美术讲了讲他们的地形样条线，正巧发现已经有现成的[样条线轮子](https://assetstore.unity.com/packages/tools/utilities/b-zier-path-creator-136082)，以及[实现参考](https://forum.unity.com/threads/adjust-terrain-along-a-spline.929658/)，于是开始造车。

直接把所有的参考导入到项目里，似乎总是不能运行起来，在一番操作之后，做了如下修改：

- **目录结构的确定**：PathCreator放到Asset/EditorTool之下，在子目录新建一个AddOn文件夹（与Core文件夹并列），这个文件夹再建两个文件夹Editor和Runtime分别放TerrainAdjusterRuntimeEditor和TerrainAdjusterRuntime。

- **bug处理**：
    - `Vector3[] vertexPoints = pathCreator.path.vertices;`这个致使报错而且没用，注释掉
    - `distancePoints.Sort((a, b) => -a.y.CompareTo(b.y));`用处不大而且会导致路面出现扭曲极窄，注释掉
    - `AdjustTerrain`函数中，在获取了宽高后加上`if (centerX < 0 || centerX >= width || centerY < 0 || centerY >= height) return;`，防止样条线超出地形的时候报错

- **将所有功能集成到自己的面板** 这个Gui是放在Asset/Editor里面的

![样条线](/assets/images/unity-terrain-tools/splineToolGui.png)

- **运行顺序的修改**：TerrainAdjusterRuntime里面的`pathCreator = GetComponent<PathCreator>();`需要放到`Awake`函数里，确保在Editor的`OnEnable`之前运行，其实这里如果不添加自己的工具面板是没问题的，因为脚本挂上去会让Runtime先于Editor运行，我这里使用自己写的工具面板挂脚本，就会出现问题，所以才要改顺序

- **在面板的工具种记录样条线建立之前的地形高度，便于恢复**

现在还有的问题是，如果把Brush Spacing弄小，可以发现笔刷的范围交叉的地方不对劲，导致边缘很硬

## 根据地形陡峭度铺贴图

如题，这个功能[参考](https://alastaira.wordpress.com/2013/11/14/procedural-terrain-splatmapping/)。

![地形贴图](/assets/images/unity-terrain-tools/steepBasedTex.png)

![地形贴图面板](/assets/images/unity-terrain-tools/steepBasedTexEditor.png)


# 材质

让地形的八层贴图都能支持调色，材质面板如下：

![地形贴图面板](/assets/images/unity-terrain-tools/terrainMaterial.png)

这个操作的重点在于，unity自己的terrain shader乍一看相当迷惑，除了前四张贴图的混合，后四张都是使用Add-Pass的

```c
Dependency "AddPassShader" = "Hidden/RF/Terrain/Standard-AddPass"
Dependency "BaseMapShader" = "RF/Terrain/StandardBase"
Dependency "BaseMapGenShader" = "RF/Terrain/StandardBaseGen"
```

使用这种Dependency要怎么往AddPass的Shader里传递参数呢？在一番神奇的试验后发现：**只要在主Pass与AddPass的Properties中做一模一样的声明就可以了**

另外第五张图的albedo alpha设置为Emission Map，不开启自发光时还是作为金属度处理。