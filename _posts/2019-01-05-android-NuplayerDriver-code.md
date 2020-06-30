---
layout: post
title: android NuplayerDriver类源码分析
featured-img: sleek
mathjax: true
categories: [andriod,NuplayerFramework,NuplayerDriverCode]
---

### NuplayerDriver.cpp 简介

​	Nuplayer框架中最顶层的类是NuplayerDriver，继承自MediaPlayerInterface，只要提供一个状态装换机制，作为Nuplayer类的wrapper（转换器）。NuplayerDriver类中最重要的成员是以下几个：

- State mState : 播放器状态标志
  - State是一个枚举类型，其中枚举了播放器的各种状态

```c++
private:
    enum State {
        STATE_IDLE,
        STATE_SET_DATASOURCE_PENDING,
        STATE_UNPREPARED,
        STATE_PREPARING,
        STATE_PREPARED,
        STATE_RUNNING,
        STATE_PAUSED,
        STATE_RESET_IN_PROGRESS,
        STATE_STOPPED,                  // equivalent to PAUSED
        STATE_STOPPED_AND_PREPARING,    // equivalent to PAUSED, but seeking
        STATE_STOPPED_AND_PREPARED,     // equivalent to PAUSED, but seek complete
    };
```

- sp<Alooper> mLooper : 内部消息驱动机制
- sp<Nuplayer> mPlayer : 智能指针，一个Nuplayer对象，真正完成播放器的类



### 构造函数和析构函数

​	构造函数如下：

```c++
NuPlayerDriver::NuPlayerDriver(pid_t pid)
    : mState(STATE_IDLE),
      mIsAsyncPrepare(false),
      mAsyncResult(UNKNOWN_ERROR),
      mSetSurfaceInProgress(false),
      mDurationUs(-1),
      mPositionUs(-1),
      mSeekInProgress(false),
      mPlayingTimeUs(0),
      mLooper(new ALooper),
      mPlayer(new NuPlayer(pid)),
      mPlayerFlags(0),
      mAnalyticsItem(NULL),
      mClientUid(-1),
      mAtEOS(false),
      mLooping(false),
      mAutoLoop(false) {
    ALOGD("NuPlayerDriver(%p) created, clientPid(%d)", this, pid);
    mLooper->setName("NuPlayerDriver Looper");

    // set up an analytics record
    mAnalyticsItem = new MediaAnalyticsItem(kKeyPlayer);
    mAnalyticsItem->generateSessionID();

    mLooper->start(
            false, /* runOnCallingThread */
            true,  /* canCallJava */
            PRIORITY_AUDIO);

    mLooper->registerHandler(mPlayer);

    mPlayer->setDriver(this);
}
```

构造函数的主要作用是：给成员变量赋值，并且创建ALooper和Nuplayer的实例，并且将它们关联起来，实现Nuplayer和NuplayerDriver之间的相互引用。



析构函数如下：

```c++
NuPlayerDriver::~NuPlayerDriver() {
    ALOGV("~NuPlayerDriver(%p)", this);
    mLooper->stop();

    // finalize any pending metrics, usually a no-op.
    updateMetrics("destructor");
    logMetrics("destructor");

    if (mAnalyticsItem != NULL) {
        delete mAnalyticsItem;
        mAnalyticsItem = NULL;
    }
}
```

析构函数的主要作用是：销毁创建的ALooper和Nuplayer。



### SetDataSource函数

​	在此类中总对这个方法有4种重载，分别对应Nuplayer类中的4种重载。分别如下：

```c++
status_t NuPlayerDriver::setDataSource(
        const sp<IMediaHTTPService> &httpService,
        const char *url,
        const KeyedVector<String8, String8> *headers) {
    ALOGV("setDataSource(%p) url(%s)", this, uriDebugString(url, false).c_str());
    Mutex::Autolock autoLock(mLock);

    if (mState != STATE_IDLE) {
        return INVALID_OPERATION;
    }

    mState = STATE_SET_DATASOURCE_PENDING;

    mPlayer->setDataSourceAsync(httpService, url, headers);

    while (mState == STATE_SET_DATASOURCE_PENDING) {
        mCondition.wait(mLock);
    }

    return mAsyncResult;
}
```



