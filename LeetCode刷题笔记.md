# LeetCode 刷题笔记

> 整理时间：2026-10-24　|　语言：C++　|　共 16 题（简单 8 · 中等 8）

---

## 目录

| 题号 | 题目 | 难度 | 核心方法 |
|---|---|---|---|
| 1 | 两数之和 | 简单 | 哈希表 |
| 2 | 两数相加 | 中等 | 链表模拟竖式 |
| 3 | 无重复字符的最长子串 | 中等 | 滑动窗口 |
| 9 | 回文数 | 简单 | 反转一半数字 |
| 12 | 整数转罗马数字 | 中等 | 贪心 + 值表 |
| 27 | 移除元素 | 简单 | 快慢指针 |
| 54 | 螺旋矩阵 | 中等 | 按层模拟 |
| 56 | 合并区间 | 中等 | 排序 + 合并 |
| 59 | 螺旋矩阵 II | 中等 | 按层模拟（填充） |
| 209 | 长度最小的子数组 | 中等 | 滑动窗口 |
| 217 | 存在重复元素 | 简单 | 哈希集合 |
| 704 | 二分查找 | 简单 | 二分 |
| 2409 | 统计共同度过的日子数 | 简单 | 日期转序号 + 区间交集 |
| 3644 | 排序排列 | 中等 | 按位与性质 |
| 3659 | 数组元素分组 | 中等 | 计数 + 抽屉原理 |
| LCR 146 | 螺旋遍历二维数组 | 简单 | 同 54 |

---

## 1. 两数之和

**难度**：简单

**题目**：给定整数数组 `nums` 和整数目标值 `target`，在数组中找出和为 `target` 的两个整数，返回它们的下标（每种输入只对应一个答案，同一元素不能重复使用）。

**思路**：一次遍历，哈希表存「**已经见过的数 → 下标**」。对当前 `nums[i]`，直接查表里有没有 `target - nums[i]`：有就返回，没有就把 `nums[i]` 存进去。

关键点：**边遍历边存**，不要先全部存完再找，否则同一个元素可能配对到自己。

**代码**：

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> seen;          // 值 -> 下标
        for (int i = 0; i < nums.size(); ++i) {
            auto it = seen.find(target - nums[i]);
            if (it != seen.end()) return {it->second, i};
            seen[nums[i]] = i;
        }
        return {};
    }
};
```

**代码讲解**：

1. `unordered_map<int, int> seen;` —— 键是**数值**，值是**下标**。之所以要存下标而不是只存"出现过"的标记，是因为最后要返回下标。
2. `for (int i = 0; i < nums.size(); ++i)` —— 一次遍历，`i` 既是当前元素下标，也是答案里的第二个下标。
3. `auto it = seen.find(target - nums[i]);` —— 不查 `nums[i]`，查的是 `target - nums[i]`，也就是"**还差多少**"。如果这个差值之前出现过，说明配对成功。
4. `if (it != seen.end()) return {it->second, i};` —— `it->second` 是差值那头的下标（更靠前），`i` 是当前下标。返回顺序按题目要求升序。
5. `seen[nums[i]] = i;` —— 这行**必须在 if 之后**。如果放在前面，遇到 `target = 8`、`nums[i] = 4` 时，会拿 4 和自己配对（自己和自己用了两次）。
6. `return {};` —— 题目保证有答案，这里是兜底，让编译器不报"缺少返回"。
7. 整体 O(n)：`unordered_map` 的 `find` 和插入平均都是 O(1)。

**复杂度**：时间 O(n)，空间 O(n)

**易错点**：先 `find` 再 `insert`，顺序不能反。

---

## 2. 两数相加

**难度**：中等

**题目**：两个非空链表，各自表示一个非负整数（**逆序存储**，即头结点是个位）。把两数相加，用同样的链表形式返回和。

**思路**：小学竖式加法，从个位开始逐位相加并记录进位。因为是逆序存储，链表从头往后走正好是"个位 → 十位 → 百位"，天然对齐。

**代码**：

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode dummy(0), *cur = &dummy;
        int carry = 0;
        while (l1 || l2 || carry) {
            int sum = carry;
            if (l1) { sum += l1->val; l1 = l1->next; }
            if (l2) { sum += l2->val; l2 = l2->next; }
            carry = sum / 10;
            cur->next = new ListNode(sum % 10);
            cur = cur->next;
        }
        return dummy.next;
    }
};
```

**代码讲解**：

1. `ListNode dummy(0), *cur = &dummy;` —— 虚头结点。没有它的话，每加一个结点都要判断"是不是第一个结点"，代码会多一层 if。有个假头就统一了：所有结点都往 `cur->next` 挂。
2. `while (l1 || l2 || carry)` —— **三个条件缺一不可**：
   - `l1`：第一条链表还没走完
   - `l2`：第二条链表还没走完
   - `carry`：两条都走完了但还有进位（如 `5 + 5 = 10`，最后要补一个结点）
