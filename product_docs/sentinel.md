---
layout: default
---
Last modified:  
2024-09-11  

# 为什么记录？

二道贩子公司要求写一些产品相关的使用记录或者使用体验以便推销给客户，但不知道我为何成为了那个衰仔。
不知为何谷歌中文搜索下确实很少看到有软家产品的使用(食用)记录，但日文或者英文的笔记倒是很多

# 正片之前

其实软家相关的产品只要是只使用软家产品的话，通常来讲下限不会很低。  
但是一旦开始想要把软家的产品和别家的产品绑一起用时，就会闻到非常奇怪的味道。  

# 正片

## 0. Before using
确认是想用自己的服务器还是准备在AZ上购买。
## 1. Preparing
### 1.1 Add server with Azure Arc
如果是准备用自己的环境/服务器，则需要确认自己的OS能不能装Azure Arc Agent，因为需要将自己的环境/服务器连接到Azure才能进行日志连接等操作。  
### 1.2 Azure Virtual Machines
如果准备自己买的话则挑个能安装Azure Monitor Agent的VM即可。我测试时使用的是2vcpu 4GiB memory的Linux机子。
