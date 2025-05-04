## 前置
掌握hPy v2固件编译.
建议使用VSCode编辑器
对应教程：[[hPy v2 固件编译教程]]
掌握[[为hPy v2固件内嵌基于C的Py模块]]技能
> 二者都是贯通的 只有相辅相成才能更上一层楼.
## 模块要求
```c
// 此处头文件用于访问mPy API
#include "py/dynruntime.h"

// 注册函数
STATIC mp_obj_t hello_world() {
    mp_printf(&mp_plat_print,"Hello World！\n");
    return mp_const_none;
}
// 绑定对应函数
STATIC MP_DEFINE_CONST_FUN_OBJ_0(HelloWorld, hello_world);

// 导入模块时调用此处 注册对应对象
mp_obj_t mpy_init(mp_obj_fun_bc_t *self, size_t n_args, size_t n_kw, mp_obj_t *args) {
    // 标准结构 函数结尾的 MP_DYNRUNTIME_INIT_EXIT 也一样 勿动
    MP_DYNRUNTIME_INIT_ENTRY

    // 注册HelloWorld函数
    mp_store_global(MP_QSTR_HelloWorld, MP_OBJ_FROM_PTR(&HelloWorld));
    
    
    MP_DYNRUNTIME_INIT_EXIT
}
```
在编写完毕后 只需要如下`Makefile`即可编译(记得替换对应路径/名称)：
```Makefile
# microPython本地仓库位置
MPY_DIR = ../micropython

# 模块名称
MOD = test

# 源代码
SRC = example.c

# 构建架构 勿动
# xtensawin为掌控板的架构
ARCH = xtensawin

# 导入编译和链接模块的脚本
include $(MPY_DIR)/py/dynruntime.mk
```