# Extension Libraries and Loading Other Image Formats

## 前言

SDL库本身还是比较偏向底层的，并且没有提供一些更为方便的函数用于和音乐、字体、图片进行交互。所以我们不得不借助各种额外的库，例如`SDL_image`、`SDL_ttf`、`SDL_mixer`等。在本章我们会讲解一个简单的额外库，`SDL_image`。

## 代码展示

```cpp
#include <SDL.h>
#include <SDL_image.h>
#include <iostream>

struct {
public:
	void Init() {
		SDL_Init(SDL_INIT_VIDEO);
		IMG_Init(IMG_INIT_PNG);
		window = SDL_CreateWindow("Hello", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, width, height, SDL_WINDOW_SHOWN);
	}
	void LoadImage() {
		SDL_Surface* image, * surface, * convertImage;
		surface = SDL_GetWindowSurface(window);
		image = IMG_Load("./gkw.png");
		convertImage = SDL_ConvertSurface(image, surface->format, 0);
		SDL_FreeSurface(image);
		SDL_BlitSurface(convertImage, nullptr, surface, nullptr);
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

代码结构没什么好解释的，我们还是直接进行函数的讲解吧。

## 函数讲解

### 目录

[IMG_Init](#img_init)
[IMG_Quit](#img_quit)
[IMG_Load](#img_load)
[IMG_LoadPNM_RW](#img_loadpng_rw)
[IMG_GetError](#img_geterror)
[IMG_InitFlags](#img_initflags)

### IMG_Init

<font color=orange>函数原型</font>

```cpp
int IMG_Init(int flags);
```

<font color=orange>描述</font>

初始化 `SDL_image` 库，指定支持的图像格式。该函数通常在程序启动时调用，且必须在加载任何图像之前进行初始化。如果没有初始化 `SDL_image` ，就无法正确加载各种图像格式。

<font color=orange>输入</font>

`flags` ：一个位掩码，指定支持的图像格式。**可用`|`操作符进行拼接**，具体掩码可见[IMG_InitFlags](#img_initflags)。

<font color=orange>输出</font>

返回一个整数，**这个整数是位掩码的组合，表示成功初始化的图像格式类型**。它的值是传入 `flags` 参数的子集，表示成功初始化的图像格式。可以通过 `IMG_GetError()` 获取错误信息。

### IMG_Quit

<font color=orange>函数原型</font>

```cpp
void IMG_Quit(void);
```

<font color=orange>描述</font>

用于关闭 `SDL_image` 库，释放 `SDL_image` 库所占用的资源。它不接受任何参数。

<font color=orange>输入</font>

无输入。

<font color=orange>输出</font>

无输出。

### IMG_Load

<font color=orange>函数原型</font>

```cpp
SDL_Surface* IMG_Load(const char* filename);
```

<font color=orange>描述</font>

读取指定路径的图像文件，并将其加载为一个 `SDL_Surface` 对象。加载后的图像可以被用来在 SDL 窗口上显示，或者进行其他操作，如处理图像数据或创建纹理。

<font color=orange>输入</font>

`filename` ：一个指向字符串的指针，表示要加载的图像文件的路径和文件名。路径可以是相对路径或者绝对路径。支持多种格式，如PNG、JPEG等。

<font color=orange>输出</font>

成功时：返回一个指向 `SDL_Surface` 的指针，表示加载的图像。返回的 `SDL_Surface` 包含图像数据，可以用来显示或处理。
失败时：返回 `NULL`，并且可以通过 `IMG_GetError()` 获取错误信息。

### IMG_GetError

<font color=orange>函数原型</font>

```cpp
const char* IMG_GetError(void);
```

<font color=orange>描述</font>

不接受任何参数，它会返回一个常量字符串，表示最近的 `SDL_image` 错误信息。如果没有发生错误，返回的字符串为空字符串 `""`。

<font color=orange>输入</font>

无输入。

<font color=orange>输出</font>

如果没有错误发生，返回空字符串 `""`。失败时，返回一个描述错误的字符串，通常是有关图像加载失败或不支持图像格式等的详细描述。

### IMG_LoadPNG_RW

<font color=orange>函数原型</font>

```cpp
SDL_Surface* IMG_LoadPNG_RW(SDL_RWops* src);
```

<font color=orange>描述</font>

通过一个 `SDL_RWops` 对象来加载 `PNG` 格式的图像。

> `SDL_RWops*` 是 SDL 提供的统一I/O接口，用于处理文件、内存等各种数据源的读写操作。它的作用是让开发者能够通过一致的API操作不同的I/O来源，不需要关心底层细节。
> `SDL_RWops` 操作的数据本质上是二进制数据，并且通常被视为流数据。其操作抽象并不区分数据的具体格式，主要关注的是字节流的读写。因此，无论数据是字符串、二进制数据，还是复杂结构体，只要能以字节流的形式表示，就可以通过 `SDL_RWops` 进行操作。

<font color=orange>输入</font>

`src` ：指向 `SDL_RWops` 对象的指针。`SDL_RWops` 表示一个读取数据源的对象，可以是一个打开的文件、内存块或其他数据流。

<font color=orange>输出</font>

成功时：返回一个指向 `SDL_Surface` 的指针，表示加载的PNG图像。
失败时：返回 `NULL`，并且可以通过 `IMG_GetError()` 获取详细的错误信息。

## IMG_InitFlags

<font color=orange>结构体声明</font>

```cpp
typedef enum
{
    IMG_INIT_JPG    = 0x00000001,  // 支持 JPEG 格式
    IMG_INIT_PNG    = 0x00000002,  // 支持 PNG 格式
    IMG_INIT_TIF    = 0x00000004,  // 支持 TIFF 格式
    IMG_INIT_WEBP   = 0x00000008,  // 支持 WebP 格式
    IMG_INIT_JXL    = 0x00000010,  // 支持 JPEG XL 格式
    IMG_INIT_AVIF   = 0x00000020   // 支持 AVIF 格式
} IMG_InitFlags;
```

<font color=orange>描述</font>

该枚举包含了 `SDL_image` 所支持的不同图像格式的标志，允许开发者通过组合标志来初始化所需格式。使用按位或操作将不同的标志位组合在一起传递给 `IMG_Init` 函数，支持多种格式。