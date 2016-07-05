[toc]

## （未）6. 通讯API

### （未）6.1 跨文档消息

之前，出于完全原因，Frame、标签和窗口之间的通讯是完全禁止的。Cross Document Messaging使得它们之间可以相互通信。发消息通过`postMessage`方法。

	chatFrame.contentWindow.postMessage('Hello, world', 'http://www.example.com/');

接受消息通过监听`message`事件：

    window.addEventListener(“message”, messageHandler, true);
    function messageHandler(e) {
     switch(e.origin) {
     case “friend.example.com”:
     // process message
     processMessage(e.data);
     break;
     default:
     // message origin not recognized
     // ignoring message
     }
    }

消息事件有两个属性：`data`和`origin`。The data property is the actual message that the sender passed along and the origin property is the sender’s origin. 利用`origin`可以判定消息发送方是否是可信的。















