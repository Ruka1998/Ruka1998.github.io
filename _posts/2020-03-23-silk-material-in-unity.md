---
title: 在unity中实现基于PBR的丝绸材质
tags: Unity, Shader, 布料, PBR
---

丝绸的实现挺普遍的了，但是自己实际操作一遍之后还是会发现有些小细节需要注意的地方。

# 开始前的准备

## 丝绸的特性
![常见的丝绸](/assets/images/satin.jpg)

丝绸最主要的就是其各向异性高光，这也正是为什么在unity中，丝绸的效果必须单独实现而不能借助standard shader（不支持各向异性的高光，当然，可以自己添加库，修改自带BRDF，但本质上和本文的实现是相同的原理）。

## 模型制作
在Blender中做布料是很方便的，Google一下就有很多教程。为了让布料有很细腻的褶皱，我专门弄了一个平面斜着往下落，由一个圆环和一个另一个平面接住，于是可以形成一个比较方便参考的模型。在Unity中看起来是这样的：

![自己做的丑丑的模型](/assets/images/fabricModel.jpg)

最后观察的时候就只用关注褶皱的位置。

## 相应算法

我觉得这里有必要简单说一下实时PBR实现的一些知识基础，可能有不准确的地方。

PBR全称为Physically Based Rendering，重点在于这个“基于物理”，这个方法描述的是光与物质相互作用的过程。可以把PBR想象成一个黑盒，输入光照信息，然后返回一个处理过的光照，这束光回到摄像机里头形成我们看到的图像。

一般来说，光撞击到某一物质体积的表面上，会产生反射光和透射光，反射光形成后面将要计算的镜面光（specular），透射光视物质性质或被吸收或被散射回原来的介质，如果散射回去了，这个光就是次表面散射光（subsurface scattering）。如果光从进入到散射出去的距离不大（在一个着色样本内），可以称为我们常见的漫射光，如果距离很远（跨越了着色样本），就涉及到了全局的次表面散射效果。

这里，我直接用了Lambert做漫反射，Lambert公式很简单:
```c
float Lambert(){
    return 1.0 / PI; //在Unity内使用UNITY_PI
}
```
老实说用这个我是有点犹豫的，因为Lambert看起来比较糊糊的，但是最后的效果还行，效率很香，就没有改动了，之后可能会改进一下。

镜面光的实现一般参考微表面原理，大意是再怎么粗糙的表面也是由无数个小的光滑面组成的。计算公式如下：

$$
f_{spec} = \frac{F\cdot G\cdot D}{4|n\cdot l||n\cdot v|}
$$

F指Fresnel函数，G是遮蔽函数，D是法线分布函数。Fresnel有一个有名的效应：站在海岸边，如果低头往下看，水是很透彻的，可以看到水底的沙子，如果眺望远一点的海面，就发现水面变得和镜子一样，只能看到反射过来的景象。简单理解，入射角越大，光反射的越多。遮蔽函数讲的是微表面相互遮挡的现象，可以过滤一部分不应该返回摄像机的光线。法线分布函数可以理解为高光分布，我们看到的高光形状主要由这个函数决定。

法线分布函数用的是filament实现的GGX各向异性[^1]算法：

[^1]: Burley B, Studios W D A. Physically-based shading at disney[C]//ACM SIGGRAPH. 2012, 2012: 1-7.

```c
float D_GGX_Anisotropic(float at, float ab, float ToH, float BoH, float NoH)
{
    float a2 = at * ab;
    float3 d = float3(ab * ToH, at * BoH, a2 * NoH);
    float d2 = dot(d, d);
    float b2 = a2 / d2;
    return a2 * b2 * b2 * (1 / UNITY_PI);
}
```
其中at，ab指的是切向（tangent）、副切向（bitangent）的粗糙度，为了避免麻烦，可以仅用一个各向异性参数来计算at和ab[^2]:
```c
float at = max(roughness * (1 + _Anisotropy), 0.001);
float ab = max(roughness * (1 - _Anisotropy), 0.001);
```
[^2]: Kulla C, Conty A. Revisiting physically based shading at Imageworks[J]. SIGGRAPH Course, Physically Based Shading, 2017.

各向异性在1到-1之间调节，分别可以在切向和副切向上延展高光。

遮蔽函数也有很多实现，现在使用最多的是Smith函数[^3]，有趣的是，这个函数的高度相关版本可以和微表面模型中的分母结合起来优化，于是用新的函数Vis来指代这一优化项：

