# 00 Hello SDL

## 前言

啊B的文本专栏真的是越改越难用了，只好干脆放弃B站的专栏直接使用markdown了。说起来好几年前就有很多UP向B站反应，希望让专栏可以直接对接markdown，但是啊B一直都没能推出这一改进。 ~~果然专栏对于啊B就是个多余的板块呢。~~ 本专栏主要参照如下网址的教程。

<div align=center>

[https://lazyfoo.net/tutorials/SDL/index.php](https://lazyfoo.net/tutorials/SDL/index.php)

</div>

专栏内容为对该教程的复现、理解和进一步思考。 等等，我为什么要说专栏？

## 环境搭建

废话不多说，直接进入主题，环境搭建步骤就直接省略了，没什么好讲的。只要去GitHub下载SDL2提供的库文件就好了，如果你使用的是VS2024作为编辑器，那么就下载SDL2-devel-2.30.7-VC.zip这个文件，解压后在VS2024内添加对include和lib文件的包含，并添加所有的静态库，将动态库添加进系统全局变量或者放入.vcxproj文件同级目录即可，其实放在编译后的.exe同级目录下也是可以的。总之环境搭建的教程网上多的是，如果是看过我的OpenGL环境搭建专栏的朋友，我相信就算不看其他教程也能自己搭建起来。所以现在让我们跳过这一步！

## 代码展示

```cpp
#include <SDL.h>
#include <iostream>

const int SCREEN_WIDTH = 640;
const int SCREEN_HEIGHT = 480;

int main(int argc, char* args[]) {
	SDL_Window* window = NULL;
	SDL_Surface* screenSurface = NULL;
	if (SDL_Init(SDL_INIT_VIDEO) < 0) {
		std::cout << "SDL counld not init! " << SDL_GetError() << std::endl;
		exit(0);
	}

	else {
		window = SDL_CreateWindow("SDL", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, SCREEN_WIDTH, SCREEN_HEIGHT, SDL_WINDOW_SHOWN);
		if (window == NULL) {
			std::cout << "window counld not be created! " << SDL_GetError() << std::endl;
		}

		else {
			screenSurface = SDL_GetWindowSurface(window);
			SDL_FillRect(screenSurface, NULL, SDL_MapRGB(screenSurface->format, 0xEF, 0xEF, 0xEF));
			SDL_UpdateWindowSurface(window);
			SDL_Event e;
			bool quit = false;
			while (quit == false) {
				while (SDL_PollEvent(&e)) {
					if (e.type == SDL_QUIT) {
						quit = true;
					}
				}
			}
		}
	}

	SDL_DestroyWindow(window);
	SDL_Quit();
	return 0;
}
```

## 代码讲解

额，我相信有看过我开头提到的那篇教程的一定已经发现我这段代码完全照抄教程的，好吧我承认，我又能咋办呢，只是个“Hello World”的程序，玩出花来可是会让新人秒退的，开头就还是朴实简单一些吧。
这段代码基本可以分为四个部分，分别为SDL的初始化（即SDL_Init部分）、窗口的初始化（即SDL_CreateWindow部分）、窗口缓冲的写入和更新（即SDL_UpdateWindowSurface部分）和SDL关闭（即SDL_DestroyWindow部分）。其中初始化阶段没什么好说的，我们还是说说其余阶段吧。窗口缓冲写入和更新这部分做的就是先捕获申请窗口的缓冲区，接着在缓冲区内画一个矩形，最后再更新缓冲区以显示经过绘制操作后窗口的画面。值得注意的是， **这里的缓冲区位于CPU而非GPU。** 而关闭阶段的双层while并不会造成死循环，内层循环的SDL_PollEvent会不断地将事件队列的队首事件写入e，并将该事件从事件队列中弹出，若事件队列为空则会返回0，使得内层循环结束。而在用户点击关闭按钮后内层循环将quit设置为true，这使得外层循环结束。两层循环都可以正常结束，那么死循环就不会发生。至于各个函数具体的作用，我们下一个模块详细讲解。

## 函数讲解

因为有了markdown这一强大的书写工具，我们这一栏也终于有了索引功能了，可喜可贺，可喜可贺。本专栏所有函数讲解会分为如下四部分：

* 函数原型
* 描述
* 输入
* 输出

### 目录

[SDL_Init](#sdl_init)
[SDL_CreateWindow](#sdl_createwindow)
[SDL_GetError](#sdl_geterror)
[SDL_GetWindowSurface](#sdl_getwindowsurface)
[SDL_FillRect](#sdl_fillrect)
[SDL_MapRGB](#sdl_maprgb)
[SDL_MapRGBA](#sdl_maprgba)
[SDL_UpdateWindowSurface](#sdl_updatewindowsurface)
[SDL_PollEvent](#sdl_pollevent)
[SDL_DestoryWindow](#sdl_destroywindow)
[SDL_Quit](#sdl_quit)

### SDL_Init

<font color=orange>函数原型</font>

```cpp
int SDL_Init(Uint32 flags)
```

<font color=orange>描述</font>

用于初始化 SDL 库的函数。它设置 SDL 运行所需的子系统，如视频、音频和事件处理等。通过指定不同的标志，开发者可以选择初始化哪些功能，从而为后续的 SDL 操作做好准备。成功初始化后，程序可以使用 SDL 提供的各种功能。

<font color=orange>输入</font>

```flags``` ：为一个位标志，用于指定要初始化的子系统。 **该值可以使用“|”运算符进行组合。** 其具体值可在.h文件中查看，常用值如下：

|值|含义|
|:---:|:---:|
|SDL_INIT_VIDEO|初始化视频子系统，允许创建窗口和渲染图形|
|SDL_INIT_AUDIO|初始化音频子系统，允许播放声音|
|SDL_INIT_TIMER|初始化定时器子系统，支持时间管理功能|
|SDL_INIT_EVENTS|初始化事件子系统，处理用户输入和其他事件|
|SDL_INIT_JOYSTICK|初始化游戏控制器子系统，支持手柄输入|
|SDL_INIT_HAPTIC|初始化触觉反馈子系统，支持振动功能|
|SDL_INIT_GAMECONTROLLER|初始化游戏控制器的扩展支持|

<font color=orange>输出</font>

成功时返回0，失败时返回一个负值，并可通过 SDL_GetError() 获取错误信息。

### SDL_CreateWindow

<font color=orange>函数原型</font>

```cpp
SDL_Window* SDL_CreateWindow(const char* title, int x, int y, int width, int height, Uint32 flags)
```

<font color=orange>描述</font>

用于创建一个新的窗口，供应用程序使用。

<font color=orange>输入</font>

```title``` ：窗口的标题字符串。
```x``` ：窗口的初始x坐标。
```y``` ：窗口的初始y坐标。其和x的值可以为指定的精确值，也可以为预设值，常用预设值如下：

|值|含义|
|:---:|:---:|
|SDL_WINDOWPOS_UNDEFINED|表示窗口的位置由系统决定，窗口将根据系统的默认行为进行定位|
|SDL_WINDOWPOS_CENTERED|表示窗口将在屏幕中心创建（仅适用于窗口的宽度和高度已知的情况）|

```width``` ：窗口的宽度（以像素为单位）。
```height``` ：窗口的高度（以像素为单位）。
```flags``` ：窗口的标志，可以控制窗口的属性，如是否可调整大小或是否为全屏。 **部分预设值可通过“|”运算符组合** ，其常用预设值如下：

|值|含义|
|:---:|:---:|
|SDL_WINDWO_SHOWN|窗口可见，创建后立即显示|
|SDL_WINDOW_HIDDEN|窗口创建后隐藏，之后可以通过代码显示|
|SDL_WINDOW_FULLSCREEN|创建全屏窗口|
|SDL_WINDOW_FULLSCREEN_DESKTOP|创建全屏窗口，使用桌面分辨率|
|SDL_WINDOW_OPENGL|创建OpenGL上下文的窗口，支持OpenGL渲染|
|SDL_WINDOW_RESIZABLE|允许用户调整窗口大小|
|SDL_WINDOW_BORDERLESS|创建无边框窗口，不显示标题、边框|
|SDL_WINDOW_MINIMIZED|窗口创建后立即最小化|
|SDL_WINDOW_MAXIMIZED|窗口创建后立即最大化|

<font color=orange>输出</font>

成功时返回一个指向 ```SDL_Window``` 的指针，失败时返回 ```NULL``` ，并可以通过 ```SDL_GetError()``` 获取错误信息。

### SDL_GetError

<font color=orange>函数原型</font>

```cpp
const char* SDL_GetError(void)
```

<font color=orange>描述</font>

用于获取 SDL 库中最近发生错误的描述信息。当调用某个 SDL 函数失败时，可以通过此函数获取详细的错误信息，帮助开发者进行调试和错误处理。这些错误信息是对导致函数失败的原因的简要描述。

<font color=orange>输入</font>

无输入

<font color=orange>输出</font>

返回一个指向字符串的指针，该字符串包含最后一个发生的错误的描述。如果没有错误发生，则返回一个空字符串。

### SDL_GetWindowSurface

<font color=orange>函数原型</font>

```cpp
SDL_Surface* SDL_GetWindowSurface(SDL_Window* window)
```

<font color=orange>描述</font>

用于获取与指定窗口关联的表面，该表面可以用于在窗口中进行低级别的像素操作和渲染。获取的表面可以通过直接操作像素数据来进行绘图，通常用于简单的2D图形渲染。 **请注意，表面是位于系统内存中的，而不是 GPU 内存。** 

<font color=orange>输入</font>

```window``` ：指向要获取表面的SDL_Window指针

<font color=orange>输出</font>

功时返回指向SDL_Surface的指针，该表面与指定窗口相关联；失败时返回NULL，并可通过 ```SDL_GetError()``` 获取错误信息。

### SDL_FillRect

<font color=orange>函数原型</font>

```cpp
int SDL_FillRect(SDL_Surface* dst, const SDL_Rect* rect, Uint32 color)
```

<font color=orange>描述</font>

用于在指定的表面上填充一个矩形区域，填充颜色由color参数指定。此函数常用于在渲染之前或更新屏幕时清除或填充背景。可以通过传递不同的矩形区域实现局部填充。

<font color=orange>输入</font>

```dst``` ：指向要填充的目标表面（ ```SDL_Surface``` ）的指针。
```rect``` ：指向要填充的矩形区域的指针，如果为NULL，则填充整个表面。可通过以下方式创建：

|创建语句|
|:---:|
|SDL_Rect rect; rect.x = 10, rect.y = 20; rect.w = 100; rect.h = 50|
|SDL_Rect rect = SDL_Rect{10, 20, 100, 50}|

```color``` ：要使用的填充颜色，以32位整数获取，可以通过 [SDL_MapRGB](#sdl_maprgb) 或者 [SDL_MapRGBA](#sdl_maprgba) 获取。

<font color=orange>输出</font>

成功时返回0，失败时返回负值，错误信息可通过 ```SDL_GetError()``` 获取。

### SDL_MapRGB

<font color=orange>函数原型</font>

```cpp
Uint32 SDL_MapRGB(const SDL_PixelFormat* format, Uint8 r, Uint8 g, Uint8 b)
```

<font color=orange>描述</font>

将 RGB 颜色值转换为适合特定像素格式的颜色值，便于在表面上使用。

<font color=orange>输入</font>

```format``` ：指向目标表面像素格式的指针，通常可以通过SDL_Surface的format成员获得。
```r``` ：红色分量（0-255）。
```g``` ：绿色分量（0-255）。
```b``` ：蓝色分量（0-255）。

<font color=orange>输出</font>

返回映射后的颜色值，以 ```Uint32``` 表示，适用于指定的像素格式。

### SDL_MapRGBA

<font color=orange>函数原型</font>

```cpp
Uint32 SDL_MapRGBA(const SDL_PixelFormat* format, Uint8 r, Uint8 g, Uint8 b, Uint8 a)
```

<font color=orange>描述</font>

将 RGB 颜色值转换为适合特定像素格式的颜色值，便于在表面上使用。

<font color=orange>输入</font>

```format``` ：指向目标表面像素格式的指针，通常可以通过SDL_Surface的format成员获得。
```r``` ：红色分量（0-255）。
```g``` ：绿色分量（0-255）。
```b``` ：蓝色分量（0-255）。
```a``` ：透明度分量（0-255）。（0：完全透明，255：完全不透明）

<font color=orange>输出</font>

返回映射后的颜色值，包括 alpha 通道，以 ```Uint32``` 表示。

### SDL_UpdateWindowSurface

<font color=orange>函数原型</font>

```cpp
int SDL_UpdateWindowSurface(SDL_Window* window)
```

<font color=orange>描述</font>

用于将指定窗口的表面内容更新到屏幕上。

<font color=orange>输入</font>

```window``` ：指向要更新的窗口的 ```SDL_Window``` 指针。

<font color=orange>输出</font>

成功时返回0，失败时返回负值，错误信息可通过 ```SDL_GetError()``` 获取。

### SDL_PollEvent

<font color=orange>函数原型</font>

```cpp
int SDL_PollEvent(SDL_Event* event)
```

<font color=orange>描述</font>

用于从事件队列中获取下一个事件。调用此函数后，事件信息会填充到 ```event``` 指向的结构中。该函数通常在主循环中使用，用于处理用户输入（如键盘、鼠标事件）和其他事件（如窗口状态变化）。 **与 ```SDL_WaitEvent``` 不同， ```SDL_PollEvent``` 是非阻塞的** ，如果没有事件，它会立即返回。

<font color=orange>输入</font>

```event``` ：指向 ```SDL_Event``` 结构的指针，用于存储事件信息。

<font color=orange>输出</font>

如果有事件可用，返回非零值（通常为1），否则返回0。其中 ```event.type``` 常见值如下：

|值|含义|
|:---:|:---:|
|SDL_QUIT|表示用户请求退出程序（如点击关闭窗口按钮）|
|SDL_KEYDOWN|表示按键被按下的事件|
|SDL_KEYUP|表示按键被释放的事件|
|SDL_MOUSEMOTION|表示鼠标移动事件|
|SDL_MOUSEBUTTONDOWN|表示鼠标按钮被按下的事件|
|SDL_MOUSEBUTTONUP|表示鼠标按钮被释放的事件|
|SDL_MOUSEWHEEL|表示鼠标滚轮滚动的事件|
|SDL_WINDOWEVENT|表示窗口状态变化的事件（如最小化、恢复等）|
|SDL_JOYAXISMOTION|表示游戏手柄轴的运动事件|
|SDL_JOYBUTTONDOWN|表示游戏手柄按钮被按下的事件|
|SDL_JOYBUTTONUP|表示游戏手柄按钮被释放的事件|

### SDL_DestroyWindow

<font color=orange>函数原型</font>

```cpp
void SDL_DestroyWindow(SDL_Window* window)
```

<font color=orange>描述</font>

用于销毁之前创建的窗口，释放与窗口相关的资源。当窗口不再需要时，应调用此函数以避免内存泄漏。销毁窗口后，该指针将变得无效，因此应该避免再访问已销毁的窗口。通常在应用程序结束或窗口关闭时调用。

<font color=orange>输入</font>

```window``` ：指向要销毁的SDL_Window指针。

<font color=orange>输出</font>

无输出。

### SDL_Quit

<font color=orange>函数原型</font>

```cpp
void SDL_Quit(void)
```

<font color=orange>描述</font>

用于清理 SDL 库所占用的所有资源，并关闭SDL子系统。在调用 ```SDL_Quit``` 之前，通常需要确保所有SDL创建的资源（如窗口、表面、音频等）都已被销毁。该函数在程序结束时调用，确保SDL正常退出并释放相关资源。

<font color=orange>输入</font>

无输入。

<font color=orange>输出</font>

无输出。