> https://jibinquan.github.io/posts/cloudflare2v6/

通过Cloudflare实现IPv6免流上网
2025年4月9日
·
1655 字
·
4 分钟
技术
网络
Ji Binquan
作者
Ji Binquan
现在也没研究明白
背景
实现方法
第一步：fork edgetunnel
第二步：Cloudflare部署
第三步：pages变量设置
第四步：重新部署Pages
第五步：订阅到Clash
实现原理分析
反向代理与协议转换
配置与变量定制
背景
#

校园网高昂的流量费用让许多学生苦不堪言。然而，许多高校的IPv6网络流量是免费的，因此在使用百度、知乎等支持IPv6访问的网站时，可以显著节省流量消耗。然而，这一优势在访问仅支持IPv4的网站（如B站等视频平台）时却无法体现，这使得流量消耗最大的场景反而无法享受免费红利，就是离谱。

本文将介绍一种技术方案：通过Cloudflare Workers和Pages服务部署反向代理节点，实现通过IPv6网络访问仅支持IPv4的网站。该方法利用Cloudflare的全球分布式网络，将请求经由支持IPv6的路径转发至目标服务器，从而绕过校园网IPv4流量计费限制。

本文基于edgetunnel项目实现，项目连接： cmliu/edgetunnel: 在原版的基础上修改了显示 VLESS 配置信息转换为订阅内容。使用该脚本，你可以方便地将 VLESS 配置信息使用在线配置转换到 Clash 或 Singbox 等工具中。

edgetunnel部署教程： CF新服务条款解读 & edgetunnel最新教程：全面解析与功能演示 新老玩家必看内容 CF VLESS 免费节点 优选订阅 Workers & Pages CM喂饭干货满满27 #科学上网

实现方法
#

第一步：fork edgetunnel
#

首先，访问 edgetunnel 项目 并 fork 到自己的仓库。

image-20250408165110026
第二步：Cloudflare部署
#

注册 Cloudflare 免费账号： 个人 | Cloudflare 注册并完成邮箱验证后，进入 Cloudflare 控制台中的 Workers 和 Pages 服务。
image-20250409153102005
点击“创建” → 选择 Pages → 通过导入现有 Git 存储库创建项目。
image-20250408164908371
登录 GitHub 后选择你 fork 后的 edgetunnel 项目。 请注意，项目名称不能命名为 “edgetunnel”，其他任意名称均可。保存并启动部署。
image-20250408165339583
部署过程中会显示项目的部署状态。

image-20250408165505190
image-20250408165547073
第三步：pages变量设置
#

在 Cloudflare Pages 控制台中，点击你刚刚部署的 Page。
image-20250409153006102
进入“设置” → “变量和机密”
image-20250408165923718
添加以下变量：UUID、ADD、ADDAPI 和 PROXYIP。

image-20250409105629724
在弹出的窗口中逐个添加变量，变量类型均选择“文本”。

image-20250409150139889
各变量的参考值如下：

UUID: 可设置任意字符串（建议使用小写字母和数字组合）。

ADD:

[2400:cb00:f00e:a8:5d:c71d:2c71:7c63]
[2803:f800:50:0:d6ce:3a6:7088:f992]
[2803:f800:51::f49d:b305:cbc7]
[2803:f800:51::84f0:bd51:326a]
ADDAPI:

https://addressesapi.090227.xyz/cmcc-ipv6
https://addressesapi.090227.xyz/ct
https://addressesapi.090227.xyz/cmcc
PROXYIP:

ProxylPSG.CMLiussss.net
第四步：重新部署Pages
#

设置完变量后，返回部署页，重试部署

image-20250409150848025
第五步：订阅到Clash
#

部署完成后，进入相应页面点击访问，会跳转到一个特定网址。

image-20250409151024010
在该网址后面添加你之前设置的 /UUID 参数：

image-20250409151609008
下载Clash-verge，导入复制的链接即可完成订阅的导入

image-20250409151729804
实现原理分析
#

尚不明确，GPT写的大概看看吧：

反向代理与协议转换
#

利用 Cloudflare 的 Workers 和 Pages 服务，可以搭建一个反向代理节点。反向代理工作原理如下：

请求转发： 当用户发起一个访问请求时，Cloudflare 将该请求首先路由到部署在其边缘节点的 Worker 服务。Worker 服务接受客户端的请求后，再将请求转发至目标服务器（仅支持 IPv4 的网站）。
协议转换： Worker 层既支持 IPv6 也支持 IPv4，通过配置不同的网络接口，Cloudflare 可以在源端使用 IPv6进行传输，再将请求转换并通过其内部网络转发为 IPv4请求。这一过程完全在 Cloudflare 自身内部网络完成，有效地绕过了校园网对于 IPv4 流量的计费限制。
配置与变量定制
#

通过将关键参数（如 UUID、ADD、ADDAPI 和 PROXYIP）设为 Cloudflare Pages 的环境变量，系统得以灵活管理和动态调整：

UUID 用于标识用户或特定的订阅配置，使每个请求可以关联到特定的配置。
ADD 与 ADDAPI 则为内部提供地址解析和接口调用，决定了请求如何在 Cloudflare 内部路由及转换。
PROXYIP 则用来指定目标代理服务器的地址，从而实现最终请求的转发及响应返回。
免责声明：

本文所述技术方案仅供网络技术学习交流，严禁用于任何形式的非法网络访问或流量欺诈行为
使用者应确保自身行为符合所在国家/地区的法律法规及所在高校的网络使用规定，因技术滥用导致的一切后果由使用者自行承担
Cloudflare服务存在明确的使用条款限制（ 服务协议），部署者需自行确保技术实现方式不违反服务提供商的相关规定
网络协议转换可能带来潜在的网络安全风险，包括但不限于数据泄露、网络攻击等安全隐患
作者不对任何因参照本文实施造成的账号封禁、法律纠纷或经济损失承担连带责任
高校网络政策存在动态调整可能，实施前请务必确认本校IPv6流量计费规则
本技术方案可能因服务商策略变更随时失效，请保持对网络技术发展的持续关注