3. `int sum = carry;` —— 先把上一位的进位数放进来，再加上两个当前位。
4. `if (l1) { sum += l1->val; l1 = l1->next; }` —— 用 `if` 而不是直接访问，因为两条链表**可能不等长**，短的先变成 `nullptr`，直接访问会崩溃。
5. `carry = sum / 10;` —— 整数除法，`sum = 13` 得 `1`，`sum = 7` 得 `0`。
6. `new ListNode(sum % 10)` —— 取个位，`13 % 10 = 3`。
7. `cur = cur->next;` —— 指针前移，准备挂下一个结点。
8. `return dummy.next;` —— 返回真正的头结点，跳过那个假的。

**复杂度**：时间 O(max(m, n))，空间 O(1)（除结果链表外）

**易错点**：
- `while` 漏掉 `carry` 会让 `[5] + [5]` 只返回 `[0]`
- 不要 `free` 掉 `l1/l2` 的原结点，题目没要求，释放反而容易出错

---

## 3. 无重复字符的最长子串

**难度**：中等

**题目**：给定字符串 `s`，找出其中不含重复字符的**最长子串**的长度。

**思路**：滑动窗口。右指针 `r` 逐个扩张，用数组记录每个字符**上次出现的位置**。如果上次出现在窗口内，就把左边界 `l` 跳到它右边一格。

