# Python 常用方法速查（含返回值）

---

## 一、字符串方法

| 方法 | 示例 | 返回值 | 说明 |
|------|------|--------|------|
| `split(sep)` | `"a,b,c".split(",")` | `['a', 'b', 'c']` | 按分隔符拆成**列表** |
| `join(iterable)` | `",".join(["a","b"])` | `'a,b'` | 列表拼成**字符串**（调用在分隔符上） |
| `replace(old, new)` | `"hello".replace("l","x")` | `'hexxo'` | 替换所有，返回新**字符串** |
| `strip()` | `" abc ".strip()` | `'abc'` | 去首尾空格/换行，返回新**字符串** |
| `find(sub)` | `"hello".find("l")` | `2` | 返回**索引**，找不到返回 `-1` |
| `index(sub)` | `"hello".index("l")` | `2` | 返回**索引**，找不到报 `ValueError` |
| `startswith(prefix)` | `"hello".startswith("he")` | `True` | 判断开头，返回**布尔** |
| `endswith(suffix)` | `"hello".endswith("lo")` | `True` | 判断结尾，返回**布尔** |
| `count(sub)` | `"hello".count("l")` | `2` | 统计出现次数，返回**整数** |
| `upper()` | `"hello".upper()` | `'HELLO'` | 全大写，返回新**字符串** |
| `lower()` | `"HELLO".lower()` | `'hello'` | 全小写，返回新**字符串** |
| `isdigit()` | `"123".isdigit()` | `True` | 是否全是数字，返回**布尔** |
| `isalpha()` | `"abc".isalpha()` | `True` | 是否全是字母，返回**布尔** |
| `format()` | `"x={}".format(1)` | `'x=1'` | 格式化，返回新**字符串** |

---

## 二、列表方法（都是对列表的操作）

```python
lst = [1, 3, 2]    # 初始值
```

| 方法 | 示例 | 返回值 | 操作后的 lst |
|------|------|--------|-------------|
| `append(obj)` | `lst.append(4)` | **None** | `[1, 3, 2, 4]` |
| `insert(idx, obj)` | `lst.insert(1, "a")` | **None** | `[1, 'a', 3, 2, 4]` |
| `extend(iterable)` | `lst.extend([5,6])` | **None** | `[1, 'a', 3, 2, 4, 5, 6]` |
| `pop()` | `lst.pop()` | `6` | `[1, 'a', 3, 2, 4, 5]` |
| `pop(idx)` | `lst.pop(0)` | `1` | `['a', 3, 2, 4, 5]` |
| `remove(val)` | `lst.remove('a')` | **None** | `[3, 2, 4, 5]` |
| `clear()` | `lst.clear()` | **None** | `[]` |
| `index(val)` | `lst.index(5)` | `3` | 不变（纯查询） |
| `count(val)` | `lst.count(5)` | `1` | 不变（纯查询） |
| `sort()` | `lst.sort()` | **None** | `[2, 3, 4, 5]` 原地升序 |
| `sort(reverse=True)` | `lst.sort(reverse=True)` | **None** | `[5, 4, 3, 2]` 原地降序 |
| `reverse()` | `lst.reverse()` | **None** | `[2, 3, 4, 5]` 倒过来 |
| `copy()` | `lst.copy()` | `[2, 3, 4, 5]` | 原列表不变，返回**新列表**（浅拷贝） |

**内置函数（不修改原列表）**：

| 函数 | 示例 | 返回值 | 原列表 |
|------|------|--------|--------|
| `sorted(iterable)` | `sorted([3,1,2])` | `[1, 2, 3]` | 不变 |
| `len(lst)` | `len([1,2,3])` | `3` | 不变 |
| `max(lst)` | `max([1,3,2])` | `3` | 不变 |
| `min(lst)` | `min([1,3,2])` | `1` | 不变 |
| `sum(lst)` | `sum([1,3,2])` | `6` | 不变 |
| `list(iterable)` | `list("abc")` | `['a','b','c']` | 字符串转列表 |

---

## 三、字典方法

```python
d = {"name": "test", "age": 25}
```

