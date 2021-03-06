# [Aria2](https://github.com/aria2/aria2) 的配置
自用配置。伪装UA(提供了几个，可自行修改)，添加了一些`tracker`以适用BT下载。
当前配置适用于 aira2 1.33.0 ↑，理论上低于此版本号会有点小问题，例如peer-agent

注意事项:

1. 对于 PT 用户, `enable-dht bt-enable-lpd enable-peer-exchange`参数是否应该开启还存在争议，建议设置为`false`
1. 加密参数 bt-require-crypto 等 保留软件的默认值了，强制加密之后，许多磁力链根本连种子都获取不到
1. BT/PT 下载默认保存种子，如果不想保存，在`bt-save-metadata=true`前加`#`
1. 文件预分配参数`file-allocation`为不预分配(none)，可自行修改,[参数参考](https://aria2.github.io/manual/en/html/aria2c.html#cmdoption--file-allocation)
1. 如果你不知道前几项在说什么，大可**不必在意**，直接**看后面的**即可
1. **重点** 配置里需要自己修改的参数如下:
    - `dir`为下载文件的保存路径，例如我的`dir=${HOME}/Downloads`。
    - `rpc-secret`为一个可选的口令(推荐使用，非必选，已在行首加#注释)，如果设置了`rpc-secret=12345`, 那么在一些控制界面和浏览器扩展中:
        + **默认** 的 JSON-RPC Path 应该设为 `http://token:12345@127.0.0.1:6800/jsonrpc`
        + **如果** 能分别设置`JSON-RPC Path`和`rpc-secret`，JSON-RPC Path 应设置为 `http://127.0.0.1:6800/jsonrpc`，rpc-secret 直接填
        + **如果** 你当前使用的设备和安装 Aria2 的设备不是同一个设备(即远程/局域网下载管理)，要把`127.0.0.1`替换成安装了 Aria2 的那台设备的 IP 地址。感觉我在普及计算机基础知识ORZ...
    - `input-file` 为 aria2.session 会话文件 的保存位置，该文件用于读取和保存`下载任务列表`。
        + 其实这个文件一开始是空的，随便建立一个保存为这个名字即可。**但这之后就不要改动了！**
    - `save-session` 同上，这两个参数务必保持一致

## 安装 Aria2
在 Mac OS 下推荐用 `brew install aria2` 安装，其他平台请自行判断；[获取源码/可执行文件](https://github.com/aria2/aria2)。
### Andorid
可以在官方项目releases里获取二进制文件，然后把aria2c放到/system/bin/目录，777，然后就能调用了

## 配置文件存放的路径
在 macOS 中，推荐的存放路径为 `~/.aria2/aria2.conf`。其他平台自行决定～

## 启动
```bash
aria2c -D
```
- 推荐写进 shell 脚本(例如本项目的 aria2.sh)，并将它添加到开机启动项。(安卓得把aria2c.sh丢到/system/bin/ 并且改777, 不然应该会报执行权限错误)
- 如果配置文件没按本文推荐的路径放置，启动命令则为`aria2c --conf-path="*****/aria2.conf" -D`。
- 至于 Windows 用户， 可以看看[这个说明](https://github.com/acgotaku/BaiduExporter/tree/master/aria2c)

## Aria2 控制界面
- [AriaNg](https://github.com/mayswind/AriaNg)
    + 频繁维护，界面清新。
    + 不过当前作者貌似主要精力放在维护上面， 没有chrome扩展打包。坐等dalao打包

- 当前个人推荐使用[YAAW](http://binux.github.io/yaaw/demo/) [获取源码](https://github.com/binux/yaaw) [Chrome扩展](https://chrome.google.com/webstore/detail/yaaw-for-chrome/dennnbdlpgjgbcjfgaohdahloollfgoc)
    + Chrome扩展可接管浏览器下载，也可以在网页上右键加入aria2下载
    + 占用资源比较小
    + 在 chrome 上有打包好的扩展
    + 相比 Aria2 WebUI 要简洁易用
    + 最后一次更新源码是在 16年5月底
    + 下载任务进度条上的 已下载百分比 的字体为白色，在刚开始下载时基本看不清
  
- 另外还有[Aria2 WebUI](http://ziahamza.github.io/webui-aria2/) [获取源码](https://github.com/ziahamza/webui-aria2)
    + 有折线图显示速度
    + 更新维护比较频繁
    + 有多语言支持
    + 占用系统资源比较大
    + 添加任务比 YAAW 要多点一次鼠标按键
    + chrome 无法保存界面语言设置，默认显示英文。在 Firefox 上可以保存，其他浏览器未测试

## 浏览器扩展
在浏览器里安装扩展以便快速添加下载链接到 Aria2

- Firefox 可以用 [send-to-aria2](https://addons.mozilla.org/zh-CN/firefox/addon/send-to-aria2)
- Chrome 可以用 [YAAW for Chrome](https://chrome.google.com/webstore/detail/yaaw-for-chrome/dennnbdlpgjgbcjfgaohdahloollfgoc)

###### 引用说明
配置里大部分 tracker 使用了 [@ngosang](https://github.com/ngosang) 的 [trackerslist](https://github.com/ngosang/trackerslist) 项目里提供的 tracker，
有部分 tracker 是从各个动漫 BT 站的种子上 copy 下来的

## 多线程下载百度云盘的资源(测试功能)，不喜欢折腾的别往下看了
**要求会获取文件的下载直链，可以用油猴脚本或者抓包** 

推荐抓包，只要百度云还提供下载服务，那么抓包的方法就不会失效，而油猴脚本有可能被河蟹

### `generateOrder.js`参数说明
domainArr 数组保存的是百度盘的域名，当前数量共31(推荐仅用前22个，后面添加的几个经常拿不到资源)，即最多能开到31线程(百度貌似做了单服务器连接数限制，有些超过两线程就会断线了)。另外也可以将域名替换为解析后的IP，因为一个域名可能有几个IP，选择你的网络下比较快的那个
`generateOrder.js`代码里需要更改的参数有：

- fileURL 这是度盘文件真实下载地址(直链)，抓包获得，或者用脚本导出直链；最好是 HTTPS
- confPath 这是aria2配置文件的路径。
    + 项目里有提供一个示例`bdy.conf`，仅供下载百度云文件使用；因为百度单线程速度和服务器连接数貌似有上限，用aria2.conf的话有可能会被拒绝连接
    + Windows 平台的命令提示符有路径斜杠转义问题，可以 cd 进配置所在目录运行生成的 aria2c 命令，然后 cPath 变量直接填文件名即可(即相对路径)，还不行可以把配置文件重命名，总之请自行解决，这个锅我不背
- connectionNum 这是要生成的 aria2 下载命令的线程数。要求**不大于**domainArr.length

### 用法
要抓包的域名是`d.pcs.baidu.com`

1. 先用百度云盘客户端下载文件，然后抓这个域的包，一般返回的 JSON 里包含有**至少4个**该文件的真实下载地址。抓浏览器的包也可以，不过地址数量我就不保证了      
1. 把获得的其中一个地址填到`generateOrder.js`代码里的`fileURL`变量；推荐用 HTTPS 的，比较稳定      
1. 把`generateOrder.js`里的 JavaScript 代码复制到浏览器控制台，按回车键运行后生成 aria2c 下载命令(字符串形式)，复制命令到 iTerm / Terminal 按回车即可

然后你就能享受多线程高速下载了；文件名**可能**会是一串没后缀的奇怪的数字字母组合，把文件**改个名字就行了**

###### 注意，浏览器运行代码后，控制台返回的是一个字符串，注意删掉字符串开头的一个和末尾的一个双引号。即命令的开头应该没有引号，末尾应该只有一个引号