```c++
status_t NuPlayerDriver::setDataSource(int fd, int64_t offset, int64_t length) {
    ALOGV("setDataSource(%p) file(%d)", this, fd);
    Mutex::Autolock autoLock(mLock);

    if (mState != STATE_IDLE) {
        return INVALID_OPERATION;
    }

    mState = STATE_SET_DATASOURCE_PENDING;

    mPlayer->setDataSourceAsync(fd, offset, length);

    while (mState == STATE_SET_DATASOURCE_PENDING) {
        mCondition.wait(mLock);
    }

    return mAsyncResult;
}
```



```c++
status_t NuPlayerDriver::setDataSource(const sp<IStreamSource> &source) {
    ALOGV("setDataSource(%p) stream source", this);
    Mutex::Autolock autoLock(mLock);

    if (mState != STATE_IDLE) {
        return INVALID_OPERATION;
    }

    mState = STATE_SET_DATASOURCE_PENDING;

    mPlayer->setDataSourceAsync(source);

    while (mState == STATE_SET_DATASOURCE_PENDING) {
        mCondition.wait(mLock);
    }

    return mAsyncResult;
}
```



```c++
status_t NuPlayerDriver::setDataSource(const sp<DataSource> &source) {
    ALOGV("setDataSource(%p) callback source", this);
    Mutex::Autolock autoLock(mLock);

    if (mState != STATE_IDLE) {
        return INVALID_OPERATION;
    }

    mState = STATE_SET_DATASOURCE_PENDING;

    mPlayer->setDataSourceAsync(source);

    while (mState == STATE_SET_DATASOURCE_PENDING) {
        mCondition.wait(mLock);
    }

    return mAsyncResult;
}
```

​	mAsyncResult为本类中的一个int 类型的成员变量。在创建对象时被初始化为UNKNOWN_ERROR，定义如下：

```c++
#ifndef __ANDROID__
    OK                  = 0,
    BAD_VALUE           = -EINVAL,
    BAD_INDEX           = -EOVERFLOW,
    UNKNOWN_TRANSACTION = -EBADMSG,
    ALREADY_EXISTS      = -EEXIST,
    NAME_NOT_FOUND      = -ENOENT,
    INVALID_OPERATION   = -ENOSYS,
    NO_MEMORY           = -ENOMEM,
    PERMISSION_DENIED   = -EPERM,
    TIMED_OUT           = -ETIMEDOUT,
    UNKNOWN_ERROR       = -EINVAL,
#endif
```

​	

​	可以看出，这4个方法实现都差不多，首先判断播放器当前状态，如果是STATE_IDLE状态，则返回INVALID_OPERATION。

​	然后设置播放器的状态，并调用mPlayer（即Nuplayer类的实例）相对应的setDataSourceAsync方法。

​	等待setDataSourceAsync方法执行完成，setDataSourceAsync方法执行完成之后，会调用此类中的notifySetDataSourceCompleted方法，此方法会改变播放器的状态（mState）和mAsyncResult的值。此方法如下:

```c++
void NuPlayerDriver::notifySetDataSourceCompleted(status_t err) {
    Mutex::Autolock autoLock(mLock);

    CHECK_EQ(mState, STATE_SET_DATASOURCE_PENDING);

    mAsyncResult = err;
    mState = (err == OK) ? STATE_UNPREPARED : STATE_IDLE;
    mCondition.broadcast();
}
```



### setVideoSurfaceTexture方法

​	这个方法用于播放视频，方法体如下：

```c++
status_t NuPlayerDriver::setVideoSurfaceTexture(
        const sp<IGraphicBufferProducer> &bufferProducer) {
    ALOGV("setVideoSurfaceTexture(%p)", this);
    Mutex::Autolock autoLock(mLock);

    if (mSetSurfaceInProgress) {
        return INVALID_OPERATION;
    }

    switch (mState) {
        case STATE_SET_DATASOURCE_PENDING:
        case STATE_RESET_IN_PROGRESS:
            return INVALID_OPERATION;

        default:
            break;
    }

    mSetSurfaceInProgress = true;

    mPlayer->setVideoSurfaceTextureAsync(bufferProducer);

    while (mSetSurfaceInProgress) {
        mCondition.wait(mLock);
    }

    return OK;
}
```

