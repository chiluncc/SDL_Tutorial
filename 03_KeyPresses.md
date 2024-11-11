# 03 Key Presses

## 前言

淦，才发现markdown的内嵌代码块只需要用“ \` \` ”符号包含就可以了，不需要打6个“ \` ”。突然被自己气乐了，也是，一个简洁的文档格式语言怎么可能允许那么愚蠢的写法啊！是我大意了。本章依旧不会完全按照原教程进行，我们会给出一些有意思的代码，而不仅仅是几个切换 `surface` ~~（哈哈，这里我就没用连续6个反引号）。~~ 在上一章中我们讲解的各种原型已经够多了，我们在这里直接亮出代码吧。

## 代码展示

```cpp
#include <SDL.h>
#include <iostream>

struct State {
public:
	SDL_Rect rect;

	State(SDL_Rect rect) : rect(rect) {
		next = nullptr;
		pre = nullptr;
	};
	void SetPre(State* preState) {
		preState->next = this;
		this->pre = preState;
	}
	void Next(State*& now) {
		now = now->next;
	}
	void Pre(State*& now) {
		now = now->pre;
	}

private:
	State* pre, * next;
};

class SDL_Entity {
public:
	SDL_Entity(int width = 800, int height = 600) : width(width), height(height) {
		SDL_Init(SDL_INIT_VIDEO);
		window = SDL_CreateWindow("states change", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, this->width, this->height, SDL_WINDOW_SHOWN);
		SPictrue = nullptr;
		SWindow = nullptr;
		state = nullptr;
		event = SDL_Event();
		quit = false;
		Prepare();
	}
	~SDL_Entity() {
		SDL_FreeSurface(SPictrue);
		SDL_DestroyWindow(window);
		SDL_Quit();
	}
	bool Quit() {
		return quit;
	}
	void PollEvent() {
		if (SDL_PollEvent(&event)) {
			switch (event.type)
			{
			case SDL_QUIT:
				this->quit = true;
				break;
			case SDL_MOUSEBUTTONUP:
				if (!state) break;
				if (event.button.button == SDL_BUTTON_LEFT) state->Next(state);
				else if (event.button.button == SDL_BUTTON_RIGHT) state->Pre(state);
				SDL_BlitSurface(SPictrue, &state->rect, SWindow, nullptr);
				std::cout << state->rect.x << " " << state->rect.y << " " << state->rect.w << " " << state->rect.h << std::endl;
				SDL_UpdateWindowSurface(window);
				break;
			default:
				break;
			}
		}
	}
private:
	const int width, height;
	SDL_Surface* SPictrue, * SWindow;
	SDL_Window* window;
	SDL_Event event;
	bool quit;
	State* state;
	void Prepare(const char* file = "./hello_world.bmp") {
		SPictrue = SDL_LoadBMP(file);
		SWindow = SDL_GetWindowSurface(window);
		SDL_BlitSurface(SPictrue, nullptr, SWindow, nullptr);
		SDL_UpdateWindowSurface(window);

		SDL_Rect rect = SPictrue->clip_rect;
		state = new State(rect);
		State* now = new State({ 0, 0, rect.w / 2, rect.h / 2 });
		now->SetPre(state);
		State* pre = now;
		now = new State({ rect.w / 2, 0, rect.w / 2, rect.h / 2 });
		now->SetPre(pre);
		pre = now;
		now = new State({ 0, rect.h / 2, rect.w / 2, rect.h / 2 });
		now->SetPre(pre);
		pre = now;
		now = new State({ rect.w / 2, rect.h / 2, rect.w / 2, rect.h / 2 });
		now->SetPre(pre);
		state->SetPre(now);
	}
};

