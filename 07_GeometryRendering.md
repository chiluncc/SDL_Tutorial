# Geometry Rendering

## 前言

到这一章节才发现，`SDL_Init` 并非必须的。SDL的原始渲染允许您渲染形状而无需加载特殊图形，因此不需要 `SDL_Init`。本章唯一需要注意的是SDL与OpenGL的坐标系不同，其坐标系与Windows平台窗口的坐标系一致，即从左向右x递增，从上向下y递增。

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
	//SDL_Init(SDL_INIT_VIDEO);
	SDL_SetHint(SDL_HINT_RENDER_SCALE_QUALITY, "1");
	frame::window = SDL_CreateWindow("Render", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, frame::width, frame::height, SDL_WINDOW_SHOWN);
	frame::render = SDL_CreateRenderer(frame::window, -1, 0);
	SDL_SetRenderDrawColor(frame::render, 0XFF, 0XFF, 0X00, 0XFF);
}

void DrawTexture() {
	SDL_Rect fillRect;
	frame::texture = SDL_CreateTexture(frame::render, SDL_PIXELFORMAT_RGBA8888, SDL_TEXTUREACCESS_TARGET, 800, 600);
	SDL_SetRenderTarget(frame::render, frame::texture);
	SDL_RenderClear(frame::render);
	
	fillRect = { frame::width / 4, frame::height / 4, frame::width / 2, frame::height / 2 };
	SDL_SetRenderDrawColor(frame::render, 0XFF, 0X00, 0X00, 0XFF);
	SDL_RenderFillRect(frame::render, &fillRect);
	
	SDL_SetRenderDrawColor(frame::render, 0X00, 0X00, 0X000, 0XFF);
	for (int i = 0; i < frame::width; i += 4) {
		SDL_RenderDrawPoint(frame::render, frame::width / 2, i);
	}

	SDL_SetRenderTarget(frame::render, nullptr);
	SDL_SetRenderDrawColor(frame::render, 0XFF, 0XFF, 0X00, 0XFF);
}

void Quit() {
	SDL_DestroyRenderer(frame::render);
	SDL_DestroyTexture(frame::texture);
	SDL_DestroyWindow(frame::window);
}

