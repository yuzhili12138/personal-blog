
```
// websocket心跳
class webSocketClass {
    lockReconnect=false
    localUrl
    globalCallback=null
    userClose=false
    webSocketState=false
    thatVue
    ws

    constructor(thatVue) {
      this.lockReconnect = false;
      // vue
      this.localUrl = process.env.NODE_ENV === 'production' ? 你的websocket生产地址' : '测试地址';
      this.globalCallback = null;
      this.userClose = false;
      this.createWebSocket();
      this.webSocketState = false
      this.thatVue = thatVue
    }
  
    createWebSocket() {
      let that = this;
      // console.log('开始创建websocket新的实例', new Date().toLocaleString())
      if( typeof(WebSocket) != "function" ) {
        alert("您的浏览器不支持Websocket通信协议，请更换浏览器为Chrome或者Firefox再次使用！")
      }
      try {
        that.ws = new WebSocket(that.localUrl);
        that.initEventHandle();
        that.startHeartBeat()
      } catch (e) {
        that.reconnect();
      }
    }

    //初始化
    initEventHandle() {
      let that = this;
      // //连接成功建立后响应
      that.ws.onopen = function() {
        console.log("连接成功");
      }; 
      //连接关闭后响应
      that.ws.onclose = function() {
        // console.log('websocket连接断开', new Date().toLocaleString())
        if (!that.userClose) {
          that.reconnect(); //重连
        }
      };
      that.ws.onerror = function() {
        // console.log('websocket连接发生错误', new Date().toLocaleString())
        if (!that.userClose) {
          that.reconnect(); //重连
        }
      };
      that.ws.onmessage = function(event) {
        that.getWebSocketMsg(that.globalCallback);
        // console.log('socket server return '+ event.data);
      };
    }
    startHeartBeat () {
      // console.log('心跳开始建立', new Date().toLocaleString())
      setTimeout(() => {
          let params = {
            request: 'ping',
          }
          this.webSocketSendMsg(JSON.stringify(params))
          this.waitingServer()
      }, 30000)
    }
    //延时等待服务端响应，通过webSocketState判断是否连线成功
    waitingServer () {
      this.webSocketState = false//在线状态
      setTimeout(() => {
          if(this.webSocketState) {
              this.startHeartBeat()
              return
          }
          // console.log('心跳无响应，已断线', new Date().toLocaleString())
          try {
            this.closeSocket()
          } catch(e) {
            console.log('连接已关闭，无需关闭', new Date().toLocaleString())
          }
          this.reconnect()
          //重连操作
      }, 5000)
    }
    reconnect() {
      let that = this;
      if (that.lockReconnect) return;
      that.lockReconnect = true; //没连接上会一直重连，设置延迟避免请求过多
      setTimeout(function() {
        that.createWebSocket();
        that.thatVue.openSuccess(that) //重连之后做一些事情
        that.thatVue.getSocketMsg(that)
        that.lockReconnect = false;
      }, 15000);
    }
  
    webSocketSendMsg(msg) {
      this.ws.send(msg);
    }
  
    getWebSocketMsg(callback) {
      this.ws.onmessage = ev => {
        callback && callback(ev);
      };
    }
    onopenSuccess(callback) {
      this.ws.onopen = () => {
        // console.log("连接成功", new Date().toLocaleString())
        callback && callback()
      }
    }
    closeSocket() {
      let that = this;
      if (that.ws) {
        that.userClose = true;
        that.ws.close();
      }
    }
  }
  export default webSocketClass;
```


