#### Android Netty 实现“智能”聊天

Netty，JBoss提供的一个高性能、异步事件驱动的NIO框架，在Java后端很常用，可以更好的使用Socket通信，微服务模块通信等。由于我是做Android开发，更多所以本篇就用来做一个简单的IM文字聊天。

![Android Netty 实现“智能”聊天.jpeg](https://upload-images.jianshu.io/upload_images/1641428-526355e1bc5c9ae5.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

![估值1个亿的AI代码.jpeg](https://upload-images.jianshu.io/upload_images/1641428-99f0348fc9364d3d.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

IM通信通用3步走：

1. 设置连接地址，设置连接回调、设置消息接收监听。
2. 开始连接，连接成功、失败处理。
3. 收到消息，解析消息，做其他UI处理等。

代码分服务端和客户端，下面我贴出服务端的代码，我找了个SpringBoot结合Netty的demo改了一下。

#### 服务端

[后端代码](https://github.com/archine/springboot-netty)

改动：NettyServerHandler类的channelRead()方法中的返回信息改成如下：

```
@Slf4j
public class NettyServerHandler extends ChannelInboundHandlerAdapter {
    /**
     * 客户端连接会触发
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        log.info("Channel active......");
    }

    /**
     * 客户端发消息会触发
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //----------- 只改了这里 -----------
        log.info("服务器收到消息: {}", msg.toString());
        String result;
        result = String.valueOf(msg)
                .replace("吗", "")
                .replace("?", "!")
                .replace("？", "！");
        ctx.write("人工智能：" + result);
        ctx.flush();
        //----------- 只改了这里 -----------
    }

    /**
     * 发生异常触发
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

#### 客户端

1. 添加Netty依赖

```
dependencies {
    //...
    
    //Netty
    implementation 'io.netty:netty-all:4.1.9.Final'
}
```

我主要对Android端的Netty进行一些封装，主要有以下类：

2. NettyClient，Netty客户端，内部封装连接、发送消息的API。

```
class NettyClient private constructor() {
    /**
     * Socket通道
     */
    private var mSocketChannel: SocketChannel? = null
    /**
     * 消息监听
     */
    private val mMsgReceiveMsgCallbacks by lazy {
        CopyOnWriteArrayList<ReceiveMsgCallback>()
    }
    /**
     * 是否已经连接
     */
    private var isConnect: Boolean = false
    /**
     * 主线程Handler
     */
    private val mMainHandler by lazy {
        Handler(Looper.getMainLooper())
    }

    companion object {
        private class SingleHolder {
            companion object {
                @JvmStatic
                val INSTANCE = NettyClient()
            }
        }

        @JvmStatic
        fun getInstance(): NettyClient {
            return SingleHolder.INSTANCE
        }
    }

    /**
     * 开始连接
     * @param host 地址
     * @param port 端口号
     */
    fun connect(host: String, port: Int, callback: Callback) {
        val group = NioEventLoopGroup()
        Bootstrap()
            .group(group)
            .option(ChannelOption.TCP_NODELAY, true)
            .channel(NioSocketChannel::class.java)
            .handler(object : ChannelInitializer<SocketChannel>() {
                override fun initChannel(socketChannel: SocketChannel?) {
                    socketChannel?.pipeline()?.apply {
                        addLast("decoder", StringDecoder(CharsetUtil.UTF_8))
                        addLast("encoder", StringEncoder(CharsetUtil.UTF_8))
                        addLast(NettyClientHandler(object : Callback {
                            override fun onSuccess() {
                                isConnect = true
                            }

                            override fun onFail() {
                                isConnect = false
                                close()
                            }
                        }, object : ReceiveMsgCallback {
                            override fun onReceiveMsg(msg: String) {
                                mMsgReceiveMsgCallbacks.forEach {
                                    it.onReceiveMsg(msg)
                                }
                            }
                        }))
                    }
                }
            })
            .connect(InetSocketAddress(host, port))
            .addListener(ChannelFutureListener { future: ChannelFuture ->
                if (future.isSuccess) {
                    isConnect = true
                    mSocketChannel = future.channel() as SocketChannel
                    mMainHandler.post {
                        callback.onSuccess()
                    }
                } else {
                    isConnect = false
                    close()
                    //这里一定要关闭，不然一直重试会引发OOM
                    future.channel().close()
                    group.shutdownGracefully()
                    mMainHandler.post {
                        callback.onFail()
                    }
                }
            })
    }

    /**
     * 断开连接
     */
    fun disconnect() {
        close()
    }

    /**
     * 发送消息
     */
    fun sendMsg(msg: String, callback: Callback) {
        if (!isConnected()) {
            callback.onFail()
            return
        }
        mSocketChannel?.run {
            writeAndFlush(msg).addListener { future ->
                mMainHandler.post {
                    if (future.isSuccess) {
                        callback.onSuccess()
                    } else {
                        callback.onFail()
                    }
                }
            }
        }
    }

    /**
     * 是否已连接
     */
    fun isConnected(): Boolean {
        return isConnect
    }

    /**
     * 注册消息回调
     */
    fun registerReceiveMsgCallback(receiveMsgCallback: ReceiveMsgCallback) {
        if (!mMsgReceiveMsgCallbacks.contains(receiveMsgCallback)) {
            mMsgReceiveMsgCallbacks.add(receiveMsgCallback)
        }
    }

    /**
     * 注销消息回调
     */
    fun unregisterReceiveMsgCallback(receiveMsgCallback: ReceiveMsgCallback) {
        mMsgReceiveMsgCallbacks.remove(receiveMsgCallback)
    }

    /**
     * 关闭连接
     */
    private fun close() {
        mSocketChannel?.close()
    }

    /**
     * 消息回调
     */
    inner class NettyClientHandler(
        private val activeCallback: Callback,
        private val receiveMsgCallback: ReceiveMsgCallback
    ) :
        SimpleChannelInboundHandler<String>() {
        override fun channelActive(ctx: ChannelHandlerContext?) {
            super.channelActive(ctx)
            //连接成功
            mMainHandler.post {
                activeCallback.onSuccess()
            }
        }

        override fun channelInactive(ctx: ChannelHandlerContext?) {
            super.channelInactive(ctx)
            //失去连接
            mMainHandler.post {
                activeCallback.onFail()
            }
        }

        override fun userEventTriggered(ctx: ChannelHandlerContext?, evt: Any?) {
            super.userEventTriggered(ctx, evt)
            if (evt is IdleStateEvent) {
                if (evt.state() == IdleState.WRITER_IDLE) {
                    //空闲了，发送心跳
                    //ctx!!.writeAndFlush(message.toJson())
                }
            }
        }

        override fun channelRead0(ctx: ChannelHandlerContext?, msg: String?) {
            //接收到消息
            mMainHandler.post {
                msg?.let { message ->
                    receiveMsgCallback.onReceiveMsg(message)
                    ReferenceCountUtil.release(message)
                }
            }
        }

        override fun exceptionCaught(ctx: ChannelHandlerContext?, cause: Throwable?) {
            super.exceptionCaught(ctx, cause)
            ctx?.close()
        }
    }
}
```

3. Callback，操作回调，告知操作成功还是失败。

```
interface Callback {
    /**
     * 成功
     */
    fun onSuccess()

    /**
     * 失败
     */
    fun onFail()
}
```

4. ReceiveMsgCallback，收到消息回调，消息类型为String。

```
interface ReceiveMsgCallback {
    /**
     * 接收到消息回调
     */
    fun onReceiveMsg(msg: String)
}
```

#### 界面布局

布局很简单，一个文本显示连接状态，一个输入框，一个发送按钮，一个文本显示以往消息。

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/connect_status"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="13dp"
        android:text="连接状态：未连接"
        android:textColor="@android:color/black"
        android:textSize="17sp" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="13dp">

        <EditText
            android:id="@+id/msg_input"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginEnd="13dp"
            android:layout_weight="1"
            android:hint="聊聊吧~"
            android:textColor="@android:color/black"
            android:textColorHint="@android:color/darker_gray"
            android:textSize="15sp" />

        <Button
            android:id="@+id/send"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="发送" />
    </LinearLayout>

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:fillViewport="true">

        <TextView
            android:id="@+id/chat_content"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:lineSpacingMultiplier="1.4"
            android:padding="13dp"
            android:textColor="@android:color/black"
            android:textSize="16sp"
            tools:text="客户端：你好\n服务端：你好呀" />
    </ScrollView>
</LinearLayout>
```

#### Java代码

界面代码很简单， 处理输入框、发送按钮以及收到消息后显示以往消息。

```
class MainActivity : AppCompatActivity(), ReceiveMsgCallback {
    private lateinit var vConnectStatus: TextView
    private lateinit var vMsgInput: EditText
    private lateinit var vSend: Button
    private lateinit var vChatContent: TextView

    companion object {
        private const val TAG = "MainActivity"
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val layout: View = findViewById(android.R.id.content)
        findView(layout)
        bindView()
        setData()
    }

    override fun onDestroy() {
        super.onDestroy()
        NettyClient.getInstance().apply {
            disconnect()
            unregisterReceiveMsgCallback(this@MainActivity)
        }
    }

    private fun findView(view: View) {
        vConnectStatus = view.findViewById(R.id.connect_status)
        vMsgInput = view.findViewById(R.id.msg_input)
        vSend = view.findViewById(R.id.send)
        vChatContent = view.findViewById(R.id.chat_content)
    }

    private fun bindView() {
        vSend.setOnClickListener {
            val inputText = vMsgInput.text.toString().trim()
            if (inputText.isBlank()) {
                Toast.makeText(this@MainActivity, "请输入你要发送的消息", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }
            //发送消息
            NettyClient.getInstance().sendMsg(inputText, object : Callback {
                override fun onSuccess() {
                    Log.d(TAG, "发送成功，内容：$inputText")
                    appendClientMsg("我：$inputText")
                    vMsgInput.setText("")
                }

                override fun onFail() {
                    Log.d(TAG, "发送失败，内容：$inputText")
                    Toast.makeText(this@MainActivity, "发送失败，请重试", Toast.LENGTH_SHORT).show()
                }
            })
        }
    }

    private fun setData() {
        //连接
        NettyClient.getInstance().connect("192.168.101.244", 8090, object : Callback {
            override fun onSuccess() {
                vConnectStatus.text = "连接状态：已连接"
                Toast.makeText(this@MainActivity, "连接成功", Toast.LENGTH_SHORT).show()
            }

            override fun onFail() {
                vConnectStatus.text = "连接状态：连接失败"
                Toast.makeText(this@MainActivity, "连接失败", Toast.LENGTH_SHORT).show()
            }
        })
        NettyClient.getInstance().registerReceiveMsgCallback(this)
    }

    override fun onReceiveMsg(msg: String) {
        //收到消息，拼接以往信息
        appendClientMsg(msg)
    }

    /**
     * 添加客户端消息
     */
    @SuppressLint("SetTextI18n")
    private fun appendClientMsg(msg: String) {
        val beforeText = vChatContent.text
        vChatContent.text = "$beforeText\n $msg"
    }
}
```

#### Android项目Github地址

[Github地址](https://github.com/hezihaog/SlideLockScreen)