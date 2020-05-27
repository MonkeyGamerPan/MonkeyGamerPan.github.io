---
layout: post
title: android DecoderBase类源码分析
featured-img: sleek
mathjax: true
categories: [andriod,NuplayerFrame]
---

## DecoderBase.cpp简介

​	DecoderBase是AHandler的一个子类，主要功能是负责解码，按照MediaPlayer的框架，一般是调用MediaCodec完成解码的，功能类似FFmpeg的libavcodec。其主要功能是解码器初始化、解码以及和其他模块的交互，比如Source、Render等。

​	Nuplayer::Decoder是DecoderBase的子类。



### Nuplayer中关于DecoderBase的调用

Nuplayer的成员变量中有两个DecoderBase的智能指针：

```c++
sp<DecoderBase> mVideoDecoder;
sp<DecoderBase> mAudioDecoder;
```





### Nuplayer::DecoderBase类分析

路径：/frameworks/av/media/libmediaplayerservice/nuplayer/NuPlayerDecoderBase.h

此类的声明如下：

```c++
namespace android {

struct ABuffer;
struct MediaCodec;
class MediaBuffer;
class MediaCodecBuffer;
class Surface;

struct NuPlayer::DecoderBase : public AHandler {
    explicit DecoderBase(const sp<AMessage> &notify);

    void configure(const sp<AMessage> &format);
    void init();
    void setParameters(const sp<AMessage> &params);

    // Synchronous call to ensure decoder will not request or send out data.
    void pause();

    void setRenderer(const sp<Renderer> &renderer);
    virtual status_t setVideoSurface(const sp<Surface> &) { return INVALID_OPERATION; }

    void signalFlush();
    void signalResume(bool notifyComplete);
    void initiateShutdown();

    virtual sp<AMessage> getStats() const {
        return mStats;
    }

    virtual status_t releaseCrypto() {
        return INVALID_OPERATION;
    }

    enum {
        kWhatInputDiscontinuity  = 'inDi',
        kWhatVideoSizeChanged    = 'viSC',
        kWhatFlushCompleted      = 'flsC',
        kWhatShutdownCompleted   = 'shDC',
        kWhatResumeCompleted     = 'resC',
        kWhatEOS                 = 'eos ',
        kWhatError               = 'err ',
    };

protected:

    virtual ~DecoderBase();

    void stopLooper();

    virtual void onMessageReceived(const sp<AMessage> &msg);

    virtual void onConfigure(const sp<AMessage> &format) = 0;
    virtual void onSetParameters(const sp<AMessage> &params) = 0;
    virtual void onSetRenderer(const sp<Renderer> &renderer) = 0;
    virtual void onResume(bool notifyComplete) = 0;
    virtual void onFlush() = 0;
    virtual void onShutdown(bool notifyComplete) = 0;

    void onRequestInputBuffers();
    virtual bool doRequestBuffers() = 0;
    virtual void handleError(int32_t err);

    sp<AMessage> mNotify;
    int32_t mBufferGeneration;
    bool mPaused;
    sp<AMessage> mStats;

private:
    enum {
        kWhatConfigure           = 'conf',
        kWhatSetParameters       = 'setP',
        kWhatSetRenderer         = 'setR',
        kWhatPause               = 'paus',
        kWhatRequestInputBuffers = 'reqB',
        kWhatFlush               = 'flus',
        kWhatShutdown            = 'shuD',
    };

    sp<ALooper> mDecoderLooper;
    bool mRequestInputBuffersPending;

    DISALLOW_EVIL_CONSTRUCTORS(DecoderBase);
};

}  // namespace android
```

​	从声明上来看，DecoderBase主要是基于AHandler-ALooper搭建一个解码器框架和消息循环泵，将对public接口的调用直接转移到onXXX接口上。

​	下面以configure接口调用为例简单说明DecoderBase中的调用逻辑。代码如下：

```c++
void NuPlayer::DecoderBase::configure(const sp<AMessage> &format) {
    sp<AMessage> msg = new AMessage(kWhatConfigure, this);
    msg->setMessage("format", format);
    msg->post();
}
```

这个方法主要是发送了一个kWhatConfigure消息。然而消息的接受者是在DecoderBase类中的onMessageReceived(const sp<AMessage> &msg)方法中：

```c++
void NuPlayer::DecoderBase::onMessageReceived(const sp<AMessage> &msg) {
    switch (msg->what()) {
        case kWhatConfigure:
        {
            sp<AMessage> format;
            CHECK(msg->findMessage("format", &format));
            onConfigure(format);
            break;
        }
        ...
}
```

​	在这个方法中就调用了onConfigure方法，这就说明了其实DecoderBase将对public接口的调用直接转移到onXXX接口上。

​	**但是在DecoderBase这个类中，并没有对onConfigure这个方法进行实现。**

​	**Nuplayer中的DecoderBase类有两个子类，一个是Decoder，另一个是DecoderPassThrough（从Nuplayer来看这个类只和音频解码有关）。**

​	这里我们先来看下第一个子类Decoder。



### Nuplayer: :Decoder接口和主要成员

这个类是真正调用MediaCodec实现解码的类，此类的声明如下：