**代码**：

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        vector<int> last(128, -1);       // ASCII 字符上次出现的下标
        int ans = 0, l = 0;
        for (int r = 0; r < (int)s.size(); ++r) {
            if (last[s[r]] >= l) l = last[s[r]] + 1;   // 冲突，收缩左边界
            last[s[r]] = r;
            ans = max(ans, r - l + 1);
        }
        return ans;
    }
};
```

**代码讲解**：

1. `vector<int> last(128, -1);` —— 128 是 ASCII 字符总数，下标直接当字符用（`s[r]` 自动转成整数）。初值 `-1` 表示"从没出现过"。
2. `int ans = 0, l = 0;` —— `ans` 记最大长度，`l` 是窗口左边界。窗口就是 `[l, r]`。
3. `if (last[s[r]] >= l)` —— **整个算法最关键的一行**。为什么是 `>= l` 而不是 `!= -1`？因为字符上次出现的位置可能已经在窗口**左边之外**（`< l`），那就不算冲突，窗口不用收缩。
4. `l = last[s[r]] + 1;` —— 把左边界跳到重复字符的**下一位**，这样窗口内就只剩一个 `s[r]`。注意 `l` 只增不减，不会回头。
5. `last[s[r]] = r;` —— 刷新位置。这行**必须在 if 之后**，否则下一步判断会用到错误的位置。
6. `ans = max(ans, r - l + 1);` —— 当前窗口长度是 `r - l + 1`（闭区间）。
7. 复杂度 O(n)：`r` 走 n 步，`l` 最多也走 n 步，总共 2n。

**复杂度**：时间 O(n)，空间 O(1)（128 个字符的固定数组）

**易错点**：
- 判断条件写成 `last[c] != -1` 会出错：字符以前出现过但在窗口外，不该收缩
- `vector<int>(128, -1)` 只覆盖 ASCII，输入含中文（UTF-8 多字节）时要换 `unordered_map<char,int>`

---

## 9. 回文数

**难度**：简单

**题目**：判断一个整数是否是回文数（正着读和倒着读一样）。例：`121` 是，`123` 不是，`-121` 不是，`10` 不是。

**思路**：**只反转后一半数字**，和前一半比较。整体反转会溢出（`2147483647` → `7463847412` 超出 int）。

**代码**：

```cpp
class Solution {
public:
    bool isPalindrome(int x) {
        if (x < 0 || (x % 10 == 0 && x != 0)) return false;

        int rev = 0;
        while (x > rev) {                 // rev 追上 x 就停，说明处理了一半
            rev = rev * 10 + x % 10;
            x /= 10;
        }
        return x == rev || x == rev / 10;
    }
};
```

**代码讲解**：

1. `if (x < 0 || ...) return false;` —— 负数一定不是，因为多了个负号，反过来是 `121-`。
2. `x % 10 == 0 && x != 0` —— 末位是 0 的正数一定不是（如 `10` 反过来是 `01`）。但 `x == 0` 要放行，`0` 本身是回文。两个条件必须**同时**成立才返回 false。
3. `while (x > rev)` —— 这个循环条件本身就是"走到一半"的判据，**不需要提前算总位数**。`x` 每次除 10 变小，`rev` 每次乘 10 变大，两者相遇就说明取了半个数。
4. `rev = rev * 10 + x % 10;` —— `rev * 10` 把已有数字整体左移一位腾出个位，`x % 10` 取出末位塞进去。
5. `x /= 10;` —— 剥掉刚取走的末位。**这行不能省**，否则 `x % 10` 永远是同一个数字，死循环。
6. `x == rev || x == rev / 10` —— 分两种情况：
   - `x == rev`：总位数是**偶数**，正好对半分（如 `1221` → `12` 和 `12`）
   - `x == rev / 10`：总位数是**奇数**，`rev` 多了一位，那一位就是正中间的数字（如 `121` → `x=1`、`rev=12`），砍掉它再比
7. 为什么必须先试 `x == rev`？如果只写 `rev / 10`，偶数位的 `1221` 会变成 `12 == 1` 而误判 false。

**复杂度**：时间 O(log₁₀n)，空间 O(1)

**易错点**：
- 中间那一位不用比，因为它左右对称，永远相等
- `x == 0` 必须返回 `true`，别被第二条排除规则误杀

---

## 12. 整数转罗马数字

**难度**：中等

**题目**：把整数转成罗马数字。符号值：I=1, V=5, X=10, L=50, C=100, D=500, M=1000。减法形式只有 6 种：IV(4)、IX(9)、XL(40)、XC(90)、CD(400)、CM(900)。

**思路**：贪心。把 13 种"面值"按从大到小排好，能减就一直减，每减一次拼一个符号。

**代码**：

```cpp
class Solution {
public:
    string intToRoman(int num) {
        static const vector<pair<int, string>> mp = {
            {1000,"M"}, {900,"CM"}, {500,"D"}, {400,"CD"},
            {100, "C"}, {90, "XC"}, {50, "L"}, {40, "XL"},
            {10,  "X"}, {9,  "IX"}, {5,  "V"}, {4,  "IV"}, {1, "I"}
        };

        string r;
        r.reserve(15);                    // 最长 15 字符：3888 = MMMDCCCLXXXVIII
        for (const auto &p : mp) {
            while (num >= p.first) {
                r += p.second;
                num -= p.first;
            }
        }
        return r;
    }
};
```

**代码讲解**：

1. `static const vector<pair<int, string>> mp` —— 三件事：
   - `pair<int,string>` 把"值"和"符号"绑在一起，遍历时不用维护两个数组的下标对应
   - `static` 让它**只构造一次**，避免每次调用都重新创建 13 个 string
   - `const` 防止手滑修改
2. 值表**必须严格降序**，这是贪心成立的前提。并且必须包含 `900/400/90/40/9/4` 这 6 个减法形式，否则 `4` 会被拼成 `IIII`（非法）。
3. `for (const auto &p : mp)` —— `const` 表示不改 `p`，`&` 用引用避免拷贝整个 pair（内含 string）。
4. `while (num >= p.first)` —— **`while` 不是 `if`**。同一个符号可能连续拼多次：`3000 = MMM`、`30 = XXX`。
5. `r += p.second;` —— 拼上符号。
6. `num -= p.first;` —— 扣掉对应的值，剩下的继续用后面更小的面值凑。
7. `return r;` —— 循环结束时 `num` 必然是 0，因为最后一项是 1，任何正整数都能被减到 0。
8. 复杂度 O(1)：`num ≤ 3999`，循环次数有上界（最多几十次），不随输入规模增长。

**复杂度**：时间 O(1)，空间 O(1)

**易错点**：
- 漏掉任意一个减法形式，都会得到非法罗马数字
- `reserve(15)` 是可选的性能优化，不影响正确性

**举例**（1994）：

| num | 匹配值 | 拼上 | 剩余 |
|---|---|---|---|
| 1994 | 1000 (M) | M | 994 |
| 994 | 900 (CM) | MCM | 94 |
| 94 | 90 (XC) | MCMXC | 4 |
| 4 | 4 (IV) | MCMXCIV | 0 |

---

## 27. 移除元素

**难度**：简单

**题目**：给定数组 `nums` 和值 `val`，原地删除所有等于 `val` 的元素，返回新长度。元素顺序可以改变，不需要考虑超出新长度后面的元素。

**思路**：快慢指针。`k` 指向"下一个要放的位置"，遍历时遇到不等于 `val` 的就往前挪。

**代码**：

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int k = 0;
        for (int x : nums) {
            if (x != val) nums[k++] = x;
        }
        return k;
    }
};
```

**代码讲解**：

1. `int k = 0;` —— 既是"新数组的长度"，也是"下一个要写的位置"。这两个含义在这里是重合的，因为保留下来的元素从下标 0 开始紧挨着放。
2. `for (int x : nums)` —— 范围 for，读的是值。这里读 `int` 拷贝无所谓；如果元素是大对象，应该写 `const auto&`。
3. `if (x != val)` —— 只处理要保留的元素，等于 `val` 的直接跳过（相当于"删除"）。
4. `nums[k++] = x;` —— 把保留下来的元素**往前搬**。`k++` 是后自增：先用当前 `k` 作下标写入，再把 `k` 加 1。
5. `return k;` —— 循环结束时 `k` 正好是保留下来的元素个数。
6. 为什么这样不会覆盖还没读的元素？因为 `k` 一定 `<=` 当前遍历位置（`k` 只在保留时才增），写的位置永远在或等于读的位置。

**复杂度**：时间 O(n)，空间 O(1)

**易错点**：
- 必须**原地**修改，不能新建数组返回
- 返回的是新长度 `k`，不是删掉的个数

---

## 54. 螺旋矩阵

**难度**：中等

