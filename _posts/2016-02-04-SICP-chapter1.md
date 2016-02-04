---
layout:     post
title:      "A Review of SICP Chapter1"
date:       2016-02-04 17:12:00
author:     "William"
header-img: "img/post-bg-sicp-chp1.jpg"
excerpt_separator: <!--more-->
tags:
    - Scheme
    - 寒假
    - SICP
    - 读书笔记
    - Functional Programming
---

> "A computational process is indeed much like a sorcerer's idea of a spirit. "
<!--more-->

刚刚看完SICP第一章，Building Abstractions with Procedures.

 <a href="https://github.com/willx8/sicp-log/blob/master/chp1/SICP-C1.scm">这里是我的第一章<s>跳了不少题的</s>题解。</a>

写一篇读书笔记，算是让自己回顾一下。只提几个有趣之处。

<s>实在找不到好看的题图就只好自己做了张。</s>

# 1.Evaluation
Exercise 1.5可以看到两种方式(applicative-order evaluation & normal-order)的区别，为了避免一句话里中英夹杂让人不爽，下面有些解答直接用英文了。

### Exercise 1.5.
Ben Bitdiddle has invented a test to determine whether the interpreter he is faced with is using applicative-order evaluation or normal-order evaluation. He defines the following two procedures:

```scheme
(define (p) (p))

(define (test x y)
  (if (= x 0)
      0
      y))
```

Then he evaluates the expression

```scheme
(test 0 (p))
```

What behavior will Ben observe with an interpreter that uses applicative-order evaluation? What behavior will he observe with an interpreter that uses normal-order evaluation? Explain your answer. (Assume that the evaluation rule for the special form if is the same whether the interpreter is using normal or applicative order: The predicate expression is evaluated first, and the result determines whether to evaluate the consequent or the alternative expression.)

### Solution 1.5.
* If an interpreter uses normal-order evaluation:
`(test 0 (p))` will be evaluated `(if (= 0 0) 0 p)`. Then "the predicate expression is evaluated first", thus `(= 0 0)` generate true, the result shall be `0`.
* If an interpreter uses applicative-order evalution:
`(test 0 (p))` will be `(test 0 (p))`, intepreter 'substitute' `(p)` for `(p)` forever, the answer will never be genearted.

---

下面关于new-if的题可以和上面这道连起来看：

### Exercise 1.6.
Alyssa P. Hacker doesn't see why if needs to be provided as a special form. ``Why can't I just define it as an ordinary procedure in terms of cond?'' she asks. Alyssa's friend Eva Lu Ator claims this can indeed be done, and she defines a new version of if:

```scheme
(define (new-if predicate then-clause else-clause)
  (cond (predicate then-clause)
        (else else-clause)))
```

Eva demonstrates the program for Alyssa:

```scheme
(new-if (= 2 3) 0 5)
5

(new-if (= 1 1) 0 5)
0
```

Delighted, Alyssa uses new-if to rewrite the square-root program:

```scheme
(define (sqrt-iter guess x)
  (new-if (good-enough? guess x)
          guess
          (sqrt-iter (improve guess x)
                     x)))
```
What happens when Alyssa attempts to use this to compute square roots? Explain.

### Solution 1.6.
Since our interpreter uses applicative-order evaluation, new-if is interpreted later than \<else-clause\>. The interpreter evaluates \<predicate expression\>. Then it evaluates \<else-clause\>: we have `(improve guess x)` evaluated first, call it `y1` , then `sqrt-iter` evaluated:

```scheme
(define (sqrt-iter guess x)
  (new-if <some-evaluated-value>
          guess
          ((new-if (good-enough? y0 x)
          		guess
	          	(sqrt-iter (improve y0 x)
                     		x)))))
```
Since no branch decision is made due to the late-evaluation of `new-if`, there will be an infinite substition of `sqrt-iter`.

---

# 2.花式斐波那契
1.2.4的练习题很巧妙
一步一步地引到1.19上，教你把一次求斐波那契数列下一项的过程看做一次变换*T*，如果找到等价于运用两次变换*T*的变换*T'*，由于找到了*T*的“平方”，就可以像快速幂一样用O(log n)的速度求出答案。而*T*实际就是矩阵乘法，所谓两次*T*变换就是矩阵的平方。所以这题也可以看成是矩阵乘法的“快速幂”。看题：

### Exercies 1.19.

There is a clever algorithm for computing the Fibonacci numbers in a logarithmic number of steps.

