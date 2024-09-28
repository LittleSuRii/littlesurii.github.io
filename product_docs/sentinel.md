---
layout: default
---
Last modified:  
2024-09-28  

# 为什么记录？

二道贩子公司要求写一些产品相关的使用记录或者使用体验以便推销给客户, 但不知道我为何成为了那个衰仔。  
此外, 不知为何谷歌中文搜索下确实很少看到有软家产品的使用(食用)记录, 但日文或者英文的笔记倒是很多。  

# 正片之前

其实只要是只使用软家产品的话, 它/它们通常来讲下限不会很低。  
但是一旦开始想要把软家的产品和别家的产品绑一起用时, 就会闻到非常奇怪的味道。  
同时欢迎各位捉虫。  

# 正片
整理了一下我的思绪, 其实可以通过以下方式来将日志发送到Azure Monitor, 接着Azure Monitor会发送到指定的Log Analytics Workspace, 接着再让Sentinel制作检测规则或结合Data Connectors来让警报自动生成。  
（待更新）
## 0. Before using
确认日志的形式, 以及是想用自己的服务器还是准备在AZ上购买。
## 1. Preparation
### 1.1 Add server with Azure Arc
如果是准备用自己的环境/服务器, 则需要确认自己的OS能不能装Azure Arc Agent, 因为需要将自己的环境/服务器连接到Azure才能进行日志连接等操作。  
### 1.2 Azure Virtual Machines
如果准备自己买的话则挑个能安装Azure Monitor Agent的VM即可。我测试时使用的是2vcpu 4GiB memory的Linux机子(OS是Ubuntu 22.04)。
### 1.3 Azure Monitor Agent
为什么选择AMA而不是Legacy Agent? 因为微软宣布从2024-08-31以后停止支持Legacy Agent。  
安装这个玩意挺邪门的。一般是在建立Data collection rules时会给你的VM装上。  
如果你的VM在关机时建立了Data collection rules的话那自然是装不上的(再开机也不会装上)。  
此外, 正确建立Data collection rules的方法是先建立一个Data collection endpoint, 这样你在建立Data Collection rules时则可以直接选择你的Endpoint, 而不是在前进到resources时建立一个新的endpoint以后发现没法在第一页(Basic)里选择刚刚建立的Endpoint, 导致Data source里可选择的日志只有Linux Syslog或者是Windows日志而不是Custom Logs。安装成功/失败的话可以在VM页面的extensions+applications里看到结果。
##### Data Collection Rules
![Data Collection Rules](https://littlesurii.github.io/imgs/sentinel/sentinel_data_collection_rules_creation.png)
创建该规则时, 你需要注意的是你的日志是什么, 以及你的日志在哪里。
##### AMA Status
![Azure Monitor Agent](https://littlesurii.github.io/imgs/sentinel/sentinel_ama_status.png)
可以从该页面确认AMA是否安装成功。
## 2. Logs
解决完了服务器的AMA问题, 接下来就是将日志导入到Sentinel。导入之前需要创建Log Analytics Workspace, 然后再建立一个Sentinel工作区。当然你在创建一个Sentinel工作区时微软会要求你创建一个Log Analytics Workspace, 所以直接点Sentinel->Create即可。  
##### Sentinel creation
![Sentinel](https://littlesurii.github.io/imgs/sentinel/sentinel_creation.png)  
### 2.1 Syslog
如果想要将当前VM的Syslog导入至Sentinel中, 可以通过创建Data Collection Rules, 选择你的data source来自于哪个VM后再将Data source设置为Syslog即可。由于Log Analytics Workspace本身就含有Syslog的table, 所以你不需要额外创建一个table来解析日志。
##### Data collection rules creation
![Syslog](https://littlesurii.github.io/imgs/sentinel/sentinel_log_source.png)
### 2.2 Custom Logs
Custom logs通常只有两种形式可以被Sentinel识别接收, 聪明的你在创建Data Collection Rules时就发现了是哪两种。而Sentinel并不像Splunk一样拥有自动识别日志field的功能, 所以对于文字(Text)形式记录的日志通常要给各个分段添加命名, 且必须有一个TimeGenerated的项目(该项目会在微软接收某条日志时自动生成, 也可以自己选择日志记录的时间转换为该项目), 故在解析Text形式的日志或Json日志时并不如Splunk那样直接, 简单且方便。  
为了方便测试Sentinel的日志搜集功能, 本文采用了Nginx默认配置下生成的访问日志作为数据源。
#### 2.2.1 Text Logs
假设你在当前设备下有任意可以读取的日志置于任意目录下, 且其大致内容(Apache/Nginx的日志内容)假设如下:
```
127.0.0.1 - - [14/Sep/2024:15:16:40 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.68.0" "-"
127.0.0.1 - - [14/Sep/2024:15:16:51 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.68.0" "-"
```
那么对于上述日志搜集, 你可以通过自行创建data collection rules, 或者是使用 Azure sentinel content hub 内的 Custom logs via AMA (Preview) 的解决方案来进行(推荐)。在不修改Nginx配置的情况下, 默认日志会以filename.log的名称保存在 /var/log/ngnix/目录下。  
如果是自行创建data collection rules, 你需要在Log Analytics Workspace内先创建一个Table。你可以选择DCR-Based(Data collection rules based, 基于创建数据收集规则)的方式来创建Table, 也可以通过MMA的方式来创建Table。
##### Custom table creation
![Table](https://littlesurii.github.io/imgs/sentinel/sentinel_custom_table_creation.jpg)  
由于通过DCR创建时只能选择Json格式的日志文件作为模板, 所以你可以通过GPT来帮你解决Text到Json的过程, 让它随意定义一些field名称, 或者你自己在Nginx配置文件中决定这些名称。  
##### Transformation
![Table](https://littlesurii.github.io/imgs/sentinel/sentinel_custom_table_settings.jpg)  
但Sentinel只支持[部分](https://learn.microsoft.com/en-us/kusto/query/scalar-data-types/datetime?view=microsoft-fabric)时间格式, 故当前日志的时间格式无法被接受。需要做的只是将它修改成sentinel可以接受的形式, 然后保存。  
通过Content hub 内的 Custom logs via AMA (Preview) 的解决方案来一把梭地创建Table获取日志的操作如下:  
Content Hub -> Install Custom logs via AMA (Preview) -> Data Connector -> Custom logs via AMA (Preview) -> Create Data Collection Rules  
##### Data connector
![Table](https://littlesurii.github.io/imgs/sentinel/sentinel_custom_logs_ama_solution.jpg)  
在创建Table时你可以选择自定义或者是默认提供给你的Nginx日志的解决方案。  
##### Data collection rules creation in Data connector
![Table](https://littlesurii.github.io/imgs/sentinel/sentinel_custom logs creation.jpg)  
而Transform这一栏你可以默认只填一个source, 默认source的情况下你在本地写入的所有日志信息都会以出现在RawData的项目下。也可以通过解析RawData来自行创建一些Field来配合需要储存的项目。例如:  
```
source
| parse RawData with ip " - - [" date_time: string "] \"" method" "url" "protocol "\" " status:int" "length:int " \"" referrer "\" \"" userAgent "\" \"" _ 
| extend datetime_transformed = replace(@'(\d{2})/(\w{3})/(\d{4}):(\d{2}:\d{2}:\d{2}) \+0000', @"\3-\2-\1T\4Z", date_time) 
| extend datetime_transformed = replace("Jan", "01", datetime_transformed) 
| extend datetime_transformed = replace("Feb", "02", datetime_transformed) 
| extend datetime_transformed = replace("Mar", "03", datetime_transformed) 
| extend datetime_transformed = replace("Apr", "04", datetime_transformed) 
| extend datetime_transformed = replace("May", "05", datetime_transformed) 
| extend datetime_transformed = replace("Jun", "06", datetime_transformed) 
| extend datetime_transformed = replace("Jul", "07", datetime_transformed) 
| extend datetime_transformed = replace("Aug", "08", datetime_transformed) 
| extend datetime_transformed = replace("Sep", "09", datetime_transformed) 
| extend datetime_transformed = replace("Oct", "10", datetime_transformed) 
| extend datetime_transformed = replace("Nov", "11", datetime_transformed) 
| extend datetime_transformed = replace("Dec", "12", datetime_transformed) 
| extend datetime_parsed = todatetime(datetime_transformed) 
| project TimeGenerated, RawData, datetime_parsed, ip, method, url, protocol, status, length, referrer, userAgent
```
经过上述变换后, 你还需要添加NGINX_CL的表格项目以达到数据同步保存, 这样你就能在日志当中看到信息。  
##### Table settings
![Table](https://littlesurii.github.io/imgs/sentinel/sentinel_nginx_table.jpg)  
##### Logs
![Table](https://littlesurii.github.io/imgs/sentinel/sentinel_nginx_log_results.jpg)  
上述变换是基于Text格式的日志来进行的, 由于Nginx配置文件修改后可以定义你想要的信息, 比如端口号之类的, 所以请根据你的实际情况来进行调整。  
也许你会在想:“KQL怎么这么长?不能通过regex来直接一把梭吗?”  
不能, 因为这不是Splunk。  
##### Functions
你也可以不对日志进行任何变换, 通过调用Data Connectors内部的NGINXHTTPServer函数来直接进行标准化解析, 解析结果如下。  
![Functions](https://littlesurii.github.io/imgs/sentinel/sentinel_nginx_log_functions.jpg)
此外, 只有在完成创建Table以及Data Collection Rules后, 新写入的日志才会被发送到Azure Monitor中, 在这之前生成的日志不会被发送。  
#### 2.2.2 Custom Logs
然而Custom Json Log处于更新中, 只能使用Azure CLI来导入Json信息而不是通过DCR, 只能说一句狗屎微软(Azure CLI真不熟)。

