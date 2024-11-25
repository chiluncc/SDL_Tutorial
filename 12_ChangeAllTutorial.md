# 12 Change All Tutorial

嘿，朋友们，我今个终于看到SDL3的发行版了（肯定不是今天发行的，只能说我不经常查看相关信息导致的。）但是管他呢，反正现在我们要更改所有的计划了，因为：

<font size=64 color=red>我们终于可以用SDL3了</font>
<font size=64 color=red>去他的SDL2吧！！！！！</font>

很好，我们接下来全面转进SDL3的学习 ~~（因为本人喜新厌旧）~~ 。因为最近事情多，对SDL的学习可能会变得断断续续，但是我们一定会一直更到55章甚至更后面。对了，根据这半个月的学习、写作经验，我们在接下来会更改一些习惯：

- 章节序号改为四位数，3|XX|Y，3代表SDL3，XX为章节序号，Y为对一个章节的切分。
- 这次我们会将函数讲解和实验代码部分分开，尽量5-10章的内容统一进行实验。我相信这个改变会让我们的进度更快，并且让我们的实验更加精彩。
- 我们的函数讲解部分的排版也会发生小的改变。注意事项不会再像现在那样杂乱无章
- 各个函数作用对比、作用域、作用对象等我们也会进行单独段落讲解，而不是像现在这样混杂在函数讲解中。

祝各位SDL3的学习能够更加愉快。

> 在这里扔点本来想写的东西好了，防止以后忘了。
> SDL_SetSurfaceAlphaMod;
> SDL_GetSurfaceAlphaMod;
> SDL_SetSurfaceBlendMode;
> SDL_GetSurfaceBlendMode;
> SDL_SetTextureAlphaMod;
> SDL_GetTextureAlphaMod;
> SDL_SetTextureBlendMode;
> SDL_GetTextureBlendMode;
> SDL_SetRenderDrawBlendMode;
> SDL_GetRenderDrawBlendMode;

再写点TTF相关的吧。

### TTF_RenderUNICODE_Blended

<font color=orange>函数原型</font>

```cpp
SDL_Surface* TTF_RenderUNICODE_Blended(TTF_Font* font, const Uint16* text, SDL_Color fg);
```

<font color=orange>描述</font>

将一段 Unicode 编码的文本渲染成一个高质量（抗锯齿）的 `SDL_Surface`，可用于进一步生成纹理或直接绘制。

<font color=orange>输入</font>

`font` ：描述：加载的字体对象，代表字体的样式和大小。通过 TTF_OpenFont 打开。
`text` ：描述：指向包含 Unicode 编码的宽字符（wchar_t）的文本缓冲区，以 NULL 或 0 结尾。
`fg` ：描述：前景色，用于指定文本颜色（如白色 {255, 255, 255, 255}）。

<font color=orange>输出</font>

生成的文本图像表面。失败时返回 NULL，可以用 `TTF_GetError` 获取错误信息。

> **其他 `TTF_RenderUNICODE` 系列函数**
> 
> |函数名称|功能说明|
> |:---:|:---:|
> |`TTF_RenderUNICODE_Solid`|渲染低质量（无抗锯齿）的 Unicode 文本到 `SDL_Surface`。速度快但质量差。|
> |`TTF_RenderUNICODE_Shaded`|渲染带背景色的 Unicode 文本到 `SDL_Surface`。需要额外指定背景颜色。|
> |`TTF_RenderUNICODE_Blended`|渲染高质量（抗锯齿）的 Unicode 文本到 `SDL_Surface`。渲染效果最佳。|

### TTF_SizeUNICODE

<font color=orange>函数原型</font>

```cpp
int TTF_SizeUNICODE(TTF_Font* font, const Uint16* text, int* w, int* h);
```

<font color=orange>描述</font>

计算指定 Unicode 文本在屏幕上的宽度和高度，不实际渲染。

<font color=orange>输入</font>

`font` ：描述：字体对象。
`text` ：描述：宽字符（Unicode 编码）的文本缓冲区。
`w` ：描述：存储文本宽度的指针。
`h` ：描述：存储文本高度的指针。

<font color=orange>输出</font>

描述：成功返回 0，失败返回 -1。

### TTF_Init

<font color=orange>函数原型</font>

```cpp
int TTF_Init(void);
```

<font color=orange>描述</font>

初始化 SDL_ttf 库，在任何使用 TTF 函数之前必须调用。如果未调用，后续操作将失败。

<font color=orange>输入</font>

无输入。

<font color=orange>输出</font>

初始化成功返回 0，失败返回 -1。使用 `TTF_GetError` 获取错误信息。

### TTF_OpenFont

<font color=orange>函数原型</font>

```cpp
TTF_Font* TTF_OpenFont(const char* file, int ptsize);
```

<font color=orange>描述</font>

从指定路径加载一个字体文件并设置字号。

<font color=orange>输入</font>

`file` ：字体文件路径（支持 .ttf 格式）。
`ptsize` ：字体大小（以点为单位，通常与像素值相关）。

<font color=orange>输出</font>

返回加载的字体对象指针，失败时返回 `NULL`。

### TTF_CloseFont

<font color=orange>函数原型</font>

```cpp
void TTF_CloseFont(TTF_Font* font);
```

<font color=orange>描述</font>

释放通过 `TTF_OpenFont` 或 `TTF_OpenFontIndex` 加载的字体对象，防止内存泄漏。

> `TTF_OpenFontIndex` ：与 `TTF_OpenFont` 类似，但支持加载字体文件中的特定字体（对于包含多个字体的文件，例如 .ttc 格式）。

<font color=orange>输入</font>

`font` ：需要关闭的字体对象。

<font color=orange>输出</font>

无输出。

### TTF_Quit

<font color=orange>函数原型</font>

```cpp
void TTF_Quit(void);
```

<font color=orange>描述</font>

关闭 SDL_ttf 库并释放与其相关的资源。在程序退出前必须调用。

<font color=orange>输入</font>

无输入。

<font color=orange>输出</font>

无输出。

### TTF_GetError

<font color=orange>函数原型</font>

```cpp
const char* TTF_GetError(void);
```

<font color=orange>描述</font>

获取最近一次 SDL_ttf 函数失败的错误信息。

<font color=orange>输入</font>

无输入。

<font color=orange>输出</font>

包含错误描述的字符串。