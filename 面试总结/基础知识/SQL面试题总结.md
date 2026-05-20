# SQL/MySQL 软件测试面试总结

## 一、基础概念

### 1. 什么是SQL？
SQL（Structured Query Language）结构化查询语言，是用于访问和操作数据库的标准语言。软件测试中主要用SQL做**数据验证**——接口返回的数据是否正确写入数据库，数据修改后数据库是否更新正确。

### 2. 数据库三大范式
核心思想：**一张表只干一件事，数据不要冗余存储**

- **第一范式（1NF）**：每一列都是不可分割的原子数据项，每个格子只放一个信息，不能把多个信息揉进一个字段
- **第二范式（2NF）**：一张表只描述一件事情，消除数据冗余，不要把订单和商品信息混在一张表里
- **第三范式（3NF）**：表中不包含已在其他表中存在的非主键信息，避免冗余，需要的时候联表查询即可

**举个通俗例子：**
- 1NF：联系方式要拆成电话和邮箱两个字段，不能揉一块
- 2NF：订单信息放订单表，商品信息放商品表，别混一张表
- 3NF：订单表只存用户ID就行，别再存一遍用户名（用户名已经在用户表里了）

**测试面试不需要深入设计，能说清楚这三点就够了**，主要考察你懂不懂数据库设计。

### 3. 什么是事务？事务的四个特性（ACID）
事务是一组SQL操作，要么全部成功，要么全部失败回滚，保证数据一致性。

**ACID四大特性：**
- **A 原子性（Atomicity）**：事务是不可分割的工作单位，要么全部执行，要么不执行
- **C 一致性（Consistency）**：事务执行前后，数据库完整性约束没有被破坏，数据保持一致
- **I 隔离性（Isolation）**：多个并发事务之间相互隔离，不互相干扰
- **D 持久性（Durability）**：事务提交后，对数据的修改是永久的，即使系统崩溃也不会丢失

### 4. 事务隔离级别
- **读未提交（Read Uncommitted）**：一个事务能看到另一个事务未提交的数据
- **读已提交（Read Committed）**：只能看到另一个事务已提交的数据（解决**脏读**）
- **可重复读（Repeatable Read）**：同一事务中多次读取同一数据结果一致（MySQL默认级别，解决**不可重复读**）
- **串行化（Serializable）**：最高隔离级别，完全串行执行，解决所有并发问题，但性能最低

### 5. 脏读、不可重复读、幻读是什么？
- **脏读**：一个事务读取了另一个事务**未提交**的数据，之后那个事务回滚了，读到的数据就是脏数据
- **不可重复读**：同一事务中，两次读取同一数据，得到结果不一样（因为另一个事务修改并提交了）
- **幻读**：同一事务中，两次按条件查询，得到的行数不一样（另一个事务插入/删除了数据）

### 6. 索引是什么？为什么要用索引？
索引是帮助数据库高效查询的数据结构，相当于书的目录。

**优点：**
- 大大提高查询速度
- 可以通过索引实现唯一性约束

**缺点：**
- 索引占用物理空间
- 对数据进行增删改时，索引需要维护，降低了增删改的速度

**哪些列适合建索引：**
- 经常作为查询条件的列
- 经常用于排序、分组的列
- 主键、外键一般都建索引

---

## 二、SQL基础语法（测试常用）

### 1. 查询（最常用，测试90%场景都是查询）
```sql
-- 查询所有列
SELECT * FROM user;

-- 查询指定列
SELECT id, username, email FROM user;

-- 条件查询
SELECT * FROM user WHERE status = 1 AND create_time > '2026-01-01';

-- 模糊查询
SELECT * FROM user WHERE username LIKE '%张%';

-- IN 查询
SELECT * FROM user WHERE id IN (1, 3, 5);

-- 排序 ASC升序 DESC降序
SELECT * FROM user ORDER BY create_time DESC;

-- 分页查询（MySQL）
SELECT * FROM user LIMIT 10 OFFSET 0; -- 第1页，每页10条
SELECT * FROM user LIMIT 0, 10; -- 同上
```

### 2. 插入
```sql
INSERT INTO user (username, email, status) VALUES ('张三', 'zhangsan@example.com', 1);
```

### 3. 更新
```sql
UPDATE user SET status = 0 WHERE id = 1;
```

### 4. 删除
```sql
DELETE FROM user WHERE id = 1;
```

### 5. 聚合函数
```sql
--  COUNT：统计行数
SELECT COUNT(*) FROM user;

-- SUM：求和
SELECT SUM(amount) FROM order WHERE user_id = 1;

-- AVG：平均值
SELECT AVG(price) FROM product;

-- MAX/MIN：最大最小值
SELECT MAX(price), MIN(price) FROM product;
```

### 6. 分组查询
```sql
-- 按状态分组统计用户数
SELECT status, COUNT(*) FROM user GROUP BY status;

-- 分组后筛选，用HAVING
SELECT status, COUNT(*) FROM user GROUP BY status HAVING COUNT(*) > 10;
```

> **注意：WHERE vs HAVING**
> - WHERE 是**分组前**对原始数据过滤
> - HAVING 是**分组后**对分组聚合结果过滤

