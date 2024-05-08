
QEMU提供了一套面向对象编程的模型——QOM，即QEMU Object Model，几乎所有的设备如CPU、内存、总线等都是利用这一面向对象的模型来实现的。

QOM模型的实现代码位于qom/文件夹下的文件中，这涉及了几个结构TypeImpl, ObjectClass, Object和TypeInfo。它们的定义都在`include/qom/object.h`可以找到，只有TypeImpl的具体结构是在`qom/object.c`中。

> 下面代码流程都是根据qemu-4.1.0代码写的，与最新的qemu代码结构差不多，部分可能存在差异。

### 1、各模块初始化函数注册，TypeInfo转为TypeImpl

attribute属性constructor即构造函数属性，该属性的设置使得该函数在main函数之前被执行，所以qemu中会先执行调用module_init的方法，调用module_init的一些方法block_init、opts_init、type_init、trace_init、xen_backend_init、libqos_init会先于main函数执行。

[`__attrivbute__((constructor))用法`](https://www.jianshu.com/p/dd425b9dc9db)

```c
#define module_init(function, type)                                         \
// __attrivbute__((constructor))属性的函数为构造函数，先于main函数执行
static void __attribute__((constructor)) do_qemu_init_ ## function(void)    \
{                                                                           \
    register_module_init(function, type);                                   \
}

typedef enum {
    MODULE_INIT_BLOCK,
    MODULE_INIT_OPTS,
    MODULE_INIT_QOM,
    MODULE_INIT_TRACE,
    MODULE_INIT_XEN_BACKEND,
    MODULE_INIT_LIBQOS,
    MODULE_INIT_MAX
} module_init_type;

static ModuleTypeList init_type_list[MODULE_INIT_MAX];

static ModuleTypeList *find_type(module_init_type type)
{
    init_lists();

    return &init_type_list[type];
}

// 注册各模块到init_type_list中，每个模块有个链表
void register_module_init(void (*fn)(void), module_init_type type)
{
    ModuleEntry *e;
    ModuleTypeList *l;

    e = g_malloc0(sizeof(*e));
    e->init = fn;
    e->type = type;
    e->initialized = false;

    l = find_type(type); // 根据类型找到ModuleTypeList列表

    QTAILQ_INSERT_TAIL(l, e, node); // 将注册的各初始化方法放入列表末尾
}
```

```c
// 每个模块封装成不同的方法
#define block_init(function) module_init(function, MODULE_INIT_BLOCK)
#define opts_init(function) module_init(function, MODULE_INIT_OPTS)
#define type_init(function) module_init(function, MODULE_INIT_QOM)
#define trace_init(function) module_init(function, MODULE_INIT_TRACE)
#define xen_backend_init(function) module_init(function, \
                                               MODULE_INIT_XEN_BACKEND)
#define libqos_init(function) module_init(function, MODULE_INIT_LIBQOS)
```

以x86 cpu的注册函数为例，target/i386/cpu.c代码如下：

```c
static const TypeInfo x86_cpu_type_info = {
    .name = TYPE_X86_CPU, // 最终qom/object.c中type_table这个HashTable的key值
    .parent = TYPE_CPU, // TYPE_X86_CPU的父对象为TYPE_CPU
    .instance_size = sizeof(X86CPU),
    .instance_init = x86_cpu_initfn,
    .abstract = true,
    .class_size = sizeof(X86CPUClass),
    .class_init = x86_cpu_common_class_init,
};
static const TypeInfo x86_base_cpu_type_info = {
        .name = X86_CPU_TYPE_NAME("base"), // 最终qom/object.c中type_table这个HashTable的key值
        .parent = TYPE_X86_CPU,
        .class_init = x86_cpu_base_class_init,
};

static void x86_cpu_register_types(void)
{
    int i;

    type_register_static(&x86_cpu_type_info);
    for (i = 0; i < ARRAY_SIZE(builtin_x86_defs); i++) {
        x86_register_cpudef_types(&builtin_x86_defs[i]);
    }
    type_register_static(&max_x86_cpu_type_info);
    type_register_static(&x86_base_cpu_type_info);
#if defined(CONFIG_KVM) || defined(CONFIG_HVF)
    type_register_static(&microvm_x86_cpu_type_info);
    type_register_static(&host_x86_cpu_type_info);
#endif
}
// 这里调用type_init，就会相当于构造函数，先于main函数注册
type_init(x86_cpu_register_types)
```

将TypeInfo转换为TypeImpl，插入到HashTable type_table里
qom/object.c

```c
// 获取type_table，如果未初始化，调用g_hash_table_new初始化一个新的type_table
static GHashTable *type_table_get(void)
{
    static GHashTable *type_table;
    // static函数和变量，type_table相当于全局变量，仅初始化一次
    if (type_table == NULL) {
        type_table = g_hash_table_new(g_str_hash, g_str_equal);
    }

    return type_table;
}
// 最后插入HashTable，key为TypeImpl->name,value为TypeImpl
static void type_table_add(TypeImpl *ti)
{
    assert(!enumerating_types);
    g_hash_table_insert(type_table_get(), (void *)ti->name, ti);
}
// TypeInfo转换为TypeImpl
static TypeImpl *type_new(const TypeInfo *info)
{
    TypeImpl *ti = g_malloc0(sizeof(*ti));
    int i;

    g_assert(info->name != NULL);

    if (type_table_lookup(info->name) != NULL) {
        fprintf(stderr, "Registering `%s' which already exists\n", info->name);
        abort();
    }

    ti->name = g_strdup(info->name);
    ti->parent = g_strdup(info->parent);

    ti->class_size = info->class_size;
    ti->instance_size = info->instance_size;

    ti->class_init = info->class_init;
    ti->class_base_init = info->class_base_init;
    ti->class_data = info->class_data;

    ti->instance_init = info->instance_init;
    ti->instance_post_init = info->instance_post_init;
    ti->instance_finalize = info->instance_finalize;

    ti->abstract = info->abstract;

    for (i = 0; info->interfaces && info->interfaces[i].type; i++) {
        ti->interfaces[i].typename = g_strdup(info->interfaces[i].type);
    }
    ti->num_interfaces = i;

    return ti;
}
static TypeImpl *type_register_internal(const TypeInfo *info)
{
    TypeImpl *ti;
    ti = type_new(info);

    type_table_add(ti);
    return ti;
}

