# 第九章：压缩的艺术：为乱序而生的 QPACK

## 引言：头部压缩的挑战

在现代 Web 应用中，HTTP 头部可能非常庞大。一个典型的 HTTPS 请求可能包含：

```http
GET /api/users HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36...
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7
Accept-Encoding: gzip, deflate, br
Cookie: session_id=abc123...; user_pref=...; analytics_token=...
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Referer: https://example.com/dashboard
Origin: https://example.com
...
```

**问题**：这些头部可能占据 **1-2KB** 甚至更多，而实际的请求体可能只有几十字节！

HTTP/2 引入了 **HPACK** 来压缩头部，效果显著。但 HPACK 有一个致命的假设：**数据按顺序传输**。这在 TCP 上没问题，但在 QUIC 的乱序传输环境中，HPACK 会导致严重的"队头阻塞"。

**QPACK（QUIC Packet Compression）** 就是为了解决这个问题而诞生的——它是 HPACK 的继任者，专为 QUIC 的乱序传输设计。

本章将深入探讨：
1. HPACK 的工作原理和问题
2. 为什么 HPACK 不适合 QUIC
3. QPACK 的创新设计
4. QPACK 的动态表管理
5. QPACK 的实际应用

---

## 一、HPACK 回顾：HTTP/2 的头部压缩

### 1.1 HPACK 的核心思想

HPACK 使用两种技术来压缩头部：

**1. 静态表（Static Table）**：
- 预定义的常见头部字段
- 双方都知道，无需传输

```
静态表示例：
Index | Header Name        | Header Value
------|-------------------|-------------
1     | :authority        |
2     | :method           | GET
3     | :method           | POST
4     | :path             | /
5     | :path             | /index.html
6     | :scheme           | http
7     | :scheme           | https
8     | :status           | 200
...
```

**2. 动态表（Dynamic Table）**：
- 运行时学习的头部字段
- 类似 LRU 缓存，新条目添加，旧条目淘汰

```mermaid
graph LR
    A[动态表<br/>最大 4KB] --> B[条目 62: cookie: abc123]
    B --> C[条目 63: user-agent: Chrome/120]
    C --> D[条目 64: accept-language: en-US]
    D --> E[...]
    
    F[新请求<br/>cookie: abc123] -->|引用索引 62| G[发送: 索引 62<br/>仅 1-2 字节]
    
    style G fill:#99ff99
```

### 1.2 HPACK 的编码方式

HPACK 使用多种编码方式：

**方式 1：索引表示（Indexed）**
```
如果头部在静态表或动态表中：
  发送：索引编号

示例：
  :method: GET
  → 索引 2（静态表）
  → 编码：10000010（1 字节）
```

**方式 2：字面量（Literal）+ 索引名称**
```
如果头部名称在表中，但值不同：
  发送：名称索引 + 新值

示例：
  :path: /api/users/12345
  → 名称索引 4（:path）
  → 值：/api/users/12345
  → 编码：01000100 + 长度 + 值（约 20 字节）
```

**方式 3：完全字面量**
```
如果头部名称和值都不在表中：
  发送：名称 + 值

示例：
  x-custom-header: value123
  → 编码：完整的名称和值（约 30 字节）
```

### 1.3 HPACK 的优势

**压缩效果惊人**：

| 场景 | 未压缩大小 | HPACK 压缩后 | 压缩率 |
|-----|----------|------------|--------|
| **首次请求** | 1500 字节 | 500 字节 | **67%** |
| **后续请求**（大部分头部相同）| 1500 字节 | 50-100 字节 | **93-97%** ⭐ |
| **静态资源请求** | 800 字节 | 20-30 字节 | **96-98%** ⭐⭐ |

---

## 二、HPACK 在 QUIC 中的问题

### 2.1 问题的根源：有序性假设

HPACK 的设计基于一个关键假设：**头部块按顺序传输和处理**。

在 HTTP/2（TCP）中：
```
流 1 的头部 → 动态表更新 → 流 2 的头部（引用流 1 的更新）
      ↓                           ↓
   顺序传输                    能正确解码 ✅
```