**题目**：给定 `m × n` 矩阵，按螺旋顺序（顺时针，从左上开始）返回所有元素。

**思路**：按层模拟。维护四个边界 `top / bottom / left / right`，按「上边 → 右边 → 下边 → 左边」走一圈，边界各缩一格。

**代码**：

```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        vector<int> res;
        if (matrix.empty() || matrix[0].empty()) return res;

        int top = 0, bottom = matrix.size() - 1;
        int left = 0, right = matrix[0].size() - 1;

        while (top <= bottom && left <= right) {
            for (int j = left; j <= right; ++j) res.push_back(matrix[top][j]);
            ++top;

            for (int i = top; i <= bottom; ++i) res.push_back(matrix[i][right]);
            --right;

            if (top <= bottom) {
                for (int j = right; j >= left; --j) res.push_back(matrix[bottom][j]);
                --bottom;
            }
            if (left <= right) {
                for (int i = bottom; i >= top; --i) res.push_back(matrix[i][left]);
                ++left;
            }
        }
        return res;
    }
};
```

**代码讲解**：

1. `if (matrix.empty() || matrix[0].empty()) return res;` —— 防空。两个都要判：外层空是"没有行"，`matrix[0]` 空是"有行但没列"。
2. `int bottom = matrix.size() - 1;` 是**行数**减一，`int right = matrix[0].size() - 1;` 是**列数**减一。行和列是两个不同的维度，别搞混。
3. `while (top <= bottom && left <= right)` —— 两个条件都要满足，说明"还剩至少一行且至少一列"。
4. 第一段 `for (int j = left; j <= right; ++j)` —— 走上边，横坐标固定 `top`，纵坐标从 `left` 到 `right`。走完 `++top`，因为这一行以后不再需要。
5. 第二段 `for (int i = top; ...)` —— **起点是 `top` 而不是 `top+1`**。因为 `top` 刚刚已经自增过了，现在的 `top` 就是"下一行"。
6. `if (top <= bottom)` —— 关键判断。如果刚走完上边和右边，剩下的区域已经没有行了（单行矩阵的情况），就不能再走"下边"，否则重复添加。
7. 第三段走下边（从右到左），走完 `--bottom`；第四段走左边（从下到上），走完 `++left`。左边那段同样有 `if (left <= right)` 保护。
8. 每轮四个边界各缩一格，直到交叉为止。

**复杂度**：时间 O(m·n)，空间 O(1)（除结果数组外）

**易错点**：两个 `if` 不能省。例 `[[1,2,3]]` 只有一行，省掉会把元素读两遍。

---

## 56. 合并区间

**难度**：中等

**题目**：给定若干区间 `intervals`，合并所有重叠的区间，返回互不重叠的区间集合。

**思路**：先按左端点排序，再逐个与结果集最后一个区间比较：不重叠就加入，重叠就把右端扩到最大。

**代码**：

```cpp
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end());   // vector 默认按字典序，等价于按左端排序
        vector<vector<int>> res;
        for (const auto &in : intervals) {
            if (res.empty() || res.back()[1] < in[0]) {
                res.push_back(in);                  // 不重叠，开新区间
            } else {
                res.back()[1] = max(res.back()[1], in[1]);   // 重叠，扩展右端
            }
        }
        return res;
    }
};
```

**代码讲解**：

1. `sort(intervals.begin(), intervals.end());` —— `vector<vector<int>>` 的默认比较是**字典序**：先比第一个元素，相同再比第二个。因为区间不重复且有序，这个效果正好等于"按左端点升序"。想显式写清楚可以用 lambda，但没必要。
2. `res.empty()` —— 第一个区间没有"前一个"可比，直接放进去。这个判断必须写在 `||` 前面，因为 `res.back()` 在空 vector 上调用是未定义行为。
3. `res.back()[1] < in[0]` —— 用**严格小于**判断"不重叠"。`[1,2]` 和 `[2,3]` 在本题算重叠（端点相接），所以要合并，只有 `2 < 2` 为假才走合并分支——正好符合。
4. `res.push_back(in)` —— 不重叠，作为一个新区间放进去。
5. `max(res.back()[1], in[1])` —— **必须取 max**。例 `[[1,10],[2,3]]`，如果直接 `= in[1]`，右端会从 10 缩成 3，丢了区间。
6. 为什么只跟"最后一个"比就够？因为排序后所有区间按左端递增，前面处理过的区间右端点一定不比当前的小（已被合并拉大），所以当前区间只可能和最后一个有交集。

**复杂度**：时间 O(n log n)（排序），空间 O(log n)（排序栈）或 O(n)（结果）

**易错点**：
- 判断重叠要用 `<`，不是 `<=`
- 扩展右端要 `max`，不能直接赋值

---

## 59. 螺旋矩阵 II

**难度**：中等

**题目**：给定正整数 `n`，生成一个 `n × n` 矩阵，元素 `1 ~ n²` 按顺时针螺旋顺序填充。

**思路**：和 54 完全一样，只是从"读"变成"写"，用计数器依次填入。