```c++
struct NuPlayer::Decoder : public DecoderBase {
    Decoder(const sp<AMessage> &notify,
            const sp<Source> &source,
            pid_t pid,
            uid_t uid,
            const sp<Renderer> &renderer = NULL,
            const sp<Surface> &surface = NULL,
            const sp<CCDecoder> &ccDecoder = NULL);

    virtual sp<AMessage> getStats() const;

    // sets the output surface of video decoders.
    virtual status_t setVideoSurface(const sp<Surface> &surface);

    virtual status_t releaseCrypto();

protected:
    virtual ~Decoder();

    virtual void onMessageReceived(const sp<AMessage> &msg);

    virtual void onConfigure(const sp<AMessage> &format);
    virtual void onSetParameters(const sp<AMessage> &params);
    virtual void onSetRenderer(const sp<Renderer> &renderer);
    virtual void onResume(bool notifyComplete);
    virtual void onFlush();
    virtual void onShutdown(bool notifyComplete);
    virtual bool doRequestBuffers();

private:
    enum {
        kWhatCodecNotify         = 'cdcN',
        kWhatRenderBuffer        = 'rndr',
        kWhatSetVideoSurface     = 'sSur',
        kWhatAudioOutputFormatChanged = 'aofc',
        kWhatDrmReleaseCrypto    = 'rDrm',
    };

    enum {
        kMaxNumVideoTemporalLayers = 32,
    };

    sp<Surface> mSurface;

    sp<Source> mSource;
    sp<Renderer> mRenderer;
    sp<CCDecoder> mCCDecoder;

    sp<AMessage> mInputFormat;
    sp<AMessage> mOutputFormat;
    sp<MediaCodec> mCodec; //真正的解码
    sp<ALooper> mCodecLooper;

    List<sp<AMessage> > mPendingInputMessages;

    Vector<sp<MediaCodecBuffer> > mInputBuffers;//输入数据
    Vector<sp<MediaCodecBuffer> > mOutputBuffers;//输出数据
    Vector<sp<ABuffer> > mCSDsForCurrentFormat;
    Vector<sp<ABuffer> > mCSDsToSubmit;
    Vector<bool> mInputBufferIsDequeued;
    Vector<MediaBuffer *> mMediaBuffers;
    Vector<size_t> mDequeuedInputBuffers;

    const pid_t mPid;
    const uid_t mUid;
    int64_t mSkipRenderingUntilMediaTimeUs;
    int64_t mNumFramesTotal;
    int64_t mNumInputFramesDropped;
    int64_t mNumOutputFramesDropped;
    int32_t mVideoWidth;
    int32_t mVideoHeight;
    bool mIsAudio;
    bool mIsVideoAVC;
    bool mIsSecure;
    bool mIsEncrypted;
    bool mIsEncryptedObservedEarlier;
    bool mFormatChangePending;
    bool mTimeChangePending;
    float mFrameRateTotal;
    float mPlaybackSpeed;
    int32_t mNumVideoTemporalLayerTotal;
    int32_t mNumVideoTemporalLayerAllowed;
    int32_t mCurrentMaxVideoTemporalLayerId;
    float mVideoTemporalLayerAggregateFps[kMaxNumVideoTemporalLayers];

    bool mResumePending;
    AString mComponentName;
    bool mlog_enable;  // mtk add for log reduce
    int64_t mInputReadBegin;  // mtk add for debug input read time

    void handleError(int32_t err);
    bool handleAnInputBuffer(size_t index);
    bool handleAnOutputBuffer(
            size_t index,
            size_t offset,
            size_t size,
            int64_t timeUs,
            int32_t flags);
    void handleOutputFormatChange(const sp<AMessage> &format);

    void releaseAndResetMediaBuffers();
    void requestCodecNotification();
    bool isStaleReply(const sp<AMessage> &msg);

    void doFlush(bool notifyComplete);
    status_t fetchInputData(sp<AMessage> &reply);
    bool onInputBufferFetched(const sp<AMessage> &msg);
    void onRenderBuffer(const sp<AMessage> &msg);

    bool supportsSeamlessFormatChange(const sp<AMessage> &to) const;
    bool supportsSeamlessAudioFormatChange(const sp<AMessage> &targetFormat) const;
    void rememberCodecSpecificData(const sp<AMessage> &format);
    bool isDiscontinuityPending() const;
    void finishHandleDiscontinuity(bool flushOnTimeChange);

    void notifyResumeCompleteIfNecessary();

    void onReleaseCrypto(const sp<AMessage>& msg);

    DISALLOW_EVIL_CONSTRUCTORS(Decoder);
};
```

从声明上来看接口多数是继承自DecoderBase的public和protected成员函数，这里最主要的成员是mCodec，接下来的流程梳理也是围绕这一点展开。



#  DecoderBase/Decoder实现解析

### 构造函数和析构函数

从代码上来看这两个函数的主要作用就是创建和销毁Looper，同时释放MediaCodec相关资源。代码如下：

```c++
NuPlayer::Decoder::Decoder(
        const sp<AMessage> &notify,
        const sp<Source> &source,
        pid_t pid,
        uid_t uid,
        const sp<Renderer> &renderer,
        const sp<Surface> &surface,
        const sp<CCDecoder> &ccDecoder)
    : DecoderBase(notify),
      mSurface(surface),
      mSource(source),
      mRenderer(renderer),
      mCCDecoder(ccDecoder),
      mPid(pid),
      mUid(uid),
      mSkipRenderingUntilMediaTimeUs(-1ll),
      mNumFramesTotal(0ll),
      mNumInputFramesDropped(0ll),
      mNumOutputFramesDropped(0ll),
      mVideoWidth(0),
      mVideoHeight(0),
      mIsAudio(true),
      mIsVideoAVC(false),
      mIsSecure(false),
      mIsEncrypted(false),
      mIsEncryptedObservedEarlier(false),
      mFormatChangePending(false),
      mTimeChangePending(false),
      mFrameRateTotal(kDefaultVideoFrameRateTotal),
      mPlaybackSpeed(1.0f),
      mNumVideoTemporalLayerTotal(1), // decode all layers
      mNumVideoTemporalLayerAllowed(1),
      mCurrentMaxVideoTemporalLayerId(0),
      mResumePending(false),
      mComponentName("decoder") {
    mCodecLooper = new ALooper;
    mCodecLooper->setName("NPDecoder-CL");
    mCodecLooper->start(false, false, ANDROID_PRIORITY_AUDIO);
    mVideoTemporalLayerAggregateFps[0] = mFrameRateTotal;

    // mtk add:
    mInputReadBegin = 0ll;
#ifdef CONFIG_MT_ENG_BUILD
    mlog_enable = 1;
#else
    mlog_enable = __android_log_is_loggable(ANDROID_LOG_DEBUG, "ME6_Common", ANDROID_LOG_INFO);
#endif
}
```

​	此函数主要是创建Looper并且开始循环读取消息，然后将mVideoTemporalLayerAggregateFps[]这个数组（定义大小为32）的地1个值设置为mFrameRateTotal=kDefaultVideoFrameRateTotal=30.0f；

​	再看析构函数：

