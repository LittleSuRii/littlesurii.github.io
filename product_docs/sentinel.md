---
layout: default
---
Last modified:  
2024-09-11  

# 为什么记录？

二道贩子公司要求写一些产品相关的使用记录或者使用体验以便推销给客户，但不知道我为何成为了那个衰仔。  
此外，不知为何谷歌中文搜索下确实很少看到有软家产品的使用(食用)记录，但日文或者英文的笔记倒是很多。  

# 正片之前

其实只要是只使用软家产品的话，它/它们通常来讲下限不会很低。  
但是一旦开始想要把软家的产品和别家的产品绑一起用时，就会闻到非常奇怪的味道。  

# 正片
整理了一下我的思绪，其实可以通过以下方式来将日志发送到Azure Monitor，接着Azure Monitor会发送到指定的Log Analytics Workspace，接着再让Sentinel制作检测规则或结合Data Connectors来让警报自动生成。  
（待更新）
## 0. Before using
确认日志的形式，以及是想用自己的服务器还是准备在AZ上购买。
## 1. Preparation
### 1.1 Add server with Azure Arc
如果是准备用自己的环境/服务器，则需要确认自己的OS能不能装Azure Arc Agent，因为需要将自己的环境/服务器连接到Azure才能进行日志连接等操作。  
### 1.2 Azure Virtual Machines
如果准备自己买的话则挑个能安装Azure Monitor Agent的VM即可。我测试时使用的是2vcpu 4GiB memory的Linux机子(OS是Ubuntu 22.04)。
### 1.3 Azure Monitor Agent
为什么选择AMA而不是Legacy Agent? 因为微软宣布从2024-08-31以后停止支持Legacy Agent。  
安装这个玩意挺邪门的。一般是在建立Data collection rules时会给你的VM装上。  
如果你的VM在关机时建立了Data collection rules的话那自然是装不上的。  
此外，正确建立Data collection rules的方法是先建立一个Data collection endpoint，这样你在建立Data Collection rules时则可以直接选择你的Endpoint，而不是在前进到resources时建立一个新的endpoint以后发现没法在第一页(Basic)里选择刚刚建立的Endpoint，导致Data source里可选择的日志只有Linux Syslog而不是Custom Logs。安装成功/失败的话可以在VM页面的extensions+applications里看到结果。
#### Data Collection Rules Snapshot
![Data Collection Rules](https://littlesurii.github.io/imgs/sentinel/sentinel_data_collection_rules_creation.png)
#### AMA Status Snapshot
![Azure Monitor Agent](https://littlesurii.github.io/imgs/sentinel/sentinel_ama_status.png)

## 2. Logs
解决完了服务器的AMA问题，接下来再看日志怎样导入到Sentinel。导入之前需要创建Log Analytics Workspace，然后再建立一个Sentinel。当然你在创建一个Sentinel过程中微软会要求你创建一个Log Analytics Workspace，所以直接点Sentinel->Create即可。  
#### Sentinel creation Snapshot
![Sentinel](https://littlesurii.github.io/imgs/sentinel/sentinel_creation.png)
### 2.1 Syslog
