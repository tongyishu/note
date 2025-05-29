# GObject/GObjectClass

glib2 本质上是使用面向过程的C语言构造面向对象的代码。

glib2 使用两种类型来表示一个"类"：GObject（存放成员变量）和GObjectClass（存放成员方法）。这两种结构体的实例为分别称为"对象实例"和"类实例"。"对象实例"可以有多个，而"类实例"只能有一个。


| **特性**     | **GObject（实例）**               | **GObjectClass（类）**                   |
| -------------- | ----------------------------------- | ------------------------------------------ |
| **存储内容** | 实例数据（如属性、状态）          | 类行为（如方法、信号、虚函数）           |
| **生命周期** | 动态创建和销毁（如g_object_new）  | 单例，随类型注册生成，程序生命周期内存在 |
| **继承关系** | 继承父类的实例结构体（如GObject） | 继承父类的类结构体（如GObjectClass）     |
| **访问方式** | 通过对象指针                      | 通过G_TYPE_INSTANCE_GET_CLASS            |
| **典型操作** | 修改属性、调用实例方法            | 定义虚函数、注册信号、覆盖父类方法       |

# G_DEFINE_TYPE

G_DEFINE_TYPE用于简化 **GObject 类型的定义和注册** 。它会自动生成与 GObject 类型系统兼容的代码，包括类初始化、实例初始化以及父子类关系的绑定。该宏展开后生成`xxx_get_type()`函数，该函数返回要定义类的ID，此ID全局唯一。

示例代码：

```c
#include <glib-object.h>

typedef struct __Tongyishu Tongyishu;
typedef struct __TongyishuClass TongyishuClass;

struct __Tongyishu {
	GObject parent;
	gint type;
};

struct __TongyishuClass {
	GObjectClass parent_class;
};

/*
G_DEFINE_TYPE(TN, t_n, T_P)
  @TN  自定义类型，驼峰格式
  @t_n 自定义类型，小写字母加下划线格式
  @T_P 父类型
*/
G_DEFINE_TYPE(Tongyishu, tongyishu, G_TYPE_OBJECT);

static void tongyishu_init(Tongyishu *self)
{
}
static void tongyishu_class_init(TongyishuClass *kclass)
{
}

int main(int argc, char *argv[])
{
	Tongyishu *tongyishu = g_object_new(tongyishu_get_type(), NULL);
	g_print("+++ tongyishu_get_type() = %ld\n", tongyishu_get_type());
	return 0;
}

```

预处理：

```bash
gcc -E ./test.c \
	-lgobject-2.0 \
	-lglib-2.0 \
	-I/usr/include/glib-2.0 \
	-I/usr/lib64/glib-2.0/include \
	-I/usr/lib/x86_64-linux-gnu/glib-2.0/include
```

G_DEFINE_TYPE(Tongyishu, tongyishu, G_TYPE_OBJECT)宏展开：

```c
static void tongyishu_init(Tongyishu *self);
static void tongyishu_class_init(TongyishuClass *klass);
static GType tongyishu_get_type_once(void);
static gpointer tongyishu_parent_class = ((void *)0);
static gint Tongyishu_private_offset;
static void tongyishu_class_intern_init(gpointer klass)
{
	tongyishu_parent_class = g_type_class_peek_parent(klass);
	if (Tongyishu_private_offset != 0)
		g_type_class_adjust_private_offset(klass, &Tongyishu_private_offset);
	tongyishu_class_init((TongyishuClass *)klass);
}
__attribute__((__unused__)) static inline gpointer tongyishu_get_instance_private(Tongyishu *self)
{
	return (((gpointer)((guint8 *)(self) + (glong)(Tongyishu_private_offset))));
}
GType tongyishu_get_type(void)
{
	static gsize static_g_define_type_id = 0;
	if ((__extension__({
		    _Static_assert(sizeof *(&static_g_define_type_id) == sizeof(gpointer),
				   "Expression evaluates to false");
		    (void)(0 ? (gpointer) * (&static_g_define_type_id) : ((void *)0));
		    (!(__extension__({
			    _Static_assert(sizeof *(&static_g_define_type_id) == sizeof(gpointer),
					   "Expression evaluates to false");
			    __typeof__(*(&static_g_define_type_id)) gapg_temp_newval;
			    __typeof__((&static_g_define_type_id)) gapg_temp_atomic =
				(&static_g_define_type_id);
			    __atomic_load(gapg_temp_atomic, &gapg_temp_newval, 5);
			    gapg_temp_newval;
		    })) &&
		     g_once_init_enter(&static_g_define_type_id));
	    }))) {
		GType g_define_type_id = tongyishu_get_type_once();
		(__extension__({
			_Static_assert(sizeof *(&static_g_define_type_id) == sizeof(gpointer),
				       "Expression evaluates to false");
			0 ? (void)(*(&static_g_define_type_id) = (g_define_type_id)) : (void)0;
			g_once_init_leave((&static_g_define_type_id), (gsize)(g_define_type_id));
		}));
	}
	return static_g_define_type_id;
}
__attribute__((__noinline__)) static GType tongyishu_get_type_once(void)
{
	GType g_define_type_id = g_type_register_static_simple(
	    ((GType)((20) << (2))), g_intern_static_string("Tongyishu"), sizeof(TongyishuClass),
	    (GClassInitFunc)(void (*)(void))tongyishu_class_intern_init, sizeof(Tongyishu),
	    (GInstanceInitFunc)(void (*)(void))tongyishu_init, (GTypeFlags)0);
	{
		{
			{};
		}
	}
	return g_define_type_id;
};
```

编译：

```bash
gcc -g -o ./test ./test.c \
	-lgobject-2.0 \
	-lglib-2.0 \
	-I/usr/include/glib-2.0 \
	-I/usr/lib64/glib-2.0/include \
	-I/usr/lib/x86_64-linux-gnu/glib-2.0/include
```

运行结果：

```bash
# ./test
+++ tongyishu_get_type() = 93919452136496
```
