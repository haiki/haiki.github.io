---
title: string-problems-easy
date: 2019-10-05 10:30:38
categories:
- leetcode
tags:
- string
- easy
---
@[toc]



### 1108.Defanging an IP Address

**题目描述：**

Given a valid (IPv4) IP `address`, return a defanged version of that IP address.

A *defanged IP address* replaces every period `"."` with `"[.]"`.

 

**Example 1:**

```
Input: address = "1.1.1.1"
Output: "1[.]1[.]1[.]1"
```



#### solution1-遍历

- 用一个新的串来记录结果，遍历原串，当遇到'.'时，结果串用"[.]"替代，否则用原串字符
- 时间：O(n),空间：O(n)
- 本来是想用string的库做，但是对于插入定位这件事就难倒我了。
    - 用iterator，用for循环去判断是否到结尾，但是只要插入元素，add.end()的iterator变了
    - 用replace，也不行，insert 和 find 结合，编译总是显示 out of range，在codeblock上是没错的。

```c++
class Solution {
public:
    string defangIPaddr(string address) {
        string res = "", str = "[.]";
        string::iterator it;
        for(it = address.begin(); it != address.end(); it++){
            if(*it == '.')
                res += str;
            else
                res += *it;
        }
        return res;
    }
};
```

### 709. To Lower Case

**题目描述**：

Implement function ToLowerCase() that has a string parameter str, and returns the same string in lowercase.

**Example 1:**

```
Input: "Hello"
Output: "hello"
```

#### solution

- ascii码表：大写字母 ：65-90，小写字母：97-122

```C++
class Solution {
public:
    string toLowerCase(string str) {
        int len = str.size();
        for(int i = 0; i< len; i++){
            if(str[i] <= 90 && str[i] >= 65)
                str[i] += 32;
        }
        return str;
    }
};
```

### 804. Unique Morse Code Words

**问题描述**

将字符串转换为摩尔斯电码表示，就是一种映射，返回的是结果中摩尔斯电码的不同的个数

**Example:**

```
Input: words = ["gin", "zen", "gig", "msg"]
Output: 2
Explanation: 
The transformation of each word is:
"gin" -> "--...-."
"zen" -> "--...-."
"gig" -> "--...--."
"msg" -> "--...--."

There are 2 different transformations, "--...-." and "--...--.".
```

#### solution-集合set的使用

- 看到那么长的映射，以为是考智力，找规律，直接放弃，看了别人的解法，不得不钦佩
- 每遍历一个单词，依次遍历其字符，并记录对应的国际摩尔斯电码，当一个单词遍历完，则拼接出单词对应的结果。把结果放入集合set中，因为unordered_set是无序的，比set快，且其中元素无重复。最后返回set集合的大小，就是不同的转换。
- 时间：O(n)，空间：O(2n)；n是单词个数

```C++
class Solution {
public:
    int uniqueMorseRepresentations(vector<string>& words) {
        vector<string>code{".-","-...","-.-.","-..",".","..-.","--.","....","..",".---","-.-",".-..","--","-.","---",".--.","--.-",".-.","...","-","..-","...-",".--","-..-","-.--","--.."};
        unordered_set<string>ret;//一提到找不同，首当其冲的就是集合
            
        for(string str : words){
            string res = "";
            for(char c : str){
                res += code[c - 'a'];//这点很厉害，直接定位到数组中对应的元素
            }
            ret.insert(res);
        }
        return ret.size();
    }
};
```

### 657. Robot Return to Origin

**题目描述：**

Valid moves are R (right), L (left), U (up), and D (down). If the robot  returns to the origin after it finishes all of its moves, return true.  Otherwise, return false.

**Example 1:**

```
Input: "UD"
Output: true 
Explanation: The robot moves up once, and then down once. All moves have the same magnitude, so it ended up at the origin where it started. Therefore, we return true.
```

#### solution

- 想要返回true，需要左右移动，上下移动的次数一样多

```C++
class Solution {
public:
    bool judgeCircle(string moves) {
        int l = 0, u = 0;
        for(char c : moves){
            switch(c){
                case 'L':l++;break;
                case 'R':l--;break;
                case 'U':u++;break;
                case 'D':u--;break;
            }   
        }
        return l == u && l == 0;        
    }
};
```

### 929. Unique Email Addresses

**题目描述**：

For example, in `alice@leetcode.com`, `alice` is the local name, and `leetcode.com` is the domain name.

local name：如果有"."，忽略，如果有"+"，第一个加号之后的都忽略，可用来过滤邮件（这两条规则对于domain name不适用）

返回能够收到邮件的不同地址个数。

**Example 1:**

```
Input: ["test.email+alex@leetcode.com","test.e.mail+bob.cathy@leetcode.com","testemail+david@lee.tcode.com"]
Output: 2
Explanation: "testemail@leetcode.com" and "testemail@lee.tcode.com" actually receive mails
```

#### solution-遍历

- 将email地址分为local和doamin两部分处理，只有两部分都满足题目要求时，将处理的结果拼接成合法的地址，insert到set中，最后返回set中的地址个数。
- 时间：O(n)，n是email的个数；空间：O(n)

- 看别人比较快的代码，似乎domain部分不需要做判断

```C++
class Solution {
public:
    int numUniqueEmails(vector<string>& emails) {
        unordered_set<string>res; //用set存储唯一的地址
        
        for(string email : emails){
            int pos = email.find("@"),len = email.size();
            string ret1 = "";
            
            string local(email,0,pos);
            string domain(email,pos+1,len-pos);
            
            bool isLocal = checkLocal(local,ret1);
            //bool isDomain = checkDomain(domain);
            
// && isDomain
            if(isLocal){
                //cout<<ret1+"@"+domain<<endl;
                 res.insert(ret1+"@"+domain);
            }               
        }
        return res.size();
        
    }
    bool checkLocal(string local, string &ret1){
        for(char c : local){
            if(c == '.')//点则跳过，
                continue;
            else if(c == '+')//之后的都忽略
                break;
            else
                ret1+=c;
        }
        return true;
    }
    //没看懂题目，domain里面原来可以有多个'.'
    /*bool checkDomain(string domain){
        for(char c : domain){
            if((c >= 'a' && c <= 'z') || c == '.')//记得加等号！！！
                continue;
            else
                return false;
        }
        return true;

    }*/
};
```

### 557. Reverse Words in a String III

**题目描述**：

Given a string, you need to reverse the order of characters in each  word within a sentence while still preserving whitespace and initial  word order.

**Example 1:**


```
Input: "Let's take LeetCode contest"
Output: "s'teL ekat edoCteeL tsetnoc"
```

#### solution

```C++
class Solution {
public:
    string reverseWords(string s) {
        int len = s.size();
        int beg = 0;
        int pos = s.find(" ",0);//老年人的劝告，少用这种find空格的方式
        while(pos != std::string::npos){
            reverseWord(s,beg,pos-1);
            beg = pos+1;
            pos = s.find(" ",beg);
        }
        reverseWord(s,beg,len-1);
        return s;
        
    }
    void reverseWord(string& str,int beg, int en){
        for(int i = beg,j = en; i < j; i++,j--){
            str[i] ^= str[j];
            str[j] ^= str[i];
            str[i] ^= str[j];
        }
    }
};
```

