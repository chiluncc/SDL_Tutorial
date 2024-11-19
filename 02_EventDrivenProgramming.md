# 02 Event Driver Programming

## 前言

这一章所对应的原教程实在是过于简短了，完全没有看过瘾啊！这里我们就详细的说明一下事件驱动吧。
首先，什么是事件？鼠标移动、点击、键盘的点击、手柄的拨动等等都是事件。简而言之，一切输入系统的输入都以事件的形式保存在SDL。而保存这些事件的数据结构为队列，就是个遵循先来先处理的数组或者链表啦。

## ```SDL_Event``` 结构体

感觉本章节没有上代码的必要性，让我们跳过代码展示，直接来介绍下“事件”这一结构体吧。首先给出整个结构体的声明，额，说实话，我没想到这玩意会这么大。

```cpp
typedef union SDL_Event
{
    Uint32 type;                            /**< Event type, shared with all events */
    SDL_CommonEvent common;                 /**< Common event data */
    SDL_DisplayEvent display;               /**< Display event data */
    SDL_WindowEvent window;                 /**< Window event data */
    SDL_KeyboardEvent key;                  /**< Keyboard event data */
    SDL_TextEditingEvent edit;              /**< Text editing event data */
    SDL_TextEditingExtEvent editExt;        /**< Extended text editing event data */
    SDL_TextInputEvent text;                /**< Text input event data */
    SDL_MouseMotionEvent motion;            /**< Mouse motion event data */
    SDL_MouseButtonEvent button;            /**< Mouse button event data */
    SDL_MouseWheelEvent wheel;              /**< Mouse wheel event data */
    SDL_JoyAxisEvent jaxis;                 /**< Joystick axis event data */
    SDL_JoyBallEvent jball;                 /**< Joystick ball event data */
    SDL_JoyHatEvent jhat;                   /**< Joystick hat event data */
    SDL_JoyButtonEvent jbutton;             /**< Joystick button event data */
    SDL_JoyDeviceEvent jdevice;             /**< Joystick device change event data */
    SDL_JoyBatteryEvent jbattery;           /**< Joystick battery event data */
    SDL_ControllerAxisEvent caxis;          /**< Game Controller axis event data */
    SDL_ControllerButtonEvent cbutton;      /**< Game Controller button event data */
    SDL_ControllerDeviceEvent cdevice;      /**< Game Controller device event data */
    SDL_ControllerTouchpadEvent ctouchpad;  /**< Game Controller touchpad event data */
    SDL_ControllerSensorEvent csensor;      /**< Game Controller sensor event data */
    SDL_AudioDeviceEvent adevice;           /**< Audio device event data */
    SDL_SensorEvent sensor;                 /**< Sensor event data */
    SDL_QuitEvent quit;                     /**< Quit request event data */
    SDL_UserEvent user;                     /**< Custom event data */
    SDL_SysWMEvent syswm;                   /**< System dependent window event data */
    SDL_TouchFingerEvent tfinger;           /**< Touch finger event data */
    SDL_MultiGestureEvent mgesture;         /**< Gesture event data */
    SDL_DollarGestureEvent dgesture;        /**< Gesture event data */
    SDL_DropEvent drop;                     /**< Drag and drop event data */

    Uint8 padding[sizeof(void *) <= 8 ? 56 : sizeof(void *) == 16 ? 64 : 3 * sizeof(void *)];
} SDL_Event;

```

你可能注意到了，这是一个 ```union``` ，即在任何情况下只能存放一种数据，那么，SDL是如何让 ```type``` 和其他数据共存的呢？让我们想一下union的物理结构，在win+vs环境下，union会直接申请所包含的数据中最长的那个所占大小的空间。每次写入数据时都会从内存低地址开始连续写入。而如何判断里面存放的数据是什么类型的呢？答案是不用判断，数据是固定的，用什么类型去解释解释什么类型。那么我们就可以这样做，让该空间低32位永远存放 ```type``` ，其他空间存放其他数据。如何做到这一点呢？很简单，让所有其他结构体的第一个元素是 ```Uint32``` 类型的占位空数据就好了，不，不是空元素，应该说这两个 ```type``` 是同一个地址，虽然看起来一个是外层 ```union``` 的，一个是内层 ```struct``` 的。实时也如我们想的那样，以下是部分结构体的声明：

