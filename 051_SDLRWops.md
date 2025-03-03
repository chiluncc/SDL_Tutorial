# 051 SDL_RWops


## 前言

在上一章的学习中偶然遇到了`SDL_RWops`这一结构体，本人很感兴趣，故此特地开一章说说`SDL_RWops`。
`SDL_RWops` 是一个结构体，它定义了文件或内存流的操作接口。这个结构体的成员是一组函数指针，每个函数指针对应一个操作（例如读取、写入、获取大小、关闭等）。作用是可以让程序员用统一的接口使用内存、文件中的数据。

## 函数指针

`SDL_RWops`内部有大量的函数指针，在通过其他函数获取`SDL_RWops`时会自动将一些函数绑定到这些函数指针上。`SDL_RWops`的函数指针包括：

```cpp
//win+vs2022平台下SDLCALL为__cdecl
Sint64 (SDLCALL * size) (struct SDL_RWops * context);
Sint64 (SDLCALL * seek) (struct SDL_RWops * context, Sint64 offset, int whence);
size_t (SDLCALL * read) (struct SDL_RWops * context, void *ptr, size_t size, size_t maxnum);
size_t (SDLCALL * write) (struct SDL_RWops * context, const void *ptr, size_t size, size_t num);
int (SDLCALL * close) (struct SDL_RWops * context);
```

## 相关函数

### 目录