### 7. 联表查询
```sql
-- INNER JOIN：内连接，只返回两个表匹配的数据
-- 查询订单和对应用户信息
SELECT o.id, o.amount, u.username
FROM `order` o
INNER JOIN user u ON o.user_id = u.id;

-- LEFT JOIN：左连接，左表全部返回，右表匹配不到显示NULL
-- 查询所有用户，以及他们的订单信息
SELECT u.id, u.username, o.id AS order_id
FROM user u
LEFT JOIN `order` o ON u.id = o.user_id;
```

---

## 三、软件测试中SQL使用场景

| 测试场景 | SQL用法 |
|---------|---------|
| 接口新增数据后验证 | `SELECT * FROM 表 WHERE id = 新增ID`，查数据库看字段是否正确插入 |
| 修改功能验证 | `UPDATE 修改后，SELECT 查询验证字段值是否更新正确 |
| 删除功能验证 | 删除后 `SELECT` 验证数据是否真的删除（或逻辑删除标记是否更新） |
| 分页验证 | 用 `COUNT(*)` 先算总条数，再验证分页页数计算是否正确 |
| 统计报表验证 | 用 `SUM/COUNT/AVG` 计算结果，和页面展示的统计数字对比，验证报表准确性 |
| 测试数据准备 | `INSERT` 插入测试需要的数据，用完 `DELETE` 删除 |
| 脏数据清理 | 测试完清理测试数据 `DELETE FROM 表 WHERE name LIKE '测试%'` |

---

## 四、常见面试题

### Q1：你在测试中怎么用SQL？举个例子

**A：**
我在接口测试和功能测试中，经常用SQL做**数据验证**。比如：
- 新增一个订单，前端页面提交后，我会查数据库 `SELECT * FROM order WHERE id = xxx`，看看各个字段是不是正确写入了，金额、状态、时间对不对
- 做统计报表功能测试时，页面显示GMV是100万，我会自己用 `SELECT SUM(amount) FROM order WHERE create_time BETWEEN '开始时间' AND '结束时间'` 算一遍，核对结果是否一致
- 删除数据后，也要查数据库确认是物理删除了还是逻辑删除（打了删除标记），业务要求不同验证方式也不同

### Q2：WHERE 后跟 AND 和 OR 优先级？
**A：** AND 优先级比 OR 高，如果不确定就加括号，避免逻辑错。

### Q3：DROP、DELETE、TRUNCATE 区别？

| 命令 | 类型 | 作用 | 能否回滚 |
|------|------|------|---------|
| DELETE | DML | 删除表中部分数据（WHERE条件筛选），可以回滚 | 可以（事务中） |
| TRUNCATE | DDL | 删除表中所有数据，保留表结构，不能加WHERE，不可回滚 | 不可 |
| DROP | DDL | 删除整个表（结构+数据），不可回滚 | 不可 |

### Q4：CHAR和VARCHAR区别？
- **CHAR**：固定长度，存储速度快，浪费空间，适合存长度固定的数据（比如手机号、身份证号）
- **VARCHAR**：可变长度，节省空间，存储速度稍慢，适合存长度不固定的数据（比如用户名、地址）

### Q5：主键和外键区别？
- **主键**：能唯一标识表中每一行的字段，一个表只能有一个主键，主键不能为NULL，不能重复
- **外键**：表中一个字段关联另一张表的主键，用于保证数据一致性，实现关联查询

### Q6：什么是索引？什么情况下索引会失效？

索引是帮助数据库高效查询的数据结构，相当于书的目录，能大大提高查询速度。

**索引失效场景（带实际例子）：**

| 场景 | 错误写法 | 正确写法 |
|------|---------|---------|
| **对索引字段使用函数** | `WHERE YEAR(create_time) = 2026` | `WHERE create_time BETWEEN '2026-01-01' AND '2026-12-31'` |
| **索引字段参与表达式计算** | `WHERE id + 1 = 5` | `WHERE id = 5 - 1` |
| **LIKE以%开头** | `WHERE username LIKE '%张%'` | `WHERE username LIKE '张%'`（后缀%不影响） |
| **字符串不加引号** | `WHERE phone = 13800000000`（phone是varchar） | `WHERE phone = '13800000000'` |
| **索引字段允许NULL，查询`IS NULL`** | `WHERE phone IS NULL` | 尽量设置字段为 `NOT NULL`，NULL会让索引统计不准确 |
| **使用不等于操作** | `WHERE age != 20` | 如果一定要用，数据量不大影响不大，否则建议调整业务逻辑 |
| **OR条件不全有索引** | `WHERE id=1 OR name='张三'`（只有id有索引） | 改成 `UNION` 分开查询：`SELECT ... WHERE id=1 UNION SELECT ... WHERE name='张三'` |

### 补充：NULL字段、数据倾斜为什么会导致不走索引？

#### 1. NULL字段问题
- 索引不存储 `NULL` 值，所以如果字段允许 `NULL`，索引中就没有这一行记录
- 如果查询 `WHERE phone IS NULL`，索引用不上，会走全表扫描
- 如果查询 `WHERE phone IS NOT NULL`，MySQL优化器也可能因为统计信息不准选择不走索引
- **最佳实践：** 尽量把字段设置为 `NOT NULL`，设置默认值（比如空字符串、0）代替NULL

#### 2. 数据倾斜导致不走索引
**什么是数据倾斜？** 就是某一个值在表中占比太大。

**例子：** 假设有一个 `status` 字段，90% 记录都是 `status=1`，只有10%是 `status=0`。你建了索引在 `status` 上：
```sql
-- 你查询status=1，满足条件的记录占了90%
SELECT * FROM table WHERE status = 1;
```

MySQL优化器会估算：走索引需要回表查询大部分数据，还不如直接全表扫描更快。所以优化器会**放弃索引用全表扫描**。

**这不是索引失效，是优化器根据数据分布做出的选择**——因为确实全表扫描更快。

#### 总结：
- 字段尽量设置 `NOT NULL`，避免NULL导致索引统计不准
- 数据分布倾斜时，优化器可能选择全表扫描，这是优化器的合理选择，不是索引真的"失效"

### Q：什么是组合索引最左前缀原则？

**组合索引**：在多个列上建立一个联合索引，比如 `INDEX(name, age, create_time)`。

**最左前缀原则**：
- 查询时必须从索引的**最左边第一列开始用**，才能用到索引
- 跳过了左边某一列，后面的列就用不到索引了
- 查询条件中左边的列是范围查询（> < between），右边列用不到索引

**实际例子：** 组合索引 `INDEX(name, age, create_time)`

| 查询条件 | 能否用到索引 |
|---------|-------------|
| `WHERE name = '张三' AND age = 20` | ✅ 能用，用到name+age两列 |
| `WHERE name = '张三'` | ✅ 能用，用到name列 |
| `WHERE age = 20` | ❌ 不能用，跳过了左边name列 |
| `WHERE name = '张三' AND create_time = '2026-01-01'` | ✅ 能用name列，create_time用不到 |
| `WHERE name LIKE '张%' AND age = 20` | ✅ 能用name前缀匹配，age也能用 |
| `WHERE name > '张' AND age = 20` | ✅ 能用name列（范围），age用不到 |

**总结：** 建组合索引要把**经常作为查询条件**的列放最左边，遵守最左前缀才能高效利用索引。

### Q7：如何定位慢SQL？
- 开启慢查询日志，找出执行慢的SQL
- 用 `EXPLAIN` 分析SQL执行计划，看有没有走索引

### Q8：什么是视图？
视图是基于SQL查询结果的虚拟表，本身不存储数据，数据还是来自原表，可以简化复杂查询，也能做权限控制。

### Q9：索引在什么情况下会失效？
（补充完整索引失效场景）
- 对字段进行函数操作 (`WHERE YEAR(create_time) = 2026`)
- 索引列参与表达式计算 (`WHERE id+1 = 5`)
- 使用 `!=`、`<>`、`NOT IN` 不等于操作
- LIKE 以通配符开头 (`%abc`)，无法使用索引
- 字符串字段未加引号，发生隐式类型转换
- OR 条件中，不是所有条件列都有索引

### Q10：千万级大表如何加索引？
千万级数据量加索引要注意避免锁表影响业务：
- 使用 **`ONLINE DDL`**（MySQL 5.6+ 支持），加索引的时候不锁表，不影响线上业务读写
- 如果MySQL版本不支持ONLINE DDL，可以用 **`pt-online-schema-change`** 工具，在线改表不锁表
- 一定要**避开业务高峰期**，选择深夜或周末流量低的时候操作
- 先在**测试环境验证**，确认执行时间、对性能的影响、会不会锁表，再去生产环境操作

### Q11：如何验证UI显示的数据和数据库一致？

测试接口或功能完成后，需要验证数据是否正确入库，步骤：
```sql
-- 步骤1：UI操作前，先查询数据库初始状态
SELECT status, balance FROM accounts WHERE id = 123;

