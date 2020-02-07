flask_app server for Flask
============

.. image:: http://lis
    :target: http://lis
    :alt: Latest PyPI version

.. image:: http://lis
   :target: http://lis
   :alt: Latest Travis CI build status

`flask_app` is a `flask_ingeg` integration for Flask.   interface
is implemented as a flask route. Its possible to create any number of different sockjs routes, ie `/sockjs/*` or `/mycustom-sockjs/*`.
You can provide different session implementation and management for each sockjs route.

Usage
-----

Client side code::

  <script src="//cdnjs.cloudflare.com/ajax/libs/sockjs-client/1.1.4/sockjs.js"></script>
  <script>
      var sock = new SockJS('http://localhost:5000/sockjs');

      sock.onopen = function() {
        console.log('open');
      };

      sock.onmessage = function(obj) {
        console.log(obj);
      };

      sock.onclose = function() {
        console.log('close');
      };
  </script>

 ES::
    Elasticsearch支持很多查询方式，其中一种就是DSL，它是把请求写在JSON里面，然后进行相关的查询
    DSL查询语言中存在两种：查询DSL（query DSL）和过滤DSL（filter DSL）
    query DSL:查询上下文 是在 使用query进行查询时的执行环境，比如使用search的时候
    filter DSL:它不会去计算任何分值，也不会关心返回的排序问题，因此效率会高一点,经常使用过滤器，ES会自动的缓存过滤器的内容，这对于查询来说，会提高很多性能

 SRS::
    1.
App：RTMP的Application（应用）名称，可以类比为文件夹。以文件夹来分类不同的流，没有特殊约定，可以任意划分
Stream：RTMP的Stream（流）名称，可以类比为文件

2.Vhost
一个分发网络支持多个客户：譬如CDN，一个分发网络中，有N个客户公用一套流媒体系统，如何区分用户，计费，监控等等？通过app么？大家可能都叫做live之类。最好是通过各自的域名。
不同的应用配置：譬如FMLE推上来的流是h264+mp3，可以将音频转码后放到其他的vhost分发hls，这样接入h264+mp3的vhost就不用切hls。

3. NoVhost
其实，vhost大多数用户都用不到，而且不推荐用，有点复杂。一般的用户用app就可以了。因为vhost/app/stream，只是一个分类方法而已；vhost需要在配置文件中说明，app/stream都不需要配置。

什么时候用vhost？如果你是提供服务，譬如你有100个客户，都要用一套平台，走同样的流媒体服务器分发。那可以每个客户一个vhost，这样他们的app和stream可以相同都可以。

一般的用法，举个例子，有个视频网站，自己搭建服务器，所以只有他自己一个客户，就不要用vhost了，直接用app就足够了。假设视频网站提供聊天服务，聊天有不同的话题类型，每个话题就是一个app，譬如：军事栏目，读书栏目，历史栏目三个分类，每个分类下面有很多聊天室。只要这么配置就好：

listen              1935;
vhost __defaultVhost__ {
}
生成网页时，譬如军事栏目的网页，都用app名称为military，某个聊天室叫做火箭，这个页面的流可以用：rtmp://yourdomain.com/military/rock，编码器也推这个流，所有观看这个军事栏目/火箭聊天室的页面的人，都播放这个流。
军事栏目另外的网页，都用同样的app名称military，但是流不一样，譬如某个聊天室叫做雷达，这个页面的流可以用：rtmp://yourdomain.com/military/radar，推流和观看一样。
如此类推，军事栏目页面生成时，不用更改srs的任何配置。也就是说，新增聊天室，不用改服务器配置；新增分类，譬如加个公开课的聊天室，也不用改服务器配置。足够简单！
另外，读书栏目可以用app名称为reader，栏目下的某个聊天室叫红楼梦，这个页面的流可以用：rtmp://yourdomain.com/reader/red_mansion，所有在这个聊天室的人都是播放这个流。


Vhost的应用
RTMP的Vhost和HTTP的Vhost概念是一样的：虚拟主机。详见下表（假设域名demo.srs.com被解析到IP为192.168.1.10的服务器）
RTMP的Vhost和HTTP的Vhost概念是一样的：虚拟主机。详见下表（假设域名demo.srs.com被解析到IP为192.168.1.10的服务器）：

HTTP	Host	Port	Vhost
http://demo.srs.com:80/players/srs_player.html	192.168.1.10	80	demo.srs.com
rtmp://demo.srs.com:1935/live/livestream	192.168.1.10	1935	demo.srs.com
Vhost主要的作用是：

