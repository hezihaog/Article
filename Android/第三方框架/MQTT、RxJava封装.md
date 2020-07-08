#### MQTT、RxJava封装

最近物联网的项目需要做推送，功能实现方式有以下几种：

- 轮训，我们客户端定时请求后端接口，并不是推送，缺点是耗电、耗流量。

- 发送短信并拦截，服务端向客户端发送短信，客户端进行拦截。拦截短信需要申请权限，并且容易被安全管家类的App拦截，并且也需要耗费费用来发送短信，不可靠并且也耗财力。

- 长连接，客户端和服务端维持一个连接（定时心跳），真正的推送。
	1. GCM，国内环境，不考虑。
	2. XMPP，XML格式，协议比较复杂，比较费流量，费电。
	3. MQTT，协议简洁，省流量、省点。
	4. 第三方推送，例如友盟推送、百度推送、个推等，优点是容易接入，缺点嘛暂时没发现什么缺点，不过第三方推送对于物联网可能会有个天生不足，就是物联网项目，有些政府的地方使用只能是局域网，而如果是第三方推送，服务端需要调用第三方服务端Api，再推到客户端，在局域网里面就用不了。

#### MQTT简介

- MQTT使用发布、订阅的消息模式，支持一对多方式订阅。
- 只是一种协议，消息内容并不会强制绑定，只是一个字符串，一般我们为json。
- 使用TCP/IP提供网络连接。
- 小型传输，开销很小（固定长度的头部是2字节），协议交换最小化，以降低网络流量。

消息发布有以下质量选择：

1. “至多一次”，消息发布完全依赖底层TCP/IP网络。会发生消息丢失或重复。这一级别可用于如下情况，环境传感器数据，丢失一次读记录无所谓，因为不久后还会有第二次发送。这一种方式主要普通APP的推送，倘若你的智能设备在消息推送时未联网，推送过去没收到，再次联网也就收不到了。

2. “至少一次”，确保消息到达，但消息重复可能会发生。

3. “只有一次”，确保消息到达一次。在一些要求比较严格的计费系统中，可以使用此级别。在计费系统中，消息重复或丢失会导致不正确的结果。这种最高质量的消息发布服务还可以用于即时通讯类的APP的推送，确保用户收到且只会收到一次。　

#### 选用方案

物联网一般也使用MQTT，为了后续其他项目也方便用，需要对MQTT做一次封装，由于RxJava的Retry重试系列操作符对重连可以很简单的封装。消息一对多分发刚好RxJava的多订阅者进行消息，很容易解耦，所以将MQTT、RxJava整合封装。

#### 第三方库

我这里采用**paho.mqtt.android**这个第三方库。

- 项目根目录下的build.gradle中添加：

```
repositories {
    maven {
        url "https://repo.eclipse.org/content/repositories/paho-releases/"
    }
}
```

- app或lib目录下的build.gradle中添加

```
dependencies {
    implementation 'org.eclipse.paho:org.eclipse.paho.android.service:1.1.1'
    implementation 'org.eclipse.paho:org.eclipse.paho.client.mqttv3:1.1.1'
}
```

- AndroidManifest.xml清单文件中
	1. 添加权限
	2. 声明库中的MqttService服务

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.app.mqtt">

    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.WAKE_LOCK"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

    <application>
        <!-- MqttService -->
        <service android:name="org.eclipse.paho.android.service.MqttService" />
    </application>