-- 步骤2：在UI上执行操作（比如转账、修改状态）

-- 步骤3：操作完成后，再次查询数据库验证
SELECT status, balance, update_time FROM accounts WHERE id = 123;
```
核对字段值：
- 状态变更是否正确
- 金额计算是否正确（扣款/加款）
- 更新时间是否正确
- 关联表数据是否一致

### Q12：如何测试存储过程和触发器？
- **存储过程**：准备好各种测试数据（正常/边界/异常），调用存储过程，检查输出参数和各个表的数据变化是否符合预期
- **触发器**：在主表执行插入/更新/删除操作，检查触发器关联的表是否自动同步变更（比如插入订单后自动扣减库存），验证数据一致性

---

## 四、常见手写SQL面试题（笔试高频）

### 1. 查询第二高的薪水

**employee表：** `id, salary`

**要求：** 编写SQL查询得到第二高的薪水，如果不存在第二高，返回`null`。

**写法一：LIMIT + OFFSET + IFNULL**（推荐）
```sql
SELECT IFNULL(
    (SELECT DISTINCT salary 
     FROM employee 
     ORDER BY salary DESC 
     LIMIT 1 OFFSET 1),
    NULL
) AS second_highest_salary;
```

**写法二：MAX 嵌套查询**
```sql
SELECT MAX(salary) AS second_highest_salary
FROM employee
WHERE salary < (SELECT MAX(salary) FROM employee);
```

---

### 2. 查询/删除重复数据

**查重复姓名：**
```sql
SELECT name, COUNT(*) 
FROM employees 
GROUP BY name 
HAVING COUNT(*) > 1;
```

**查所有字段的完全重复记录：**
```sql
SELECT * 
FROM employees 
WHERE (name, email) IN (
    SELECT name, email 
    FROM employees 
    GROUP BY name, email 
    HAVING COUNT(*) > 1
);
```

**删除重复邮箱，只保留id最小的：**
```sql
DELETE p1 
FROM person p1
INNER JOIN person p2 
WHERE p1.email = p2.email 
AND p1.id > p2.id;
```

**考点**：`GROUP BY` + `HAVING` 组合筛选

---

### 3. 行转列（报表场景）
```sql
-- 场景：成绩表转置（学生姓名+语文/数学/英语三列）
SELECT 
    name,
    MAX(CASE WHEN subject='语文' THEN score END) as 语文,
    MAX(CASE WHEN subject='数学' THEN score END) as 数学,
    MAX(CASE WHEN subject='english' THEN score END) as 英语
