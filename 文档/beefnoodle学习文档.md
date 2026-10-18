# BeefNoodle 学习文档

---

## 一、初步了解 BeefNoodle

### （一）它是什么

**BeefNoodle 是一门数值计算脚本语言。**

* 写工程计算、试验数据处理、公式推导这类脚本
* 但**必须是能自己跑起来的独立程序**：要么用 `beefnoodle.exe` 解释执行，要么编译成
  一个**不依赖任何东西的 .exe**，拷到别的电脑上双击就能跑
* 语法上贴近数学书写：`A * B` 是矩阵乘，`1:2:9` 是数列

**同一个程序有四种跑法，输出逐字节一致**。

| 跑法 | 命令 |
|---|---|
| 树遍历解释器 | `beefnoodle.exe run 程序.bn` |
| 字节码虚拟机 | `beefnoodle.exe vm 程序.bn` |
| 打包成 exe | `beefnoodle.exe build 程序.bn` |
| 机器码 | `beefnoodle.exe native 程序.bn` |

### （二）它能做什么

* **数值计算**：矩阵、向量、行列式、逆、特征值、拟合、插值、积分
* **数据处理**：非参检验 `signrank`、`ranksum`、`kendalltau`，相关系数，统计量
* **写库**：函数、结构体、类（对象）、分文件 `include`
* **做小工具**：读命令行参数 `argv()`、读标准输入 `readline()`、输出 `print`/`fprintf`
* **调试**：内置文本协议的调试服务器

---

## 二、环境准备

### （一） BeefNoodle

### （二）第一条命令

打开 PowerShell，进入解压目录（下面用 `D:\beefnoodle` 举例）：

```powershell
cd D:\beefnoodle
.\beefnoodle.exe version
```

看到 `BeefNoodle 0.9.0` 就成功了。

### （三）在 VS Code 里写代码

1. 装插件：命令行（VS Code 的 `code` 命令）或界面"扩展 → … → 从 VSIX 安装"：

   ```powershell
   code --install-extension 编辑器插件\beefnoodle-*.vsix
   ```

2. 把编译器放到固定位置（插件默认去这里找）：

   ```powershell
   New-Item -ItemType Directory -Force $env:USERPROFILE\.beefnoodle | Out-Null
   Copy-Item .\beef.exe $env:USERPROFILE\.beefnoodle\beef.exe
   ```

3. 新建 `test.bn`，写第一行代码，然后：

   * **F7** → 直接运行（看到输出在终端里）
   * **Ctrl+F7** → 先编译成独立 exe，再运行
   * 也可以命令面板（`Ctrl+Shift+P`）输入 `BeefNoodle` 看所有命令

---

## 三、程序学习

## 第 1 章 第一个程序：输出与注释

```beefnoodle
// hello.bn —— 第一个程序
a = 3;                   // 赋值：不会有任何输出
b = 4;
print(a + b);            // 想要输出，必须写 print
print("勾股定理：");
print(sqrt(a^2 + b^2));

// => 7
// => 勾股定理：
// => 5
```

## 第 2 章 变量与类型

BeefNoodle 是动态类型：变量不用声明就能用。但**也允许先声明类型**，看你习惯：

```beefnoodle
// 直接赋值，类型自动定
n = 42;
x = 3.5;
s = "牛肉面";
ok = true;

print(class(n));      // 
print(class(s));      // 
print(class(ok));     // 

// 也可以先声明再赋值
int m = 7;
double y = 2.5;
long long big = 1000000;

print(m + y);

// => double
// => string
// => logical
// => 9.5
```

---

## 第 3 章 矩阵

**按列优先存储，下标从 1 开始。**

```beefnoodle
A = [1 2 3; 4 5 6; 7 8 9];   // 分号 = 换行（也可以真换行写）

print(size(A));      // 3  3   行列数
print(A(2, 3));      // 6     第 2 行第 3 列
print(A(:, 2));      // 第 2 列：2 5 8
print(A(2:end, :));  // 后两行
print(A(end));       // 9     线性索引，列优先的最后一个
print(A(:)');        // 1..9  展平成列向量后转置成行

v = [1 2 3];
v(5) = 7;            // 线性索引赋值会自动加长，中间补 0
print(v);

// => 3  3
// => 6
// => 2
// => 5
// => 8
// => 4  5  6
// => 7  8  9
// => 9
// => 1  4  7  2  5  8  3  6  9
// => 1  2  3  0  7
```

---

## 第 4 章 运算与优先级

带点的运算符是**逐元素**，不带点的才是矩阵运算。