TypeImpl *type_register(const TypeInfo *info)
{
    assert(info->parent);
    return type_register_internal(info);
}

TypeImpl *type_register_static(const TypeInfo *info)
{
    return type_register(info);
}
```

### 2、各模块初始化

vl.c的main函数中调用各模块初始化

```c
int main(int argc, char **argv, char **envp)
{
    ……
    module_call_init(MODULE_INIT_TRACE);
    module_call_init(MODULE_INIT_QOM);
    module_call_init(MODULE_INIT_OPTS);
    ……
}
```

```c
void module_call_init(module_init_type type)
{
    ModuleTypeList *l;
    ModuleEntry *e;

    l = find_type(type);

    QTAILQ_FOREACH(e, l, node) {
        if (!e->initialized) {
            e->init(); // 此处对比上面例子，相当于调用x86_cpu_register_types函数，注册TYPE_X86_CPU类型的初始化函数
            e->initialized = true; // 设置标志位initialized，不会重复初始化
        }
    }
}
```

### 3、ObjectClass初始化

以 select_machine为例看下QOM初始化流程
main->select_machine->object_class_get_list->object_class_foreach->type_table_get;object_class_foreach_tramp->type_initialize->class_base_init;class_init

`vl.c`中main函数调用select_machine

```c
machine_class = select_machine();

static MachineClass *select_machine(void)
{
    GSList *machines = object_class_get_list(TYPE_MACHINE, false);
    MachineClass *machine_class = find_default_machine(machines);
    ……
}
```

`qom/object.c`

```c
static void object_class_get_list_tramp(ObjectClass *klass, void *opaque)
{
    GSList **list = opaque;
    // g_slist_prepend(*list, klass);是将klass插入到*list的开头的地方
    *list = g_slist_prepend(*list, klass);
}