</manifest>
```

#### 知识点补充

MQTT是基于发布、订阅的消息模式，支持一对多消息发布和订阅多个主题，先来说一下一些必要的知识点：

- 主题，也叫Topic，为一个字符串，只要客户端订阅，服务端则会将这个Topic的消息推送过来，所以一个客户端订阅多个Topic则可以收到多方消息，例如广告模块推送、新闻模块推送等。例如我们用以下规则组合Topic：system/ad，system/news。

- ClientId，客户端的唯一标识，也是一个字符串，必须唯一，否则会互相抢连接，不断断开、不断连接的情况。很容易出现这种情况，例如一个账号再多个手机上的客户端上登录就会出现该情况，所以一般为用户id+一定随机数来保证唯一。

- 关于一对一推送，上面说到的一对多订阅，可以实现群发，对于一对于推送并没有实现，但是实现也很简单，只要保证一个Topic只有一个客户端订阅即可。例如我们用以下规则组合Topic：/user/uesrId。

- 账号、密码，连接服务端的账号和密码，一般服务端会开启账号、密码验证才允许连接，所以代码中必须保证账号、密码正确性。

#### 类结构

- MqttApi：提供给外部的Api接口
- MqttImpl：实现了MqttApi的类，为接口功能的实现。
- MqttProxy：一个实现了MqttApi接口的代理类，它的存在是为了后续调用时做一些参数校验和拦截，还可以对Api做装饰器拓展，暂时它是转调所有Api到MqttImpl实现类。
- MqttMessage：消息实体，内部封装Topic主题和接收到的消息字符串。
- MqttOption：MQTT连接的配置类，采用Builder模式实现参数配置链式调用。
- MqttQos：Mqtt消息质量枚举（对于上面3种质量）
- NetworkUtil：网络连接工具类，用于断线重试时，避免断网了，不断重试调用MQTTClient的连接方法，不断报错，可能会大量耗电，是一种优化。
- MessagePublishStatus：发送消息发送状态模型，用于RxJava订阅发送状态时转发使用的Model实体类。
- ConnectionStatus：连接状态模型，用于RxJava订阅连接状态时转发使用的Model实体类。
- MqttImproperCloseException，自定义异常类，用于RxJava的Retry操作符时，手动抛出异常，让retry操作符生效，重走连接逻辑。

#### 代码时间

MqttImpl的代码较多，我们最后逐个讲解~

- MqttApi：提供给外部的Api接口，有以下定义：

	1. connectServer()：连接服务器
	2. disconnectServer()：断开连接
	3. restartConnectServer()：重启连接
	4. publishMessage()：发布消息
	5. subscribeMessage()：监听消息接收
	6. subscribeConnectionStatus()：订阅连接状态
	7. subscribeMessagePublishStatus()：订阅消息发送状态
	8. subscribeTopic()：动态订阅一个主题
	9. unsubscribeTopic()：动态取消订阅一个主题

```
public interface MqttApi {
    /**
     * 连接服务器
     *
     * @param option 配置
     */
    Observable<Boolean> connectServer(MqttOption option);

    /**
     * 断开连接
     */
    Observable<Boolean> disconnectServer();

    /**
     * 重启连接
     */
    Observable<Boolean> restartConnectServer();

    /**
     * 发布消息
     *
     * @param topic   主题
     * @param message 消息文本
     */
    Observable<Boolean> publishMessage(String topic, String message);

    /**
     * 监听消息接收
     */
    Observable<MqttMessage> subscribeMessage();

    /**
     * 订阅连接状态
     */
    Observable<ConnectionStatus> subscribeConnectionStatus();

    /**
     * 订阅消息发送状态
     */
    Observable<MessagePublishStatus> subscribeMessagePublishStatus();

    /**
     * 动态订阅一个主题
     *
     * @param topic 主题
     */
    Observable<Boolean> subscribeTopic(String... topic);

    /**
     * 动态取消订阅一个主题
     *
     * @param topic 主题
     */
    Observable<Boolean> unsubscribeTopic(String... topic);
}
```

- MqttProxy：一个实现了MqttApi接口的代理类，为一个单例类，它的存在是为了后续调用时做一些参数校验和拦截，还可以对Api做装饰器拓展，暂时它是转调所有Api到MqttImpl实现类。

```
public class MqttProxy implements MqttApi {
    private final MqttApi mImpl;

    private MqttProxy() {
        mImpl = new MqttImpl(ContextProvider.get().getContext());
    }

    private static final class SingleHolder {
        private static final MqttProxy INSTANCE = new MqttProxy();
    }

