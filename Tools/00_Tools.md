---
layout: default
title: Tools
has_children: true
nav_order: 15
---

# Tools
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}



<script type="text/javascript">


 function getURL(url) {
        var xmlhttp = new ActiveXObject( "Microsoft.XMLHTTP");
        xmlhttp.open("GET", url, false);
        xmlhttp.send();
        if(xmlhttp.readyState==4) {
            if(xmlhttp.Status != 200) alert("不存在");
            return xmlhttp.Status==200;
        }
        return false;
}
function checkAll(){
    var elements = document.getElementsByTagName("p");
    for(var index=0 ; index< elements.length; index ++){
        var ele = elements[index];
        var mark = ele.getAttribute("mark");
        if (mark!='linkMe'){
            continue ;
        }
        var childNodes = ele.childNodes;
        for(var a=0 ; a< childNodes.length; a ++){
                var eleLink = elements[index];
                var link = eleLink.getAttribute("href");
                console.log(  "yes "  + link);
                /* && (link.startsWith("https", 0) || link.startsWith("http", 0) ) */
                if(  (link.startsWith("https") || link.startsWith("http") ) && getURL(link)){
                    
                    eleLink.style.backgroundColor = 'green';
                }else{
                    eleLink.style.backgroundColor = 'red';
                }
                
        }
        
    }
    
}

/*
if (typeof String.prototype.startsWith !== 'function') {
    String.prototype.startsWith = function(prefix) {
        return this.slice(0, prefix.length) === prefix;
    };
}

if (typeof String.prototype.endsWith !== 'function') {
    String.prototype.endsWith = function(suffix) {
        return this.indexOf(suffix, this.length - suffix.length) !== -1;
    };
}

 

String.prototype.startsWith = function(str) {
    if (!str || str.length > this.length){
        return false;
    }
    if (this.substr(0, str.length) == str){
        return true;
    }else{
        return false;
    }
    return true;
}
 */
/*  使用正则表达式 */ 
String.prototype.startsWith = function(str) {
    var reg = new RegExp("^" + str);
    return reg.test(this);
}

/* 测试ok，直接使用str.endsWith("abc")方式调用即可  */ 
String.prototype.endsWith = function(str) {
    var reg = new RegExp(str + "$");
    return reg.test(this);
}


</script>
<button class="btn btn-purple mr-2" onclick="checkAll()" value="一键检测" >一键检测</button>


---
### github

GitHub访问
镜像站直接访问（别登录）

