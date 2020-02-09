flask_app server for python
============
支持功能如下：

    1. 支持sockjs，支持websocket进行通信，可作为客户系统对外提供服务， 你可以为每一个sockjs 路由实现不同的session进行通信，管理多个不同的session;
    
    2. 支持ES: Elasticsearch支持很多查询方式，其中一种就是DSL，它是把请求写在JSON里面，然后进行相关的查询,SL查询语言中存在两种：查询DSL（query DSL）和过滤DSL（filter DSL）;
    
    3. 支持rtsp 与rtmp: 框架支持rtsp进行流媒体操作和处理, 可以根据不同的协议类型进行推流和拉流操作;
    
    4. 支持flask_admin， 可以通过flask_admin快速生产，做后台管理很轻松;
    
    5. 支持flask和tornado切换，可以根据不同业务需求选择不同的启动方式;
    
    6. 继承定时任务机制，支持quartzjob， 可以在页面上定制定制任务，进行业务动态调度;
    
    7. 支持grpc服务, 支持跨语言访问;
    
    8. 继承swagger， 提供apidocs,提供在线调试接口，支持集访问模式;
    
    9. 支持人脸识别程序，集成recognize， 提供在线人脸识别接口，对人脸进行识别;
    
    10. 支持流媒体，支持海康和大华等摄像头集成，可以远程拉流和对流媒体进行处理;
    
    11. 集成nlp中的词向量和jieba分词，用于自然语言处理;
    


Usage
-----
可以在客户端直接通信::

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

 --ES使用--::
 
    query DSL:查询上下文 是在 使用query进行查询时的执行环境，比如使用search的时候
    filter DSL:它不会去计算任何分值，也不会关心返回的排序问题，因此效率会高一点,经常使用过滤器，ES会自动的缓存过滤器的内容，这对于查询来说，会提高很多性能

 --流媒体--::
 
    App：RTMP的Application（应用）名称，可以类比为文件夹。以文件夹来分类不同的流，没有特殊约定，可以任意划分
    Stream：RTMP的Stream（流）名称，可以类比为文件
    HTTP	Host	Port	Vhost
    http://xxxxxxxxx:80/players/a_player.html	192.168.1.10	80	demo.srs.com
    rtmp://xxxxxx:1935/live/livestream	192.168.1.10	1935	demo.srs.com

grpc for python
============
grpc-python 准备工作

0、序列化方式选择

使用 JSON 文本

1、安装 Python 相关库

pip install grpcio

pip install protobuf

pip install grpcio-tools
2、Python 工程相关目录结构说明
com/anoyi/grpc/facade/service/UserServiceByFastJSON.py
对应 Java 模块 samples-facade 中定义的接口 UserServiceByFastJSON
client.py
python grpc 客户端，用于远程调用服务端的方法
server.py
python grpc 服务端，用于提供方法实现，供客户端调用
service.proto
通用的 proto 文件，与 spring-boot-starter-grpc 中定义的一致
service_pb2.py 和 service_pb2_grpc.py
由工具 grpcio-tools 根据  service.proto 生成，生成方式如下：
python -m grpc_tools.protoc -I./ --python_out=. --grpc_python_out=. ./service.proto
Python Server & Java Client
3、运行 server.py

python server.py
常见问题①
Traceback (most recent call last):
  File "server.py", line 10, in <module>
    import service_pb2
  File "/Users/admin/code/python/python-grpc/service_pb2.py", line 22, in <module>
    serialized_pb=_b('\n\rservice.proto\"-\n\x07Request\x12\x11\n\tserialize\x18\x01 \x01(\x05\x12\x0f\n\x07request\x18\x02 \x01(\x0c\"\x1c\n\x08Response\x12\x10\n\x08response\x18\x01 \x01(\x0c\x32\x30\n\rCommonService\x12\x1f\n\x06handle\x12\x08.Request\x1a\t.Response\"\x00\x42\x1e\n\rcom.anoyi.rpcB\x0bGrpcServiceP\x00\x62\x06proto3')
TypeError: __init__() got an unexpected keyword argument 'serialized_options'
解决方案：修改 service_pb2.py 文件，将所有的 serialized_options 替换为 options

常见问题②
Traceback (most recent call last):
  File "server.py", line 11, in <module>
    import service_pb2_grpc
  File "/Users/admin/code/python/python-grpc/service_pb2_grpc.py", line 8
SyntaxError: Non-ASCII character '\xe5' in file /Users/admin/code/python/python-grpc/service_pb2_grpc.py on line 8, but no encoding declared; see http://python.org/dev/peps/pep-0263/ for details
解决方案：修改 service_pb2_grpc.py，在文件头部添加如下内容：

Installation
------------
1. 安装虚拟机::

    $ wget https://raw.github.com/pypa/virtualenv/master/virtualenv.py

2. 安装应用:
    python setup.py

3. 运行::
    python app.py

Requirements
    requirements.txt

Authors
-------

`flask_app` was written by `lishu2006ll@163.com`_.


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
