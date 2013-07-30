---
layout: post
title: "nodejs mqtt库部分源码分析"
date: 2013-07-25
---
先来看看MqttServer这个类的实现吧

    /**
     * MqttServer
     *
     * @param {Function} listener - fired on client connection
     */
    var MqttServer = module.exports.MqttServer =
    function Server(listener) {
      if (!(this instanceof Server)) return new Server(listener);
    
      var self = this;

      net.Server.call(self);

      if (listener) {
        self.on('client', listener);
      }

      self.on('connection', function(socket) {
      self.emit('client', new MqttServerClient(socket, self));
    });

      return this;
    }



net.Server.call(self); 这一行调用是为了获得nodejs核心包net.Server的属性和方法，这是javascript的一种对象继承模式。

可以看出，创建MqttServer的时候（调用createServer()方法），新创建的server会自动注册'connection'事件，一旦事件被触发，server自己就触发'client'事件，并传递一个新创建的MqttServerClient对象，这和我们平时使用的情况一致。