```c++
NuPlayer::Decoder::~Decoder() {
    // Need to stop looper first since mCodec could be accessed on the mDecoderLooper.
    stopLooper();
    if (mCodec != NULL) {
        mCodec->release();
    }
    releaseAndResetMediaBuffers();
}
```

停止Looper并且释放MediaCodec的资源，然后释放和重置MediaBuffers的资源。



### init()和configure()实现

init()函数是在DecoderBase.cpp类中实现的，并且实现比较简单，就是把Looper和Handler关联起来：

```c++
void NuPlayer::DecoderBase::init() {
    mDecoderLooper->registerHandler(this);
}
```

而configure函数的实现也是在DecoderBase.cpp中实现的：

```c++
void NuPlayer::DecoderBase::configure(const sp<AMessage> &format) {
    sp<AMessage> msg = new AMessage(kWhatConfigure, this);
    msg->setMessage("format", format);
    msg->post();
}

void NuPlayer::DecoderBase::onMessageReceived(const sp<AMessage> &msg) {
    switch (msg->what()) {
        case kWhatConfigure:
        {
            sp<AMessage> format;
            CHECK(msg->findMessage("format", &format));
            onConfigure(format);
            break;
        }
        ...
}
```

此方法用法发送消息，真正的实现是在NuplayerDecoder.cpp中的onConfigure方法：

```c++
void NuPlayer::Decoder::onConfigure(const sp<AMessage> &format) {
    CHECK(mCodec == NULL);

    mFormatChangePending = false;
    mTimeChangePending = false;

    ++mBufferGeneration;

    AString mime;
    CHECK(format->findString("mime", &mime)); //找到音视频的具体类型mime

    mIsAudio = !strncasecmp("audio/", mime.c_str(), 6);
    mIsVideoAVC = !strcasecmp(MEDIA_MIMETYPE_VIDEO_AVC, mime.c_str());

    mComponentName = mime;
    mComponentName.append(" decoder");
    ALOGV("[%s] onConfigure (surface=%p)", mComponentName.c_str(), mSurface.get());
	//根据mime的类型，创建MediaCodec解码器
    mCodec = MediaCodec::CreateByType(
            mCodecLooper, mime.c_str(), false /* encoder */, NULL /* err */, mPid, mUid);
    int32_t secure = 0;
    //如果传过来的信息中包含"secure"则执行
    if (format->findInt32("secure", &secure) && secure != 0) {
        if (mCodec != NULL) {
            mCodec->getName(&mComponentName);
            mComponentName.append(".secure");
            mCodec->release();
            ALOGI("[%s] creating", mComponentName.c_str());
            mCodec = MediaCodec::CreateByComponentName(
                    mCodecLooper, mComponentName.c_str(), NULL /* err */, mPid, mUid);
        }
    }
    if (mCodec == NULL) {
        ALOGE("Failed to create %s%s decoder",
                (secure ? "secure " : ""), mime.c_str());
        handleError(UNKNOWN_ERROR);//如果解码器没有创建成功的处理
        return;
    }
    mIsSecure = secure;

    mCodec->getName(&mComponentName);

    status_t err;
    if (mSurface != NULL) {
        // disconnect from surface as MediaCodec will reconnect
        //断开表面连接，因为MediaCodec会重新连接
        err = nativeWindowDisconnect(mSurface.get(), "onConfigure");
        // We treat this as a warning, as this is a preparatory step.
        // Codec will try to connect to the surface, which is where
        // any error signaling will occur.
        ALOGW_IF(err != OK, "failed to disconnect from surface: %d", err);
    }

    // Modular DRM
    void *pCrypto;
    if (!format->findPointer("crypto", &pCrypto)) {
        pCrypto = NULL;
    }
    sp<ICrypto> crypto = (ICrypto*)pCrypto;
    // non-encrypted source won't have a crypto
    mIsEncrypted = (crypto != NULL);
    // configure is called once; still using OR in case the behavior changes.
    mIsEncryptedObservedEarlier = mIsEncryptedObservedEarlier || mIsEncrypted;
    ALOGV("onConfigure mCrypto: %p (%d)  mIsSecure: %d",
            crypto.get(), (crypto != NULL ? crypto->getStrongCount() : 0), mIsSecure);

    err = mCodec->configure(
            format, mSurface, crypto, 0 /* flags */);

    if (err != OK) {
        ALOGE("Failed to configure [%s] decoder (err=%d)", mComponentName.c_str(), err);
        mCodec->release();
        mCodec.clear();
        handleError(err);
        return;
    }
    rememberCodecSpecificData(format);

    // the following should work in configured state
    CHECK_EQ((status_t)OK, mCodec->getOutputFormat(&mOutputFormat));
    CHECK_EQ((status_t)OK, mCodec->getInputFormat(&mInputFormat));

    mStats->setString("mime", mime.c_str());
    mStats->setString("component-name", mComponentName.c_str());

    if (!mIsAudio) {
        int32_t width, height;
        if (mOutputFormat->findInt32("width", &width)
                && mOutputFormat->findInt32("height", &height)) {
            mStats->setInt32("width", width);
            mStats->setInt32("height", height);
        }
    }

    sp<AMessage> reply = new AMessage(kWhatCodecNotify, this);
    mCodec->setCallback(reply);

    err = mCodec->start();
    if (err != OK) {
        ALOGE("Failed to start [%s] decoder (err=%d)", mComponentName.c_str(), err);
        mCodec->release();
        mCodec.clear();
        handleError(err);
        return;
    }

    releaseAndResetMediaBuffers();

    mPaused = false;
    mResumePending = false;
    if (!strcmp(mComponentName.c_str(), "OMX.MTK.AUDIO.DECODER.MP3")) {
        int mtkMp3Codec = 0;
        if (format->findInt32("mtkMp3Codec", &mtkMp3Codec)) {
            sp<MetaData> meta = new MetaData;
            if (meta != NULL) {
                meta->setInt32(kKeyMtkMP3Power, 1);
                mSource->setVendorMeta(true/*audio*/, meta);
            }
        }
    }
}
```

**此方法主要是，创建解码器，然后执行mCodec的configure方法**



### setParameters、setRenderer、setVideoSurface函数

setParameters实现相对简单，直接将参数传递给MediaCodec，其实现是在父类CoderBase中实现的。

