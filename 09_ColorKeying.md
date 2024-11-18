# Color Keying

## 前言

说实在的，为了一个“色键”概念大张旗鼓的用一大堆代码，真的听不推荐的。当时看原教程一长串代码，还以为介绍多少新的函数呢！原来就一个“色键”啊。我们这里就简单说下“色键”好了，本章就不给出代码展示了。

## 色键作用和特点

### 作用

<font color=orange>创建透明效果</font>

色键允许将纹理或表面中的某种特定颜色（通常是纯绿色、纯粉色等不常用的颜色）指定为透明色。在绘制该纹理时，这种颜色会被忽略，从而实现透明效果。

<font color=orange>创建透明效果</font>

在 2D 游戏中，色键是实现简单透明的高效方法，**相比于使用 alpha 通道，可以减少内存占用和计算复杂度。**

<font color=orange>处理图像边缘</font>

在资源制作中，常会为精灵图或纹理添加某种背景色（如纯绿色或纯黑色）。通过设置色键，可以去除这些背景色，只保留主体部分。

### 特点

**请注意色键是CPU计算的行为** 如果需要复杂的透明度（如半透明），应直接使用带 Alpha 通道的图像格式，而不是色键。色键是一种高效的简单透明处理方式，适用于资源优化和较低复杂度的透明效果。

## 函数讲解

### 目录

[SDL_SetColorKey](#sdl_setcolorkey)

### SDL_SetColorKey

<font color=orange>函数原型</font>

```cpp
int SDL_SetColorKey(SDL_Surface* surface, int flag, Uint32 key);
```

<font color=orange>描述</font>

为**表面（ `SDL_Surface` ）** 设置色键。`SDL_CreateTextureFromSurface` 会保留色键效果，将设置为色键的颜色转换为透明（ `Alpha` 值为 0）。

<font color=orange>输入</font>

`surface` ：要应用色键的 `SDL_Surface`。
`flag` ：其值如下所示。

|值|含义|
|:---:|:---:|
|SDL_TRUE|启用色键功能|
|SDL_FALSE|禁用色键功能|

`key` ：表示要设置为透明的颜色。这个颜色是通过 `SDL_MapRGB` 或 `SDL_MapRGBA` 将 `RGB` 值映射成像素格式后的值。

<font color=orange>输出</font>

返回0表示成功。
返回负值表示出错，可以通过 `SDL_GetError()` 获取错误信息。