| 方法 | 示例 | 返回值 | 说明 |
|------|------|--------|------|
| `get(key, default)` | `d.get("name")` | `'test'` | key 存在返回值，不存在返回 None |
| `get(key, default)` | `d.get("xx", "默认")` | `'默认'` | 不存在返回指定的默认值 |
| `keys()` | `d.keys()` | `dict_keys(['name','age'])` | 所有 key，可迭代 |
| `values()` | `d.values()` | `dict_values(['test',25])` | 所有 value，可迭代 |
| `items()` | `d.items()` | `dict_items([('name','test'),('age',25)])` | 键值对元组，可迭代 |
| `update(dict)` | `d.update({"age":26})` | **None** | d 变成 `{'name':'test','age':26}` |
| `pop(key)` | `d.pop("age")` | `25` | 删除并返回值，key 不存在报 KeyError |
| `popitem()` | `d.popitem()` | `('age', 25)` | 删除并返回最后一个键值对（元组） |
| `setdefault(key,val)` | `d.setdefault("x", 1)` | `1` | key 存在返回值，不存在设默认值并返回 |

**遍历字典**：

```python
for k, v in d.items():
    print(k, v)            # name test \n age 25
```

---

## 四、集合方法

```python
s = {1, 2, 3}
```

| 方法 | 示例 | 返回值 | 操作后 s |
|------|------|--------|---------|
| `add(elem)` | `s.add(4)` | **None** | `{1, 2, 3, 4}` |
| `remove(elem)` | `s.remove(2)` | **None** | `{1, 3, 4}`（不存在报 KeyError） |
| `discard(elem)` | `s.discard(2)` | **None** | `{1, 3, 4}`（不存在不报错） |
| `pop()` | `{"a","b"}.pop()` | `'a'`（随机） | 随机删一个并返回 |
| `clear()` | `s.clear()` | **None** | `set()` 空集合 |

**集合运算**：

| 表达式 | 返回值 | 说明 |
|--------|--------|------|
| `{1,2} & {2,3}` | `{2}` | 交集 |
| `{1,2} \| {3,4}` | `{1, 2, 3, 4}` | 并集 |
| `{1,2} - {2,3}` | `{1}` | 差集（前面有后面没有的） |
| `{1,2} ^ {2,3}` | `{1, 3}` | 对称差集（只在一边的） |
| `1 in {1,2}` | `True` | 成员判断 |

---

## 五、文件操作

```python
# 读
with open("file.txt", "r", encoding="utf-8") as f:
    f.read()           # 返回字符串：整个文件内容
    f.readline()       # 返回字符串：一行（含换行符）
    f.readlines()      # 返回列表：['第1行\n', '第2行\n', ...]

# 写
with open("file.txt", "w", encoding="utf-8") as f:
    f.write("hello")   # 返回写入的字符数：5

# 追加
with open("file.txt", "a", encoding="utf-8") as f:
    f.write("追加内容")

# 模式说明
# "r"  只读，文件不存在报错
# "w"  只写，文件不存在创建，存在则清空
# "a"  追加，文件不存在创建
# "r+" 读写
```

---

## 六、JSON 操作（接口测试高频）

```python
import json

# 字符串 → 字典/列表
json.loads('{"name":"test"}')    # 返回 {'name': 'test'}

# 字典/列表 → 字符串
json.dumps({"name":"test"})      # 返回 '{"name":"test"}'
json.dumps({"name":"test"}, ensure_ascii=False)  # 返回 '{"name":"test"}'（中文不转义）

# 读 JSON 文件
with open("data.json", "r") as f:
    json.load(f)                 # 返回字典/列表

# 写 JSON 文件
with open("data.json", "w") as f:
    json.dump({"name":"test"}, f)
```

---

## 七、YAML 操作（接口自动化/配置管理高频）

```bash
pip install pyyaml
```

```python
import yaml

# 字符串 → 字典/列表
yaml.safe_load("name: test\nage: 25")
# 返回 {'name': 'test', 'age': 25}

# 字典/列表 → 字符串
yaml.dump({"name": "test", "age": 25})
# 返回 'age: 25\nname: test\n'
yaml.dump({"name": "测试"}, allow_unicode=True)
# 返回 'name: 测试\n'（中文不转义）

# 读 YAML 文件
with open("config.yaml", "r", encoding="utf-8") as f:
    data = yaml.safe_load(f)        # 返回字典/列表

# 写 YAML 文件
with open("config.yaml", "w", encoding="utf-8") as f:
    yaml.dump({"name": "test"}, f, allow_unicode=True)
```

**测试中 YAML 常用场景**：

```yaml
# config.yaml — 测试配置统一管理
base_url: http://api.example.com
timeout: 30
account:
  username: admin
  password: "123456"
db:
  host: 10.0.0.1
  port: 3306
```

```python
# conftest.py 里读取配置
import yaml

with open("config.yaml", "r", encoding="utf-8") as f:
    config = yaml.safe_load(f)

base_url = config["base_url"]           # 'http://api.example.com'
username = config["account"]["username"] # 'admin'
db_host  = config["db"]["host"]          # '10.0.0.1'
```