int main(int argc, char* args[]) {
	Init();
	DrawTexture();
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

可以看到，我们的代码和原教程的代码差距还是很大的。~~虽然实现的功能一致。~~ 本代码将需要不断渲染的图片在渲染之前就进行了绘制，这是考虑不断地切换渲染器填充颜色会导致性能急速下降。另外必须说明，`SDL_Texture`在开始创建时其颜色为黑色，而非默认填充颜色。
在这里我们顺便说下上一章一些遗留问题：

- `SDL_TEXTUREACCESS_STATIC`静态纹理

纹理的内容一旦创建就不可更改。用于加载固定的图像或在创建时初始化纹理内容。**这种类型的纹理通常用于加载固定图像资源，如通过 `SDL_CreateTextureFromSurface` 获取。**

- `SDL_TEXTUREACCESS_STREAMING`动态纹理

允许在渲染期间更新纹理的像素数据。适用于需要频繁修改或更新纹理内容的场景，例如视频流、动态纹理更新或游戏中的实时图像处理。使用 `SDL_LockTexture` 锁定纹理，将其内容加载到内存中，然后修改像素数据（使用 `SDL_UpdateTexture` ）。修改完成后，**使用 `SDL_UnlockTexture` 解锁纹理，这样更新的数据才会反映到纹理中。**

## 函数讲解

### 目录

[SDL_RenderFillRect](#sdl_renderfillrect)
[SDL_RenderDrawPoint](#sdl_renderdrawpoint)
[SDL_RenderDrawLine](#sdl_renderdrawline)
[SDL_UpdateTexture](#sdl_updatetexture)
[SDL_LockTexture](#sdl_locktexture)
[SDL_UnlockTexture](#sdl_unlocktexture)

### SDL_RenderFillRect

<font color=orange>函数原型</font>

```cpp
int SDL_RenderFillRect(SDL_Renderer *renderer, const SDL_Rect *rect);
```

<font color=orange>描述</font>

用于在当前渲染目标上绘制一个填充矩形。矩形的大小和位置由 `SDL_Rect` 结构体指定。此函数将矩形区域用当前渲染器的绘制颜色进行填充，并且绘制操作会立即生效。渲染器的当前颜色（通过 `SDL_SetRenderDrawColor` 设置）决定填充矩形的颜色。**如果 `rect` 为 `NULL`，则绘制整个渲染目标（通常是屏幕或窗口）。**

<font color=orange>输入</font>

`renderer` ：指定在哪个渲染器上执行绘制操作。
`rect` ：指向 `SDL_Rect` 结构体的指针，定义了矩形的位置和大小。**如果矩形区域超出渲染目标的范围，SDL 会自动裁剪矩形，确保它只在渲染目标的有效区域内显示。**

<font color=orange>输出</font>

返回0表示成功，返回-1表示失败，并且可以通过 `SDL_GetError()` 获取错误信息。

### SDL_RenderDrawPoint

<font color=orange>函数原型</font>

```cpp
int SDL_RenderDrawPoint(SDL_Renderer *renderer, int x, int y);
```

<font color=orange>描述</font>

用于在当前渲染目标上绘制一个单独的像素点。该函数使用当前渲染器的绘制颜色来绘制一个点。如果渲染目标支持像素操作（例如窗口或纹理），该点将被绘制到指定的 (x, y) 坐标处。

<font color=orange>输入</font>

`renderer` ：指向 SDL_Renderer 对象的指针，指定在哪个渲染器上执行绘制操作。
`x` ：点的 x 坐标，表示在渲染目标上的位置。
`y` ：点的 y 坐标，表示在渲染目标上的位置。

<font color=orange>输出</font>

返回0表示成功，返回-1表示失败，并且可以通过 `SDL_GetError()` 获取错误信息。

### SDL_RenderDrawLine

<font color=orange>函数原型</font>

```cpp
int SDL_RenderDrawLine(SDL_Renderer *renderer, int x1, int y1, int x2, int y2);
```

<font color=orange>描述</font>

用于在当前渲染目标上绘制一条从 (x1, y1) 到 (x2, y2) 的直线。直线的颜色由当前渲染器的绘制颜色决定。

<font color=orange>输入</font>

`renderer` ：指向 `SDL_Renderer` 对象的指针，指定在哪个渲染器上执行绘制操作。
`x1` ：直线起始点的 x 坐标。
`y1` ：直线起始点的 y 坐标。
`x2` ：直线终止点的 x 坐标。
`y2` ：直线终止点的 y 坐标。

<font color=orange>输出</font>

返回0表示成功，返回-1表示失败，并且可以通过 `SDL_GetError()` 获取错误信息。

### SDL_UpdateTexture

<font color=orange>函数原型</font>

```cpp
int SDL_UpdateTexture(SDL_Texture *texture, const SDL_Rect *rect, const void *pixels, int pitch);
```

<font color=orange>描述</font>

用于更新现有纹理的像素数据。你可以通过此函数将一个内存缓冲区中的像素数据更新到一个纹理上。它适用于动态纹理。

<font color=orange>输入</font>

`texture` ：指向需要更新的 `SDL_Texture` 对象的指针。
`rect` ：指向一个 `SDL_Rect` 结构的指针，定义了更新区域的位置和大小。如果为 `NULL`，则更新整个纹理。
`pixels` ：指向包含新像素数据的内存缓冲区的指针。此数据将被更新到纹理的指定区域。
`pitch` ：**每一行像素的字节数（即一行的内存跨度）**，通常是纹理宽度乘以每个像素的字节数。如果使用 `SDL_PIXELFORMAT_RGBA8888` 格式，pitch 通常是纹理宽度的四倍。

<font color=orange>输出</font>

返回0表示成功，返回-1表示失败，并且可以通过 `SDL_GetError()` 获取错误信息。

### SDL_LockTexture

<font color=orange>函数原型</font>

```cpp
int SDL_LockTexture(SDL_Texture *texture, const SDL_Rect *rect, void **pixels, int *pitch);
```

<font color=orange>描述</font>

用于锁定纹理的像素数据，使你能够直接修改纹理内容。该函数将纹理的数据暴露给你，允许你在内存中直接操作纹理的像素。锁定后，你可以通过返回的指针 (pixels) 访问纹理的内存数据。**在修改完数据后，必须调用 `SDL_UnlockTexture` 来解锁纹理。**

<font color=orange>输入</font>

`texture` ：指向要锁定的 `SDL_Texture` 对象的指针。
`rect` ：指向一个 `SDL_Rect` 结构的指针，指定锁定的区域。如果为 `NULL` ，则锁定整个纹理。
`pixels` ：指向 `void*` 指针的指针，返回锁定纹理的数据的地址。你可以通过此指针访问和修改纹理的像素。
`pitch` ：指向整数的指针，返回每行像素的字节数（即纹理的“跨度”）。

<font color=orange>输出</font>

返回0表示成功，返回-1表示失败，并且可以通过 `SDL_GetError()` 获取错误信息。

### SDL_UnlockTexture

<font color=orange>函数原型</font>

```cpp
void SDL_UnlockTexture(SDL_Texture *texture);
```

<font color=orange>描述</font>

用于解锁一个之前使用 `SDL_LockTexture` 锁定的纹理。当你完成了对纹理的像素数据修改后，必须调用 `SDL_UnlockTexture` 来解锁纹理，这样它就可以被渲染器使用，并且后续的渲染操作才会生效。

<font color=orange>输入</font>

`texture` ：指向被锁定的 `SDL_Texture` 对象的指针。这个纹理必须是在之前通过 `SDL_LockTexture` 锁定的。

<font color=orange>输出</font>

无输出。
