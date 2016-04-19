
---
title: 【翻译】Java SE 5.0 增强的语法特性 自动装箱 
tags: Java SE 5.0 增强的语法特性,自动装箱
categories: Java基础
src: http://docs.oracle.com/javase/8/docs/technotes/guides/language/autoboxing.html
---

<style type="text/css">
    .dn {
        display: none;
    }
</style>
<script type="application/javascript">
			function showOriginalText() {
				var doms = getByClass(document, "original-text");
				if (doms && doms.length > 0) {
					for (var index in doms) {
						removeClass(doms[index], "dn"); 
					}
				}
			}
			function hasClass(dom, className){
				if(dom && className && dom.className){
					return dom.className.match(new RegExp('(\\s|^)' + className + '(\\s|$)'));	
				}else {
					return false; 
				}
			}
			function removeClass(dom, className){
				if (hasClass(dom, className)) {  
			        var reg = new RegExp('(\\s|^)' + className + '(\\s|$)');  
			        dom.className = dom.className.replace(reg, ' ');  
			    }  
			}
			function addClass(dom, className){
				 if(dom && className){
				 	if (!hasClass(dom, className)) dom.className += " " + className;	
				 }
			}
			function getByClass(dom, className) {
				if (dom && className && dom.getElementsByTagName) {
					var elements = dom.getElementsByTagName("*");
					if (elements != null && elements.length && elements.length > 0) {
						var result = [];
						for (var i = 0; i < elements.length; i++) {
							if (hasClass(elements[i], className)) {
								result.push(elements[i]);
							}
						}
						return result;
					} else {
						return null;
					}
				} else {
					return null;
				}
			}
</script>
# **自动装箱/自动拆箱**

Java开发者都知道不能将int（或者其他基本类型）直接put到集合类中。集合只能持有对象的引用，所以必须将原始类型的值装箱到对应的包装类中（int对应的包装类是Integer）.当你从集合中获取对象，你得到的是你put进去的Integer，如果你需要的是int类型，你需要使用intValue方法将Integer对象拆箱。所有的装箱和拆箱代码的编写是一种痛苦的事情，而且干扰正常业务逻辑的代码。自动装箱、拆箱特性让装箱和拆箱过程自动化，消除编码的痛苦和对代码的干扰。

下面的例子说明了自动装箱/拆箱，顺带演示泛型和for-each循环增强。仅用10行代码，计算并按字母排序打印命令行输入的单词使用频度:

``` Java
import java.util.*;

//Prints a frequency table of the words on the command line
public class Frequency {
	 public static void main(String[] args) {
	      Map<String, Integer> m = new TreeMap<String, Integer>();
	      for (String word : args) {
	          Integer freq = m.get(word);
	          m.put(word, (freq == null ? 1 : freq + 1)); //highlighted
	      }
	      System.out.println(m);
	   }
}

```
``` Bash 

java Frequency if it is to be it is up to me to do the watusi
{be=1, do=1, if=1, is=2, it=2, me=1, the=1, to=3, up=1, watusi=1}

```

程序首先定义一个map,从String类型映射到Integer类型，代表命令行输入的词与其出现次数的关系。然后遍历命令行输入的每一个词，查找map并修改每个词的出现次数。注释highlighted的代码行包含了自动装箱和拆箱操作。为了计算单词对应的值，首先需要查看当前的值(freq)。如果值为null， 表明是第一次出现该词，所以将1作为value存入map中。否则，将词当前值加上1，然后存入map中。但是，你当然不能将int类型的值作为value存入map中，也不能将一个Integer对象加1。事实上是：为了将freq加1，必须将freq自动拆箱，得到类型int的值，才能让加号操作符的表达式结果为int类型。这样，三目运算符条件表达式的二选一表达式的结果值才能都是int类型，这也是条件表达式的结果类型。为了将int类型的值存入map中，还需要将值自动装箱成Integer类型的对象。

The result of all this magic is that you can largely ignore the distinction between int and Integer, with a few caveats. An Integer expression can have a null value. If your program tries to autounbox null, it will throw a NullPointerException. The == operator performs reference identity comparisons on Integer expressions and value equality comparisons on int expressions. Finally, there are performance costs associated with boxing and unboxing, even if it is done automatically.

Here is another sample program featuring autoboxing and unboxing. It is a static factory that takes an int array and returns a List of Integer backed by the array. In a mere ten lines of code this method provides the full richness of the List interface atop an int array. All changes to the list write through to the array and vice-versa. The lines that use autoboxing or unboxing are highlighted in green:

``` Java 
// List adapter for primitive int array
public static List<Integer> asList(final int[] a) {
    return new AbstractList<Integer>() {
        public Integer get(int i) { return a[i]; }
        // Throws NullPointerException if val == null
        public Integer set(int i, Integer val) {
            Integer oldVal = a[i];
            a[i] = val;
            return oldVal;
        }
        public int size() { return a.length; }
    };
}
``` 
The performance of the resulting list is likely to be poor, as it boxes or unboxes on every get or set operation. It is plenty fast enough for occasional use, but it would be folly to use it in a performance critical inner loop.

So when should you use autoboxing and unboxing? Use them only when there is an "impedance mismatch" between reference types and primitives, for example, when you have to put numerical values into a collection. It is not appropriate to use autoboxing and unboxing for scientific computing, or other performance-sensitive numerical code. An Integer is not a substitute for an int; autoboxing and unboxing blur the distinction between primitive types and reference types, but they do not eliminate it.