Recall the transformation of the state variables *a* and *b* in the <tt>fib-iter</tt> process of section 1.2.2: *a* <- <em>a</em> + <em>b</em> and <em>b</em> <-<em>a</em>.  Call this transformation <em>T</em>, and observe that applying <em>T</em> over and over again <em>n</em> times, starting with 1 and 0, produces the pair <em>F</em><em>i</em><em>b</em>(*n + 1*) and  <em>F</em><em>i</em><em>b</em>(<em>n</em>).  In other words, the Fibonacci numbers are produced by applying <em>T</em><sup><em>n</em></sup>, the <em>n</em>th power of the transformation <em>T</em>, starting with the pair (1,0).  Now consider <em>T</em> to be the special case of <em>p</em> = 0 and <em>q</em> = 1 in a family of transformations <em>T</em><sub><em>p</em><em>q</em></sub>, where <em>T</em><sub><em>p</em><em>q</em></sub> transforms the pair *(a, b)* according to <em>a</em> <- <em>b</em><em>q</em> + <em>a</em><em>q</em> + <em>a</em><em>p</em> and <em>b</em> <- <em>b</em><em>p</em> + <em>a</em><em>q</em>. 

Show that if we apply such a transformation <em>T</em><sub><em>p</em><em>q</em></sub> twice, the effect is the same as using a single transformation *T<sub>p'q'</sub>* of the same form, and compute <em>p</em>' and <em>q</em>' in terms of <em>p</em> and <em>q</em>.  This gives us an explicit way to square these transformations, and thus we can compute <em>T</em><sup><em>n</em></sup> using successive squaring, as in the <tt>fast-expt</tt> procedure.  Put this all together to complete the following procedure, which runs in a logarithmic number of steps:

```scheme
(define (fib n)
  (fib-iter 1 0 0 1 n))
(define (fib-iter a b p q count)
  (cond ((= count 0) b)
        ((even? count)
         (fib-iter a
                   b
                   <??>      ; compute p'
                   <??>      ; compute q'
                   (/ count 2)))
        (else (fib-iter (+ (* b q) (* a q) (* a p))
                        (+ (* b p) (* a q))
                        p
                        q
                        (- count 1)))))
```

### Solution 1.19.

Notice that:

<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
<mrow><mfenced open='(' close=')'><mtable><mtr><mtd><mrow><mi>p</mi><mo>+</mo><mi>q</mi></mrow></mtd><mtd><mrow><mi>q</mi></mrow></mtd></mtr><mtr><mtd><mrow></mrow></mtd></mtr><mtr><mtd><mrow></mrow></mtd></mtr><mtr><mtd><mrow><mi>r</mi><mi>q</mi></mrow></mtd><mtd><mrow><mi>p</mi></mrow></mtd></mtr></mtable></mfenced><mfenced><mfrac linethickness="0"><mrow><mrow><msub><mrow><mi>a</mi></mrow><mrow><mrow><mi>n</mi></mrow></mrow></msub></mrow></mrow><mrow><mrow><msub><mrow><mi>b</mi></mrow><mrow><mrow><mi>n</mi></mrow></mrow></msub></mrow></mrow></mfrac></mfenced><mo>=</mo><mfenced><mfrac linethickness="0"><mrow><mrow><msub><mrow><mi>a</mi></mrow><mrow><mrow><mi>n</mi></mrow></mrow></msub><mo>(</mo><mi>p</mi><mo>+</mo><mi>q</mi><mo>)</mo><mo>+</mo><msub><mrow><mi>b</mi></mrow><mrow><mrow><mi>n</mi></mrow></mrow></msub><mi>q</mi></mrow></mrow><mrow><mrow><msub><mrow><mi>a</mi></mrow><mrow><mrow><mi>n</mi></mrow></mrow></msub><mi>q</mi><mo>+</mo><msub><mrow><mi>b</mi></mrow><mrow><mrow><mi>n</mi></mrow></mrow></msub><mi>p</mi></mrow></mrow></mfrac></mfenced><mo>=</mo><mfenced><mfrac linethickness="0"><mrow><mrow><msub><mrow><mi>a</mi></mrow><mrow><mrow><mi>n</mi><mo>+</mo><mn>1</mn></mrow></mrow></msub></mrow></mrow><mrow><mrow><msub><mrow><mi>b</mi></mrow><mrow><mrow><mi>n</mi><mo>+</mo><mn>1</mn></mrow></mrow></msub></mrow></mrow></mfrac></mfenced></mrow></math>

Which indicates the transformation *T* can be regard as matrix multiplication, or linear transformation. Thus applying *T* twice is merely square a matrix:

<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
<mrow><msup><mrow><mfenced open='(' close=')'><mtable><mtr><mtd><mrow><mi>p</mi><mo>+</mo><mi>q</mi></mrow></mtd><mtd><mrow><mi>q</mi></mrow></mtd></mtr><mtr><mtd><mrow></mrow></mtd></mtr><mtr><mtd><mrow></mrow></mtd></mtr><mtr><mtd><mrow><mi>r</mi><mi>q</mi></mrow></mtd><mtd><mrow><mi>p</mi></mrow></mtd></mtr></mtable></mfenced></mrow><mrow><mrow><mn>2</mn></mrow></mrow></msup><mo>=</mo><mfenced open='(' close=')'><mtable><mtr><mtd><mrow><msup><mrow><mi>p</mi></mrow><mrow><mrow><mn>2</mn></mrow></mrow></msup><mo>+</mo><mn>2</mn><mi>p</mi><mi>q</mi><mo>+</mo><mn>2</mn><msup><mrow><mi>q</mi></mrow><mrow><mrow><mn>2</mn></mrow></mrow></msup></mrow></mtd><mtd><mrow><mn>2</mn><mi>p</mi><mi>q</mi><mo>+</mo><msup><mrow><mi>q</mi></mrow><mrow><mrow><mn>2</mn></mrow></mrow></msup></mrow></mtd></mtr><mtr><mtd><mrow></mrow></mtd></mtr><mtr><mtd><mrow></mrow></mtd></mtr><mtr><mtd><mrow><mi>r</mi><mn>2</mn><mi>p</mi><mi>q</mi><mo>+</mo><msup><mrow><mi>q</mi></mrow><mrow><mrow><mn>2</mn></mrow></mrow></msup></mrow></mtd><mtd><mrow><msup><mrow><mi>p</mi></mrow><mrow><mrow><mn>2</mn></mrow></mrow></msup><mo>+</mo><msup><mrow><mi>q</mi></mrow><mrow><mrow><mn>2</mn></mrow></mrow></msup></mrow></mtd></mtr></mtable></mfenced></mrow></math>

We derive a new matrix in the same form of the former one, where

*p' = p<sup>2</sup>+q<sup>2</sup>, q' = 2pq+q<sup>2</sup>,*

which is the answer we are looking for.

---

# 3. Fermat Test
这部分用费马小定理做质数检验的内容挺有趣。这是一个把指数检验的时间复杂度从O(n<sup>1/2</sup>)降到O(log n)的*probabilistic methods*. 注释47的一段话很有意思：
> In testing primality of very large numbers chosen at random, the chance of stumbling upon a value that fools the Fermat test is less than the chance that cosmic radiation will cause the computer to make an error in carrying out a "correct" algorithm. Considering an algorithm to be inadequate for the first reason but not for the second illustrates the difference between mathematics and engineering.

---

# 4. Lambda大法好
1.3节大概是真的开始函数式编程了。这一节每组题基本都层层递进，不断往高层次抽象过程，很有趣，像我这样喜欢跳题的全做了，有兴趣可以看我的题解。两道题值得再提一下。

### Exercise 1.41.  

Define a procedure double that takes a procedure of one argument as argument and returns a procedure that applies the original procedure twice. For example, if inc is a procedure that adds 1 to its argument, then (double inc) should be a procedure that adds 2. What value is returned by`(((double (double double)) inc) 5)`

### Solution 1.41: 

```scheme
(define (double f)
  (lambda (i) (f (f i))))
```
21

### 补充

值得注意的是语法。我们会发现这句话`(((double (double double)) inc) 5)`其实和下面是等价的：

```scheme
((((lambda (i) (double (double i)))
 double) inc)
 5)
```
而`inc`竟然能这样作为参数被进新的procedure`(double (double double))`中。我觉得，毕竟还是第一章，既然书里也没进一步解释，**just remember that this patterns works.**等后面进一步讲语法的时候再来看解释器的具体实现吧，那么这里就要挖一个坑等以后填了。

---

### Exercise 1.45.  
We saw in section 1.3.3 that attempting to compute square roots by naively finding a fixed point of y->x/y does not converge, and that this can be fixed by average damping. The same method works for finding cube roots as fixed points of the average-damped y->x/y<sup>2</sup>. Unfortunately, the process does not work for fourth roots -- a single average damp is not enough to make a fixed-point search for y->x/y<sup>3</sup> converge. On the other hand, if we average damp twice (i.e., use the average damp of the average damp of y->x/y<sup>3</sup>) the fixed-point search does converge. Do some experiments to determine how many average damps are required to compute nth roots as a fixed-point search based upon repeated average damping of y->x/y<sup>n-1</sup>. Use this to implement a simple procedure for computing nth roots using `fixed-point`, `average-damp`, and the `repeated` procedure of exercise 1.43. Assume that any arithmetic operations you need are available as primitives.

### Solution 1.45.

```scheme  
(define (repeated f times)
  (lambda (i)
    (if (<= times 0)
        i
        ((composition f (repeated f (- times 1))) i))))

(define (n-th-root i n)
  (fixed-point
   ((lambda (x)
      ((repeated average-damp (/ (log n) (log 2)))
       (lambda (y) (/ x (fast-expt y (- n 1))))))
    i) 1.0))
```
*`fixed-point` & `average-damp` is omitted.*

After some experiments, the times of average-damp required is like:

* 2 ~ 3:  	1
* 4 ~ 7:  	2
* 8 ~ 15: 	3
* ...
* 2<sup>n</sup> ~ 2<sup>n+1</sup>-1: n

Which leads to *times = log<sub>2</sub>n*

Notice that by using `(<= times 0)` instead of `(= times 0)` in the *predicate expression* in `repeated`, rounding log<sub>2</sub>n can be avoided.

---

# END
**希望我的小喵会平安地飞回来。**
