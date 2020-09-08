<p data-source-line="3">写下这篇文章缘于我最近在使用Lua时，有一次在表达式中无意识地使用括号，这种对括号的泛滥使用是一种偷懒，我尝试回忆起学生时代中在程序设计语言的课程上的一个重要主题，即运算符优先级。本文针对一个简单的语句，在两个相似的程序设计语言中——一个高级语言Lua一个低级语言C——展开讨论，通过查询官方手册，我们很容易找到答案。</p>

<h2 id="lua中的运算符优先级" data-source-line="3">Lua中的运算符优先级</h2>
<p data-source-line="5">在Lua中写如下表达式</p>

<pre data-source-line="7"><code class="hljs"><span class="hljs-keyword">if</span> <span class="hljs-keyword">not</span> <span class="hljs-keyword">a</span> <span class="hljs-keyword">or</span> <span class="hljs-keyword">not</span> b</code></pre>
<p data-source-line="10">它应该等价于下列语句中的哪一种</p>

<pre data-source-line="11"><code class="hljs"><span class="hljs-keyword">if</span> (<span class="hljs-keyword">not</span> <span class="hljs-keyword">a</span>) <span class="hljs-keyword">or</span> (<span class="hljs-keyword">not</span> b)
<span class="hljs-keyword">if</span> (<span class="hljs-keyword">not</span> (<span class="hljs-keyword">a</span> <span class="hljs-keyword">or</span> (<span class="hljs-keyword">not</span> b))</code></pre>
<p data-source-line="16">通过参考文档：</p>

<blockquote data-source-line="18">Operator precedence in Lua follows the table below, from lower to higher priority:
<pre data-source-line="20"><code class="hljs">or
and
<span class="hljs-params"><     ></span>     <span class="hljs-params"><=    ></span>=    ~=    ==
|
~
&
<span class="hljs-params"><<    ></span>>
..
+     -
*     /     <span class="hljs-comment">//    %</span>
unary operators (not   <span class="hljs-meta">#     -     ~)</span></code></pre>
As usual, you can use parentheses to change the precedences of an expression. The concatenation ('..') and exponentiation ('^') operators are right associative. All other binary operators are left associative.

<a href="https://www.lua.org/manual/5.3/manual.html">https://www.lua.org/manual/5.3/manual.html</a></blockquote>
<p data-source-line="37">显然</p>

<pre data-source-line="39"><code class="hljs"><span class="hljs-keyword">if</span> <span class="hljs-keyword">not</span> <span class="hljs-keyword">a</span> <span class="hljs-keyword">or</span> <span class="hljs-keyword">not</span> b</code></pre>
<p data-source-line="43">等价于</p>

<pre data-source-line="45"><code class="hljs"><span class="hljs-keyword">if</span> <span class="hljs-comment">(not a)</span> <span class="hljs-keyword">or</span> <span class="hljs-comment">(not b)</span></code></pre>
<h2 id="c中的运算符优先级" data-source-line="49">C中的运算符优先级</h2>
<pre data-source-line="51"><code class="hljs"><span class="hljs-keyword">if</span> <span class="hljs-comment">(! a || ! b)</span></code></pre>
<p data-source-line="55">它应该等价于下列语句中的哪一种</p>

<pre data-source-line="57"><code class="hljs"><span class="hljs-keyword">if</span> <span class="hljs-comment">(! (a || !b)</span>
<span class="hljs-keyword">if</span> <span class="hljs-comment">((! a)</span> || <span class="hljs-comment">(! b)</span>)</code></pre>
<blockquote data-source-line="63">C 运算符的优先级和结合性将影响表达式中操作数的分组和计算。 仅当存在优先级较高或较低的其他运算符时，运算符的优先级才有意义。 首先计算带优先级较高的运算符的表达式。 也可以通过“绑定”一词描述优先级。优先级较高的运算符被认为具有更严格的绑定。 下表总结了 C 运算符的优先级和结合性（计算操作数的顺序），并按照从最高优先级到最低优先级的顺序将其列出。 如果几个运算符一起出现，则其具有相同的优先级并且将根据其结合性对其进行计算。 以后缀运算符开头的部分描述了表中的运算符。 此部分的其余部分提供了有关优先级和结合性的常规信息。</blockquote>
<blockquote data-source-line="66">C 运算符的优先级和结合性
<table>
<thead>
<tr>
<th>Symbol1</th>
<th>运算类型</th>
<th>结合性</th>
</tr>
</thead>
<tbody>
<tr>
<td>[ ] ( ) .–> 后缀 ++ 和后缀 ––</td>
<td>表达式</td>
<td>从左到右</td>
</tr>
<tr>
<td>前缀 ++ 和前缀 –– sizeof & * + – ~ !</td>
<td>一元</td>
<td>从右到左</td>
</tr>
<tr>
<td>typecasts</td>
<td>一元</td>
<td>从右到左</td>
</tr>
<tr>
<td>* / %</td>
<td>乘法</td>
<td>从左到右</td>
</tr>
<tr>
<td>+ –</td>
<td>加法</td>
<td>从左到右</td>
</tr>
<tr>
<td><< >></td>
<td>按位移动</td>
<td>从左到右</td>
</tr>
<tr>
<td>< > <= >=</td>
<td>关系</td>
<td>从左到右</td>
</tr>
<tr>
<td>== !=</td>
<td>相等</td>
<td>从左到右</td>
</tr>
<tr>
<td>&</td>
<td>按位“与”</td>
<td>从左到右</td>
</tr>
<tr>
<td>^</td>
<td>按位“异或”</td>
<td>从左到右</td>
</tr>
<tr>
<td>|</td>
<td>按位“与或”</td>
<td>从左到右</td>
</tr>
<tr>
<td>&&</td>
<td>逻辑“与”</td>
<td>从左到右</td>
</tr>
<tr>
<td>||</td>
<td>逻辑“或”</td>
<td>从左到右</td>
</tr>
<tr>
<td>?:</td>
<td>条件表达式</td>
<td>从右到左</td>
</tr>
<tr>
<td>= *= /= %= += –= <<= >>=&= ^= |=</td>
<td>简单和复合 assignment2</td>
<td>从右到左</td>
</tr>
<tr>
<td>,</td>
<td>顺序计算</td>
<td>从左到右</td>
</tr>
</tbody>
</table>
<ol>
 	<li>运算符按优先级的降序顺序列出。 如果多个运算符出现在同一行或一个组中，则它们具有相同的优先级。</li>
 	<li>所有简单的和复合的赋值运算符都有相同的优先级。</li>
</ol>
<a href="https://msdn.microsoft.com/zh-cn/library/2bxt6kc4.aspx">https://msdn.microsoft.com/zh-cn/library/2bxt6kc4.aspx</a></blockquote>
<p data-source-line="92">因为 ! 运算符优先级比 || 高</p>
<p data-source-line="94">所以</p>

<pre data-source-line="96"><code class="hljs"><span class="hljs-keyword">if</span> <span class="hljs-comment">(! a || ! b)</span></code></pre>
<p data-source-line="100">等价于</p>

<pre data-source-line="102"><code class="hljs"><span class="hljs-keyword">if</span> <span class="hljs-comment">((! a)</span> || <span class="hljs-comment">(! b)</span>)</code></pre>