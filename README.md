# 百一测评网站切屏检测绕过
> 事情是这样的，这几天不是临近期末嘛，老师都开始划重点，准备在线考试的老师也在开始测试线上考试了，今天人工智能在百一测评发下来一套测试，想点进去看看能不能粘贴，结果刚出去百度，就弹出离开页面警告，这谁顶得住，人工智能一点都不会岂不是就要挂科了，只能动动信安专业头脑想想办法了！

说是破解，其实也就是想办法把防切屏解了。
之前有些考试软件防止切屏可以用虚拟机，稍微复杂一点，至于浏览器检测切屏，无非就是检测焦点，像有些网站的动态标题就是这样，那么用什么来实现检测焦点呢？这里不得不提到JavaScript。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225144746535.jpg)

众所周知，js是一种较常用的Web页面开发脚本语言，功能一般是为web页面添加用户与页面的交互行为，介质是通过浏览器。这里要涉及到的是js的响应浏览器事件的功能，之前我的一篇写pjax和ajax的文章的时候提到过pjax和ajax加载事件，用到的就是大名鼎鼎的jQuery框架中的方法。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225145419192.jpg)

浏览器检测焦点用到的也是jQuery框架中的blur()和focus()方法，具体用法可以参考：https://www.runoob.com/jquery/event-blur.html
好了，科普就到这里，我们直接进入正题。
首先进入老师给的测试考试页面，首先我们使用f12大法来看看有没有引入jQuery框架。注意这里从点进考试页面开始就进入了ajax模式，题目和提交都是通过ajax方式加载，所以我们打开开发者工具之后要刷新一下来刷新network模块获取到的数据。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225150633778.png)

可以看到是引入了jQuery框架，但是还不确定是不是调用了blur()方法来检测焦点，如果不是那么有可能是重新定义的方法名。但是这里用了这么多js，我们怎么才能找出用来监听焦点的js文件呢。
因为是测试考试，所以有无数次重考的机会，那我们就给了咱们慢慢尝试的机会了，真是天助我也 :@(赞一个) ，我们先点击开始考试，并且“主动被检测作弊”。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225151724267.png)

我们在console模块里面可以看到有console.log，没想到连调试信息都没关，那岂不是更简单了？点击外面使考试窗口失焦
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225152026154.png)

不出我所料，整个过程的console.log调试信息都没关，不知道是运维师傅疏忽还是等着咱来挑战（如果是疏忽，运维师傅挨打，请 :@(脸红) ）。既然整个过程都有调试信息，那么我们就可以很方便的跟踪调试整个过程。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225153746204.png)

首先是点击开始考试，弹出startExam，点击进入，发现是调用的这个js：
https://kaoba.101test.com/cand/app/exam/view-exam-examing.js?__v=180104
我们来分析一下这个js
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122515510473.png)

这应该就是一个开始考试调用题目的js，我们再来跟踪下一条
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225160034569.png)

调用的是这个js：
https://kaoba.101test.com/cand/app/exam/view-exam-listeningLeave.js?__v=180104
注释有记录焦点的函数，那么应该就是这个js没错了，继续跟踪
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225160757200.png)

看注释这是考试页面获取焦点事件的函数，再看下一个
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225161721864.png)

这是考试页面失焦后三秒弹出的提示，然后三秒之后弹出考试界面记录离开页面次数。至此整个流程走完，可以知道，记录离开页面次数的核心文件是view-exam-listeningLeave.js
然后我试过用AdblockPlus插件把它拦截掉，最后发现无法正常加载题目，应该是有哪个地方检测js文件的引用。那我们也不必大费周章的去找哪里有debug检测了（我要是把这段错误信息发送给老师那岂不就是自投罗网嘛 :@(尴尬) ）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225163951648.png)

我们直接把view-exam-listeningLeave.js保存下来分析
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225165524545.png)

这里require了同目录下的app/exam/model-exam-listeningLeave，我们暂时先不管，这个是用来提交离开页面次数和返回答题视图的
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225165806929.png)