FROM scores 
GROUP BY name;
```
**考点**：`CASE WHEN` 条件判断、聚合函数

---

### 4. 三表联查（电商场景）
```sql
-- 查询：用户姓名、订单号、商品名称（近30天订单）
SELECT 
    u.name, 
    o.order_no, 
    p.product_name
FROM users u
INNER JOIN orders o ON u.id = o.user_id
LEFT JOIN order_items oi ON o.id = oi.order_id
LEFT JOIN products p ON oi.product_id = p.id
WHERE o.created_at >= DATE_SUB(NOW(), INTERVAL 30 DAY);
```
**考点**：`JOIN` 类型选择、多表关联、时间函数

---

### 5. 实际项目复杂场景题

#### 场景1：电商订单统计 - 统计每个用户近30天的下单次数和总金额

**原始数据：users表**
| id | username |
|----|----------|
| 1  | 张三     |
| 2  | 李四     |
| 3  | 王五     |

**原始数据：orders表**
| id | user_id | amount | status | created_at |
|----|---------|--------|--------|------------|
| 1  | 1       | 100    | PAID   | 2026-04-01 |
| 2  | 1       | 200    | PAID   | 2026-04-05 |
| 3  | 2       | 150    | PAID   | 2026-04-08 |

**查询结果：**
| id | username | order_count | total_amount |
|----|----------|-------------|--------------|
| 1  | 张三     | 2           | 300          |
| 2  | 李四     | 1           | 150          |
| 3  | 王五     | 0           | NULL         |

**说明：** `LEFT JOIN`保证没下单的用户也会出来（总金额0或NULL），只统计已支付订单，近30天下单。

```sql
SELECT 
    u.id,
    u.username,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_amount
FROM users u
LEFT JOIN orders o 
    ON u.id = o.user_id 
    AND o.status = 'PAID'
    AND o.created_at >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY u.id, u.username
ORDER BY total_amount DESC;
```

#### 场景2：找出连续登录天数 >= 3天的用户

**原始数据：user_login**
| user_id | login_date |
|---------|------------|
| 1       | 2026-04-01 |
| 1       | 2026-04-02 |
| 1       | 2026-04-03 |
| 2       | 2026-04-01 |
| 2       | 2026-04-03 |

**计算过程：**
| user_id | login_date | row_num | DATE_SUB → diff |
|---------|------------|---------|-----------------|
| 1       | 2026-04-01 | 1       | 2026-03-31      |
| 1       | 2026-04-02 | 2       | 2026-03-31      |
| 1       | 2026-04-03 | 3       | 2026-03-31      |
| 2       | 2026-04-01 | 1       | 2026-03-31      |
| 2       | 2026-04-03 | 2       | 2026-04-01      |

**结果：** user_id=1 的diff都是`2026-03-31`，count=3，满足≥3天，所以被选中。

**考点**：开窗函数 `ROW_NUMBER()`、连续问题处理思路

```sql
-- 方法：开窗函数row_number，计算连续日期
SELECT DISTINCT user_id
FROM (
    SELECT 
        user_id,
        login_date,
        DATE_SUB(login_date, INTERVAL row_num DAY) AS diff
    FROM (
        SELECT 
            user_id,
            login_date,
            ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) AS row_num
        FROM user_login
        GROUP BY user_id, login_date -- 去重同一天多次登录
    ) t1
) t2
GROUP BY user_id, diff
HAVING COUNT(*) >= 3;
```

#### 场景3：每个商品分类下价格Top3的商品

**原始数据：products**
| category_id | product_name | price |
|-------------|--------------|-------|
| 1 (电子)    | iPhone      | 5999  |
| 1 (电子)    | Huawei      | 4999  |
| 1 (电子)    | Xiaomi      | 2999  |
| 1 (电子)    | Redmi       | 1999  |
| 2 (服装)    | T恤         | 99    |
| 2 (服装)    | 牛仔裤      | 199   |
| 2 (服装)    | 羽绒服      | 599   |

**计算排名：**
DENSE_RANK() 按 category_id 分组，价格降序排名：
| category_id | product_name | price | rk |
|-------------|--------------|-------|----|
| 1           | iPhone      | 5999  | 1  |
| 1           | Huawei      | 4999  | 2  |
| 1           | Xiaomi      | 2999  | 3  |
| 1           | Redmi       | 1999  | 4  |
| 2           | 羽绒服      | 599   | 1  |
| 2           | 牛仔裤      | 199   | 2  |
| 2           | T恤         | 99    | 3  |

**筛选 rk <= 3**，就能得到每个分类价格前三的商品。

**考点**：开窗函数 `PARTITION BY` 分组排序、TopN问题

```sql
-- 开窗函数DENSE_RANK实现
SELECT category_id, product_name, price
FROM (
    SELECT 
        category_id,
        product_name,
        price,
        DENSE_RANK() OVER (
            PARTITION BY category_id 
            ORDER BY price DESC
        ) AS rk
    FROM products
) t
WHERE rk <= 3;
```

#### 场景4：计算每个月销售额，以及环比上月增长率

**原始数据：orders（按月份聚合后）**
| month   | sales  |
|---------|--------|
| 2026-01 | 100000 |
| 2026-02 | 120000 |
| 2026-03 | 150000 |

**计算过程：**
`LAG(sales)` 取**上一行**的销售额：
| month   | sales  | LAG(sales) | 计算 (120000-100000)/100000*100 | growth_rate |
|---------|--------|-----------|--------------------------------|-------------|
| 2026-01 | 100000 | NULL      | -                              | NULL        |
| 2026-02 | 120000 | 100000    | 20%                            | 20.00       |
| 2026-03 | 150000 | 120000    | 25%                            | 25.00       |

**考点**：CTE公用表达式、`LAG()` 窗口函数获取前一行数据

```sql
WITH monthly_sales AS (
    SELECT 
        DATE_FORMAT(created_at, '%Y-%m') AS month,
        SUM(amount) AS sales
    FROM orders
    WHERE status = 'PAID'
    GROUP BY DATE_FORMAT(created_at, '%Y-%m')
)
SELECT 
    month,
    sales,
    ROUND(
        (sales - LAG(sales) OVER (ORDER BY month)) 
        / LAG(sales) OVER (ORDER BY month) * 100, 
        2
    ) AS growth_rate