支持多用户：当一台服务器需要服务多个客户，譬如CDN有cctv（央视）和wasu（华数传媒）两个客户时，如何隔离他们两个的资源？相当于不同的用户共用一台计算机，他们可以在自己的文件系统建立同样的文件目录结构，但是彼此不会冲突。
域名调度：CDN分发内容时，需要让用户访问离自己最近的边缘节点，边缘节点再从源站或上层节点获取数据，达到加速访问的效果。一般的做法就是Host是DNS域名，这样可以根据用户的信息解析到不同的节点。
支持多配置：有时候需要使用不同的配置，考虑一个支持多终端（PC/Apple/Android）的应用，PC上RTMP分发，Apple和Android是HLS分发，如何让PC延迟最低，同时HLS也能支持，而且终端播放时尽量地址一致（降低终端开发难度）？可以使用两个Vhost，PC和HLS；PC配置为最低延迟的RTMP，并且将流转发给HLS的Vhost，可以对音频转码（可能不是H264/AAC）后切片为HLS。PC和HLS这两个Vhost的配置肯定是不一样的，播放时，流名称是一样，只需要使用不同的Host就可以。

Vhost支持多用户

假设cctv和wasu都运行在一台边缘节点(192.168.1.10)上，用户访问这两个媒体的流时，Vhost的作用见下表：

RTMP	Host	Port	Vhost	App	Stream
rtmp://show.cctv.cn/live/livestream	192.168.1.10	1935	show.cctv.cn	live	livestream
rtmp://show.wasu.cn/live/livestream	192.168.1.10	1935	show.wasu.cn	live	livestream
在边缘节点（192.168.1.10）上的SRS，需要配置Vhost，例如：

listen              1935;
vhost show.cctv.cn {
}
vhost show.wasu.cn {
}
Vhost域名调度

详细参考DNS和CDN的实现。

Vhost支持多配置

以上面举的例子，若cctv需要延迟最低（意味着启动时只有声音，画面是黑屏），而wasu需要快速启动（打开就能看到视频，服务器cache了最后一个gop，延迟会较大）。

只需要对这两个Vhost进行不同的配置，例如：

listen              1935;
vhost show.cctv.cn {
    chunk_size 128;
}
vhost show.wasu.cn {
    chunk_size 4906;
}
总之，这两个Vhost的配置完全没有关系，不会相互影响。

__defaultVhost__

FMS的__defaultVhost__是默认的vhost，当用户请求的vhost没有匹配成功时，若配置了defaultVhost，则使用它来提供服务。若匹配失败，也没有defaultVhost，则返回错误。

譬如，服务器192.168.1.10上的SRS配置如下：

listen              1935;
vhost demo.srs.com {
    enabled         on;
}
那么，当用户访问以下vhost时：

rtmp://demo.srs.com/live/livestream：成功，匹配vhost为demo.srs.com
rtmp://192.168.1.10/live/livestream：失败，没有找到vhost，也没有defaultVhost。
defaultVhost和其他vhost的规则一样，只是用来匹配那些没有匹配成功的vhost的请求的。

访问指定的Vhost

如何访问某台服务器上的Vhost？有两个方法：

配置hosts：因为Vhost实际上就是DNS解析，所以可以配置客户端的hosts，将域名（Vhost）解析到指定的服务器，就可以访问这台服务器上的指定的vhost。
使用app的参数：需要服务器支持。在app后面带参数指定要访问的Vhost。SRS支持?vhost=VHOST和...vhost...VHOST这两种方式，后面的方式是避免一些播放器不识别？和=等特殊字符。

普通用户不用这么麻烦，直接访问RTMP地址就好了，有时候运维需要看某台机器上的Vhost的流是否有问题，就需要这种特殊的访问方式。考虑下面的例子：
RTMP URL: rtmp://demo.srs.com/live/livestream
边缘节点数目：50台
边缘节点IP：192.168.1.100 至 192.168.1.150
边缘节点SRS配置：
    listen              1935;
    vhost demo.srs.com {
        mode remote;
        origin: xxxxxxx;
    }

各种访问方式见下表：

用户	RTMP URL	hosts设置	目标
普通用户	rtmp://demo.srs.com/live/livestream	无	由DNS
解析到指定边缘
运维	rtmp://demo.srs.com/live/livestream	192.168.1.100 demo.srs.com	查看192.168.1.100上的流
运维	rtmp://192.168.1.100/live?
vhost=demo.srs.com/livestream	无	查看192.168.1.100上的流
运维	rtmp://192.168.1.100/live
...vhost...demo.srs.com/livestream	无	查看192.168.1.100上的流



Installation
------------
1. Install virtualenv::

    $ wget https://raw.github.com/pypa/virtualenv/master/virtualenv.py
    $ python3.6 ./virtualenv.py --no-site-packages sockjs

3. Install sockjs-flask from pypi and then install::

    $ pip install sockjs-flask
    $ cd sockjs
    $ ../sockjs/bin/python setup.py

To run chat example use following command::

    $ ./sockjs/bin/python ./sockjs-flask/examples/main.py

Requirements
^^^^^^^^^^^^
requirements.txt

Authors
-------

`flask_app` was written by `lis`_.


开发步骤
-------
1. Initializing the database
    flask create-database

2. Running Flask
    flask run
3. 


Features
--------

1. 打开索引:
    curl -i -XPOST 'http://localhost:9200/my_index/_open

2. 根据command 创建初始化数据库
    flask create-database


3. swagger：
    通过访问／apidocs   来获得所有的接口标准