```cpp
typedef struct SDL_CommonEvent
{
    Uint32 type;
    Uint32 timestamp;   /**< In milliseconds, populated using SDL_GetTicks() */
} SDL_CommonEvent;

typedef struct SDL_WindowEvent
{
    Uint32 type;        /**< ::SDL_WINDOWEVENT */
    Uint32 timestamp;   /**< In milliseconds, populated using SDL_GetTicks() */
    Uint32 windowID;    /**< The associated window */
    Uint8 event;        /**< ::SDL_WindowEventID */
    Uint8 padding1;
    Uint8 padding2;
    Uint8 padding3;
    Sint32 data1;       /**< event dependent data */
    Sint32 data2;       /**< event dependent data */
} SDL_WindowEvent;
```

对此，我只能说，堪称妙绝！精妙至极！物理和逻辑此刻完美的汇聚在一起！

## 常用事件

### 目录

[SDL_CommonEvent](#sdl_commonevent)
[SDL_WindowEvent](#sdl_windowevent)
[SDL_KeyboardEvent](#sdl_keyboardevent)
[SDL_MouseWheelEvent](#sdl_mousewheelevent)
[SDL_MouseButtonEvent](#sdl_mousebuttonevent)
[SDL_MouseMotionEvent](#sdl_mousemotionevent)
[SDL_QuitEvent](#sdl_quitevent)
[Else](#else-usually-use-but-here-not-detailed-introduction)


### SDL_CommonEvent

<font color=orange>结构体声明</font>

```cpp
typedef struct SDL_CommonEvent {
    Uint32 type;      // 事件类型（如 SDL_QUIT）
    Uint32 timestamp; // 事件触发的时间戳
} SDL_CommonEvent;
```

<font color=orange>描述</font>

通用事件，包含所有事件类型的公共部分。原理参考前面的 ```type``` 部分。
```type``` ：事件类型，这里就不介绍 ```type``` 的取值了，在00章节我们有详细讲解的。
```timestamp``` ：事件发生的时间戳，以毫秒为单位。在大约 49.7 天后会发生溢出，所以正常情况放心用就好了。


### SDL_WindowEvent

<font color=orange>结构体声明</font>

```cpp
typedef struct SDL_WindowEvent
{
    Uint32 type;        /**< 事件类型, SDL_WINDOWEVENT 表示窗口事件 */
    Uint32 timestamp;   /**< 时间戳，以毫秒为单位，使用 SDL_GetTicks() 填充 */
    Uint32 windowID;    /**< 事件关联的窗口的 ID */
    Uint8 event;        /**< 窗口事件的类型，使用 SDL_WindowEventID 枚举来表示 */
    Uint8 padding1;     /**< 填充字节，确保数据对齐 */
    Uint8 padding2;     /**< 填充字节，确保数据对齐 */
    Uint8 padding3;     /**< 填充字节，确保数据对齐 */
    Sint32 data1;       /**< 与事件相关的数据1，具体含义视事件类型而定 */
    Sint32 data2;       /**< 与事件相关的数据2，具体含义视事件类型而定 */
} SDL_WindowEvent;

```

<font color=orange>描述</font>

```event``` ：常用取值和含义，及对应 ```data1``` 和 ```data2``` 如下所示。

|值|含义|data1|data2|
|:---:|:---:|:---:|:---:|
|SDL_WINDOWEVENT_SHOWN|窗口显示|未使用|未使用|
|SDL_WINDOWEVENT_HIDDEN|窗口隐藏|未使用|未使用|
|SDL_WINDOWEVENT_EXPOSED|需要重新绘制窗口内容|未使用|未使用|
|SDL_WINDOWEVENT_MOVED|窗口移动|新的窗口X坐标|新的窗口Y坐标|
|SDL_WINDOWEVENT_RESIZED|窗口被调整大小|新的窗口宽度|新的窗口高度|
|SDL_WINDOWEVENT_SIZE_CHANGED|窗口大小已改变|新的窗口宽度|新的窗口高度|
|SDL_WINDOWEVENT_MINIMIZED|窗口最小化|未使用|未使用|
|SDL_WINDOWEVENT_MAXIMIZED|窗口最大化|未使用|未使用|
|SDL_WINDOWEVENT_RESTORED|窗口恢复|未使用|未使用|
|SDL_WINDOWEVENT_ENTER|鼠标进入窗口区域|未使用|未使用|
|SDL_WINDOWEVENT_LEAVE|鼠标离开窗口区域|未使用|未使用|
|SDL_WINDOWEVENT_FOCUS_GAINED|窗口获得输入焦点|未使用|未使用|
|SDL_WINDOWEVENT_FOCUS_LOST|窗口失去输入焦点|未使用|未使用|
|SDL_WINDOWEVENT_CLOSE|请求关闭窗口|未使用|未使用|
|SDL_WINDOWEVENT_TAKE_FOCUS|请求获得输入焦点|未使用|未使用|
|SDL_WINDOWEVENT_HIT_TEST|请求运行自定义点击测试|未使用|未使用|


### SDL_KeyboardEvent

<font color=orange>结构体声明</font>

```cpp
typedef struct SDL_KeyboardEvent
{
    Uint32 type;
    Uint32 timestamp;
    Uint32 windowID;    /**< 接收键盘输入焦点的窗口的 ID。**/
    Uint8 state;        /**< 按下或者弹起*/
    Uint8 repeat;       /**< 检测是否为长按产生*/
    Uint8 padding2;
    Uint8 padding3;
    SDL_Keysym keysym;  /**< 包含关于所按下或释放的按键的信息。*/
} SDL_KeyboardEvent;
```

<font color=orange>描述</font>

```state``` ： ```SDL_PRESSED``` ：表示按键按下。 ```SDL_RELEASED``` ：表示按键释放。
```repeat``` ：如果该按键事件是重复产生的（由于长按），则该值为非零。
```keysym``` ：用于获取按键的符号、修饰符和扫描码等信息，便于识别具体按键。 ```SDL_Keysym``` 结构体中有一个 ```mod``` 字段，它表示当前按下的修饰键的状态。 ```mod``` 字段是一个按位掩码，可以通过与 ```SDL_KMOD_*``` 常量进行按位与操作，检查是否按下了特定的修饰键。

### SDL_MouseWheelEvent

<font color=orange>结构体声明</font>

```cpp
typedef struct SDL_MouseWheelEvent
{
    Uint32 type;        /**< 事件类型，表示滚轮事件（SDL_MOUSEWHEEL） */
    Uint32 timestamp;   /**< 事件的时间戳（以毫秒为单位），由 SDL_GetTicks() 函数填充 */
    Uint32 windowID;    /**< 接收鼠标焦点的窗口的 ID */
    Uint32 which;       /**< 鼠标实例的 ID */
    Sint32 x;           /**< 水平滚动的量值，正值表示向右滚动，负值表示向左滚动 */
    Sint32 y;           /**< 垂直滚动的量值，正值表示远离用户，负值表示朝向用户 */
    Uint32 direction;   /**< 设置为 SDL_MOUSEWHEEL_* 定义之一，若为 FLIPPED，则 x 和 y 的值相反。可乘以 -1 改变回正常方向 */
    float preciseX;     /**< 具有浮点精度的水平滚动量值，正值向右，负值向左（在 2.0.18 中添加） */
    float preciseY;     /**< 具有浮点精度的垂直滚动量值，正值远离用户，负值朝向用户（在 2.0.18 中添加） */
    Sint32 mouseX;      /**< 鼠标相对于窗口的 X 坐标（在 2.26.0 中添加） */
    Sint32 mouseY;      /**< 鼠标相对于窗口的 Y 坐标（在 2.26.0 中添加） */
} SDL_MouseWheelEvent;

```

<font color=orange>描述</font>

```which``` ：当系统上有多个鼠标设备时，可以用于区分具体是哪一个鼠标发出了滚轮事件。 ```SDL_TOUCH_MOUSEID``` 表示触摸板。
```x``` ：水平滚动量值，表示鼠标滚轮在水平方向上的滚动程度。正值：表示向右滚动。负值：表示向左滚动。
```y``` ：垂直滚动量值，表示鼠标滚轮在垂直方向上的滚动程度。正值：表示滚轮向前滚动，远离用户。负值：表示滚轮向后滚动，朝向用户。
```direction``` ：用于指示滚轮方向模式的字段。该字段的值会被设置为 ```SDL_MOUSEWHEEL_NORMAL``` 或 ```SDL_MOUSEWHEEL_FLIPPED``` 。 ```SDL_MOUSEWHEEL_NORMAL``` ：表示 ```x``` 和 ```y``` 的值直接反映滚动方向。 ```SDL_MOUSEWHEEL_FLIPPED``` ：表示 ```x``` 和 ```y``` 的值与正常方向相反。
```preciseX``` ：精准型 ```x```。
```preciseY``` ：精准型 ```y```。
> **注意，字段的值代表了滚轮在触发该事件时滚动的量，而不是和上一次滚动事件的差值。**

### SDL_MouseButtonEvent

<font color=orange>结构体声明</font>

```cpp
typedef struct SDL_MouseButtonEvent
{
    Uint32 type;        /**< 事件类型，SDL_MOUSEBUTTONDOWN 或 SDL_MOUSEBUTTONUP */
    Uint32 timestamp;   /**< 事件发生的时间戳，单位为毫秒 */
    Uint32 windowID;    /**< 接收鼠标焦点的窗口 ID */
    Uint32 which;       /**< 鼠标实例 ID，或 SDL_TOUCH_MOUSEID */
    Uint8 button;       /**< 鼠标按钮索引 (如 SDL_BUTTON_LEFT, SDL_BUTTON_RIGHT) */
    Uint8 state;        /**< 按钮状态，SDL_PRESSED 或 SDL_RELEASED */
    Uint8 clicks;       /**< 单击次数，1 为单击，2 为双击，依此类推 */
    Uint8 padding1;     /**< 填充字节，用于内存对齐 */
    Sint32 x;           /**< 鼠标 X 坐标，相对于窗口 */
    Sint32 y;           /**< 鼠标 Y 坐标，相对于窗口 */
} SDL_MouseButtonEvent;
```

<font color=orange>描述</font>

```clicks``` ：字段指的是当前事件为连续事件中的第几次点击。其值一般是在 **鼠标按下时** 被记录，并且在相邻的按下和释放事件之间被用来检测单击或双击等行为。

### SDL_MouseMotionEvent

<font color=orange>结构体声明</font>

```cpp
typedef struct SDL_MouseMotionEvent
{
    Uint32 type;        /**< 事件类型，SDL_MOUSEMOTION */
    Uint32 timestamp;   /**< 事件发生的时间戳，单位为毫秒 */
    Uint32 windowID;    /**< 接收鼠标焦点的窗口 ID */
    Uint32 which;       /**< 鼠标实例 ID，或 SDL_TOUCH_MOUSEID */
    Uint32 state;       /**< 当前的鼠标按钮状态，表示哪些按钮被按下（例如 SDL_BUTTON_LMASK） */
    Sint32 x;           /**< 鼠标当前位置的 X 坐标，相对于窗口左上角 */
    Sint32 y;           /**< 鼠标当前位置的 Y 坐标，相对于窗口左上角 */
    Sint32 xrel;        /**< 鼠标相对上次位置的 X 坐标变化量（相对移动量） */
    Sint32 yrel;        /**< 鼠标相对上次位置的 Y 坐标变化量（相对移动量） */
} SDL_MouseMotionEvent;
```

<font color=orange>描述</font>

```state``` ：其常取值和含义如下表所示：

|值|含义|
|:---:|:---:|
|SDL_BUTTON_LMASK|表示鼠标左键被按下|
|SDL_BUTTON_MMASK|表示鼠标中键被按下|
|SDL_BUTTON_RMASK|表示鼠标右键被按下|
|SDL_BUTTON_X1MASK|表示鼠标 X1 键（一般为前进按钮）被按下|
|SDL_BUTTON_X2MASK|表示鼠标 X2 键（一般为后退按钮）被按下|

### SDL_QuitEvent

<font color=orange>结构体声明</font>

```cpp
typedef struct SDL_QuitEvent
{
    Uint32 type;        /**< 事件类型，SDL_QUIT */
    Uint32 timestamp;
} SDL_QuitEvent;
```

<font color=orange>描述</font>

无特殊用处，不做解释。

### Else, Usually use but here not detailed introduction

<font color=orange>Detailed Introduction</font>

文件拖放事件，用于处理文件拖放到窗口的情况。

<font color=orange>SDL_UserEvent</font>

用户自定义事件，用于 ```SDL_PushEvent``` 中。

<font color=orange>SDL_TextInputEvent</font>

文本输入事件，用于捕获文本输入（如字符）。

## 函数讲解

### 目录

[SDL_GetTicks](#sdl_getticks)
[SDL_WaitEvent](#sdl_waitevent)
[SDL_PushEvent](#sdl_pushevent)
[SDL_AddTimer](#sdl_addtimer)
[SDL_FlushEvent](#sdl_flushevent)

### SDL_GetTicks

<font color=orange>函数原型</font>

```cpp
Uint32 SDL_GetTicks(void);
```

<font color=orange>描述</font>

返回自SDL库初始化以来经过的时间（以毫秒为单位）。SDL使用系统的时间戳或计时器来实现该功能。

<font color=orange>输入</font>

无输入。

<font color=orange>输出</font>

返回一个 ```Uint32``` 类型的值，表示自SDL库初始化以来的毫秒数。

### SDL_WaitEvent

<font color=orange>函数原型</font>

```cpp
int SDL_WaitEvent(SDL_Event* event);
```

<font color=orange>描述</font>

从SDL事件队列中获取事件并将其填充到传入的 ```SDL_Event``` 结构体中。该函数会一直阻塞，直到事件队列中有事件可以处理。它通常在主事件循环中使用，用于等待并处理用户输入、窗口事件等。

<font color=orange>输入</font>

```event``` ：指向 ```SDL_Event``` 结构体的指针，用于接收从事件队列中提取的事件数据。

<font color=orange>输出</font>

成功时：返回 1。失败时：返回 0，并且可以使用 ```SDL_GetError()``` 查看具体的错误信息。

### SDL_PushEvent

<font color=orange>函数原型</font>

```cpp
int SDL_PushEvent(SDL_Event* event);

```

<font color=orange>描述</font>

允许用户将一个 ```SDL_Event``` 结构体推送到事件队列的末尾。事件队列中的事件会按照顺序依次处理，直到队列为空或程序退出。

<font color=orange>输入</font>

```event``` ：指向 ```SDL_Event``` 结构体的指针，表示要添加到事件队列的事件。该结构体应包含事件的类型以及相应的事件数据。

<font color=orange>输出</font>

成功时：返回 1。失败时：返回 0，失败时可以使用 ```SDL_GetError()``` 获取具体的错误信息。

### SDL_AddTimer

<font color=orange>函数原型</font>

```cpp
SDL_TimerID SDL_AddTimer(Uint32 interval, SDL_TimerCallback callback, void* param);
```

<font color=orange>描述</font>

用于注册一个定时器，使得每隔一定的时间（由 ```interval``` 参数指定）触发一次回调函数。这个回调函数在指定的时间间隔到达时会被调用，直到取消该定时器或程序退出。定时器会在后台线程中运行，因此它不会阻塞主线程。

<font color=orange>输入</font>

```interval``` ： 定时器触发的时间间隔，单位是毫秒（ms）。表示每隔多少毫秒调用一次回调函数。
```callback``` ： 定时器触发时要调用的回调函数。回调函数的原型如下：

```cpp
Uint32 SDL_TimerCallback(Uint32 interval, void* param);
```
```interval``` ： 上次触发事件时的时间间隔（与注册定时器时传入的 ```interval``` 相同）。
```param``` ：从 ```SDL_AddTimer``` 传递的自定义数据。可以用来传递任何信息。
返回值：回调函数需要返回一个新的时间间隔（单位：毫秒）。如果返回 0，定时器将停止；否则，它会重新在指定的时间间隔后触发。

<font color=orange>输出</font>

成功时：返回一个 ```SDL_TimerID```，可以用来管理定时器。
失败时：返回 ```NULL```，并且可以使用 ```SDL_GetError()``` 获取详细错误信息。

### SDL_FlushEvent

<font color=orange>函数原型</font>

```cpp
void SDL_FlushEvent(Uint32 eventType);
```

<font color=orange>描述</font>

清空事件队列中所有类型为 ```eventType``` 的事件。它不会触发任何事件，也不会返回错误。通过调用此函数，可以丢弃所有特定类型的事件，避免它们影响程序的后续事件处理。

<font color=orange>输入</font>

```eventType``` ：需要清除的事件类型。可以是SDL事件类型中的某一个（如 ```SDL_QUIT``` ， ```SDL_KEYDOWN``` 等），也可以是自定义事件类型。通过指定事件类型， ```SDL_FlushEvent``` 只会清除该类型的事件，而不会影响其他类型的事件。如果传入 ```SDL_ALLEVENTS``` （一个常量），它将清除所有事件。

<font color=orange>输出</font>

无输出。