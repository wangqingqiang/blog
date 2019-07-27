# DNS查询
DNS解析基于UDP协议  

![DNS](imags/dns.jpg)  

**DNS可以做负载均衡**

# HTTPDNS
DNS查询慢，缓存导致更新不及时等问题。  
HTTPDNS 发送一个http请求获取对应域名的IP，类似于请求个接口，HTTPDNS服务器会根据负载情况返回IP。  