**一个 YAML 存多条接口测试用例（实际项目最常用）**：

```yaml
# ====================================================================
# 分类                数量  参数/说明
# ====================================================================
# 基础场景              1个  正常批量发布
# 参数枚举测试          5个  不同状态：草稿0/待发布1/发布2/失败3/取消4
# 边界测试              3个  空发布列表、空素材列表、空账户列表
# 异常测试              4个  缺少必填字段
# ====================================================================
# 总计                13个 测试用例
# ====================================================================

# 基础场景
- name: "批量发布-单个素材发布到单个账户"
  method: POST
  url: /api/publish/article/batch
  headers:
    Content-Type: application/json
    e-project-id: "${projectId}"
  json:
    pubishDTOList:
      - projectId: $projectId
        accountIds: [1]
        source:
          - sourceId: 1
            sourceTitle: "test image"
            sourceType: 0
            mediaUrl: "https://example.com/image.jpg"
        status: 1
        description: "test batch publish"
  expected_status: 200
  expected_response:
    code: 0
  max_retries: 3

# 参数枚举 - 不同发布状态
- name: "批量发布-草稿状态"
  method: POST
  url: /api/publish/article/batch
  headers:
    Content-Type: application/json
    e-project-id: "${projectId}"
  json:
    pubishDTOList:
      - projectId: $projectId
        accountIds: [1]
        source:
          - sourceId: 1
            sourceTitle: "draft test"
            sourceType: 0
            mediaUrl: "https://example.com/image.jpg"
        status: 0
  expected_status: 200
  expected_response:
    code: 0
  max_retries: 3

# 边界测试 - 空列表
- name: "批量发布-空发布列表"
  method: POST
  url: /api/publish/article/batch
  headers:
    Content-Type: application/json
    e-project-id: "${projectId}"
  json:
    pubishDTOList: []
  expected_status: 200
  expected_response:
    code: 400
  max_retries: 1

# 异常测试 - 缺少必填字段
- name: "批量发布-缺少pubishDTOList"
  method: POST
  url: /api/publish/article/batch
  headers:
    Content-Type: application/json
    e-project-id: "${projectId}"
  json: {}
  expected_status: 200
  expected_response:
    code: 400
  max_retries: 1
```

```python
# 读 YAML，遍历执行，断言结果
import yaml
import requests
import re

with open("test_batch_publish.yaml", "r", encoding="utf-8") as f:
    cases = yaml.safe_load(f)          # 返回列表（因为顶层是 - xxx）

# 变量替换：${projectId} → 实际值
vars_dict = {"projectId": "123"}

for case in cases:
    name     = case["name"]
    method   = case["method"]          # 'POST' / 'GET'
    url      = "https://star.digiplus-intl.com" + case["url"]
    headers  = case.get("headers", {})
    body     = case.get("json", case.get("data", {}))
    expected_code = case["expected_response"]["code"]
    max_retries   = case.get("max_retries", 1)

    # 变量替换：把 headers 和 body 里的 ${xxx} / $xxx 替换为实际值
    headers_str = yaml.dump(headers)
    body_str    = yaml.dump(body)
    for k, v in vars_dict.items():
        headers_str = headers_str.replace("${" + k + "}", v).replace("$" + k, v)
        body_str    = body_str.replace("${" + k + "}", v).replace("$" + k, v)
    headers = yaml.safe_load(headers_str)
    body    = yaml.safe_load(body_str)

    # 根据 method 发请求
    if method == "POST":
        resp = requests.post(url, json=body, headers=headers)
    elif method == "GET":
        resp = requests.get(url, params=body, headers=headers)

    # 断言
    assert resp.status_code == case["expected_status"], \
        f"{name} HTTP状态码断言失败: {resp.status_code}"
    assert resp.json()["code"] == expected_code, \
        f"{name} 业务code断言失败: {resp.json()['code']}"
    print(f"✓ {name} 通过")
```

**格式要点（你们项目的实际规范）**：

| 要点 | 说明 |
|------|------|
| 顶层是**列表** | `- name: xxx`，不是嵌套在 dict 下 |
| `json` 不是 `data` | 请求体用 `json` 字段 |
| 变量 `${projectId}` / `$projectId` | 两种写法都能用，运行时替换为实际值 |
| `expected_status` | HTTP 状态码断言，和业务 `code` 分开 |
| `expected_response.code` | 业务 code 断言 |
| `max_retries` | 异常场景设 `1`，正常场景设 `3` |
| 注释 `#` | 写分类说明、统计数量，方便人看 |