​	此方法调用了Nuplayer：：setVideoSurfaceTextureAsync方法，然后等待调用方法执行完成。其中调用的方法又会调用本类（NuplayerDriver）中的notifySetSurfaceComplete方法，改变成员mSetSurfaceInProgress（boolean类型）的值为false。

```C++
void NuPlayerDriver::notifySetSurfaceComplete() {
    ALOGV("notifySetSurfaceComplete(%p)", this);
    Mutex::Autolock autoLock(mLock);

    CHECK(mSetSurfaceInProgress);
    mSetSurfaceInProgress = false;

    mCondition.broadcast();
}
```



### prepare/prepareAsync方法

这两个接口功能基本一致，只是第二个是异步的调用过程。

```c++
status_t NuPlayerDriver::prepare() {
    ALOGV("prepare(%p)", this);
    Mutex::Autolock autoLock(mLock);//使用互斥类，保证只有一个线程能访问
    return prepare_l();
}

//通过判断播放器的状态（mState）来执行不同的方法
status_t NuPlayerDriver::prepare_l() {
    switch (mState) {
        case STATE_UNPREPARED:
            mState = STATE_PREPARING;

            // Make sure we're not posting any notifications, success or
            // failure information is only communicated through our result
            // code.
            mIsAsyncPrepare = false;
            mPlayer->prepareAsync();
            while (mState == STATE_PREPARING) {
                mCondition.wait(mLock);
            }
            return (mState == STATE_PREPARED) ? OK : UNKNOWN_ERROR;
        case STATE_STOPPED:
            // this is really just paused. handle as seek to start
            mAtEOS = false;
            mState = STATE_STOPPED_AND_PREPARING;
            mIsAsyncPrepare = false;
            mPlayer->seekToAsync(0, MediaPlayerSeekMode::SEEK_PREVIOUS_SYNC /* mode */,
                    true /* needNotify */);
            while (mState == STATE_STOPPED_AND_PREPARING) {
                mCondition.wait(mLock);
            }
            return (mState == STATE_STOPPED_AND_PREPARED) ? OK : UNKNOWN_ERROR;
        default:
            return INVALID_OPERATION;
    };
}
```



```c++
status_t NuPlayerDriver::prepareAsync() {
    ALOGV("prepareAsync(%p)", this);
    Mutex::Autolock autoLock(mLock);

    switch (mState) {
        case STATE_UNPREPARED:
            mState = STATE_PREPARING;
            mIsAsyncPrepare = true;
            mPlayer->prepareAsync();
            return OK;
        case STATE_STOPPED:
            // this is really just paused. handle as seek to start
            mAtEOS = false;
            mState = STATE_STOPPED_AND_PREPARING;
            mIsAsyncPrepare = true;
            mPlayer->seekToAsync(0, MediaPlayerSeekMode::SEEK_PREVIOUS_SYNC /* mode */,
                    true /* needNotify */);
            return OK;
        default:
            return INVALID_OPERATION;
    };
}
```

可以看出两个接口都是调用了Nuplayer：：prepareAsync（）方法，prepare方法会等待调用方法的执行，而prepareAsync方法不会等待。在调用的方法执行完成之后，会向上面一样，使用ALooper/AHandler机制，发送信息给Nuplayer实例，并且会调用NuplayerDriver::notifyPrepareCompleted方法，此方法如下：