然后可以看到这里有一个configMap的js对象，里面定义了几个键值对，其中功能看注释可以知道

    configMap = {
            isInit :false,
    	pubMap : {
    	gevent_arriveSwitchLimit : "view-exam-listeningLeave-arriveSwitchLimit"
    	},
    	countLeaveTime :0,//计时离开的时间
    	leaveTimeLimit :3,//离开多少秒才算,以秒为单位
    	isBlur:false,
    	currentKeyCode : -1,//按键code
    	rightKey:false,//右键
    	partSeq:null, 
    	questionId :null
    }

按理来说只要把leaveTimeLimit替换成一个大数字使其限制增大就能绕过，那么我们怎么修改js的内容，这里提供两种思路，如果没有特殊情况的话应该也适用于其他网站考试系统的防切屏。
[quote color="success"]第一种：
修改host文件或使用浏览器插件替换引用到的js内容，毕竟js响应浏览器事件都是本地响应，js都到我自己的电脑上来了那操作还不简单。插件推荐有ReRes，可以添加规则替换js来源，用法比较简单，大家可以自己摸索。[/quote]
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225173925815.png)

这种方法优点是稳定生效，替换了就肯定有用；缺点是操作比较麻烦，且修改host方法仅适用于引用js文件来源和考试页面不同源的情况下。

[quote color="warning"]第二种：
F12大法好，通过开发者工具的source模块可以直接修改本地的js缓存，直接修改后按Ctrl+S保存即可。
这种方法的优点是：简单方便，即改即用，打开开发者工具就可以自己调试；缺点是，可以通过反调试来阻止你修改，能不能绕过反调试就看对面的设计逻辑了，比如万一给你整个debugger无限断点什么的。[/quote]
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225174823562.png)

因为百一测评的考试页面没有debugger反调试，我们就以最方便的第二种方法来分析，其他情况还请大家可以举一反三。
刚刚也提到了，按理来说只要修改了leaveTimeLimit的值为超大数值，就可以绕过限制，那么结果究竟是不是这样呢？我们来一步一步调试，端好小板凳跟我来。
打开开发者工具的source模块，找到configMap对象，修改leaveTimeLimit值为超大值，然后Ctrl+S保存
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225175340535.png)

事实证明是不可行的，为什么呢？我们在configMap.leaveInterval循环里面添加一个console.log(configMap)来输出我们修改后的configMap对象
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122517595852.png)

Ctrl+S保存，if循环自动执行输出configMap对象
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225180446381.png)

可以看到我们修改的值是完全没有生效的，这是因为js对象在读取的时候已经将键值对储存在了内存里，要想修改键值对，只有重新赋值把它覆盖掉才行，但是这里因为js作用域的问题无法直接通过console访问对象，我作用域又学的垃圾，构造不出来什么像样的脚本，我们换个方法，既然访问不了configMap对象，那我们就直接改储存在缓存中的判断条件。
我这里一共找到了两个if判断语句，一个是用来过滤特殊情况不记录离开次数的，另一个是本身判定countLeaveTime和leaveTimeLimit值的
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225181711255.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225181810847.png)

改特殊情况判定条件：
直接往switchTimesFilter函数里面加一个无条件过滤
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225182349505.png)

改leaveInterval判定条件：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225182845707.png)

成功绕过切屏检测 :@(赞一个) 

这次提到的几个方法理论上来说对所有浏览器切屏检测都有效，至于用法还请大家举一反三，如果还有什么其他的我不知道的切屏检测方法，那属实是我见识短浅，还请大佬们一定要告诉我，不要让我误人子弟 :@(害羞) ，当然如果用这个方法如果被老师抓住了一定不要来找俺啊，我什么都没说我也什么都不知道哈 :@(哭泣) 

[quote color="danger"]另外，还说一个事，这次调试过程中我发现百一测评没有强制https，因此可以用burpsuite抓包，并且离开页面次数是通过js以post方式提交的，然后我试着抓了一下包。。。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225190514686.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225190427809.png)

也就是说，我们可以通过分析js的提交规则，然后构造一个提交离开次数的数据包，然后。。。
诶等等等等，我在想什么啊，我的想法很危险啊，咳咳，大家当没看到就好（仇人眼 :@(阴暗) ）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201225190937891.png)
[/quote]
