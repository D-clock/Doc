# AIDL开发笔记+读书笔记

近一年来的开发工作都涉及到AIDL，恰好最近又看了任玉刚著的《Android开发艺术探索》。于是，决定整理一下思路，把这段时间零零散散的开发经验和读书笔记一齐来写一篇总结。错误之处，还望批评指出。（本文所有示例代码都是AndroidStudio环境下产生，至于ADT，已经不用很久了....）

## 1.基本使用

AIDL用于Android应用进行多进程通信，其目的是为了简化多进程操作，方便开发者忽略多进程通信的一些底层细节，而把精力集中到业务编写上。它的基本使用模式如下：

### 1.1 创建AIDL文件（AndroidStudio可以使用IDE生成）

```java

// 举个栗子，创建IServerHelper.aidl

// IServerHelper.aidl
package com.clock.aipc;

// Declare any non-default types here with import statements

interface IServerHelper {

    /**
    *两个参数相加
    *
    * @param a
    * @param b
    **/
    int add(int a , int b);
}


```

### 1.2 创建Service，并在AndroidManifest.xml中作如下配置Service的android:process属性（也可以不配置，具体区别后面会说明）

```java

public class RemoteService extends Service {

    private final static String TAG = RemoteService.class.getSimpleName();

    private IServerHelper.Stub mServerHelper = new IServerHelper.Stub() {
        @Override
        public int add(int a, int b) throws RemoteException {
            return a + b;
        }
    };

    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return START_STICKY;
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return mServerHelper;
    }

}

```

```xml

<service
	android:name=".service.RemoteService"
    android:process=":process" />

```

### 1.3 创建绑定Service，实现方法调用

```java

public class AidlActivity extends AppCompatActivity implements View.OnClickListener {

    private final static String TAG = AidlActivity.class.getSimpleName();

    private IServerHelper mServiceHelper;

    private ServiceConnection mServiceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mServiceHelper = IServerHelper.Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            mServiceHelper = null;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_aidl);

        findViewById(R.id.btn_bind_service).setOnClickListener(this);
        findViewById(R.id.btn_add_method).setOnClickListener(this);

    }

    @Override
    public void onClick(View v) {
        int viewId = v.getId();
        if (viewId == R.id.btn_bind_service) {
            if (!serviceIsLive()) {//创建绑定远程Service
                Intent intent = new Intent(this, RemoteService.class);
                startService(intent);
                bindService(intent, mServiceConnection, Context.BIND_AUTO_CREATE);
            }

        } else if (viewId == R.id.btn_add_method) {
            if (serviceIsLive()) {
                try {
                    int result = mServiceHelper.add(1, 2);//调用远程方法
                    Log.i(TAG, "1+2 = " + result);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * 判断当前绑定的服务是否还存活
     *
     * @return
     */
    private boolean serviceIsLive() {
        if (mServiceHelper == null) {
            return false;
        }
        return true;
    }

}


```

到此，已经能够完成一个AIDL的基本使用。


## 2.读书笔记和心得汇总

下面揉合了《Android开发艺术探索》读书笔记和自己平常开发的知识汇总

### 2.1 aidl文件的作用

aidl文件是google为了方便我们编写代码而提供的一种特殊格式的文件，编译器会在编译的时候根据aidl文件生成与其相对于的java类。

