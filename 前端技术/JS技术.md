js实现网页自动刷新
-------------------

```
var timeout = prompt("设置刷新时间间隔[S]");  
    // 获取当前的URL  
    var current = location.href;  

    if(timeout > 0)  
    {  
        // 时间间隔大于0，timeout秒之后执行reload函数  
        setTimeout('reload()', 1000 * timeout);  
    }  
    else  
    {  
        // 时间间隔不大于0，仅刷新一次  
        location.replace(current);  
    }  

    function reload()  
    {  
        // timeout秒后执行reload函数，实现无限循环刷新  
        setTimeout('reload()', 1000 * timeout);  
        // 下面两行代码的格式化后的内容为：  
        // <frameset cols='*'>  
        //     <frame src='当前地址栏的URL' />  
        // </frameset>  
        var fr4me = '<frameset cols=\'*\'>\n<frame src=\'' + current + '\' />';  
        fr4me += '</frameset>';  

        with(document)  
        {  
            // 引用document对象，调用write方法写入框架，打开新窗口  
            write(fr4me);  
            // 关闭上面的窗口  
            void(close());  
        };  
    }  
```