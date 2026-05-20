# Python 软件测试面试总结（基础知识+常见题）

## 一、Python基础概念

### 1. Python 解释型还是编译型？

**解释型语言**：代码不需要编译成二进制，运行时逐行解释执行，跨平台性好。

对比：
- 编译型：C/C++，一次性编译成二进制再运行，运行快
- 解释型：Python/JavaScript，运行时解释，开发快，跨平台好

### 2. Python2 和 Python3 区别

| 特性 | Python2 | Python3 |
|------|---------|---------|
| `print` | `print "hello"` | `print("hello")` 必须加括号 |
| 编码 | 默认ASCII | 默认UTF-8 |
| `xrange` | 有 | 直接用，返回迭代器 | 移除，只用`range`就是迭代器 |
| 除法 | `3/2 = 1` 整数除法 | `3/2 = 1.5` 浮点除法 |
| 异常 | `except Exception, e` | `except Exception as e` |

### 3. 可变对象 vs 不可变对象

**不可变对象（修改后地址改变id变化）：
- `int`、`float`、`bool`、`str`、`tuple`

**可变对象（修改后地址不变）**：
- `list`、`dict`、`set`

```python
# 不可变例子
a = 1
print(id(a))  # 地址xxx
a = 2
print(id(a))  # 地址yyy → 变了，因为int不可变

# 可变例子
lst = [1, 2, 3]
print(id(lst))  # 地址xxx
lst.append(4)
print(id(lst))  # 还是xxx → 没变，list可变
```

### 4. `is` vs `==` 区别

- `is`：比较**地址**（是不是同一个对象）
- `==`：比较**值**（内容是不是相等）

```python
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)  # True 值相等
print(a is b)  # False 不是同一个对象，地址不同
```

### 5. `*args` 和 `**kwargs`

- `*args`：接收**可变数量 positional 参数**，打包成tuple
- `**kwargs`：接收**可变数量 keyword 参数**，打包成dict

```python
def func(*args, **kwargs):
    print(args)   # (1, 2, 3)
    print(kwargs) # {'a': 4, 'b': 5}

func(1, 2, 3, a=4, b=5)
```

### 6. 装饰器是什么？举个例子

装饰器本质是**Python语法糖**，作用：**不修改原函数代码，给函数增加额外功能**。

原理：装饰器是一个函数，接收**原函数**作为参数，返回**包装后的新函数**。

---

### 一步步理解，用测试场景举例：

**需求：给每个接口请求函数，增加「打印日志+统计请求耗时」功能**

#### 第一步：不用装饰器，怎么写？
```python
import time

def get_user_info():
    # 手动加日志和计时，每个函数都要写一遍，重复代码
    print(f"[INFO] 开始调用 get_user_info")
    start = time.time()
    
    # ===== 原函数逻辑 =====
    resp = requests.get("https://api.example.com/user/1")
    # ====================
    
    print(f"[INFO] 调用完成，耗时: {time.time() - start:.2f}s")
    return resp
```
问题：每个接口函数都要复制粘贴日志计时代码，重复代码太多，不好维护。

---

#### 第二步：用装饰器，怎么写？
```python
import time
import requests

# 定义装饰器：接收原函数，返回包装后的新函数
def log_time(func):
    def wrapper(*args, **kwargs):
        # ===== 新增功能：打印日志 + 计时 =====
        print(f"[INFO] 开始调用 {func.__name__}")
        start = time.time()
        
        # ===== 执行原函数 =====
        result = func(*args, **kwargs)
        
        # ===== 新增功能：打印耗时 =====
        print(f"[INFO] 调用完成，耗时: {time.time() - start:.2f}s")
        return result
    return wrapper

# 使用装饰器，一句话搞定
@log_time
def get_user_info():
    # 原函数只有核心逻辑，不用写重复代码
    return requests.get("https://api.example.com/user/1")

@log_time
def create_order():
    return requests.post("https://api.example.com/order", json={"goods_id": 1})
```

**效果：**
```
get_user_info()
# 输出：
# [INFO] 开始调用 get_user_info
# [INFO] 调用完成，耗时: 0.23s
```

你看：**不用修改原函数核心代码，只需要加个 `@log_time`，就自动加上了日志计时功能**，这就是装饰器的作用。

---

### 为什么要用 `(*args, **kwargs)`？

为了**兼容任意参数**的原函数，让装饰器通用：
- `*args` 接收所有**位置参数**，打包成 tuple
- `**kwargs` 接收所有**关键字参数**，打包成 dict
- 然后原封不动传给原函数，不管原函数参数是什么签名，都能兼容

举例子：
```python
# 原函数无参数 → 兼容
@log_time
def f():
    pass

# 原函数两个位置参数 → 兼容
@log_time
def add(a, b):
    return a + b

# 原函数有关键字参数 → 兼容
@log_time
def create_user(name, age=18):
    pass
```
如果不写 `*args, **kwargs`，只能装饰特定参数个数的函数，不通用。

---

### 总结：
- 装饰器 = "在不修改原代码的前提下，给函数增加新功能"
- 本质：闭包的应用，接收函数参数，返回新函数

---

## 装饰器在pytest测试框架中的常见场景（实际天天用）：

### 场景1：参数化测试
同一个测试用例，多组输入输出，不用写重复代码：
```python
import pytest

@pytest.mark.parametrize("username, password, expected_code", [
    ("admin", "123456", 0),       # 正确密码，登录成功
    ("admin", "wrong", 1),         # 错误密码，登录失败
    ("", "123456", 2),             # 用户名为空
    ("test", "", 2),               # 密码为空
])
def test_login(username, password, expected_code):
    resp = login(username, password)
    assert resp["code"] == expected_code
```

