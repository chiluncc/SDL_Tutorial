# 08 The Viewport

## 前言

本章只讲述一个小的知识点，即如何选择Render绘制的区域。

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
	SDL_Rect fillRect, useRect;
	frame::texture = SDL_CreateTexture(frame::render, SDL_PIXELFORMAT_RGBA8888, SDL_TEXTUREACCESS_TARGET, 800, 600);
	SDL_SetRenderTarget(frame::render, frame::texture);
	SDL_RenderClear(frame::render);
	
	fillRect = { frame::width / 4, frame::height / 4, frame::width / 8, frame::height / 8 };
	SDL_SetRenderDrawColor(frame::render, 0XFF, 0X00, 0X00, 0XFF);

	useRect = { 0, 0, frame::width / 2, frame::height / 2 };
	SDL_RenderSetViewport(frame::render, &useRect);
	SDL_RenderFillRect(frame::render, &fillRect);
	useRect = { frame::width / 2, 0, frame::width / 2, frame::height / 2 };
	SDL_RenderSetViewport(frame::render, &useRect);
	SDL_RenderFillRect(frame::render, &fillRect);
	useRect = { 0, frame::height / 2, frame::width / 2, frame::height / 2 };
	SDL_RenderSetViewport(frame::render, &useRect);
	SDL_RenderFillRect(frame::render, &fillRect);
	useRect = { frame::width / 2, frame::height / 2, frame::width / 2, frame::height / 2 };
	SDL_RenderSetViewport(frame::render, &useRect);
	SDL_RenderFillRect(frame::render, &fillRect);

	SDL_RenderSetViewport(frame::render, nullptr);
	SDL_SetRenderDrawColor(frame::render, 0X00, 0X00, 0X00, 0XFF);
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

这里我们使用了`useRect`以不断地更改指定的绘制区域。在四个区域分别绘制了一个矩形。

## 函数讲解

### 目录

[SDL_RenderSetViewport](#sdl_rendersetviewport)

### SDL_RenderSetViewport

<font color=orange>函数原型</font>

```cpp
int SDL_RenderSetViewport(SDL_Renderer *renderer, const SDL_Rect *rect);
```

<font color=orange>描述</font>

用于设置当前渲染器的视口（viewport）。视口是一个矩形区域，定义了渲染操作的输出范围。在视口内的图像将被渲染，而在视口外的部分则不会显示。视口通常用于实现缩放或裁剪的效果。

> 你可以利用视口来实现图像的缩放效果，例如缩小绘制区域的大小，或者仅渲染图像的某个部分。
> **视口设置是渲染目标和渲染器的共同属性**，即视口定义是针对当前渲染目标、当前渲染器的。当切换渲染目标时，渲染器会为新目标应用一个默认视口（通常覆盖整个目标大小）。如果在某个渲染目标上设置了特定视口，这个视口不会影响其他渲染目标的视口状态。**渲染器 + 当前渲染目标 = 唯一的视口状态。**

<font color=orange>输入</font>

`renderer` ：指向 `SDL_Renderer` 的指针，表示渲染器对象。
`rect` ：指向一个 `SDL_Rect` 结构的指针，表示要设置的视口矩形。可以为 `NULL`，**如果为 `NULL`，视口会恢复为默认大小（即渲染器的输出目标大小，通常是窗口的大小）。**

<font color=orange>输出</font>

如果成功，返回0。
如果失败，返回-1，并且可以通过 `SDL_GetError()` 获取错误信息。