```c++
void NuPlayer::DecoderBase::setParameters(const sp<AMessage> &params) {
    sp<AMessage> msg = new AMessage(kWhatSetParameters, this);
    msg->setMessage("params", params);
    msg->post();
}

case kWhatSetParameters:
        {
            sp<AMessage> params;
            CHECK(msg->findMessage("params", &params));
            onSetParameters(params);
            break;
        }


void NuPlayer::Decoder::onSetParameters(const sp<AMessage> &params) {
    bool needAdjustLayers = false;
    float frameRateTotal;
    if (params->findFloat("frame-rate-total", &frameRateTotal)
            && mFrameRateTotal != frameRateTotal) {
        needAdjustLayers = true;
        mFrameRateTotal = frameRateTotal;
    }

    int32_t numVideoTemporalLayerTotal;
    if (params->findInt32("temporal-layer-count", &numVideoTemporalLayerTotal)
            && numVideoTemporalLayerTotal >= 0
            && numVideoTemporalLayerTotal <= kMaxNumVideoTemporalLayers
            && mNumVideoTemporalLayerTotal != numVideoTemporalLayerTotal) {
        needAdjustLayers = true;
        mNumVideoTemporalLayerTotal = std::max(numVideoTemporalLayerTotal, 1);
    }

    if (needAdjustLayers && mNumVideoTemporalLayerTotal > 1) {
        // TODO: For now, layer fps is calculated for some specific architectures.
        // But it really should be extracted from the stream.
        mVideoTemporalLayerAggregateFps[0] =
            mFrameRateTotal / (float)(1ll << (mNumVideoTemporalLayerTotal - 1));
        for (int32_t i = 1; i < mNumVideoTemporalLayerTotal; ++i) {
            mVideoTemporalLayerAggregateFps[i] =
                mFrameRateTotal / (float)(1ll << (mNumVideoTemporalLayerTotal - i))
                + mVideoTemporalLayerAggregateFps[i - 1];
        }
    }

    float playbackSpeed;
    if (params->findFloat("playback-speed", &playbackSpeed)
            && mPlaybackSpeed != playbackSpeed) {
        needAdjustLayers = true;
        mPlaybackSpeed = playbackSpeed;
    }

    if (needAdjustLayers) {
        float decodeFrameRate = mFrameRateTotal;
        // enable temporal layering optimization only if we know the layering depth
        if (mNumVideoTemporalLayerTotal > 1) {
            int32_t layerId;
            for (layerId = 0; layerId < mNumVideoTemporalLayerTotal - 1; ++layerId) {
                if (mVideoTemporalLayerAggregateFps[layerId] * mPlaybackSpeed
                        >= kDisplayRefreshingRate * 0.9) {
                    break;
                }
            }
            mNumVideoTemporalLayerAllowed = layerId + 1;
            decodeFrameRate = mVideoTemporalLayerAggregateFps[layerId];
        }
        ALOGV("onSetParameters: allowed layers=%d, decodeFps=%g",
                mNumVideoTemporalLayerAllowed, decodeFrameRate);

        if (mCodec == NULL) {
            ALOGW("onSetParameters called before codec is created.");
            return;
        }

        sp<AMessage> codecParams = new AMessage();
        codecParams->setFloat("operating-rate", decodeFrameRate * mPlaybackSpeed);
        mCodec->setParameters(codecParams);
    }
}
```



setRenderer接口主要的目的是设置成员变量mRender。代码如下：

```c++
void NuPlayer::DecoderBase::setRenderer(const sp<Renderer> &renderer) {
    sp<AMessage> msg = new AMessage(kWhatSetRenderer, this);
    msg->setObject("renderer", renderer);
    msg->post();
}

case kWhatSetRenderer:
        {
            sp<RefBase> obj;
            CHECK(msg->findObject("renderer", &obj));
            onSetRenderer(static_cast<Renderer *>(obj.get()));
            break;
        }

void NuPlayer::Decoder::onSetRenderer(const sp<Renderer> &renderer) {
    mRenderer = renderer;
}

```



setVideoSurface函数实现如下:

```c++
status_t NuPlayer::Decoder::setVideoSurface(const sp<Surface> &surface) {
    if (surface == NULL || ADebug::isExperimentEnabled("legacy-setsurface")) {
        return BAD_VALUE;
    }

    sp<AMessage> msg = new AMessage(kWhatSetVideoSurface, this);

    msg->setObject("surface", surface);
    sp<AMessage> response;
    status_t err = msg->postAndAwaitResponse(&response);
    if (err == OK && response != NULL) {
        CHECK(response->findInt32("err", &err));
    }
    return err;
}

//消息响应的地方
case kWhatSetVideoSurface:
        {
            sp<AReplyToken> replyID;
            CHECK(msg->senderAwaitsResponse(&replyID));

            sp<RefBase> obj;
            CHECK(msg->findObject("surface", &obj));
            sp<Surface> surface = static_cast<Surface *>(obj.get()); // non-null
            int32_t err = INVALID_OPERATION;
            // NOTE: in practice mSurface is always non-null, but checking here for completeness
            if (mCodec != NULL && mSurface != NULL) {
                // TODO: once AwesomePlayer is removed, remove this automatic connecting
                // to the surface by MediaPlayerService.
                //
                // at this point MediaPlayerService::client has already connected to the
                // surface, which MediaCodec does not expect
                err = nativeWindowDisconnect(surface.get(), "kWhatSetVideoSurface(surface)");
                if (err == OK) {
                    err = mCodec->setSurface(surface);
                    ALOGI_IF(err, "codec setSurface returned: %d", err);
                    if (err == OK) {
                        // reconnect to the old surface as MPS::Client will expect to
                        // be able to disconnect from it.
                        (void)nativeWindowConnect(mSurface.get(), "kWhatSetVideoSurface(mSurface)");
                        mSurface = surface;
                    }
                }
                if (err != OK) {
                    // reconnect to the new surface on error as MPS::Client will expect to
                    // be able to disconnect from it.
                    (void)nativeWindowConnect(surface.get(), "kWhatSetVideoSurface(err)");
                }
            }

            sp<AMessage> response = new AMessage;
            response->setInt32("err", err);
            response->postReply(replyID);
            break;
        }
```