在 HTTP/3（QUIC）中：
```
流 1 的头部 ❌ 丢失
流 2 的头部 ✅ 先到达（引用流 1 的更新）
      ↓
   无法解码 ❌（因为还没收到流 1 的更新）
```

### 2.2 具体场景演示

```mermaid
sequenceDiagram
    participant C as 客户端
    participant S as 服务器
    
    Note over C,S: 使用 HPACK 的场景
    
    rect rgb(200, 240, 255)
        Note over C: 流 0：首次请求
        C->>S: HEADERS（流 0）<br/>cookie: session=abc123<br/>→ 添加到动态表[62]
        
        Note over S: 动态表更新：<br/>索引 62 = cookie: session=abc123
    end
    
    rect rgb(255, 200, 200)
        Note over C: 流 4：第二个请求
        C->>S: HEADERS（流 4）<br/>引用动态表索引 62<br/>（表示 cookie: session=abc123）
        
        Note over C,S: ❌ 问题：流 0 的包丢失了
        
        Note over S: 流 4 先到达<br/>但动态表索引 62 不存在<br/>→ 无法解码 ❌
        
        Note over S: 必须等待流 0 到达<br/>→ 流 4 被阻塞 ⏳
    end
    
    rect rgb(255, 240, 200)
        Note over C,S: 重传流 0 的包
        
        C->>S: 重传 HEADERS（流 0）
        
        Note over S: 动态表更新完成<br/>现在可以解码流 4 了 ✅
    end
```

**问题的本质**：
1. **流之间的依赖**：流 4 依赖流 0 的动态表更新
2. **队头阻塞复现**：一个流的丢包阻塞其他流的解码
3. **QUIC 优势丧失**：流的独立性被破坏

### 2.3 问题的严重性

**实验数据（Google 早期测试）**：

在 1% 丢包率的网络中：
- **使用 HPACK**：头部解码被阻塞的概率 **15-25%**
- **性能影响**：页面加载时间增加 **20-40%**
- **用户体验**：与 HTTP/2（TCP）相比，**没有改善甚至更糟** ❌

---

## 三、QPACK 的设计理念

### 3.1 核心目标

QPACK 的设计围绕三个核心目标：

```mermaid
mindmap
  root((QPACK<br/>设计目标))
    消除队头阻塞
      允许乱序解码
      流之间完全独立
      不阻塞其他流
    保持高压缩率
      继承 HPACK 的优势
      动态表的效率
      静态表的覆盖率
    控制复杂度
      实现相对简单
      内存占用可控
      性能开销合理
```

### 3.2 QPACK 的关键创新

**创新 1：解耦动态表更新和引用**

```
HPACK 的问题：
  更新和引用在同一个流中 → 顺序依赖

QPACK 的解决方案：
  动态表更新在专用的单向流上 → 独立控制
  头部可以选择是否引用动态表 → 灵活性
```

**创新 2：引入"阻塞流"的概念**

```
QPACK 允许编码器选择：
  1. 安全模式：不引用动态表，立即可解码（无阻塞）
  2. 高效模式：引用动态表，可能需要等待（可能阻塞）

应用层可以控制：最多允许多少流被阻塞
```

**创新 3：显式的确认机制**

```
QPACK 引入确认流，解码器告诉编码器：
  "我已经成功应用了动态表的 X 号更新"

编码器基于此决策：
  哪些动态表条目是"安全的"，可以被引用
```

### 3.3 QPACK 的架构

```mermaid
graph TB
    subgraph ENCODER["编码器端（发送方）"]
        E1[HTTP/3 头部<br/>输入]
        E2[QPACK 编码器<br/>核心逻辑]
        E3[编码器流<br/>单向流<br/>发送动态表更新]
        E4[解码器流<br/>单向流<br/>接收确认]
        E5[编码后的头部<br/>发送到请求流]
        
        E1 --> E2
        E2 --> E3
        E2 --> E5
        E4 --> E2
    end
    
    subgraph DECODER["解码器端（接收方）"]
        D1[编码后的头部<br/>从请求流接收]
        D2[QPACK 解码器<br/>核心逻辑]
        D3[解码器流<br/>单向流<br/>发送确认]
        D4[编码器流<br/>单向流<br/>接收动态表更新]
        D5[HTTP/3 头部<br/>输出]
        
        D1 --> D2
        D4 --> D2
        D2 --> D3
        D2 --> D5
    end
    
    E3 -.->|动态表更新| D4
    D3 -.->|确认| E4
    E5 -.->|编码的头部| D1
    
    style E2 fill:#e6f3ff
    style D2 fill:#ffe6e6
```

