---
title: "js双击复制"
layout: post
date: 2017-07-17
tags:
- JavaScript
categories: JAVASCRIPT
blog: true
excerpt: js双击复制
---

## 双击复制


```html
<textarea id="contents" rows="10" cols="10" style="height:0;width:0;opacity: 0;"></textarea>  
```

```js
    $('.span').dblclick(function(){
    	var e=document.getElementById("contents");
    	e.value = $(this).text();
    	e.select();
       	document.execCommand("Copy");
       	var dlg = BootstrapDialog.show({  
            message: '复制完成',
            cssClass: 'copy-dialog'
        });  
       	setTimeout(function(){
       	    dlg.close();
       	}, 200);
    });
```