这个方法的主要作用是发送kWhatSetVideoSurface消息，然后等待返回值response。





### signalFlush、signalResume、initiateShutdown函数

signalFlush实现代码如下，主要调用Renderer和MediaCodec的flush接口：

```c++
void NuPlayer::DecoderBase::signalFlush() {
    (new AMessage(kWhatFlush, this))->post();
}

//DecoderBase.cpp 中的onMessageReceived方法
case kWhatFlush:
        {
            onFlush();
            break;
        }
        

void NuPlayer::Decoder::onFlush() {
    doFlush(true);

    if (isDiscontinuityPending()) {
        // This could happen if the client starts seeking/shutdown
        // after we queued an EOS for discontinuities.
        // We can consider discontinuity handled.
        finishHandleDiscontinuity(false /* flushOnTimeChange */);
    }

    sp<AMessage> notify = mNotify->dup();
    notify->setInt32("what", kWhatFlushCompleted);
    notify->post();
}

void NuPlayer::Decoder::doFlush(bool notifyComplete) {
    if (mCCDecoder != NULL) {
        mCCDecoder->flush();
    }

    if (mRenderer != NULL) {
        mRenderer->flush(mIsAudio, notifyComplete);
        mRenderer->signalTimeDiscontinuity();
    }

    status_t err = OK;
    if (mCodec != NULL) {
        err = mCodec->flush();
        mCSDsToSubmit = mCSDsForCurrentFormat; // copy operator
        ++mBufferGeneration;
    }

    if (err != OK) {
        ALOGE("failed to flush [%s] (err=%d)", mComponentName.c_str(), err);
        handleError(err);
        // finish with posting kWhatFlushCompleted.
        // we attempt to release the buffers even if flush fails.
    }
    releaseAndResetMediaBuffers();
    mPaused = true;
}
```



signalResume相对简单，直接调用MediaCodec接口，代码如下：

```c++
void NuPlayer::DecoderBase::signalResume(bool notifyComplete) {
    sp<AMessage> msg = new AMessage(kWhatResume, this);
    msg->setInt32("notifyComplete", notifyComplete);
    msg->post();
}

//DecoderBase.cpp 中的onMessageReceived方法
case kWhatResume:
        {
            int32_t notifyComplete;
            CHECK(msg->findInt32("notifyComplete", &notifyComplete));

            onResume(notifyComplete);
            break;
        }

void NuPlayer::Decoder::onResume(bool notifyComplete) {
    mPaused = false;

    if (notifyComplete) {
        mResumePending = true;
    }

    if (mCodec == NULL) {
        ALOGE("[%s] onResume without a valid codec", mComponentName.c_str());
        handleError(NO_INIT);
        return;
    }
    mCodec->start();//主要就是调用这个mCodec解码器的start（）方法
}

```



initiateShutdown主要是关闭解码器，实现如下：

```c++
void NuPlayer::DecoderBase::initiateShutdown() {
    (new AMessage(kWhatShutdown, this))->post();
}

//DecoderBase.cpp 中的onMessageReceived方法
case kWhatShutdown:
        {
            onShutdown(true);
            break;
        }


void NuPlayer::Decoder::onShutdown(bool notifyComplete) {
    status_t err = OK;

    // if there is a pending resume request, notify complete now
    notifyResumeCompleteIfNecessary();

    if (mCodec != NULL) {
        err = mCodec->release();//释放资源
        mCodec = NULL;
        ++mBufferGeneration;

        if (mSurface != NULL) {
            // reconnect to surface as MediaCodec disconnected from it
            status_t error = nativeWindowConnect(mSurface.get(), "onShutdown");
            ALOGW_IF(error != NO_ERROR,
                    "[%s] failed to connect to native window, error=%d",
                    mComponentName.c_str(), error);
        }
        mComponentName = "decoder";
    }

    releaseAndResetMediaBuffers();

    if (err != OK) {
        ALOGE("failed to release [%s] (err=%d)", mComponentName.c_str(), err);
        handleError(err);
        // finish with posting kWhatShutdownCompleted.
    }
	//根据情况发送kWhatShutdownCompleted消息。
    if (notifyComplete) {
        sp<AMessage> notify = mNotify->dup();
        notify->setInt32("what", kWhatShutdownCompleted);
        notify->post();
        mPaused = true;
    }
}
```



### getStats函数

getStats实现如下，获取解码帧数、输入输出的丢帧数目等：

```c++
sp<AMessage> NuPlayer::Decoder::getStats() const {
    mStats->setInt64("frames-total", mNumFramesTotal);
    mStats->setInt64("frames-dropped-input", mNumInputFramesDropped);
    mStats->setInt64("frames-dropped-output", mNumOutputFramesDropped);
    return mStats;
}
```





## Decoder解码流程分析

​	前面是以接口为界进行代码功能分析，实际上最为主要的解码流程并没在这里。实际解码过程中，无外乎获取输入的压缩数据，MediaCodec解码，返回解码之后的数据，渲染。在onConfigure函数实现时有下面代码：

```c++
mCodec->setCallback(reply);
```

​	这里就是将MediaCodec的消息发送给当前类Decoder消息队列中，然后在onMessageReceived中处理。实际的解码开始是从setRenderer开始，循环解码的逻辑来在于onRequestInputBuffers函数，其中会调用doRequestBuffers，其实现如下：

```c++
bool NuPlayer::Decoder::doRequestBuffers() {
    if (isDiscontinuityPending()) {
        return false;
    }
    status_t err = OK;
    while (err == OK && !mDequeuedInputBuffers.empty()) {
        size_t bufferIx = *mDequeuedInputBuffers.begin();
        sp<AMessage> msg = new AMessage();
        msg->setSize("buffer-ix", bufferIx);
        err = fetchInputData(msg);          // 取一个输入缓冲
        if (err != OK && err != ERROR_END_OF_STREAM) {
            // if EOS, need to queue EOS buffer
            break;
        }
        mDequeuedInputBuffers.erase(mDequeuedInputBuffers.begin());

        if (!mPendingInputMessages.empty()
                || !onInputBufferFetched(msg)) {
            mPendingInputMessages.push_back(msg);// 实际取出的数据放到这个缓冲消息队列中
        }
    }

    return err == -EWOULDBLOCK
            && mSource->feedMoreTSData() == OK;
}
```

