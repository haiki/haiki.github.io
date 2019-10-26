---
title: LinkedList-problems-easy
date: 2019-8-28 21:40:38
categories:
- leetcode
tags:
- linkedList
- easy
---
@[TOC]
```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
```

### 876.Middle of the Linked List

**题目描述：**

Given a non-empty, singly linked list with head node head, return a middle node of linked list.
If there are two middle nodes, return the second middle node.

**Example** :

```
Input: [1,2,3,4,5,6]
Output: Node 4 from this list (Serialization: [4,5,6])
Since the list has two middle nodes with values 3 and 4, we return the second one.
```

#### Solution 1-遍历

- 将链表遍历一遍，得到链表的长度，求出中间节点的位置，重新从头开始遍历，输出中间节点。

时间：O(2n)，空间O(1)？

```c++
class Solution {
public:
    ListNode* middleNode(ListNode* head) {
        if(head == NULL)
            return NULL;
        ListNode* tmp = head;
        int cnt = 0;
        while(tmp != NULL){
            cnt++;
            tmp = tmp->next;
        }
        int i = cnt/2;
        tmp = head;
        while(i != 0){
            tmp = tmp->next;
            i--;
        }
        return tmp;
    }
};
```



#### Solution 2-快慢指针

- 这个本来是想到了，但是自己不知道为什么演算错了。。。要考虑奇偶个数的情况；

时间：O(n/2)，空间O(1)-快慢指针

```c++
class Solution {
public:
    ListNode* middleNode(ListNode* head) {
        if(head == NULL)
            return NULL;
        ListNode* slow = head, *fast = head;
        
        //快指针比慢指针快一倍的速度，不是两个。。。
        while(slow->next != NULL && fast->next != NULL){
            //奇数个元素
            if(fast->next->next != NULL){
                fast = fast->next->next;
                slow = slow->next;
            }else{
                //返回方法一
                fast = fast->next;
                slow = slow->next;
                //return slow->next;  //返回方法二
            }
        }
        return slow;
    }
};
```

#### Solution 3-辅助数组

- 使用数组把所有元素放进去，按照数组随机存取的特性，直接定位到元素，这样就不是原来链接起来的链表了,用空间换时间。

- 时间O(n)，空间O(n)-vec用的空间

```c++
class Solution {
public:
    ListNode* middleNode(ListNode* head) {
        if(head == NULL)
            return NULL;
        vector<ListNode*>vec{head};//注意初始化是{}，不是[]
        
        ListNode* cur = vec.back();
        while(cur->next != NULL){
            vec.push_back(cur->next);
            cur = cur->next;            
        }
        return vec[vec.size()/2];//相对于第一种的优点就是考虑到了数组的随机访问快
    }
};
```



### 206. Reverse Linked List
**题目描述：**

Reverse a singly linked list.

**Example:**

```
Input: 1->2->3->4->5->NULL
Output: 5->4->3->2->1->NULL
```

#### solution1-迭代

- 主要目的是不要让链表断了，用三个指针分别表示当前节点，前一个，后一个.
- 时间：O(n)，空间：O(1)

```c++
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(head == NULL)
            return NULL;
        
        ListNode *cur=NULL,*pre=NULL,*post=NULL;
        cur=head;
        while(cur!=NULL)   //要注意判断条件
        {
            post=cur->next;
            cur->next=pre;
            pre=cur;
            cur=post;
        }     
        return pre;        
    }
};
//这个和上面的差不多，根本不是递归，就是迭代吧
/*
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(head == NULL)
            return NULL;
        
        ListNode *cur=NULL,*pre=NULL,*post=NULL;
        cur=head;
        return reverse(pre,cur,post);
    }
    ListNode* reverse(ListNode* pre, ListNode* cur, ListNode* post){
        if(cur == NULL)
            return pre;
        post = cur->next;
        cur->next = pre;
        pre  = cur;
        return reverse(pre,post,post);
    }
};
*/
```



#### Solution2-递归

