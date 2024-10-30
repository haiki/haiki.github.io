---
title: stack-problems-easy
date: 2019-08-23 21:40:38
mathjax: true
categories:
- leetcode
tags:
- stack
- easy
---

@[toc]

### 1021.Remove Outermost Parentheses

题目描述太复杂的情况下看看给的case就大概知道题意了

**Example 2:**

```
Input: "(()())(())(()(()))"
Output: "()()()()(())"
Explanation: 
The input string is "(()())(())(()(()))", with primitive decomposition "(()())" + "(())" + "(()(()))".
After removing outer parentheses of each part, this is "()()" + "()" + "()(())" = "()()()()(())".
```



#### solution1-记录左括号个数，分割primitive串

- 因为是一道和栈相关的题目，所以第一反应就是使用栈，比如"(()())(())"，设定一个全局遍历s0("()")；遇到"("进栈，遇到")"时，栈顶的匹配"("出栈；如果此时栈为空，那么说明最后出栈的那对是外层括号，不计入最终的结果，直接开始遍历下一个元素。否则，res+=s0; 这种方法很闹心的一个地方是，对于"(()(()))"没法做。
- 另一种解法就是现在的解法，抛开stack，用string的知识来做。为的是找到最外层的括号，那么就从左向右遍历字符串，并用cnt记录当前左括号的个数，如果有匹配的右括号，则cnt--，有左括号时cnt++，当cnt==0时，即表示当前的右括号是外层括号，那么就把这个外层括号剥去，中间的加入res，那么此时有了待加入字符串的结束为止，还需要一个起始位置，则用start来表示，相当于字符串的截取。每次有了新的子串，start则初始化为新的起始位置。
- 时间O(n)，空间：O(n)——存储结果



```C++
class Solution {
public:
    string removeOuterParentheses(string S) {
        int len = S.size();
        string res = "";     
        int cnt = 1;//记录左括号的个数，当有右括号与左括号匹配时，减一
        int start = 0;//整个串被分为多个primative的串，记录每次新primative串的开始位置      
        for(int i = 1; i < len;i++){
            if(S[i] == '('){
                cnt++;
            }else{
                cnt--;
            }             
            if(cnt == 0){
                if(i - start >1){
                    string tmp(S.begin()+start+1,S.begin()+i);
                    res += tmp;
                }
                start = i+1;
            }
        }
        return res;      
    }
};
```



#### solution2-

- 别人的解法，和上面的思路差不多，但是只用open去记录当前的字符是否是结果的一部分。
（我的比这个看起来复杂，主要是被题目给的example引导着做复杂了。即先找出整个大括号包含的字符串，再去除外面的那层，看这个解法，完全可以边走边判断）
- open的数值意义：当前没有匹配的左括号的个数，左括号时加，右括号时减
- 如果open>0 && c == '('，那么加入res，open++
- 如果c == ')' && open >1，那么加入res，open--；大于1的目的是把最外层的去掉
- 时间：O(n)，空间：O(n)——存储结果



```c++
class Solution {
public:
    string removeOuterParentheses(string S) {
        int open = 0;
        string res = "";
        for(char c:S){
            if(c == '(' && open++ > 0) res += c; //最外面的左括号跳过
            if(c == ')' && open-- > 1 ) res += c;           
        }
        return res;
    }
};
```



### 1047.Remove All Adjacent Duplicates In String

**题目描述**

Given a string `S` of lowercase letters, a *duplicate removal* consists of choosing two adjacent and equal letters, and removing them.

We repeatedly make duplicate removals on S until we no longer can.

Return the final string after all such duplicate removals have been made.  It is guaranteed the answer is unique.

**Example 1:**

```
Input: "abbaca"
Output: "ca"
Explanation: 
For example, in "abbaca" we could remove "bb" since the letters are adjacent and equal, and this is the only possible move.  The result of this move is that the string is "aaca", of which only "aa" is possible, so the final string is "ca".
```

#### solution1-反向遍历，栈

- 每次将压入的元素和栈顶元素比较，如果一致就是重复，则新元素不入栈，栈顶元素出栈，这样可以很好的把中间有其他相同元素隔开的多个相同元素找出来。
- 反向遍历只是为了正向输出结果
- 时间：O(n)，空间：O(n)

