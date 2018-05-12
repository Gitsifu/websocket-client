<template>
  <div>
    Welcome<br/><input id="text" type="text"/>
    <button @click="send()">发送消息</button>
    <button @click="subscribe2()">订阅消息/topic/sendTest</button>
    <button @click="subscribe1()">订阅消息/topic/subscribeTest</button>
    <hr/>
    <button @click="closeWebSocket()">关闭WebSocket连接</button>
    <button @click="toConnect()">重新WebSocket连接</button>
    <hr/>
    <ul>
      <li
        is="todo-item"
        v-for="(todo, index) in todos"
        v-bind:key="todo.id"
        v-bind:title="todo.title"
        v-on:remove="todos.splice(index, 1)"
      ></li>
    </ul>
  </div>
</template>

<script>
  import SockJS from 'sockjs-client'
  import Stomp from 'stompjs'
  // let Stomp = require('stompjs');
  export default {
    name: 'HelloWorld',
    data() {
      return {
        //newTodoText: '',
        todos: [],
        nextTodoId: 1,
        stompClient: ''
      }
    },
    methods: {
      addNewTodo: function (newTodoText) {
        this.todos.push({
          id: this.nextTodoId++,
          title: newTodoText
        })
      },
      //将消息显示在网页上
      setMessageInnerHTML(innerHTML) {
        this.addNewTodo(innerHTML)
        //this.msg = this.msg + innerHTML;
        // document.getElementById('message').innerHTML += innerHTML + '<br/>';
      },

      //发送消息
      send() {
        let message = document.getElementById('text').value;
        let messageJson = JSON.stringify({"name": message});
        this.stompClient.send("/app/sendTest", {}, messageJson);
        this.setMessageInnerHTML("/app/sendTest 你发送的消息:" + message);
      },

      //订阅消息
      subscribe1() {

        let that = this
        this.stompClient.subscribe('/topic/subscribeTest', function (response) {
          that.setMessageInnerHTML("已成功订阅/topic/subscribeTest");
          let returnData = JSON.parse(response.body);
          that.setMessageInnerHTML("/topic/subscribeTest 你接收到的消息为:" + returnData.responseMessage);
        });
      },

      //订阅消息
      subscribe2() {
        let that = this;
        this.stompClient.subscribe('/topic/sendTest', function (response) {
          that.setMessageInnerHTML("已成功订阅/topic/sendTest");
          let returnData = JSON.parse(response.body);
          that.setMessageInnerHTML("/topic/sendTest 你接收到的消息为:" + returnData.responseMessage);
        });
      },


      //关闭WebSocket连接
      closeWebSocket() {
        this.stompClient.disconnect(function () {
          alert("See you next time!");
        });
      },

      toConnect() {
        // 建立连接对象（还未发起连接）
        let socket = new SockJS("http://localhost:8080/webSocketServer");

        // 获取 STOMP 子协议的客户端对象
        let stompClient = Stomp.over(socket);
        let that = this;
        // 向服务器发起websocket连接并发送CONNECT帧
        stompClient.connect(
          {},
          function connectCallback(frame) {
            // 连接成功时（服务器响应 CONNECTED 帧）的回调方法
            that.setMessageInnerHTML("连接成功");

            stompClient.subscribe('/app/subscribeTest', function (response) {
              that.setMessageInnerHTML("已成功订阅/app/subscribeTest");
              let returnData = JSON.parse(response.body);
              that.setMessageInnerHTML("/app/subscribeTest 你接收到的消息为:" + returnData.responseMessage);
            });
          },
          function errorCallBack(error) {
            // 连接失败时（服务器响应 ERROR 帧）的回调方法
            this.setMessageInnerHTML("连接失败");
          }
        );
        this.stompClient = stompClient
      }
    },

    mounted() {
      this.toConnect()
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
