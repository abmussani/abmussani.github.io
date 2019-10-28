---
layout: post
title:  "How to avoid Modulo Operators"
description: ""
date:   2018-04-15 20:25:15 +0500
categories: jekyll update
---
Mostly programmers are aware of modulo (%) and integer div (/) operators regardless of language they used. They are least used arithmetic operator in most programming languages. Modern off-the-shelf processors have built in instructions to perform such operations.

<strong>What is Modulo Operator:</strong>
<blockquote>In computing, the <b>modulo</b> operation finds the remainder after division of one number (<em>dividend</em>) by another (<em>divisor) -- <a href="https://en.wikipedia.org/wiki/Modulo_operation">Wikipedia</a></em></blockquote>
One common use case for this operator is to check if a number is even or odd. If remainder of a number is Zero when 2 divide it then it is an Even number else it is Odd.

<pre class="lang:c++ decode:true " >0 % 2 = 0
1 % 2 = 1
2 % 2 = 0
3 % 2 = 1
4 % 2 = 0
5 % 2 = 1</pre> 


The result of modulo operator can never be greater than divisor, it will always be one lesser then second number. The reason this happens is because division is all about fair sharing.

<strong>How to avoid Modulo operator:</strong>

I modern architecture, Division operator may take several clock cycles to perform its operation. Modulo operators can be easily translated into <em>Bitwise operators</em>. If a divisor is <em>power of 2</em> then it can be replaced by bitwise <em>AND</em> operator. It will take only one CPU cycle. for example:
<pre class="EnlighterJSRAW" data-enlighter-language="cpp">int remainder = value % 1024;</pre>
it can be translated into:
<pre class="EnlighterJSRAW" data-enlighter-language="cpp">int remainder = value &amp; 0x3FF;
int remainder = value &amp; (1024-1);</pre>
in general, If divisor n is power of 2, the modulo operators can be translated into bitwise AND with <em>divisor - 1</em>.

<strong>What is the real impact of modulo operator:</strong>

Marco Ziccardi wrote a program to answer this question:
<pre class="EnlighterJSRAW" data-enlighter-language="cpp">int main(int argc, char** argv) { 
    int remainder;
    int value = 1301081;
    int divisor = 1024;
    int i = 0;
    while (i &lt; 10000000) {
        remainder = value % divisor;
        i++;
    }
}</pre>
And compared its execution times with the same program implemented using binary AND (&amp;) operator. Results are shown below (1000 runs of the program for both % and &amp; versions):

<img class="size-medium aligncenter" src="http://mziccard.me/public/images/MODvsANDresults.png" width="842" height="418" />

As you can see, in the average case the program with the modulo operation is 62% slower than the one using &amp;  operator.

<strong>Conclusion:</strong>

It is not always possible to avoid modulo operator in the favor of bitwise.