**代码**：

```cpp
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> m(n, vector<int>(n));
        int top = 0, bottom = n - 1, left = 0, right = n - 1, num = 1;

        while (top <= bottom && left <= right) {
            for (int j = left; j <= right; ++j) m[top][j] = num++;
            ++top;
            for (int i = top; i <= bottom; ++i) m[i][right] = num++;
            --right;
            if (top <= bottom) {
                for (int j = right; j >= left; --j) m[bottom][j] = num++;
                --bottom;
            }
            if (left <= right) {
                for (int i = bottom; i >= top; --i) m[i][left] = num++;
                ++left;
            }
        }
        return m;
    }
};
```

**代码讲解**：

1. `vector<vector<int>> m(n, vector<int>(n));` —— 创建 `n` 行、每行 `n` 个 0。注意**不能**写 `vector<vector<int>> m(n, n)`，那样第二维会变成一个装 int 的 vector（只有一个元素 n）。
2. `num = 1` —— 要填入的数字，从 1 开始。
3. `m[top][j] = num++;` —— `num++` 后自增：先用当前的 `num` 写入，再 `num` 加 1。所以填的是 1, 2, 3… 顺序正确。
4. 四段循环的边界和 54 完全对应：上边 → 右边 → 下边 → 左边，每段走完就收缩对应的边界。
5. `if (top <= bottom)` 和 `if (left <= right)` —— 同样不能省。虽然 `n × n` 时最后只剩中心一格，但如果省掉判断会把中心重复填。
6. 因为是方阵，`bottom` 和 `right` 初值都是 `n - 1`，可以直接合成一个变量，但分开写更清晰、也好迁移到非方阵。

**复杂度**：时间 O(n²)，空间 O(1)（除结果矩阵外）

**易错点**：`vector<vector<int>> m(n, vector<int>(n))` 这个初始化语法必须写对。

---

## 209. 长度最小的子数组

**难度**：中等

**题目**：给定正整数数组 `nums` 和目标值 `target`，找出和 ≥ `target` 的**长度最小**的连续子数组，返回其长度；不存在则返回 0。

**思路**：滑动窗口。右指针扩张累加，一旦 `sum >= target` 就尝试收缩左边界，并记录最小长度。

**代码**：

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int ans = INT_MAX, l = 0, sum = 0;
        for (int r = 0; r < (int)nums.size(); ++r) {
            sum += nums[r];
            while (sum >= target) {
                ans = min(ans, r - l + 1);
                sum -= nums[l++];
            }
        }
        return ans == INT_MAX ? 0 : ans;
    }
};
```

**代码讲解**：

1. `int ans = INT_MAX` —— 用最大整数当"还没找到"的哨兵。因为正数数组的和有上界，用 INT_MAX 不会被真实答案撞上。
2. `sum += nums[r];` —— 右指针扩张，把新元素纳入窗口。
3. `while (sum >= target)` —— **必须是 `while` 不是 `if`**。收缩一次后可能仍然 ≥ target，那就继续缩，直到不满足为止，这样才能找到"最短"。
4. `ans = min(ans, r - l + 1);` —— 这一步**要在收缩之前**做。此时窗口正好满足条件，`r - l + 1` 就是当前合法窗口长度。
5. `sum -= nums[l++];` —— 把左端元素吐出去，左边界右移一格，尝试更短的窗口。
6. 为什么左边界不用回头？因为元素全是**正整数**，窗口越长和越大、越短和越小，单调性保证了"缩过的左边界不需要再往左"。如果数组里允许负数，这个写法就失效了。
7. `return ans == INT_MAX ? 0 : ans;` —— 一次都没满足过条件时，`ans` 还是原始哨兵，要转成 0。

**复杂度**：时间 O(n)（每个元素最多进出窗口各一次），空间 O(1)

**易错点**：
- 内层 `while` 写成 `if` 会漏掉"能缩得更短"的情况
- 忘记处理 `INT_MAX` 会返回 `2147483647`

---

## 217. 存在重复元素

**难度**：简单

**题目**：给定整数数组，如果任一值在数组中出现至少两次返回 `true`，否则 `false`。

**思路**：哈希集合，逐个插入，插入失败说明已存在。

**代码**：

```cpp
class Solution {
public:
    bool containsDuplicate(vector<int>& nums) {
        unordered_set<int> st;
        for (int x : nums) {
            if (!st.insert(x).second) return true;   // 插入失败 = 已存在
        }
        return false;
    }
};
```

**代码讲解**：

1. `unordered_set<int> st;` —— 只需要知道"出现过没有"，不需要下标，所以用 `set` 而不是 `map`。
2. `st.insert(x)` —— 返回值是 `pair<iterator, bool>`。`bool` 为 `true` 表示**插入成功**（原来没有），为 `false` 表示**已存在**。
3. `!st.insert(x).second` —— 取反：插入失败（已存在）就返回 true。这一行同时完成了"插入"和"判断"，省掉先 `find` 再 `insert` 的两次查找。
4. `return false;` —— 全部插入成功，说明没有重复。

**复杂度**：时间 O(n)，空间 O(n)

**替代写法**（更省空间，但会修改原数组）：

```cpp
sort(nums.begin(), nums.end());
for (int i = 1; i < nums.size(); ++i)
    if (nums[i] == nums[i - 1]) return true;