​	如果doRequestBuffers返回true的话，kWhatRequestInputBuffers会循环发送kWhatRequestInputBuffers消息，驱动正常解码逻辑。

​	下面是关于MediaCodec返回消息的处理逻辑，代码主要集中在onMessageReceived中，如下：

```c++
void NuPlayer::Decoder::onMessageReceived(const sp<AMessage> &msg) {
    ALOGV("[%s] onMessage: %s", mComponentName.c_str(), msg->debugString().c_str());

    switch (msg->what()) {
        case kWhatCodecNotify:
        {
            int32_t cbID;
            CHECK(msg->findInt32("callbackID", &cbID));

            ALOGV("[%s] kWhatCodecNotify: cbID = %d, paused = %d",
                    mIsAudio ? "audio" : "video", cbID, mPaused);

            if (mPaused) {
                break;
            }

            switch (cbID) {
                case MediaCodec::CB_INPUT_AVAILABLE: //可以填充数据
                {
                    int32_t index;
                    CHECK(msg->findInt32("index", &index));

                    mInputReadBegin = ALooper::GetNowUs();
                    handleAnInputBuffer(index);
                    break;
                }

                case MediaCodec::CB_OUTPUT_AVAILABLE://解码成功的消息，输出数据在这里
                {
                    int32_t index;
                    size_t offset;
                    size_t size;
                    int64_t timeUs;
                    int32_t flags;

                    CHECK(msg->findInt32("index", &index));
                    CHECK(msg->findSize("offset", &offset));
                    CHECK(msg->findSize("size", &size));
                    CHECK(msg->findInt64("timeUs", &timeUs));
                    CHECK(msg->findInt32("flags", &flags));

                    handleAnOutputBuffer(index, offset, size, timeUs, flags);
                    break;
                }

                case MediaCodec::CB_OUTPUT_FORMAT_CHANGED://通知输出格式改变
                {
                    sp<AMessage> format;
                    CHECK(msg->findMessage("format", &format));

                    handleOutputFormatChange(format);
                    break;
                }

                case MediaCodec::CB_ERROR://发生未知错误，需要处理
                {
                    status_t err;
                    CHECK(msg->findInt32("err", &err));
                    ALOGE("Decoder (%s) reported error : 0x%x",
                            mIsAudio ? "audio" : "video", err);

                    handleError(err);
                    break;
                }

                default://其他消息不做处理
                {
                    TRESPASS();
                    break;
                }
            }

            break;
        }
	 ...
}
```

先看如何向MediaCodec输入数据。代码如下：

```c++
bool NuPlayer::Decoder::handleAnInputBuffer(size_t index) {
    if (isDiscontinuityPending()) {
        return false;
    }

    if (mCodec == NULL) {
        ALOGE("[%s] handleAnInputBuffer without a valid codec", mComponentName.c_str());
        handleError(NO_INIT);
        return false;
    }

    sp<MediaCodecBuffer> buffer;
    mCodec->getInputBuffer(index, &buffer);

    if (buffer == NULL) {
        ALOGE("[%s] handleAnInputBuffer, failed to get input buffer", mComponentName.c_str());
        handleError(UNKNOWN_ERROR);
        return false;
    }

    if (index >= mInputBuffers.size()) {
        for (size_t i = mInputBuffers.size(); i <= index; ++i) {
            mInputBuffers.add();
            mMediaBuffers.add();
            mInputBufferIsDequeued.add();
            mMediaBuffers.editItemAt(i) = NULL;
            mInputBufferIsDequeued.editItemAt(i) = false;
        }
    }
    mInputBuffers.editItemAt(index) = buffer;

    //CHECK_LT(bufferIx, mInputBuffers.size());

    if (mMediaBuffers[index] != NULL) {
        mMediaBuffers[index]->release();
        mMediaBuffers.editItemAt(index) = NULL;
    }
    mInputBufferIsDequeued.editItemAt(index) = true;
    // mtk change: vorbis decoder not support csd resubmit
    if (!mCSDsToSubmit.isEmpty()
            && strcmp("OMX.google.vorbis.decoder", mComponentName.c_str())) {
        sp<AMessage> msg = new AMessage();
        msg->setSize("buffer-ix", index);

        sp<ABuffer> buffer = mCSDsToSubmit.itemAt(0);
        ALOGI("[%s] resubmitting CSD", mComponentName.c_str());
        msg->setBuffer("buffer", buffer);
        mCSDsToSubmit.removeAt(0);
        if (!onInputBufferFetched(msg)) {
            handleError(UNKNOWN_ERROR);
            return false;
        }
        return true;
    }

    while (!mPendingInputMessages.empty()) {
        sp<AMessage> msg = *mPendingInputMessages.begin();
        if (!onInputBufferFetched(msg)) {
            break;
        }
        mPendingInputMessages.erase(mPendingInputMessages.begin());
    }

    if (!mInputBufferIsDequeued.editItemAt(index)) {
        return true;
    }

    mDequeuedInputBuffers.push_back(index);

    onRequestInputBuffers();
    return true;
}
```

那么看看onInputBufferFetched的实现，这里会操作MediaCodec：