```c++
class Solution {
public:
    string removeDuplicates(string S) {
        stack<char>s;
        int len = S.size();
        string res = "";
        
        for(int i = len-1; i >= 0 ; i--){
            if(!s.empty()){
                if(s.top() == S[i]){
                    s.pop();
                }else{
                    s.push(S[i]);
                }
            }else{
                s.push(S[i]);
            }     
        }
        while(!s.empty()){
            res += s.top();
            s.pop();
        }
           return res;   
    }
```



#### solution2-用string变相作为栈

- 这种写法，应该对string中的方法有充足的了解
- 时间：O(n)，空间：O(1)

```C++
class Solution {
public:
    string removeDuplicates(string S) {
        string res = "";
        for(char c : S){
            if(res.size() && res.back() == c) 
                res.pop_back();//凡是涉及到删除元素的操作，都要首先判断是否有元素
            else
                res.push_back(c);   
        }
        return res;
    }
};
```



### 682.Baseball Game

- “C”表示无效前一轮分数，“D”表示将前一轮有效分数乘2作为本轮分数，“+”表示将前两轮的分数之和作为本轮分数。

**Example 1:**

```
Input: ["5","2","C","D","+"]
Output: 30
Explanation: 
Round 1: You could get 5 points. The sum is: 5.
Round 2: You could get 2 points. The sum is: 7.
Operation 1: The round 2's data was invalid. The sum is: 5.  
Round 3: You could get 10 points (the round 2's data has been removed). The sum is: 15.
Round 4: You could get 5 + 10 = 15 points. The sum is: 30.
```

#### solution-辅助栈，字符串处理

- 完全按照人脑思维，对每个字符可能的情况进行判断
- 需要注意的就是，输入是一个字符串的数组，即数组中每个元素为字符串，当score为整数，且位数多于两位，则需要一个个遍历，算出字符串对应的整数。手动的算了。这里要注意有负数的情况。
- 时间：O(n)，空间：O(2)——一个辅助栈

```C++
class Solution {
public:
    int calPoints(vector<string>& ops) {
        stack<int>score;
        
        for(string s : ops){
            char c = s[0];
            
            if(c == 'C'){
                score.pop();
            }else if(c == 'D'){
                int tmp = score.top();
                score.push(tmp*2);
            }else if(c == '+'){
                int tmp1 = score.top();
                score.pop();
                int cur = tmp1 + score.top();
                score.push(tmp1);
                score.push(cur);
            }else{
                int num = 0;
                bool flag = false;//是否为负数
                for(char c : s){
                    if(c == '-'){
                        flag = true;
                        continue;
                    }
                    int val = c - '0';
                    num = num*10 + val;
                }
                if(flag)
                    score.push(-num);
                else 
                    score.push(num);
            } 
        }
        int sum = 0;
        while(!score.empty()){
            sum += score.top();
            score.pop();
        }
        return sum;
    }
```



#### solution2-vector

- 我简直就是一个被残害的少年，为什么想不开总是用stack，既不能随机访问，还得控制顺序
- 语法：STL 算法求和
    - int sum = accumulate(score.begin(),score.end(),10);//以10为基础开始加
    - string sum = accumulate(v.begin() , v.end() , string(" "));将字符串连接起来
- 找到了一个C++11里面字符串转整数的操作stoi(s,0,10)——将s从第0位开始，转换成10进制数。
- 判断是否为数字：isdigit()

```C++
class Solution {
public:
    int calPoints(vector<string>& ops) {
        vector<int>score;
        int index = 0;
        for(string s : ops){
            if(isdigit(s[0]) || s[0] == '-'){
               score.push_back(stoi(s));                
            }else if(s == "+"){
                score.push_back(score[index]+score[index-1]);
            }else if(s == "D"){
                score.push_back(score[index]*2);
            }else{
                score.pop_back();
            }
        index = score.size()-1;
        }
        return accumulate(score.begin(),score.end(),0);        
    }
};
```



### 844.Backspace String Compare

**题目描述**

Given two strings `S` and `T`, return if they are equal when both are typed into empty text editors. `#` means a backspace character.

**Example :**