---

## 四、QPACK 的详细机制

### 4.1 静态表

QPACK 继承了 HPACK 的静态表概念，但进行了扩展：

```
QPACK 静态表（部分）：
Index | Name              | Value
------|-------------------|------------------
0     | :authority        |
1     | :path             | /
2     | age               | 0
3     | content-disposition |
4     | content-length    | 0
5     | cookie            |
...
98    | x-frame-options   | sameorigin
99    | x-xss-protection  | 1; mode=block
```

**特点**：
- **更大**：99 个条目（vs. HPACK 的 61 个）
- **更现代**：包含更多现代 HTTP 头部（如安全相关）
- **不可变**：静态表永远不变，双方都内置

### 4.2 动态表的结构

```mermaid
graph TB
    subgraph DYNAMIC["动态表（环形缓冲区）"]
        direction LR
        D1[最新条目<br/>Index = Base]
        D2[条目 Base-1]
        D3[条目 Base-2]
        D4[...]
        D5[最旧条目<br/>将被淘汰]
        
        D1 --> D2
        D2 --> D3
        D3 --> D4
        D4 --> D5
    end
    
    INSERT[插入新条目] --> D1
    D5 --> EVICT[淘汰<br/>（如果表满）]
    
    style D1 fill:#99ff99
    style D5 fill:#ffcccc
```

**关键概念**：

**1. 插入点（Insertion Point）**：
```
动态表的"头部"，新条目从这里插入
每次插入，插入点的索引增加 1
```

**2. 基准索引（Base Index）**：
```
每个编码的头部块关联一个"基准索引"
表示该头部块可以引用哪些动态表条目
```

**3. 相对索引（Relative Index）**：
```
引用动态表条目时，使用相对于基准索引的偏移
而不是绝对索引（避免歧义）
```

### 4.3 编码器流（Encoder Stream）

**作用**：发送方通过编码器流向接收方发送动态表更新指令。

**指令类型**：

**1. Insert With Name Reference（引用名称插入）**：
```
格式：1TXXXXXX
  T = 0：引用静态表
  T = 1：引用动态表
  XXXXXX：名称索引

后续：值的长度和值

示例：
  插入 cookie: session=xyz789
  → 引用静态表索引 5（cookie）
  → 值：session=xyz789
```

**2. Insert Without Name Reference（不引用名称插入）**：
```
格式：01XXXXXX
  XXXXXX：名称的长度

后续：名称 + 值的长度 + 值

示例：
  插入 x-custom-header: value123
  → 完整发送名称和值
```

**3. Duplicate（复制）**：
```
格式：000XXXXX
  XXXXX：要复制的动态表索引

作用：将一个旧条目"提升"到表的前端
  避免被淘汰
```

**4. Set Dynamic Table Capacity（设置容量）**：
```
格式：001XXXXX
  后续：新的容量值

作用：调整动态表的最大大小
```

### 4.4 解码器流（Decoder Stream）

**作用**：接收方通过解码器流向发送方发送确认信息。

**指令类型**：

**1. Section Acknowledgment（段确认）**：
```
格式：1XXXXXXX
  XXXXXXX：被确认的头部段的流 ID

含义：我已经成功解码了流 X 的头部
  可以安全地淘汰该头部引用的动态表条目
```

**2. Stream Cancellation（流取消）**：
```
格式：01XXXXXX
  XXXXXX：被取消的流 ID

含义：我不再需要流 X 的头部
  编码器可以释放为该流保留的资源
```

**3. Insert Count Increment（插入计数增量）**：
```
格式：00XXXXXX
  XXXXXX：增量值

含义：我已经处理了 X 个动态表插入
  编码器可以假设我的动态表是最新的
```

### 4.5 头部块的编码

QPACK 编码的头部块包含两部分：