    public static MqttProxy getInstance() {
        return SingleHolder.INSTANCE;
    }

    @Override
    public Observable<Boolean> connectServer(MqttOption option) {
        return mImpl.connectServer(option);
    }

    @Override
    public Observable<Boolean> disconnectServer() {
        return mImpl.disconnectServer();
    }
	
	//...省略其他方法的转调代码
}
```

- MqttMessage：消息实体，内部封装Topic主题和接收到的消息字符串。有2个字段：

	1. topic，本次消息来自的主题Topic
	2. message，本次消息内容字符串，不限定内容，一般为json

```
public class MqttMessage {
    /**
     * 订阅的主题
     */
    private String topic;
    /**
     * 接收到的消息
     */
    private String message;

    public MqttMessage(String topic, String message) {
        this.topic = topic;
        this.message = message;
    }

    //...省略get、set方法
}
```

- MqttOption：MQTT连接的配置类，采用Builder模式实现参数配置链式调用。有以下参数可配置：

	1. serverUrl：服务连接
	2. username：用户名（账号）
	3. passWord：密码
	4. clientId：客户端唯一Id
	5. topics：需要订阅的主题
	6. retryIntervalTime：重试间隔时间，单位为秒，默认为2秒，不能小于等于0
	7. connectionTimeout：连接超时时间，单位为秒，默认为10秒，不能小于等于0
	8. keepAliveInterval：心跳间隔时间，单位为秒，默认为30秒，不能小于等于0

```
public class MqttOption {
    private String serverUrl;
    private String username;
    private String passWord;
    private String clientId;
    private String[] topics;
    private int retryIntervalTime;
    private int connectionTimeout;
    private int keepAliveInterval;

    private MqttOption(Builder builder) {
        this.serverUrl = builder.serverUrl;
        this.username = builder.username;
        this.passWord = builder.passWord;
        this.clientId = builder.clientId;
        this.topics = builder.topics == null ? new String[0] : builder.topics;
        this.retryIntervalTime = builder.retryIntervalTime == 0 ? 2 : builder.retryIntervalTime;
        this.connectionTimeout = builder.connectionTimeout <= 0 ? 10 : builder.connectionTimeout;
        this.keepAliveInterval = builder.keepAliveInterval <= 0 ? 30 : builder.keepAliveInterval;
    }

    public String getServerUrl() {
        return serverUrl;
    }

    public String getUsername() {
        return username;
    }

    public String getPassWord() {
        return passWord;
    }
    
    //...省略其他参数的get方法

    public static class Builder {
        /**
         * 服务端地址
         */
        private String serverUrl;
        /**
         * 账号
         */
        private String username;
        /**
         * 密码
         */
        private String passWord;
        /**
         * 客户端Id，必须唯一
         */
        private String clientId;
        /**
         * 需要订阅的主题，连接成功后，会自动订阅
         */
        private String[] topics;
        /**
         * 重试间隔时间，单位为秒
         */
        private int retryIntervalTime;
        /**
         * 连接超时时间
         */
        private int connectionTimeout;
        /**
         * 保持活动时间，超过时间没有消息收发将会触发ping消息确认
         */
        private int keepAliveInterval;

        public Builder setServerUrl(String serverUrl) {
            this.serverUrl = serverUrl;
            return this;
        }

        public Builder setUsername(String username) {
            this.username = username;
            return this;
        }

        //...省略其他参数的set方法

        public MqttOption build() {
            return new MqttOption(this);
        }
    }
}
```

- MqttQos：Mqtt消息质量枚举（对于上面3种质量）

	1. MOST_ONCE：值为0，最多一次，有可能重复或丢失
	2. LEAST_ONE：值为1，至少一次，有可能重复
	3. ONLY_ONE：值为2，只有一次，确保消息只到达一次（用于比较严格的计费系统）

```
public enum MqttQos {
    /**
     * 最多一次，有可能重复或丢失
     */
    MOST_ONCE(0),
    /**
     * 至少一次，有可能重复
     */
    LEAST_ONE(1),
    /**
     * 只有一次，确保消息只到达一次（用于比较严格的计费系统）
     */
    ONLY_ONE(2);