```c++
void NuPlayerDriver::notifyPrepareCompleted(status_t err) {
    ALOGV("notifyPrepareCompleted %d", err);

    Mutex::Autolock autoLock(mLock);

    if (mState != STATE_PREPARING) {
        // We were preparing asynchronously when the client called
        // reset(), we sent a premature "prepared" notification and
        // then initiated the reset. This notification is stale.
        CHECK(mState == STATE_RESET_IN_PROGRESS || mState == STATE_IDLE);
        return;
    }

    CHECK_EQ(mState, STATE_PREPARING);

    mAsyncResult = err;

    if (err == OK) {
        // update state before notifying client, so that if client calls back into NuPlayerDriver
        // in response, NuPlayerDriver has the right state
        mState = STATE_PREPARED;
        if (mIsAsyncPrepare) {
            notifyListener_l(MEDIA_PREPARED);
        }
    } else {
        mState = STATE_UNPREPARED;
        if (mIsAsyncPrepare) {
            notifyListener_l(MEDIA_ERROR, MEDIA_ERROR_UNKNOWN, err);
        }
    }

    sp<MetaData> meta = mPlayer->getFileMeta();
    int32_t loop;
    if (meta != NULL
            && meta->findInt32(kKeyAutoLoop, &loop) && loop != 0) {
        mAutoLoop = true;
    }

    mCondition.broadcast();
}

```



### start和stop方法

这两个函数作为开始播放和停止播放的接口，主要涉及到播放器内部状态的切换和判断。方法体如下：

```c++
status_t NuPlayerDriver::start() {
    ALOGD("start(%p), state is %d, eos is %d", this, mState, mAtEOS);
    Mutex::Autolock autoLock(mLock);
    return start_l();
}

status_t NuPlayerDriver::start_l() {
    switch (mState) {
        case STATE_UNPREPARED:
        {
            status_t err = prepare_l();

            if (err != OK) {
                return err;
            }

            CHECK_EQ(mState, STATE_PREPARED);//看C++语法解释中的内容

            // fall through
        }

        case STATE_PAUSED:
        case STATE_STOPPED_AND_PREPARED:
        case STATE_PREPARED:
        {
            mPlayer->start();

            // fall through
        }

        case STATE_RUNNING:
        {
            if (mAtEOS) {
                mPlayer->seekToAsync(0);
                mAtEOS = false;
                mPositionUs = -1;
            }
            break;
        }

        default:
            return INVALID_OPERATION;
    }

    mState = STATE_RUNNING;

    return OK;
}
```



```c++
status_t NuPlayerDriver::stop() {
    ALOGD("stop(%p)", this);
    Mutex::Autolock autoLock(mLock);

    switch (mState) {
        case STATE_RUNNING:
            mPlayer->pause();
            // fall through

        case STATE_PAUSED:
            mState = STATE_STOPPED;
            notifyListener_l(MEDIA_STOPPED);
            break;

        case STATE_PREPARED:
        case STATE_STOPPED:
        case STATE_STOPPED_AND_PREPARING:
        case STATE_STOPPED_AND_PREPARED:
            mState = STATE_STOPPED;
            break;

        default:
            return INVALID_OPERATION;
    }

    return OK;
}
```

start方法的最终功能是通过调用Nuplayer::start方法实现的。

stop方法的最终功能实现是通过调用Nuplayer::pause方法实现的。

这两个方法主要是判断不同状态下的调用逻辑。



### pause和reset方法

pause方法用于暂停，其实现比较简单，直接调用Nuplayer中的pause方法，方法体如下：

```c++
status_t NuPlayerDriver::pause() {
    ALOGD("pause(%p)", this);
    // The NuPlayerRenderer may get flushed if pause for long enough, e.g. the pause timeout tear
    // down for audio offload mode. If that happens, the NuPlayerRenderer will no longer know the
    // current position. So similar to seekTo, update |mPositionUs| to the pause position by calling
    // getCurrentPosition here.
    int unused;
    getCurrentPosition(&unused);

    Mutex::Autolock autoLock(mLock);

    switch (mState) {
        case STATE_PAUSED:
        case STATE_PREPARED:
            return OK;

        case STATE_RUNNING:
            mState = STATE_PAUSED;
            notifyListener_l(MEDIA_PAUSED);
            mPlayer->pause();
            break;

        default:
            return INVALID_OPERATION;
    }

    return OK;
}
```

reset方法是重置播放器，这是一个同步调用的接口，最终实现是通过调用Nuplayer::resetAsync接口，然后在调用NuplayerDriver::notifyResetComplete方法通知。reset方法如下：

