# 04 Optimized Surface Loading and Soft Stretching

## 前言

这一章涉及到两个函数，一个用于缩放的，另外一个用于转化格式的。我们废话少说，直接上代码。

## 代码展示

```cpp
#include <SDL.h>
#include <iostream>

struct {
public:
	void Init() {
		SDL_Init(SDL_INIT_VIDEO);
		window = SDL_CreateWindow("Hello", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, width, height, SDL_WINDOW_SHOWN);
	}
	void LoadImage() {
		SDL_Surface* image, * surface, * convertImage;
		surface = SDL_GetWindowSurface(window);
		image = SDL_LoadBMP("./hello_world.bmp");
		convertImage = SDL_ConvertSurface(image, surface->format, 0);
		SDL_FreeSurface(image);
		SDL_BlitScaled(convertImage, NULL, surface, &surface->clip_rect);
		SDL_UpdateWindowSurface(window);
	}
	bool Quit() {
		if (quit == true) {
			SDL_DestroyWindow(window);
			SDL_Quit();
			return true;
		}
		return false;
	}
	void PollEvent() {
		SDL_PollEvent(&event);
		if (event.type == SDL_QUIT) quit = true;
	}
private:
	SDL_Window* window;
	SDL_Event event;
	bool quit = false;
	const int width = 800, height = 600;
} SDL_Struct;


int main(int argc, char* argv[]) {
	SDL_Struct.Init();
	SDL_Struct.LoadImage();
	while (!SDL_Struct.Quit()) {
		SDL_Struct.PollEvent();
	}
	return 0;
}
```

## 代码讲解

首先代码里在 `SDL_BlitScaled` 部分耍了个小聪明，其实 `clip_rect` 不一定就是实际大小，还是新建一个 `SDL_Rect` 比较保险的。另外还需要注意一点， **无论是 `SDL_BlitScaled` 还是 `SDL_BlitSurface` 都不允许对同一个 `Surface` 进行操作，以及不能对 `nullptr` 的表面进行操作。** 前者会产生未定义行为（在win+vs环境中是直接变为白色），后者直接报错，请使用 `SDL_CreateRGBSurface` 创建新的空白表面。

## 函数讲解

### 目录

[SDL_ConvertSurface](#sdl_convertsurface)
[SDL_BlitScaled](#sdl_blitscaled)
[SDL_CreateRGBSurface](#sdl_creatergbsurface)

### SDL_ConvertSurface

<font color=orange>函数原型</font>

```cpp
SDL_Surface* SDL_ConvertSurface(SDL_Surface* src, const SDL_PixelFormat* fmt, Uint32 flags);
```

<font color=orange>描述</font>

通过转换源表面的像素格式来创建一个新的表面。转换后的表面可以更高效地用于显示，例如匹配当前显示器的像素格式或优化透明度效果。生成的表面会使用与目标格式兼容的像素格式，从而减少显示时的像素转换开销。

<font color=orange>输入</font>

`src` ：指向要转换的源表面的指针，即 SDL_Surface 结构体。
`fmt` ：指向 `SDL_PixelFormat` 结构体的指针，表示要转换到的目标像素格式。可以使用当前显示器的 `SDL_GetWindowSurface` 获取的格式来确保兼容。
`flags` ：表面标志。通常设置为0，该参数是为未来扩展设计的，目前未使用。

<font color=orange>输出</font>

成功时：返回一个指向新 `SDL_Surface` 的指针，代表转换后的表面。
失败时：返回 `NULL` ，可以调用 `SDL_GetError()` 获取错误消息。

### SDL_BlitScaled

<font color=orange>函数原型</font>

```cpp
int SDL_BlitScaled(SDL_Surface* src, const SDL_Rect* srcrect, SDL_Surface* dst, SDL_Rect* dstrect);
```

<font color=orange>描述</font>

用于将 `src` 源表面的一部分（由 `srcrect` 指定的矩形区域）按比例缩放后，复制到 `dst` 目标表面的指定区域（由 `dstrect` 指定的矩形区域）。如果 `srcrect` 或 `dstrect` 为 `NULL` ，则会使用整个源表面或目标表面。

<font color=orange>输入</font>

`src` ：指向源 `SDL_Surface` 的指针，即要复制的表面。
`srcrect` ：指向 `SDL_Rect` 结构体的指针，指定源表面上要复制的矩形区域。如果为 `NULL` ，则表示复制整个源表面。
`dst` ：指向目标 `SDL_Surface` 的指针，即将图像复制到的表面。
`dstrect` ：指向 `SDL_Rect` 结构体的指针，指定目标表面上图像要放置的位置和大小。为 `NULL` 则表示填满整个目标表面。

<font color=orange>输出</font>

成功时：返回0。
失败时：返回-1，可通过 `SDL_GetError()` 获取错误信息。

### SDL_CreateRGBSurface

<font color=orange>函数原型</font>

```cpp
SDL_Surface* SDL_CreateRGBSurface(Uint32 flags, int width, int height, int depth, Uint32 Rmask, Uint32 Gmask, Uint32 Bmask, Uint32 Amask);

```

<font color=orange>描述</font>

创建一个指定大小和像素格式的空白表面。表面创建完成后，可以将图像数据直接绘制到表面上，或者将其用于渲染操作（例如复制到窗口）。创建的表面需要在使用完成后调用 `SDL_FreeSurface` 释放内存。

<font color=orange>输入</font>

`flags` ：通常设置为0。也可以设置为 `SDL_SWSURFACE` ，表示创建的软件渲染表面。
`width` ：表面的宽度（像素）。
`height` ：表面的高度（像素）。
`depth` ：颜色深度，表示每个像素的位数。常用值是8/16、24或32位。
`Rmask` , `Gmask` , `Bmask` , `Amask` ：分别是红色、绿色、蓝色和 Alpha 通道的掩码。通常情况下，这些掩码会根据设备的字节序自动设置。

> `SDL_CreateRGBSurface` 中指定的颜色掩码会直接影响 `SDL_Surface` 的 `format` ，即像素格式信息。具体而言，创建表面时传入的 `Rmask` 、 `Gmask` 、 `Bmask` 和 `Amask` 会被用于构建表面的 `SDL_PixelFormat` 结构体，并将该结构体赋值给表面的 `format` 字段。

<font color=orange>输出</font>

成功时:返回指向新创建的 `SDL_Surface` 的指针。
失败时:返回 `NULL` ，可以调用 `SDL_GetError()` 获取错误信息。

```cpp
//示例代码
Uint32 rmask, gmask, bmask, amask;
//判断大小端
//SDL_BYTEORDER 是一个宏，表示当前系统的字节序。
//SDL_BIG_ENDIAN 是 SDL 定义的大端字节序标志。
//SDL_LIL_ENDIAN 是 SDL 定义的小端字节序标志。
#if SDL_BYTEORDER == SDL_BIG_ENDIAN
	rmask = 0xff000000;
	gmask = 0x00ff0000;
	bmask = 0x0000ff00;
	amask = 0x000000ff;
#else
	rmask = 0x000000ff;
	gmask = 0x0000ff00;
	bmask = 0x00ff0000;
	amask = 0xff000000;
#endif

// 创建 RGB 表面
SDL_Surface* rgbSurface = SDL_CreateRGBSurface(0, 800, 600, 32, rmask, gmask, bmask, amask);
```