    private int mCode;

    MqttQos(int code) {
        mCode = code;
    }

    public int getCode() {
        return mCode;
    }
}
```

- NetworkUtil：网络连接工具类，用于断线重试时，避免断网了，不断重试调用MQTTClient的连接方法，不断报错，可能会大量耗电，是一种优化。

```
public class NetworkUtil {
    private NetworkUtil() {
    }

    /**
     * 当前是否有网络状态
     *
     * @param context  上下文
     * @param needWifi 是否只有连接上wifi才算是连接上网络
     */
    public static boolean hasNetWorkStatus(Context context, boolean needWifi) {
        NetworkInfo info = getActiveNetwork(context);
        if (info == null) {
            return false;
        }
        if (!needWifi) {
            return info.isAvailable();
        } else if (info.getType() == ConnectivityManager.TYPE_WIFI) {
            return info.isAvailable();
        }
        return false;
    }

    /**
     * 获取活动网络连接信息
     *
     * @param context 上下文
     * @return NetworkInfo
     */
    public static NetworkInfo getActiveNetwork(Context context) {
        ConnectivityManager mConnMgr = (ConnectivityManager) context
                .getSystemService(Context.CONNECTIVITY_SERVICE);
        if (mConnMgr == null) {
            return null;
        }
        // 获取活动网络连接信息
        return mConnMgr.getActiveNetworkInfo();
    }
}
```

- MessagePublishStatus：发送消息发送状态模型，用于RxJava订阅发送状态时转发使用的Model实体类。有2个字段：

	1. isComplete：表示是否发送完毕
	2. message：发送的消息内容

```
public class MessagePublishStatus {
    /**
     * 是否发送完毕
     */
    private boolean isComplete;
    /**
     * 发送的消息内容
     */
    private String message;

    public MessagePublishStatus(boolean isComplete, String message) {
        this.isComplete = isComplete;
        this.message = message;
    }
	
	//省略get、set方法
}
```

- ConnectionStatus：连接状态模型，用于RxJava订阅连接状态时转发使用的Model实体类。有3个字段：

	1. isLost：表示是否丢失连接
	2. isRetry：是否正在重新连接
	3. error：丢失连接时的异常

```
public class ConnectionStatus {
    /**
     * 是否丢失连接
     */
    private boolean isLost;
    /**
     * 是否在重新连接
     */
    private boolean isRetry;
    /**
     * 丢失连接时的异常
     */
    private Throwable error;

    public ConnectionStatus(boolean isLost, Throwable error) {
        this.isLost = isLost;
        this.error = error;
    }

    public ConnectionStatus(boolean isRetry) {
        this.isRetry = isRetry;
    }

	//省略get、set方法
}
```

- MqttImproperCloseException，自定义异常类，用于RxJava的Retry操作符时，手动抛出异常，让retry操作符生效，重走连接逻辑。

```
public class MqttImproperCloseException extends Exception {
    private static final long serialVersionUID = -4030414538155742302L;

    MqttImproperCloseException() {
    }

    public MqttImproperCloseException(String message) {
        super(message);
    }
}
```

- MqttImpl：实现了MqttApi的类，为接口功能的实现。

1. 构造方法初始化，保存Context上下文。创建MqttAndroidClient时需要用到。

```
public class MqttImpl implements MqttApi {
    /**
     * 上下文
     */
    private Context mContext;