### 场景2：标记分组执行
```python
import pytest

# 标记为冒烟测试，执行 `pytest -m smoke` 只跑冒烟用例
@pytest.mark.smoke
def test_create_order():
    ...

# 标记为接口测试
@pytest.mark.api
def test_api_get_user():
    ...
```

### 场景3：跳过用例
```python
import pytest
import sys

# 直接跳过，功能还没开发完
@pytest.mark.skip("这个功能还没开发完成，暂不测试")
def test_xxx():
    ...

# 条件跳过：Windows环境跳过，只在Linux跑
@pytest.mark.skipif(sys.platform == "win32", reason="Linux环境才需要测试")
def test_xxx_linux():
    ...
```

### 场景4：fixture前置（fixture本身就是装饰器）
用来做前置准备，复用测试数据：
```python
import pytest

@pytest.fixture(scope="session")
def login_token():
    # 整个测试会话只执行一次：登录获取token
    resp = requests.post("/login", json={"username": "admin", "password": "123456"})
    return resp.json()["token"]

# 测试用例直接注入token，不用每个用例都写登录逻辑
def test_get_user(login_token):
    resp = requests.get(
        "/user", 
        headers={"Authorization": f"Bearer {login_token}"}
    )
    assert resp.status_code == 200
```

### 场景5：自定义装饰器 - 接口失败自动重试
```python
import tenacity
# 第三方装饰器，失败重试3次，间隔2秒
@tenacity.retry(
    stop=tenacity.stop_after_attempt(3), 
    wait=tenacity.wait_fixed(2)
)
def test_create_order():
    resp = requests.post(...)
    assert resp.status_code == 200
```

### 场景6：自定义装饰器 - 统计接口请求耗时
```python
import time
def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} 耗时: {time.time() - start:.2f}s")
        return result
    return wrapper

@timer
def test_get_user():
    return requests.get("https://api.example.com/user")
```

总结：装饰器就是Python给函数"打标签"、"加功能"的语法，pytest框架大量使用装饰器来实现各种特性，非常方便。

### 7. 迭代器 vs 生成器

**迭代器**：实现了 `__iter__()` 和 `__next__()` 方法的对象，可以逐个取值。

**生成器**：特殊的迭代器，用 `yield` 关键字，每次返回一个值，**暂停执行**，下次调用继续执行。

**核心优点：** 生成器**节省内存**，不用一次性把所有数据加载到内存，大数据量场景特别有用。

---

### 软件测试中实际使用场景：

#### 场景1：逐行读取大日志/大测试数据文件
如果测试数据文件有几个G，一次性 `readlines()` 全读到内存，内存直接爆了：

❌ 不好写法：
```python
# 一次性全部读进内存，文件大了直接OOM
with open("huge_test_data.log", "r") as f:
    lines = f.readlines()
for line in lines:
    process(line)
```

✅ 生成器写法：
```python
# 生成器逐行读，一次只占一行内存，文件再大也不怕
def read_large_file(file_path):
    with open(file_path, "r") as f:
        for line in f:  # 文件本身就是迭代器，逐行yield
            yield line

for line in read_large_file("huge_test_data.log"):
    process(line)  # 处理完一行再读下一行
```

#### 场景2：批量造测试数据插入数据库
要造10万条测试订单数据，一次性生成完放内存再插入，内存占用很高：

```python
def generate_test_orders(count):
    for i in range(count):
        # 生成一条订单数据，yield返回，插入数据库后再生成下一条
        yield {
            "order_no": f"TEST-{i:06d}",
            "amount": i * 10,
            "status": "PAID"
        }

# 一次生成一条插入一条，内存一直很低
for order in generate_test_orders(100000):
    db.insert(order)
```

#### 场景3：分页接口遍历所有数据
接口分页返回数据，每次只能拿100条，需要遍历所有数据：

```python
def fetch_all_data(page_size=100):
    page = 1
    while True:
        resp = requests.get(f"https://api.example.com/data?page={page}&size={page_size}")
        data = resp.json()["list"]
        if not data:
            break  # 没数据了停止
        for item in data:
            yield item  # 逐行返回，不用一次性存所有数据
        page += 1

# 遍历处理所有数据，内存占用稳定
for item in fetch_all_data():
    process(item)
```

---

### 生成器执行原理：遇到 `yield` 就暂停，下次调用再继续往下走

**一步步举例说明，拿读大文件例子：

```python
def read_large_file(file_path):
    print("进入函数")
    with open(file_path, "r") as f:
        for line in f:  # 文件本身就是迭代器，逐行读
            yield line  # ✅ 关键在这里！
```

**执行流程：**
1. 你调用 `gen = read_large_file("big.log")` → 得到一个生成器对象，**函数还没开始执行！**
2. 你第一次 `next(gen)` → 函数开始执行，走到 `yield line` → **遇到yield，函数**暂停在这里**，把line返回给你
3. 你处理完这一行，下次要下一行 → 函数从暂停的地方**继续往下走**，循环到下一行，又遇到yield，又暂停返回
4. 重复2-3，直到文件读完，函数结束

### 和一次性读完的区别：

