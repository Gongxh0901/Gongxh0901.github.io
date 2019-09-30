---
layout: post
title: Cocos集成FMOD音效
categories: Cocos
---

<p><small>git完整项目地址<a href="https://github.com/Gongxh0901/fmod_example" target="_blank">https://github.com/Gongxh0901/fmod_example</a>.</small></p>

# Cocos集成FMOD音效
---

- cocos: cocos2d-x-3.17.1

- <p>FMOD官网 <a href="https://www.fmod.com">https://www.fmod.com</a></p>
- <p>FMOD下载地址 <a href="https://www.fmod.com/download">https://www.fmod.com/download</a></p>

### 集成

	首先下载对应的api
	
### macOS

1. 把`api/lowlevel/lib/`下的`libfmod.dylib` 和 `api/lowlevel/inc`下的代码添加项目中
2. 项目配置把`libfmod.dylib`所在的路径添加到`build settings`下的`Library Search Paths`中，头文件的路径添加到`build settings`下的`Header Search Paths`中
3. 在`Build Phases`中点击`+`选择`New Copy File Phases`,选择`Frameworks`,然后把`libfmod.dylib`添加到新建的phases中
4. 在`build settings`下的`Other Linker Flags`添加`-lfmod`,在 `Runpath Search Paths`设置`@runpath`的路径，示例项目中是`@loader_path/../Frameworks`
<p>@runpath参考地址 <a href="http://www.cnblogs.com/csuftzzk/p/mac_run_path.html">http://www.cnblogs.com/csuftzzk/p/mac_run_path.html</a></p> 

### IOS

1. 把`api/lowlevel/lib/`下的`.a` 和 `api/lowlevel/inc`下的代码添加项目中
2. 项目配置把静态库`.a`所在的路径添加到`build settings`下的`Library Search Paths`中，头文件的路径添加到`build settings`下的`Header Search Paths`中

### Windows

1. 把`api/lowlevel/lib/`下的`.a` 和 `api/lowlevel/inc`下的代码添加项目中
2. 右键点击工程，最下边找到`属性`，`属性配置`下找到`C/C++`，然后点击`常规`,在附加包含目录中添加刚才添加的`inc`下的代码的目录。（release和debug）都要加
3. 在`C/C++`平级的列表中找到`链接器`，点击`常规`，在右侧`附加库目录`中添加刚才添加的`lib`目录（release和debug）都要加，在`常规`平级列表中找到`输入`，然后在`附加依赖项`中`debug`下添加`fmodL_vc.lib，fmodL64_vc.lib`，`release`下添加`fmod_vc.lib，fmod64_vc.lib`

### Android

1. 把`api/lowlevel/lib/` 和 `api/lowlevel/inc`添加到项目中
2. `fmod.jar`包添加到项目中`app/libs`下，没有目录则新建一个，在build.gradle中添加`implementation files('libs/fmod.jar')`引用 `jar`包。
3. 编辑`Android.mk`文件，在`LOCAL_PATH := $(call my-dir)`下边添加

```
FMOD_ANDROID_DIR := ../../../../fmod/android
FMOD_LIBRARY_PATH := $(FMOD_ANDROID_DIR)/lib/$(TARGET_ARCH_ABI)
	
include $(CLEAR_VARS)
LOCAL_MODULE := fmod
LOCAL_SRC_FILES := $(FMOD_LIBRARY_PATH)/libfmod.so
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/$(FMOD_ANDROID_DIR)/inc
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE := fmodL
LOCAL_SRC_FILES := $(FMOD_LIBRARY_PATH)/libfmodL.so
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/$(FMOD_ANDROID_DIR)/inc
include $(PREBUILT_SHARED_LIBRARY)
```
在`include $(BUILD_SHARED_LIBRARY)`上边添加
	
```
LOCAL_STATIC_LIBRARIES += fmod
LOCAL_STATIC_LIBRARIES += fmodL
```

4. 如果使用lua，还需要编辑lua_bindings下边的Android.mk
5. AppActivity中添加必要的函数调用，初始化、退出、load lib

```java
public class AppActivity extends Cocos2dxActivity{
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.setEnableVirtualButton(false);
        super.onCreate(savedInstanceState);
        org.fmod.FMOD.init(this);
    }
    protected void onDestroy() {
        super.onDestroy();
        org.fmod.FMOD.close();//用于关闭fmod功能
    }
    static
    {
        System.loadLibrary("fmod");
    }
}
```

### 在lua中使用

打开 `frameworks/cocos2d-x/tools/tolua/` 仿着写一个导出文件 `<.ini 和 .py>`导出c++ to lua

### 注意
cocos tolua 需要使用cocos版本对应的android-ndk，对应关系查看cocos官网


### 简单使用

~~~ 伪代码
//初始化FMOD系统
FMOD::System *system;
FMOD::System_Create(&system); 
system->init(maxchannels, FMOD_INIT_NORMAL, nullptr); // maxchannels表示最大channel数量（可同时播放的音效数量）

//创建音效的实例
FMOD::Sound * sound;
system->createSound(filename, FMOD_DEFAULT, 0, &sound); // 第二个参数根据使用环境不同选择不同的枚举

// 播放上边创建的音效
FMOD::Channel *channel = 0;
system->playSound(sound, 0, false, &channel);

//设置不同属性（音量，模式等）
channel->setVolume(volume);
channel->setMode(FMOD_DEFAULT);
~~~


<p><big>示例工程<a href="https://github.com/Gongxh0901/fmod_example" target="_blank">https://github.com/Gongxh0901/fmod_example</a>.</big></p>
在 `frameworks/fmod`中的FmodSound类，简单实现了背景音乐和播放音效的基础方法

```
void playSound(std::string filename, bool loop, float volume);
void setSoundVolume(float volume);
void stopSoundByName(std::string filename);
void stopAllSound();
void playMusic(std::string filename, bool isloop);
void stopMusic();
void preloadSound(std::string filename);
```