    MqttImpl(Context context) {
        mContext = context.getApplicationContext();
    }
}
```

2. 开启连接。

将传入的MqttOption配置转换为MQTT库提供的MqttConnectOptions配置对象并配置相关参数。创建一个MqttAndroidClient客户端，注意ClientId需要加上随机数保证多个账号在不同客户端中登录时，不会Id相同导致互抢连接。

因为连接可能会失败，所以加入了RxJava的retry()操作符，在连接失败时，手动Observable.error()抛出一个MqttImproperCloseException自定义异常，让retry操作符生效，再使用Observable.timer()操作符延时配置的重试间隔时间后重走可观察者的订阅方法进行重连。

由于很多情况都是手机网络不好或手动切换网络时，断开了MQTT的连接，所以加入了NetworkUtil工具类，检查当前网络情况，如果网络被关闭，则手动发送onNext(false)，继续重走重试逻辑，避免不断地重试，调用MqttClient的connect()连接方法，不断的连接失败。

```
/**
 * 当前引用的配置
 */
private MqttOption mCurrentApplyOption;
/**
 * Mqtt客户端
 */
private MqttAndroidClient mMqttClient;
/**
 * 是否已连接
 */
private boolean isConnect;

@Override
public Observable<Boolean> connectServer(MqttOption option) {
    mCurrentApplyOption = option;
    return Observable.create(new ObservableOnSubscribe<Boolean>() {
        @Override
        public void subscribe(ObservableEmitter<Boolean> emitter) throws Exception {
            if (mMqttClient == null) {
                mMqttClient = new MqttAndroidClient(
                        mContext,
                        option.getServerUrl(),
                        option.getClientId(),
                        new MemoryPersistence());
            }
            if (isConnect) {
                emitter.onNext(true);
                return;
            }
            //如果没有网络，不开启连接
            if (!NetworkUtil.hasNetWorkStatus(mContext, false)) {
                emitter.onNext(false);
                return;
            }
            //进行链接配置
            MqttConnectOptions connectOptions = new MqttConnectOptions();
            //如果为false(flag=0)，Client断开连接后，Server应该保存Client的订阅信息
            //如果为true(flag=1)，表示Server应该立刻丢弃任何会话状态信息
            connectOptions.setCleanSession(true);
            //设置用户名和密码
            connectOptions.setUserName(option.getUsername());
            connectOptions.setPassword(option.getPassWord().toCharArray());
            //设置连接超时时间
            connectOptions.setConnectionTimeout(option.getConnectionTimeout());
            //设置心跳发送间隔时间，单位秒
            connectOptions.setKeepAliveInterval(option.getKeepAliveInterval());
            //设置遗嘱
            connectOptions.setWill("android-mqtt-offline-topic", "android-mqtt-is_offline".getBytes(), MqttQos.ONLY_ONE.getCode(), false);
            //开始连接
            mMqttClient.connect(connectOptions, null, new IMqttActionListener() {
                @Override
                public void onSuccess(IMqttToken asyncActionToken) {
                    emitter.onNext(true);
                }

                @Override
                public void onFailure(IMqttToken asyncActionToken, Throwable exception) {
                    exception.printStackTrace();
                    emitter.onNext(false);
                }
            });
        }
    }).flatMap(new Function<Boolean, ObservableSource<Boolean>>() {
        @Override
        public ObservableSource<Boolean> apply(Boolean isSuccess) throws Exception {
            //连接成功，
            if (isSuccess) {
                return subscribeTopic(mCurrentApplyOption.getTopics());
            } else {
                //连接失败，重试
                return Observable.error(new MqttImproperCloseException());
            }
        }
    }).doOnNext(new Consumer<Boolean>() {
        @Override
        public void accept(Boolean isSuccess) throws Exception {
            isConnect = isSuccess;
        }
    }).retryWhen(new Function<Observable<Throwable>, ObservableSource<?>>() {
        @Override
        public ObservableSource<?> apply(Observable<Throwable> throwableObservable) throws Exception {
            return throwableObservable.flatMap(new Function<Throwable, ObservableSource<?>>() {
                @Override
                public ObservableSource<?> apply(Throwable throwable) throws Exception {
                    mMqttClient = null;
                    //所有异常都走重试，延时指定秒值进行重试
                    return Observable.timer(mCurrentApplyOption.getRetryIntervalTime(), TimeUnit.SECONDS)
                            .doOnNext(new Consumer<Long>() {
                                @Override
                                public void accept(Long aLong) throws Exception {
                                    if (mConnectionStatusListener != null) {
                                        mConnectionStatusListener.onRetryConnection();
                                    }
                                }
                            });
                }
            });
        }
    });
}
```

3. 断开连接，将当前的链接断开，如果已经断开了，则不执行断开操作。

```
@Override
public Observable<Boolean> disconnectServer() {
    //已经断开连接了
    if (!isConnect) {
        return Observable.just(true);
    }
    return Observable.create(new ObservableOnSubscribe<Boolean>() {
        @Override
        public void subscribe(ObservableEmitter<Boolean> emitter) throws Exception {
            try {
                mMqttClient.disconnect(null, new IMqttActionListener() {
                    @Override
                    public void onSuccess(IMqttToken asyncActionToken) {
                        mMqttClient = null;
                        emitter.onNext(true);
                    }

                    @Override
                    public void onFailure(IMqttToken asyncActionToken, Throwable exception) {
                        exception.printStackTrace();
                        emitter.onNext(false);
                    }
                });
            } catch (MqttException e) {
                e.printStackTrace();
                emitter.onNext(false);
            }
        }
    }).doOnNext(new Consumer<Boolean>() {
        @Override
        public void accept(Boolean isSuccess) throws Exception {
            isConnect = !isSuccess;
        }
    });
}
```

4. 重新连接，就是先断开连接，再调用连接。

```
@Override
public Observable<Boolean> restartConnectServer() {
    //先断开，再重新连接
    return disconnectServer().flatMap(new Function<Boolean, ObservableSource<Boolean>>() {
        @Override
        public ObservableSource<Boolean> apply(Boolean isSuccess) throws Exception {
            if (isSuccess) {
                return connectServer(mCurrentApplyOption);
            }
            return Observable.just(false);
        }
    });
}
```

5. 发布消息，对特定Topic主题，发布消息。消息质量为MqttQos.ONLY_ONE，只有一次，确保消息只到达一次（用于比较严格的计费系统）

```
@Override
public Observable<Boolean> publishMessage(String topic, String message) {
    return Observable.create(new ObservableOnSubscribe<Boolean>() {
        @Override
        public void subscribe(ObservableEmitter<Boolean> emitter) throws Exception {
            //连接未打开
            if (!isConnect) {
                emitter.onNext(false);
            }
            try {
                //发布消息
                mMqttClient.publish(topic, message.getBytes(), MqttQos.ONLY_ONE.getCode(), false);
                emitter.onNext(true);
            } catch (MqttException e) {
                e.printStackTrace();
                emitter.onNext(false);
            }
        }
    });
}
```

6. 订阅消息，订阅回调对象为MqttMessage，MqttMessage中有消息的Topic主题和消息内容。

```
@Override
public Observable<MqttMessage> subscribeMessage() {
    return Observable.create(new ObservableOnSubscribe<MqttMessage>() {
        @Override
        public void subscribe(ObservableEmitter<MqttMessage> emitter) throws Exception {
            mMqttClient.setCallback(new MqttCallback() {
                @Override
                public void connectionLost(Throwable cause) {
                    //与服务器之间的连接失效
                    isConnect = false;
                    if (mConnectionStatusListener != null) {
                        mConnectionStatusListener.onConnectionLost(cause);
                    }
                    emitter.onError(cause);
                }

                @Override
                public void messageArrived(String topic, org.eclipse.paho.client.mqttv3.MqttMessage message) throws Exception {
                    //接收到消息
                    emitter.onNext(new MqttMessage(topic, message.toString()));
                }

                @Override
                public void deliveryComplete(IMqttDeliveryToken token) {
                    //发送完成
                    if (mMessagePublishStatusListener != null) {
                        try {
                            String message = token.getMessage().toString();
                            mMessagePublishStatusListener.onMessagePublishComplete(message);
                        } catch (MqttException e) {
                            e.printStackTrace();
                        }
                    }
                }
            });
        }
    }).retryWhen(new Function<Observable<Throwable>, ObservableSource<?>>() {
        @Override
        public ObservableSource<?> apply(Observable<Throwable> throwableObservable) throws Exception {
            return throwableObservable.flatMap(new Function<Throwable, ObservableSource<?>>() {
                @Override
                public ObservableSource<?> apply(Throwable throwable) throws Exception {
                    //延时重连
                    return Observable.timer(mCurrentApplyOption.getRetryIntervalTime(), TimeUnit.SECONDS).flatMap(new Function<Long, ObservableSource<?>>() {
                        @Override
                        public ObservableSource<?> apply(Long aLong) throws Exception {
                            return connectServer(mCurrentApplyOption);
                        }
                    });
                }
            });
        }
    });
}
```

7. 订阅连接状态，有2种情况，连接丢失和重连时的回调。

```
/**
 * 连接状态监听
 */
