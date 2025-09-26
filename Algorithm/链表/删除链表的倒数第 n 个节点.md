
```
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* dummyHead = new ListNode(0);
        dummyHead->next = head;
        ListNode* slow = dummyHead;
        ListNode* fast = dummyHead;
        while(n-- && fast != NULL) {
            fast = fast->next;
        }
        fast = fast->next; // fast再提前走一步，因为需要让slow指向删除节点的上一个节点
        while (fast != NULL) {
            fast = fast->next;
            slow = slow->next;
        }
        slow->next = slow->next->next; 
        
        // ListNode *tmp = slow->next;  C++释放内存的逻辑
        // slow->next = tmp->next;
        // delete tmp;
        
        return dummyHead->next;
    }
};
```

### 核心思想

1.  **快慢指针 (Two Pointers)**: 要找到倒数第 `n` 个节点，一个直观的方法是先遍历一遍链表得到总长度 `L`，然后再从头遍历 `L-n` 步。但这样需要遍历两次。快慢指针法只需要一次遍历。我们设置一个 `fast` 指针和一个 `slow` 指针，让 `fast` 指针先走 `n` 步。然后，`fast` 和 `slow` 一起走。当 `fast` 走到链表末尾时，`slow` 指针正好指向我们需要的节点（或者它的前一个节点）。

2.  **虚拟头结点 (Dummy Node)**: 如果要删除的恰好是头结点（例如，删除一个长度为5的链表的倒数第5个节点），那么删除操作会比较特殊，需要改变 `head` 指针本身。为了统一所有情况（删除头结点、中间节点、尾结点），我们创建一个 `dummyHead` 节点，让 `dummyHead->next = head`。这样，任何节点的删除都变成了“删除某个节点 `A` 的 `next` 节点”的操作，代码逻辑就统一了。

---

### 代码逐行解析与图解

我们以一个链表 `1 -> 2 -> 3 -> 4 -> 5` 和 `n = 2` 为例，目标是删除倒数第2个节点，也就是 `[4]`。

**1. 初始化**

```cpp
ListNode* dummyHead = new ListNode(0);
dummyHead->next = head;
ListNode* slow = dummyHead;
ListNode* fast = dummyHead;
```
*   创建一个虚拟头结点 `dummyHead`。
*   将 `slow` 和 `fast` 指针都初始化为 `dummyHead`。

**初始状态:**
```
[dummy] -> [1] -> [2] -> [3] -> [4] -> [5] -> null
   ^
   |
 slow, fast
```

**2. 让 `fast` 指针先走 `n` 步**

```cpp
while(n-- && fast != NULL) {
    fast = fast->next;
}
```
*   这个循环让 `fast` 指针从 `dummyHead` 开始向前移动 `n` 次。
*   在我们的例子中 (`n=2`)，`fast` 会移动2步：
    *   第一次：`fast` 指向 `[1]`
    *   第二次：`fast` 指向 `[2]`

**当前状态:**
```
[dummy] -> [1] -> [2] -> [3] -> [4] -> [5] -> null
   ^              ^
   |              |
 slow           fast
```
此时，`slow` 和 `fast` 之间正好相隔 `n` 个节点。

**3. 【关键步骤】让 `fast` 再多走一步**

```cpp
fast = fast->next; // fast再提前走一步...
```
*   **这是这段特定代码实现中最关键、最巧妙的一步。** 为什么要多走一步？
*   **目标**: 我们的最终目的是让 `slow` 指针停在**待删除节点的前一个节点**上。这样我们才能执行 `slow->next = slow->next->next` 来完成删除。
*   通过让 `fast` 再多走一步，我们人为地在 `slow` 和 `fast` 之间制造了一个 `n+1` 的距离。这样，当 `fast` 走到链表末尾的 `null` 时，`slow` 就会恰好停在倒数第 `n` 个节点的前一个位置。

**当前状态:**
```
[dummy] -> [1] -> [2] -> [3] -> [4] -> [5] -> null
   ^                     ^
   |                     |
 slow                  fast
```

**4. `fast` 和 `slow` 指针一起移动**

```cpp
while (fast != NULL) {
    fast = fast->next;
    slow = slow->next;
}
```
*   现在 `fast` 和 `slow` 保持固定的距离，同步向后移动，直到 `fast` 指向 `null`。
*   我们来追踪它们的移动：
    *   **第1步**: `fast` -> `[4]`, `slow` -> `[1]`
    *   **第2步**: `fast` -> `[5]`, `slow` -> `[2]`
    *   **第3步**: `fast` -> `null`, `slow` -> `[3]`
*   当 `fast` 变为 `null` 时，循环终止。

**当前状态:**
```
[dummy] -> [1] -> [2] -> [3] -> [4] -> [5] -> null
                          ^                      ^
                          |                      |
                        slow                   fast
```

看！`slow` 指针现在正好停在了 `[3]`，也就是待删除节点 `[4]` 的前一个节点。

**5. 执行删除操作**

```cpp
slow->next = slow->next->next;
```
*   `slow->next` 是 `[4]`。
*   `slow->next->next` 是 `[5]`。
*   这行代码的效果是，让 `[3]` 的 `next` 指针直接指向 `[5]`，从而跳过了 `[4]`，实现了逻辑上的删除。

**删除后链表结构:**
```
[dummy] -> [1] -> [2] -> [3] -> [5] -> null
                          |      ^
                          +------+
                           ( [4] 还在内存中，但已从链中断开 )
```

**6. C++ 内存释放 (可选但规范)**
```cpp
// ListNode *tmp = slow->next;  // 这部分逻辑是更完整的 C++ 写法
// slow->next = tmp->next;
// delete tmp;
```
*   你代码中注释掉的这部分，是 C++ 中手动管理内存的规范做法。它先用一个临时指针 `tmp` 保存待删除节点 `[4]` 的地址，然后才修改 `slow` 的 `next` 指针，最后通过 `delete tmp` 释放 `[4]` 占用的内存，防止内存泄漏。
*   在很多在线编程平台，系统会自动回收内存，所以即使不写 `delete` 也能通过。但这是一个很好的编程习惯。

**7. 返回结果**

```cpp
return dummyHead->next;
```
*   为什么要返回 `dummyHead->next` 而不是 `head`？
*   因为如果原始的 `head` 被删除了（例如 `n=5` 的情况），`head` 指针就失效了。而 `dummyHead` 是我们自己创建的，它的 `next` 永远指向链表当前正确的头节点。

### 总结
该算法通过以下步骤，高效且优雅地解决了问题：
1.  **创建虚拟头结点**，统一删除逻辑。
2.  **设置快慢指针 `fast` 和 `slow`**，都指向虚拟头结点。
3.  **让 `fast` 先走 `n+1` 步**，为 `slow` 定位到“待删除节点的前驱”创造条件。
4.  **同时移动 `fast` 和 `slow`**，直到 `fast` 到达终点。
5.  此时 `slow` 就在正确的位置，**执行删除操作**。
6.  **返回新链表的头节点** `dummyHead->next`。