FROM monthly_sales;
```

---

### 开窗函数基础知识

什么是 `ROW_NUMBER()` 开窗函数？一句话：
> 给**分组内**的每一行按顺序编一个**行号**，从1开始递增。

语法：
```sql
ROW_NUMBER() OVER (
    PARTITION BY 分组字段  -- 按这个字段分组，每组内部单独编号
    ORDER BY 排序字段      -- 分组内按这个字段排序，然后编号
)
```

**最简单例子：**
| 分类 | 商品 | 价格 |
|------|------|------|
| 电子 | iPhone | 5999 |
| 电子 | Huawei | 4999 |
| 服装 | 羽绒服 | 599 |
| 服装 | T恤 | 99 |

编号后：
| 分类 | 商品 | 价格 | row_number |
|------|------|------|------------|
| 电子 | iPhone | 5999 | **1** |
| 电子 | Huawei | 4999 | **2** |
| 服装 | 羽绒服 | 599 | **1** |
| 服装 | T恤 | 99 | **2** |

> 每个分组（分类）内部，自己从1开始编号 → 这就是开窗函数干的事。

---

### 常见问题：开窗函数会导致索引失效吗？

**不会！**

- SQL执行顺序：`FROM → WHERE → GROUP BY → SELECT → 开窗函数计算 → ORDER BY`
- 开窗函数是在 **WHERE过滤完数据之后** 才执行的
- 索引失效不失效，只看WHERE/JOIN这些过滤条件写法对不对，和开窗函数没关系

真正会导致索引失效的，还是这些情况（和开窗函数没关系）：
- 对索引字段用函数：`WHERE YEAR(create_time) = 2026`
- 字符串不加引号发生隐式转换：`WHERE phone = 13800000000`（phone是varchar）
- LIKE以通配符开头：`WHERE username LIKE '%张%'`

开窗函数唯一的性能问题：如果数据量很大，分组排序会比较耗CPU/内存，但这是计算量大，不是索引失效，该走的索引还是会走。

---

## 逐个场景思路讲解

#### 场景1：电商订单统计 - 统计每个用户近30天的下单次数和总金额

**原始数据：users表**
| id | username |
|----|----------|
| 1  | 张三     |
| 2  | 李四     |
| 3  | 王五     |

**原始数据：orders表**
| id | user_id | amount | status | created_at |
|----|---------|--------|--------|------------|
| 1  | 1       | 100    | PAID   | 2026-04-01 |
| 2  | 1       | 200    | PAID   | 2026-04-05 |
| 3  | 2       | 150    | PAID   | 2026-04-08 |

**查询结果：**
| id | username | order_count | total_amount |
|----|----------|-------------|--------------|
| 1  | 张三     | 2           | 300          |
| 2  | 李四     | 1           | 150          |
| 3  | 王五     | 0           | NULL         |

**思路：** `LEFT JOIN` 关联用户和订单，`GROUP BY 用户`，用 `COUNT` 统计次数，`SUM` 统计总金额。

```sql
SELECT 
    u.id,
    u.username,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_amount
FROM users u
LEFT JOIN orders o 
    ON u.id = o.user_id 
    AND o.status = 'PAID'
    AND o.created_at >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY u.id, u.username