return false;
// 时间 O(n log n)，空间 O(1)
```

**易错点**：`insert` 返回的是 `pair`，别忘了 `.second`。

---

## 704. 二分查找

**难度**：简单

**题目**：给定**升序**数组 `nums` 和目标值 `target`，搜索目标值并返回下标，不存在返回 `-1`。

**思路**：标准二分。区间取闭区间 `[l, r]`。

**代码**：

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int l = 0, r = (int)nums.size() - 1;
        while (l <= r) {
            int mid = l + (r - l) / 2;        // 防止 l + r 溢出
            if (nums[mid] == target) return mid;
            if (nums[mid] < target) l = mid + 1;
            else                    r = mid - 1;
        }
        return -1;
    }
};
```

**代码讲解**：

1. `int l = 0, r = (int)nums.size() - 1;` —— 闭区间 `[l, r]`，`r` 指向最后一个元素的下标，不是长度。
2. `while (l <= r)` —— 闭区间的循环条件用 `<=`，因为 `l == r` 时区间里还有一个元素要检查。
3. `int mid = l + (r - l) / 2;` —— 等价于 `(l + r) / 2`，但避免了 `l + r` 爆 `int`（本题数据不大，属于通用习惯）。注意先减后加，不能写成 `l + (r - l + 1) / 2` 弄错方向。
4. `if (nums[mid] == target) return mid;` —— 命中直接返回。
5. `l = mid + 1` —— 目标在右半边，`mid` 已经排除，所以从 `mid + 1` 开始。
6. `r = mid - 1` —— 目标在左半边，同理。
7. 循环结束还没返回，说明区间空了，目标不存在。

**复杂度**：时间 O(log n)，空间 O(1)

**易错点**：
- 闭区间配 `l <= r`，左闭右开 `[l, r)` 配 `l < r` 且 `r = mid`。**两种写法不能混用**，混了会死循环或漏元素
- `r = mid` 而不是 `mid - 1` 是左闭右开的写法，别在闭区间里用

**对比：左闭右开写法**

```cpp
int l = 0, r = nums.size();          // r 是越界位
while (l < r) {
    int mid = l + (r - l) / 2;
    if (nums[mid] == target) return mid;
    if (nums[mid] < target) l = mid + 1;
    else                    r = mid;  // 注意不是 mid - 1
}
```

---

## 2409. 统计共同度过的日子数

**难度**：简单

**题目**：Alice 和 Bob 分别去罗马，给定各自的到达和离开日期（格式 `"MM-DD"`，闭区间，同一年、非闰年），返回两人同时在罗马的天数。

**思路**：把日期转成"一年中的第几天"，然后求区间交集。

**代码**：

```cpp
class Solution {
public:
    int countDaysTogether(string arriveAlice, string leaveAlice, string arriveBob, string leaveBob) {
        int md[12] = {31,28,31,30,31,30,31,31,30,31,30,31};
        int pre[13] = {0};                         // pre[i] = 前 i 个月的总天数
        for (int i = 0; i < 12; ++i)
            pre[i + 1] = pre[i] + md[i];

        auto toDay = [&](const string& s) {        // "MM-DD" -> 一年中第几天
            int m = stoi(s.substr(0, 2));
            int d = stoi(s.substr(3, 2));
            return pre[m - 1] + d;
        };

        int l = max(toDay(arriveAlice), toDay(arriveBob));
        int r = min(toDay(leaveAlice),  toDay(leaveBob));
        return max(0, r - l + 1);
    }
};
```

**代码讲解**：

1. `int md[12] = {31,28,...};` —— 每月天数。题目保证非闰年，所以 2 月固定 28，不用判闰年。
2. `pre[i + 1] = pre[i] + md[i];` —— 前缀和。填完后 `pre[0]=0`、`pre[1]=31`、`pre[2]=59`……规律是 **`pre[k]` = 前 k 个月的总天数**。
3. `auto toDay = [&](const string& s) {` —— lambda 定义局部小函数。`[&]` 是捕获列表，让函数体能用到外面的 `pre`（不写的话编译器报 `pre` 未定义）。`auto` 是因为 lambda 的类型没有名字。
4. `stoi(s.substr(0, 2))` —— 字符串格式是 `MM-DD`，下标 0、1 是月份，下标 2 是 `-`，下标 3、4 是日期。`substr(0, 2)` 取两位，`stoi` 转成整数（`"08"` → `8`，前导 0 自动忽略）。
5. `return pre[m - 1] + d;` —— **必须是 `m - 1`**。`pre[m-1]` 是"上个月底之前过了多少天"，加本月的第 `d` 天，就是全年第几天。写成 `pre[m]` 会把本月整月也算进去，多 30 天左右。
6. `int l = max(...)` —— 交集左端取两人的**较晚到达**时间。
7. `int r = min(...)` —— 交集右端取两人的**较早离开**时间。
8. `return max(0, r - l + 1);` —— 闭区间长度要 `+1`（8-16 到 8-18 是 3 天，`18-16=2`）。`max(0, ...)` 防止两人完全没重叠时出现负天数。