[^3]: Heitz E. Understanding the masking-shadowing function in microfacet-based BRDFs[J]. 2014.

$$
Vis = \frac{G}{4|n\cdot l||n\cdot v|}
$$

所以

$$
f_{spec} = F\cdot V(Vis) \cdot D
$$

实现如下：
```c
float V_SmithGGXCorrelated_Anisotropic(float at, float ab, float ToV, float BoV, float ToL, float BoL, float NoV, float NoL)
{
    float lambdaV = NoL * length(float3(at * ToV, ab * BoV, NoV));
    float lambdaL = NoV * length(float3(at * ToL, ab * BoL, NoL));
    float v = 0.5 / (lambdaV + lambdaL);
    return v;
}
```

最后需要处理的就是fresnel函数，这里的实现比较普遍[^4]。

[^4]: Schlick C. An inexpensive BRDF model for physically‐based rendering[C]//Computer graphics forum. Edinburgh, UK: Blackwell Science Ltd, 1994, 13(3): 233-246.

```c
float3 F_Schlick(float3 f0, float VoH)
{
    float f = pow(1.0 - VoH, 5.0);
    return f + f0 * (1.0 - f);
}
```

## 在Unity里面实现自己的PBR框架

在我写这篇博客的过程中，其实没有很正规的做金属度在漫反射和镜面反射之间的系数调节映射，这个我之后加上。

### 数据结构

在传入顶点着色器的appdata结构中添加法线和切线，unity就会将mesh中的对应信息传入。
```c
struct appdata
{
    float4 vertex: POSITION;
    float2 uv: TEXCOORD0;
    float3 normal: NORMAL;
    float4 tangent: TANGENT;
};

struct v2f
{
    float2 uv: TEXCOORD0;
    float4 vertex: SV_POSITION;
    float3 normal: TEXCOORD1;
    float3 tangent: TEXCOORD2;
    float3 bitangent: TEXCOORD3;
    float4 wPos: TEXCOORD4;
};
```
### vert
```c
v2f vert(appdata v)
{
    v2f o;
    // UNITY_INITIALIZE_OUTPUT(v2f, o);
    o.vertex = UnityObjectToClipPos(v.vertex);
    o.uv = TRANSFORM_TEX(v.uv, _MainTex);
    o.normal = UnityObjectToWorldNormal(v.normal);
    o.tangent = UnityObjectToWorldDir(v.tangent.xyz);
    o.bitangent = UnityObjectToWorldDir(cross(v.normal, v.tangent) * v.tangent.w);
    o.wPos = mul(unity_ObjectToWorld, v.vertex);
    return o;
}
```
### frag

```c
fixed4 frag(v2f i): SV_Target
{
    float3 N = normalize(i.normal);
    float3 T = normalize(i.tangent);
    float3 B = normalize(i.bitangent);
    float3 L = normalize(UnityWorldSpaceLightDir(i.wPos));
    float3 V = normalize(UnityWorldSpaceViewDir(i.wPos));
    float3 H = normalize(L + V);
    
    float NoH = dot(N, H); float ToH = dot(T, H); float BoH = dot(B, H);
    float NoL = dot(N, L); float ToL = dot(T, L); float BoL = dot(B, L);
    float NoV = dot(N, V); float ToV = dot(T, V); float BoV = dot(B, V);
    float VoH = dot(V, H);
    
    float roughness = _Roughness * _Roughness;
    
    float at = max(roughness * (1 + _Anisotropy), 0.001);
    float ab = max(roughness * (1 - _Anisotropy), 0.001);
    
    float NDF = D_GGX_Anisotropic(at, ab, ToH, BoH, NoH);
    float Vis = V_SmithGGXCorrelated_Anisotropic(at, ab, ToV, BoV, ToL, BoL, NoV, NoL);
    float3 Fresnel = F_Schlick(_SpecularColor, VoH);
    
    float3 specularLobe = NDF * Fresnel;
    float3 diffuseColor = tex2D(_MainTex, i.uv) * _Tint * (1 / UNITY_PI);
    
    fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
    
    // sample the texture
    float4 col = float4(1, 1, 1, 1);
    col.xyz *= (diffuseColor + specularLobe + ambient) * _LightColor0.rgb * NoL;
    return col;
}
```

# 最后的效果
![渲染出来的效果](/assets/images/renderedSatin.jpg)