ORDER BY total_amount DESC;
```

---

#### 场景2：找出连续登录天数 >= 3天的用户

**原始数据：user_login**
| user_id | login_date |
|---------|------------|
| 1       | 2026-04-01 |
| 1       | 2026-04-02 |
| 1       | 2026-04-03 |
| 2       | 2026-04-01 |
| 2       | 2026-04-03 |

**计算过程：**
| user_id | login_date | row_num | DATE_SUB → diff |
|---------|------------|---------|-----------------|
| 1       | 2026-04-01 | 1       | 2026-03-31      |
| 1       | 2026-04-02 | 2       | 2026-03-31      |
| 1       | 2026-04-03 | 3       | 2026-03-31      |
| 2       | 2026-04-01 | 1       | 2026-03-31      |
| 2       | 2026-04-03 | 2       | 2026-04-01      |

**思路（开窗函数版本）：**
- `ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY login_date)` → 每个用户按登录日期编号
- `DATE_SUB(login_date, INTERVAL row_num DAY)` → 如果日期连续，这个差值会相等
- 最后统计相同差值行数≥3，就是连续登录≥3天的用户

```sql
SELECT DISTINCT user_id
FROM (
    SELECT 
        user_id,
        login_date,
        DATE_SUB(login_date, INTERVAL row_num DAY) AS diff
    FROM (
        SELECT 
            user_id,
            login_date,
            ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) AS row_num
        FROM user_login
        GROUP BY user_id, login_date -- 去重同一天多次登录
    ) t1
) t2
GROUP BY user_id, diff
HAVING COUNT(*) >= 3;
```

---

**不用开窗函数版本（MySQL 5.x可用）：自连接实现**

```sql
SELECT DISTINCT a.user_id
FROM user_login a
INNER JOIN user_login b 
    ON a.user_id = b.user_id 
    AND DATEDIFF(b.login_date, a.login_date) = 1
INNER JOIN user_login c 
    ON a.user_id = c.user_id 
    AND DATEDIFF(c.login_date, b.login_date) = 1
GROUP BY a.user_id;
```

**思路：**
- 三次自连接，a第一天，b第二天（比a大1天），c第三天（比b大1天）
- 能连起来三天，说明连续登录三天，直接得到结果
- 优点：写法简单易懂；缺点：N天连续就要N次自连接，SQL会变长



---

#### 场景3：每个商品分类下价格Top3的商品

**原始数据：products**
| category_id | product_name | price |
|-------------|--------------|-------|
| 1 (电子)    | iPhone      | 5999  |
| 1 (电子)    | Huawei      | 4999  |
| 1 (电子)    | Xiaomi      | 2999  |
| 1 (电子)    | Redmi       | 1999  |
| 2 (服装)    | T恤         | 99    |
| 2 (服装)    | 牛仔裤      | 199   |
| 2 (服装)    | 羽绒服      | 599   |

**计算排名：**
DENSE_RANK() 按 category_id 分组，价格降序排名：
| category_id | product_name | price | rk |
|-------------|--------------|-------|----|
| 1           | iPhone      | 5999  | 1  |
| 1           | Huawei      | 4999  | 2  |
| 1           | Xiaomi      | 2999  | 3  |
| 1           | Redmi       | 1999  | 4  |
| 2           | 羽绒服      | 599   | 1  |
| 2           | 牛仔裤      | 199   | 2  |
| 2           | T恤         | 99    | 3  |

**思路（开窗函数版本）：**
- `DENSE_RANK() OVER(PARTITION BY category_id ORDER BY price DESC)` → 按分类分组，价格降序排名
- 筛选 `rk <= 3` → 每个分类价格前三的商品就出来了

> `DENSE_RANK()` vs `ROW_NUMBER()`：
> - 如果价格相同，`DENSE_RANK` 会给相同排名，比如两个第一名，下一个就是第二名
> - `ROW_NUMBER` 即使价格相同也会强行分不同排名
> - TopN问题一般用 `DENSE_RANK` 更合理

```sql
SELECT category_id, product_name, price
FROM (
    SELECT 
        category_id,
        product_name,
        price,
        DENSE_RANK() OVER (
            PARTITION BY category_id 
            ORDER BY price DESC
        ) AS rk
    FROM products
) t
WHERE rk <= 3;
```

---

**不用开窗函数版本（MySQL 5.x可用）：**

```sql
SELECT a.category_id, a.product_name, a.price
FROM products a
WHERE (
    SELECT COUNT(*) 
    FROM products b 
    WHERE b.category_id = a.category_id 
      AND b.price >= a.price
) <= 3
ORDER BY a.category_id, a.price DESC;
```

**思路：**
对于每个商品a，统计同分类下**价格大于等于它**的商品有多少个
- 如果count ≤ 3，说明它至少是前三，保留
- 如果count > 3，说明它排在前三名之外，过滤掉

这个方法叫**相关性子查询**，不用开窗函数也能实现TopN。

---

#### 场景4：计算每个月销售额，以及环比上月增长率

**原始数据：orders（按月份聚合后）**
| month   | sales  |
|---------|--------|
| 2026-01 | 100000 |
| 2026-02 | 120000 |
| 2026-03 | 150000 |

**计算过程：**
`LAG(sales)` 取**上一行**（上个月）的销售额：
| month   | sales  | LAG(sales) → 上月 | 增长率 = (本月-上月)/上月 × 100 | growth_rate |
|---------|--------|------------------|------------------------------|-------------|
| 2026-01 | 100000 | NULL             | -                            | NULL        |
| 2026-02 | 120000 | 100000           | 20%                          | 20.00       |
| 2026-03 | 150000 | 120000           | 25%                          | 25.00       |

**思路：**
- 先用CTE按月聚合销售额
- `LAG(sales) OVER (ORDER BY month)` → LAG是开窗函数，取出当前行**上一行**（上个月）的值
- 增长率公式：`(本月销售额 - 上月销售额) / 上月销售额 × 100%`

```sql
WITH monthly_sales AS (
    SELECT 
        DATE_FORMAT(created_at, '%Y-%m') AS month,
        SUM(amount) AS sales
    FROM orders
    WHERE status = 'PAID'
    GROUP BY DATE_FORMAT(created_at, '%Y-%m')
)
SELECT 
    month,
    sales,
    ROUND(
        (sales - LAG(sales) OVER (ORDER BY month)) 
        / LAG(sales) OVER (ORDER BY month) * 100, 
        2
    ) AS growth_rate
