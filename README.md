# Scrapy-proxies

一个简单的动态代理池，通过较高频率的自检保证池内代理的高可靠性。

# 代码结构
代码分三个部分：
*  一个scrapy爬虫去爬代理网站，获取免费代理，验证后入库   (proxy_fetch)
*  一个scrapy爬虫把代理池内的代理全部验证一遍，若验证失败就从代理池内删除   (proxy_check)
*  一个调度程序用于管理上面两个爬虫   (start.py)

![Scrapy-proxies.png](http://upload-images.jianshu.io/upload_images/4610828-edbea71e6ff36157.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 部署
需要先改一下配置文件hq-proxies.yml，把Redis的地址密码之类的填上，改完后放到/etc/hq-proxies.yml下。
在配置文件中也可以调整相应的阈值和免费代理源和测试页面。


```python
#中间件
import redis
import random

class DynamicProxyMiddleware(object):
    def process_request(self, request, spider):
        redis_db = redis.StrictRedis(
            host= '127.0.0.1', 
            port= 6379, 
            password= '',
            db= 6
        ) 
        proxy = random.choice(list(redis_db.smembers("hq-proxies:proxy_pool"))).decode('utf-8')
        spider.logger.debug('使用代理[%s]访问[%s]' % (proxy, request.url))
        request.meta['proxy'] = proxy
```