- 主要是要理解递归。边界条件，cur == NULL则链表是空的，cur->next == NULL意味着链表已经到最后一个节点了，那么这个节点是新的head，开始返回。
- 递归时会为每个参数变量压栈，返回后就是当时的cur，那么cur的next是逆转的节点，这个节点的next就是cur。思路参考https://www.youtube.com/watch?v=MRe3UsRadKw
- 做的时候难点在于：指针的理解，`**`的理解及传参，以及如何改变head指针。
- 时间：O(n)，空间：O(n)-因为递归而使用的栈空间    

```c++
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(head == NULL)
            return NULL;
        
        ListNode *cur = head;
        reverse(cur,&head);
        return head;
    }
     void reverse(ListNode* cur,ListNode** head){
         if(cur->next == NULL){
             *head = cur;
             return ;
         }
         reverse(cur->next,head);
         cur->next->next = cur;
         cur->next = NULL;
     }
};
```



### 237.Delete Node in a Linked List

**题目描述：**

Write a function to delete a node (except the tail) in a singly linked list, given only access to that node.

Given linked list -- head = [4,5,1,9], which looks like following:

![img](https://assets.leetcode.com/uploads/2018/12/28/237_example.png)

 

**Example 1:**

```
Input: head = [4,5,1,9], node = 5
Output: [4,1,9]
Explanation: You are given the second node with value 5, the linked list should become 4 -> 1 -> 9 after calling your function.
```

#### solution-值替换

- 因为尾节点不为空：所以可以不删除当前节点，而是删除后一个节点。把给定当前节点，想要找到要删除节点的前一个节点，只能从头遍历;
- 将后一个节点的值赋值给当前节点，并让当前节点指向下下个节点，从而删除下个节点
- 如果要删除尾节点元素，那么就不能这样做，必须给head，从头向后遍历。

```c++
class Solution {
public:
    void deleteNode(ListNode* node) {
        /*
        node->val = node->next->val;
        ListNode* tmp = node->next;
        node->next =  node->next->next;
        delete(tmp);   //还是应该把这个指针释放掉，不然会内存泄漏，但不这样写也能通过
    */
        auto next = node->next;//直接操作指针
        *node = *next;
        delete next;   
    }
};
```



### 21.Merge Two Sorted Lists

**题目描述：**

Merge two sorted linked lists and return it as a new list. The new  list should be made by splicing together the nodes of the first two  lists.

**Example:** 

```
Input: 1->2->4, 1->3->4
Output: 1->1->2->3->4->4
```



#### solution1-迭代

- 面对list的问题，可以尝试一个dummyHead，避免对特殊情况的处理

```c++
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode *pNewHead = NULL; //合并后新的头结点
        ListNode *pTail = NULL;      //不断连接新节点的尾节点
        
        ListNode *pl1 = l1, *pl2 = l2;
        
        if(l1 == NULL)   //对于特殊情况：两个链表为空做判断，都为空时这句话就可以捕获到
            return l2;
        else if(l2 == NULL)
            return l1;
        else{
            if((l1->val) < (l2->val)){
                pNewHead = l1;
                l1 = l1->next;
            }else{
                pNewHead = l2;
                l2 = l2->next;
            }    
        }
        
        pTail = pNewHead;
        while(l1 != NULL && l2 !=NULL){
            if((l1->val) < (l2->val)){
                pTail->next = l1;
                pTail = pTail->next;
                l1 = l1->next;
            }else{
                pTail->next = l2;
                pTail = pTail->next;
                l2 = l2->next;
            }         
        }
        
        //对于两个链表不一样长的情况做处理 ，因为是直接在链表本身上做连接，不是新创建一条链表，所以这里剩下的节点只需要一个指针指向即可，不需要遍历连接。
        if(l1 != NULL){
            pTail->next = l1;
        }
        if(l2 != NULL){
            pTail->next = l2;
        }            
        return pNewHead; 
    }    
};
```



#### solution2-递归

因为在递归调用的过程当中，系统为每一层的返回点、局部量等开辟了栈来存储，因此递归次数过多容易造成栈溢出。所以选择递归时要慎重。思路看代码吧~

```c++
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        if(l1 == NULL)
            return l2;
        if(l2 == NULL)
            return l1;
        
        if((l1->val) < (l2->val)){
            l1->next = mergeTwoLists(l1->next, l2);
            return l1; //就是cur节点，一直指向当前链表的尾节点
        }else{
            l2->next = mergeTwoLists(l1,l2->next);
            return l2;
        }                    
    }
};
```

### 83.Remove Duplicates from Sorted List

**题目描述：**

Given a sorted linked list, delete all duplicates such that each element appear only *once*.

**Example 1:**

```
Input: 1->1->2
Output: 1->2
```

#### solution

- 看清楚题目，不要求把重复的所有元素都删除
- cur表示当前节点，依次和下一个节点比较，如果值一样，则与下一个节点比较，并每次删除一个值一样的节点
- 时间：O(n)，必须从头遍历到尾，空间：O(1)



```c++
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        if(head == NULL)
            return NULL;
        ListNode* cur = head;
       // ListNode* tmp = NULL;

        while(cur){
            while(cur->next && cur->val == cur->next->val){
               // tmp = cur->next;
                cur->next = cur->next->next;
                //delete(tmp);
            }
            cur = cur->next;
        }
        return head;
    }
};
```



### 141.Linked List Cycle

**题目描述**

Given a linked list, determine if it has a cycle in it.

To represent a cycle in the given linked list, we use an integer `pos` which represents the position (0-indexed) in the linked list where tail connects to. If `pos` is `-1`, then there is no cycle in the linked list.

**Example 1:**

```
Input: head = [3,2,0,-4], pos = 1
Output: true
Explanation: There is a cycle in the linked list, where tail connects to the second node.
```

![img](https://assets.leetcode.com/uploads/2018/12/07/circularlinkedlist.png)



#### solution1—快慢指针

- 第一种解法就是快慢指针，而且只需要O(1)空间，O(n)时间，但是奇怪的是，我因为指针指向原因，错了三次？

```c++
class Solution {
public:
    bool hasCycle(ListNode *head) {
        if(head == NULL || head->next == NULL)
            return false;
        
        ListNode* slow = head, *fast = head->next;
        while(fast != slow){
            if(fast == NULL || fast->next == NULL){
                return false;
            }
            slow = slow->next;
            fast = fast->next->next;
        }
        return true;
    }
```

#### solution2-辅助集合

- 检查某个节点是否被第二次访问，使用hash表，查看当前元素在hash表中是否存在，则需要find函数(set)
- 时间：O(n)，空间：O(n)

```c++
class Solution {
public:
    bool hasCycle(ListNode *head) {
        if(head == NULL || head->next == NULL)
            return false;
        
        ListNode* tmp = head;
        set<ListNode*>mySet;
        mySet.insert(tmp);
        tmp = tmp->next;
        
        while(tmp != NULL){
            if(mySet.find(tmp) == mySet.end()){
                mySet.insert(tmp);
                tmp = tmp->next;
            }else
                return true;
        }
        return false;
    }
};
```



### 234.Palindrome Linked List

**题目描述**

Given a singly linked list, determine if it is a palindrome.

回文是指：正向和反向读取得到的是一样的

**Example 1:**

```
Input: 1->2
Output: false
```

**Example 2:**

```
Input: 1->2->2->1
Output: true
```

#### solution1-辅助栈

- 遍历链表，使用栈存储，再次遍历链表，和出栈的元素比较
- 时间：O(n)，空间：O(n)

```c++
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        if(head == NULL || head->next == NULL)
            return true;
        
        stack<int>node;
        ListNode* tmp = head;
        while(tmp != NULL){
            node.push(tmp->val);
            tmp = tmp->next;
        }
        tmp = head;
        while(tmp != NULL){
            int v = node.top();
            if(tmp->val != v)
                return false;
            node.pop();
            tmp = tmp->next;
        }
            
        return true;
    }
```



#### solution2-递归

- 边界条件：cur == NULL，返回true
- 用一个指针指在链表头；递归不断执行，到了链表结尾，然后出栈，不断返回，表头指针不断往前走，也是一个后面和前面比较的过程。但是我在想，怎么能让它比较到中间的时候就停止呢？
- 时间：O(n)，空间：O(n)

```c++
class Solution {
public:
    ListNode* temp ;
    
    bool isPalindrome(ListNode* head) {
        if(head == NULL || head->next == NULL)
            return true;
        temp = head;
        return check(head);
    }
    bool check(ListNode* p){
        if(p == NULL)
            return true;
        bool isPal = check(p->next) & (p->val == temp->val);
        temp = temp->next;
        
        return isPal; 
    }
};
```

### 203.Remove Linked List Elements

**题目描述**

Remove all elements from a linked list of integers that have value **val**.

**Example:**

```
Input:  1->2->6->3->4->5->6, val = 6
Output: 1->2->3->4->5
```

#### solution-遍历

- 删除具有相同值的元素。因为要删除，所以需要记录当前元素的上一个，才能把指针连接到下一个
  为了处理头结点，引入dummyHead。或者直接，用cur->next和值做比较
- 时间：O(n)，空间：O(1)

```C++
class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        if(head == NULL)
            return NULL ;
        ListNode* dummyHead = new ListNode(0);
        dummyHead->next = head;
        ListNode* pre = dummyHead, *cur = head;
        
        while(cur != NULL){
            if(cur->val == val){
                pre->next = cur->next;
                cur = pre->next;
            }else{
                pre = cur;
                cur = cur->next;                
            }         
        }
        return dummyHead->next;                
    }
};
```

### 160.Intersection of Two Linked Lists

**题目描述**

Write a program to find the node at which the intersection of two singly linked lists begins.

For example, the following two linked lists:

![img](https://assets.leetcode.com/uploads/2018/12/13/160_statement.png)

begin to intersect at node c1.

#### solution1-求长度

- 求出两个链表的长度，因为两个链表可能不一样长，则让长的那个先走几步，然后两个一起走，去比较是否有一致的节点
- 时间：O(n)，空间：O(1)

```c++
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if(headA == NULL || headB == NULL)
            return NULL;
        //查询链表长度
        ListNode* p1 = headA, *p2 = headB;
        int lenA = 0, lenB = 0;
        while(p1 != NULL){
            lenA++;
            p1 = p1->next;
        }
        while(p2 != NULL){
            lenB++;
            p2 = p2->next;
        }
        //将长的那个链表定位到与短链表一样长的地方
        p1 = headA, p2 = headB;
        int step = (lenA>lenB)?(lenA-lenB):(lenB-lenA);
        if(lenA >= lenB){
            while(step){
                p1 = p1->next;
                step--;
            }
        }else{
            while(step){
                p2 = p2->next;
                step--;
            }
        }
        //开始同时遍历并比较两个链表的节点
        while(p1 != NULL && p2 != NULL){
            if(p1 != p2){//判断的是节点是否相同，不是节点的值
                p1 = p1->next;
                p2 = p2->next;
            }
            else
                return p1;  
        }
        return NULL;        
    }
};
```



#### solution2—辅助栈

- 如果有交集，那么倒着遍历时第一个不一样的节点就是分岔处。
- 顺序遍历，将元素分别存储在辅助的栈中。依次从栈中弹出元素比较
- 时间：O(n)，空间：O(n)
- 没写。。。

#### solution3-链表粘合

```c++
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if(headA == NULL || headB == NULL)
            return NULL;
        ListNode *p1 = headA, *p2 = headB;
        //只有拼接后的两条链都遍历完，才结束
        while(p1 != NULL && p2 != NULL && p1 != p2){
            p1 = p1->next;
            p2 = p2->next;
            if(p1 == p2)
                return p1;
            if(p1 == NULL)  p1 = headB;
            if(p2 == NULL)  p2 = headA;
        }
        return p1;//p1 == p2 的情况是两条链的公共节点是第一个的时候，不会执行while里面的语句，否则，在有前缀节点的情况下，一定是在循环内找到交点。如果没有交点，则p1是返回NULL
    }
};
```





### 707. Design Linked List

**题目描述：**

Design your implementation of the linked list. You can choose to use  the singly linked list or the doubly linked list. A node in a  singly linked list should have two attributes: `val` and `next`. `val` is the value of the current node, and `next` is a pointer/reference to the next node. If you want to use the doubly linked list, you will need one more attribute `prev` to indicate the previous node in the linked list. Assume all nodes in the linked list are 0-indexed.

Implement these functions in your linked list class:

- get(index) : Get the value of the `index`-th node in the linked list. If the index is invalid, return `-1`.
- addAtHead(val) : Add a node of value `val` before the first element of the linked list. After the insertion, the new node will be the first node of the linked list.
- addAtTail(val) : Append a node of value `val` to the last element of the linked list.
- addAtIndex(index, val) : Add a node of value `val` before the `index`-th node in the linked list. If `index` equals to  the length of linked list, the node will be appended to the end of  linked list. If index is greater than the length, the node will not be  inserted. If index is negative, the node will be inserted at the head of  the list.
- deleteAtIndex(index) : Delete the `index`-th node in the linked list, if the index is valid.

**Example**:

**Note:**

- All values will be in the range of `[1, 1000]`.
- The number of operations will be in the range of `[1, 1000]`.
- Please do not use the built-in LinkedList library.

#### solution

这道题对我来说最大的挑战不是思路，是C++语法的问题。
首先是不知道在类里面怎么去声明一个节点的类型，一个是不知道构造函数怎么用了，汗。。。
别人的思路的一个亮点：因为题目中要求了使用了头、尾，还有index的合法与不合法，所以设置三个全局变量head,tail,len, 在更新节点数量的时候要记着把len值做改变。

```c++
class MyLinkedList {
private:
    struct ListNode{
        int val;
        ListNode* next;
        ListNode(int v):val(v),next(NULL){}
    };
    ListNode* head;
    ListNode* tail;
    int len ;
    
public:
    /** Initialize your data structure here. */
    MyLinkedList() {
        head = NULL;
        tail = NULL;
        len = 0;
    }
    
    /** Get the value of the index-th node in the linked list. If the index is invalid, return -1. */
    int get(int index) {
        if(index >= len || index < 0)
            return -1;
        ListNode* p  = head;
    
        for(int i = 0;i < index; i++){
            p = p->next;
        }
        return p->val;       
    }

    /** Add a node of value val before the first element of the linked list. After the insertion, the new node will be the first node of the linked list. */
    void addAtHead(int val) {
        ListNode* newHead = new ListNode(val);
        newHead->next = head;
        head = newHead;
        if(len == 0)
            tail = newHead;
        ++len;
    }    
    /** Append a node of value val to the last element of the linked list. */
    void addAtTail(int val) {
        ListNode* newTail = new ListNode(val);
        if(len == 0){
            head = newTail;
            tail = newTail;
        }            
        tail->next = newTail;
        tail = newTail;
        newTail->next = NULL;
        ++len;
    }
    
    /** Add a node of value val before the index-th node in the linked list. If index equals to the length of linked list, the node will be appended to the end of linked list. If index is greater than the length, the node will not be inserted. */
    void addAtIndex(int index, int val) {
        if(index == len)
            addAtTail(val);
        else if(index > len)
            return ;
        else if(index <= 0)
            addAtHead(val);
        else{
            ListNode* p = head;
            ListNode* newNode = new ListNode(val);
            //找到要插入index的前一个节点，这样直接操作前一个节点的指向即可，所以是index-1，删除同理
            for(int i = 0; i < index-1;i++){
                p = p->next;
            }
            newNode->next = p->next;
            p->next = newNode;
            ++len;
        }                    
    }
    
    /** Delete the index-th node in the linked list, if the index is valid. */
    void deleteAtIndex(int index) {
        if(index < 0 || index >= len)
            return ;
        
        ListNode* tmp = NULL;//用来做delete操作，在OJ中，我发现不用delete也可以，但是为了养成好习惯，建议始终把不要的节点delete掉，以免内存泄漏
        
        //如果删除头节点,删除完就可以return了
        if(index == 0){
            tmp = head;
            head = head->next;
            delete tmp;
            --len;
            return;
        }

        ListNode* p = head;
        for(int i = 0; i < index-1; i++){
            p = p->next;
        }
        //删除尾节点
        if(index == len-1)
            tail = p;
        
        tmp = p->next;
        p->next = p->next->next;
        delete tmp;
        
        --len;
    }
};

/**
 * Your MyLinkedList object will be instantiated and called as such:
 * MyLinkedList* obj = new MyLinkedList();
 * int param_1 = obj->get(index);
 * obj->addAtHead(val);
 * obj->addAtTail(val);
 * obj->addAtIndex(index,val);
 * obj->deleteAtIndex(index);
 */
```