[SDL_RWFromFile](#sdl_rwfromfile)
[SDL_RWFromMem](#sdl_rwfrommem)
[SDL_RWsize](#sdl_rwsize)
[SDL_RWseek](#sdl_rwseek)
[SDL_RWtell](#sdl_rwtell)
[SDL_RWread](#sdl_rwread)
[SDL_RWwrite](#sdl_rwwrite)
[SDL_RWclose](#sdl_rwclose)
[SDL_RWgetData](#sdl_rwgetdata)

### SDL_RWFromFile

<font color=orange>函数原型</font>

```cpp
SDL_RWops* SDL_RWFromFile(const char *file, const char *mode);
```

<font color=orange>描述</font>

从文件创建一个 `SDL_RWops` 对象，提供对文件的读写访问。

<font color=orange>输入</font>

`file` ：文件路径。
`mode` ：文件打开模式（例如 `r`, `w`, `rb`, `wb` 等）。

<font color=orange>输出</font>

返回一个 `SDL_RWops` 指针，用于操作该文件。

### SDL_RWFromMem

<font color=orange>函数原型</font>

```cpp
SDL_RWops* SDL_RWFromMem(void *mem, int size);
```

<font color=orange>描述</font>

从内存创建一个 `SDL_RWops` 对象，提供对内存数据的读写访问。可以对`int`，`string`，`char`等简单类型、数组、结构体和图像或音频数据等进行操作。**`SDL_RWFromMem` 函数是将内存的引用传递给 `SDL_RWops` 对象。**

<font color=orange>输入</font>

`mem` ：指向内存缓冲区的指针。
`size` ：缓冲区的大小。

<font color=orange>输出</font>

返回一个 `SDL_RWops` 指针，用于操作内存数据。

### SDL_RWsize

<font color=orange>函数原型</font>

```cpp
Sint64 SDL_RWsize(SDL_RWops *context);
```

<font color=orange>描述</font>

描述：获取 `SDL_RWops` 指向的数据源（如文件或内存）的大小。

<font color=orange>输入</font>

`context` ：一个指向 `SDL_RWops` 对象的指针，表示需要获取大小的数据源。

<font color=orange>输出</font>

返回数据源的大小，单位是字节。

### SDL_RWseek

<font color=orange>函数原型</font>

```cpp
Sint64 SDL_RWseek(SDL_RWops *context, Sint64 offset, int whence);
```

<font color=orange>描述</font>

设置当前读取/写入位置的偏移。

<font color=orange>输入</font>

`context` ：一个指向 SDL_RWops 对象的指针。
`offset` ：偏移量，表示相对于 whence 的位置。
`whence` ：参考位置，可以是如下值：

|值|含义|
|:---:|:---:|
|`RW_SEEK_SET`|文件的开头|
|`RW_SEEK_CUR`|当前读取/写入位置|
|`RW_SEEK_END`|文件的末尾|

<font color=orange>输出</font>

返回新的位置，如果失败，则返回-1。

### SDL_RWtell

<font color=orange>函数原型</font>

```cpp
Sint64 SDL_RWtell(SDL_RWops *context);
```

<font color=orange>描述</font>

获取当前读取/写入位置。

<font color=orange>输入</font>

`context` ：一个指向 SDL_RWops 对象的指针。

<font color=orange>输出</font>

返回当前的文件/流位置。

### SDL_RWread

<font color=orange>函数原型</font>

```cpp
size_t SDL_RWread(SDL_RWops *context, void *ptr, size_t size, size_t maxnum);
```

<font color=orange>描述</font>

从 `SDL_RWops` 指定的数据源读取数据。

<font color=orange>输入</font>

`context` ：一个指向 `SDL_RWops` 对象的指针。
`ptr` ：指向存储读取数据的缓冲区。
`size` ：每个数据项的大小。
`maxnum` ：最大数据项数。

<font color=orange>输出</font>

成功读取的数据项数。

### SDL_RWwrite

<font color=orange>函数原型</font>

```cpp
size_t SDL_RWwrite(SDL_RWops *context, const void *ptr, size_t size, size_t num);
```

<font color=orange>描述</font>

将数据写入 `SDL_RWops` 指定的数据源。**常情况下，`SDL_RWwrite` 函数是将数据复制到目标，而不是移动。**

<font color=orange>输入</font>

`context` ：一个指向 `SDL_RWops` 对象的指针。
`ptr` ：指向要写入数据的缓冲区。
`size` ：每个数据项的大小。
`num` ：要写入的数据项数。

<font color=orange>输出</font>

成功写入的数据项数。

### SDL_RWclose

<font color=orange>函数原型</font>

```cpp
int SDL_RWclose(SDL_RWops *context);
```

<font color=orange>描述</font>

关闭 `SDL_RWops` 对象，释放与其关联的资源。

<font color=orange>输入</font>

`context` ：一个指向 `SDL_RWops` 对象的指针。

<font color=orange>输出</font>

如果成功关闭，返回0；如果失败，返回-1。

### SDL_RWgetData

<font color=orange>函数原型</font>

```cpp
void* SDL_RWgetData(SDL_RWops *context);
```

<font color=orange>描述</font>

获取 `SDL_RWops` 中的数据（适用于内存操作）。**`SDL_RWgetData` 函数的作用并不是复制或移动数据，而是返回与 `SDL_RWops` 对象相关联的内存指针。**

<font color=orange>输入</font>

`context` ：一个指向 `SDL_RWops` 对象的指针。

<font color=orange>输出</font>

返回一个指向数据源的指针（对于内存，返回实际的内存缓冲区）。

> `SDL_RWgetData` 和 `SDL_RWFromMem` 的结合可以实现 **装包和拆包** 的操作，尤其是在需要将数据存储在内存中或从内存中提取数据时。SDL_RWFromMem 提供了一个基于内存的 SDL_RWops 对象，允许我们在内存缓冲区内读写数据，而 SDL_RWgetData 可以用来获取该缓冲区的实际数据内容。
> ```cpp
> int bufferSize = sizeof(GameObject) * 2;
> char* buffer = (char*)malloc(bufferSize);
> GameObject obj1 = {1, "Player"};
> GameObject obj2 = {2, "Enemy"};
> //装包
> SDL_RWwrite(rw, &obj1, sizeof(GameObject), 1);
> SDL_RWwrite(rw, &obj2, sizeof(GameObject), 1);
> //拆包
> GameObject* unpackedData = (GameObject*)SDL_RWgetData(rw);
> ```
