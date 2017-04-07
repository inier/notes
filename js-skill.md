## 页面跳转
1. Javascript 返回上一页 history.go(-1), 返回两个页面: history.go(-2); 

2. history.back(). 

3. window.history.forward()返回下一页 

4. window.history.go(返回第几页,也可以使用访问过的URL) 
```
例:   
[url=javascript:history.go(-1);]向上一页[/url]  
  
response.Write("<script language=javascript>")  
response.Write("if(!confirm('完成任务?')){history.back();}")  
response.Write("</script>")  
response.Write("<script language=javascript>history.go(-1);</script>")  
[url=javascript:history.go(-1);]向上一页[/url]  
  
页面跳转:onclick="window.location.href='xx.nsf/form?openform'"  
```


### JS引用JS
```
<script type=text/javascript>  
<!--  
if (typeof SWFObject == "undefined") {  
document.write('<scr' + 'ipt type="text/javascript" src="/scripts/swfobject-1.5.js"></scr' + 'ipt>');}  
//-->  
</script>  
```

### Javascript刷新页面的几种方法： 
```
1    history.go(0)  
2    location.reload()  
3    location=location  
4    location.assign(location)  
5    document.execCommand('Refresh')  
6    window.navigate(location)  
7    location.replace(location)  
8    document.URL=location.href  
```

### 自动刷新页面的方法: 
```
1.页面自动刷新：把如下代码加入<head>区域中  
<meta http-equiv="refresh" content="20">  
其中20指每隔20秒刷新一次页面.  
  
2.页面自动跳转：把如下代码加入<head>区域中  
<meta http-equiv="refresh" content="20;url=http://www.hanhe-tech.com">  
其中20指隔20秒后跳转到http://www.hanhe-tech.com页面  
  
3.页面自动刷新js版  
<script language="JavaScript">  
function myrefresh()  
{  
    window.location.reload();  
}  
setTimeout('myrefresh()',1000); //指定1秒刷新一次  
</script>  
```

### 如何输出刷新父窗口脚本语句 
```
1.   this.response.write("<script>opener.location.reload();</script>");  
  
2.   this.response.write("<script>opener.window.location.href = opener.window.location.href;</script>");  
  
  
3.   Response.Write("<script language=javascript>opener.window.navigate(''你要刷新的表单'');</script>") 
```

### JS刷新框架的脚本语句 
```
//如何刷新包含该框架的页面用  
<script language=JavaScript>  
   parent.location.reload();  
</script>  
  
//子窗口刷新父窗口  
<script language=JavaScript>  
    self.opener.location.reload();  
</script>  
(　或　[url=javascript:opener.location.reload()]刷新[/url]   )  
  
//如何刷新另一个框架的页面用  
<script language=JavaScript>  
   parent.另一FrameID.location.reload();  
</script>  
```

> 如果想关闭窗口时刷新或者想开窗时刷新的话，在<body>中调用以下语句即可。
```
<body onload="opener.location.reload()"> 开窗时刷新  
<body onUnload="opener.location.reload()"> 关闭时刷新  
  
<script language="javascript">  
window.opener.document.location.reload()  
</script>  
```
