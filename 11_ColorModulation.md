# 11 Color Modulation

## 前言

ColorModulation，即色彩调制，简单地说就是让一个颜色RGB对应位置地值以指定地比例下降，**无法上升**。主要用于画面减淡退出。

## 代码展示

```cpp
#include <SDL.h>
#include <iostream>

namespace Vars {
	SDL_Window* window;
	SDL_Texture* texture;
	SDL_Renderer* render;
	SDL_Event event;
	const Uint32 width = 800;
	const Uint32 height = 600;
}

void Init() {
	SDL_Init(SDL_INIT_VIDEO);
	Vars::window = SDL_CreateWindow("Texture Color Mod", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, Vars::width, Vars::height, SDL_WINDOW_SHOWN);
	Vars::render = SDL_CreateRenderer(Vars::window, -1, 0);
}

void DrawTexture() {
	SDL_Rect rect;
	Vars::texture = SDL_CreateTexture(Vars::render, SDL_PIXELFORMAT_RGBA8888, SDL_TEXTUREACCESS_TARGET, Vars::width, Vars::height);
	SDL_SetRenderTarget(Vars::render, Vars::texture);

	SDL_SetRenderDrawColor(Vars::render, 0XFF, 0XFF, 0XFF, 0XFF);
	rect = { 0, 0, Vars::width / 2, Vars::height / 2 };
	SDL_RenderFillRect(Vars::render, &rect);

	SDL_SetRenderDrawColor(Vars::render, 0, 0XFF, 0XFF, 0XFF);
	rect = { Vars::width / 2, 0, Vars::width / 2, Vars::height / 2 };
	SDL_RenderFillRect(Vars::render, &rect);

	SDL_SetRenderDrawColor(Vars::render, 0XFF, 0, 0XFF, 0XFF);
	rect = { 0, Vars::height / 2, Vars::width / 2, Vars::height / 2 };
	SDL_RenderFillRect(Vars::render, &rect);

	SDL_SetRenderDrawColor(Vars::render, 0XFF, 0XFF, 0, 0XFF);
	rect = { Vars::width / 2, Vars::height / 2, Vars::width / 2, Vars::height / 2 };
	SDL_RenderFillRect(Vars::render, &rect);

	SDL_SetTextureColorMod(Vars::texture, 64, 128, 255);
	SDL_SetRenderTarget(Vars::render, nullptr);
	SDL_SetRenderDrawColor(Vars::render, 0XFF, 0XFF, 0XFF, 0XFF);
}

void Quit() {
	SDL_DestroyWindow(Vars::window);
	SDL_DestroyTexture(Vars::texture);
	SDL_DestroyRenderer(Vars::render);
	SDL_Quit();
}

int main(int argc, char* argv[]) {
	Init();
	DrawTexture();
	while (!(SDL_PollEvent(&Vars::event) && Vars::event.type == SDL_QUIT)) {
		SDL_RenderClear(Vars::render);
		SDL_RenderCopy(Vars::render, Vars::texture, nullptr, nullptr);
		SDL_RenderPresent(Vars::render);
	}
	return 0;
}
```

## 代码讲解

没什么好讲解的，我们这里直接给出色彩调制的计算公式好了。假设初始色彩分量为$V$，该分量对应的色彩调制参数为$P$，最终色彩分量值为$V'$那么他们之间的关系可以用以下公式表示：

$$
V'=\frac{P}{255}\cdot{V}
$$

## 函数讲解

### 目录

[SDL_SetTextureColorMod](#sdl_settexturecolormod)
[SDL_GetTextureColorMod](#sdl_gettexturecolormod)

### SDL_SetTextureColorMod

<font color=orange>函数原型</font>

```cpp
int SDL_SetTextureColorMod(SDL_Texture *texture, Uint8 r, Uint8 g, Uint8 b);
```

<font color=orange>描述</font>

该函数用于设置纹理的颜色调制。通过指定 RGB 值，您可以改变纹理的渲染颜色，从而影响其显示效果。颜色调制是通过将指定的 RGB 值与纹理的像素颜色值进行混合来实现的。

<font color=orange>输入</font>

`texture` ：指向需要设置颜色调制的纹理的指针。
`r` ：红色分量（0-255）。指定纹理调制的红色强度。
`g` ：绿色分量（0-255）。指定纹理调制的绿色强度。
`b` ：蓝色分量（0-255）。指定纹理调制的蓝色强度。

<font color=orange>输出</font>

返回值为0表示成功。
返回负值表示失败，可通过 `SDL_GetError()` 获取错误信息。

### SDL_GetTextureColorMod

<font color=orange>函数原型</font>

```cpp
int SDL_GetTextureColorMod(SDL_Texture *texture, Uint8 *r, Uint8 *g, Uint8 *b
```

<font color=orange>描述</font>

该函数用于获取纹理当前的颜色调制值，即当前设置的红、绿、蓝分量值。通过调用此函数，您可以查询由 `SDL_SetTextureColorMod` 设置的调制颜色。

<font color=orange>输入</font>

`texture` ：指向需要查询颜色调制值的纹理的指针。
`r` ：指向一个 Uint8 变量的指针，用于接收红色分量值。
`g` ：指向一个 Uint8 变量的指针，用于接收绿色分量值。
`b` ：指向一个 Uint8 变量的指针，用于接收蓝色分量值。

<font color=orange>输出</font>

返回值为0表示成功。
返回负值表示失败，可通过 `SDL_GetError()` 获取错误信息。