**复杂度**：时间 O(1)，空间 O(1)

**易错点**：
- `pre[m-1]` 写成 `pre[m]` 是多算整月的经典错误
- `+1` 和 `max(0, ...)` 两个都要有

**举例**：

```
Alice: 8-15 → 227    8-18 → 230
Bob:   8-16 → 228    8-19 → 231
左端 = max(227,228) = 228
右端 = min(230,231) = 230
天数 = 230 - 228 + 1 = 3
```

---

## 3644. 排序排列

**难度**：中等

**题目**：`nums` 是 `[0, n-1]` 的一个排列。可以在 `nums[i] AND nums[j] == k` 时交换下标 `i`、`j` 的元素。求能使数组升序排序的**最大 k**；已有序返回 0。

**思路**：**答案 = 所有「错位元素」的按位与**（`nums[i] != i` 的那些 `nums[i]`）。

**为什么**：排序目标固定是 `nums[i] = i`（因为是排列）。设错位集合为 `M`，`g = AND(M)`。

- **k 不可能比 g 大**：k 的某一位要等于 1，就必须参与交换的两个数在这一位都是 1。所以 k 的每一位，每个错位元素都必须有。
- **k = g 一定能排**：`g` 本身必然在数组里（`g ≤ 每个错位元素 < n`），且 `g` 和任何错位元素 `x` 都满足 `g & x == g`——它是"万能中转站"，可以当华容道里的空格用。

**代码**：

```cpp
class Solution {
public:
    int sortPermutation(vector<int>& nums) {
        int g = -1;                                  // -1 二进制全 1，与任何数 AND 等于它自己
        for (int i = 0; i < (int)nums.size(); ++i)
            if (nums[i] != i) g &= nums[i];           // 只累计错位元素
        return g < 0 ? 0 : g;                        // 仍为 -1 说明已经有序
    }
};
```

**代码讲解**：

1. `int g = -1;` —— `-1` 的补码是 32 位**全 1**，跟任何数做 AND 都原样保留（`-1 & 5 = 5`）。拿它当初值，省掉"第一个错位元素直接赋值"的特判。
2. `for (int i = 0; i < nums.size(); ++i)` —— 遍历所有下标。注意这里比较的是**值和下标**，不是相邻元素。
3. `if (nums[i] != i)` —— 只处理**错位**的元素。已经在正确位置的不能算进去：它们不需要移动，硬算会让 k 无谓地变小（比如已就位的 6 会把 k 的某些位抹掉）。
4. `g &= nums[i]` —— 等价于 `g = g & nums[i]`，即"把 `g` 里、`nums[i]` 中没有的 1 位全部清掉"。循环下来 `g` 就是所有错位元素的公共 1 位，也就是按位与。
5. `return g < 0 ? 0 : g;` —— 如果一次都没进过 `if`（数组本来就有序），`g` 还是 `-1`，按题目要求返回 0。
6. **代码里没有任何 `swap`**——题目只要求返回 k，不要求输出交换步骤，交换的可行性由上面的证明保证。

**复杂度**：时间 O(n)，空间 O(1)

**易错点**：
- 已就位的元素不能参与按位与
- `g` 初值用 `-1` 而不是 `0`（用 0 的话与出来永远是 0）

**举例**（`nums = [0,1,2,3,4,7,6,5]`）：

```
错位元素：7（在位置5）、5（在位置7）
g = -1 & 7 & 5 = 111 & 101 = 101 = 5
验证：7 & 5 = 5 == k，交换位置5和7 → [0,1,2,3,4,5,6,7]
k 取 6 或 7 都换不动（7&5 恒等于 5），所以 5 是最大
```

---

## 3659. 数组元素分组

**难度**：中等

**题目**：给定数组 `nums` 和整数 `k`，判断能否把所有元素分成若干组，满足：每组恰好 `k` 个元素、组内元素互不相同、每个元素恰好用一次。

**思路**：两个条件——`n` 必须能被 `k` 整除；任一元素出现次数不能超过组数。

**代码**：

```cpp
class Solution {
public:
    bool partitionArray(vector<int>& nums, int k) {
        int n = nums.size();
        if (n % k) return false;              // 分不出整组

        int m = n / k;                        // 组数
        unordered_map<int, int> cnt;
        for (int x : nums) {
            if (++cnt[x] > m) return false;   // 出现次数超过组数
        }
        return true;
    }
};
```

**代码讲解**：