FROM monthly_sales;
```

---

#### 场景5：统计新用户7日留存

**什么是7日留存？**：注册日第7天还登录过的用户 ÷ 总新用户 = 留存率

**原始数据：user_login**
| user_id | login_time          |
|---------|---------------------|
| 1       | 2026-04-01 10:00   | 首次注册登录
| 1       | 2026-04-08 14:00   | 注册后第7天又登录了 → 留存
| 2       | 2026-04-01 11:00   | 首次注册登录
| 2       | 2026-04-05 09:00   | 中间登录，但第7天没登录 → 不留存
| 3       | 2026-04-01 08:00   | 首次注册登录，之后再也没登录 → 不留存

**计算结果：**
| register_date | new_users | retained_users | retention_rate_7d |
|---------------|-----------|----------------|-------------------|
| 2026-04-01    | 3         | 1              | 33.33             |

**思路：**
1. 第一步CTE：`MIN(DATE(login_time))` 找出每个用户**第一次登录日期**（就是注册日）
2. 第二步LEFT JOIN：把同一个用户，注册日和7天后的登录记录连起来
3. 如果能join到，说明这个用户注册7天后还登录了，就是留存用户
4. 最后 `留存数 ÷ 新用户数 × 100% = 留存率`

```sql
-- 第一步：找出每个用户的首次注册日期
WITH first_login AS (
    SELECT 
        user_id,
        MIN(DATE(login_time)) AS register_date
    FROM user_login
    GROUP BY user_id
)
-- 第二步：统计7天后还登录的用户，计算留存率
SELECT 
    fl.register_date,
    COUNT(DISTINCT fl.user_id) AS new_users,
    COUNT(DISTINCT ul.user_id) AS retained_users,
    ROUND(
        COUNT(DISTINCT ul.user_id) / COUNT(DISTINCT fl.user_id) * 100, 
        2
    ) AS retention_rate_7d
FROM first_login fl
LEFT JOIN user_login ul 
    ON fl.user_id = ul.user_id 
    AND DATE(ul.login_time) = DATE_ADD(fl.register_date, INTERVAL 7 DAY)
GROUP BY fl.register_date
ORDER BY fl.register_date;
```

---

## 五、测试专用SQL场景题

### 1. 批量构造测试数据
测试需要多条不同状态的订单数据，手动插入太麻烦，用SQL批量生成：
```sql
-- 批量插入100条测试订单（不同状态）
INSERT INTO orders (order_no, user_id, status, amount, created_at)
SELECT 
    CONCAT('TEST', LPAD(seq, 6, '0')),
    FLOOR(1 + RAND() * 1000),
    ELT(1 + FLOOR(RAND() * 3), 'PENDING', 'PAID', 'SHIPPED'),
    ROUND(RAND() * 1000, 2),
    DATE_SUB(NOW(), INTERVAL FLOOR(RAND() * 30) DAY)
FROM (
    SELECT @seq := @seq + 1 as seq 
    FROM (SELECT 0 UNION ALL SELECT 1 UNION ALL SELECT 2) t1,
         (SELECT 0 UNION ALL SELECT 1 UNION ALL SELECT 2) t2,
         (SELECT @seq := 0) t3
    LIMIT 100
) numbers;
```
执行一次就能生成100条随机测试数据，不用一条条手敲。

### 2. 边界值测试数据查询
```sql
-- 测试金额边界（0元、负数、超大金额），找出不符合业务规则的数据
SELECT * FROM orders WHERE amount <= 0 OR amount > 999999999;

-- 测试日期边界（1970年前、未来日期）
SELECT * FROM users WHERE birth_date < '1970-01-01' OR birth_date > NOW();
```

### 3. 脏数据检查（测试经常用）
```sql
-- 查找孤儿记录：订单表有user_id，但用户表找不到这个用户
SELECT o.* 
FROM orders o 
LEFT JOIN users u ON o.user_id = u.id 
WHERE u.id IS NULL;