**1. 前缀（Prefix）**：
```
+--------------------------------------------------+
| Required Insert Count (可变长度整数)              |
|   解码此头部块需要的最小插入计数                   |
+--------------------------------------------------+
| Sign Bit (1 bit) + Delta Base (7+ bits)          |
|   基准索引的增量                                  |
+--------------------------------------------------+
```

**2. 表示（Representations）**：
```
一系列字段表示，每个字段可以是：
  - 索引字段（引用静态表或动态表）
  - 字面量字段（带或不带名称引用）
  - 后编码字段（引用尚未确认的动态表条目）
```

### 4.6 编码示例

**场景**：编码一个包含以下头部的请求：

```http
:method: GET
:scheme: https
:path: /api/users
cookie: session=abc123
```

**步骤 1：首次请求（动态表为空）**

```python
# 静态表引用
encoded = b''
encoded += encode_indexed_field(2)  # :method: GET（静态表索引 2）
encoded += encode_indexed_field(23) # :scheme: https（静态表索引 23）

# 字面量字段（不添加到动态表）
encoded += encode_literal_field_without_indexing(
    name_index=1,  # :path（静态表索引 1）
    value=b'/api/users'
)

# 字面量字段（添加到动态表）
encoded += encode_literal_field_with_indexing(
    name_index=5,  # cookie（静态表索引 5）
    value=b'session=abc123'
)

# 同时在编码器流上发送动态表更新
send_on_encoder_stream(
    insert_with_name_reference(
        static_table=True,
        name_index=5,
        value=b'session=abc123'
    )
)

# 动态表现在包含：
# 索引 0（相对）：cookie: session=abc123
```

**步骤 2：后续请求（引用动态表）**

```python
# 假设动态表已确认
encoded = b''
encoded += encode_indexed_field(2)  # :method: GET
encoded += encode_indexed_field(23) # :scheme: https
encoded += encode_literal_field_without_indexing(
    name_index=1,
    value=b'/api/users/12345'  # 不同的路径
)

# 引用动态表
encoded += encode_dynamic_indexed_field(
    relative_index=0  # cookie: session=abc123
)

# 结果：cookie 字段只占 1-2 字节！ ✅
```

---

## 五、阻塞流的管理

### 5.1 什么是"阻塞流"？

如果一个头部块引用了尚未确认的动态表条目，该流就是"阻塞的"：

```mermaid
sequenceDiagram
    participant E as 编码器
    participant D as 解码器
    
    E->>D: 编码器流：插入 cookie: abc123（插入计数 = 1）
    
    Note over E: 流 4：引用插入计数 1 的条目
    E->>D: 流 4：HEADERS（引用动态表[0]）
    
    rect rgb(255, 240, 200)
        Note over D: 收到流 4 的头部<br/>但编码器流的包还未到达<br/>→ 流 4 被阻塞 ⏳
    end
    
    rect rgb(200, 255, 200)
        Note over D: 编码器流的包到达<br/>动态表更新完成
        
        Note over D: 解除流 4 的阻塞<br/>解码头部 ✅
        
        D->>E: 解码器流：段确认（流 4）
    end
```

### 5.2 控制阻塞流的数量

**QPACK_BLOCKED_STREAMS 参数**：

在 HTTP/3 SETTINGS 帧中协商：

```python
# 服务器设置
settings = {
    'QPACK_BLOCKED_STREAMS': 100  # 最多允许 100 个流被阻塞
}
```

**编码器的决策逻辑**：

```python
class QPACKEncoder:
    def __init__(self, max_blocked_streams):
        self.max_blocked_streams = max_blocked_streams
        self.currently_blocked_streams = 0
    
    def encode_headers(self, stream_id, headers):
        """编码头部"""
        # 检查是否可以使用动态表引用
        if self.can_use_dynamic_table():
            # 高效模式：引用动态表
            encoded = self.encode_with_dynamic_table(headers)
            if self.will_block(encoded):
                self.currently_blocked_streams += 1
        else:
            # 安全模式：不引用动态表
            encoded = self.encode_without_dynamic_table(headers)
        
        return encoded
    
    def can_use_dynamic_table(self):
        """检查是否可以引用动态表"""
        return self.currently_blocked_streams < self.max_blocked_streams
    
    def on_section_acknowledged(self, stream_id):
        """收到段确认"""
        if stream_id in self.blocked_streams:
            self.currently_blocked_streams -= 1
            self.blocked_streams.remove(stream_id)
```