![](http://g.hiphotos.baidu.com/image/pic/item/2f738bd4b31c8701276e01f5207f9e2f0608ffcd.jpg)

所以，我们也可以不创建aidl文件，直接参照aidl生成的对应java代码的格式，来编写我们需要的跨进程通讯类。


### 2.2 aidl文件生成的类的结构


同样以1.1中的IServerHelper.aidl为例，它所生成的java类的代码和解析。

```java

/*
 * This file is auto-generated.  DO NOT MODIFY.
 * Original file: E:\\AndroidStudio\\ChatLink\\AndroidIPC\\src\\main\\aidl\\com\\clock\\aipc\\IServerHelper.aidl
 */
package com.clock.aipc;
// Declare any non-default types here with import statements

public interface IServerHelper extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.clock.aipc.IServerHelper {
		
		// Binder的唯一标识，一般用Binder的类名全称表示
        private static final java.lang.String DESCRIPTOR = "com.clock.aipc.IServerHelper";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * 将服务器端的Binder转换成为客户端所需要的对应的AIDL对象，这种转换区分进程，如果客户端和服务端处于同一进程，则返回* * Stub对象本身，否则，返回系统封装好的Stub.Proxy代理对象。(这就是1.2中属性配不配置的区别)
         *
         */
        public static com.clock.aipc.IServerHelper asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.clock.aipc.IServerHelper))) {
                return ((com.clock.aipc.IServerHelper) iin);//返回Stub对象本身
            }
            return new com.clock.aipc.IServerHelper.Stub.Proxy(obj);//返回系统封装好的Stub.Proxy代理对象
        }

		/**
		 * 返回当前的Binder对象
		 *
		 */
        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

		/**
		 * 客户端中所有跨进程请求调用，最终都会经过此方法，它运行在服务端Binder线程池中
		 *
		 */
        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(DESCRIPTOR);
                    return true;
                }
                case TRANSACTION_add: {
                    data.enforceInterface(DESCRIPTOR);
                    int _arg0;
                    _arg0 = data.readInt();
                    int _arg1;
                    _arg1 = data.readInt();
                    int _result = this.add(_arg0, _arg1);
                    reply.writeNoException();
                    reply.writeInt(_result);
                    return true;
                }
            }
            return super.onTransact(code, data, reply, flags);
        }

        private static class Proxy implements com.clock.aipc.IServerHelper {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            /**
             * 两个参数相加
             *
             * @param a
             * @param b
             */
            @Override
            public int add(int a, int b) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                int _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeInt(a);
                    _data.writeInt(b);
                    mRemote.transact(Stub.TRANSACTION_add, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.readInt();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }
        }
		// 远程方法对应的编码，每个远程方法都有唯一一个，它在onTransact方法中起作用
        static final int TRANSACTION_add = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
    }

    /**
     * 两个参数相加
     *
     * @param a
     * @param b
     */
    public int add(int a, int b) throws android.os.RemoteException;
}

```

### 2.3 aidl的支持传递的数据类型

- 基本数据类型
- **String** 和** CharSequence**
- **List**：仅支持 **ArrayList**，里面每个元素都必须能够被AIDL支持
- **Map**：只支持**HashMap**，里面的所有key、value都能够被AIDL支持
- **Parcelable**：实现了所有Parcelable接口的对象
- **AIDL**本身

### 2.4 跨进程的普通调用和UI更新

- 所有跨进程的调用都是在Binder线程池中进行，会导致当前调用线程阻塞挂起。
- 为了避免产生**ANR**问题，不要再UI线程中进行跨进程调用。可以使用Handler+线程池，不过我更建议用**AsyncTask**，毕竟Google已经封装得很好了，省去我们很多事。
- 服务端对客户端的接口回调也是在Binder线程池中，因此不能直接更新UI，需要使用**Handler**来进行更新，这时候就不能用**AsyncTask**了，因为AsyncTask是不允许在非UI线程中被创建的。

### 2.5 跨进程的注册与反注册

在同一个进程下普通的的register和unregister是无法在跨进程终身生效的，需要使用系统提供的 **RemoteCallbackList**

### 2.6 监听进程是否死亡

利用系统提供的**IBinder.DeathRecipient**来给监听一个进程是否死亡：

- 客户端在成功绑定服务端时，可以利用拿到的Binder调用**linkToDeath(IBinder.DeathRecipient recipient, int flags)**方法，注册监听服务端进程是否存活。
- 客户端可以向服务端注册一个Binder作为token，服务端可以拿到此token后，也按照如上操作，就可以得知客户端进程是否存活。
- **onServiceDisconnected**方法也可以得知客户端是否存活，与DeathRecipient的区别在于DeathRecipient接口是在异步线程中被调用执行
- 客户端可以利用此接口重新和挂掉的服务端建立连接，服务端可以利用此接口重新唤起客户端进程（UI进程）

### 2.7 访问权限控制

为了不让其他应用能够调用我们创建的远程方法，我们可以做以下相应的处理

1. **自定义私有权限**
2. **在Service的onBind方法或Binder的onTransact方法中，使用Context.checkCallingOrSelfPermission()方法校验权限**



## 相关参考资源来源

- 任玉刚的《Android开发艺术探索》
- http://developer.android.com/guide/components/aidl.html
- http://www.androiddesignpatterns.com/2013/08/binders-death-recipients.html
- http://developer.android.com/reference/android/os/RemoteCallbackList.html
- http://developer.android.com/reference/android/os/IBinder.html
- http://developer.android.com/reference/android/os/IBinder.DeathRecipient.html


以上属于个人开发总结+读书笔记整合，如有错误，还请批评指正。