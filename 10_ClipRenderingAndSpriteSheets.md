# 10 Clip Rendering and Sprite Sheets

## 前言

本章会介绍可乐图，啊不，这里被称为精灵图。写过html、游戏或者其他应用的朋友一定很熟悉这个理念。即将大量需要的素材绘制在同一张图片上，在使用时通过截取对应区域来获得对应的图片。在本章我们会利用前面的很多函数做一点与教程完全不同的活！是的，更有意思，更灵活。

## 代码展示



## 代码讲解

首先这里必须说明一下，**`SDL_QUIT` 事件是全局的，表示所有窗口的关闭请求，而不是针对特定窗口的。要实现单独关闭某个窗口，需要监听窗口关闭事件 `SDL_WINDOWEVENT` 并检查子事件 `SDL_WINDOWEVENT_CLOSE`**。并且还要注意，仅仅监听到该事件并不会自动关闭窗口，必须调用`SDL_DestroyWindow`方可关闭窗口。

## 函数讲解

### 目录

[SDL_GetWindowID](#sdl_getwindowid)

### SDL_GetWindowID