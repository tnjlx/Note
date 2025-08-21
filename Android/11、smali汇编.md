# 1. 代码结构
每个类对应一个.smali 文件。
```smali
# .class public → 定义类
# .super → 继承 Activity
.class public Lcom/example/MainActivity;
.super Landroid/app/Activity;

# .method public onCreate()V → 定义 onCreate 方法
# .locals 1 → 分配 1 个寄存器 v0
# const-string v0, "Hello Smali!" → 加载字符串到 v0
# invoke-static {v0}, Ljava/lang/System;->println(Ljava/lang/String;)V → 调用 System.out.println
# return-void → 返回
.method public onCreate()V
    .locals 1
    const-string v0, "Hello Smali!"
    invoke-static {v0}, Ljava/lang/System;->println(Ljava/lang/String;)V
    return-void
.end method
```

# 2. 关键指令
## 2.1. 变量寄存器
```smali
.locals 2
const/4 v0, 1
const/16 v1, 100
# v开头为变量寄存器
```

## 2.2. 方法调用
```smali
# 调用静态方法
invoke-static {}, Lcom/example/Utils;->getFlag()Ljava/lang/String;
# 调用实例方法
invoke-virtual {p0}, Lcom/example/User;->getName()Ljava/lang/String;
# 调用构造方法
invoke-direct {p0}, Lcom/example/MainActivity;-><init>()V
```

## 2.3. 条件与跳转
```smali
# 等于判断
if-eq v0, v1, :label_true
# 大于判断
if-gt v0, v1, :label_greater

# 跳转
:label_true
    const-string v0, "True!"
    goto :label_end

:label_greater
    const-string v0, "Greater!"

:label_end
    return-void
```