```beefnoodle
A = [1 2; 3 4];
B = [5 6; 7 8];

print(A * B);          // 矩阵乘
print(A .* B);         // 逐元素乘
print(A .^ 2);         // 逐元素平方
print(2^3^2);          // 512：^ 右结合
print(-2^2);           // -4：一元负号比 ^ 松
print([1 2 3] > 1.5);  // 逐元素比较 -> 0 1 1
print(true && false);  // false：标量逻辑（短路）

// => 19  22
// => 43  50
// => 5  12
// => 21  32
// => 1   4
// => 9  16
// => 512
// => -4
// => 0  1  1
// => false
```

| 类别 | 运算符 |
|---|---|
| 算术 | `+ - * / ^` |
| 逐元素 | `.* ./ .^`（**别漏点**） |
| 比较 | `== != < <= > >=`（不等于是 `!=`） |
| 逻辑 | `&& \|\| ~`（标量、短路）；`& \|`（逐元素） |
| 优先级 | 一元 > `^`（右结合）> `* /` > `+ -` > 比较 > 逻辑 |

> 没有 `1 < 2 < 3`这种

---

## 第 5 章 判断与循环

块用 `end` 结束。

```beefnoodle
score = 78;

if score >= 90
  print("优秀");
elseif score >= 60
  print("及格");
else
  print("不及格");
end

total = 0;
for i = 1:100
  total = total + i;
end
print(total);

n = 0;
while n < 5
  n = n + 1;
  if n == 3
    continue;        // 跳过本次
  end
  if n == 5
    break;           // 跳出循环
  end
  print(n);
end

// => 及格
// => 5050
// => 1
// => 2
// => 4
```

---

## 第 6 章 函数

`返回类型 名字(参数) { return 值; }`

```beefnoodle
// 一个入参、一个返回值
double square(double x) {
  return x * x;
}

// 没有返回值就写 void
void banner(string title) {
  print("=== " + title + " ===");
}

// 递归、互相调用都可以（函数声明提升到顶层）
double fib(double n) {
  if n < 2
    return n;
  end
  return fib(n - 1) + fib(n - 2);
}

// 想返回多个值：打包成矩阵或结构体一次交回
double stats(double v) {
  return [sum(v), prod(v)];        // [和, 积]
}

print(square(7));
banner("标定结果");
print(fib(20));

sp = stats([1 2 3 4]);
print(sp(1));
print(sp(2));

// => 49
// => === 标定结果 ===
// => 6765
// => 10
// => 24
```

* 声明：`返回类型 名字(参数) { … }`（只有这一种）
* 返回值：`return 一个值`；多个值 → 矩阵/结构体
* 参数按值传递（矩阵会拷贝）；结构体/对象按引用
* 函数是值：`f = square; f(9)` 可以

---

## 第 7 章 结构体

```beefnoodle
// 点号赋值会自动建结构体
p.x = 3;
p.y = 4;
print(p);


cfg.model.name = "triaxial";
cfg.model.step = 20000;
print(cfg.model.name);
print(cfg.model.step);

// 也可以显式构造
person = struct("name", "张三", "age", 27);
print(person.name + " 今年 " + person.age + " 岁");
print(isfield(person, "age"));
print(fieldnames(person));

// => struct(x = 3, y = 4)
// => triaxial
// => 20000
// => 张三 今年 27 岁
// => true
// => name, age
```

* 建：点号赋值 / `struct(...)`；查：`isfield` `fieldnames` `isstruct`
* 访问：`s.a`、`s.a.b`；字段不存在会报错（不会静默给空）
* 引用语义，可以当"可变对象"用

---

## 第 8 章 类与对象

类 = 固定字段 + 方法。
构造函数与类同名。

```beefnoodle
class Point
  x, y

  Point(double x, double y) {          // 与类同名 = 构造函数
    self.x = x;
    self.y = y;
  }

  double dist(Point other) {
    return sqrt((self.x - other.x)^2 + (self.y - other.y)^2);
  }

  Point shift(double dx) {             
    self.x = self.x + dx;
    return self;
  }

  void show() {
    print("(" + self.x + ", " + self.y + ")");
  }
end

p = Point(3, 4);
q = Point(0, 0);

print(p.dist(q));      
print(p.x);
p.shift(10);           
print(p.x);
print(p.shift(1).x);   
p.show();

move_right(p);        
print(p.x);

void move_right(Point o) {
  o.x = o.x + 1;
}

// => 5
// => 3
// => 13
// => 14
// => (14, 4)
// => 15
```

* 定义：`class 名字` / 字段一行 / 方法若干 / `end`
* 构造函数：与类同名（`Point(...) { }`），`self` 由运行时给
* 方法：`返回类型 名字(参数) { }`，`self` 隐式
* 查询：`isobject(p)` `class(p)` `properties(p)` `fieldnames(p)`
* 对象是引用，矩阵是值 

