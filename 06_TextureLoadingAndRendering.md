# 06 Texture Loading And Rendering

## 前言

在此之前我们一直在使用CPU进行图像的处理，但在当下这个用图形加速器（或者说GPU）进行图像处理的时代，CPU处理画面的速度实在是太慢了。这一章，我们会将如何让SDL用GPU来处理图片，即如何使用SDL提供的纹理和渲染功能。

> **纹理**是一个二维或三维的图像，用于在三维或二维物体表面添加细节，比如颜色、图案、粗糙度等。它是实现物体外观多样性的关键。
> **渲染**是将场景中的所有信息（几何、材质、纹理、光照等）转换为二维图像的过程，是生成最终图像的步骤。

## 代码展示

```cpp
#include <SDL.h>
#include <iostream>

namespace frame {
	SDL_Window* window;
	SDL_Renderer* render;
	SDL_Texture* texture;
	const int width = 800;
	const int height = 600;
}

void Init() {
	SDL_Init(SDL_INIT_VIDEO);
	SDL_SetHint(SDL_HINT_RENDER_SCALE_QUALITY, "1");
	frame::window = SDL_CreateWindow("Render", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, frame::width, frame::height, SDL_WINDOW_SHOWN);
	frame::render = SDL_CreateRenderer(frame::window, -1, 0);
	SDL_SetRenderDrawColor(frame::render, 0XFF, 0XFF, 0X00, 0XFF);
}

void LoadTexture(const char* file) {
	SDL_Surface* temp;
	temp = SDL_LoadBMP(file);
	frame::texture = SDL_CreateTextureFromSurface(frame::render, temp);
	SDL_FreeSurface(temp);
}

void Quit() {
	SDL_DestroyRenderer(frame::render);
	SDL_DestroyTexture(frame::texture);
	SDL_DestroyWindow(frame::window);
}

int main(int argc, char* args[]) {
	Init();
	LoadTexture("./hello_world.bmp");
	SDL_Event event;
	while (!(SDL_PollEvent(&event) && event.type == SDL_QUIT)) {
		SDL_RenderClear(frame::render);
		SDL_RenderCopy(frame::render, frame::texture, nullptr, nullptr);
		SDL_RenderPresent(frame::render);
	}
	Quit();
	return 0;
}
```

## 代码讲解

因为是第一次接触到SDL的渲染功能，所以我们这里不会像官网那样将一堆的功能混杂到一起全部呈现出来。我们这里仅仅呈现出与渲染相关的主要代码。并且为了让大家更容易看清整个结构，我们去除了所有的错误判断分支。用渲染功能的流程大致为：

- 设置渲染相关提示
- 在特定的窗口上绑定渲染
- 设置渲染色彩
- 获取`Texture`结构
- 将`Texture`通过渲染加载至对应窗口。

`SDL_Renderer` 使用GPU加速渲染，通过后台调用图形API（如OpenGL、Direct3D、Metal或Vulkan），极大提升了绘图性能。且提供了一些直接由GPU支持的高级渲染功能。支持设置自定义的渲染目标（`SDL_SetRenderTarget`），可以将渲染内容绘制到纹理中，而非直接绘制到屏幕。


## 函数讲解

### 目录