1. `if (n % k) return false;` —— 每组固定 `k` 个，总数不是 `k` 的倍数就分不出整组。`n % k` 非 0 即为真，所以可以省略 `!= 0`。
2. `int m = n / k;` —— 一共能分成 `m` 组。这个数是后面判断的核心。
3. `++cnt[x] > m` —— 两个动作合一：先把 `x` 的计数加 1，再和组数 `m` 比较。**边遍历边判断**，一旦超了立刻返回，不用等统计完。
4. 为什么「出现次数 ≤ m」是充分条件？因为每个元素在每组里最多出现 1 次（组内互不相同），一共 `m` 组，所以最多出现 `m` 次。反过来，只要所有元素都不超过 `m` 次，按次数从多到少依次往各组填，一定能填满（抽屉原理）。
5. `return true;` —— 走到这里说明所有元素都合格。

**复杂度**：时间 O(n)，空间 O(n)

**易错点**：
- 先判 `n % k` 再算组数，`n < k` 时也不会出错
- 题面中文"每组 k 个不同元素"容易理解成"k 种元素"，实际是"恰好 k 个，且这 k 个互不相同"

**举例**：

```
nums = [3,5,2,2], k = 2  → n=4, m=2
3:1 次, 5:1 次, 2:2 次 → 最大 2 ≤ m=2 → true
分组：[2,3] [2,5]

nums = [1,5,2,3], k = 3  → n=4, 4%3≠0 → false
```

---

## LCR 146. 螺旋遍历二维数组

**难度**：简单

**题目**：给定一个二维数组 `array`（可能是 `m × n` 的非方阵），按螺旋顺序返回所有元素。

**思路**：与 **54. 螺旋矩阵** 完全一致，只是输入可能是非方阵，行列边界要分开取。

**代码**：

```cpp
class Solution {
public:
    vector<int> spiralArray(vector<vector<int>>& array) {
        vector<int> res;
        if (array.empty() || array[0].empty()) return res;

        int top = 0, bottom = array.size() - 1;
        int left = 0, right = array[0].size() - 1;

        while (top <= bottom && left <= right) {
            for (int j = left; j <= right; ++j) res.push_back(array[top][j]);
            ++top;
            for (int i = top; i <= bottom; ++i) res.push_back(array[i][right]);
            --right;
            if (top <= bottom) {
                for (int j = right; j >= left; --j) res.push_back(array[bottom][j]);
                --bottom;
            }
            if (left <= right) {
                for (int i = bottom; i >= top; --i) res.push_back(array[i][left]);
                ++left;
            }
        }
        return res;
    }
};
```

**代码讲解**：

1. `array.size()` 是**行数**，`array[0].size()` 是**列数**。非方阵时这两个不相等，绝不能混用。
2. `if (array.empty() || array[0].empty())` —— 两重防空。`array.empty()` 为真时访问 `array[0]` 会越界，所以 `||` 的短路求值顺序很重要（空的那次直接返回，不会执行后半句）。
3. 其余四段与前文 54 完全一致：上边 → 右边 → 下边 → 左边，每段走完收缩边界，`if` 保护单行/单列的情况。
4. 如果输入是规则的 `m × n` 矩阵，这题和 54 就是同一道题，可以共用一份代码。

**复杂度**：时间 O(m·n)，空间 O(1)（除结果数组外）

**易错点**：用 `array.size()` 当列数是常见错误，方阵时看不出问题，非方阵立刻出错。

---

## 附录：几个通用套路速记

| 场景 | 套路 | 关键代码特征 |
|---|---|---|
| 找配对（和、差、次数） | 哈希表，边遍历边存 | `find(target - x)` 在 `insert` 之前 |
| 连续子数组 / 子串 | 滑动窗口（要有单调性） | 内层 `while` 收缩，不是 `if` |
| 区间重叠 | 按左端排序，与"最后一个"比 | `res.back()[1] < in[0]` 判不重叠 |
| 矩阵按层处理 | 四边界收缩 | 后两段要 `if` 保护 |
| 有序数组查找 | 二分 | 闭区间配 `<=`，`mid = l+(r-l)/2` |
| 原地删除 | 快慢指针 | `nums[k++] = x` |
| 链表构造 | 虚头结点 | `dummy` + 返回 `dummy.next` |
| 数字逐位处理 | `% 10` 取末位，`/ 10` 去掉 | 注意剥位那行不能省 |
| 日期转序号 | 月份天数前缀和 | `pre[m-1] + d` |

---

## 复习建议

按这个顺序过一遍，正好串起基础套路：

1. **704 二分查找** → **27 移除元素** → **217 存在重复元素** → **1 两数之和**（热身，建立"哈希 = 空间换时间"的直觉）
2. **209 长度最小的子数组** → **3 无重复字符的最长子串**（滑动窗口，数组和字符串各一遍）
3. **56 合并区间**（排序 + 区间套路）
4. **54 螺旋矩阵** → **59 螺旋矩阵 II**（读和写各一遍，边界判断是同一个坑）
5. **2 两数相加**（链表 + 进位）
6. **9 回文数** → **12 整数转罗马数字**（数字逐位处理）
7. **2409 统计共同度过的日子数**（日期 → 序号，前缀和）
8. **3659 数组元素分组**（计数 + 抽屉原理）
9. **3644 排序排列**（位运算 + 证明，最难，放最后）