// 根据传入的类型TYPE_MACHINE获取对应的ObjectClass
GSList *object_class_get_list(const char *implements_type,
                              bool include_abstract)
{
    GSList *list = NULL;

    object_class_foreach(object_class_get_list_tramp,
                         implements_type, include_abstract, &list);
    return list;
}
void object_class_foreach(void (*fn)(ObjectClass *klass, void *opaque),
                          const char *implements_type, bool include_abstract,
                          void *opaque)
{
    OCFData data = { fn, implements_type, include_abstract, opaque };

    enumerating_types = true;
    // 从HashTable type_table中根据type类型拿到TypeImpl
    g_hash_table_foreach(type_table_get(), object_class_foreach_tramp, &data);
    enumerating_types = false;
}

static void object_class_foreach_tramp(gpointer key, gpointer value,
                                       gpointer opaque)
{
    OCFData *data = opaque;
    TypeImpl *type = value;
    ObjectClass *k;
    // 此处调用machine类型的初始化函数
    type_initialize(type);
    k = type->class;

    if (!data->include_abstract && type->abstract) {
        return;
    }

    if (data->implements_type && 
        !object_class_dynamic_cast(k, data->implements_type)) {
        return;
    }
    // 最后调用data->fn，实际就是调用object_class_get_list_tramp函数
    data->fn(k, data->opaque);
}
```

```c
static void type_initialize(TypeImpl *ti)
{
    TypeImpl *parent;
    // 如果class已被初始化过，则不需要再初始化了
    if (ti->class) {
        return;
    }

    ti->class_size = type_class_get_size(ti);
    ti->instance_size = type_object_get_size(ti);
    /* Any type with zero instance_size is implicitly abstract.
     * This means interface types are all abstract.
     */
    if (ti->instance_size == 0) {
        ti->abstract = true;
    }
    // 中间省略……

    ti->class->type = ti;
    // 如果有父对象，则先初始化父对象，递归调用初始化全部对象
    while (parent) {
        if (parent->class_base_init) {
            parent->class_base_init(ti->class, ti->class_data);
        }
        parent = type_get_parent(parent);
    }
    // 此处初始化这个ObjectClass
    if (ti->class_init) {
        ti->class_init(ti->class, ti->class_data);
    }
}
```

上面实际调用的则是TypeImpl中的class_init，这在最开始先于main函数执行的注册，实际调用的是`machine_class_init`
hw/core/machine.c

```c
static const TypeInfo machine_info = {
    .name = TYPE_MACHINE,
    .parent = TYPE_OBJECT,
    .abstract = true,
    .class_size = sizeof(MachineClass),
    .class_init    = machine_class_init,
    .class_base_init = machine_class_base_init,
    .instance_size = sizeof(MachineState),
    .instance_init = machine_initfn,
    .instance_finalize = machine_finalize,
};

static void machine_register_types(void)
{
    type_register_static(&machine_info);
}

type_init(machine_register_types)
```

> 总结一下就是：
> 1、`__attribute__((constructor))`的修饰函数会先于main函数执行，type_init的参数是XXX_register_types函数指针，将函数指针传递到ModuleEntry的init函数指针，最后就是将这个ModuleEntry插入到ModuleTypeList
> 2、main函数中的module_call_init(MODULE_INIT_QOM);调用了MODULE_INIT_QOM类型的ModuleTypeList中的所有ModuleEntry中的init()函数，也就是第一步type_init的第一个参数XXX_register_types函数指针
> 3、然后就是XXX_register_types函数的初始化操作了，就是创建TypeImpl的哈希表
> 4、ObjectClass获取的时候，会根据Type类型拿到对应的TypeImpl对象，如果class已经被初始化过，就无需初始化，未被初始化，则还是会调用TypeInfo定义的class_init函数初始化对象。

推荐书籍：QEMUKVM源码解析与应用带书签.pdf 第二章QEMU基本纽件


> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5NDU2NjAzMTFdfQ==
-->