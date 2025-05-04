## 前置
掌握hPy v2固件编译.
建议使用VSCode编辑器
对应教程：[[hPy v2 固件编译教程]]
## 模块格式/要求
模块代码应位于`/port/boards/mpython`下
格式如下:
```c
// 必要的运行库
#include "py/obj.h"
#include "py/runtime.h"
#include "py/builtin.h"

// 定义模块函数
STATIC mp_obj_t example_add_ints(mp_obj_t a_obj, mp_obj_t b_obj) {
    int a = mp_obj_get_int(a_obj);
    int b = mp_obj_get_int(b_obj);
    return mp_obj_new_int(a + b);
}
// 将该函数与python对象绑定起来
STATIC MP_DEFINE_CONST_FUN_OBJ_2(example_add_ints_obj, example_add_ints);

// 定义模块对象属性
// 此表由mpy对象名(标识符)与对应对象组成
// 所有标识符应均以QSTR(MP_QSTR_xxx)的形式存在
STATIC const mp_rom_map_elem_t example_module_globals_table[] = {
    { MP_ROM_QSTR(MP_QSTR___name__), MP_ROM_QSTR(MP_QSTR_example) },
    { MP_ROM_QSTR(MP_QSTR_add_ints), MP_ROM_PTR(&example_add_ints_obj) },
};
STATIC MP_DEFINE_CONST_DICT(example_module_globals, example_module_globals_table);

// 定义模块对象
const mp_obj_module_t example_user_cmodule = {
    .base = { &mp_type_module },
    .globals = (mp_obj_dict_t*)&example_module_globals,
};
```

在编写完模块后 你需要修改`/port/mpconfigport.h`/`Makefile`以使模块能够被正确集成并工作：
1. 使模块能够被识别
	1. 在`Makefile`中搜索关键字`# List of MicroPython source and object files`
	2. 向下滑动 找到**SRC_C**的定义位置 为最后一行加上反斜杠`\` 并换行 输入你模块的文件名
2. 声明模块
	1. 在`mpconfigport.sh`中搜索`// extra built in modules to add to the list of known ones`
	2. 向下滑动 在这一堆代码的最后方加上`extern const struct _mp_obj_module_t [你模块对象的名字];`
3. 注册模块
	1. 在刚刚声明模块的地方往下滑 找到`BOARD_PORT_BUILTIN_MODULES`这一行
	2. 在该行的上一行新增`{ MP_ROM_QSTR(MP_QSTR_\[模块被导入时所需的名字\]), (mp_obj_t)(&\[模块对象名]\) }, \`
4. 注册QSTR
	- 在`/port/qstrdefsport.h`下 用`Q( )`包裹你的QSTR 一行一个

## 模块编写
### 导入
相较于非内嵌式的C模块 此类模块的导入可以说是非常自由的 你几乎可以调用大部分的py api 下面给各位列举几个例子：
- [BEGK-RunPyFile](https://github.com/gxxk-dev/BaseGeek/blob/master/port/boards/mpython/run_pyfile.c)
	- 它来自于笔者的一款已经放弃开发的个人作品
	- 它导入了`pyexec` 因此它可以更加直接地运行本地文件/冻结模块
> Tip:在hPy中 来自Py2的execfile仍然可用
- [BEGK-RawOLed](https://github.com/gxxk-dev/BaseGeek/blob/master/port/boards/mpython/raw_oled.c)
	- 它同样来自于笔者的那款已经放弃开发的个人作品
	- 它导入了hPy v2 oled屏幕的驱动/依赖 同时凭借着C语言本身的速度优势 大大减小了绘制/刷新的时间.
### 函数定义
在笔者个人的理解中 在此类模块中创建函数分为3部分
1. 函数本体
	- 函数的核心部分
2. 函数绑定
	- 主要用于参数标注 为函数绑定一个抽象标识符
3. 函数链接
	- 通过抽象标识符 为函数分配一个在micropy中可直接调用的名称
#### Py对象
在Python中我们所使用的各种数据类型 在C这里都会归类为特定的抽象对象 不能够直接操作
因此 我们需要将其转换为我们在c中熟悉的数据类型
##### int
int->mp_int_t
```c
mp_obj_t xxxxxxxxxx=mp_obj_new_int(123)
```
mp_int_t->int
```c
int xxxxxxxxxx=mp_obj_get_int([mp_obj_t对象])
```
> TODO:待补充...

#### 参数注明
在绑定函数时 `MP_DEFINE_CONST_FUN_OBJ_`的后方跟了一个数字.
该数字代表着此函数将接收多少参数. 范围:0-3 若超出范围 则需要填写为`X`
> TODO:多参转换待补充...