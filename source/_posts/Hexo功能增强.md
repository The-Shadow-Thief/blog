---
title: Hexo功能增强
date: 2016-05-08 12:11:07
toc: true
tags:
  - Hexo
categories:
  - DevTools 
---
由于最近写的文章越来越长了, 没有`文章目录`和`返回顶部`, 阅读体验十分糟糕, 返回顶部是为了查看总目录服务的, 所以参考两篇文章在自己的博客上实现了这两个功能
<!--more-->

### **添加文章目录**

在原文[在Hexo中给文章加目录(Table Of Contents)](http://morningchen.com/2015/07/15/Table-Of-Contents-for-hexo/)中已经介绍得很详细了, 我在我的文章目录中添加了编号

1. yilia的主题在`/themes/yilia/layout/_partial/article.ejs`里修改, 将下面代码放在`<%- post.content %>`之上

    ```
    <% if(post.toc == true){ %>
    <div id="toc" class="toc-article">
      <strong class="toc-title">文章目录</strong>
    <%- toc(post.content, {list_number: false}) %>
    </div>
    <% } %>
    ```

    如果想在文章中使用目录的话，可以在 `front-matter`里加上` toc: true `即可. 像这样子：

    ```
    ---
    title: "在Hexo中给文章加目录(Table Of Contents)"
    date: 2016-09-08 12:40:42
    tags:
    - Hexo
    toc: true
    ---
    ```

2. 修改目录样式, 这部分在原文的基础上添加了标题序号, 效果见本博客

    在`/themes/yilia/source/css/_partial/article.styl`中的末尾加上：
    
    ```css
    .toc-article {
      background: #eee;
      margin: 0 0 0 .5em;
      padding: 1em
    }
    .toc-article strong {
      padding: .3em 0
    }
    
    #toc {
      line-height: 1.6em;
      font-size: .8em;
      float: right
    }
    #toc .toc {
      padding: 0
    }
    #toc .toc li {
      list-style-type: none
      list-style-type:decimal;
      list-style-position:inside;
    }
    #toc ol {
      margin-left: 0
    }
    #toc .toc-child {
      padding-left: 1.5em
    }
    ```
    
### **添加返回顶部按钮**

1. 在`/themes/yilia/layout/_partial/totop.ejs `新建`.ejs`文件

    添加以下内容, 如果要需要修改按钮的位置, 修改`bottom:95px`和`right:-5px`这两个参数就可以

    ```
    <div id="totop" style="position:fixed;bottom:95px;right:-5px;cursor: pointer;">
    <a title="返回顶部"><img src="/img/scrollup.png"/></a>
    </div>
    ```
    
2. 在`/themes/yilia/source/js/totop.js`新建`.js`文件

    添加以下内容, // 如果对下拉显示距离和回滚速度不太满意，可以修改代码的`upperLimit`和`scrollSpeed`参数

    ```javascript
    (function($) { 
    	// When to show the scroll link
    	// higher number = scroll link appears further down the page   
    	var upperLimit = 1000;
    	
    	// Our scroll link element
    	var scrollElem = $('#totop');
        
    	// Scroll to top speed
    	var scrollSpeed = 500;
        
    	// Show and hide the scroll to top link based on scroll position   
    	scrollElem.hide();
    	$(window).scroll(function () {            
    		var scrollTop = $(document).scrollTop();       
    		if ( scrollTop > upperLimit ) {
    			$(scrollElem).stop().fadeTo(300, 1); // fade back in           
    		}else{       
    			$(scrollElem).stop().fadeTo(300, 0); // fade out
    		}
    	});
    
    	// Scroll to top animation on click
    	$(scrollElem).click(function(){
    		$('html, body').animate({scrollTop:0}, scrollSpeed); return false;
    	});
    })(jQuery);    
    ```
    
   

3. 修改`themes/yilia/layout/_partial/after_footer.ejs`文件

    在文件末尾添加以下两行代码：

    ```
    <%- partial('totop') %>
    <script src="<%- config.root %>js/totop.js"></script>
    
    ```
    
4. 在 [Material icons](https://design.google.com/icons/)选择自己的图标, 将图片复制到 `/themes/yilia/source/img` 目录下，文件名为 `scrollup.png`


> 参考文章 : 在此特别感谢喜欢写博客的人

- [在Hexo中给文章加目录(Table Of Contents)](http://morningchen.com/2015/07/15/Table-Of-Contents-for-hexo/)
- [Hexo博客优化：添加返回顶部功能](http://wuchong.me/blog/2014/01/08/hexo-scrollup/)