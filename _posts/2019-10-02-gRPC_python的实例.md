---
title: gRPC的实例
description: gRPC的实例
categories:
- python
tags:
- gRPC
---


### 定义服务



在`.proto`文件中 使用 `protocol buffers`([Protocol Buffers](https://developers.google.com/protocol-buffers/docs/overview)) 定义

`gRPC`的`service 和 request , response`的类型


#### 1 定义一个 `service 服务`

    
    service RouteGuide {
       // (Method definitions not shown)
    }



#### 2 定义具体的 `rpc 方法`, 需要指定 请求request 和 响应response的类型


`gRPC` 中有四种类型的service方法：


- 一个 简单 RPC ， 客户端使用存根发送请求到服务器并等待响应返回，就像平常的函数调用一样。


    
         // Obtains the feature at a given position.
         
           rpc GetFeature(Point) returns (Feature) {}
    


- 一个 应答流式 RPC ， 客户端发送请求到服务器，拿到一个流去读取返回的消息序列。 客户端读取返回的流，直到里面没有任何消息。

从例子中可以看出，通过在 响应 类型前插入 `stream 关键字`，可以指定一个服务器端的流方法。

    
    / Obtains the Features available within the given Rectangle.  Results are
      // streamed rather than returned at once (e.g. in a response message with a
      // repeated field), as the rectangle may cover a large area and contain a
      // huge number of features.
      rpc ListFeatures(Rectangle) returns (stream Feature) {}


- 一个 请求流式 RPC ， 客户端写入一个消息序列并将其发送到服务器，同样也是使用流。一旦客户端完成写入消息，它等待服务器完成读取返回它的响应。

通过在 请求 类型前指定 stream 关键字来指定一个客户端的流方法。
    
    // Accepts a stream of Points on a route being traversed, returning a
      // RouteSummary when traversal is completed.
      rpc RecordRoute(stream Point) returns (RouteSummary) {}
      

- 一个 双向流式 RPC 是双方使用读写流去发送一个消息序列。两个流独立操作，因此客户端和服务器可以以任意喜欢的顺序读写：比如， 服务器可以在写入响应前等待接收所有的客户端消息，或者可以交替的读取和写入消息，或者其他读写的组合。 每个流中的消息顺序被预留。

你可以通过在请求和响应前加 stream 关键字去制定方法的类型


    
      // Accepts a stream of RouteNotes sent while a route is being traversed,
      // while receiving other RouteNotes (e.g. from other users).
      rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
      


#### 3 生成客户端和服务器端代码



    python -m grpc_tools.protoc -I ../../protos --python_out=. --grpc_python_out=. ../../protos/xxxx.proto

    protoc -I=$SRC_DIR --python_out=$DST_DIR $SRC_DIR/xxxx.proto
    
    $ protoc -I ../../protos --python_out=. --grpc_out=. --plugin=protoc-gen-grpc=`which grpc_python_plugin` ../../protos/xxxx.proto




### 完成客户端， 服务端逻辑



#### 1 proto文件中的定义


    
    package routeguide;
    
    // Interface exported by the server.
    service RouteGuide {
     
      rpc GetFeature(Point) returns (Feature) {}
    
     
      rpc ListFeatures(Rectangle) returns (stream Feature) {}
    
     
      rpc RecordRoute(stream Point) returns (RouteSummary) {}
    
      rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
    }
    
  
    message Point {
      int32 latitude = 1;
      int32 longitude = 2;
    }
    
    
    message Rectangle {
      Point lo = 1;
      Point hi = 2;
    }
    
  
    message Feature {
      string name = 1;
      Point location = 2;
    }
    
    message RouteNote {
      Point location = 1;
      string message = 2;
    }
    
 
    message RouteSummary {
      int32 point_count = 1;
      int32 feature_count = 2;
      int32 distance = 3;
      int32 elapsed_time = 4;
    }
    
    


#### 2 client 代码实现


```python
from __future__ import print_function

import random
import logging

import grpc

import route_guide_pb2
#request 和 response 数据类型
import route_guide_pb2_grpc
# service
import route_guide_resources
# 读取自定义的数据


def make_route_note(message, latitude, longitude):
    return route_guide_pb2.RouteNote(
        message=message,
        location=route_guide_pb2.Point(latitude=latitude, longitude=longitude))


def guide_get_one_feature(stub, point):
    feature = stub.GetFeature(point)
    if not feature.location:
        print("Server returned incomplete feature")
        return

    if feature.name:
        print("Feature called %s at %s" % (feature.name, feature.location))
    else:
        print("Found no feature at %s" % feature.location)


def guide_get_feature(stub):
    guide_get_one_feature(stub, route_guide_pb2.Point(latitude=409146138, longitude=-746188906))
    guide_get_one_feature(stub, route_guide_pb2.Point(latitude=0, longitude=0))


def guide_list_features(stub):
    rectangle = route_guide_pb2.Rectangle(
        lo=route_guide_pb2.Point(latitude=400000000, longitude=-750000000),
        hi=route_guide_pb2.Point(latitude=420000000, longitude=-730000000))
    print("Looking for features between 40, -75 and 42, -73")

    features = stub.ListFeatures(rectangle)

    for feature in features:
        print("Feature called %s at %s" % (feature.name, feature.location))


def generate_route(feature_list):
    for _ in range(0, 10):
        random_feature = feature_list[random.randint(0, len(feature_list) - 1)]
        print("Visiting point %s" % random_feature.location)
        yield random_feature.location


def guide_record_route(stub):
    feature_list = route_guide_resources.read_route_guide_database()

    route_iterator = generate_route(feature_list)
    route_summary = stub.RecordRoute(route_iterator)
    print("Finished trip with %s points " % route_summary.point_count)
    print("Passed %s features " % route_summary.feature_count)
    print("Travelled %s meters " % route_summary.distance)
    print("It took %s seconds " % route_summary.elapsed_time)


def generate_messages():
    messages = [
        make_route_note("First message", 0, 0),
        make_route_note("Second message", 0, 1),
        make_route_note("Third message", 1, 0),
        make_route_note("Fourth message", 0, 0),
        make_route_note("Fifth message", 1, 0),
    ]
    for msg in messages:
        print("Sending %s at %s" % (msg.message, msg.location))
        yield msg


def guide_route_chat(stub):
    responses = stub.RouteChat(generate_messages())
    for response in responses:
        print("Received message %s at %s" % (response.message,
                                             response.location))


def run():
    # NOTE(gRPC Python Team): .close() is possible on a channel and should be
    # used in circumstances in which the with statement does not fit the needs
    # of the code.
    with grpc.insecure_channel('localhost:50051') as channel:
        stub = route_guide_pb2_grpc.RouteGuideStub(channel)
        # 初始化 service
        print("-------------- GetFeature --------------")
        guide_get_feature(stub)
        print("-------------- ListFeatures --------------")
        guide_list_features(stub)
        print("-------------- RecordRoute --------------")
        guide_record_route(stub)
        print("-------------- RouteChat --------------")
        guide_route_chat(stub)


if __name__ == '__main__':
    logging.basicConfig()
    run()



```


#### 3 服务端代码实现



```python

# Copyright 2015 gRPC authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
"""The Python implementation of the gRPC route guide server."""

from concurrent import futures
import time
import math
import logging

import grpc

import route_guide_pb2
import route_guide_pb2_grpc
import route_guide_resources


def get_feature(feature_db, point):
    """Returns Feature at given location or None."""
    for feature in feature_db:
        if feature.location == point:
            return feature
    return None


def get_distance(start, end):
    """Distance between two points."""
    coord_factor = 10000000.0
    lat_1 = start.latitude / coord_factor
    lat_2 = end.latitude / coord_factor
    lon_1 = start.longitude / coord_factor
    lon_2 = end.longitude / coord_factor
    lat_rad_1 = math.radians(lat_1)
    lat_rad_2 = math.radians(lat_2)
    delta_lat_rad = math.radians(lat_2 - lat_1)
    delta_lon_rad = math.radians(lon_2 - lon_1)

    # Formula is based on http://mathforum.org/library/drmath/view/51879.html
    a = (pow(math.sin(delta_lat_rad / 2), 2) +
         (math.cos(lat_rad_1) * math.cos(lat_rad_2) * pow(
             math.sin(delta_lon_rad / 2), 2)))
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
    R = 6371000
    # metres
    return R * c


class RouteGuideServicer(route_guide_pb2_grpc.RouteGuideServicer):
    """Provides methods that implement functionality of route guide server."""

    def __init__(self):
        self.db = route_guide_resources.read_route_guide_database()

    def GetFeature(self, request, context):
        feature = get_feature(self.db, request)
        if feature is None:
            return route_guide_pb2.Feature(name="", location=request)
        else:
            return feature

    def ListFeatures(self, request, context):
        left = min(request.lo.longitude, request.hi.longitude)
        right = max(request.lo.longitude, request.hi.longitude)
        top = max(request.lo.latitude, request.hi.latitude)
        bottom = min(request.lo.latitude, request.hi.latitude)
        for feature in self.db:
            if (feature.location.longitude >= left and
                    feature.location.longitude <= right and
                    feature.location.latitude >= bottom and
                    feature.location.latitude <= top):
                yield feature

    def RecordRoute(self, request_iterator, context):
        point_count = 0
        feature_count = 0
        distance = 0.0
        prev_point = None

        start_time = time.time()
        for point in request_iterator:
            point_count += 1
            if get_feature(self.db, point):
                feature_count += 1
            if prev_point:
                distance += get_distance(prev_point, point)
            prev_point = point

        elapsed_time = time.time() - start_time
        return route_guide_pb2.RouteSummary(
            point_count=point_count,
            feature_count=feature_count,
            distance=int(distance),
            elapsed_time=int(elapsed_time))

    def RouteChat(self, request_iterator, context):
        prev_notes = []
        for new_note in request_iterator:
            for prev_note in prev_notes:
                if prev_note.location == new_note.location:
                    yield prev_note
            prev_notes.append(new_note)


def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    route_guide_pb2_grpc.add_RouteGuideServicer_to_server(
        RouteGuideServicer(), server)
    server.add_insecure_port('[::]:50051')
    server.start()
    server.wait_for_termination()


if __name__ == '__main__':
    logging.basicConfig()
    serve()


```



