# 10 Clip Rendering and Sprite Sheets

## 前言

本章会介绍可乐图，啊不，这里被称为精灵图。写过html、游戏或者其他应用的朋友一定很熟悉这个理念。即将大量需要的素材绘制在同一张图片上，在使用时通过截取对应区域来获得对应的图片。在本章我们会利用前面的很多函数做一点与教程完全不同的活！是的，更有意思，更灵活。

## 代码展示

```cpp
#include <iostream>
#include <string>
#include <SDL.h>
#include <SDL_image.h>

namespace Vars {
	SDL_Window* window[4];
	bool windowOpen[4] = { true, true, true, true };
	SDL_Renderer* render[4];
	SDL_Event event;

	const IMG_InitFlags initFlag = IMG_INIT_PNG;
	const unsigned int width = 800;
	const unsigned int height = 600;
}

class Texture {
public:
	Texture(const char* filePath) {
		SDL_Surface* pic = IMG_Load(filePath);
		SDL_SetColorKey(pic, SDL_TRUE, SDL_MapRGB(pic->format, 64, 64, 64));
		for (int i = 0; i < 4; i++) {
			texture[0] = nullptr;
		}
		ClipPic(pic);
		SDL_FreeSurface(pic);
	}
	void ClipPic(SDL_Surface* pic) {
		SDL_Rect rect[4];
		int width = pic->clip_rect.w, height = pic->clip_rect.h;

		rect[0] = {0, 0, width / 2, height / 2};
		rect[1] = { width / 2, 0, width / 2, height / 2 };
		rect[2] = { 0, height / 2, width / 2, height / 2 };
		rect[3] = { width / 2, height / 2, width / 2, height / 2 };

		for (int i = 0; i < 4; i++) {
			SDL_Texture* temp = SDL_CreateTextureFromSurface(Vars::render[i], pic);
			texture[i] = SDL_CreateTexture(Vars::render[i], SDL_PIXELFORMAT_RGBA8888, SDL_TEXTUREACCESS_TARGET, Vars::width / 2, Vars::height / 2);
			SDL_SetRenderTarget(Vars::render[i], texture[i]);
			SDL_RenderClear(Vars::render[i]);
			SDL_RenderCopy(Vars::render[i], temp, &rect[i], nullptr);
			SDL_RenderPresent(Vars::render[i]);
			SDL_SetRenderTarget(Vars::render[i], nullptr);
			SDL_DestroyTexture(temp);
		}

	}
	SDL_Texture* getPic(int index) { 
		return texture[index];
	}
	~Texture() {
		for (int i = 0; i < 4; i++) {
			SDL_DestroyTexture(texture[i]);
		}
	}
private:
	SDL_Texture* texture[4];
};

static void Init();
static void Quit();
static bool Close();

int main(int argc, char* args[]) {
	Init();
	Texture* textures = new Texture("./ABCD.png");
	while (!Close()) {
		for (int i = 0; i < 4; i++) {
			if (Vars::windowOpen[i]) {
				SDL_RenderClear(Vars::render[i]);
				SDL_RenderCopy(Vars::render[i], textures->getPic(i), nullptr, nullptr);
				SDL_RenderPresent(Vars::render[i]);
			}
		}
	}
	delete textures;
	Quit();
	return 0;
}

void Init() {
	SDL_Init(SDL_INIT_VIDEO);
	IMG_Init(Vars::initFlag);
	SDL_SetHint(SDL_HINT_RENDER_SCALE_QUALITY, "0");
	Vars::window[0] = SDL_CreateWindow("ColaPicA", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, Vars::width, Vars::height, SDL_WINDOW_SHOWN);
	Vars::window[1] = SDL_CreateWindow("ColaPicB", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, Vars::width, Vars::height, SDL_WINDOW_SHOWN);
	Vars::window[2] = SDL_CreateWindow("ColaPicC", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, Vars::width, Vars::height, SDL_WINDOW_SHOWN);
	Vars::window[3] = SDL_CreateWindow("ColaPicD", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, Vars::width, Vars::height, SDL_WINDOW_SHOWN);
	for (int i = 0; i < 4; i++) {
		Vars::render[i] = SDL_CreateRenderer(Vars::window[i], -1, 0);
		SDL_SetRenderDrawColor(Vars::render[i], 0XF0, 0XFF, 0X0F, 0XFF);
	}
}

void Quit() {
	for (int i = 0; i < 4; i++) {
		if (Vars::windowOpen[i]) {
			SDL_DestroyWindow(Vars::window[i]);
			SDL_DestroyRenderer(Vars::render[i]);
		}
	}
	SDL_Quit();
	IMG_Quit();
}

bool Close() {
	if (SDL_PollEvent(&Vars::event)) {
		if (Vars::event.type == SDL_WINDOWEVENT && Vars::event.window.event == SDL_WINDOWEVENT_CLOSE) {
			int windowID = Vars::event.window.windowID;
			for (int i = 0; i < 4; i++) {
				if (SDL_GetWindowID(Vars::window[i]) != windowID)
					continue;
				SDL_DestroyWindow(Vars::window[i]);
				SDL_DestroyRenderer(Vars::render[i]);
				Vars::windowOpen[i] = false;
				break;
			}
		}
		if (Vars::event.type == SDL_QUIT) {
			return true;
		}
	}
	return false;
}
```

## 代码讲解

首先这里必须说明一下，**`SDL_QUIT` 事件是全局的，表示所有窗口的关闭请求，而不是针对特定窗口的。要实现单独关闭某个窗口，需要监听窗口关闭事件 `SDL_WINDOWEVENT` 并检查子事件 `SDL_WINDOWEVENT_CLOSE`**。并且还要注意，仅仅监听到该事件并不会自动关闭窗口，必须调用`SDL_DestroyWindow`方可关闭窗口。
其次，还必须强调一下，渲染器和纹理是强绑定的，**一个渲染器创建的纹理（无论通过什么方法）无法被其他渲染器使用。**
本代码实现了打开四个窗口，每个窗口分别绘制原图片的一角。

## 函数讲解

### 目录

[SDL_GetWindowID](#sdl_getwindowid)

### SDL_GetWindowID

<font color=orange>函数原型</font>

```cpp
Uint32 SDL_GetWindowID(SDL_Window *window);
```

<font color=orange>描述</font>

数返回给定窗口的唯一标识符 (window ID)。该标识符可以用来在事件处理或与窗口相关的其他操作中引用该窗口。每个窗口在SDL程序中都会有一个唯一的 window ID。

<font color=orange>输入</font>

`window` ： 一个指向 SDL_Window 对象的指针，表示你想获取其 ID 的窗口。

<font color=orange>输出</font>

返回一个Uint32类型的值，表示窗口的唯一ID。**如果传入的window指针为 NULL，该函数返回0。**