```c++
bool NuPlayer::Decoder::onInputBufferFetched(const sp<AMessage> &msg) {
    if (mCodec == NULL) {
        ALOGE("[%s] onInputBufferFetched without a valid codec", mComponentName.c_str());
        handleError(NO_INIT);
        return false;
    }

    size_t bufferIx;
    CHECK(msg->findSize("buffer-ix", &bufferIx));
    CHECK_LT(bufferIx, mInputBuffers.size());
    sp<MediaCodecBuffer> codecBuffer = mInputBuffers[bufferIx];

    sp<ABuffer> buffer;
    bool hasBuffer = msg->findBuffer("buffer", &buffer);
    bool needsCopy = true;

    if (buffer == NULL /* includes !hasBuffer */) {
        int32_t streamErr = ERROR_END_OF_STREAM;
        CHECK(msg->findInt32("err", &streamErr) || !hasBuffer);

        CHECK(streamErr != OK);

        // attempt to queue EOS
        status_t err = mCodec->queueInputBuffer(
                bufferIx,
                0,
                0,
                0,
                MediaCodec::BUFFER_FLAG_EOS);
        if (err == OK) {
            mInputBufferIsDequeued.editItemAt(bufferIx) = false;
        } else if (streamErr == ERROR_END_OF_STREAM) {
            streamErr = err;
            // err will not be ERROR_END_OF_STREAM
        }

        if (streamErr != ERROR_END_OF_STREAM) {
            ALOGE("Stream error for [%s] (err=%d), EOS %s queued",
                    mComponentName.c_str(),
                    streamErr,
                    err == OK ? "successfully" : "unsuccessfully");
            handleError(streamErr);
        }
    } else {
        sp<AMessage> extra;
        if (buffer->meta()->findMessage("extra", &extra) && extra != NULL) {
            int64_t resumeAtMediaTimeUs;
            if (extra->findInt64(
                        "resume-at-mediaTimeUs", &resumeAtMediaTimeUs)) {
                ALOGI("[%s] suppressing rendering until %lld us",
                        mComponentName.c_str(), (long long)resumeAtMediaTimeUs);
                mSkipRenderingUntilMediaTimeUs = resumeAtMediaTimeUs;
            }
            //mtk add seekmode for ALPS03567323 AND ALPS03607769
            int64_t seekTimeUsForDecoder = 0;
            if (extra->findInt64(
                        "decode-seekTime", &seekTimeUsForDecoder)) {
                if (!mIsAudio && mCodec != NULL) {
                    sp<AMessage> msg = new AMessage;
                    msg->setInt64("seekTimeUs", seekTimeUsForDecoder);
                    mCodec->setParameters(msg);
                    ALOGI("set video decode seek time:%lld", (long long)seekTimeUsForDecoder);
                }
            }
#ifdef MTK_AUDIO_APE_SUPPORT
            int32_t newframe =0; //for ape seek
            int32_t firstbyte =0;
            if (extra->findInt32("nwfrm", &newframe))
            {
                ALOGI("APE nwfrm found :%d line:%d",(int)newframe,__LINE__);
            }
            if (extra->findInt32("sekbyte", &firstbyte))
            {
                ALOGI("APE sekbyte found :%d line:%d",(int)firstbyte,__LINE__);
            }
            if (newframe != 0 || firstbyte !=0)
            {
                sp<AMessage> msg = new AMessage;
                msg->setInt32("nwfrm", newframe);
                msg->setInt32("sekbyte", firstbyte);
                mCodec->setParameters(msg);
            }
#endif
        }

        int64_t timeUs = 0;
        uint32_t flags = 0;
        CHECK(buffer->meta()->findInt64("timeUs", &timeUs));

        int32_t eos, csd;
        // we do not expect SYNCFRAME for decoder
        if (buffer->meta()->findInt32("eos", &eos) && eos) {
            flags |= MediaCodec::BUFFER_FLAG_EOS;
        } else if (buffer->meta()->findInt32("csd", &csd) && csd) {
            flags |= MediaCodec::BUFFER_FLAG_CODECCONFIG;
        }

        // Modular DRM
        MediaBuffer *mediaBuf = NULL;
        NuPlayerDrm::CryptoInfo *cryptInfo = NULL;

        // copy into codec buffer
        if (needsCopy) {
            if (buffer->size() > codecBuffer->capacity()) {
                handleError(ERROR_BUFFER_TOO_SMALL);
                mDequeuedInputBuffers.push_back(bufferIx);
                return false;
            }

            if (buffer->data() != NULL) {
                codecBuffer->setRange(0, buffer->size());
                memcpy(codecBuffer->data(), buffer->data(), buffer->size());
            } else { // No buffer->data()
                //Modular DRM
                mediaBuf = (MediaBuffer*)buffer->getMediaBufferBase();
                if (mediaBuf != NULL) {
                    codecBuffer->setRange(0, mediaBuf->size());
                    memcpy(codecBuffer->data(), mediaBuf->data(), mediaBuf->size());

                    sp<MetaData> meta_data = mediaBuf->meta_data();
                    cryptInfo = NuPlayerDrm::getSampleCryptoInfo(meta_data);

                    // since getMediaBuffer() has incremented the refCount
                    mediaBuf->release();
                } else { // No mediaBuf
                    ALOGE("onInputBufferFetched: buffer->data()/mediaBuf are NULL for %p",
                            buffer.get());
                    handleError(UNKNOWN_ERROR);
                    return false;
                }
            } // buffer->data()
        } // needsCopy

        status_t err;
        AString errorDetailMsg;
        if (cryptInfo != NULL) {
            err = mCodec->queueSecureInputBuffer(
                    bufferIx,
                    codecBuffer->offset(),
                    cryptInfo->subSamples,
                    cryptInfo->numSubSamples,
                    cryptInfo->key,
                    cryptInfo->iv,
                    cryptInfo->mode,
                    cryptInfo->pattern,
                    timeUs,
                    flags,
                    &errorDetailMsg);
            // synchronous call so done with cryptInfo here
            free(cryptInfo);
        } else {
            if (mlog_enable) {
                int64_t totalTime = ALooper::GetNowUs() - mInputReadBegin;
                ALOGD("onInputBufferFetched(%s, %lld) cost %lld ms",
                        mIsAudio ? "audio" : "video", (long long)timeUs, (long long)totalTime / 1000ll);
            }
            err = mCodec->queueInputBuffer(
                    bufferIx,
                    codecBuffer->offset(),
                    codecBuffer->size(),
                    timeUs,
                    flags,
                    &errorDetailMsg);
        } // no cryptInfo

        if (err != OK) {
            ALOGE("onInputBufferFetched: queue%sInputBuffer failed for [%s] (err=%d, %s)",
                    (cryptInfo != NULL ? "Secure" : ""),
                    mComponentName.c_str(), err, errorDetailMsg.c_str());
            handleError(err);
        } else {
            mInputBufferIsDequeued.editItemAt(bufferIx) = false;
        }

    }   // buffer != NULL
    return true;
}
```