### 344. Reverse String

题目描述：

Write a function that reverses a string. The input string is given as an array of characters `char[]`.

Do not allocate extra space for another array, you must do this by **modifying the input array in-place** with O(1) extra memory.

You may assume all the characters consist of [printable ascii characters](https://en.wikipedia.org/wiki/ASCII#Printable_characters).

**Example 1:**

```
Input: ["h","e","l","l","o"]
Output: ["o","l","l","e","h"]
```

#### solution1-按异或操作符实现元素交换

- 看运行时间，似乎也不是很有效
- 时间O(n)，空间O(1)
- swap：和1的做法差不多，但是时间会少一些

```C++
class Solution {
public:
    void reverseString(vector<char>& s) {
        char tmp;
        int len = s.size();
        for(int i = 0; i < len/2; i++){
            tmp = s.at(i) ;
            s.at(i) = s.at(len - i - 1);
            s.at(len - i - 1) = tmp;
        }
    }
};
```

#### solution2-用string的库函数.reverse

```C++
class Solution {
public:
    void reverseString(vector<char>& s) {
        reverse(s.begin(),s.end());
    }
};
```

### 1170. Compare Strings by Frequency of the Smallest Character

**题目描述**

Let's define a function `f(s)` over a non-empty string `s`, which **calculates the frequency of the smallest character** in `s`. For example, if `s = "dcce"` then `f(s) = 2` because the smallest character is `"c"` and its frequency is 2.

Now, given string arrays `queries` and `words`, return an integer array `answer`, where each `answer[i]` is the number of words such that `f(queries[i])` < `f(W)`, where `W` is a word in `words`.

answer的个数和queries的个数一样，值的范围是[0,w.size()];

**Example 2:**

```
Input: queries = ["bbb","cc"], words = ["a","aa","aaa","aaaa"]
Output: [1,2]
Explanation: On the first query only f("bbb") < f("aaaa"). On the second query both f("aaa") and f("aaaa") are both > f("cc").
```

#### solution-sort

- 没有任何技术含量的，一步步按照人脑实现
- 先分别统计最小字符的个数，然后进行比较
- 时间：O(n^2)，n是字符的个数，空间：O(3n)，包括存储次数，以及结果的vector

```C++
class Solution {
public:
    //计算每个字符串内最小的字符出现的次数
        int numbers(string query){
            int num1 = 1;
            char tmp = query[0];

            for(int i = 1; i< query.size();i++){
                if(query[i] == tmp){
                    num1++;
                }else
                    break;
            }
            return num1;
    }
    
    vector<int> numSmallerByFrequency(vector<string>& queries, vector<string>& words) {
        
        vector<int> q,w;
        
    //分别计算queries和words中字符串的最小字符个数
        for(string query:queries){
            sort(query.begin(),query.end()); 
            int numq = numbers(query);
            q.push_back(numq);
            
        }
            
        for(string word:words){
            sort(word.begin(),word.end());            
            int numw = numbers(word); 
            w.push_back(numw);
        }
       
        //统计结果
        //sort(q.begin(),q.end());//输出时不能改变原来的顺序
        sort(w.begin(),w.end());
    
        vector<int>res;
        int lenw = w.size();
        for(int numq : q){
            int ret = 0;
            for(int i = 0; i < lenw; i++){
                if(numq < w[i]){
                    ret = lenw-i;
                    break;
                }                    
            }
            res.push_back(ret);
        }
        return res;
    }
};
```

#### solution2-比较精巧，但是得仔细理解

```
class Solution {
public:
    //用hash表的形式获取字符串的最小字符出现频率，这样就不需要对原串进行排序了
    int getF(string s){
        int f[26] = {0,};
        for(char c:s){
            f[c-'a']++;
        }
        for(int i: f){
            if(i > 0)
                return i;
        }
        return 0;
    }
    
    vector<int> numSmallerByFrequency(vector<string>& queries, vector<string>& words) {
        vector<int>fr(12,0);//用来统计words中，最小字符串的长度在1-10之间的个数
        for(string word:words){
            int f = getF(word);
            fr[f]++;
        }
        //从后向前统计，大于当前个数的最小字符串是多少个
        for(int i = 9; i >=0; i--){
            fr[i] += fr[i+1];
        }
        
        vector<int>res;
        for(string query:queries){
            int f = getF(query);
            res.push_back(fr[f+1]);//fr必须是12，如果是11，当这里的f是10，那么访问fr[11]，需要的就是12个元素大小的空间
        }
        return res;
        
    }
};
```

### 824. Goat Latin

**题目描述**

The rules of Goat Latin are as follows:

- If a word begins with a vowel (a, e, i, o, or u), append `"ma"` to the end of the word.
   	For example, the word 'apple' becomes 'applema'.
   
- If a word begins with a consonant (i.e. not a vowel), remove the first letter and append it to the end, then add `"ma"`.
   	For example, the word `"goat"` becomes `"oatgma"`.
   
- Add one letter `'a'` to the end of each word per its word index in the sentence, starting with 1.
   	For example, the first word gets `"a"` added to the end, the second word gets `"aa"` added to the end and so on.

Return the final sentence representing the conversion from `S` to Goat Latin. 

**Example 1:**

```
Input: "I speak Goat Latin"
Output: "Imaa peaksmaaa oatGmaaaa atinLmaaaaa"
```

#### solution-string库函数使用，下标移动

- 主要思路：按照第一个字符是否为元音，分为两种处理方法
    - 是元音，直接在结尾添加ma
    - 不是元音，将子串开头字符添加到结尾，删除开头字符，添加ma
- 对于拼接好的单词，还需要根据这是第几个单词，在单词最后加入几个a，这个用cnt来记录当前处理的是第几个元素，生成cnt长度的新子串，添加在单词后即可。
- 时间：O(n)，n是单词的个数，空间：O(C)，C是a的个数

```C++
class Solution {
public:
    void convert(string& S,int beg, int pos){
        switch(S[beg]){
                case 'a': case 'A':
                case 'e': case 'E':
                case 'i': case 'I':
                case 'o': case 'O':
                case 'u': case 'U':
                    S.insert(pos,"ma");
                    break;
                default:
                    S.insert(pos,1,S[beg]);
                    S.erase(beg,1);
                    S.insert(pos,"ma");
                    break;            
            }
    }
    
    string toGoatLatin(string S) {
        int pos = S.find(" ",0);
        int beg = 0;
        int cnt = 0;
        
        while(pos!=std::string::npos ){
            cnt++;
            convert(S,beg,pos);

            pos = S.find(" ",pos);
            string tmp(cnt,'a');
            S.insert(pos,tmp);
                    
            beg = pos+cnt+1;
            pos = S.find(" ",beg);
        }
        
        cnt++;//处理最后一个元素
        
        convert(S,beg,S.size());
        
        int len = S.size();
        string tmp(cnt,'a');
        S.insert(len,tmp);
        return S;
    }
};
```

### 521. Longest Uncommon Subsequence I 

**题目描述**

Given a group of two strings, you need to find the longest uncommon  subsequence of this group of two strings.
The longest uncommon subsequence is defined as the longest subsequence  of one of these strings and this subsequence should not be **any** subsequence of the other strings.

A **subsequence** is a sequence that can be derived from one sequence by deleting some characters without changing the order of the remaining elements. Trivially, any string is a subsequence of itself and an empty
string is a subsequence of any string.

**Example 1:**

```
Input: "aba", "cdc"
Output: 3
Explanation: The longest uncommon subsequence is "aba" (or "cdc"), 
because "aba" is a subsequence of "aba", 
but not a subsequence of any other strings in the group of two strings. 
```

#### solution

- 看solution，完全是脑筋急转弯
- 按道理，要把两个串的所有子序列求出来，对比看有没有子序列才对吧？但是这种情况是指，有公共子序列。对于没有的情况，就是长度最长的串本身。
- 时间：O(1)，空间：O(1)

```C++
class Solution {
public:
    int findLUSlength(string a, string b) {
        if(a == b)
            return -1;
        int len1 = a.size(),len2 = b.size();
        return len1>=len2?len1:len2;        
    }
};
```

### 917. Reverse Only Letters

**题目描述**

Given a string `S`, return the "reversed" string where all  characters that are not a letter stay in the same place, and all letters reverse their positions.

**Example 2:**

```
Input: "a-bC-dEf-ghIj"
Output: "j-Ih-gfE-dCba"
```

#### solution1-双指针遍历，交换

- 最左，最右双指针遍历
    - 遇到非字母的，则跳过
    - 只有当两个都是字母时交换；
- 时间：O(n)，空间：O(1)

```C++
class Solution {
public:
    string reverseOnlyLetters(string S) {
        int len = S.size();
        int left = 0, right = len-1;
        while(left < right){
            char l = S[left] ,r = S[right];
            if(isLetter(l)  && isLetter(r)){
                char tmp = S[left];
                S[left] = S[right];
                S[right] = tmp;
                left++;
                right--;
            }else if(!isLetter(l) && !isLetter(r)){
                left++;
                right--;
            }else if(!isLetter(l))
                left++;
            else
                right--;
        }
        return S;
        
    }
    bool isLetter(char s){
        if((s >= 'a' && s <= 'z') || (s >='A' && s <= 'Z') )
            return true;
        else 
            return false;
    }
};
```

#### solution2-stack

- 看解答，发现还是很有意思的一个思路，因为需要逆序字符串，因此可以用栈，这道题目就是将字符串逆序输出，其他字符保持在原位置不变。
- 时间：O(2n),需要扫描两边原字符串，空间：O(n)，额外的stack及返回字符串

```C++
class Solution {
public:
    bool isLetter(char s){
        if((s >= 'a' && s <= 'z') || (s >='A' && s <= 'Z') )
            return true;
        else 
            return false;
    }
    
    string reverseOnlyLetters(string S) {
        stack<char> tmp;
        for(char c:S){
            if(isLetter(c))
                tmp.push(c);
        }
        string res;
        for(char c:S){
            if(isLetter(c)){
                res.append(1,tmp.top());
                tmp.pop();
            }else
                res.append(1,c);      
        }
        return res;
    }
};
```

### 788. Rotated Digits

**题目描述**

A number is valid if each digit remains a digit after rotation. 0, 1,  and 8 rotate to themselves; 2 and 5 rotate to each other; 6 and 9  rotate to each other, and the rest of the numbers do not rotate to any  other number and become invalid.

Now given a positive number `N`, how many numbers X from `1` to `N` are good?

**Example**:

```
Input: 10
Output: 4
Explanation: 
There are four good numbers in the range [1, 10] : 2, 5, 6, 9.
Note that 1 and 10 are not good numbers, since they remain unchanged after rotating.
```

#### solution-dp记录表

- 本以为代码越敲，就会渐入佳境，结果却是：连题目都读不懂了呜呜呜
- 这道题乍得一看有点难，看了别人的解法才知道题目想表达的意思。
- 越来越大的数字都是由小数字组成的，那么通过读小数字是否为good num就可以判断大数是否为good
- 当数字为good时，必须包含2||5||6||9，同时可以包含0，1，8
- 只包含0，1，8 时只能说是valid的，不能是good的
- 当数字包含 3，4，7时，一定是invalid的
- 时间：O(n)，n是数字的个数，空间，O(n)

```C++
class Solution {
public:
    int rotatedDigits(int N) {
        int dp[N+1] = {0,} ;
        int cnt = 0;
        for(int i = 0; i <= N;i++){
        if(i < 10){
            if(i == 0 || i == 1 || i == 8)
                dp[i] = 1;
            else if(i == 2 || i == 5 || i == 6 || i == 9){       
                dp[i] = 2; 
                cnt++;
            }
        }else{
            int a = dp[i/10], b = dp[i%10];
            if(a == 1 && b == 1)
                dp[i] = 1;
            else if(a >= 1 && b >= 1){
                dp[i] = 2;//说明这个数的组成是2，5，6，9，0，1，8
                cnt++;               
                }      
            }
        }
        return cnt;
    }
};
```

### 696. Count Binary Substrings

**题目描述**

Give a string `s`, count the number of non-empty  (contiguous) substrings that have the same number of 0's and 1's, and  all the 0's and all the 1's in these substrings are grouped  consecutively.  

Substrings that occur multiple times are counted the number of times they occur.

**Example 2:**

```
Input: "10101"
Output: 4
Explanation: There are 4 substrings: "10", "01", "10", "01" that have equal number of consecutive 1's and 0's.
```

#### solution-线性扫描

- 遇到相同的就加，遇到不同的就开始统计，看这种连续性是几个
- 相当于把1和0分成group
- 时间：O(n),空间：O(1)

```C++
class Solution {
public:
    int countBinarySubstrings(string s) {
        int len = s.size();
        int pre = 0,cur = 1, sum = 0;//pre是上一个group中相同元素的个数，cur是当前的，这就是相当于对0和1分组

        for(int i = 1; i < len; i++){
            if(s[i] == s[i-1]){
                cur++;
            }else{
                sum += min(cur,pre);
                pre = cur;
                cur = 1;
            }            
        }
        sum += min(cur,pre);
        return sum;
    }
};
```

### 937. Reorder Data in Log Files

**题目描述**

Reorder the logs so that all of the letter-logs come before any  digit-log.  The letter-logs are ordered lexicographically ignoring  identifier, with the identifier used in case of ties.  The digit-logs  should be put in their original order.

Return the final order of the logs.

**Example 1:**

```
Input: logs = ["dig1 8 1 5 1","let1 art can","dig2 3 6","let2 own kit dig","let3 art zero"]
Output: ["let1 art can","let3 art zero","let2 own kit dig","dig1 8 1 5 1","dig2 3 6"]
```

#### solution-stable_sort

- 通过对标识符后的第一个字符进行比较，得出是isalpha or isdigit
- isalpha,则进入结果vector，否则，进入临时vector
    - 这样分开装的原因是，两者的排序要求不同，对于前者，则需要对标识符之外的串进行比较，而对于后者，则简单的排在前者后面就可以。
- 比较花时间的地方就是：如何对vector中的字符串，去掉某部分后进行大小比较，用comparator，要注意的是：cmp函数写在类内需要用static关键字修饰，类外则可以不用。
- 时间：O(n)，n是字符串的个数，空间：O(n)，两部分用于存储字符串的vector

```
bool cmp(string str1, string str2){
        int pos1 = str1.find(' ');
        int pos2 = str2.find(' ');
        string sa = str1.substr(pos1+1);
        string sb = str2.substr(pos2+1);
        if(sa.compare(sb) == 0)//如果除了标识符之外的地方相等，那么就要对标识符也做判断进行排序了
            return str1 < str2;
        else
            return sa < sb;
    }

class Solution {
public:
    //很迷，就一个static搞了我半个小时,如果放在类内作为比较函数，则必须要加static
    /*
    static bool cmp(string str1, string str2){
        int pos1 = str1.find(' ');
        int pos2 = str2.find(' ');
        return str1.substr(pos1+1) < str2.substr(pos2+1);
    }
    */
    vector<string> reorderLogFiles(vector<string>& logs) {
        vector<string>res;
        vector<string>tmp;
        
        for(string str:logs){
            int pos = str.find(' ');//find还可以用来查找某个完整的单词呢·就是string了
            if(isdigit(str[pos+1]))
                tmp.push_back(str);
            else
                res.push_back(str);
        }
        stable_sort(res.begin(),res.end(),cmp);//不仅要比较，对于后面都相等的，还需要保持原来的顺序 
        for(string s:tmp){
            res.push_back(s);
        }
        return res;
    }
};
```

### 1071. Greatest Common Divisor of Strings

For strings `S` and `T`, we say "`T` divides `S`" if and only if `S = T + ... + T`  (`T` concatenated with itself 1 or more times)

Return the largest string `X` such that `X` divides str1 and `X` divides str2.

#### solution-gcd

- 就是两个整数的最大公约数的变形，还做了一个多小时。。。。蠢啊
- 自己写的代码也一眼难尽，丑陋！
- 时间：O(n)，空间：O(1)

```C++
class Solution {
public:
    string gcdOfStrings(string str1, string str2) {
        //使得较长串始终为第一个串,这tm都能写错，这谁顶得住？？
        if(str1.size() < str2.size()){
            string tmp = str2;
            str2 = str1;
            str1 = tmp;
        }
        int len2 = str2.size();
		//==0，表示两个字符串相等
        //>0，表示被比较字符串小
        if(str1.compare(0,len2,str2) != 0)
            return "";
        
        if(str2.empty())
            return str1;
        
        return gcdOfStrings(str2,str1.substr(len2));
    }
};
```

#### solution-gcd

- 别人的解释，对于两个串，S1 = nT，S2 = mT，n>m，S3 = (n-m)T = xT; S1 = xT, S2 = mT,...,直到S2为空，返回S1，因为S1始终是长一点的那个。

```C++
class Solution {
public:
    string gcdOfStrings(string str1, string str2) {
        int len2 = str2.size();
        if(str1.size() < len2)
            return gcdOfStrings(str2,str1);
        if(str2.empty())
            return str1;
        if(str1.compare(0,len2,str2) != 0)
            return "";
        return gcdOfStrings(str1.substr(len2),str2);//str2始终是短的那个，所以一直在后面
    }
};
```

###  13. Roman to Integer

Roman numerals are represented by seven different symbols: `I`, `V`, `X`, `L`, `C`, `D` and `M`.

```
Symbol       Value
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
```

For example, two is written as `II` in Roman numeral, just two one's added together. Twelve is written as, `XII`, which is simply `X` + `II`. The number twenty seven is written as `XXVII`, which is `XX` + `V` + `II`.

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not `IIII`. Instead, the number four is written as `IV`. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as `IX`. There are six instances where subtraction is used:

- `I` can be placed before `V` (5) and `X` (10) to make 4 and 9. 
- `X` can be placed before `L` (50) and `C` (100) to make 40 and 90. 
- `C` can be placed before `D` (500) and `M` (1000) to make 400 and 900.

Given a roman numeral, convert it to an integer. Input is guaranteed to be within the range from 1 to 3999.

**Example 5:**

```
Input: "MCMXCIV"
Output: 1994
Explanation: M = 1000, CM = 900, XC = 90 and IV = 4.
```



- 总的来说，解题思路是，结果是每一位的和，但是存在加完了前面的需要减去的情况，比如IV，对于这种特例，两种解决方案。

### solution1-hash_map

- 从后向前遍历，如果s[i] < s[i+1]，那么需要减去s[i]，否则加上就行。
- 时间：O(n)，空间：O(1)

```C++
class Solution {
public:
    int romanToInt(string s){
        unordered_map<char,int> M = {{'I',1},
                                     {'V',5},
                                     {'X',10},
                                     {'L',50},
                                     {'C',100},
                                     {'D',500},
                                     {'M',1000}};
        int sum = M[s.back()];
        for(int i = s.size() - 2; i >= 0; i--){
            if(M[s[i]] < M[s[i+1]])
                sum -= M[s[i]];
            else 
                sum += M[s[i]];
        }
        return sum;
    }
};
```

### solution2-

- 先搜索字符串内有没有这类组合，有的话分别减去2，20，200，接着从前向后遍历，依次相加即可；因为加的过程中会多加
- 时间：O(n)，空间：O(2n)

```C++
class Solution {
public:
    int romanToInt(string s) {
        int sum = 0;
        if(s.find("IV",0) != string::npos) sum-=2;//原本是加4，但是后面遍历加的时候是先加了1，又加了5，就是多加了2
        if(s.find("IX",0) != string::npos) sum-=2;
        if(s.find("XL",0) != string::npos) sum-=20;
        if(s.find("XC",0) != string::npos) sum-=20;
        if(s.find("CD",0) != string::npos) sum-=200;
        if(s.find("CM",0) != string::npos) sum-=200;
        
        for(char c: s){
            switch(c){
                case 'I':sum+=1;break;
                case 'V':sum+=5;break;
                case 'X':sum+=10;break;
                case 'L':sum+=50;break;
                case 'C':sum+=100;break;
                case 'D':sum+=500;break;
                case 'M':sum+=1000;break;                    
            }
        }
        return sum;
        
    }
};
```

### 520. Detect Capital

**题目描述**

Given a word, you need to judge whether the usage of capitals in it is right or not.

We define the usage of capitals in a word to be right when one of the following cases holds:

1. All letters in this word are capitals, like "USA".
2. All letters in this word are not capitals, like "leetcode".
3. Only the first letter in this word is capital, like "Google".

 Otherwise, we define that this word doesn't use capitals in a right way.  

#### solution

- 字符串的前两位决定了接下来的字符串需要是大写还是小写才能合法，如果是大写，那么转入判断大写的函数，一旦发现小写就返回false，反之亦然。
- 注意要判断 word 只有一个字符的特例
- 时间：O(n)，空间：(1)

```C++
class Solution {
public:
    bool detectCapitalUse(string word) {
        if(word.size() == 1)
            return true;
        if(word[0] >= 'A' && word[0] <= 'Z'){
            if(word[1] >= 'A' && word[1] <= 'Z')
                return isUpper(word.substr(2));
            else 
                return isLower(word.substr(2));
        }
        else 
            return isLower(word.substr(1));        
    }
    bool isUpper(string str){
        for(char c:str){
            if(c < 'A' || c > 'Z')
                return false;
        }
        return true;
    }
    bool isLower(string str){
        for(char c:str){
            if(c < 'a' || c > 'z')
                return false;
        }
        return true;
    }
};
```

### 606. Construct String from Binary Tree

You need to construct a string consists of parenthesis and integers from a binary tree with the preorder traversing way.

**Example 2:**


```
Input: Binary tree: [1,2,3,null,4]
       1
     /   \
    2     3
     \  
      4 

Output: "1(2()(4))(3)"
```

- 需要注意的是，对于字符串使用“+”,则连续拼接的不能是两个常字符串，需要把整数转换为字符串
  - 在整数后面+""强制类型转换为字符串
  - to_string(整数)
  - ostringstre
  - sprintf等

#### solution1-递归，先序遍历

- 每到一个节点，只考虑以这个节点为根的子树
- 三种情况
    - 左右孩子都没有，则返回当前值即可
    - 有左孩子，但是没有右孩子，给左孩子加上括号，遍历左孩子即可
    - 有左右孩子，给左孩子及右孩子加括号，分别遍历
- 值得注意的是，没有单独考虑有右孩子，没左孩子的情况，这是因为，左孩子不管有没有，都不需要特殊处理，是需要加括号的，而右孩子没有的情况下，是没有括号的。

```C++
class Solution {
public:
    string tree2str(TreeNode* t) {
        if(t == NULL)
            return "";
        string str = to_string(t->val);//拼接的时候一直有问题，必须转成字符串？？？？？fuck，我的一个小时。。。
        
        if((t->left) == NULL && (t->right) == NULL)
            return  str; 
        if((t->right) == NULL)
            return str + "(" + tree2str(t->left) + ")";
        return str + "(" + tree2str(t->left) + ")(" + tree2str(t->right) + ")";
    }
};
```

#### solution2-stack

- 先序遍历，顺序为根，左，右
- 对每一个未访问的当前节点，设置为已访问，res+="(" + to_string(val)
- 对当前节点的左右孩子进行判断，还是三种情况：
    - 如果是有右孩子但是没左孩子，那么结果中直接加上"()"，不需要将元素入栈
    - 如果有右孩子，右孩子入栈
    - 如果有左孩子，左孩子入栈，这样的先后顺序使得左孩子可以先被访问
- 每一步只处理一个节点，当前栈顶节点，被访问过则弹出，没被访问过则访问
- 时间：O(n)，空间：O(n)

```C++
class Solution {
public:
    string tree2str(TreeNode* t) {
        if(t == NULL)
            return "";
        stack<TreeNode*> s;
        set<TreeNode*> visited;
        string res = "";
        s.push(t);
        
        while(!s.empty()){
            t = s.top();
            if(visited.find(t) != visited.end()){
                s.pop();
                res+= ")";
                //cout<<res<<endl;
            }else{
                visited.insert(t);
                res+="(" + to_string(t->val);
                if(t->left == NULL && t->right != NULL)
                    res+="()";
                if(t->right != NULL)//先push右边，再push左边，这样就可以先访问左边，后右边
                    s.push(t->right);
                if(t->left != NULL)
                    s.push(t->left);
            }
        }
        return res.substr(1,res.size()-2);//为了通用性，给第一个元素也是加了左右括号，最后要去掉
    }
            
};
```

### 383. Ransom Note

**题目描述**

ransom note 可以被magazine 构造，就是看 magazine中拥有的字符，是否足够形成ransome note，不用管字母顺序

**example**

```
canConstruct("a", "b") -> false
canConstruct("aa", "ab") -> false
canConstruct("aa", "aab") -> true
```

#### solution1-库函数调用

- 有点类似原串中找子序列的感觉，依次比较就完事了，结果却是理解错题目了。。
- 不用管字母出现的次序
- 时间：O(n)，空间：O(1)，效率非常低

```C++
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {   
        for(char c : ransomNote){
            int pos = magazine.find(c);
            if(pos != string::npos)
                magazine.erase(pos,1);
            else
                return false;
        }
        return true;
    }
};
```

#### solution2-map\数组

- 对原串中有的字符进行映射，依次累加对应字符的次数
- 对匹配串中的字符进行映射，一旦字符对应次数小于=0，则说明没法匹配了，返回false

```C++
/*
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        unordered_map<char,int> maga(26);
        for(char c: magazine){
            maga[c]++;
        }
        for(char c : ransomNote){
            if(maga[c] <= 0 )
                return false;
            maga[c]--;
        }
        return true;
    }
};*/
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        int maga[26] = {0,};
        for(char c: magazine){
            maga[c-'a']++;
        }
        for(char c : ransomNote){
            if(maga[c-'a'] <= 0 )
                return false;
            maga[c-'a']--;
        }
        return true;
    }
};
```

### 387. First Unique Character in a String

**题目描述**

Given a string, find the first non-repeating character in it and return it's index. If it doesn't exist, return -1.

**example**

```
s = "loveleetcode",
return 2.
```

#### solution-map

- 时间：O(n)，空间：O(n)

```C++
class Solution {
public:
    int firstUniqChar(string s) {
        int len = s.size();
        int cnt[26] = {0,};
        for(char c:s)
            cnt[c-'a']++;
        //因为要找出第一个单独出现的字符，那么遍历原串就可以是原来的顺序啊 ~~~你怎么这么笨，map仅仅作为一个查询的东西
        for(int i = 0 ; i < len; i++){
            if(cnt[s[i]-'a'] == 1)
                return i;
        }
        return -1;
    }
};
```

### 541. Reverse String II

**题目描述**

Given a string and an integer k, you need to reverse the first k  characters for every 2k characters counting from the start of the string. 

If there are less than k characters left, reverse all of them. 

If there are less than 2k but greater than or equal to k characters,  then reverse the first k characters and left the other as original.

**Example:**

```
Input: s = "abcdefg", k = 2
Output: "bacdfeg"
```

#### solution

- 将字符串按照2K分组（可能有余数，可能没有），对每个组内的前k个做逆转
- 对于最后一个不足2k的，单独处理
    - 有k个，逆转
    - 不够k个，全逆转！！！仔细读题目谢谢！
- 时间：O(n)，空间：O(1)

```C++
class Solution {
public:
    string reverseStr(string s, int k) {
        int len = s.size();
        if(len < k){
            reverse(s,0,len-1);//对于结束的点要注意
            return s;
        }

        //这种情况很容易忽略,都是不看题目的锅
        if(len < 2*k){
            reverse(s,0,k-1);
        }
            
        //len至少是大于2k的
        int cnt = len/(2*k);
        
        int start = 0;
        for(int i = 0; i < cnt; i++){
            start = 2*i*k;
            reverse(s,start,start+k-1);
        }
        start += 2*k;
        if(len - start >= k)
            reverse(s,start,start+k-1);
        if(len - start < k)
            reverse(s,start,len -1);
        
        return s;
    }
    
    void reverse(string& s, int start, int en){
        for(int i = start, j = en; i < j; i++,j--){
            s[i] ^= s[j];
            s[j] ^= s[i];
            s[i] ^= s[j];
        }
    }
};
```

#### solution2

```C++
class Solution {
public:
    string reverseStr(string s, int k) {
        int start = 0;
        int len = s.size();
        while(start < len){
            if(start + k < len)
                reverse(s.begin()+start, s.begin()+start+k );
            else
                reverse(s.begin()+start,s.end());
            start += k + k; 
        }
        return s;
    }
};
```

### 345. Reverse Vowels of a String

**Example 1:**

```
Input: "hello"
Output: "holle"
```

#### solution- 双指针

- 分别向前，向后扫描，交换元音字母
- 时间：O(n)，空间：O(1)

```C++
class Solution {
public:
    bool isVowel(char c){
        if(c == 'a' || c == 'A'|| c == 'e'|| c == 'E'|| c == 'i'|| c == 'I'|| c == 'o'|| c == 'O' || c == 'u' || c == 'U')
            return true;
        else
            return false;
    }
    string reverseVowels(string s) {
        
        int len = s.size();
        int i = 0, j = len-1;
        while(i < j){
            while(i < j && !isVowel(s[i])){
                    i++;
            }
             while(i < j && !isVowel(s[j])){              
                    j--;
            }
            if(i < j){
                cout<<s[i]<<":"<<s[j]<<endl;
                char tmp = s[i];
                s[i] = s[j];
                s[j] = tmp;                
            }
            i++;
            j--;        
        }
        return s;
    }  
};
```



### 551. Student Attendance Record I

**题目描述**

You are given a string representing an attendance record for a student. The record only contains the following three characters:  

1. **'A'** : Absent. 
2. **'L'** : Late.
3.  **'P'** : Present. 

 A student could be rewarded if his attendance record doesn't contain **more than one 'A' (absent)** or **more than two continuous 'L' (late)**.    

#### solution-统计

-  时间：O(n)，空间：O(1)
- 题目得仔细看，一不小心就写错了

```C++
class Solution {
public:
    bool checkRecord(string s) {
        int len = s.size();
        int cntA = 0;
        for(int i = 0; i < len; i++){
            if(s[i] == 'A')
                cntA++;
        }
        if(cntA > 1 || s.find("LLL") != string::npos)
            return false;
        else
            return true;    
    }
};
```

#### solution2-if-else

- 别人写的，还没有理解其妙处

```C++
class Solution {
public:
    bool checkRecord(string s) {
        int l = 0, a = 0;
        for(char c:s){
            if(c == 'A')
                a++;
            
            if(c == 'L')//为啥这里不可以用if else？c不是只能等于一个数？我知道了，对于连续的L，会一直计数，然后下面的if会判断是否已经到了不合格的时候，一旦不连续，则直接被置0，从头开始
                l++;
            else
                l = 0;
            if(a > 1 || l > 2)
                return false;
        }
        return true;
    }
};
```

### 415. Add Strings

**题目描述**

Given two non-negative integers `num1` and `num2` represented as string, return the sum of `num1` and `num2`

#### solution

- 就是小学生加法，一步步加，比如 A+B，需要考虑A长度，B长度，以及进位
- 这么简单的题，也能耗个几天才做完，自己写的代码里有一半都有点冗余，所以直接放参考别人写的了
- 每次只需要考虑当前位的A[i],B[j],carry
- 时间：O(n)，空间：O(n)，n表示相加的数最长长度

```C++
class Solution {
public:
    string addStrings(string num1, string num2) {
        string res = "";    
        int sum = 0;
        int i = num1.size()-1, j = num2.size()-1,carry = 0;
        while(i >= 0 || j >= 0 || carry == 1){
            int x = i<0?0:num1[i]-'0';
            int y = j<0?0:num2[j]-'0';
            sum = x+y+carry;
            carry = sum/10;
            res+=to_string(sum%10);
            i--;
            j--;
        }
        reverse(res.begin(),res.end());
        return res;
    }
};
```

### 67. Add Binary

**题目描述**

Given two binary strings, return their sum (also a binary string).

**Example 2:**

```
Input: a = "1010", b = "1011"
Output: "10101"
```

#### solution-类似string的加法

- 需要考虑的就是a[i]，b[j]，carry

```C++
class Solution {
public:
    string addBinary(string a, string b) {
        int i = a.size()-1,j = b.size()-1;
        int carry = 0;
        string res = "";
        while(i >= 0 || j >= 0 || carry == 1 ){
            int sum = carry;
            if(i >= 0){
                sum += a[i]-'0';
                i--;
            }
            if(j >=0 ){
                sum += b[j]-'0';
                j--;
            }
            res += to_string(sum%2);
            carry = sum/2; 
        }
        reverse(res.begin(),res.end());
        return res;       
    }
};
```

### 925. Long Pressed Name

**题目描述**

name是真名字，typed是不小心有多打的，typed是不是真的多打的

#### solution

- 因为typed中有长重复输入，那么如果它是name的长输入，长度一定是大于等于name的，如果两者是相等的，直接返回成功。
- 遍历typed，将typed中的连续相同字符用cnt表示为一个group
- 遍历name，依次探测typed中 group 的元素个数以及元素本身是否匹配
- 时间：O(n)，空间：O(1)

```C++
class Solution {
public:
    bool isLongPressedName(string name, string typed) {
        if(name == typed)
            return true;
        
        int len1= name.size(),len2 = typed.size();
        if(len1 > len2)
            return false;
        
        int i = 0, j = 0;
        while(i < len1 && j < len2){
            if(name[i] != typed[j])
                return false;
            i++;
            j++;
            int cnt = 0;
            int typeTmp = typed[j-1],nameTmp = name[i-1];
            while((typed[j] == typeTmp ) && (j < len2)){
                cnt++;
                j++;
            }
            while((name[i] == nameTmp) && (i < len1)){
                if(cnt > 0){
                    cnt--;
                    i++;
                }else
                    return false;
            }
        }
        return i == len1 && j == len2;
    }
};
```

### 819. Most Common Word

**题目描述**

Given a paragraph and a list of banned words, return the most frequent word that is not in the list of banned words. 

Words in the list of banned words are given in lowercase, and free of  punctuation.  Words in the paragraph are not case sensitive.  The answer is in lowercase.

**Example:**

```
Input: 
paragraph = "Bob hit a ball, the hit BALL flew far after it was hit."
banned = ["hit"]
Output: "ball"
```

#### solution

- 做题之前先想清楚再做，每次都是，要不做一半，发现理解错题意了，要不就是做到最后，越做越乱
- 新语法：sstringstream，map的make_pair
- 思路就是：
    - 首先把paragraph做处理，把其他punctuation都变为“ ”，在这个过程中，把大写都转换为小写。因为在banned中的单词都是小写。这里要注意的是，把逗号之类的转换为空格，加上本来就存在的空格，会出现两个空格的清空，用string的find空格，会出错！！！
    - 遍历段落中的单词，记录出现次数最多并且不再banned中的
        - 这里的一个技巧点，并不是总需要一步一步向下走，像这种一边查看是否在banned中，一边就可以去比较出现次数，出现次数一旦是最多的，就可以记录，而不是先将所有的单词出现次数记录一遍，然后再去遍历这个次数，找出最多
- 因为用到了在banned中find的操作，因此将vector转换为set了
- 要返回的是string，这个string的次数还要出现最多，因此选用map作为存储中间结果的数据结构
- 时间：O(n+m),空间：O(n+m)，n和m分别为para和banned的大小

```C++
class Solution {
public:
    string mostCommonWord(string para, vector<string>& banned) {
        unordered_set<string>s(banned.begin(),banned.end());
        unordered_map<string,int>mymap;
        int len = para.size();

        for(char &c : para)
            c = isalpha(c)?tolower(c):' ';
       
        pair<string,int> res("",0);
        istringstream iss(para);
        string word;
       while(iss >> word){
            if((s.find(word) == s.end()) && (++mymap[word] > res.second))
                res = make_pair(word,mymap[word]);
        }       
        return res.first;
    }
};
```

### 443. String Compression

**Example 1:**

```
Input:
["a","a","b","b","c","c","c"]

Output:
Return 6, and the first 6 characters of the input array should be: ["a","2","b","2","c","3"]

Explanation:
"aa" is replaced by "a2". "bb" is replaced by "b2". "ccc" is replaced by "c3".
```

#### solution

- 思路比较简单，但是有点难写
- 考虑两种情况
    - 连续出现的字符，首先定位一个字符，接着一直向后扫描
    - 相邻两个字符不一致（这里比较时用的是：当前字符和下一个字符）/已经到了字符末尾
        - 记录当前字符，记录当前字符出现的次数
- 时间：O(n)，空间：O(1)

```C++
class Solution {
public:
    int compress(vector<char>& chars) {
        int len = chars.size();
        
        int cur = 0, newIndex = 0;//cur：当前比较的字符，newIndex：是新记录的vector，因为新的结果大小 <= 原vector，因此可以一边遍历重复使用，最后返回index
        for(int i = 0; i< len; i++){
            if(i+1 == len || chars[i] != chars[i+1]){
                chars[newIndex++] = chars[i];
                
                 //记录当前重复字符出现的次数，对于只出现一次的，则没有这步
                if(cur < i){
                    string cnt = to_string(i-cur+1);
                    for(char c: cnt)
                        chars[newIndex++] = c;
                }              
                cur = i+1;
            }
        }
        return newIndex;     
    }
};
```

### 20. Valid Parentheses

**题目描述**

Given a string containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.

An input string is valid if:

1. Open brackets must be closed by the same type of brackets.
2. Open brackets must be closed in the correct order.

Note that an empty string is also considered valid.

#### solution2-辅助栈

 - 如果是左括号，进栈，如果是右括号，匹配栈顶是否为对应的左括号
 - 时间：O(n)，空间：O(n)-辅助栈

```C++
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

### 434. Number of Segments in a String

**题目描述**

Count the number of segments in a string, where a segment is defined to be a contiguous sequence of non-space characters.

**Example:**

```
Input: "Hello, my name is John"
Output: 5
```

#### solution-查找空格-错误

- 这道题就是在考察各种特殊情况
    - 结尾有没有空格
    - 字符串为空
    - 只有一个字符串
- 事实证明我写的这个不行，突然想到了前几天用到的istringstream

```C++
class Solution {
public:
    int countSegments(string s) {
        if(s.empty())
            return 0;
        int cnt = 0;
        istringstream ss(s);
        string w;
        while(ss>>w){
            cnt++;
        }
        return cnt;  
    }
};
```

### 125. Valid Palindrome

**题目描述**

Given a string, determine if it is a palindrome, considering only alphanumeric characters and ignoring cases.

**Example 1:**

```
Input: "A man, a plan, a canal: Panama"
Output: true
```

#### solution1

- 对原字符串进行处理，注意题目的意思包括字符和数字
- 判断是否为回文
- 时间：O(n),空间：O(1)

```C++
class Solution {
public:
    bool isPalindrome(string s) {
        if(s.empty())
            return true;
        
        int len = s.size();
        for(int i = 0; i < len; i++){
            char c = s[i];
            //s[i] = isalpha(c)?tolower(s[i]):'';//' '不知道为啥不能在这里放
            
            if(isalpha(c))
                s[i] = tolower(c);
            else if(isdigit(c)){
                continue;
            }
            else
                s[i] = ' ';                  
        } 
        
        int k = 0, j = len-1; 
        
        while(k < j){
            while(s[k] == ' ' && k < j)  k++;              
            while(s[j] == ' ' && k < j)  j--;
                
            if(s[k] != s[j])
                return false;
            k++;
            j--;//一定要记得在这里更新循环值
        }
       return true;
        
    }
};
```

#### solution2

- isnumalp()：既可以判断是否为数字，又可以判断是否为字母
- transform()：对大小写的转换

```C++
class Solution {
public:
    bool isPalindrome(string s) {
        if(s.empty())
            return true;
        transform(s.begin(),s.end(),s.begin(),::tolower);
        
        int i = 0, j = s.size()-1;
        while(i < j){
            bool flag1 = isAlphaNumeric(s[i]), flag2 = isAlphaNumeric(s[j]);
            
            if(flag1 && flag2){
                if(s[i] == s[j]){
                    i++;
                    j--;
                }else
                    return false;                    
            }else if(flag1){
                j--;
            }else if(flag2){
                i++;
            }else{
                i++;
                j--;
            }
        }
        return true;       
    }
    bool isAlphaNumeric(char c){
        if((c >= '0' && c <= '9') ||(c >= 'a' && c <= 'z'))
            return true;
        else
            return false;
    }
};
```



### 680. Valid Palindrome II

 **题目描述**

Given a non-empty string `s`, you may delete **at most** one character.  Judge whether you can make it a palindrome. 

**Example 1:**

```
Input: "abca"
Output: True
```

#### solution

- 回文就是正着反着读出来都一样
- 出栈读取和原字符串一样，这里的区别是，可以有一次机会，某个字符可以不是回文
- 这里有个问题就是，在遍历过程中，当两个字符不一致的时候，到达是哪一个字符向前走一个，因此，想到了递归

- 不断地缩小问题，双指针，一个从前往后，一个从后往前，每次比较两个字符   - 如果相等，则缩小范围
    - 如果不相等，则向左或者向右缩小范围 
- 时间：O(n)，空间：O(n)

```C++
class Solution {
public:
    bool isEqual(string s, int beg, int en){
         //cout<<s.substr(beg,en-beg+1)<<endl;
        /* // 内存超了
        if(beg > en)
            return true;
        if(s[beg] == s[en])
            return isEqual(s, beg+1, en-1);
        else
            return false;               
       */ 
        for(int i = beg, j = en; i < j ; i++, j--){
            if(s[i] != s[j])
                return false;
        }
        return true;                 
    }
    
    bool validPalindrome(string s) {
        int len = s.size();
        int beg = 0, en = len-1;
        
        while(beg <= en){
            if(s[beg] != s[en]){
                return (isEqual(s,beg+1,en)) || (isEqual(s,beg,en-1));
            }    
            beg++;
            en--;
        }        
        return true;       
    }
};
```

### 28. Implement strStr()

**题目描述**

Return the index of the first occurrence of needle in haystack, or **-1** if needle is not part of haystack.

**Example 1:**

```
Input: haystack = "hello", needle = "ll"
Output: 2
```
**solution**
```C++

class Solution {
public:
    int strStr(string haystack, string needle) {
        if(needle.empty() || haystack == needle)
            return 0;

        int len1 = haystack.size(),len2= needle.size();
        if( len1 < len2 )
            return -1;
        int pos = haystack.find(needle,0);
        if(pos != string::npos)
            return pos;
        else
            return -1;    
    }
};
```

### 14. Longest Common Prefix

**题目描述**

Write a function to find the longest common prefix string amongst an array of strings.

If there is no common prefix, return an empty string `""`.

**Example 1:**

```
Input: ["flower","flow","flight"]
Output: "fl"
```

#### solution

- 依次遍历vector，依次比较每个单词对应的元素,用set中元素是unique的特性来存储相同元素
- 时间：有一个排序的过程，O(Nlog(N))?，双层循环：O(n*N)，n是最短字符串的长度，N是vector的大小
- 空间：O(1)

```C++
class Solution {
public:
    static bool cmp(const string& s1, const string& s2){
        return s1.size() < s2.size();
    }
    string longestCommonPrefix(vector<string>& strs) {
        int vLen = strs.size();
        if(strs.empty())
            return "";
        
        if( vLen == 1)
            return strs[0];
        
        sort(strs.begin(),strs.end(),cmp);//表示最长公共前缀是最短的那个字符串,直接用sort是对字符本身的排序，而这里需要的是对字符串的长度进行排序
        int sLen = strs[0].size();
        //cout<<sLen<<endl;
                
        for(int i = 0; i< sLen; i++){
            char tmp = strs[0][i];
            for(int j = 1; j < vLen; j++){
                if(strs[j][i] != tmp)
                    return strs[0].substr(0,i);;
            } 
        }        
        return strs[0];        
    }
};
```

### 58. Length of Last Word

**题目描述**

Given a string *s* consists of upper/lower-case alphabets and empty space characters `' '`, return the length of last word in the string.

#### solution

- 看别人的代码，总有一种胜读十年书的感觉
- 时间：O(n)，空间：O(1)
- note:isblank()
- 让别人的智慧来熏陶一下自己，自己的代码就放在后面来激励一下自己吧

```C++
class Solution {
public:
    int lengthOfLastWord(string s) {
        if(s.empty())
            return 0;
        int i = s.size()-1;
        while(i >= 0 && isblank(s[i]))
            i--;
        int len = 0;
        while(i >= 0 && !isblank(s[i])){
            len++;
            i--;
        }
        return len;
    }
};
```



```c++
class Solution {
public:
    int lengthOfLastWord(string s) {
        if(s.empty())
            return 0;
        int len = s.size();
        int i = len-1;
        while(i >= 0 && s[i] == ' ') i--;
        if(i < 0)
            return 0;
        
        int start = i;
        while(i >= 0){
            if(s[i] == ' ')//这里是统计最后一个单词的停止地方
                return start - i;
        }            i--;

        return start - i;
    }
};
```

### 686. Repeated String Match

**题目描述**

Given two strings A and B, find the minimum number of times A has to  be repeated such that B is a substring of it. If no such solution,  return -1.

For example, with A = "abcd" and B = "cdabcdab".

Return 3, because by repeating A three times (“abcdabcdabcd”), B is a  substring of it; and B is not a substring of A repeated two times  ("abcdabcd").

#### solution

- B是A*k的子串，求最小的k
- 首先肯定需要A的长度大于等于B，之后比较B是否包含在A内
- 当长度是k时不包含，则再加一个A如果还不包含，那指定不包含了
- 时间：O(n)，空间：O(n)：n是字符的个数

```c++
class Solution {
public:
    int repeatedStringMatch(string A, string B) {
        if(A.find(B) != string::npos)
            return 1;
        
        int time = 1;
        string resA = A;
        while(resA.size() < B.size()){
            resA += A;//这里不能是A的自加，因为A变化后，下次加的是其新值
            time++;
        }
        int pos1 = resA.find(B);
        if(pos1 != string::npos)
            return time;
        
        resA += A;
        int pos2 = resA.find(B);
        if(pos2 != string::npos)
            return time+1;
        
        return -1;
    }
};
```

### 859. Buddy Strings

**题目描述**

Given two strings `A` and `B` of lowercase letters, return `true` if and only if we can swap two letters in `A` so that the result equals `B`.

**Example 1:**

```
Input: A = "ab", B = "ba"
Output: true

Input: A = "aa", B = "aa"
Output: true
```

#### solution1-遍历

- A、B的长度不等，那么一定不可能相等
- A、B长度为2，A[1] == B[0] && A[0] == B[1]
- 如果两者都相等，除非有个元素是重复出现过，否则也不可以。
- 主要思路是：依次比较A和B的每个元素，如果第一次不相等，则记录位置，等到第二次不相等的时候，看A，B不相等的元素是否和第一次不相等位置的元素正好匹配
- 时间：O(n)，空间：O(1)

```C++
class Solution {
public:
    bool buddyStrings(string A, string B) {
        int lenA = A.size(), lenB = B.size();
        if(lenA != lenB)
            return false;
        if(lenA == 2)
            return (A[1] == B[0]) && (A[0] == B[1]);
        if(A == B)
            return hasDup(A);//对于相等的A和B，当且仅当A有重复元素时，可以交换相同元素，否则，不可以
            
        int i = 0,first = 0;//first记录第一次不一致的位置
        bool flag = false,flag1 = false;//flag为true时表示已经有过不一致,flag1为true表示有过第二次不一致，而且已经有过一次交换的机会了，不能再有不一致了
        while(i < lenA){
            if(A[i] == B[i]){
                i++;
            }else if(flag){
                if(!flag1  && A[first] == B[i] && B[first] == A[i]){
                     i++;
                     flag1 = true;
                }else
                    return false;
            }else{
                flag = true;
                first = i;
                i++;
            }
        }
        return true;;   
    }
    
    bool hasDup(string s){
        int len = s.size();
        for(int i = 0; i < len-1; i++){
            if(s.find(s[i],i+1) != string::npos)
                return true;
        }
        return false;
    }   
};
```

#### solution2-利用额外空间

- 主要的不同就是
    - 对于相同的两个字符串，判断是否有重复元素，用了set
    - 对于不同的元素位置的记录，使用了vector
- 时间：O(n)，都是一遍扫描，空间：O(n)：用到了set，vector

```C++
class Solution {
public:
    bool buddyStrings(string A, string B) {
        int lenA = A.size(), lenB = B.size();
        if(lenA != lenB)
            return false;
        //这里我第一个想到的就是set，但是不知道怎么用，看了别人写的，简直太机智了
        if(A == B){
            if(set<char>(A.begin(), A.end()).size() < A.size()) 
                return true;//这里只判断了有重复元素的情况
        }
        vector<int> index;
        int i = 0; 
        while(i < lenA){
            if(A[i] != B[i])
                index.push_back(i);
            i++;
        }
        //size == 2这点是判断没有重复元素的情况
        return (index.size()==2) && (A[index[0]] == B[index[1]]) && (A[index[1]] == B[index[0]]);
    }
};
 
```