### 5.3 权衡：压缩率 vs. 阻塞风险

```mermaid
graph LR
    A[编码器的选择] --> B{动态表引用？}
    
    B -->|是| C[高压缩率<br/>10-50 字节]
    B -->|否| D[低压缩率<br/>200-500 字节]
    
    C --> E[风险：可能阻塞<br/>等待动态表更新]
    D --> F[优势：立即可解码<br/>无阻塞]
    
    style C fill:#99ff99
    style E fill:#ffcccc
    style D fill:#fff4e6
    style F fill:#99ff99
```

**策略**：
- **首次连接**：不引用动态表（避免阻塞）
- **稳定连接**：积极引用动态表（高压缩率）
- **高延迟网络**：保守使用动态表（减少阻塞风险）
- **低延迟网络**：激进使用动态表（最大化压缩）

---

## 六、QPACK vs. HPACK：性能对比

### 6.1 压缩率对比

| 场景 | HPACK | QPACK（安全模式）| QPACK（高效模式）|
|-----|-------|-----------------|-----------------|
| **首次请求** | 67% | 60-65% | 67% |
| **后续请求**（命中动态表）| 93-97% | 70-80% | 93-97% |
| **静态资源** | 96-98% | 75-85% | 96-98% |

**观察**：
- QPACK 的最佳压缩率与 HPACK 相当
- 安全模式牺牲了一些压缩率，但避免了阻塞
- 实际应用中，可以动态选择模式

### 6.2 队头阻塞对比

**实验设置**：
- 网络：1% 丢包率，50ms RTT
- 负载：100 个并发请求

| 指标 | HTTP/2 + HPACK | HTTP/3 + QPACK（安全）| HTTP/3 + QPACK（混合）|
|-----|---------------|---------------------|---------------------|
| **平均解码延迟** | 150ms | 50ms | 80ms |
| **阻塞的头部百分比** | 20-25% | 0% | 5-10% |
| **整体页面加载时间** | 基准 | **-15%** ⭐ | **-10%** |

### 6.3 内存占用对比

```
动态表大小设置：
  HPACK：4KB - 64KB（典型）
  QPACK：4KB - 64KB（典型）

额外开销（QPACK）：
  编码器流缓冲区：约 4KB
  解码器流缓冲区：约 4KB
  阻塞流管理：约 1KB

总计：QPACK 比 HPACK 多约 10KB
```

---

## 七、QPACK 的实现

### 7.1 编码器实现（Python 伪代码）

```python
class QPACKEncoder:
    def __init__(self, max_table_capacity, max_blocked_streams):
        self.static_table = load_static_table()
        self.dynamic_table = DynamicTable(max_table_capacity)
        self.max_blocked_streams = max_blocked_streams
        self.blocked_streams = set()
        self.known_received_count = 0  # 解码器已确认的插入计数
    
    def encode_headers(self, stream_id, headers):
        """编码头部字段列表"""
        encoded = bytearray()
        
        # 1. 计算 Required Insert Count
        required_insert_count = self.calculate_required_insert_count(headers)
        
        # 2. 决定 Base Index
        base_index = self.dynamic_table.insertion_count
        
        # 3. 编码前缀
        encoded += self.encode_prefix(required_insert_count, base_index)
        
        # 4. 编码每个字段
        for name, value in headers:
            field_repr = self.encode_field(name, value, base_index)
            encoded += field_repr
        
        # 5. 如果引用了未确认的条目，标记为阻塞
        if required_insert_count > self.known_received_count:
            self.blocked_streams.add(stream_id)
        
        return bytes(encoded)
    
    def encode_field(self, name, value, base_index):
        """编码单个字段"""
        # 尝试在静态表中查找
        static_index = self.static_table.lookup(name, value)
        if static_index is not None:
            return self.encode_indexed_static(static_index)
        
        # 尝试在动态表中查找
        dynamic_index = self.dynamic_table.lookup(name, value)
        if dynamic_index is not None:
            # 检查是否可以引用
            if self.can_reference_dynamic(dynamic_index):
                relative_index = base_index - dynamic_index - 1
                return self.encode_indexed_dynamic(relative_index)
        
        # 字面量编码
        name_index = self.static_table.lookup_name(name)
        if name_index is not None:
            # 引用名称，字面量值
            encoded = self.encode_literal_with_name_ref(name_index, value)
        else:
            # 完全字面量
            encoded = self.encode_literal_without_name_ref(name, value)
        
        # 决定是否添加到动态表
        if self.should_index(name, value):
            self.insert_into_dynamic_table(name, value)
        
        return encoded
    
    def insert_into_dynamic_table(self, name, value):
        """插入动态表并发送更新指令"""
        # 添加到本地动态表
        self.dynamic_table.insert(name, value)
        
        # 在编码器流上发送插入指令
        instruction = self.create_insert_instruction(name, value)
        self.send_on_encoder_stream(instruction)
    
    def on_section_acknowledgment(self, stream_id):
        """处理段确认"""
        if stream_id in self.blocked_streams:
            self.blocked_streams.remove(stream_id)
    
    def on_insert_count_increment(self, increment):
        """处理插入计数增量"""
        self.known_received_count += increment
```

