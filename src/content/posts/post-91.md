---
title: "NestJS通过第三方库socket.io进行WebSocket连接"
published: 2024-09-04
updated: 2024-09-20
description: ""
tags: ["Vue3","NestJs"]
category: "后端相关"
draft: false
---

**安装所需包**

    npm install @nestjs/platform-socket.io @nestjs/websockets socket.io

**新建一个chat.gateway.ts文件**
    //chat.gateway.ts
    import {
      WebSocketGateway,
      WebSocketServer,
      SubscribeMessage,
      MessageBody,
      ConnectedSocket,
      OnGatewayConnection,
      OnGatewayDisconnect,
    } from \'@nestjs/websockets\';
    import { Server, Socket } from \'socket.io\';
    
    @WebSocketGateway(3001, {
      namespace: \'chat\',
      // 指定端口和命名空间
      cors: {
        origin: \'*\',
      },
      //配置跨域
    })
    //可以通过socket.io连接http://localhost:3001/chat
    export class ChatGateway implements OnGatewayConnection, OnGatewayDisconnect {
      @WebSocketServer()
      server: Server;
    
      handleConnection(client: Socket): void {
        console.log(\'Client connected:\', client.id);
      }
    
      handleDisconnect(client: Socket): void {
        console.log(\'Client disconnected:\', client.id);
      }
    
      @SubscribeMessage(\'message\')
      handleMessage(
        @MessageBody() message: string,
        @ConnectedSocket() client: Socket,
      ): void {
        // 处理特定消息事件
        console.log(`Received message: ${message} from client ${client.id}`);
        this.server.emit(\'message\', message); // 广播消息给所有连接的客户端
      }
    }
    //app.module.ts中引入并注册
    import { ChatGateway } from \'./utils/chat.gateway\';
    providers: [AppService, ChatGateway],

**通过postman调试，先连接再发消息**
![2024-09-04T02:32:59.png][1]
![2024-09-04T02:37:05.png][2]
![2024-09-04T02:37:16.png][3]
**在vue3中连接**
    //先安装socket.io客户端
    npm i socket.io-client
    //页面使用
    <template>
      <div id=\"app\">
         <h1>WebSocket Test</h1>
         <input v-model=\"message\" placeholder=\"Enter message\" />
         <button @click=\"sendMessage\">Send Message</button>
         <ul>
           <li v-for=\"(msg, index) in messages\" :key=\"index\">{{ msg }}</li>
         </ul>
      </div>
     </template>
    
     <script setup>
     import { ref, onMounted } from \'vue\';
     import { io } from \'socket.io-client\';
         const socket = ref(null);
         const message = ref(\'\');
         const messages = ref([]);
    
         const sendMessage = () => {
           if (message.value) {
             socket.value.emit(\'message\', message.value);
             message.value = \'\';
           }
         };
         onMounted(() => {
           socket.value = io(\'http://localhost:3001/chat\');
           socket.value.on(\'connect\', () => {
             console.log(\'Connected to WebSocket server\');
           });
           socket.value.on(\'message\', (msg) => {
             messages.value.push(msg);
           });
           socket.value.on(\'disconnect\', () => {
             console.log(\'Disconnected from WebSocket server\');
           });
        });
     </script>
    
     <style>
     #app {
      font-family: Avenir, Helvetica, Arial, sans-serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
      text-align: center;
      color: #2c3e50;
      margin-top: 60px;
     }
     </style>
**在线体验**
https://nav.heiyu.fun/#/test


  [1]: https://blog.heiyu.fun/usr/uploads/2024/09/291407181.png
  [2]: https://blog.heiyu.fun/usr/uploads/2024/09/1813496167.png
  [3]: https://blog.heiyu.fun/usr/uploads/2024/09/3539525372.png
