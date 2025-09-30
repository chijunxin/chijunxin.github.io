# Skynet如何创建和启动的c服务



<!--more-->

## 核心概念
在 Skynet 中，无论是 Lua 服务还是 C 服务，本质上都是一个 skynet 模块（module）。这个模块是一个动态链接库（在 Linux 上是 .so 文件，在 macOS 上是 .dylib 文件，在 Windows 上是 .dll 文件）。


## 编写 C 服务模块
C 服务模块必须实现并导出几个特定的函数供 Skynet 框架调用。
1. xxx_create： 创建服务实例。
2. xxx_init： 初始化服务实例。
3. xxx_release： 释放服务实例（可选）。
4. xxx_signal： 处理信号（可选）。

``` service_simple.c {open=true}
// service_simple.c
#include "skynet.h" // 必须包含 Skynet 头文件

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 定义服务的数据结构
struct simple {
  int number; // 示例：一个简单的成员变量
};

// 1. 创建函数 (simple_create)
// 当 Skynet 启动该服务时，首先调用此函数来创建服务对象。
struct simple *
simple_create(void) {
  struct simple * s = skynet_malloc(sizeof(*s)); // 使用 Skynet 的内存管理函数
  s->number = 0;
  return s;
}

// 消息处理回调函数
static int
simple_callback(struct skynet_context *ctx, void *ud, int type, int session, uint32_t source, const void *msg, size_t sz) {
    struct simple *s = ud;
    s->number++;
    skynet_error(ctx, "[simple] Received message number:%d", s->number);
    
    switch (type) {
    case PTYPE_TEXT: {
        // 处理文本消息
        skynet_error(ctx, "[simple] Received text: %.*s", (int)sz, (const char *)msg);
        
        // 回显消息
        skynet_send(ctx, 0, source, PTYPE_RESPONSE, session, (void *)msg, sz);
        break;
    }
    case PTYPE_RESERVED_LUA: {
        // 处理 Lua 消息
        skynet_error(ctx, "[simple] Received lua: %s", (const char *)msg);
        break;
    }
    case PTYPE_RESPONSE: {
        skynet_error(ctx, "[simple] Received response");
        break;
    }
    default: {
        skynet_error(ctx, "[simple] Unknown message type: %d", type);
        break;
    }
    }
    
    return 0;
}

// 2. 初始化函数 (simple_init)
// 创建成功后，Skynet 会调用此函数进行初始化。
// ctx: 该服务的上下文指针，非常重要，用于发送消息、注册回调等。
// context: 启动服务时传入的参数（字符串形式）。
int
simple_init(struct simple * s, struct skynet_context * ctx, const char * context) {
  printf("Simple Service Init: %s\n", context);
  
  // 示例：解析启动参数
  if (context != NULL) {
    s->number = atoi(context);
  }

  // 这里可以注册消息回调函数(非必要)
  skynet_callback(ctx, s, simple_callback);

  return 0; // 返回 0 表示初始化成功
}

// 3. 释放函数 (simple_release)
// 当服务被退出时，Skynet 会调用此函数来释放资源。
void
simple_release(struct simple * s) {
  printf("Simple Service Release, number = %d\n", s->number);
  skynet_free(s); // 释放内存
}
```

## 编译成动态链接库
你需要编写一个 `Makefile`，将你的 C 代码编译成 Skynet 可以加载的 `.so` 文件。
``` makefile
# Makefile
# 设置 Skynet 的根目录路径。
SKYNET_PATH ?= ../../skynet-1.8.0
SKYNET_LUA_PATH ?= $(SKYNET_PATH)/3rd/lua
SKYNET_CSERVICE_PATH ?= $(SKYNET_PATH)/cservice

# 包含 Skynet 的编译配置
include $(SKYNET_PATH)/platform.mk

# 设置需要的额外编译参数
# MYCFLAGS = 
CFLAGS = -g -O2 -Wall -I$(SKYNET_LUA_PATH) $(MYCFLAGS)

# 你的服务名，通常和你的 C 文件名一致。
CSERVICE = simple

define CSERVICE_TEMP
  $$(SKYNET_CSERVICE_PATH)/$(1).so : ./service_$(1).c | $$(SKYNET_CSERVICE_PATH)
    $$(CC) $$(CFLAGS) $$(SHARED) $$< -o $$@ -I$(SKYNET_PATH)/skynet-src
endef

$(foreach v, $(CSERVICE), $(eval $(call CSERVICE_TEMP,$(v))))

all : \
    $(foreach v, $(CSERVICE), $(SKYNET_CSERVICE_PATH)/$(v).so) 

$(SKYNET_CSERVICE_PATH) :
    mkdir -p $(SKYNET_CSERVICE_PATH)
```
然后在终端执行：
``` bash
make linux
```
确保最终生成的.so文件路径已配置在 `cpath` 中


## 在 Lua 中启动 C 服务
通常我们使用 `skynet.newservice` 启动lua服务，但我们无法直接使用 · 启动一个C服务，我们需要使用 `skynet.call(".launcher", "lua" , "LAUNCH", "服务名")` 启动C服务。当然也可以直接使用 `skynet.launch("服务名")`，但是这会跳过 launcher 服务的服务管理（不建议）。
``` lua
skynet.call(".launcher", "lua" , "LAUNCH", "simple")
```