-- 查找状态不一致：已支付但超过7天还没发货
SELECT * FROM orders 
WHERE status = 'PAID' 
AND DATEDIFF(NOW(), pay_time) > 7 
AND ship_time IS NULL;
```

### 4. 测试数据清理
测试完清理环境，删掉测试数据：
```sql
-- 删除所有测试订单（订单号以TEST开头）
DELETE FROM orders WHERE order_no LIKE 'TEST%';
```

---

## 六、场景题（实际测试中常见）

### 场景1：有一张用户表 `user`，字段：id(主键), username, email, status(0禁用 1启用), create_time

**Q1：查询状态为启用，2026年1月以后注册的用户，按注册时间倒序排列**

```sql
SELECT id, username, email, create_time
FROM user
WHERE status = 1 AND create_time >= '2026-01-01'
ORDER BY create_time DESC;
```

**Q2：统计不同状态下的用户数量**

```sql
SELECT status, COUNT(*) AS user_count
FROM user
GROUP BY status;
```

**Q3：查询用户名包含"张"的用户，分页第2页，每页10条**

```sql
SELECT * FROM user WHERE username LIKE '%张%' LIMIT 10 OFFSET 10;
-- 或者
SELECT * FROM user WHERE username LIKE '%张%' LIMIT 10, 10;
```

---

### 场景2：有两张表，用户表 `user` 和订单表 `order`

**user表：**
```
id   username
1    张三
2    李四
3    王五
```

**order表：**
```
id   user_id   amount
1    1         100
2    1         200
3    2         150
```

**Q1：查询每个用户的订单总金额，要求显示用户名和总金额**

```sql
SELECT u.username, IFNULL(SUM(o.amount), 0) AS total_amount
FROM user u
LEFT JOIN `order` o ON u.id = o.user_id
GROUP BY u.id, u.username;
```

> 说明：用LEFT JOIN保证没有订单的用户也能显示出来，用IFNULL把NULL换成0

**结果：**
```
username  total_amount
张三      300
李四      150
王五      0
```

**Q2：查询订单总金额大于200的用户**

```sql
SELECT u.username, SUM(o.amount) AS total_amount
FROM user u
LEFT JOIN `order` o ON u.id = o.user_id
GROUP BY u.id, u.username
HAVING SUM(o.amount) > 200;
```

---

### 场景3：测试工作中，接口新增一条数据，你怎么验证数据正确入库？

**回答：**
1. 先记住新增成功后返回的主键ID
2. 根据ID查数据库：`SELECT * FROM 表名 WHERE id = 新增ID;`
3. 逐个核对每个字段的值：
   - 字符串类看内容、编码是否正确
   - 数字类看数值计算是否正确
   - 日期时间看格式、时区是否正确
   - 状态值看枚举值是否正确
4. 如果有关联数据，还要查关联表验证关联关系是否正确

---

### 场景4：页面删除一条数据，测试你要怎么验证？

**回答：**
要看业务要求是**物理删除**还是**逻辑删除**：
- **物理删除**：删除后 `SELECT * FROM 表 WHERE id = xxx`，查不到数据就是对的
- **逻辑删除**：删除后查询，能查到数据，但删除标记字段（deleted/is_deleted/status）会变成已删除状态（比如1表示删除）

---

### 场景5：怎么验证分页功能的数据总数和页数正确？

**回答：**
1. 先用 `SELECT COUNT(*) FROM 表 WHERE 查询条件` 得到总条数
2. 用总条数 ÷ 每页条数 = 总页数，看页面显示的总页数是否和计算的一致
3. 翻到最后一页，看数据条数对不对（如果最后一页不满）

---

## 七、面试应答技巧

### Q：遇到不会写的SQL怎么办？
1. **先讲思路**："我需要先找到A表和B表的关联字段，然后筛选条件..."，不要直接说我不会
2. **分步实现**："第一步先查出所有满足条件的ID，第二步再关联另一张表拿详情..."
3. **展示优化意识**："这个写法在数据量大时可能慢，可以考虑给XX字段加索引"

### Q：加分回答示例：你发现页面显示订单金额和数据库不一致，怎么办？

> 我会按这个步骤排查：
> 1. 先用F12抓包，确认前端传给后端的参数对不对，排除前端显示问题
> 2. 直接查数据库核对数据：
> ```sql
> SELECT order_amount, discount, final_amount 
> FROM orders WHERE order_no = 'xxx';
> ```
> 3. 如果金额对不上，先看字段类型，排查是不是FLOAT精度丢失问题，应该用DECIMAL存金额
> 4. 如果是并发操作导致的数据不一致，我会查事务隔离级别和锁机制，看看是不是脏读导致
> 5. 最后我会写个SQL脚本批量核对历史数据，评估问题影响范围，协助开发定位修复

### Q：你在测试工作中，什么时候会用SQL？
（回答要点：结合测试场景说）
> 主要做数据验证：
> 1. 接口新增/修改数据后，页面没展示或者展示不全，我直接查数据库看是不是真的写对了
> 2. 统计报表功能，页面显示的总数、金额不对，我自己用COUNT/SUM算一遍核对
> 3. 删除功能，验证是不是真删了还是只是逻辑删除
> 4. 排查问题的时候，页面报错了，去数据库看数据是不是脏数据导致的
> 5. 批量造测试数据，或者测完清理测试数据

---

## 八、总结（软件测试对SQL要求）

软件测试不需要会写复杂的存储过程、深度优化SQL，重点掌握：
1. **会写增删改查，特别是条件查询、聚合统计、联表查询**
2. **能在测试中用SQL做数据验证**——接口/功能操作后，去数据库核对数据对不对
3. 懂基础概念：事务、索引、隔离级别这些，能回答出基础概念就行

面试中**结合实际测试场景说你怎么用SQL**，比光背概念加分更多。