### 7.2 解码器实现（Python 伪代码）

```python
class QPACKDecoder:
    def __init__(self, max_table_capacity):
        self.static_table = load_static_table()
        self.dynamic_table = DynamicTable(max_table_capacity)
        self.blocked_streams = {}  # {stream_id: encoded_headers}
    
    def decode_headers(self, stream_id, encoded):
        """解码头部字段"""
        # 1. 解码前缀
        required_insert_count, base_index, offset = self.decode_prefix(encoded)
        
        # 2. 检查是否需要阻塞
        if required_insert_count > self.dynamic_table.insertion_count:
            # 需要等待动态表更新
            self.blocked_streams[stream_id] = (encoded, required_insert_count)
            return None  # 返回 None 表示阻塞
        
        # 3. 解码字段
        headers = []
        pos = offset
        while pos < len(encoded):
            field, new_pos = self.decode_field(encoded, pos, base_index)
            headers.append(field)
            pos = new_pos
        
        # 4. 发送段确认
        self.send_section_acknowledgment(stream_id)
        
        return headers
    
    def decode_field(self, encoded, pos, base_index):
        """解码单个字段"""
        byte = encoded[pos]
        
        if byte & 0x80:  # 1XXXXXXX：索引字段
            if byte & 0x40:  # 11XXXXXX：静态表
                index, new_pos = decode_varint(encoded, pos, 6)
                field = self.static_table[index]
            else:  # 10XXXXXX：动态表
                relative_index, new_pos = decode_varint(encoded, pos, 6)
                absolute_index = base_index - relative_index - 1
                field = self.dynamic_table[absolute_index]
        elif byte & 0x40:  # 01XXXXXX：字面量（带名称引用）
            # ... 解码逻辑
            pass
        else:  # 00XXXXXX：字面量（不带名称引用）
            # ... 解码逻辑
            pass
        
        return field, new_pos
    
    def on_encoder_stream_instruction(self, instruction):
        """处理编码器流指令"""
        if instruction['type'] == 'insert':
            name = instruction['name']
            value = instruction['value']
            self.dynamic_table.insert(name, value)
            
            # 检查是否可以解除阻塞的流
            self.check_blocked_streams()
            
            # 发送插入计数增量
            self.send_insert_count_increment(1)
    
    def check_blocked_streams(self):
        """检查并解除阻塞的流"""
        for stream_id, (encoded, required_count) in list(self.blocked_streams.items()):
            if required_count <= self.dynamic_table.insertion_count:
                # 可以解码了
                headers = self.decode_headers(stream_id, encoded)
                del self.blocked_streams[stream_id]
                # 通知应用层
                self.on_headers_ready(stream_id, headers)
```

---

## 八、QPACK 的优化技巧

### 8.1 动态表容量的选择

**小动态表（4-16KB）**：
- 优点：内存占用小，适合移动设备
- 缺点：命中率低，压缩效果一般

