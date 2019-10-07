.. post::Sep 06 , 2019
   :tags: directive
   :category: ablog
   :author: HicoderDR
   :images:3

我如何创建自己的博客
#####################################
GitHub Pages
**************
    转到GitHub并创建一个名为username.github.io的Repository，其中username是GitHub上的用户名

.. image:: _static/githubpages.png

ablog
*********
首先在终端输入
**pip install -U ablog**
安装ablog

然后cd至你希望保存源码的目录下
**ablog start**
初始化项目

你需要键入几项初始信息，如下

.. image:: _static/ablogstart.png 

在此之后
**ablog build**
完成构建

**ablog serve**
在本地预览博客

最后使用
**ablog deploy -g username**
将博客更新至github上

至此初始博客已经被推送到username.github.io上了

如何更改博客的主题
*******************
安装第三方主题

很多第三方主题可以在PyPI, GitHub and sphinx-themes.org上找到

在此以我使用的sphinx_rtd_themew为例

首先
**pip install sphinx_rtd_theme**

然后

.. image:: _static/theme4.png 

此外推荐一些其他可行的主题：
    | `kr-sphinx-themes <https://github.com/kennethreitz-archive/kr-sphinx-themes>`_   
    | 有两个漂亮，干净的主题，用于许多流行（和好看）的项目。
    | `sphinx-readable-theme <https://sphinx-readable-theme.readthedocs.io/en/latest/>`_ 
    | 是一个针对apidoc生成的文档的可读性进行了优化的主题。
    | `sphinxtheme-readability <https://sphinxtheme-readability.readthedocs.io/en/latest/>`_ 
    | 是另一种干净主题的尝试。

如何发布自己的贴子
****************************
新建xxx.rst文件

.. image:: _static/post.png 

首先必须申明..post的时间和作者

然后根据特殊需要添加参数，比如给文章添加Tag(标记)，category(分类)
……`详见官方文档 <https://ablog.readthedocs.io/manual/posting-and-listing/>`_ 

将其添加在index.rst的toctree中

.. image:: _static/toctree.png 

rst文档的写法
******************
`reStructuredtext快速入门 <https://www.jianshu.com/p/f60e9be4781d>`_

此外可以使用VScode安装reStructuredtext扩展，即可分屏预览

.. image:: _static/vscode.png 

ablog常用指令
******************
**ablog start**
初始化ablog

**ablog build**
构建ablog

**ablog serve**
本地预览ablog

**ablog deploy -g username**
将博客同步到github

**ablog clean -D**
清楚build文件

资料链接
***********

`Github官方文档 <https://pages.github.com/>`_

`ablog官方文档 <https://ablog.readthedocs.io/>`_

`sphinx官方文档 <http://www.sphinx-doc.org/en/master/>`_