这样解码前的数据准备就完成了。解码后的数据处理函数是handleAnOutputBuffer，响应MediaCodec::CB_OUTPUT_AVAILABLE消息，代码如下：

```c++
bool NuPlayer::Decoder::handleAnOutputBuffer(
        size_t index,
        size_t offset,
        size_t size,
        int64_t timeUs,
        int32_t flags) {
    if (mCodec == NULL) {
        ALOGE("[%s] handleAnOutputBuffer without a valid codec", mComponentName.c_str());
        handleError(NO_INIT);
        return false;
    }

//    CHECK_LT(bufferIx, mOutputBuffers.size());
    sp<MediaCodecBuffer> buffer;
    mCodec->getOutputBuffer(index, &buffer);

    if (buffer == NULL) {
        ALOGE("[%s] handleAnOutputBuffer, failed to get output buffer", mComponentName.c_str());
        handleError(UNKNOWN_ERROR);
        return false;
    }

    if (index >= mOutputBuffers.size()) {
        for (size_t i = mOutputBuffers.size(); i <= index; ++i) {
            mOutputBuffers.add();
        }
    }

    mOutputBuffers.editItemAt(index) = buffer;

    buffer->setRange(offset, size);
    buffer->meta()->clear();
    buffer->meta()->setInt64("timeUs", timeUs);

    bool eos = flags & MediaCodec::BUFFER_FLAG_EOS;
    // we do not expect CODECCONFIG or SYNCFRAME for decoder

    sp<AMessage> reply = new AMessage(kWhatRenderBuffer, this);
    reply->setSize("buffer-ix", index);
    reply->setInt32("generation", mBufferGeneration);

    if (eos) {
        ALOGI("[%s] saw output EOS", mIsAudio ? "audio" : "video");

        buffer->meta()->setInt32("eos", true);
        reply->setInt32("eos", true);
    } else if (mSkipRenderingUntilMediaTimeUs >= 0) {
        if (timeUs < mSkipRenderingUntilMediaTimeUs) {
            ALOGV("[%s] dropping buffer at time %lld as requested.",
                     mComponentName.c_str(), (long long)timeUs);

            reply->post();
            return true;
        }

        mSkipRenderingUntilMediaTimeUs = -1;
    }

    mNumFramesTotal += !mIsAudio;

    // wait until 1st frame comes out to signal resume complete
    notifyResumeCompleteIfNecessary();

    if (mRenderer != NULL) {
        // send the buffer to renderer.
        mRenderer->queueBuffer(mIsAudio, buffer, reply);
        if (mlog_enable) {
            ALOGD("[%s] queueBuffer(%s, %lld)",
                    mComponentName.c_str(), mIsAudio ? "audio" : "video", (long long)timeUs);
        }
        if (eos && !isDiscontinuityPending()) {
            mRenderer->queueEOS(mIsAudio, ERROR_END_OF_STREAM);
        }
    }

    return true;
}
```

还有最后一个消息MediaCodec::CB_OUTPUT_AVAILABLE，处理函数是handleOutputFormatChange，代码如下：

```c++
void NuPlayer::Decoder::handleOutputFormatChange(const sp<AMessage> &format) {
    if (!mIsAudio) {
        int32_t width, height;
        if (format->findInt32("width", &width)
                && format->findInt32("height", &height)) {
            mStats->setInt32("width", width);
            mStats->setInt32("height", height);
        }
        sp<AMessage> notify = mNotify->dup();
        notify->setInt32("what", kWhatVideoSizeChanged);
        notify->setMessage("format", format);
        notify->post();
    } else if (mRenderer != NULL) {
        uint32_t flags;
        int64_t durationUs;
        bool hasVideo = (mSource->getFormat(false /* audio */) != NULL);
        if (getAudioDeepBufferSetting() // override regardless of source duration
                || (mSource->getDuration(&durationUs) == OK
                        && durationUs > AUDIO_SINK_MIN_DEEP_BUFFER_DURATION_US)) {
            flags = AUDIO_OUTPUT_FLAG_DEEP_BUFFER;
            ALOGD("handleOutputFormatChange set flag AUDIO_OUTPUT_FLAG_DEEP_BUFFER");
        } else {
            flags = AUDIO_OUTPUT_FLAG_NONE;
        }

        sp<AMessage> reply = new AMessage(kWhatAudioOutputFormatChanged, this);
        reply->setInt32("generation", mBufferGeneration);
        mRenderer->changeAudioFormat(
                format, false /* offloadOnly */, hasVideo,
                flags, mSource->isStreaming(), reply);
    }
}

```

还剩最后一部分，显示是如何处理的。那就是handleAnOutputBuffer发送的一个kWhatRenderBuffer消息的处理，代码如下：

```c++
case kWhatRenderBuffer:
        {
            if (!isStaleReply(msg)) {
                onRenderBuffer(msg);
            }
            break;
        }
        
        
void NuPlayer::Decoder::onRenderBuffer(const sp<AMessage> &msg) {
    status_t err;
    int32_t render;
    size_t bufferIx;
    int32_t eos;
    CHECK(msg->findSize("buffer-ix", &bufferIx));

    if (!mIsAudio) {
        int64_t timeUs;
        sp<MediaCodecBuffer> buffer = mOutputBuffers[bufferIx];
        buffer->meta()->findInt64("timeUs", &timeUs);

        if (mCCDecoder != NULL && mCCDecoder->isSelected()) {
            mCCDecoder->display(timeUs);
        }
    }

    if (mCodec == NULL) {
        err = NO_INIT;
    } else if (msg->findInt32("render", &render) && render) {
        int64_t timestampNs;
        CHECK(msg->findInt64("timestampNs", &timestampNs));
        err = mCodec->renderOutputBufferAndRelease(bufferIx, timestampNs);
    } else {
        mNumOutputFramesDropped += !mIsAudio;
        err = mCodec->releaseOutputBuffer(bufferIx);
    }
    if (err != OK) {
        ALOGE("failed to release output buffer for [%s] (err=%d)",
                mComponentName.c_str(), err);
        handleError(err);
    }
    if (msg->findInt32("eos", &eos) && eos
            && isDiscontinuityPending()) {
        finishHandleDiscontinuity(true /* flushOnTimeChange */);
    }
}        
```





























