[SDL_SetHint](#sdl_sethint)
[SDL_CreateRenderer](#sdl_createrenderer)
[SDL_DestroyRenderer](#sdl_destroyrenderer)
[SDL_SetRenderDrawColor](#sdl_setrenderdrawcolor)
[SDL_CreateTextureFromSurface](#sdl_createtexturefromsurface)
[SDL_RenderClear](#sdl_renderclear)
[SDL_RenderCopy](#sdl_rendercopy)
[SDL_RenderPresent](#sdl_renderpresent)
[SDL_SetRenderTarget](#sdl_setrendertarget)
[SDL_SetRenderDrawBlendMode](#sdl_setrenderdrawblendmode)
[SDL_CreateTexture](#sdl_createtexture)

### SDL_SetHint

<font color=orange>函数原型</font>

```cpp
SDL_bool SDL_SetHint(const char *name, const char *value);
```

<font color=orange>描述</font>

用于设置一个提示（hint）的值。这些提示是用于控制SDL库行为的全局变量，允许用户调整SDL在特定场景中的表现。例如，可以通过提示控制渲染驱动程序的选择或音频设备的初始化方式。

<font color=orange>输入</font>

`name` ：提示的名称，即要设置的变量名。它通常是一个字符串，SDL内部定义了一系列标准提示名称。
`value` ：提示的新值。这个值可以是特定的字符串（通常和提示的定义有关，常用 `name` `value` 对如下所示）。

|name|value|作用|
|:---:|:---:|:---:|
|`SDL_HINT_RENDER_DRIVER`|`direct3d` `opengl` `opengles2` `software`|指定渲染驱动程序|
|`SDL_HINT_RENDER_SCALE_QUALITY`|`0`|设置渲染器的纹理缩放质量。最近邻采样（低质量，速度快）|
|`SDL_HINT_RENDER_SCALE_QUALITY`|`1`|设置渲染器的纹理缩放质量。线性插值|
|`SDL_HINT_RENDER_SCALE_QUALITY`|`2`|设置渲染器的纹理缩放质量。各向异性过滤|
|`SDL_HINT_VIDEO_MINIMIZE_ON_FOCUS_LOSS`|`0` `1`|设置当窗口失去焦点时是否最小化窗口。`1` 为最小化（默认行为）。|

> 可以查看SDL文档或 `SDL_hints.h 文件`，了解所有支持的提示名称及其可能的值。

<font color=orange>输出</font>

返回值为 `SDL_TRUE`，表示提示被成功设置。
返回值为 `SDL_FALSE`，表示提示设置失败（可能是因为无法创建新提示或内存不足）。

### SDL_CreateRenderer

<font color=orange>函数原型</font>

```cpp
SDL_Renderer* SDL_CreateRenderer(SDL_Window *window, int index, Uint32 flags);
```

<font color=orange>描述</font>

用于创建并返回一个与特定窗口关联的2D渲染器。渲染器可以用来绘制纹理、线条、矩形等图形元素。渲染器的具体实现可能是基于硬件加速或软件模拟的。**注意，即便是从同一个窗口创建的`SDL_Render`也不是同一个。**

<font color=orange>输入</font>

`window` ：目标窗口的指针，渲染器将在该窗口上进行绘制。必须是一个有效的窗口对象。
`index` ：渲染驱动程序的索引，若为-1，SDL会自动选择第一个可用的驱动程序。若指定具体索引，则使用该索引对应的驱动程序（如Direct3D、OpenGL）。**索引对应于 SDL 支持的渲染器驱动列表中的位置。每个系统上可用的渲染器驱动可能不同。**
`flags` ：渲染器的标志，用于控制渲染器的行为。可以为0代表不设置任何标志，也**可以是以下值的组合**：

|值|含义|
|:---:|:---:|
|SDL_RENDERER_SOFTWARE|使用软件渲染|
|SDL_RENDERER_ACCELERATED|使用硬件加速|
|SDL_RENDERER_PRESENTVSYNC|启用垂直同步（VSync）|
|SDL_RENDERER_TARGETTEXTURE|允许渲染到纹理目标|

<font color=orange>输出</font>

返回值是一个指向创建的渲染器对象的指针。

### SDL_DestroyRenderer

<font color=orange>函数原型</font>

```cpp
void SDL_DestroyRenderer(SDL_Renderer *renderer);
```

<font color=orange>描述</font>

用于销毁由 `SDL_CreateRenderer` 或类似函数创建的渲染器，并释放其占用的所有相关资源。调用该函数后，渲染器指针不再有效。

<font color=orange>输入</font>

`render` ：要销毁的渲染器的指针。如果传递 `NULL` ，函数什么都不做。

<font color=orange>输出</font>

无输出。

### SDL_SetRenderDrawColor

<font color=orange>函数原型</font>

```cpp
int SDL_SetRenderDrawColor(SDL_Renderer *renderer, Uint8 r, Uint8 g, Uint8 b, Uint8 a);
```

<font color=orange>描述</font>

用于设置渲染器的绘图颜色。这种颜色会影响后续的绘图操作（如清除窗口、绘制线条或矩形等）。颜色由红、绿、蓝、透明度（RGBA）组成，分别使用 8 位整数表示。

<font color=orange>输入</font>

`render` ：渲染器的指针，用于绘制图形。必须是由 `SDL_CreateRenderer` 创建的有效渲染器。
`r` ：红色分量（0-255）。
`g` ：绿色分量（0-255）。
`b` ：蓝色分量（0-255）。
`a` ：透明度分量（0-255）。0表示完全透明，255表示完全不透明。

> 如果需要在支持透明度的纹理上绘制，确保渲染器标志包含 `SDL_RENDERER_TARGETTEXTURE` 并正确设置混合模式（`SDL_SetRenderDrawBlendMode`）。
> 

<font color=orange>输出</font>

返回值为0表示成功。返回值为负值表示失败，可以使用 `SDL_GetError()` 获取详细错误信息。

### SDL_CreateTextureFromSurface

<font color=orange>函数原型</font>

```cpp
SDL_Texture* SDL_CreateTextureFromSurface(SDL_Renderer *renderer, SDL_Surface *surface);
```

<font color=orange>描述</font>

 用于从现有的 `SDL_Surface` 创建一个与指定渲染器相关联的 `SDL_Texture` 对象。这个函数常用于将加载的位图或其他表面对象转换为可以加速渲染的纹理。`SDL_Surface` 的像素格式会被自动转换为渲染器支持的格式。**如果需要高效转换，可以预先确保表面的像素格式与渲染器的纹理格式匹配。创建纹理是一种开销较大的操作，建议只在初始化或资源加载阶段进行。**

<font color=orange>输入</font>

`renderer` ：渲染器的指针，表示纹理将关联到的渲染器。
`surface` ：要转换的表面对象。该对象包含了像素数据以及表面格式信息。

<font color=orange>输出</font>

成功时，返回指向新创建纹理的指针。
失败时，返回 `NULL`，可以使用 `SDL_GetError()` 获取错误信息。

### SDL_RenderClear

<font color=orange>函数原型</font>

```cpp
int SDL_RenderClear(SDL_Renderer *renderer);
```

<font color=orange>描述</font>

用于清除当前的渲染目标（通常是窗口或纹理），并用渲染器当前的绘图颜色（由 `SDL_SetRenderDrawColor` 设置）填充整个目标。通常在开始绘制新帧时使用。**如果使用 `SDL_SetRenderTarget` 将渲染目标设置为纹理，则 `SDL_RenderClear` 会清除该纹理，而非窗口。**

<font color=orange>输入</font>

`renderer` ：渲染器的指针，用于操作目标。该渲染器必须是有效的并已关联到一个窗口或目标纹理。

<font color=orange>输出</font>

返回值为0表示成功。
返回负值表示失败，可以调用 `SDL_GetError()` 获取详细错误信息。

### SDL_RenderCopy

<font color=orange>函数原型</font>

```cpp
int SDL_RenderCopy(SDL_Renderer *renderer, SDL_Texture *texture, const SDL_Rect *srcrect, const SDL_Rect *dstrect);
```

<font color=orange>描述</font>

用于将一个纹理渲染到指定的矩形区域。这是一个常用的函数，用于将图像（纹理）绘制到屏幕或其他渲染目标上。

<font color=orange>输入</font>

`renderer` ：渲染器的指针，指定了要使用哪个渲染器进行渲染。
`texture` ：要渲染的纹理。
`srcrect` ：指定纹理的源区域（即从纹理的哪一部分进行绘制）。如果为 `NULL`，则表示使用纹理的整个区域。
`dstrect` ：指定目标区域（即将纹理渲染到窗口中的哪个位置以及大小）。如果为 `NULL`，则表示将纹理渲染到整个渲染目标。

<font color=orange>输出</font>

返回值为0表示成功。
返回负值表示失败，可以使用 `SDL_GetError()` 获取错误信息。

### SDL_RenderPresent

<font color=orange>函数原型</font>

```cpp
void SDL_RenderPresent(SDL_Renderer *renderer);
```

<font color=orange>描述</font>

用于将当前渲染器的内容呈现到屏幕上，通常用于将所有渲染操作的结果显示出来。它会将通过渲染器绘制的图形（例如，纹理、形状、颜色等）更新到窗口中。这个函数是渲染循环中的一个关键步骤，通常在所有渲染操作完成后调用。**如果渲染目标被设置为纹理或其他非默认目标，`SDL_RenderPresent` 将更新该目标的内容而非窗口屏幕。因此，确保在正确的渲染目标上调用该函数。**

> SDL使用双缓冲（Double Buffering）技术来避免屏幕闪烁。当调用 `SDL_RenderPresent` 时，当前渲染器的内容会从后备缓冲区复制到显示缓冲区，从而实现平滑的帧显示。

<font color=orange>输入</font>

`renderer` ：渲染器的指针，表示需要将内容呈现到的渲染器。

<font color=orange>输出</font>

无输出。

### SDL_SetRenderTarget

<font color=orange>函数原型</font>

```cpp
int SDL_SetRenderTarget(SDL_Renderer *renderer, SDL_Texture *target);
```

<font color=orange>描述</font>

用于设置渲染目标。默认情况下，SDL 渲染器的渲染目标是屏幕窗口（即最后通过 `SDL_RenderPresent` 更新的内容）。通过调用 `SDL_SetRenderTarget`，可以将渲染目标更改为一个纹理或其他可用于渲染的目标纹理。此函数允许在纹理上绘制，而不是直接绘制到屏幕上，从而实现离屏渲染（off-screen rendering）。

> 渲染目标纹理必须具有 `SDL_TEXTUREACCESS_TARGET` 访问权限，这样才能作为渲染目标使用。通过 `SDL_CreateTexture` 创建纹理时，需指定这一访问权限。

<font color=orange>输入</font>

`renderer` ：渲染器的指针，表示要进行渲染目标设置的渲染器。
`target` ：渲染目标纹理。**如果为 `NULL`，则表示渲染回默认的窗口（屏幕）。** 否则，渲染器将绘制到指定的纹理上。

<font color=orange>输出</font>

返回值为0表示成功。
返回负值表示失败，可以使用 `SDL_GetError()` 获取错误信息。


### SDL_SetRenderDrawBlendMode

<font color=orange>函数原型</font>

```cpp
int SDL_SetRenderDrawBlendMode(SDL_Renderer *renderer, SDL_BlendMode blendMode);
```

<font color=orange>描述</font>

用于设置渲染器的混合模式。混合模式决定了渲染时如何将新绘制的像素与现有像素进行合成，常用于控制透明度和混合效果。此函数影响的是后续渲染操作，直到调用该函数设置新的混合模式。

> `SDL_SetRenderDrawBlendMode` 设置的是渲染器的全局混合模式，影响后续所有的渲染操作，直到调用该函数设置新的模式。

<font color=orange>输入</font>

`renderer` ：渲染器的指针，指定要设置混合模式的渲染器。
`blendMode` ：渲染混合模式，类型为枚举 `SDL_BlendMode`。常见的混合模式有：

|值|含义|
|:---:|:---:|
|SDL_BLENDMODE_NONE|不使用混合，完全覆盖|
|SDL_BLENDMODE_BLEND|默认的透明混合模式，按透明度混合源和目标像素|
|SDL_BLENDMODE_ADD|颜色加法混合，将源颜色与目标颜色相加，通常用于发光效果|
|SDL_BLENDMODE_MOD|乘法混合，将源颜色与目标颜色相乘，通常用于阴影效果|
|SDL_BLENDMODE_MULTIPLY|另一种乘法模式，也用于暗化效果|

<font color=orange>输出</font>

返回值为0表示成功。
返回负值表示失败，可以使用 `SDL_GetError()` 获取错误信息。

### SDL_CreateTexture

<font color=orange>函数原型</font>

```cpp
SDL_Texture* SDL_CreateTexture(SDL_Renderer *renderer, Uint32 format, int access, int w, int h);
```

<font color=orange>描述</font>

用于创建一个新的纹理对象，并将其与指定的渲染器关联。纹理可以用于存储图像数据，并在渲染时被映射到屏幕上。通过设置纹理的格式、访问模式和大小，开发者可以定义纹理的属性。这是创建渲染目标或加载图像数据的基础。

<font color=orange>输入</font>

`renderer` ：渲染器的指针，指定与之关联的渲染器。纹理将与此渲染器进行交互。

> 创建纹理时，传入的渲染器指定了纹理的初始状态，并为该纹理选择合适的像素格式和硬件支持，或者说决定了纹理的上下文。**`SDL_Texture` 和创建它的 `SDL_Renderer` 是强绑定的，渲染器的生命周期直接决定了纹理的生命周期。一个`SDL_Render`创建的纹理无法被其他`SDL_Render`使用，虽然其依旧存在于内存中。**

`format` ：纹理的像素格式，指定纹理中每个像素的存储方式。常见的格式有：

|值|含义|
|:---:|:---:|
|SDL_PIXELFORMAT_RGBA8888|32位像素格式，每个通道占8位|
|SDL_PIXELFORMAT_RGB24|24位像素格式，没有透明度|
|SDL_PIXELFORMAT_ABGR8888|类似于 RGBA8888，但颜色通道顺序不同|

`access` ：纹理的访问模式，定义纹理是否可以被直接修改。常见的访问模式有：

|值|含义|
|:---:|:---:|
|SDL_TEXTUREACCESS_STATIC|纹理内容不会在渲染期间改变，适用于静态图像|
|SDL_TEXTUREACCESS_STREAMING|纹理的内容可以动态更新，适用于动态图像|
|SDL_TEXTUREACCESS_TARGET|纹理用作渲染目标，可以渲染到该纹理上|

> 渲染期间指的是在调用渲染命令和更新屏幕之间的过程，主要包括以下几个阶段：
> **准备渲染**：当你调用 SDL_RenderClear 清除屏幕时，或者设置绘制颜色和其他渲染状态时，渲染器处于准备渲染的状态。这时，渲染器会根据设置的渲染状态准备图像内容。
> **绘制图像**：调用 SDL_RenderCopy 或其他绘制函数时，渲染器会从纹理中读取图像数据，并按照渲染状态将其绘制到渲染目标（如屏幕或目标纹理）上。
> **更新显示**：最后，调用 SDL_RenderPresent 会将渲染器的内容提交到显示器上，完成渲染过程。
> 所以，渲染期间指的是从你开始渲染操作（例如，绘制到屏幕或目标纹理）到实际显示结果的整个过程。

`w` ：纹理的宽度，以像素为单位。
`h` ：纹理的高度，以像素为单位。

<font color=orange>输出</font>

返回创建的纹理指针，成功时返回纹理对象。
如果创建纹理失败，则返回 `NULL`，可以使用 `SDL_GetError()` 获取错误信息。