| 方法 | 示例 | 返回值 | 说明 |
|------|------|--------|------|
| `yaml.safe_load(str)` | `yaml.safe_load("a: 1")` | `{'a': 1}` | 字符串→字典（安全模式） |
| `yaml.load(str)` | `yaml.load("a: 1", Loader=...)` | `{'a': 1}` | 全功能模式，必须指定 Loader |
| `yaml.safe_load_all(str)` | `yaml.safe_load_all("---\na:1\n---\nb:2")` | 生成器 | 多文档 YAML |
| `yaml.dump(obj)` | `yaml.dump({"a":1})` | `'a: 1\n'` | 字典→字符串 |
| `yaml.dump_all([obj1, obj2])` | 同上 | 字符串 | 多文档输出 |
| `yaml.safe_dump(obj)` | `yaml.safe_dump({"a":1})` | `'a: 1\n'` | 安全写，推荐用这个 |

**yaml.load 和 yaml.safe_load 的区别**：
- `safe_load`：只解析基本类型，**安全**，推荐用
- `load`：可以解析自定义 Python 对象，**不安全**（可被注入攻击），必须指定 Loader

---

## 八、内置函数

| 函数 | 示例 | 返回值 | 说明 |
|------|------|--------|------|
| `type(obj)` | `type("abc")` | `<class 'str'>` | 看类型 |
| `isinstance(obj, t)` | `isinstance("a", str)` | `True` | 判断类型，考虑继承 |
| `len(obj)` | `len([1,2,3])` | `3` | 长度/元素个数 |
| `range(stop)` | `list(range(3))` | `[0, 1, 2]` | 生成整数序列 |
| `range(s, e, step)` | `list(range(1,5,2))` | `[1, 3]` | 含头不含尾 |
| `enumerate(lst)` | `list(enumerate(["a","b"]))` | `[(0,'a'),(1,'b')]` | 带索引的元组 |
| `zip(a, b)` | `list(zip([1,2],["a","b"]))` | `[(1,'a'),(2,'b')]` | 并行配对 |
| `map(fn, lst)` | `list(map(str, [1,2]))` | `['1', '2']` | 每个元素执行函数 |
| `filter(fn, lst)` | `list(filter(lambda x:x>2, [1,2,3]))` | `[3]` | 过滤，保留返回 True 的 |
| `any(iterable)` | `any([False,True,False])` | `True` | 有一个 True 就 True |
| `all(iterable)` | `all([True,True,False])` | `False` | 全 True 才 True |
| `int(s)` | `int("123")` | `123` | 字符串转整数 |
| `str(obj)` | `str(123)` | `'123'` | 转字符串 |
| `bool(obj)` | `bool("")` | `False` | 空字符串/0/None/空列表 为 False |
| `print()` | `print("a","b",sep="-")` | **None** | `a-b`（sep 默认为空格） |

---

## 九、断言（pytest 高频）

```python
# 等值判断
assert 1 + 1 == 2                              # True 通过
assert "hello" == "hello"                       # True 通过

# 接口测试断言
resp.status_code                                # 200
assert resp.status_code == 200                  # HTTP 状态码断言
assert resp.json()["code"] == 0                 # 业务状态码断言
assert resp.json()["data"] is not None          # data 不为空
assert isinstance(resp.json()["data"], dict)    # data 是字典类型
assert len(resp.json()["data"]["list"]) > 0     # 列表不为空
assert "成功" in resp.text                      # 响应包含关键字
```

---

## 十、实际写脚本的例子

```python
import requests

resp = requests.post(
    "http://api.example.com/login",
    json={"username": "admin", "password": "123456"}
)

# 关键方法返回值
resp.status_code       # 200（整数）
resp.text              # 响应体字符串
resp.json()            # 响应体转字典：{"code":0,"data":{"token":"xxx"}}
resp.headers           # 响应头字典
resp.elapsed.total_seconds()  # 响应时间（秒）

# 提取 token
token = resp.json().get("data", {}).get("token")  # "xxx"，取不到返回 None
```

---

## 记忆技巧

| 记法 | 内容 |
|------|------|
| 原地操作返回 None | `append`、`insert`、`remove`、`sort`、`reverse`、`update`、`add` |
| 有返回值 | `pop`（弹出的值）、`get`（查询的值）、`sorted`（新列表）、`find`（索引/-1）、`index`（索引/报错） |
| `find` vs `index` | find 找不到返 -1，index 找不到报错 |
| `discard` vs `remove` | discard 不报错，remove 报错（集合） |
| `get` 安全取值 | 字典用 get 防 KeyError，设默认值 |
| 字符串全返回新对象 | 所有字符串方法都返回新字符串，原字符串不变 |
