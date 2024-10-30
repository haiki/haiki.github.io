---
title: java
date: 2019-10-10 14:04:57
categories:
- 编程语言
tags:
- java
---

- `int[] num = new int[0];` 开辟一个大小为 0（即 `num.length == 0` ） 的空间，可以在 `empty array`上遍历，但是不能在 `null` array 上遍历。
- `readline();` 参见脱坑指南 [被readLine()折腾了一把](<https://blog.csdn.net/swingline/article/details/5357581#commentBox>)
- `.txt` 文件一定要用 `notepad++` 看，用电脑自带的记事本，看起来不会空行之类的，所有数据堆成一片。
- [java Queue中 remove/poll, add/offer, element/peek区别](<https://blog.csdn.net/ustcjackylau/article/details/42454779>)
- [Java Math的 floor,round和ceil的总结](<https://blog.csdn.net/foart/article/details/4295645>)
- [java中Properties类的操作](<https://blog.csdn.net/u010255818/article/details/52733256>)
- [Java读取Properties文件的六种方法](<https://www.iteye.com/blog/tristan-wang-647729>)
- [HashMap 和 HashTable 的区别到底是什么？](<https://blog.csdn.net/u010983881/article/details/49762595>)
- ArrayList 和 LinkedList 的区别？
- [java中 .currentTimeMillis的用法和含义](https://blog.csdn.net/shlearry/article/details/50553986)

- [Class.forName和ClassLoader.loadClass](<https://www.iteye.com/blog/blackproof-1709336>)

- switch case中，在某个case中定义的变量，在其他case中可以直接拿来使用。return和break的区别是，前者直接返回，不执行这个函数下面的语句了。而后者是直接跳出当前的switch语句。

  ```java
  import java.io.*;
  import java.util.*;
  public class test{
  	public int[] testSwitch(int n){
  		switch(n){
  			case 1:
  			//boolean take = new boolean[10];
  			
  				int[] res = new int[10];
  				for(int i = 0; i < 5; i++)
  					res[i] = i;
  				return res;
  			case 2: 
  				res = new int[5];
  				for(int j = 0; j < 5; j++)
  					res[j] = j+10;
  				return res;
  		}
  		return null;
  	}
  	public static void main(String[] args){
  		test t = new test();
  		int[] res1 = new int[20];
  		res1 = t.testSwitch(2);
  		for(int k = 0; k < res1.length; k++)
  			System.out.println(res1[k]+'\n');		
  	}
  }
  ```

  

- [java使用FilenameFilter筛选文件](<http://outofmemory.cn/code-snippet/2099/java-usage-FilenameFilter-shaixuan-file>)
  - [stackoverflow](<https://stackoverflow.com/questions/19932962/use-of-filenamefilter>)

