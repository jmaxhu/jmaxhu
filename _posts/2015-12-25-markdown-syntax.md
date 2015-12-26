---
layout: post
title: Markdown 语法
tags: markdown
---

#Markdown: 语法

* [概述](#overview)
  * [哲学](#philosophy)
  * [行内 HTML](#html)
  * [特殊字符自动置换](#autoescape)
* [块元素](#block)
  * [段落和换行](#p)
  * [标题](#header)
  * [块引用](#blockquote)
  * [列表](#list)
  * [程序块](#precode)
  * [分隔线](#hr)
* [区段元素](#span)
  * [链接](#link)
  * [强调](#em)
  * [程序](#code)
  * [图片](#img)
* [其它](#misc)
  * [转义字符](#backslash)
  * [自动链接](#autolink)

* * *

<h2 id="overview">概述</h2>

<h3 id="philosophy">哲学</h3>

Markdown 的目标是实现易读最写。

不过最需要强调的是它的可读性。一份使用Markdown格式写的文件应该可以直接以纯文本发布，并且看盐业不会像是由许多标签或格式指令所构成。Markdown语法受到一些既有text-to-HTML格式的影响，包含[Setext][1]、[atx][2]、[Textile][3]、[reStructuredText][4]、[Grutatext][5]和[EtText][6]，然而最大的灵感来源其实是纯文本的电子邮件格式。

  [1]: http://docutils.sourceforge.net/mirror/setext.html
  [2]: http://www.aaronsw.com/2002/atx/
  [3]: http://textism.com/tools/textile/
  [4]: http://docutils.sourceforge.net/rst.html
  [5]: http://www.triptico.com/software/grutatxt.html
  [6]: http://ettext.taint.org/doc/

因此 Markdown 的语法由标点符号所组合，并经过谨慎选择，是为了让它们看来就像所要表达的意思。像是在文字两旁加上星号，看起来像是\*强调\*。Markdown的列表看起来，嗯，就是列表。假如你有使用过电子邮件，块引用看起来就真的像是引用一段文字。

<h3 id="html">行内HTML</h3>

Markdown的语法有个主要的目的：用来作为一种网络内容的*写作*语言。

Markdown不是要来取代HTML， 甚至也没有要和它相似，它的语法种类不多，只和HTML的一部分有关系，重点*不是*要创建一种更容易写HTML文件的语法，我认为HTML已经很容易了，Markdown的重点在于，它能让文件更容易阅读、编写。HTML是一种*发布*格式，Markdown是一种*编写*格式，因此，Markdown的格式语法只涵盖纯文本可以涵盖的范围。

不在Markdown涵盖范围之外的标签，都可以直接在文件里用HTML撰写。不需要额外标注这是HTML或是Markdown；只要直接加标签就可以了。

只有块元素比如`<div>`,`<table>`,`<pre>`,`<p>`等标签，必须在前后加上空行，以利于内容分隔。而且这些（元素）的开始与结尾标签，不可以用tab或都空白来缩排。Markdown的产生器有智能判断，可以避免在块标签前后加上没有必要的`<p>`标签。

举例来说，在Markdown文件里加上一段HTML表格：

    This is a regular paragraph.

    <table>
        <tr>
            <td>Foo</td>
        </tr>
    </table>

    This is another regular paragraph.

请注意，Markdown语法在HTML块标签中将不会被处理。例如，你无法在HTML块中使用Markdown形式的`*强调*`。

HTML的区段标签如`<span>`,`<cite>`,`<del>`则不受限制，可以在Markdown的段落、列表或标题里任意使用。按使用人的习惯，甚至可以不用Markdown格式，而采用HTML标签来格式化。举例说明：如果比较喜欢HTML的`<a>`或`<img>`标签，可以直接使用这些标签，而不用Markdown提供的链接或图片语法。

HTML区段标签或块标签不同，在区段标签的范围内，Markdown的语法是有效的。

<h3 id="autoescape">特殊字符自动转换</h3>

在HTML文件中，有两个字符需要特殊处理：`<`和`&`。`<`符号用于起始标签，`&`符号则用于标记HTML实体，如果你只是想要使用这些符号，你必须要使用实体的形式，像是`&lt;`和`&amp;`。

`&`符号其实很容易让写网络文件的人感到困扰，如果你要打「AT&T」，你必须要写成`「AT&amp;T」`，还得转换网址内的`&`符号，如果你要链接到：

    http://images.google.com/images?num=30&q=larry+bird

你必须要把网址转成：

    http://images.google.com/iamges?num=30&amp;q=larry+bird

才能放到链接标签的`href`属性里。不用说也知道这很容易忘记，这也可能是HTML标准检查所查到错误中，数量最多的。

Markdown允许你直接使用这些符号，但是你要小心转义字符的使用，如果你是在HTML实体中使用`&`符号的话，它不会被转换，而在其它情况下，它则会被转换成`&amp;`。所以你如果要在文件中插入一个著作权的符号，你可以这样写：

    &copy;

Markdown将不会对这段文字做修改，但是如果你这样写：

    AT&T

Markdown就会将它转为：

    AT&amp;T

类似的情况也会发生在`<`符号上，因为Markdown支持[行内HTML](#html)，如果你是使用`<`符号作为HTML标签使用，那Markdown也不会对它做任何转换，但是如果你是写：

    4 < 5

Markdown将会把它转换为：

    4 &lt; 5

不过需要注意的是，code范围内，不论是行内还是块，`<`和`&`两个符号者*一定*会被转换成HTML实体，这项特性让你可以很容易地用Markdown写HTML code（和HTML相对而言，HTML语法中，你要把所有的`<`和`&`都转换为HTML实体，才能在HTML文件里写出HTML code。）

* * *

<h2 id="block">块元素</h2>

<h3 id="p">段落和换行</h3>

一个段落是由一个以上相连接的语句组合，而一个以上的空行则会切分出不同的段落（空行的定义是显示上看起来像是空行，便会被视为空行。比方说，若某一行只包含空白和tab，则该行也会被视为空行），一般的段落不需要用空白或断行缩排。

「一个以上相连接的语句组合」这名话其它暗示了Markdown允许段落内的强迫断行，这个特性和其它大部发的text-to-HTML格式不一样（包含MovableType）和「Convert Line Breaks」选择），其它的格式会把每个断行都转成`<br />`标签。

如果你*直的*想要插入`<br />`标签的话，在行尾加上两个以上的空白，然后按enter。

是的，这确实需要花比较多的功夫来插入`<br />`，但是「每个换行都转换为`<br />`的方法在Markdown中并不适合，Markdown中email式的[块引用][bq]和多段落的[列表][l]在使用换行来排版的时候，不但更好用，还更好阅读。

  [bq]: #blockquote
  [l]:  #list

<h3 id="header">标题</h3>

Markdown支持两种标题的语法，[Setext][1]和[atx][2]形式。

Setext形式是用底线的形式，利用`=`（最高阶标题）和`-`（第二阶标题），例如：

    This is an H1
    =============

    This is an H3
    -------------

任何数量的`=`和`-`都可以有效果。

Atx形式则在行首插入1到6个`#`，对应到标题1到6阶，使用：

    # This is an H1

    ## This is an H2

    ###### This is an H6

你可以选择性的「关闭」atx样式的标题，这纯粹只是美观用的，若是觉得这样看起来比较舒适，你就可以在行尾加上`#`，而在行尾的`#`数量也不用和开头一样（行首的井字数量决定标题的阶数）：

    # This is an H1 #

    ## This is an H2 ##

    ### This is an H3 ######

<h3 id="blockquote">块引用</h3>

Markdown使用email形式的块引用，如果你很熟悉如何在email信件中引用，你就知道怎么在Markdown文件中建立一个块引用，那会看起来像是你强迫断行，然后在每行的最前面加上`>`：

    > this is a blockquote with two paragraphs. lorem ipsum dolor sit amet,
    > consectetuer adipiscing elit. aliquam hendrerit mi posuere lectus.
    > vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
    >
    > dones sit amet nisl. aliquam semper ipsum sit amet velit. suspendisse
    > is sem consectetuer libero luctus adipiscing.

Markdown也允许你只在整个段落的第一行前面加上`>`：

    > This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
  consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
  Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.

    > Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
  id sem consectetuer libero luctus adipiscing.

块引用可以有阶层（例如：引用内的引用），只要根据层数加上不同数量的`>`：

    > This is the first level of quoting.
    >
    >> This is nested blockquote.
    >
    > Back to the first level.

引用的块内也可以使用其他的Markdown语法，包括标题、列表、程序块等：

    > ## This is a header.
    >
    > 1.  This is the first list item.
    > 2.  This is the second list item.
    >
    > Here's some example code:
    >
    >   return shell_exec("echo $input | $markdown_script");

任何标准的文字编辑器都能简单地建立email格式的引用， 例如BBEdit，你可以选取文字然后选择*增加引用阶层*。

<h3 id="list">列表</h3>

Markdown支持有序列表和无序列表。

无序列表使用星号、加号或是减号作为列表标记。

    *   Red
    *   Green
    *   Blue

等同于：

    +   Red
    +   Green
    +   Blue

也等同于：

    -   Red
    -   Green
    -   Blue

有序列表则使用数字接着一个英文句点：

    1.  Bird
    2.  McHale
    3.  Parish

很重要的一点是，你在列表标记上五个人的数字并不会影响输出的HTML结果，上面的列表所产生的HTML标记为：

    <ol>
    <li>Bird</li>
    <li>McHale</li>
    <li>Parish</li>
    </ol>

如果你的列表标签写成：

    1.  Bird
    1.  McHale
    1.  Parish

或甚至是：

    3. Bird
    1. McHale
    8. Parish

你都会得到完全相同的HTML输出。重点在于，你可以让Markdown文件的列表数字和输出的结果相同。或是你懒一点，你可以完全不用在意数字的正确性。

如果你使用懒惰的写法，建议第一个项目最好还是从1.开始，因为Markdown未来可以会支持有序列表的start属性。

  列表项目标记通常是放在最左边，但是其实也可缩排，最多三个空白，项目标记后面则一定要接着至少一个空白或tab。

    *   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
        Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
        viverra nec, fringilla in, laoreet vitae, risus.
    *   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
        Suspendisse id sem consectetuer libero luctus adipiscing.

但是如果你很懒，那也不一定需要：

    *   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
    Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
    viverra nec, fringilla in, laoreet vitae, risus.
    *   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
    Suspendisse id sem consectetuer libero luctus adipiscing.

如果列表项目间用空行分开，Markdown会把项目的内容在输出时用`<p>`标签包起来，举例来说：

    *   Bird
    *   Magic

会被转换为：

    <ul>
    <li>Bird</li>
    <li>Magic</li>
    </ul>

但是这个：

    *   Bird

    *   Magic

会被转换为：

    <ul>
    <li><p>Bird</p></li>
    <li><p>Magic</p></li>
    </ul>

列表项目可以包含多个段落，每个项目下的段落都必须缩排4个空白或一个tab：

    1.  This is a list item with two paragraphs. Lorem ipsum dolor
        sit amet, consectetuer adipiscing elit. Aliquam hendrerit
        mi posuere lectus.

        Vestibulum enim wisi, viverra nec, fringilla in, laoreet
        vitae, risus. Donec sit amet nisl. Aliquam semper ipsum
        sit amet velit.

    2.  Suspendisse id sem consectetuer libero luctus adipiscing.

如果你每行都有缩排，看起来会好看很多，当然，再次地，如果你很懒惰，Markdown也允许：

    *   This is a list item with two paragraphs.

        This is the second paragraph in the list item. You're
    only required to indent the first line. Lorem ipsum dolor
    sit amet, consectetuer adipiscing elit.

    *   Another item in the same list.

如果要放程序块的话，该块就需要缩排*两次*，也就是8个空白或是两个tab：

    *   A list item with a code block:

            <code goes here>

当然，项目列表可能会不小心产生，像是下面这样的写法：

    1986\. What a great season.

<h3 id="precode">程序块</h3>

和程序相关的写作或是标签语言原始码通常会有已经排版好的程序块，通过这些块我们并不希望它以一般段落文件的方式去排版，而是按照原来的样式显示，Markdown会用`<pre>`和`<code>`标签来把程序块包起来。

要在Markdown中建立程序块很简单，只要简单地缩排4个空白或是1个tab就可以，例如，下面的输入：

    This is a normal paragraph:

        This is a code block.

Markdown会转换成：

    <p>This is a normal paragraph:</p>

    <pre><code>This is a code block.
    </code></pre>

这个第行一阶的缩排（4个空白或是1个tab），都会被移除，例如：

    Here is an example of AppleScript:

        tell application "Foo"
            beep
        end tell

会被转换为：

    <p>Here is an example of AppleScript:</p>

    <pre><code>tell application "Foo"
        beep
    end tell
    </code></pre>

一个程序块会一直持续到没有缩排的那一行（或是文件结尾）。

在程序块里面，`&`，`<`和`>`会自动转成HTML实体，这样的方式让你非常容易使用Markdown插入范例用的HTML原始码，只需要复制粘贴，再加上缩排就可以了，剩下的Markdown都会帮你处理，例如：

         <div class="footer">
            &copy; 2004 Foo Corporation
        </div>

会被转换为：

    <pre><code>&lt;div class="footer"&gt;
        &amp;copy; 2004 Foo Corporation
    &lt;/div&gt;
    </code></pre>

程序块中，一般的Markdown语法不会被转换，像是星号便只是星号，这表示你可以很容易地以Markdown语法撰写Markdown语法相关的文件。

<h3 id="hr">分隔线</h3>

你可以在一行中用三个或以上的星号、减号、底线来产生一个分隔线，行内不能有其他东西。你也可以在星号中间插入空白。下面每种写法都可以产生分隔线。

    * * *

    ***

    ******

    - - -

    --------------------------------

* * *

<h2 id="span">区段元素</h2>

<h3 id="link">链接</h3>

Markdown支持两种形式的链接语法：*行内*和*参考*两种形式。

不管是哪一种，链接的文字都是用［方括号]来标记。

要产生一个行内形式的链接，只要在方括号后面马上接着括号并插入网址链接即可，如果你还想加上链接的title文字，呆要在网址后面，用双引号把title文字包起来即可，例如：

    This is [an example](http://example.com/ "Title") inline link.

    [This link](http://example.com) has no title attribute.

会产生：

    <p>This is <a href="http://example.com/" title="Title">
    an example</a> inline link.</p>

    <p><a href="http://example.net/">This link</a> has no
    title attribute.</p>

如果你是要链接到同样主机的资源，你可以使用相对路径：

    See my [About](/about/) page form details.

参考形式的链接使用另外一个方括号接在链接文字的括号后面，而在第二个方括号里面要填入用以辨识链接的标签：

    This is [an example][id] reference-style link.

你也可以选择性的在两个方括号中间加上空白：

    This is [an example] [id] reference-style link.

接着，在文件的任意处，你可以把这个标签的链接内容定义出来：

    [id]: http://example.com/  "Optional Title Here"

链接定义的形式为：

*  方括号，里面输入链接的辨识用标签
*  接着一个冒号
*  接着一个以上的空白或tab
*  接着链接的网址
*  选择性的接着 title 内容，可以用单引号、双引号或是括号包着

下面这三种链接的定义都是相同：

    [foo]: http://example.com/  "Optional Title Here"
    [foo]: http://example.com/  'Optional Title Here'
    [foo]: http://example.com/  (Optional Title Here)

**请注意：＊* 有一个已知的问题是 Markdown.pl 1.0.1 会忽略单引号包起来的链接 title。
链接网址也可以用方括号包起来：

    [id]: <http://example.com/> "Optional Title Here"

你也可以把title属性放到下一行，也可以加一些缩排，网址太长的话，这样会比较好看：

    [id]: http://example.com/longish/path/to/resource/here
        "Optional Title Here"

网址定义只有在产生链接的时候用到，并不会直接出现在文件之中。

链接辨识标签可以有字母、数字、空白和标点符号，但是并*不*区分大小写，因此下面几个链接是一样的：

    [link text][a]
    [link text][A]

*预设的链接标签*功能让你可以省略指定链接标签，这种情形下，链接标签和链接文字会视为相同，要用预设链接标签只要在链接文字后面加上一个空的方括号，如果你要让“Google”链接到 google.com，你可以得化成：

    [Google][]

然后定义链接内容：

    [Google]: http://google.com

由于链接文字可能包含空白，所有这种简化的标签内也可以包含多个文字：

    Visit [Daring Fireball][] for more information.

然后接着定义链接：

    [Daring Fireball]: http://daringfireball.net/

链接的定义可以放在文件中的任何一个地方，我比较偏好直接放在链接出现段落的后面，你也可以把它放在文件最后面，就像是注解一样。

下面是一个参考式的链接的范例：

    I get 10 times more traffic from [Google] [1] than from
    [Yahoo] [2] or [MSN] [3].

      [1]: http://google.com/        "Google"
      [2]: http://search.yahoo.com/  "Yahoo Search"
      [3]: http://search.msn.com/    "MSN Search"

如果改成用链接名称的方式写：

    I get 10 times more traffic from [Google][] than from
    [Yahoo][] or [MSN][].

      [google]: http://google.com/        "Google"
      [yahoo]:  http://search.yahoo.com/  "Yahoo Search"
      [msn]:    http://search.msn.com/    "MSN Search"

上面两种写法都会产生下面的HTML。

    <p>I get 10 times more traffic from <a href="http://google.com/"
    title="Google">Google</a> than from
    <a href="http://search.yahoo.com/" title="Yahoo Search">Yahoo</a>
    or <a href="http://search.msn.com/" title="MSN Search">MSN</a>.</p>

下面是用行内形式写的同样一段内容的Markdown文件，提供作为比较之用：

    I get 10 times more traffic from [Google](http://google.com/ "Google")
    than from [Yahoo](http://search.yahoo.com/ "Yahoo Search") or
    [MSN](http://search.msn.com/ "MSN Search").

参考式的链接其实重点不在于它比较好写，而是它比较好读，比较一下上面的范例，使用参考式的文章本身只有81个字符，但是用行内形式的链接却会增加到176个字符，如果是用纯HTML格式来写，会有234个字符，在HTML格式中，标签比文字还要多。

使用 Markdown 的参考式链接，可以让文件更像是浏览器最后产生的结果，让你可以把一些标记相同的资讯移到段落文字之外，你就可以增加链接而不让文字的阅读感觉被打断。

<h3 id="em">强调</h3>

Markdown使用星号(`*`)和底线(`_`)作为标记强调字词的符号，被`*`或`_`包围的字词会被转成用`<em>`标签包围，用两个`*`或`_`包起来的话，则会被转成`<strong>`，例如：

    *single asterisks*

    _single underscores_

    **double asterisks**

    __double underscores__

会转成：

    <em>single asterisk</em>

    <em>single underscores</em>

    <strong>double asterisk</strong>

    <strong>double underscores</strong>

你可以随便用你喜欢的样式，唯一的限制是，你用什么符号开放标签，就要用什么符号结束。

强调也可以直接插在文字中间：

    un*frigging*believable

但是如果你的`*`和`_`两边都有空白的话，它们就只会被当成普通的符号。

如果要在文字前后直接插入普通的星号或底线，你可以用反斜线：

    \*this text is surrounded by literal asterisks\*

<h3 id="code">程序</h3>

如果要标记一小段行内程序，你可以用反引号把它包起来(`` ` ``)，例如：

    Use the `printf()` funciton

会产生：

    <p>Use the <code>printf()</code> function.</p>

如果要在程序段内插入反引号，你可以用多个反引号来开启或结束程序段：

    ``There is a literial backtick (`) here.``

这段语法会产生：

    <p><code>There is a literial backtick (`) here.</code></p>

程序段的起始和结束都可以放入一个空白，起始后面一个，结束前面一个，这样你就可以在段的一开始就插入反引号：

    A single backtick in a code span: `` ` ``

    A backtick-delimited string in a code span: `` `foo` ``

会产生：

    <p>A single backtick in a code span: <code>`</code></p>

    <p>A backtick-delimited string in a code span: <code>`foo`</code></p>

在程序段内，`&`和方括号都会被转成HTML实体，这样会比较容易插入HTML原始码，Markdown会把下面这段：

    Please don't use any `<blink>` tags.

转为：

    <p>Please don't use any <code>&lt;blink&gt;</code> tags.</p>

你也可以这样写：

    `&#8212;` is the decimal-encoded equivalent of `&mdash;`.

 会产生：

    <p><code>&amp;#8212;</code> is the decimal-encoded
    equivalent of <code>&amp;mdash;</code>.</p>


<h3 id="img">图片</h3>

很明显地，要在纯文字应用中设计一个「自然」的语法来插入图片是有一定难度的。

Markdown 使用一种和链接很相似的语法来标记图片，同样也允许两种样式： *行内*和*参考*。

行内图片的语法看起来像是：

    ![Alt text](/path/to/img.jpg)

    ![Alt text](/path/to/img.jpg "optional title")

详细叙述如下：

*  一个感叹号`！`
*  接着一对方括号，里面放上图片的替代文字
*  接着一个普通括号，里面放上图片的网址，最后还可以用引号包住并加上选择性的`title`文字。

参考式的图片语法则长得像这样：

    ![Alt text][id]

[id]是图片参考的名称，图片参考的定义方式则和链接参考一样：

    [id]: url/to/image  "Optional title attribute"

到目前为止，Markdown 还没有办法指定图片的宽高，如果你需要的话，你可以使用普通的`img`标签。

* * *

<h2 id="misc">其它</h2>

<h3 id="autolink">自动链接</h3>

Markdown 支持比较简短的自动链接形式来处理网址和电子邮件信箱，只要是用方括号包起来，Markdown就会自动把它转成链接，链接的文字和链接位置一样，例如：

    <http://example.com>

Markdown 会转成：

    <a href="http://example.com/">http://example.com/</a>

自动的邮件链接也很相似，只是Markdown会先做一个编码转换的过程，把文字字符转成16位码的HTML实体，这样的格式可以混淆一些不好的信箱地址收集机器人，例如：

    <address@example.com>

Markdown 会转成：

    <a href="&#x6D;&#x61;i&#x6C;&#x74;&#x6F;:&#x61;&#x64;&#x64;&#x72;&#x65;
    &#115;&#115;&#64;&#101;&#120;&#x61;&#109;&#x70;&#x6C;e&#x2E;&#99;&#111;
    &#109;">&#x61;&#x64;&#x64;&#x72;&#x65;&#115;&#115;&#64;&#101;&#120;&#x61;
    &#109;&#x70;&#x6C;e&#x2E;&#99;&#111;&#109;</a>

在浏览器里，这段字符串会变成一个可以点击的「address@example.com」链接。

(这种作法虽然可以混淆不少的机器人，但无法全部挡下来，不过这样也比什么都不做好些。无论如何，公开你的信箱终究会引来广告信件的。)

<h3 id="backslash">转义字符</h3>

Markdown 可以利用反斜线来插入一些在语法中有其它意义的符号，例如：如果你想要用星号加在文字旁边的方式来做出强调效果（但不用`<em>`标签），你可以在星号的前面加上反斜线：

    \*literal asterisks\*

Markdown 支持在下面这些符号前面加上反斜线来帮助插入普通的符号：

    \  反斜线
    `  反引号
    *  星号
    _  下划线
    {} 大括号
    [] 方括号
    () 括号
    #  井号
    +  加号
    -  减号
    .  英文句点
    ！  感叹号