private OnConnectionStatusListener mConnectionStatusListener;

/**
 * 连接状态监听
 */
public interface OnConnectionStatusListener {
    /**
     * 连接丢失
     *
     * @param error 异常对象
     */
    void onConnectionLost(Throwable error);

    /**
     * 正在重试连接
     */
    void onRetryConnection();
}

private void setOnConnectionStatusListener(OnConnectionStatusListener connectionStatusListener) {
    mConnectionStatusListener = connectionStatusListener;
}

@Override
public Observable<ConnectionStatus> subscribeConnectionStatus() {
    return Observable.create(new ObservableOnSubscribe<ConnectionStatus>() {
        @Override
        public void subscribe(ObservableEmitter<ConnectionStatus> emitter) throws Exception {
            setOnConnectionStatusListener(new OnConnectionStatusListener() {
                @Override
                public void onConnectionLost(Throwable error) {
                    emitter.onNext(new ConnectionStatus(true, error));
                }

                @Override
                public void onRetryConnection() {
                    emitter.onNext(new ConnectionStatus(true));
                }
            });
        }
    });
}
```

8. 订阅消息发送状态，暂时只有一个发送成功后的状态回调。

```
/**
 * 消息发布状态监听器
 */
private OnMessagePublishStatusListener mMessagePublishStatusListener;

