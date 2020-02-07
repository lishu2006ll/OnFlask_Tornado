grpc for python
============
    通过访问／apidocs   来获得所有的接口标准
    Java Server & Client
https://www.jianshu.com/p/eceb0eff8dce?utm_source=oschina-app

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

跨语言框架级对接中，使用了通用的 service.proto，所以服务端使用反射机制来将请求体映射到对应的具体实现方法，server.py 具体实现示例：

#!/usr/bin/python
# -*- coding: UTF-8 -*-

import importlib
import json
import time
import grpc
from concurrent import futures

import service_pb2
import service_pb2_grpc

_ONE_DAY_IN_SECONDS = 60 * 60 * 24


'''
公共服务
'''
class CommonService(service_pb2_grpc.CommonServiceServicer):
    def handle(self, request, context):
        # 将请求信息转为 json 格式
        _request = json.loads(str(request.request))

        # 反射加载模块及方法
        _module = importlib.import_module(_request['clazz'])
        _method = getattr(_module, _request['method'])

        # 执行方法
        args = _request.get('args')
        if not args:
            _response = "{'status': 0, 'result': %s}" % _method()
        else:
            _response = "{'status': 0, 'result': %s}" % _method(_request['args'])

        return service_pb2.Response(response=bytes(_response))


'''
服务端
'''
def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    service_pb2_grpc.add_CommonServiceServicer_to_server(CommonService(), server)
    server.add_insecure_port('127.0.0.1:6565')
    server.start()
    try:
        while True:
            time.sleep(_ONE_DAY_IN_SECONDS)
    except KeyboardInterrupt:
        server.stop(0)


'''
启动
'''
if __name__ == '__main__':
    print 'server running, listen on 6565'
    serve()
client.py 示例：

#! /usr/bin/env python
# -*- coding: utf-8 -*-

import grpc
import json

import service_pb2
import service_pb2_grpc

'''
请求远程方法-添加用户
'''
def insert():
    user = '{"id":1,"name":"anoyi","age":20}'
    _request = {
        'clazz': 'com.anoyi.grpc.facade.service.UserServiceByFastJSON',
        'method': 'insert',
        'args': [user]
    }
    _request_json = json.dumps(_request, ensure_ascii=False)
    with grpc.insecure_channel('127.0.0.1:6565') as channel:
        stub = service_pb2_grpc.CommonServiceStub(channel)
        response = stub.handle(service_pb2.Request(request=bytes(_request_json), serialize=3))
    print("received: " + response.response)


'''
请求远程方法-查询用户
'''
def findAll():
    _request = {
        'clazz': 'com.anoyi.grpc.facade.service.UserServiceByFastJSON',
        'method': 'findAll'
    }
    _request_json = json.dumps(_request, ensure_ascii=False)
    with grpc.insecure_channel('127.0.0.1:6565') as channel:
        stub = service_pb2_grpc.CommonServiceStub(channel)
        response = stub.handle(service_pb2.Request(request=bytes(_request_json), serialize=3))
    print("received: " + response.response)


'''
启动
'''
if __name__ == '__main__':
    print '--- RPC: insert ---'
    insert()

    print '--- RPC: findAll ---'
    findAll()
UserServiceByFastJSON.py 示例：

#! /usr/bin/env python
# -*- coding: utf-8 -*-
import json

userDatas = {}

def findAll():
    data = json.dumps(userDatas.values(), ensure_ascii=False)
    return data.encode("utf-8")

def insert(args):
    user = json.loads(args[0])
    userDatas[str(user['id'])] = user
    return {}

def deleteById(args):
    userDatas.pop(str(args[0]))
    return {}
Python Server 与 Java Client 对接

1、运行 server.py

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

#!/usr/bin/python
# -*- coding: UTF-8 -*-
2、启动 Java Client，运行 samples-client 的 Application

3、调用测试F