**大动态表（32-64KB）**：
- 优点：命中率高，压缩效果好
- 缺点：内存占用大

**推荐策略**：
```python
def choose_table_capacity(device_type, connection_rtt):
    """根据设备和网络选择动态表容量"""
    if device_type == 'mobile':
        return 16 * 1024  # 16KB
    elif connection_rtt < 50:  # 低延迟
        return 64 * 1024  # 64KB（高命中率价值高）
    else:  # 高延迟
        return 32 * 1024  # 32KB（平衡）
```

### 8.2 阻塞流数量的选择

```python
def choose_max_blocked_streams(network_quality):
    """根据网络质量选择最大阻塞流数"""
    if network_quality == 'excellent':  # <1% 丢包率
        return 100  # 激进使用动态表
    elif network_quality == 'good':  # 1-3% 丢包率
        return 50   # 适度使用
    else:  # >3% 丢包率
        return 10   # 保守使用
```

### 8.3 选择性索引

**不是所有头部都值得索引**：

```python
def should_index(name, value):
    """决定是否将字段添加到动态表"""
    # 不索引的情况
    if name.startswith(b':'):  # 伪头部（通常在静态表）
        return False
    
    if name == b'content-length':  # 每次都不同
        return False
    
    if len(value) > 1024:  # 值太大（如大 cookie）
        return False
    
    # 索引的情况
    if name in [b'cookie', b'authorization', b'user-agent']:
        return True  # 常见且重复的头部
    
    return True  # 默认索引
```

---

## 九、本章总结

### 9.1 核心要点

1. **HPACK 的问题**：
   - 假设顺序传输
   - 在 QUIC 的乱序环境中导致队头阻塞
   - 动态表更新和引用之间的依赖

2. **QPACK 的创新**：
   - 解耦动态表更新（编码器流）和头部编码（请求流）
   - 引入"阻塞流"概念，允许控制风险
   - 显式确认机制，编码器知道哪些条目是安全的

3. **QPACK 的架构**：
   - 编码器流：发送动态表更新
   - 解码器流：发送确认和取消
   - 头部块：可以选择引用或不引用动态表

4. **权衡**：
   - 高效模式：高压缩率，但可能阻塞
   - 安全模式：无阻塞，但压缩率较低
   - 混合策略：根据网络条件动态选择

5. **性能**：
   - 压缩率与 HPACK 相当（高效模式）
   - 消除了 QUIC 环境中的队头阻塞
   - 页面加载时间改善 10-15%

### 9.2 QPACK vs. HPACK 总结

| 维度 | HPACK | QPACK | 优势方 |
|-----|-------|-------|-------|
| **压缩率** | 93-97% | 93-97%（高效模式）| 相同 |
| **队头阻塞** | 有（在 QUIC 上）| 无（或可控）| **QPACK** ⭐⭐⭐ |
| **实现复杂度** | 简单 | 中等 | **HPACK** |
| **内存占用** | 低 | 稍高（+10KB）| **HPACK** |
| **灵活性** | 固定策略 | 可配置（安全/高效）| **QPACK** ⭐ |
| **适用场景** | TCP（顺序传输）| UDP/QUIC（乱序传输）| 各有所长 |

### 9.3 最佳实践

1. **初始连接**：使用安全模式（不引用动态表）
2. **稳定连接**：切换到高效模式（激进使用动态表）
3. **高丢包网络**：降低 `QPACK_BLOCKED_STREAMS`
4. **低延迟网络**：增加 `QPACK_MAX_TABLE_CAPACITY`
5. **选择性索引**：不是所有头部都值得索引

### 9.4 展望

在下一章中，我们将探讨 HTTP/3 的 **服务器推送** 机制。虽然在前面的章节中已有提及，下一章将深入其优化策略、实际应用和性能影响。

---

## 参考资料

- RFC 9204: QPACK: Field Compression for HTTP/3
- RFC 7541: HPACK: Header Compression for HTTP/2
- "QPACK: Header Compression for HTTP/3" (IETF Draft)
- "Understanding QPACK" by Robin Marx
- Chromium QPACK 实现源码
- "HTTP/3 Explained" by Daniel Stenberg
