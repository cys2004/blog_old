---
title: 记一次对某网盘直链下载插件的分析
cover: https://p1.meituan.net/dpplatform/2de0db0e0a4820138a86b5c50a916113116551.jpg
lang: zh-CN
toc: true
---
现在，通过基于TamperMonkey的浏览器插件来获得网盘文件直链加速下载早已不是什么黑科技，而国产的各路【网盘下载助手】可谓是“百花齐放”。本文记录了我对某个热门网盘直链下载助手代码的简单分析，其具体方法可能已经失效，但仍希望我的探索过程能给诸位带来一些启发与思考。

<!--more-->

## 脚本行为

安装后在网盘目录上出现下载按钮，点击后提示需用微信扫码关注公众号才能“免费获得”暗号来激活脚本。

![](https://p1.meituan.net/csc/11eae24634266623aaf5b0e25c8a038163948.png)

## 脚本分析

### 代码定位

在TamperMonkey管理界面中打开该脚本的代码页

大部分有着捆绑引流行为的网盘下载插件会通过API从开发者自有服务器上动态获取“暗号”/“口令”，因此通过对代码内URL的分析即可迅速定位相关语句。

![](https://p1.meituan.net/csc/6904d0a91281b267f0596c63307d2c4e39960.png)

可以看到，这里使用了JSON.parse()方法处理服务端回传数据，根据base关键字可以推断在传输相关数据时使用了base64编码。

### 抓包检验

配置好抓包工具后重载网页捕获数据包。（本文使用Fiddler5.0进行抓包操作）

根据API调用的URL可找到相应会话。

![](https://p0.meituan.net/csc/993c6a73649055885dce8f2674b7ef1e146606.png)

响应体使用了Base64编码。解码后内容如下所示。

```json
{"code":200,"pcs":{"0":"https://api-pan.xunlei.com/drive/v1/files/"},"img":"https://vkceyugu.cdn.bspapp.com/VKCEYUGU-0d8c17ea-3b18-45d5-bf2f-64e5c812dfc9/81f275bd-6a48-4474-a347-a1173a3a2761.png","btn":{"home":".pan-list-menu","share":".file-features-btns-wrap"},"d":"http://d.youxiaohou.com","name":"网盘直链下载助手","init":{"0":"请输入初始化暗号","1":"请输入暗号点亮按钮，扫二维码免费获取","2":"暗号正确！【下载助手】点亮成功！","3":"暗号不正确！","4":"试试用微信扫码回复暗号来点亮按钮吧！","5":"请先安装网盘万能助手，安装后请刷新本页！！！"},"api":{"0":"API下载<span style=\"font-size:14px;font-weight: 400;opacity: .8;\">（适用于 <a href=\"https://www.youxiaohou.com/zh-cn/idm.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">IDM</a>，<a href=\"https://www.youxiaohou.com/zh-cn/ndm.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">NDM</a> 以及浏览器自带下载）</span>","1":"点击链接直接下载，例如：<a href=\"https://www.youxiaohou.com/zh-cn/idm.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">IDM</a>，若未唤起IDM，请 <a href=\"https://www.youxiaohou.com/zh-cn/idm.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">点击这里</a> 配置文件类型，IDM 不显示文件名时，请手动复制"},"aria":{"0":"Aria下载<span style=\"font-size:14px;font-weight: 400;opacity: .8;\">（适用于 <a href=\"https://www.youxiaohou.com/zh-cn/xdown.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">XDown</a> 及 <a href=\"https://www.youxiaohou.com/zh-cn/linux.html#linux-shell\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">Linux Shell命令行</a>）</span>","1":"点击链接复制地址到剪切板，粘贴到支持 aria2c 协议的下载器中，例如：<a href=\"https://www.youxiaohou.com/zh-cn/xdown.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">XDown</a>，<a href=\"https://www.youxiaohou.com/zh-cn/linux.html#linux-shell\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">Linux Shell</a>"},"rpc":{"0":"RPC下载<span style=\"font-size:14px;font-weight: 400;opacity: .8;\">（适用于 <a href=\"https://www.youxiaohou.com/zh-cn/motrix.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">Motrix</a>，<a href=\"https://www.youxiaohou.com/download.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">Aria2 Tools</a>，<a href=\"https://www.youxiaohou.com/download.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">AriaNgGUI</a>）</span>","1":"点击按钮发送链接至本地或远程 RPC 服务，例如：<a href=\"https://www.youxiaohou.com/zh-cn/motrix.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">Motrix</a>，RPC 参数含义见<a href=\"https://www.youxiaohou.com/zh-cn/motrix.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">此处</a>"},"curl":{"0":"cURL下载<span style=\"font-size:14px;font-weight: 400;opacity: .8;\">（适用于 <a href=\"https://www.youxiaohou.com/zh-cn/curl.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">Windows，Linux，MacOS 终端</a>）</span>","1":"点击链接复制地址到剪切板，粘贴到 <a href=\"https://www.youxiaohou.com/zh-cn/curl.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">Windows，Linux，MacOS 终端</a>"},"bc":{"0":"BC下载<span style=\"font-size:14px;font-weight: 400;opacity: .8;\">（适用于 <a href=\"https://www.youxiaohou.com/zh-cn/bitcomet.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">比特彗星</a>）</span>","1":"点击链接复制地址到剪切板，粘贴到 <a href=\"https://www.youxiaohou.com/zh-cn/bitcomet.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">比特彗星</a> 下载器中，镜像地址可用于加速下载，使用方法<a href=\"https://www.youxiaohou.com/zh-cn/bitcomet.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">见此处</a>"},"mirror":["vod0067-aliyun08-vip-lixian.xunlei.com","vod0254-aliyun08-vip-lixian.xunlei.com","vod0255-aliyun08-vip-lixian.xunlei.com","vod0256-aliyun08-vip-lixian.xunlei.com","vod0257-aliyun08-vip-lixian.xunlei.com","vod0258-aliyun08-vip-lixian.xunlei.com","vod0259-aliyun08-vip-lixian.xunlei.com","vod0260-aliyun08-vip-lixian.xunlei.com","vod0261-aliyun08-vip-lixian.xunlei.com","vod0262-aliyun08-vip-lixian.xunlei.com","vod0263-aliyun08-vip-lixian.xunlei.com","vod0264-aliyun08-vip-lixian.xunlei.com","vod0265-aliyun08-vip-lixian.xunlei.com","vod0266-aliyun08-vip-lixian.xunlei.com","vod0267-aliyun08-vip-lixian.xunlei.com","vod0554-aliyun06-vip-lixian.xunlei.com","vod0555-aliyun06-vip-lixian.xunlei.com","vod0556-aliyun06-vip-lixian.xunlei.com","vod0680-aliyun08-vip-lixian.xunlei.com","vod0681-aliyun08-vip-lixian.xunlei.com","vod0682-aliyun08-vip-lixian.xunlei.com","vod0683-aliyun08-vip-lixian.xunlei.com","vod0684-aliyun08-vip-lixian.xunlei.com","vod0685-aliyun08-vip-lixian.xunlei.com","vod0686-aliyun08-vip-lixian.xunlei.com","vod0687-aliyun08-vip-lixian.xunlei.com","vod0688-aliyun08-vip-lixian.xunlei.com","vod0689-aliyun08-vip-lixian.xunlei.com","vod0690-aliyun08-vip-lixian.xunlei.com","vod0724-aliyun08-vip-lixian.xunlei.com","vod0725-aliyun08-vip-lixian.xunlei.com","vod0726-aliyun08-vip-lixian.xunlei.com","vod0727-aliyun08-vip-lixian.xunlei.com","vod0728-aliyun08-vip-lixian.xunlei.com","vod0075.aliyun06.vip.lixian.xunlei.com","vod0076.aliyun06.vip.lixian.xunlei.com","vod0077.aliyun06.vip.lixian.xunlei.com","vod0779-aliyun04-vip-lixian.xunlei.com","vod0078.aliyun06.vip.lixian.xunlei.com","vod0780-aliyun04-vip-lixian.xunlei.com","vod0781-aliyun04-vip-lixian.xunlei.com","vod0079.aliyun06.vip.lixian.xunlei.com","vod0080.aliyun06.vip.lixian.xunlei.com","vod0117.aliyun04.vip.lixian.xunlei.com","vod0118.aliyun04.vip.lixian.xunlei.com","vod0119.aliyun04.vip.lixian.xunlei.com","vod1284-aliyun06-vip-lixian.xunlei.com","vod1285-aliyun06-vip-lixian.xunlei.com","vod1363-aliyun06-vip-lixian.xunlei.com","vod1371-aliyun06-vip-lixian.xunlei.com","vod1372-aliyun06-vip-lixian.xunlei.com","vod1426-aliyun06-vip-lixian.xunlei.com","vod1427-aliyun06-vip-lixian.xunlei.com","vod1428-aliyun06-vip-lixian.xunlei.com","vod1429-aliyun06-vip-lixian.xunlei.com","vod1442-aliyun06-vip-lixian.xunlei.com","vod1443-aliyun06-vip-lixian.xunlei.com","vod1444-aliyun06-vip-lixian.xunlei.com","vod1445-aliyun06-vip-lixian.xunlei.com","vod1446-aliyun06-vip-lixian.xunlei.com","vod1447-aliyun06-vip-lixian.xunlei.com","vod1469-aliyun06-vip-lixian.xunlei.com","vod1470-aliyun06-vip-lixian.xunlei.com","vod1471-aliyun06-vip-lixian.xunlei.com","vod1489-aliyun06-vip-lixian.xunlei.com","vod1490-aliyun06-vip-lixian.xunlei.com","vod1491-aliyun06-vip-lixian.xunlei.com","vod1492-aliyun06-vip-lixian.xunlei.com","vod1493-aliyun06-vip-lixian.xunlei.com","vod0215.aliyun06.vip.lixian.xunlei.com","vod0216.aliyun06.vip.lixian.xunlei.com","vod0217.aliyun06.vip.lixian.xunlei.com","vod0218.aliyun06.vip.lixian.xunlei.com","vod0219.aliyun06.vip.lixian.xunlei.com","vod0220.aliyun06.vip.lixian.xunlei.com","vod0241.aliyun08.vip.lixian.xunlei.com","vod0244.aliyun08.vip.lixian.xunlei.com","vod0251.aliyun08.vip.lixian.xunlei.com","vod0252.aliyun08.vip.lixian.xunlei.com","vod0253.aliyun08.vip.lixian.xunlei.com","vod0254.aliyun08.vip.lixian.xunlei.com","vod0255.aliyun08.vip.lixian.xunlei.com","vod0256.aliyun08.vip.lixian.xunlei.com","vod0257.aliyun08.vip.lixian.xunlei.com","vod0260.aliyun08.vip.lixian.xunlei.com","vod0261.aliyun08.vip.lixian.xunlei.com","vod0262.aliyun08.vip.lixian.xunlei.com","vod0263.aliyun08.vip.lixian.xunlei.com","vod0264.aliyun08.vip.lixian.xunlei.com","vod0265.aliyun08.vip.lixian.xunlei.com","vod0266.aliyun08.vip.lixian.xunlei.com","vod0267.aliyun08.vip.lixian.xunlei.com","vod3379-aliyun04-vip-lixian.xunlei.com","vod3380-aliyun04-vip-lixian.xunlei.com","vod3429-aliyun04-vip-lixian.xunlei.com","vod3458-aliyun04-vip-lixian.xunlei.com","vod3459-aliyun04-vip-lixian.xunlei.com","vod3496-aliyun04-vip-lixian.xunlei.com","vod3497-aliyun04-vip-lixian.xunlei.com","vod3498-aliyun04-vip-lixian.xunlei.com","vod3499-aliyun04-vip-lixian.xunlei.com","vod3500-aliyun04-vip-lixian.xunlei.com","vod3501-aliyun04-vip-lixian.xunlei.com","vod3522-aliyun04-vip-lixian.xunlei.com","vod3523-aliyun04-vip-lixian.xunlei.com","vod3533-aliyun04-vip-lixian.xunlei.com","vod3534-aliyun04-vip-lixian.xunlei.com","vod3535-aliyun04-vip-lixian.xunlei.com","vod3536-aliyun04-vip-lixian.xunlei.com","vod3549-aliyun04-vip-lixian.xunlei.com","vod3550-aliyun04-vip-lixian.xunlei.com","vod3551-aliyun04-vip-lixian.xunlei.com","vod3552-aliyun04-vip-lixian.xunlei.com","vod3553-aliyun04-vip-lixian.xunlei.com","vod3554-aliyun04-vip-lixian.xunlei.com","vod3555-aliyun04-vip-lixian.xunlei.com","vod0551.aliyun06.vip.lixian.xunlei.com","vod0552.aliyun06.vip.lixian.xunlei.com","vod0553.aliyun06.vip.lixian.xunlei.com","vod0554.aliyun06.vip.lixian.xunlei.com","vod0555.aliyun06.vip.lixian.xunlei.com","vod0556.aliyun06.vip.lixian.xunlei.com","vod0686.aliyun08.vip.lixian.xunlei.com","vod0687.aliyun08.vip.lixian.xunlei.com","vod0688.aliyun08.vip.lixian.xunlei.com","vod0689.aliyun08.vip.lixian.xunlei.com","vod0724.aliyun08.vip.lixian.xunlei.com","vod0725.aliyun08.vip.lixian.xunlei.com","vod0726.aliyun08.vip.lixian.xunlei.com","vod0727.aliyun08.vip.lixian.xunlei.com","vod0728.aliyun08.vip.lixian.xunlei.com","vod0759.aliyun04.vip.lixian.xunlei.com","vod0760.aliyun04.vip.lixian.xunlei.com","vod0769.aliyun04.vip.lixian.xunlei.com","vod0770.aliyun04.vip.lixian.xunlei.com","vod0771.aliyun04.vip.lixian.xunlei.com","vod0772.aliyun04.vip.lixian.xunlei.com","vod0773.aliyun04.vip.lixian.xunlei.com","vod0774.aliyun04.vip.lixian.xunlei.com","vod0775.aliyun04.vip.lixian.xunlei.com","vod0776.aliyun04.vip.lixian.xunlei.com","vod0777.aliyun04.vip.lixian.xunlei.com","vod0778.aliyun04.vip.lixian.xunlei.com","vod0779.aliyun04.vip.lixian.xunlei.com","vod0780.aliyun04.vip.lixian.xunlei.com","vod0781.aliyun04.vip.lixian.xunlei.com","vod3522.aliyun04.vip.lixian.xunlei.com","vod3523.aliyun04.vip.lixian.xunlei.com","vod3533.aliyun04.vip.lixian.xunlei.com","vod3535.aliyun04.vip.lixian.xunlei.com","vod3550.aliyun04.vip.lixian.xunlei.com","vod3551.aliyun04.vip.lixian.xunlei.com","vod3552.aliyun04.vip.lixian.xunlei.com","vod3553.aliyun04.vip.lixian.xunlei.com","vod3554.aliyun04.vip.lixian.xunlei.com","vod3555.aliyun04.vip.lixian.xunlei.com"],"num":"96325","version":"5.8.5","new":"<li class=\"pl-dropdown-menu-item\"><a class=\"pl-a\" data-no-instant=\"1\" style=\"color:#F24C43\" href=\"https://www.youxiaohou.com/install.html?from=update\" target=\"_blank\"><span style=\"margin-right: 5px;\">发现新版</span><svg style=\"animation: load 2.5s cubic-bezier(0.22, 0.61, 0.36, 1) infinite;\" viewBox=\"0 0 1024 1024\" xmlns=\"http://www.w3.org/2000/svg\" width=\"12\" height=\"12\"><path d=\"M171.31 549.028c-24.558-153.572 59.801-308.76 210.442-367.477 111.637-43.53 232.005-22.71 317.236 39.47l-72.527 117.48 325.245-1.254L835.459 0l-59.547 96.426C650.34 15.104 479.156-11.493 329.258 46.95 121.578 127.91 2.038 337.29 25.64 549.03h145.67z\" fill=\"#F24C43\"/><path d=\"M852.688 464.966c24.536 153.572-59.78 308.78-210.422 367.477-102.693 40.024-215.86 24.94-302.874-29.019 16.57-26.895 65.537-106.198 65.537-106.198L55.17 676.704 203.176 1024l62.053-100.484c125.552 81.322 279.592 101.992 429.489 43.55 207.638-80.982 327.22-290.34 303.618-502.058H852.688z\" fill=\"#F24C43\"/></svg></a></li>","footer":"<div style=\"text-align: center;\">点击查看 <a href=\"https://www.youxiaohou.com/zh-cn/motrix.html\" target=\"_blank\" class=\"pl-a\" data-no-instant=\"1\">RPC配置说明</a>，配置修改后自动生效</div>"}
```

通过分析找到可疑项`"num":"96325"`

将其填入后成功“点亮”插件。

![](https://p0.meituan.net/csc/36e0b48503c9148f6f5e3550373e14d68970.png)

定位响应中“暗号”的位置后可以直接编写脚本，解码响应后自动填入，从而绕过引流环节，具体操作本文不再赘述。

## 结语

毫无疑问，各路网盘下载插件的存在大大提高了我们的网盘使用体验，但随之而来的此类行为则为其蒙上了一层阴影。我们不能否认，开发者们在编写和维护这些插件上耗费了大量精力，理应得到用户的支持，但以这样的方式强行引流终归不是一个好的解决方案。“流量”的背后，还是一个个活生生的人。本着尊重用户体验的原则，才能收获最大的转化率与用户黏度。