```
Input: S = "ab##", T = "c#d#"
Output: true
Explanation: Both S and T become "".

Input: S = "a#c", T = "b"
Output: false
Explanation: S becomes "c" while T becomes "b".
```

#### solution1-辅助栈

- 想先比较一下两个字符串的大小，但是不行，因为如果一个比另一个多个#，也返回true；比如"ab##""c#d##";
- 最朴素的做法就是用两个辅助栈,当遇到'#'且栈不为空时就pop，否则入栈，然后从栈中依次弹出元素，比较是否相等，最后还要判断栈内有没有剩余元素 。
- 时间O(n)，空间O(n)

```C++
class Solution {
public:
    bool backspaceCompare(string S, string T) {
        stack<char> s;
        stack<char> t;
        
        for(int i = 0; i < S.size(); i++){
            if(S[i] == '#'){
                if(!s.empty()){
                    s.pop();
                }
            }else
                s.push(S[i]);                  
        }
        
        for(int j = 0; j< T.size(); j++){
            if(T[j] == '#'){
                if(!t.empty())
                    t.pop();
            }else
                t.push(T[j]);                
        }
        while(!s.empty() && !t.empty()){
            if(s.top() != t.top())
                return false;
            s.pop();
            t.pop();
        }
        if(!s.empty() || !t.empty())
            return false;//一个栈内元素比另一个的多
        return true;
    }
```



#### solution2-双指针

- 从后向前遍历，用count记录出现过的‘#’
- 如果count > 0 && 当前是一个非‘#’，说明可以对这个进行删除，count--；
- 如果当前是 '#'，则count++
- 否则比较两个元素，如果不相等，则false
- 测试用例需要考虑‘#’对的位置，最开头，中间，结尾，两个字符串长度不一样。能想出这种做法，并且把边界条件控制的很好的人，也是绝了。
- 时间：O(n)，空间：O(1)

```C++
/*
class Solution {
public:
    bool backspaceCompare(string S, string T) {
        stack<char> s;
        stack<char> t;
        int lens = S.size();
        int lent = T.size();
        for(int i = 0; i < lens; i++){
            if(S[i] == '#'){
                if(!s.empty()){
                    s.pop();
                }
            }else
                s.push(S[i]);                  
        }
        
        for(int j = 0; j< lent; j++){
            if(T[j] == '#'){
                if(!t.empty())
                    t.pop();
            }else
                t.push(T[j]);                
        }
        while(!s.empty() && !t.empty()){
            if(s.top() != t.top())
                return false;
            s.pop();
            t.pop();
        }
        if(!s.empty() || !t.empty())
            return false;//一个栈内元素比另一个的多
        return true;

    }
};*/

class Solution {
public:
    bool backspaceCompare(string S, string T) {
        int i = S.size()-1;
        int j = T.size()-1;
        int countS = 0,countT = 0;
        
        //存在两个字符串不一样长的情况
        while(i >= 0 || j >= 0){
            while(i >= 0 && (S[i] == '#' || countS > 0)){
                if(S[i] == '#'){
                    countS++;
                }else{
                    countS--;
                }
                i--;
            }
            while(j >=0 && (T[j] == '#' || countT > 0)){
                if(T[j] == '#'){
                    countT++;
                }else{
                    countT--;
                }
                j--;
            }
            
            if(i < 0 || j < 0)
                return i == j;
                
            if(S[i] != T[j])
                return false;
            else{
                 i--;
                 j--;
            }           
        }       
        return i == j;
    }
};
```

### 232.Implement Queue using Stacks

```C++
class MyQueue {
public:
    stack<int>s1;
    stack<int>s2;
        
    /** Initialize your data structure here. */
    MyQueue() {}
    
    /** Push element x to the back of queue. */
    void push(int x) {
        s1.push(x);
    }
    
    /** Removes the element from in front of queue and returns that element. */
    int pop() {
        while(!s1.empty()){
            s2.push(s1.top());
            s1.pop(); 
        }
        int res = s2.top();
        s2.pop();
        while(!s2.empty()){
            s1.push(s2.top());
            s2.pop();
        }  
        return res;   
    }
    
    /** Get the front element. */
    int peek() {
        while(!s1.empty()){
            s2.push(s1.top());
            s1.pop(); 
        }
        int res = s2.top();
        while(!s2.empty()){
            s1.push(s2.top());
            s2.pop();
        }  
        return res;   
    }
    
    /** Returns whether the queue is empty. */
    bool empty() {
        if(s1.empty())
            return true;
        else
            return false;
    }
};

/**
 * Your MyQueue object will be instantiated and called as such:
 * MyQueue* obj = new MyQueue();
 * obj->push(x);
 * int param_2 = obj->pop();
 * int param_3 = obj->peek();
 * bool param_4 = obj->empty();
 */
```



