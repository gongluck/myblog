---
title: "字符动画播放器,不止BadApple!"
date: 2020-04-07T12:42:47+08:00
draft: true

categories:
  - "cpp"
tags:
  - "cpp"
  - "BadApple"

---

有屏幕的地方就有BadApple!在B站看过好多版本,于是我也用控制台做了一个。网上很多人都是直接显示bmp图片的,但我这个版本是用libvlc实时解码和播放的,音频用libvlc播放,画面就是转换成字符后输出到控制台。除了BadApple这种黑白动画,也能播放彩色视频。具体可以看一下效果视频。

<!--more-->

![](../../img/BadApple-console.jpg)

### 设计思路
libvlc解码转码出RGBA和播放音频，再将RGBA量化为黑('*')白(' ')两个颜色并输出到屏幕。

### 效果视频
[https://www.bilibili.com/video/BV1N64y1u7BD](https://www.bilibili.com/video/BV1N64y1u7BD)

### 代码仓库
[https://github.com/gongluck/Character-player/tree/console](https://github.com/gongluck/Character-player/tree/console)

```C++
#include <iostream>
#include <thread>
#include <atomic>

#ifdef _WIN32
#include <Windows.h>
#define ssize_t SSIZE_T
#endif
#include <vlc/vlc.h>

#define CHECKEQUALRET(ret, compare)\
if(ret == compare)\
{\
    std::cerr << "error ocurred in " << __FILE__\
              << "`s line " << __LINE__\
              << ", error " << ret;\
    goto END;\
}

#define CHECKNEQUALRET(ret, compare)\
if(ret != compare)\
{\
    std::cerr << "error ocurred in " << __FILE__\
              << "`s line " << __LINE__\
              << ", error " << ret;\
    goto END;\
}

static int WIDTH = 200;
static int HEIGHT = 50;
static char* out_buffer = nullptr;
static char* tmp_buffer = nullptr;
static char* print_buffer = nullptr;
std::atomic<bool> atomiclock;
std::atomic<bool> gotframe = false;
static void* lock(void* data, void** p_pixels)
{
    while (atomiclock)
    {
        std::this_thread::sleep_for(std::chrono::microseconds(10));
    }
    atomiclock = true;
    *p_pixels = out_buffer;
    return 0;
}
static void unlock(void* data, void* id, void* const* p_pixels)
{
    atomiclock = false;
    gotframe = true;
}
static void display(/*void* data, void* id*/)
{
    atomiclock = true;
    if (!gotframe)
    {
        atomiclock = false;
        std::this_thread::sleep_for(std::chrono::microseconds(10));
        return;
    }
    memcpy(tmp_buffer, out_buffer, HEIGHT * WIDTH * 4);
    gotframe = false;
    atomiclock = false;

    HANDLE hConsoleOutput = GetStdHandle(STD_OUTPUT_HANDLE);
    COORD pos = { 0 };
    SetConsoleCursorPosition(hConsoleOutput, pos);

    RGBQUAD* rgba = reinterpret_cast<RGBQUAD*>(tmp_buffer);
    for (int i = 0; i < HEIGHT; ++i)
    {
        for (int j = 0; j < WIDTH; ++j)
        {
            auto point = rgba[WIDTH * i + j];
            auto light = (point.rgbRed + point.rgbGreen + point.rgbBlue) / 3;
            print_buffer[i * (WIDTH + 2) + j] = light > 127 ? '*' : ' ';
        }
        print_buffer[i * (WIDTH + 2) + WIDTH] = '\r';
        print_buffer[i * (WIDTH + 2) + WIDTH + 1] = '\n';
    }
    puts(print_buffer);
}

int main(int argc, char** argv)
{
    libvlc_instance_t* inst_ = nullptr;
    libvlc_media_t* media_ = nullptr;
    libvlc_media_player_t* player_ = nullptr;
    libvlc_media_list_t* list_ = nullptr;
    libvlc_media_list_player_t* plist_ = nullptr;
    int ret = 0;

    std::cout << "Usage: Character-player [media file] [out width] [out height]" << std::endl;

    if (argc >= 4)
    {
        WIDTH = atoi(argv[2]);
        HEIGHT = atoi(argv[3]);
    }

    HANDLE hConsoleOutput = GetStdHandle(STD_OUTPUT_HANDLE);
    COORD size = { WIDTH + 4, HEIGHT + 5 };
    SetConsoleScreenBufferSize(hConsoleOutput, size);
    SMALL_RECT rc = { 0,0, WIDTH + 1, HEIGHT + 1 };
    SetConsoleWindowInfo(hConsoleOutput, true, &rc);

    libvlc_log_close(nullptr);

    bool exit = false;
    std::thread th([&]()
    {
        while (!exit)
        {
            while (atomiclock)
            {
                std::this_thread::sleep_for(std::chrono::microseconds(10));
            }
            display();
        }
    });

    inst_ = libvlc_new(0, nullptr);
    CHECKEQUALRET(inst_, nullptr);
    media_ = libvlc_media_new_path(inst_, argc <= 1 ? "badapple.mp4" : argv[1]);
    CHECKEQUALRET(media_, nullptr);

    out_buffer = static_cast<char*>(malloc(HEIGHT * WIDTH * 4));
    CHECKEQUALRET(out_buffer, nullptr);
    tmp_buffer = static_cast<char*>(malloc(HEIGHT * WIDTH * 4));
    CHECKEQUALRET(tmp_buffer, nullptr);
    print_buffer = static_cast<char*>(malloc(HEIGHT * (WIDTH + 2) + 1));
    CHECKEQUALRET(print_buffer, nullptr);
    print_buffer[HEIGHT * (WIDTH + 2)] = '\0';

    //player_ = libvlc_media_player_new_from_media(media_);
    player_ = libvlc_media_player_new(inst_);
    CHECKEQUALRET(player_, nullptr);

    libvlc_video_set_callbacks(player_, lock, unlock, nullptr, 0);
    libvlc_video_set_format(player_, "RGBA", WIDTH, HEIGHT, WIDTH * 4);
    //libvlc_media_player_set_hwnd(player_, GetDesktopWindow());
    
    // play loop
    list_ = libvlc_media_list_new(inst_);
    plist_ = libvlc_media_list_player_new(inst_);
    libvlc_media_list_add_media(list_, media_);
    libvlc_media_list_player_set_media_list(plist_, list_);
    libvlc_media_list_player_set_media_player(plist_, player_);
    libvlc_media_list_player_set_playback_mode(plist_, libvlc_playback_mode_loop);
    libvlc_media_list_player_play(plist_);

    std::cin.get();

END:
    exit = true;
    if (th.joinable())
    {
        th.join();
    }

    if (player_ != nullptr)
    {
        libvlc_media_player_stop(player_);
        libvlc_media_player_release(player_);
        player_ = nullptr;
    }
    if (media_ != nullptr)
    {
        libvlc_media_release(media_);
        media_ = nullptr;
    }

    if (plist_ != nullptr)
    {
        libvlc_media_list_player_release(plist_);
        plist_ = nullptr;
    }
    if (list_ != nullptr)
    {
        libvlc_media_list_release(list_);
        list_ = nullptr;
    }
    if (inst_ != nullptr)
    {
        libvlc_release(inst_);
        inst_ = nullptr;
    }

    if (print_buffer != nullptr)
    {
        free(print_buffer);
        print_buffer = nullptr;
    }
    if (out_buffer != nullptr)
    {
        free(out_buffer);
        out_buffer = nullptr;
    }
    if (tmp_buffer != nullptr)
    {
        free(tmp_buffer);
        tmp_buffer = nullptr;
    }
}
```