- [https://hub.おうか.tw](https://hub.おうか.tw)
- [https://hub.連接.台灣](https://hub.連接.台灣)
- [https://hub.fastgit.xyz](https://hub.fastgit.xyz)
- [https://cdn.githubjs.cf](https://cdn.githubjs.cf)
- [https://gitclone.com](https://gitclone.com)
- [https://hub.gitfast.tk](https://hub.gitfast.tk)
- [https://hub.gitslow.tk](https://hub.gitslow.tk)
- [https://hub.verge.tk](https://hub.verge.tk)
- [https://raw.gitfast.tk](https://raw.gitfast.tk)
- [https://raw.gitslow.tk](https://raw.gitslow.tk)
- [https://raw.verge.tk](https://raw.verge.tk)

GitHub加速下载
浏览器插件和油猴脚本

- [Github 增强 (脚本)](https://greasyfork.org/zh-CN/scripts/412245)
- [Fast-GitHub (插件)](https://fhefh2015.github.io/Fast-GitHub/)
- [FastGithub(脚本)](https://greasyfork.org/zh-CN/scripts/397419)

直接通过输入下载地址加速下载

- [Gitclone](https://gitclone.com/)	
- [下载 (serctl.com)](https://d.serctl.com/)
- [GitHub Proxy 代理加速](https://ghproxy.com/)
- [GitHub 文件加速](https://gh.api.99988866.xyz/)
- [GitHub加速链接生成工具](https://github.zhlh6.cn/)
- [socialify](https://socialify.git.ci/)


其他

[Trending repositories on GitHub today（GitHub热门项目）](https://github.com/trending)

[https://hellogithub.com/](https://hellogithub.com/)（分享GitHub上有趣、入门级的开源项目）

[https://www.pdai.tech/ Java 全栈知识体系](https://www.pdai.tech/)

[http://coderleixiaoshuai.gitee.io/java-eight-part/#/](八股文)

[http://doc.yonyoucloud.com/doc/wiki/project/GradleUserGuide-Wiki/index.html](Gradle用户指南官方文档中文版)

[https://www.githubs.cn/top](https://www.githubs.cn/top)


---
### Markdown Online
[MarkDown官方1](https://markdown.com.cn/){: .btn .btn-outline .d-inline-block fs-5 mb-4 mb-md-0 mr-2 }
[MarkDown在线2](http://editor.md.ipandao.com/examples/full.html){: .btn .btn-outline .d-inline-block fs-5 mb-4 mb-md-0 mr-2 }
[MarkDown在线3](https://www.zybuluo.com/mdeditor){: .btn .btn-outline .d-inline-block fs-5 mb-4 mb-md-0 mr-2 }
[MarkDown在线4](http://mahua.jser.me/){: .btn .btn-outline .d-inline-block fs-5 mb-4 mb-md-0 mr-2 }

[😂 Emoji在线1](http://getemoji.com/){: .btn .btn-outline .d-inline-block fs-5 mb-4 mb-md-0 mr-2 }
[😄 Emoji在线2](https://www.emojidaquan.com/){: .btn .btn-outline .d-inline-block fs-5 mb-4 mb-md-0 mr-2 }
[😁 Emoji在线3](https://www.emojiall.com/zh-hans){: .btn .btn-outline .d-inline-block fs-5 mb-4 mb-md-0 mr-2 }
[😆 Emoji在线4](https://emojipedia.org/){: .btn .btn-outline .d-inline-block fs-5 mb-4 mb-md-0 mr-2 }
  
### MixResource

<p mark="linkMe">

<a href="http://www.ziyuandi.cn/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">资源帝</a>
<a href="https://docs.qq.com/sheet/DRU5MWHZCTHFGQnhM" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">课程知识库</a>
<a href="https://www.ziyuanm.com/T" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">资源猫</a>
<a href="https://cilitiantang.me" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">磁力池</a>

<a href="https://www.iconfont.cn" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">iconFont</a>
<a href="https://www.100font.com/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">免费字体</a>
<a href="https://www.fontspace.com/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">FontSpace</a>
<a href="https://search.yibook.org/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">找书易</a>
<a href="https://www.transferfile.io/#/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">临时文件传输</a>
<a href="https://lkssite.vip" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">lkssite</a>

<a href="https://www.futurepedia.io" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">AI工具</a>
<a href="https://x.magiconch.com/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">表情包</a>
<a href="https://lab.magiconch.com/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">奇奇怪怪</a>


</p>
  
### Tool

<p mark="linkMe">


<a href="https://www.bilibili.com/video/BV1wy4y1D7JT" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">React</a>
<a href="https://babeljs.io/docs/en/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">babeljs</a>
<a href="https://www.tslang.cn/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">TypeScript</a>
<a href="https://www.jhipster.tech/cn/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">JHipster</a>

</p>

### Inter Tool

<p mark="linkMe">
<a href="https://dnschecker.org" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">Dns Checker</a>
<a href="https://ipaddress.com" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">IP ADDRESS</a>
<a href="https://www.kuaidaili.com/free/inha/" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">国内代理</a>

</p>

### Mock Tool

<p mark="linkMe">

<a href="https://github.com/wiremock/wiremock" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">WireMock</a>

<a href="https://github.com/mock-server/mockserver" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">Mock-Server</a>

<a href="http://www.jsons.cn/unicode" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">okhttp/mockwebserver</a>

<a href="https://mocki.io/mock-http-server" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">Mock-Http-Server</a>

<a href="https://www.cnblogs.com/larva-zhh/p/11678940.html" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">单元测试Mock-Serve选型r</a>
</p>

### Tools Online

<p mark="linkMe">

<a href="https://my-mind.github.io/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">My-Mind</a>
<a href="https://c.runoob.com/front-end/710/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">XML在线格式化</a>

<a href="https://www.diffchecker.com/diff" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">文本/文件在线比较</a>

<a href="http://www.jsons.cn/unicode" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">UNICODE转换</a>
<a href="https://sqltry.com" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">sqltry</a>


</p>

<p mark="linkMe">

<a href="http://linux.51yip.com/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Linux 命令手册</a>

<a href="http://cron.ciding.cc/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Cron在线表达式</a>

<a href="https://jex.im/regulex/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">regulex</a>

<a href="https://www.67tool.com/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Tool-Online</a>

<a href="https://www.toolnb.com/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Tool-NB</a>

<a href="https://fastthread.io/ft-index.jsp" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Thread Dump Analyzer</a>


<a href="https://blog.luckly-mjw.cn/tool-show/m3u8-downloader/index.html" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">m3u8-Downloader</a>

<a href="http://www.ico51.cn/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">生成透明ICO</a>

<a href="https://nav.rdonly.com/laboratory/bgimage/backimage.html" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">BgImage</a>

<a href="https://www.aigei.com/bgremover/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">BgRemover</a>

</p>


### system of Software 

<p mark="linkMe">
<a href="https://developers.redhat.com/products/rhel/download" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">RHEL</a>
<a href="https://www.centos.org/centos-stream/" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">CentOS-Stream</a>
<a href="https://mirrors.tuna.tsinghua.edu.cn/" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">mirrors.TUNA</a>
<a href="http://mirrors.163.com/" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">mirrors.163</a>
<a href="https://www.linux.org/pages/download/" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">linux.download</a>
<a href="https://www.diagrams.net/" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">drawio-desktop</a>
<a href="http://dootask.com/" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">dootask</a>
<a href="https://etherpad.org/" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">etherpad</a>

</p>

### Tools of Software 

<p mark="linkMe">
<a href="https://obsidian.md" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">知识库管理工具</a>

<a href="https://www.snipaste.com/index.html" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">snipaste</a>

<a href="http://java-decompiler.github.io/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Java Decompiler</a>

<a href="https://www.ventoy.net/cn/download.html" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Ventoy</a>

<a href="https://msdn.itellyou.cn/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Windows</a>

<a href="https://kanboard.org/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">kanboard</a>
<a href="https://github.com/wekan/wekan" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">wekan</a>
<a href="https://github.com/RocketChat/Rocket.Chat" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Rocket.Chat</a>


</p>

### Learn Online

<p mark="linkMe">
<a href="https://www.w3cschool.cn/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">w3cSchool</a>
<a href="https://www.xuetangx.com/" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">xuetangx</a>
<a href="http://www.javathinker.net" target="_blank" class="btn btn-outline  fs-5 mb-4 mb-md-0 mr-2">Java Thinker</a>

</p>


### Watch movies Online

<p mark="linkMe">

<a href="https://www.foxcup.cc" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">狐茶杯</a>
<a href="https://www.nen9.com/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">青柠</a>
<a href="https://www.dianyinggou.com" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">电影狗</a>
<a href="http://www.breakfan.com" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">breakfan</a>
<a href="http://tkys.vip/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">sky</a>
<a href="https://www.mvcat.com/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">MV CAT</a>
<a href="https://ddrk.me/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">DDRK.ME</a>
<a href="https://www.novipnoad.com/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">NO VIP NO AD</a>
<a href="https://www.rrmeiju.com/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">RR Meiju</a>
<a href="https://wanyouw.com/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Wan You W</a>
<a href="https://www.juo.cc" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">JUO.CC</a>
<a href="http://gaoqing.la/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Gao Qing</a>
<a href="https://www.hebeilaibang.com/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Mi Gu</a>
<a href="https://klyingshi.com/" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Ke Le</a>
<a href="https://www.jpys.me/"  titel="热门影视与动漫" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">jpys VIP</a>
<a href="https://www.libvio.me/" titel="热门电影和海外剧" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Lib Online</a>
<a href="https://yanetflix.com/" titel="奈飞" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">4K YA</a>
<a href="https://loli.magedn.com/" titel="影视与动漫综艺记录" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">MaGeDN YA</a>
<a href="https://555dy1.com/" titel="影视与动漫综艺记录" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">555DY YA</a>
<a href="https://nav.sbkko.com/" titel="影视与动漫综艺记录" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">SB KKO</a>
<a href="https://www.huohuo99.com/" titel="影视与动漫综艺记录" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">huohuo99</a>
<a href="https://cokemv.me/" titel="影视与动漫综艺记录" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">cokemv</a>
<a href="https://196y.com/" titel="影视与动漫综艺记录" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">196y</a>
<a href="http://wap.vipz8.com/" titel="影视与动漫综艺记录" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">vipz8</a>

</p>

### Other

<p mark="linkMe">

<a href="http://www.niuga.cn/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">牛噶解析</a>
<a href="https://www.timecn.cn/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">全民解析</a>
<a href="https://rebozj.com/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Re Bo</a>

<a href="https://adzhp.cn/yin-yue-ruan-jian.html" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Music</a>
<a href="https://www.onehourlife.com/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">One Hour Life</a>
<a href="https://adzhp.cn/vipdianyingyuduanshipinjiexiziyuansuoyin.html" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">No VIP</a>

</p>


<p mark="linkMe">
<a href="https://wallhere.com/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">wallhere</a>
<a href="https://bz.zzzmh.cn/index" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">极简壁纸</a>
<a href="https://zhutix.com/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">zhutix</a>
<a href="https://www.wallpaperup.com/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">wallpaperup</a>
<a href="https://wallpaperstock.net/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">wallpaperstock</a>
<a href="https://wallpaperscraft.com/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">wallpaperscraft</a>
</p>

<p mark="linkMe">
<a href="https://www.jinyongbook.com/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Jin Yong</a>
<a href="https://www.xuges.com/wuxia/gulong/index.htm" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Gu Long</a>
<a href="http://t.icesmall.cn/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Ming Zhu</a>

<a href="www.futureme.org" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">写信给自己</a>

</p>

<p mark="linkMe">
<a href="https://mp.weixin.qq.com/s/0QPrmVE7unxoVDSqkm8e_g" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">软件目录</a>
<a href="https://www.xuges.com/wuxia/gulong/index.htm" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Gu Long</a>
<a href="http://t.icesmall.cn/" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">Ming Zhu</a>
</p>


### GAMES

<p mark="linkMe">
<a href="https://docs.qq.com/sheet/DUGNXSUZSY05LdHZK?tab=BB08J2" titel="" target="_blank" class="btn btn-outline fs-5 mb-4 mb-md-0 mr-2">games</a>
</p>




### github 


[https://github.com/starxg/mybatis-log-plugin-free](https://github.com/starxg/mybatis-log-plugin-free)

[https://github.com/apache/shardingsphere-elasticjob](https://github.com/apache/shardingsphere-elasticjob)

[https://github.com/baomidou/jobs](https://github.com/baomidou/jobs)

[https://github.com/YunaiV/SpringBoot-Labs](https://github.com/YunaiV/SpringBoot-Labs)

[https://github.com/softwarevax/springcloud](https://github.com/softwarevax/springcloud)

[https://github.com/ityouknow/awesome-spring-cloud](https://github.com/ityouknow/awesome-spring-cloud)

[https://github.com/doocs/technical-books](https://github.com/doocs/technical-books)

[https://github.com/waylau/netty-4-user-guide-demos](https://github.com/waylau/netty-4-user-guide-demos)

[https://github.com/AutomaApp/automa](https://github.com/AutomaApp/automa)

[https://github.com/jgraph/drawio-desktop](https://github.com/jgraph/drawio-desktop)


[https://github.com/miiiku/hexo-theme-flexblock](https://github.com/miiiku/hexo-theme-flexblock)

[https://github.com/jerryc127/hexo-theme-butterfly](https://github.com/jerryc127/hexo-theme-butterfly)

[https://github.com/amehime/hexo-theme-shoka](https://github.com/amehime/hexo-theme-shoka)

[https://github.com/ppoffice/hexo-theme-icarus](https://github.com/ppoffice/hexo-theme-icarus)