int main(int argcs, char* args[]) {
	SDL_Entity sdl;
	while (!sdl.Quit()) {
		sdl.PollEvent();
	}
	return 0;
}
```

## 代码讲解

其中 `State` 是一个状态，用于记录克隆的区域，为了方便的进行切换和管理用了一个简单的 **状态模式** 。其他部分的代码和上一章的差不多。
我相信很多朋友已经看出来代码的缺陷了，包括可能的内存泄露以及PullEvent中繁杂的处理函数。因为这里只是想要写着玩，所以很多代码没有经过谨慎的思考就写下来了，其实泄露可以通过使用 `weak_ptr` 进行解决，或者手动写循环删除语句。另外状态创建语句也完全可以用一个记录倍率的数组（即当前位置乘以0、1还是$\frac{1}{2}$）和循环语句实现，无需一个个写。PullEvent完全可以仅仅标记状态变化，而真正的处理放在其他模块中完成......，总之这段代码有着很大的改进空间。在写的时候突然发现自己写Java和Python时间久了都忘记C++函数传的是值而非引用了（悲）。这段代码的运行结果也可验证 `SDL_BlitSurface` 只是简单的覆盖，不会清空整个区域或者其他区域，若想要清空区域可以使用 `SDL_FillRect` 进行表面清空操作。说到这就想起来一些古早的游戏引擎为了绘制效率会抛弃清空画布这一步，这就导致一旦游戏中在没有设置天空盒的情况下卡出地图就会看到画面如同着了魔般在物体边缘显示其前几帧的“残影”。对了，这里对 `clip_rect` 的处理也很烂不是嘛？其实完全可以通过直接替换 `clip_rect` ，而非指定 `SDL_Rect` 进行区域拷贝的。

## 函数讲解

### 目录

[Surface](#surface)

？，问号，看什么看，这章没有用到新的函数啊？算了，就说一下 `Surface` 结构体吧。

### Surface

<font color=orange>结构体声明</font>

```cpp
// SDL_Surface 结构体：表示一个位图图像，包含图像的像素数据及相关信息
typedef struct SDL_Surface
{
    Uint32 flags;               // 标志位，表示表面的状态或属性，通常只读
    SDL_PixelFormat *format;    // 像素格式指针，指定表面的颜色格式，只读
    int w, h;                   // 表面的宽度和高度（单位为像素），只读
    int pitch;                  // 每一行的字节数（通常为宽度乘以每像素字节数），只读
    void *pixels;               // 指向表面像素数据的指针，可以读写
    void *userdata;             // 应用程序数据指针，用于存储和表面关联的自定义数据，读写
    int locked;                 // 锁定标志，表示表面是否被锁定（不可编辑），只读
    void *list_blitmap;         // 维护一个 BlitMap 列表引用，用于管理与该表面相关的 blit 操作（私有字段）
    SDL_Rect clip_rect;         // 剪辑矩形，用于 blit 操作的裁剪区域，只读
    SDL_BlitMap *map;           // 指向快速 blit 映射信息的指针，用于高效绘制（私有字段）
    int refcount;               // 引用计数，用于管理表面的引用数，防止过早释放（读多写少）
} SDL_Surface;

```

<font color=orange>描述</font>

`w, h` ：是 `SDL_Surface` 的宽度和高度，单位是像素。`w` 和 `h` 是只读的，一旦创建表面就不能直接更改。
`clip_rect`：设置 blit（图像拷贝）操作的有效区域。只有 `clip_rect` 区域内的内容才会被绘制到目标表面上，其他部分将被忽略。如果在使用 `SDL_BlitSurface` 进行绘制时，源矩形的范围超出 `clip_rect` 的范围，超出的部分将不会被绘制，只会绘制 `clip_rect` 范围内的内容。`clip_rect` 可以在程序运行时动态更改，不会改变 `w` 和 `h` 的值。更改 `clip_rect` 只是临时限制绘制区域，不会改变表面的实际尺寸或像素数据。

> **当 `clip_rect` 的 `w` 和 `h` 大于图片的 `w` 和 `h` 时，SDL会自动将 `clip_rect` 调整为图片的实际范围。也就是说，即使你设置的 `clip_rect` 比图片大，SDL也只会将 `clip_rect` 限制在图片的实际尺寸内。**