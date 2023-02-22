# ニコニコ動画

## 普通视频的下载

需要准备sm号就行了（x

需要用到一个开源的工具库 [ニコニコ_dl](https://github.com/tasuren/niconico_dl)

而该库需要python3及以上的环境，macOS自带的python是2.7的，所以需要自己安装。

> 以下方法同时适用于Mac computers with Apple Silicon & *Intel* processors

1. **打开Terminal查看是否已安装python**

   python2

   ` python --version` 

   python3

   `python3 --version` 

   如果没有的话需要去官网 [Welcome to Python](https://www.python.org) 先安装

2. **查看是否安装了pip**

  python2
  
  `python -m pip --version`
  
  python3
  
  `python3 -m pip --version`
  
3. **手动安装pip**

   `   curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py`

4. **获取pip**

   `python3 get-pip.py`

   该命令运行成功也就可以通过pip安装niconico_dl了

剩下的工具使用方法请参照 [ニコニコ_dl](https://github.com/tasuren/niconico_dl) 中的README，本文不做赘述。

## Nico Live会员有料视频流的抓取

需要ffmpeg、良好的网络访问条件、一个Chrome或其他现代浏览器

> 本文根据macOS情况编写，Windows平台实际使用区别只在于不同的安装方式

1. **安装ffmpeg**

   *macOS用户可通过homebrew安装，命令如下*

   `brew install ffmpeg`

   Windows或者想手动安装的可以去官网获取[Download ffmpeg](https://ffmpeg.org/download.html)

2. **熟悉常用命令**

  [ffmpeg Documentation](https://ffmpeg.org/ffmpeg.html) 
  
3. **获取视频流中的m3u8**

   ![开发者工具中找到timeshift请求](https://bn02pap001files.storage.live.com/y4mlRpvd7PjJQ3WBqhjvn_EI9AT8B7R0GUWFhXLPnf-c-Hf65qqvohiX2Zqhw4v_Dw6PDCjU8lUOBywg_9blU6cVcseAaf550I2OsvJGxBckbRoCQBgp5anA_5z3h0GVl9d5CdlYvgSNCPQI5FXaIgXQDpjcMa9ZPWKTcFFmOI9zONeM6bPgZINqYjvzG_rkvyN?width=2788&height=2206&cropmode=none)

   和平时一样打开ニコ，登录账号，进到想要抓取视频的页面（**先不要播放视频**），右键点击页面在弹出菜单中选择inspect(检查)，在开发者工具中找到Network(网络)标签页，在过滤选项栏中过滤WS的请求，应该会看到左边列表中出现以「timeshift」开头的请求，选中它，然后在右边点击Messages(信息)，找到「{type: "stream"...」开头的JSON数据，展开它的"data"对象，可以看到图中的uri，这个就是我们要抓流的实际地址，保存起来一会儿会用到。

4. **配置terminal的代理（Windows开全局代理和在日用户并不需要配置，可以跳过该步骤）**

   因为直到现在，都是使用终端的操作，而终端并不会跟随系统的代理配置，所以就算你开了全局代理，在terminal中输出`curl cip.cc`应该也一样会返回国内的IP位置。

   为了解决该问题，需要在`~/.bashrc`或者`~/.zshrc`中添加以下配置

   ```shell
   # 设置使用代理
   alias proxyon="export http_proxy=http://127.0.0.1:port; export https_proxy=$http_proxy; export all_proxy=socks5://127.0.0.1:port; echo 'Set proxy successfully'"
   # 设置取消使用代理
   alias proxyoff="unset http_proxy; unset https_proxy; unset all_proxy; echo 'Unset proxy successfully'"
   ```

   代码中"proxyon"以及"proxyoff"可以替换为自己喜欢的好记的名称，"port"则需要改为自己代理的端口号。

   配置完成后便可在terminal中执行proxyon来开启代理了

   在终端中执行`proxyon`，然后用`curl cip.cc`看看输出是否为代理服务器的地址。

   > macOS可以通过 **系统偏好设置 -> 网络 -> 高级 -> 代理** 来查看HTTP/HTTPS以及SOCKS代理

   以上步骤完成后就可以开始尝试抓流了！

5. **使用ffmpeg抓取m3u8流**

   首先将下面shell指令中的"**uri**"替换为你刚才存下来的地址

   > 稍微解释一下上面的"- protocol_whitelist"，"- protocol_whitelist"是协议白名单，因为添加了终端代理，所以需要加上"httpproxy"这一项。

   ```shell
   # macOS
   ffmpeg -protocol_whitelist "file,http,https,tcp,tls,httpproxy" -i "uri" -movflags faststart -c copy -bsf:a aac_adtstoasc "output.mp4"
   
   # Windows
   "ffmpeg.exe" -protocol_whitelist "file,http,https,tcp,tls,httpproxy" -i "uri" -movflags faststart -c copy -bsf:a aac_adtstoasc "output.mp4"
   ```

   在终端输入上面替换uri后的命令然后回车运行，然后同时在视频页点击播放，终端如果输出的是下图这些文字，而不是报错的话，就是成功在抓取了。

   ![终端输出](https://bn02pap001files.storage.live.com/y4mCcu5nivRAl1bOh9r8G4DwhjNaEf2w6QL4aFmgCkpQfqPInjvj-927Z8ZHZhDV-lkW4QQ9JqOJdfHqoIsN6gHnlDX2MHJYmzjM79sR7T60Bd-OGPwsx_DUZDTVmEeQ8YCTIueUDcZ7DWmFFERwP_mFMHvUNic3cjIfgbIbDMjQI3yvPAblA_aFTgGeGCNXGl-?width=2092&height=1550&cropmode=none)

   这时候把终端和浏览器最小化，等视频完整播放完毕就可以完成抓取了。

   完成后会输出一行mp4的信息，表示完成了。

   ![输出](https://bn02pap001files.storage.live.com/y4mW3m1kemLxBEOQAgLD1jAeTTN8f6xMFWeSTEBNreZu9zlq0uy-bhCVKKfaitcOnpUAAzdeuodWBkGgQyNEN-nxDhyBbJSX0aana3VGu8i_LHQVjn2U02aOExVgjXsS7_C9rZWMIm6rpg5gcsRBEmyMFaDFIpnsFPDrr6zPTux2mDkuUlzWuib4bXguxBd6DfM?width=1580&height=298&cropmode=none)

### 本文参考以下博文

[ニコ生の配信を分析し、動画として保存するために頑張った話 @mueru](https://qiita.com/mueru/items/ccd891853db2bbca9cfe)

[Chrome + FFmpegでニコニコの動画を落とすメモ @yokkin](https://yokkin.com/blog/niconicodl.html)

[ffmpegでm3u(m3u8)を一つの動画に結合する方法 @pinkylab](https://qiita.com/pinkylab/items/6f05fb21c7219a680940)