| 写法 | 内存占用 |
|------|----------|
| `lines = f.readlines() | 一次性把**所有行都读到内存list → 文件几个G，内存就占几个G，文件越大占越多 |
| 生成器逐行yield | **一次只加载当前这一行在内存 → 不管文件多大，内存只占一行大小 → 几个G文件也不怕，不会爆内存 |

### 总结：
什么时候用生成器？**大数据量，不需要一次性把所有数据放内存**，就用生成器，省内存，不会OOM。

```python
# 生成器例子：生成斐波那契数列
def fib(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

for num in fib(10):
    print(num)
```

### 8. 深拷贝 vs 浅拷贝

- **浅拷贝（copy.copy()**：只拷贝对象本身，内部引用对象还是共享同一个。修改会影响原对象。
- **深拷贝（copy.deepcopy()**：递归拷贝，所有内部对象都拷贝一份，修改不影响原对象。

```python
import copy

a = [1, [2, 3], 4]
b = copy.copy(a)    # 浅拷贝
c = copy.deepcopy(a) # 深拷贝

b[1][0] = 99
print(a[1][0]) → 99  # 浅拷贝影响原对象
```

**实际测试场景：**
什么时候需要深拷贝？当你需要修改一个字典/列表，但又不想影响原来的数据时：

```python
# 接口测试中，基础请求参数，每个用例要改不同字段，又不想改脏基础数据
base_data = {
    "order_id": 123,
    "user_id": 456,
    "address": {
        "city": "Beijing",
        "street": "Road"
    }
}

import copy
# 深拷贝一份，改拷贝后的不影响原数据，下一个用例还能用原数据
test_data = copy.deepcopy(base_data)
test_data["order_id"] = 456
test_data["address"]["city"] = "Shanghai"
```

**一句话总结：** 如果对象里面嵌套了其他对象（list套list，dict套dict），想完全独立一份改了不影响原对象就用深拷贝。

### 9. `@staticmethod` vs `@classmethod` vs 实例方法

| 类型 | 第一个参数 | 能访问实例变量 | 能访问类变量 | 调用方式 |
|------|-----------|----------------|--------------|----------|
| **实例方法** | `self` | ✅ 可以 | ✅ 可以 | `obj.method()` |
| **类方法** `@classmethod` | `cls` (当前类本身) | ❌ 不能访问实例 | ✅ 可以访问类变量 | `ClassName.method()` |
| **静态方法** `@staticmethod` | 没有固定第一个参数 | ❌ 都不能访问 | ❌ 都不能访问 | `ClassName.method()` |

---

### 实际测试项目中的应用场景：

#### 1. 实例方法 → 最常用，90%都是这个
只要你需要访问/修改**实例自己的属性**，就用实例方法。

```python
# 例子：接口测试封装HttpClient
class ApiClient:
    def __init__(self, base_url, token=None):
        self.base_url = base_url
        self.token = token
    
    # 实例方法：需要访问self.base_url、self.token
    def get(self, path, params=None):
        return requests.get(f"{self.base_url}{path}", params=params, headers={"token": self.token})

# 调用：必须先创建实例，再调用
client = ApiClient("https://api.example.com", token="xxx")
resp = client.get("/user/1")  # ✅ 实例方法调用
```

#### 2. 类方法 `@classmethod` → 常用于「工厂方法」，或者操作类变量
常见场景：**不同方式创建对象**，或者统计整个类的信息。

```python
# 例子：接口测试，不同环境创建不同客户端
class ApiClient:
    # 类变量：统计总共创建了多少个客户端
    total_count = 0

    def __init__(self, base_url):
        self.base_url = base_url
        ApiClient.total_count += 1

    # 类方法：工厂方法，根据环境创建不同客户端
    @classmethod
    def from_env(cls, env):
        if env == "dev":
            return cls("https://dev-api.example.com")
        elif env == "prod":
            return cls("https://api.example.com")
    
    # 类方法：获取类变量，统计总数
    @classmethod
    def get_total_count(cls):
        return cls.total_count

# 使用：直接类名调用，不用创建实例
client = ApiClient.from_env("dev")
print(ApiClient.get_total_count()) # 输出：1

**测试中常见用途：**
- 工厂方法，根据不同配置/环境创建不同对象
- 操作类变量（统计整个类的实例个数）

#### 3. 静态方法 `@staticmethod` → 逻辑上和类相关，但**不访问类/实例变量**，就是放在类里组织一下代码

```python
# 例子：接口测试中，签名工具方法
class ApiClient:
    # 签名工具，不需要访问self/cls，只是把方法放到类里归类
    @staticmethod
    def generate_sign(params, secret_key):
        sorted_params = sorted(params.items())
        string = "&".join(f"{k}={v}" for k,v in sorted_params)
        string += secret_key
        return hashlib.md5(string.encode()).hexdigest()

# 使用：不用创建实例，直接类名调用
sign = ApiClient.generate_sign(params, "my-secret")
```

**测试中常见用途：**
- 工具类方法，和实例无关，放在类里只是为了代码组织

---

### 面试考点：

1. **基础区别**：能说清楚三个第一个参数是什么，能访问什么
2. **使用场景**：什么时候用哪个
   - 需要访问实例属性 → 实例方法
   - 需要操作类变量/工厂创建对象 → 类方法
   - 和实例/类都没关系，只是归类放这 → 静态方法
3. 面试官想看你**实际写代码有没有用过**，不是死记概念

---

## 二、Python数据结构常见题

### 1. 讲一下四个常用数据结构：list / tuple / dict / set 区别，什么是可变不可变？

先讲概念：
- **可变对象**：创建后可以修改（增加、删除、修改元素），修改后内存地址不变
- **不可变对象**：创建后不能修改，要修改只能新建对象，内存地址改变

四个常用数据结构对比：

| 数据结构 | 可变？ | 是否有序 | 底层存储 | 查找效率 | 主要使用场景（测试中举例） |
|---------|--------|----------|----------|----------|---------------------------|
| **list（列表）** | ✅ 可变 | ✅ 有序 | 动态数组 | O(n) 线性查找 | 元素顺序会变、需要增删改：<br>- 存储测试用例列表<br>- 存储接口返回的列表数据<br>- 动态添加错误日志 |
| **tuple（元组）** | ❌ 不可变 | ✅ 有序 | 固定长度数组 | O(n) 线性查找 | 元素固定不变，作为常量：<br>- 函数返回多个值（`return code, data`）<br>- 接口固定参数组合<br>- 作为字典的key（因为不可变） |
| **dict（字典）** | ✅ 可变 | 3.7+ 保证插入有序 | 哈希表 | O(1) 平均查找 | key-value键值对，通过key快速查找：<br>- 存储请求参数、响应JSON<br>- 存储测试环境配置<br>- 缓存接口返回数据 |
| **set（集合）** | ✅ 可变 | ❌ 无序 | 哈希表 | O(1) 查找 | 去重、判断存在：<br>- 接口返回数据去重<br>- 判断元素是否存在（`if id in seen_ids`）<br>- 求两个列表的交集差集 |

---

#### 可变 vs 不可变 代码举例：

```python
# list可变：修改后地址不变
lst = [1, 2, 3]
print(id(lst))  # 地址xxx
lst.append(4)    # 修改
print(id(lst))  # 还是xxx → 没变，因为list可变

# tuple不可变：不能修改，改只能新建
t = (1, 2, 3)
# t.append(4) → 报错！tuple没有append方法，不能改

# 字符串也是不可变
s = "hello"
print(id(s))
s += "world"  # 新建了一个字符串
print(id(s))  # 地址变了
```

#### 测试场景中怎么选择？

```python
# 1. 需要动态添加测试用例 → list
test_cases = []
test_cases.append(("admin", "123456", 0))
test_cases.append(("admin", "wrong", 1))

# 2. 固定不变得参数组合 → tuple
# 函数返回多个结果
def parse_response(resp):
    code = resp["code"]
    data = resp["data"]
    return code, data  # 返回tuple

# 3. 请求参数/配置 → dict
headers = {
    "Content-Type": "application/json",
    "token": "xxx"
}

# 4. 判断id是否已经处理过去重 → set
processed_ids = set()
for order in orders:
    if order["id"] not in processed_ids:
        process(order)
        processed_ids.add(order["id"])
```

**常见面试坑点：**
- tuple真的完全不可变吗？如果tuple里面套了list，list还是可变的：
```python
t = (1, 2, [3, 4])
t[2].append(5)  # ✅ 这是可以的！tuple不可变指的是元素引用不可变，引用里面的内容可变
print(t)  # (1, 2, [3, 4, 5])
```

### 2. 列表去重

方法一：转set再转list（顺序打乱）
```python
lst = [1, 2, 2, 3, 3, 3]
lst = list(set(lst))
```

方法二：保持顺序去重
```python
seen = set()
result = []
for item in lst:
    if item not in seen:
        seen.add(item)
        result.append(item)
```

### 3. 反转字符串

```python
s = "abcdef"
# 方法一：切片
s[::-1] → "fedcba"

# 方法二：转list反转
lst = list(s)
lst.reverse()
```

### 4. 字典遍历

```python
# 遍历key
for key in d:
    print(key)

# 遍历key-value
for key, value in d.items():
    print(key, value)
```

### 5. `append() 和 extend() 区别

```python
a = [1, 2]
b = [3, 4]

a.append(b) → [1, 2, [3, 4]] → append整个list作为一个元素

a.extend(b) → [1, 2, 3, 4] → extend把b中元素逐个加进来

---

## 三、Python函数和面向对象

### 1. 什么是面向对象？

面向对象是把**数据（属性）和操作数据（方法）封装在一起，以对象为单位的编程思想。

三大特性：
- **封装**：把数据和方法包装起来，对外暴露接口，隐藏内部实现
- **继承**：子类继承父类，可以继承父类属性方法，可以重写
- **多态**：不同对象对同一消息可以做出不同响应，通过方法重写实现

### 2. 什么是多态？

同一个方法名，不同对象实现不一样。比如父类是Animal，有run()方法，Dog跑，Bird飞，各自实现不同。

### 3. 什么是鸭子类型？

Python是**鸭子类型**：不关心对象是什么类型，只关心它有没有这个方法。只要有这个方法就能调用，不用继承同一个接口。

```python
class Dog:
    def run(self):
        print("dog run")

class Bird:
    def run(self):
        print("bird fly")

def do_run(obj):
    obj.run()  # 不管你是什么类，只要有run方法就能调用 → 这就是鸭子类型
```

### 4. 新式类和旧式类

- Python2：`class A: → 旧式类，`class A(object): → 新式类
- Python3：所有都是新式类，默认继承object
- 新式类：广度优先搜索继承，旧式类深度优先

### 5. 单例模式

一个类只能创建一个实例，整个应用中共享同一个实例。

**python常见写法：
```python
class Singleton:
    _instance = None
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

**使用场景（测试中）：
- 数据库连接池：整个测试框架只用一个连接
- 配置管理：配置全局唯一

### 6. 设计模式还有哪些？

- 常见：
  - 单例
  - 工厂模式
  - 工厂方法
  - 适配器模式
  - 观察者模式

软件测试常用：单例比较常考，其他知道概念就行

---

## 四、Python 高级特性（测试常用题）

### 1. 闭包是什么？

**内部函数引用外部函数变量，返回内部函数，就是闭包**，可以保存状态。

```python
def outer(x):
    def inner(y):
        return x + y
    return inner

add5 = outer(5)
print(add5(3)) → 8  # add5记住了x=5这个状态
```

**实际测试场景：** 闭包可以用来**配置化生成函数**，比如根据不同环境生成不同的URL前缀：

```python
def create_api_client(base_url):
    # 闭包记住了base_url
    def get(path):
        full_url = f"{base_url}{path}"
        return requests.get(full_url)
    return get

# 不同环境创建不同的get函数
dev_get = create_api_client("https://dev-api.example.com")
prod_get = create_api_client("https://api.example.com")

# 直接用，不用每次传base_url
dev_get("/user/1")
prod_get("/user/1")
```

**装饰器本质就是闭包**，所以闭包是Python中比较重要的概念。

### 2. 上下文管理器，`with` 关键字作用

自动管理资源，自动打开关闭，不用手动写`close`，不用怕忘记关闭。

```python
# 例子：打开文件
with open("test.txt", "r") as f:
    content = f.read()
# 退出with块自动关闭文件，不用你关
```

**自定义上下文管理器（测试场景）：** 比如管理数据库连接，自动连接自动关闭：

```python
class DBConnection:
    def __enter__(self):
        # 进入with块时执行：连接数据库
        self.conn = pymysql.connect(...)
        return self.conn
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        # 退出with块时执行：关闭连接
        self.conn.close()

# 使用，自动关闭连接
with DBConnection() as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM test")
```

**测试中常见场景：**
- 打开文件操作
- 数据库连接管理
- Selenium中 `with` 管理driver，自动quit
- 锁的获取和释放（`with lock:` 自动加锁解锁）

自动管理资源，自动打开关闭，不用手动写close，不用怕忘记关闭。

```python
# 例子：打开文件
with open("test.txt", "r") as f:
    content = f.read()
# 退出with块自动关闭文件，不用你关

### 3. 什么是闭包和装饰器关系？

装饰器本质就是闭包，接收函数作为参数，返回新函数，就是闭包应用。

### 4. 垃圾回收机制

Python垃圾回收主要三种：
1. **引用计数**：每个对象引用计数，计数为0回收（主要）
2. **标记清除**：处理循环引用
3. **分代回收**：根据对象存活时间分代，年轻代经常回收，年老代回收频率低

### 5. 常用标准模块介绍（测试脚本天天用）

**os模块 - 文件目录操作：**
```python
import os
os.listdir("/data/logs")       # 列出目录下文件
os.path.exists("app.log")      # 判断文件是否存在
os.path.getsize("app.log")     # 获取文件大小
os.remove("app.log")           # 删除文件
os.makedirs("/data/test/logs", exist_ok=True)  # 创建目录
```
测试脚本中经常用来找日志文件、清理过期文件、检查文件是否生成。

**json模块 - JSON序列化反序列化：**
```python
import json
# 字典转json字符串（存配置/发请求用）
json_str = json.dumps(data)
# json字符串转字典（解析接口响应）
data = json.loads(response.text)
# 读写json文件
with open("config.json", "r") as f:
    config = json.load(f)
```
接口测试天天用，解析响应、存配置。

**sys模块 - 路径和环境：**
```python
import sys
sys.path.append("/path/to/module")  # 添加模块搜索路径
sys.platform  # 判断操作系统 'win32'/'linux'
sys.exit(1)  # 退出脚本
```

**time/datetime模块 - 时间处理：**
```python
import time
time.sleep(3)  # 等待3秒（UI自动化常用）
time.time()    # 获取时间戳（统计接口耗时）

from datetime import datetime
datetime.now().strftime("%Y-%m-%d %H:%M:%S")  # 格式化时间
```
统计接口耗时、生成带时间戳的测试报告、等待操作都要用。

---

## 五、Python 在测试开发/自动化测试场景题

### 1. 你在测试中怎么用Python？

软件测试工程师用Python常见场景：
1. **接口自动化测试**：用requests库发请求，assert断言结果
2. **UI自动化测试**：Selenium/Appium + Python写用例
3. **写自动化框架**：Pytest+Allure做测试框架
4. **数据处理**：pandas处理测试数据，统计报表
5. **写工具脚本**：批量造测试数据，清理测试环境，批量执行sql

### 2. 接口自动化中，requests 怎么发请求？

```python
import requests

# GET
resp = requests.get("https://api.example.com/users", params={"page": 1})
print(resp.json())

# POST json
resp = requests.post("https://api.example.com/login", json={"username": "admin", "password": "123456"})
print(resp.status_code)
assert resp.json()["code"] == 0
```

### 3. 怎么处理接口依赖（上个接口返回给下个接口用？

把上个接口返回的token/数据存在全局变量或者存在yaml/json文件，下个接口提取出来用。

### 4. 怎么断言接口结果？

```python
# 状态码断言
assert resp.status_code == 200
# 字段存在断言
assert "data" in resp.json()
# 数据值断言
assert resp.json()["code"] == 0
# 长度断言
assert len(resp.json()["list"]) == 10
```

### 5. Pytest测试框架你用过吗？说说常用特性

- **fixture**：前置后置，复用测试数据，依赖注入
- **参数化**：`@pytest.mark.parametrize` 同一个用例多组参数
- **标记**：`@pytest.mark.smoke` 分组执行
- **conftest.py**：全局fixture，多个文件共享

### 6. 怎么用Python处理异常捕获，测试中什么时候用？

```python
try:
    # 可能出错的代码
except Exception as e:
    print(f"出错了: {e}")
finally:
    # 无论如何都执行，关闭资源
```

**测试中实际使用场景：**

1. **断言失败后继续执行后面用例**：
```python
for case in test_cases:
    try:
        # 执行测试用例
        assert case["expected"] == actual
        print("用例通过")
    except AssertionError as e:
        # 捕获断言失败，记录日志，继续跑下一个用例，不中断整个测试
        logging.error(f"用例失败: {e}")
```

2. **网络不稳定时重试，捕获异常重试**：
```python
# 接口请求偶发网络错误，捕获异常重试几次
for i in range(3):
    try:
        resp = requests.get(url, timeout=10)
        break
    except requests.exceptions.Timeout:
        logging.warning("请求超时，重试...")
```

3. **资源清理：finally保证一定关闭**：
```python
conn = pymysql.connect(...)
try:
    # 操作数据库
finally:
    # 无论成功失败，一定关闭连接
    conn.close()
```

**小结：** 测试脚本中，你不希望一个用例失败导致整个测试脚本停止，所以要捕获异常继续执行。

### 7. 怎么写一个日志？

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    filename='test.log'
)

logging.info("接口请求成功")
logging.error("请求失败", exc_info=True)
```

### 8. 怎么连接MySQL

```python
import pymysql

conn = pymysql.connect(host='localhost', user='root', password='xxx', database='test')
cursor = conn.cursor()
cursor.execute("SELECT * FROM user")
result = cursor.fetchall()
conn.close()
```

### 9. 你用过哪些Python自动化框架？

- **接口自动化**：requests + pytest + allure，数据驱动用YAML存储用例
- **UI自动化**：Playwright/Selenium + pytest + allure
- **APP自动化**：Appium + pytest
- **数据驱动**：pytest + yaml/json，用例和代码分离

### 10. 举例说说接口自动化数据驱动怎么设计（requests + pytest + allure + YAML）

**数据驱动思想：测试用例数据和代码分离，新增用例只需要加YAML，不用改Python代码**

项目结构：
```
api_test/
├── config/
│   └── config.yaml      # 环境配置
├── data/
│   └── login_cases.yaml # 测试用例数据
├── utils/
│   └── api_client.py    # 封装API请求
├── test_cases/
│   └── test_login.py    # 测试代码
└── conftest.py
```

**YAML用例数据（data/login_cases.yaml）：**
```yaml
- case_name: 登录成功
  username: admin
  password: 123456
  expected_code: 0
  expected_msg: 成功

- case_name: 密码错误
  username: admin
  password: wrong
  expected_code: 1
  expected_msg: 密码错误

- case_name: 用户名为空
  username: ""
  password: 123456
  expected_code: 2
  expected_msg: 用户名为空
```

**测试代码（test_cases/test_login.py）：**
```python
import pytest
import requests
import yaml
import allure

# 加载用例数据
with open("data/login_cases.yaml", "r", encoding="utf-8") as f:
    cases = yaml.safe_load(f)

@allure.feature("登录模块")
@pytest.mark.parametrize("case", cases)
def test_login(base_url, case):
    with allure.step(case["case_name"]):
        # 发送请求
        resp = requests.post(
            f"{base_url}/api/login",
            json={
                "username": case["username"],
                "password": case["password"]
            }
        )
        # 断言
        assert resp.json()["code"] == case["expected_code"]
        assert case["expected_msg"] in resp.json()["msg"]
```

**运行生成报告：**
```bash
pytest test_cases/ -v --alluredir=./reports
allure serve ./reports
```

**优点：**
- 新增用例只需要在YAML加，不用改Python代码
- 用例清晰易维护，测试人员可以直接编辑YAML
- Allure报告美观，步骤清晰，方便定位问题

### 10.1 requests 库常用方法（接口测试天天用）

```python
import requests

# 1. GET请求 - 带参数
resp = requests.get("https://api.example.com/users", params={"page": 1, "size": 10})

# 2. POST请求 - JSON格式
resp = requests.post(
    "https://api.example.com/login",
    json={"username": "admin", "password": "123456"}
)

# 3. POST请求 - form表单格式
resp = requests.post(
    "https://api.example.com/upload",
    data={"name": "test"},
    files={"file": open("test.txt", "rb")}
)

# 4. 带请求头
headers = {
    "Authorization": "Bearer {token}",
    "Content-Type": "application/json"
}
resp = requests.get("https://api.example.com/user", headers=headers)

# 5. 获取响应结果
resp.status_code  # 状态码
resp.text         # 响应文本
resp.json()       # 解析JSON转字典
resp.content      # 二进制内容（下载文件用）
resp.headers      # 响应头

# 6. 超时设置（防止卡住）
resp = requests.get(url, timeout=10)  # 10秒超时

# 7. 会话保持（自动带cookie）
session = requests.Session()
session.post("/login", json=data)  # 登录后cookie自动保存
session.get("/userinfo")  # 后续请求自动带cookie
```

**测试场景：**
- GET/POST用得最多，接口测试主要就是发这两种请求
- Session用来保持登录状态，同一个会话多次请求
- 超时一定要设，不设网络不好会卡死

---

### 10.2 pytest 常用功能和方法

pytest是Python最流行的测试框架，核心特性：

**1. 用例发现规则：**
- 文件名以 `test_*.py` 开头
- 函数名以 `test_` 开头
- 类名以 `Test` 开头

**2. 参数化测试：**
```python
@pytest.mark.parametrize("username,password,expected", [
    ("admin", "123456", 0),
    ("admin", "wrong", 1),
])
def test_login(username, password, expected):
    ...
```

**3. fixture 前置后置：**
```python
import pytest

# scope: function(每个用例都执行)/class/module/session(整个会话一次)
@pytest.fixture(scope="session")
def login_token():
    # 前置：登录获取token
    resp = requests.post("/login", json={"username": "admin", "password": "123456"})
    token = resp.json()["token"]
    yield token  # 返回给用例，yield后面是后置
    # 后置：清理数据
    ...

# 用例直接注入使用
def test_get_user(login_token):
    resp = requests.get("/user", headers={"Authorization": login_token})
    assert resp.status_code == 200
```

**4. 标记分组执行：**
```python
@pytest.mark.smoke  # 冒烟测试标记
@pytest.mark.api    # 接口测试标记
@pytest.mark.ui     # UI测试标记
def test_xxx():
    ...
```
执行：`pytest -m smoke` 只跑冒烟用例

**5. 跳过用例：**
```python
@pytest.mark.skip("功能还没开发完")
@pytest.mark.skipif(sys.platform == "win32", reason="只在Linux跑")
def test_xxx():
    ...
```

**6. conftest.py：**
- 全局fixture，多个测试文件共享
- 不用import，pytest自动发现
- 一般放项目根目录

---

### 10.3 allure 报告常用特性

allure生成美观的测试报告：

```python
import allure

# 1. 功能模块标记
@allure.feature("登录模块")
@allure.story("用户登录")

# 2. 添加测试步骤
with allure.step("输入用户名密码"):
    ...

with allure.step("点击登录按钮"):
    ...

# 3. 添加截图（UI自动化失败时截图）
allure.attach(page.screenshot(), name="失败截图", attachment_type=allure.attachment_type.PNG)

# 4. 添加请求响应日志
allure.attach(json.dumps(resp.json(), indent=2), name="响应数据", attachment_type=allure.attachment_type.JSON)
```

生成报告：
```bash
pytest --alluredir=./reports  # 生成数据
allure serve ./reports       # 本地打开浏览器查看
allure generate ./reports -o ./html  # 生成静态HTML
```

### 11. UI自动化数据驱动举例（Playwright + pytest + allure）

Playwright是微软新一代UI自动化工具，比Selenium更快，更稳定，自带自动等待。

**项目结构：**
```
ui_test/
├── data/
│   └── search_cases.yaml  # 测试数据
├── pages/
│   └── search_page.py     # 页面对象
├── test_cases/
│   └── test_search.py     # 测试用例
└── conftest.py
```

**页面对象封装（pages/search_page.py）：**
```python
from playwright.sync_api import Page

class SearchPage:
    def __init__(self, page: Page):
        self.page = page
        self.search_input = "#search-input"
        self.search_button = "#search-btn"
        self.result_item = ".result-item"
    
    def goto(self):
        self.page.goto("/search")
    
    def search(self, keyword):
        self.page.fill(self.search_input, keyword)
        self.page.click(self.search_button)
    
    def get_result_count(self):
        return self.page.locator(self.result_item).count()
```

**测试代码（test_cases/test_search.py）：**
```python
import pytest
import yaml
import allure
from pages.search_page import SearchPage

with open("data/search_cases.yaml", "r", encoding="utf-8") as f:
    cases = yaml.safe_load(f)

@allure.feature("搜索功能")
@pytest.mark.ui
@pytest.mark.parametrize("case", cases)
def test_search(page, case):
    with allure.step(f"测试搜索: {case['keyword']}"):
        search_page = SearchPage(page)
        search_page.goto()
        search_page.search(case["keyword"])
        assert search_page.get_result_count() >= case["min_result"]
```

**conftest.py 配置Playwright：**
```python
import pytest
from playwright.sync_api import sync_playwright

@pytest.fixture(scope="session")
def browser():
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=False)
        yield browser
        browser.close()

@pytest.fixture(scope="function")
def page(browser):
    context = browser.new_context()
    page = context.new_page()
    yield page
    context.close()
```

**运行：**
```bash
# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install pytest playwright allure-pytest
playwright install chromium

pytest -m ui -v --alluredir=./reports
```

**Playwright优点：**
- 自动等待元素出现，不用写大量`time.sleep`
- 内置浏览器，不用额外配置驱动
- 支持多浏览器：Chromium、Firefox、WebKit
- 速度快，稳定性比Selenium好

### 11.1 Playwright 常用操作方法

```python
from playwright.sync_api import Page

# 常用定位方式（CSS选择器最常用）
page.locator("#username").fill("admin")      # id定位
page.locator(".search-btn").click()          # class定位
page.locator("//button[text()='登录']").click()  # xpath定位
page.get_by_text("登录").click()             # 按文本定位
page.get_by_role("button", name="登录").click()  # 按角色定位

# 常用操作
page.goto("https://example.com")            # 跳转URL
page.fill("#username", "admin")             # 输入文本
page.click("#login-btn")                    # 点击
page.check("#agree-checkbox")               # 勾选复选框
page.select_option("#city", "beijing")      # 下拉框选择

# 获取信息
text = page.locator(".result").inner_text() # 获取文本
is_visible = page.locator(".success").is_visible()  # 判断是否可见
count = page.locator(".item").count()       # 统计元素个数

# 截图（失败时截图给allure）
screenshot = page.screenshot()              # 截图

# 弹窗处理
page.on("dialog", lambda dialog: dialog.accept())  # 接受弹窗

# 等待
page.wait_for_selector(".result", timeout=5000)  # 等待元素出现
# Playwright自动等待，click/fill之前自动等待，大部分时候不用手动等
```

**对比Selenium：**
- Playwright自动等待，不用写很多`time.sleep`和`WebDriverWait`
- API更简洁，定位方式更丰富
- 网络拦截、模拟设备、多标签页更容易
- 速度更快，稳定性更高

### 11.2 完整实际业务例子：测试管理员登录

**场景：打开登录页 → 输入用户名密码 → 点击登录 → 断言登录成功跳转到首页**

```python
from playwright.sync_api import Page
import allure

class LoginPage:
    """登录页面元素定位和操作封装"""
    
    def __init__(self, page: Page):
        self.page = page
        
        # 定位器定义（写在这里统一管理）
        self.url = "https://test.example.com/admin/login"
        self.username_input = "#username"         # 用户名输入框 id
        self.password_input = "#password"         # 密码输入框 id
        self.login_button = "button[type='submit']"  # 登录按钮 CSS
        self.error_msg = ".ant-message-error"     # 错误提示信息
        self.user_avatar = ".user-avatar"         # 首页用户头像，登录成功后可见
    
    def goto(self):
        """打开登录页面"""
        self.page.goto(self.url)
    
    def login(self, username: str, password: str):
        """执行登录操作"""
        with allure.step(f"输入用户名: {username}"):
            # 定位用户名输入框，输入内容
            self.page.locator(self.username_input).fill(username)
        
        with allure.step("输入密码"):
            # 定位密码输入框，输入内容
            self.page.locator(self.password_input).fill(password)
        
        with allure.step("点击登录按钮"):
            # 定位登录按钮，点击
            self.page.locator(self.login_button).click()
    
    def get_error_message(self):
        """获取错误提示"""
        # 定位错误元素，获取文本
        return self.page.locator(self.error_msg).inner_text()
    
    def is_logged_in(self):
        """判断是否登录成功：看用户头像是否可见"""
        # 定位头像元素，判断是否可见
        return self.page.locator(self.user_avatar).is_visible(timeout=5000)
```

**测试用例：**
```python
import pytest
import allure

@allure.feature("登录模块")
@pytest.mark.parametrize("username,password,expected_success,expected_msg", [
    ("admin", "123456", True, ""),        # 正确账号密码，应该登录成功
    ("admin", "wrong", False, "密码错误"),  # 密码错误，应该提示错误
    ("", "123456", False, "用户名不能为空"), # 用户名为空，提示错误
])
def test_login(page, username, password, expected_success, expected_msg):
    login_page = LoginPage(page)
    login_page.goto()
    login_page.login(username, password)
    
    if expected_success:
        # 断言登录成功：头像可见
        assert login_page.is_logged_in() is True
    else:
        # 断言登录失败：错误提示正确
        assert expected_msg in login_page.get_error_message()
```

**流程说明：**
1. 先在页面对象中**定义元素定位器**（把所有定位统一管理，维护方便）
2. 封装登录操作，每一步都先`locator()`定位元素，再调用`fill()`/`click()`操作
3. Playwright会**自动等待元素可操作**，不用你手动写等待
4. 数据驱动，不同账号密码组合测试不同场景

常见定位方式选择：
- 有id用id：`#username` 最快最简单
- id动态生成用CSS：`button[type='submit']` 或者 `.login-btn`
- 需要按文本找用：`page.get_by_text("登录")`
- 复杂定位用xpath：`//div[contains(text(),'用户名')]/following::input`

---

## 六、常见面试问答总结

### Q1：说说你对Python的理解

Python是一门解释型、面向对象、动态类型的高级语言，语法简洁，生态丰富，非常适合快速开发、写脚本、做自动化测试。

### Q2：你用过Python做什么？

我做自动化测试，接口自动化、UI自动化，写测试工具脚本，造测试数据，统计测试结果，生成测试报告。

### Q3：说说你对Python多线程和多进程区别？测试中什么场景用？

- **多线程**：在一个进程内，共享内存，GIL同一时刻只能一个线程运行，适合**IO密集型**（等待IO多，比如网络请求）
- **多进程**：每个进程独立内存空间，不共享GIL，可以多核并行，适合**CPU密集型**（计算多）

**测试中实际使用场景：**
```python
# 场景：并发压测接口，10个线程同时请求接口，看性能
import threading
import requests

def test_concurrent_request():
    resp = requests.get("https://api.example.com/health")
    print(resp.status_code)

# 启动10个线程并发请求
threads = []
for _ in range(10):
    t = threading.Thread(target=test_concurrent_request)
    t.start()
    threads.append(t)

# 等待所有线程完成
for t in threads:
    t.join()
```
- **接口并发压测**：用多线程，因为主要是等待网络IO，多线程可以同时发多个请求
- **批量处理文件**：多个线程同时处理多个日志文件，提升效率
- **CPU密集型计算**：比如大量数据加密、计算统计值，用多进程绕开GIL，利用多核

### Q4：什么是线程安全？

多个线程同时修改同一个共享变量，可能导致结果不一致，需要加锁保证安全。

### Q5：说说你的Python怎么实现线程安全？

用 `threading.Lock()` 加锁，修改前获取锁，修改完释放锁。

### Q6：Python 多进程怎么共享数据？

多进程不共享内存，可以用 `multiprocessing.Queue` 或者 `Manager` 共享。

### Q7：为什么要学Python做测试？

- 语法简单好上手，生态丰富，有很多自动化测试库（requests/pytest/Selenium），开发效率高，能快速写出自动化脚本，适合测试工作。

---

## 七、总结（软件测试对Python要求）

软件测试工程师不需要像开发那样深入，重点掌握：

1. **基础语法**：变量、循环、条件、函数、面向对象
2. **常用数据结构**：list、dict、tuple、set 会用会操作
3. **异常处理、文件操作、模块导入
4. **第三方库：requests、pymysql、pandas
5. **测试框架：pytest**
5. **能看懂代码，能写简单的自动化脚本，定位问题

不用精通，但要能写出能用

面试考的就是这些，深入的算法和设计模式知道概念就行。