---

## 第 9 章 分文件：include

程序写大了就拆开。

```beefnoodle
// lib_math.bn 
double square(double x) {
  return x * x;
}

double stats(double v) {
  return [sum(v), prod(v)];
}
```

```beefnoodle
// main.bn —— 一行 include 把库读进来
include "lib_math.bn"

print(square(6));        // 36
sp = stats([1 2 3 4]);
print(sp(1));            // 10
print(sp(2));            // 24
```

* `include "路径"`，相对本文件、去重、可嵌套
* 函数声明提升到顶层，include 放开头或结尾都行
* REPL 和 `-e "..."` 没有"文件所在目录"，不处理 include

---

## 第 10 章 命令行参数与标准输入

```beefnoodle
// 参数：beefnoodle.exe run 程序.bn hello 42
print("参数个数：" + num2str(argc()));


if argc() >= 1
  print("第 1 个：" + argv(1));
end
if argc() >= 2
  print("第 2 个：" + argv(2));
end
print("全部：" + argv());

// 标准输入：一行一行读（没输入时 readline() 返回空串，循环自然结束）
line = readline();
while length(line) > 0
  print("读到的内容：" + line);
  line = readline();
end

// 上面这个程序不带参数、也没有输入时，输出只有两行

// => 参数个数：0
// => 全部：
```



---

## 第 11 章 读写文件

数据常常在文件里。BeefNoodle 有 5 个文件相关的内置函数：

| 函数 | 作用 |
|---|---|
| `readfile(path)` | 读整个文本文件，返回字符串 |
| `writefile(path, text)` | **覆盖**写，返回写入的字节数 |
| `appendfile(path, text)` | **追加**到末尾（文件不存在就新建），返回字节数 |
| `fileexists(path)` | 文件或目录是否存在 |
| `filesize(path)` | 文件大小（字节） |

```beefnoodle
p = "试验记录.txt";
writefile(p, "第 1 组：sigma=120 kPa\n");
appendfile(p, "第 2 组：sigma=200 kPa\n");

print(fileexists(p));
print(filesize(p));
print(readfile(p));

// => true
// => 52
// => 第 1 组：sigma=120 kPa
// => 第 2 组：sigma=200 kPa
```

* 读：`readfile`；写：`writefile`（覆盖）/ `appendfile`（追加）；查：`fileexists` `filesize`
* 中文文件名、中文内容都没问题（UTF-8）
* `length(s)` 数的是 **UTF-8 字节数**，所以 `length(readfile(p))` 永远等于 `filesize(p)`

---

## 第 12 章 常用的字符串操作

```beefnoodle
s = "sigma=120 kPa";
print(length(s));                  // 字符个数
print(num2str(3.5));               // 数字转字符串
print(strtrim("  两边有空格  "));   // 去首尾空白
print(upper("abc"));               // 转大写
print(lower("ABC"));               // 转小写
print("拼接" + "没问题");

// => 13
// => 3.5
// => 两边有空格
// => ABC
// => abc
// => 拼接没问题
```

* `length(s)`：字符串长度（也算矩阵元素个数，所以长度单位是"字符"）。
* `num2str(x)`：数字 → 字符串；`strtrim` 去掉两端的空格。
* `+` 号在两边都是字符串时表示拼接。
* 想加格式化内容（比如保留几位小数）用 `sprintf` / `fprintf`，它们认 `%d`/`%f`/`%s` 这些老式格式符。

---

## 四、常见报错与解决

|                                                              |                                 |                                                              |
|---|---|---|
| `控制流块用 'end' 结束（'{ }' 只用于函数/方法体）` | `if`/`for`/`while` 后面写了 `{` | 把 `{` 换成直接换行，块结束写 `end` |
| `Java-style functions need a { } body` | 函数体没写 `{ }` | 函数/方法体必须用 `{ }` 包起来 |
| `expected 'end' before end of input` | 少写了 `end` | 数一数每个 `if`/`for`/`while`/`class` 的 `end` |
| `undefined variable or function 'x'` | 名字打错，或变量还没赋值 | 检查拼写；先用 `print(class(x))` 确认 |
| `'Point' has no field 'zzz'` | 用了没声明的类字段 | 在类体里加上字段名 |
| `call stack overflow: recursion is too deep (limit 512 frames)` | 递归超过 512 层 | 改成循环，或减小问题规模 |
| `index 9 out of range` | 下标越界 | 用 `size(A)` 检查维度；下标从 1 开始 |
| VS Code 里 F7 没反应 | 插件找不到编译器 | 把 `beef.exe` 放到 `%USERPROFILE%\.beefnoodle\`，或命令面板跑 `BeefNoodle: 安装/更新编译器` |

---

