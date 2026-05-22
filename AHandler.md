>AHandler 是 Android Native 层实现的一个异步消息处理机制，主要用于处理跨线程的消息传递。它的核心组件包括 AHandler、AMessage 和 ALooper。

# 主要组件
**AHandler**：消息处理类，负责接收和处理消息。需要继承此类并重写 onMessageReceived 方法来处理消息。

**AMessage**：消息类，用于封装要传递的数据。通过 post 方法将消息发送到 ALooper。

**ALooper**：消息轮询类，负责从消息队列中取出消息并分发给对应的 AHandler 处理。

# 工作原理

**创建ALooper**：首先创建一个 ALooper 实例，它会在后台启动一个线程来处理消息队列。

**注册AHandler**：将自定义的 AHandler 注册到 ALooper 中。

**发送消息**：通过 AMessage 创建消息，并使用 post 方法将消息发送到 ALooper。

**消息处理**：ALooper 从消息队列中取出消息，并调用对应 AHandler 的 onMessageReceived 方法进行处理。


# 示例
```
class MyHandler : public AHandler {
public:
    enum {
        kWhatExample = 1,
    };

    virtual void onMessageReceived(const sp<AMessage> &msg) {
        switch (msg->what()) {
            case kWhatExample:
                // 处理消息
                break;
            default:
                break;
        }
    }
};

int main() {
    sp<ALooper> looper = new ALooper();
    looper->setName("ExampleLooper");
    looper->start();

    sp<MyHandler> handler = new MyHandler();
    looper->registerHandler(handler);

    sp<AMessage> msg = new AMessage(MyHandler::kWhatExample, handler);
    msg->post();

    // 停止 looper
    looper->stop();
    return 0;
}

```

# 需要回复的消息

AHandler 还支持需要回复的消息机制。发送方可以等待消息处理完毕后接收回复消息：

```
sp<AMessage> msg = new AMessage(MyHandler::kWhatExample, handler);
sp<AMessage> response;
msg->postAndAwaitResponse(&response);
```

在 onMessageReceived 方法中处理完消息后，可以通过 postReply 方法发送回复消息：

```
void MyHandler::onMessageReceived(const sp<AMessage> &msg) {
    sp<AReplyToken> replyID;
    if (msg->senderAwaitsResponse(&replyID)) {
        sp<AMessage> response = new AMessage();
        response->setString("reply", "Received");
        response->postReply(replyID);
    }
}


```