```c++
status_t NuPlayerDriver::reset() {
    ALOGD("reset(%p) at state %d", this, mState);

    updateMetrics("reset");
    logMetrics("reset");

    Mutex::Autolock autoLock(mLock);

    switch (mState) {
        case STATE_IDLE:
            return OK;

        case STATE_SET_DATASOURCE_PENDING:
        case STATE_RESET_IN_PROGRESS:
            return INVALID_OPERATION;

        case STATE_PREPARING:
        {
            CHECK(mIsAsyncPrepare);

            notifyListener_l(MEDIA_PREPARED);
            break;
        }

        default:
            break;
    }

    if (mState != STATE_STOPPED) {
        notifyListener_l(MEDIA_STOPPED);
    }

    mState = STATE_RESET_IN_PROGRESS;
    mPlayer->resetAsync();

    while (mState == STATE_RESET_IN_PROGRESS) {
        mCondition.wait(mLock);
    }

    mDurationUs = -1;
    mPositionUs = -1;
    mLooping = false;
    mPlayingTimeUs = 0;

    return OK;
}
```

调用的NuplayerDriver::notifyResetComplete方法如下：

```c++
void NuPlayerDriver::notifyResetComplete() {
    ALOGD("notifyResetComplete(%p)", this);
    Mutex::Autolock autoLock(mLock);

    CHECK_EQ(mState, STATE_RESET_IN_PROGRESS);
    mState = STATE_IDLE;
    mCondition.broadcast();
}
```



### isPlaying / getDuration / getCurrentPosition方法

这几个方法用于获得播放器的状态，其实现如下：

```c++
bool NuPlayerDriver::isPlaying() {
    return mState == STATE_RUNNING && !mAtEOS;
}
```



getDuration的实现也相对简单，代码如下。不过具体获取的mDurationUs需要通过NuPlayer上报或者定期查询更新下。

```c++
status_t NuPlayerDriver::getDuration(int *msec) {
    Mutex::Autolock autoLock(mLock);

    if (mDurationUs < 0) {
        return UNKNOWN_ERROR;
    }

    *msec = (mDurationUs + 500ll) / 1000;

    return OK;
}
```



getCurrentPosition则是通过调用NuPlayer::getCurrentPosition获取。

```c++
status_t NuPlayerDriver::getCurrentPosition(int *msec) {
    int64_t tempUs = 0;
    {
        Mutex::Autolock autoLock(mLock);
        if (mSeekInProgress || (mState == STATE_PAUSED && !mAtEOS)) {
            tempUs = (mPositionUs <= 0) ? 0 : mPositionUs;
            *msec = (int)divRound(tempUs, (int64_t)(1000));
            return OK;
        }
    }

    status_t ret = mPlayer->getCurrentPosition(&tempUs);

    Mutex::Autolock autoLock(mLock);
    // We need to check mSeekInProgress here because mPlayer->seekToAsync is an async call, which
    // means getCurrentPosition can be called before seek is completed. Iow, renderer may return a
    // position value that's different the seek to position.
    if (ret != OK) {
        tempUs = (mPositionUs <= 0) ? 0 : mPositionUs;
    } else {
        mPositionUs = tempUs;
    }
    *msec = (int)divRound(tempUs, (int64_t)(1000));
    return OK;
}
```



### getMetadata方法

这个接口主要是判断播放器的属性，比如是否支持暂停、seek、向前seek、向后seek等。

```c++
status_t NuPlayerDriver::getMetadata(
        const media::Metadata::Filter& /* ids */, Parcel *records) {
    Mutex::Autolock autoLock(mLock);

    using media::Metadata;

    Metadata meta(records);

    meta.appendBool(
            Metadata::kPauseAvailable,
            mPlayerFlags & NuPlayer::Source::FLAG_CAN_PAUSE);

    meta.appendBool(
            Metadata::kSeekBackwardAvailable,
            mPlayerFlags & NuPlayer::Source::FLAG_CAN_SEEK_BACKWARD);

    meta.appendBool(
            Metadata::kSeekForwardAvailable,
            mPlayerFlags & NuPlayer::Source::FLAG_CAN_SEEK_FORWARD);

    meta.appendBool(
            Metadata::kSeekAvailable,
            mPlayerFlags & NuPlayer::Source::FLAG_CAN_SEEK);

    return OK;
}
```

其中Metadta类的分析：戳这里（未实现）