/**
 * 消息发送状态监听
 */
public interface OnMessagePublishStatusListener {
    /**
     * 发送消息完成
     *
     * @param message 原始消息
     */
    void onMessagePublishComplete(String message);
}

private void setOnMessagePublishStatusListener(OnMessagePublishStatusListener messagePublishStatusListener) {
    mMessagePublishStatusListener = messagePublishStatusListener;
}

@Override
public Observable<MessagePublishStatus> subscribeMessagePublishStatus() {
    return Observable.create(new ObservableOnSubscribe<MessagePublishStatus>() {
        @Override
        public void subscribe(ObservableEmitter<MessagePublishStatus> emitter) throws Exception {
            setOnMessagePublishStatusListener((message)
                    -> emitter.onNext(new MessagePublishStatus(true, message)));
        }
    });
}
```

9. 订阅主题，传入需要订阅的多个主题的数组即可。消息质量，我这里固定为MqttQos.ONLY_ONE，就是只有一次，确保消息只到达一次。有需要可以自行修改。

```
@Override
public Observable<Boolean> subscribeTopic(String... topic) {
    return Observable.create(new ObservableOnSubscribe<Boolean>() {
        @Override
        public void subscribe(ObservableEmitter<Boolean> emitter) throws Exception {
            if (topic.length == 0) {
                emitter.onNext(true);
                return;
            }
            try {
                int size = topic.length;
                int[] qosArr = new int[size];
                for (int i = 0; i < size; i++) {
                    qosArr[i] = MqttQos.ONLY_ONE.getCode();
                }
                mMqttClient.subscribe(topic, qosArr);
                emitter.onNext(true);
            } catch (MqttException e) {
                e.printStackTrace();
                emitter.onNext(false);
            }
        }
    });
}
```

10. 取消订阅主题，传入需要取消订阅的Topic数组即可。

```
@Override
public Observable<Boolean> unsubscribeTopic(String... topic) {
    return Observable.create(new ObservableOnSubscribe<Boolean>() {
        @Override
        public void subscribe(ObservableEmitter<Boolean> emitter) throws Exception {
            try {
                mMqttClient.unsubscribe(topic);
                emitter.onNext(true);
            } catch (MqttException e) {
                e.printStackTrace();
                emitter.onNext(false);
            }
        }
    });
}
```

#### 使用

我们新建一个Service，例如通知模块的MqttService，名为NoticeMqttService。在onStartCommand()方法回调时，使用MqttProxy类进行连接。后续则是消息接收订阅、连接状态订阅、发送状态订阅。最后是发送消息。

```
class NoticeMqttService : Service() {
    private val mDisposables: CompositeDisposable = CompositeDisposable()

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        connectMqttService()
        return super.onStartCommand(intent, flags, startId)
    }

    override fun onDestroy() {
        super.onDestroy()
        if (!mDisposables.isDisposed) {
            mDisposables.clear()
        }
    }

    /**
     * 连接服务端Mqtt
     */
    private fun connectMqttService() {
        val userId = "10086"
        mDisposables.apply {
            MqttProxy.getInstance().apply {
                val topic = "/user/${userId}"
                //连接服务
                add(
                    connectServer(
                        MqttOption.Builder()
                            .setServerUrl("tcp://192.168.0.211:1883")
                            .setUsername("admin")
                            .setPassWord("admin")
                            .setClientId(userId)
                            .setTopics(mutableListOf(topic).toTypedArray())
                            .build()
                    ).ioToMain().subscribe({ isConnectSuccess ->
                        Logger.d("Mqtt连接是否成功: $isConnectSuccess")
                    }, { error ->
                        error.printStackTrace()
                        Logger.d("Mqtt连接失败，原因: ${error.message}")
                    })
                )
                //订阅消息接收
                add(
                    subscribeMessage()
                        .ioToMain()
                        .subscribe({ mqttMessage ->
                            val msg = "Mqtt收到消息: ${mqttMessage.message}"
                            //一般会在这里解析Json，并进行数据、UI处理
                            Logger.d(msg)
                        }, { error ->
                            error.printStackTrace()
                            Logger.d("Mqtt收到消息，但是抛出异常，原因: ${error.message}")
                        })
                )
                //订阅连接状态
                add(
                    subscribeConnectionStatus()
                        .ioToMain()
                        .subscribe({ status ->
                            if (status.isLost) {
                                Logger.d("Mqtt连接丢失，原因: ${status.error.message}")
                            } else if (status.isRetry) {
                                Logger.d("Mqtt重试连接中...")
                            }
                        }, { error ->
                            error.printStackTrace()
                        })
                )
                //订阅消息发送状态
                add(
                    subscribeMessagePublishStatus()
                        .ioToMain()
                        .subscribe({ status ->
                            if (status.isComplete) {
                                Logger.d("Mqtt消息发送完毕，消息内容: ${status.message}")
                            }
                        }, { error ->
                            error.printStackTrace()
                        })
                )
                //发送消息
                val msg = "我是测试消息"
                add(
	                publishMessage(topic, msg)
	                    .subscribe({ result ->
	                        Logger.d("Mqtt发送消息是否成功: $result")
	                    }, { error ->
	                        error.printStackTrace()
	                        Logger.d("Mqtt发送消息失败，消息Topic${topic}, 消息内容: ${msg}")
	                    })
                )
            }
        }
    }

    override fun onBind(intent: Intent?): IBinder? {
        return Binder()
    }
}
```

最后在清单文件中，配置Service

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.app.notice">

    <application>
        <!-- 通知模块的Mqtt消息接收服务 -->
        <service android:name=".service.NoticeMqttService" />
    </application>
</manifest>
```

#### 总结

MQTT内容并不是太多，一般只有一个连接，如果不同模块需要不同的消息源，订阅不同的Topic主题即可。