### 225.Implement Stack using Queues

```C++
class MyStack {
public:
    queue<int>q;
    /** Initialize your data structure here. */
    MyStack() {}
    
    /** Push element x onto stack. */
    void push(int x) {
        q.push(x);
    }
    
    /** Removes the element on top of the stack and returns that element. */
    int pop() {
        int res = q.back();
        int len = q.size();
        queue<int>tmp;
        while(len > 1){
            tmp.push(q.front());
            q.pop();
            len--;
        }
        q.pop();
        while(!tmp.empty()){
            q.push(tmp.front());
            tmp.pop();
        }
        return res;
    }
    
    /** Get the top element. */
    int top() {
        return q.back();
    }
    
    /** Returns whether the stack is empty. */
    bool empty() {
        return q.empty();
    }
};

/**
 * Your MyStack object will be instantiated and called as such:
 * MyStack* obj = new MyStack();
 * obj->push(x);
 * int param_2 = obj->pop();
 * int param_3 = obj->top();
 * bool param_4 = obj->empty();
 */
```



### 155.Min Stack

```c++
/*
solution
- 用一个全局变量来保存最小值，这样可以在常数时间内返回最小值
- 在push(),pop()时，更新minElenm
*/
class MinStack {
public:
    int minElem = 0x7fffffff;
    stack<int> s;
    /** initialize your data structure here. */
    MinStack() {
    }
    
    void push(int x) {
        s.push(x);
        if(x < minElem)
            minElem = x;
    }
    
    void pop() {
        int x = s.top();
        s.pop();
        if(x == minElem){
            minElem = 0x7fffffff;
            stack<int> tmp;
            while(!s.empty()){
                int t = s.top();
                s.pop();
                tmp.push(t);
                
                if(t < minElem){
                    minElem = t;
                }
            }
            while(!tmp.empty()){
                s.push(tmp.top());
                tmp.pop();
            }            
        }        
    }
    
    int top() {
        return s.top();
    }
    
    int getMin() {
        return minElem;
    }
};

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack* obj = new MinStack();
 * obj->push(x);
 * obj->pop();
 * int param_3 = obj->top();
 * int param_4 = obj->getMin();
 */
```



### 20.Valid Parentheses

**题目描述**

Given a string containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.

An input string is valid if:

1. Open brackets must be closed by the same type of brackets.
2. Open brackets must be closed in the correct order.

Note that an empty string is also considered valid.

**Example 1:**

```
Input: "([)]"
Output: false
```

#### 错！solution1-cnt统计左括号个数

- 用cnt计数，分别表示三种括号的左括号个数，当有右括号时，cnt--，左括号时，cnt++。但是"({)"这种，无法正确判断。——只能用于只有一种括号的情况
- 时间：O(n)，空间：O(1)

#### solution2-辅助栈

 - 如果是左括号，进栈，如果是右括号，匹配栈顶是否为对应的左括号
 - 时间：O(n)，空间：O(n)-辅助栈

```c++
class Solution {
public:
    bool isValid(string s) {

        stack<char>ch;
        for(int i = 0; i < s.size(); i++){
            if(s[i] == '(' || s[i] == '{' || s[i] == '[')
                ch.push(s[i]);
            else{
                if(ch.empty()) return false;//这个很重要，要弹出，首先就要判断栈是否为空
                if(s[i] == ')' && ch.top() != '(')
                    return false;
                if(s[i] == '}' && ch.top() != '{')
                    return false;
                if(s[i] == ']' && ch.top() != '[')
                    return false;
                
                ch.pop();
            }       
        }
        return ch.empty();         
    }
};
```



