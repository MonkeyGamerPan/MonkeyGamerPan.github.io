---
Clayout: post
title: android Nuplayer框架GenericSource类源码分析
featured-img: sleek
mathjax: true
categories: [andriod,NuplayerFrame,GenericSourceCode]
---

### GenericSource.cpp简介

​	GenericSource时候Nuplayer::Source的一个子类，主要负责的功能就是负责本地多媒体文件的读取和解析，功能类似FFmpeg的libvaformat。

​	通常的GenericSource有一下功能：	

- 多媒体格式探测
- 多媒体格式解析



### GenericSource父类Nuplayer::Source

其父类源码位置：frameworks/av/media/libmediaplayerservice/nuplayer/NuPlayerSource.h

```c++
struct NuPlayer::Source : public AHandler {
    enum Flags {
        FLAG_CAN_PAUSE          = 1,
        FLAG_CAN_SEEK_BACKWARD  = 2,  // the "10 sec back button"
        FLAG_CAN_SEEK_FORWARD   = 4,  // the "10 sec forward button"
        FLAG_CAN_SEEK           = 8,  // the "seek bar"
        FLAG_DYNAMIC_DURATION   = 16,
        FLAG_SECURE             = 32, // Secure codec is required.
        FLAG_PROTECTED          = 64, // The screen needs to be protected (screenshot is disabled).
    };

    enum {
        kWhatPrepared,
        kWhatFlagsChanged,
        kWhatVideoSizeChanged,
        kWhatBufferingUpdate,
        kWhatPauseOnBufferingStart,
        kWhatResumeOnBufferingEnd,
        kWhatCacheStats,
        kWhatSubtitleData,
        kWhatTimedTextData,
        kWhatTimedMetaData,
        kWhatQueueDecoderShutdown,
        kWhatDrmNoLicense,
        kWhatInstantiateSecureDecoders,
        // Modular DRM
        kWhatDrmInfo,
    };

    // The provides message is used to notify the player about various
    // events.提供的消息用于通知玩家各种事件。
    explicit Source(const sp<AMessage> &notify)
        : mNotify(notify) {
    }

    virtual status_t getDefaultBufferingSettings(
            BufferingSettings* buffering /* nonnull */) = 0;
    virtual status_t setBufferingSettings(const BufferingSettings& buffering) = 0;

    virtual void prepareAsync() = 0; //异步准备资源方法

    virtual void start() = 0;//开始播放方法
    virtual void stop() {}//停止播放方法
    virtual void pause() {}//暂停播放方法
    virtual void resume() {}//重新开始播放方法

    // Explicitly disconnect the underling data source显式断开下属数据源
    virtual void disconnect() {}

    // Returns OK iff more data was available,
    // an error or ERROR_END_OF_STREAM if not.
    virtual status_t feedMoreTSData() = 0;

    // Returns non-NULL format when the specified track exists.
    // When the format has "err" set to -EWOULDBLOCK, source needs more time to get valid meta data.这三个方法用于获取格式
    // Returns NULL if the specified track doesn't exist or is invalid;
    virtual sp<AMessage> getFormat(bool audio);
    virtual sp<MetaData> getFormatMeta(bool /* audio */) { return NULL; }
    virtual sp<MetaData> getFileFormatMeta() const { return NULL; }
	
    
    virtual status_t dequeueAccessUnit(
            bool audio, sp<ABuffer> *accessUnit) = 0;

    virtual status_t getDuration(int64_t * /* durationUs */) {
        return INVALID_OPERATION;
    }

    
    //获取解析之后的节目的数目和信息，这里将每一个节目称之为一个Track
    //size_t 为unsigned int
    virtual size_t getTrackCount() const { 
        return 0;
    }
    virtual sp<AMessage> getTrackInfo(size_t /* trackIndex */) const {
        return NULL;
    }
    //猜测这里的ssize_t也是int类型
    virtual ssize_t getSelectedTrack(media_track_type /* type */) const {
        return INVALID_OPERATION;
    }

    virtual status_t selectTrack(size_t /* trackIndex */, bool /* select */, int64_t /* timeUs*/) {
        return INVALID_OPERATION;
    }

    //seek操作的主要执行函数
    virtual status_t seekTo(
            int64_t /* seekTimeUs */,
            MediaPlayerSeekMode /* mode */ = MediaPlayerSeekMode::SEEK_PREVIOUS_SYNC) {
        return INVALID_OPERATION;
    }

    virtual status_t setBuffers(bool /* audio */, Vector<MediaBuffer *> &/* buffers */) {
        return INVALID_OPERATION;
    }

    virtual bool isRealTime() const {
        return false;
    }

    virtual bool isStreaming() const {
        return true;
    }

    virtual void setOffloadAudio(bool /* offload */) {}

    // Modular DRM
    virtual status_t prepareDrm(
            const uint8_t /*uuid*/[16], const Vector<uint8_t> &/*drmSessionId*/,
            sp<ICrypto> */*crypto*/) {
        return INVALID_OPERATION;
    }

    virtual status_t releaseDrm() {
        return INVALID_OPERATION;
    }

    virtual status_t setVendorMeta(bool /*audio*/, const sp<MetaData> & /*meta*/)
    {
        return INVALID_OPERATION;
    }

protected:
    virtual ~Source() {}

    virtual void onMessageReceived(const sp<AMessage> &msg);

    sp<AMessage> dupNotify() const { return mNotify->dup(); }

    void notifyFlagsChanged(uint32_t flags);
    void notifyVideoSizeChanged(const sp<AMessage> &format = NULL);
    void notifyInstantiateSecureDecoders(const sp<AMessage> &reply);
    void notifyPrepared(status_t err = OK);
    // Modular DRM
    void notifyDrmInfo(const sp<ABuffer> &buffer);

private:
    sp<AMessage> mNotify;

    DISALLOW_EVIL_CONSTRUCTORS(Source);
//mtkadd+
public:
    virtual void setParams(const sp<MetaData> &) {};
};
```

从层次结构来看，Nuplayer::Source的子类有：GenericSource、HTTPLiveSource、RTSPSource、StreamingSource。

- 构建部分接口——构造/析构函数/setDataSource
- 基本播放控制接口——prepareAsync/stop/start/pause/resume/seekTo/disconnect
- Track信息相关——getTrackCount/getTrackInfo/getSelectedTrack/selectTrack/getFormat
- 辅助信息获取及设置——getDuration/isRealTime/getFormatMeta/isStreaming/setOffloadAudio/setBuffers/setBuffers/feedMoreTSData

### GenericSource类的对外接口以及主要成员

源码路径：frameworks/av/media/libmediaplayerservice/nuplayer/GenericSource.h

```c++
// 注意这里有部分代码删减，并不是全部
struct NuPlayer::GenericSource : public NuPlayer::Source {
    GenericSource(const sp<AMessage> &notify, bool uidValid, uid_t uid);

    status_t setDataSource(const sp<IMediaHTTPService> &httpService,
            const char *url, const KeyedVector<String8, String8> *headers);
    status_t setDataSource(int fd, int64_t offset, int64_t length);
    status_t setDataSource(const sp<DataSource>& dataSource);
private:
	struct Track {
        size_t mIndex;
        sp<IMediaSource> mSource;
        sp<AnotherPacketSource> mPackets;
    };
	// Helper to monitor buffering status. The polling happens every second.
    // When necessary, it will send out buffering events to the player.
    //帮助监视缓冲状态。轮询每秒钟进行一次。必要时，它会向用户发送缓冲事件。
    struct BufferingMonitor : public AHandler { ... };

	Vector<sp<IMediaSource> > mSources;
    Track mAudioTrack; // 音频流
    int64_t mAudioTimeUs;
    int64_t mAudioLastDequeueTimeUs;
    Track mVideoTrack; // 视频流
    int64_t mVideoTimeUs;
    int64_t mVideoLastDequeueTimeUs;
    Track mSubtitleTrack; // 字幕流
    Track mTimedTextTrack;

    sp<IMediaHTTPService> mHTTPService;
    AString mUri;
    KeyedVector<String8, String8> mUriHeaders;
    int mFd;
    int64_t mOffset;
    int64_t mLength;

    sp<DataSource> mDataSource;
    sp<NuCachedSource2> mCachedSource;
    sp<DataSource> mHttpSource;
    sp<WVMExtractor> mWVMExtractor;
    sp<MetaData> mFileMeta;
    DrmManagerClient *mDrmManagerClient;
    sp<DecryptHandle> mDecryptHandle;
    bool mStarted;
    bool mStopRead;
    int64_t mBitrate;
    sp<BufferingMonitor> mBufferingMonitor;
    uint32_t mPendingReadBufferTypes;
    sp<ABuffer> mGlobalTimedText;

    sp<ALooper> mLooper;
    sp<ALooper> mBufferingMonitorLooper;
};
```

从这里可以看出其中包含了内部类BufferingMonitor，继承自AHandler。并且GenericSource添加了setDataSource接口，并且包含了多个Track和各种DataSource/NuCachedSource2。



### 构造函数

GenericSource的构造函数相对简单，源码如下：

```c++
NuPlayer::GenericSource::GenericSource(
        const sp<AMessage> &notify,
        bool uidValid,
        uid_t uid)
    : Source(notify),
      mAudioTimeUs(0),
      mAudioLastDequeueTimeUs(0),
      mVideoTimeUs(0),
      mVideoLastDequeueTimeUs(0),
      mFetchSubtitleDataGeneration(0),
      mFetchTimedTextDataGeneration(0),
      mDurationUs(-1ll),
      mAudioIsVorbis(false),
      mIsSecure(false),
      mIsStreaming(false),
      mUIDValid(uidValid),
      mUID(uid),
      mFd(-1),
      mBitrate(-1ll),
      mPendingReadBufferTypes(0) {
    ALOGV("GenericSource");

    mBufferingMonitor = new BufferingMonitor(notify);
    resetDataSource();
//  M Add for support OMA DRM playback
    mDecryptHandle = NULL;
    mDrmManagerClient = NULL;
    mIsCurrentComplete = false;

    //init extra variant
    init();
}


void NuPlayer::GenericSource::resetDataSource() {
    ALOGV("resetDataSource");
    mHTTPService.clear();//DataSource类型
    mHttpSource.clear();//DataSource类型
    mUri.clear();//AString类型
    mUriHeaders.clear();//KeyedVector<String8,String8>类型的一个容器，感觉和list差不多。
    if (mFd >= 0) {
        close(mFd);
        mFd = -1; //int 类型
    }
    mOffset = 0;
    mLength = 0;
    mStarted = false;
    mStopRead = true;

    if (mBufferingMonitorLooper != NULL) {  //ALooper类型
        mBufferingMonitorLooper->unregisterHandler(mBufferingMonitor->id());
        mBufferingMonitorLooper->stop();
        mBufferingMonitorLooper = NULL;
    }
    mBufferingMonitor->stop(); //BufferingMonitor类型 内部类实现的

    mIsDrmProtected = false;
    mIsDrmReleased = false;
    mIsSecure = false;
    mMimes.clear();
#ifdef MTK_DRM_APP // 如果有#define定义了这个值才会执行其中的代码
    setDrmPlaybackStatusIfNeeded(Playback::STOP, 0);
    mDecryptHandle = NULL;
    mDrmManagerClient = NULL;
    mIsCurrentComplete = false;
#endif
}

//mtkadd+
void NuPlayer::GenericSource::init() {
    mIsMtkMusic = 0;
}

```

构造函数的逻辑比较简单，就是初始化一些参数，然后调用GenericSource::resetDataSource()方法。



### 析构函数

析构函数中，直接销毁了Looper，重置了DataSource，代码如下：

```c++
NuPlayer::GenericSource::~GenericSource() {
    ALOGV("~GenericSource");
    if (mLooper != NULL) {
        mLooper->unregisterHandler(id());
        mLooper->stop();
    }
    resetDataSource();
}
```



### setDataSource接口

设置数据源的接口，有三个重载，分别对应了Nuplayer类其中的setDataSourceAsync三个参数相同的函数，代码如下：

```c++
status_t NuPlayer::GenericSource::setDataSource(
        const sp<IMediaHTTPService> &httpService,
        const char *url,
        const KeyedVector<String8, String8> *headers) {
    ALOGV("setDataSource url: %s", url);

    resetDataSource();

    mHTTPService = httpService;
    mUri = url;

    if (headers) {
        mUriHeaders = *headers;
    }

    // delay data source creation to prepareAsync() to avoid blocking
    // the calling thread in setDataSource for any significant time.
    return OK;
}

status_t NuPlayer::GenericSource::setDataSource(
        int fd, int64_t offset, int64_t length) {
    ALOGV("setDataSource %d/%lld/%lld", fd, (long long)offset, (long long)length);

    resetDataSource();

    mFd = dup(fd);
    mOffset = offset;
    mLength = length;

    // delay data source creation to prepareAsync() to avoid blocking
    // the calling thread in setDataSource for any significant time.
    return OK;
}

status_t NuPlayer::GenericSource::setDataSource(const sp<DataSource>& source) {
    ALOGV("setDataSource (source: %p)", source.get());

    resetDataSource();
    mDataSource = source;
    return OK;
}
```

这三个函数都是先调用了resetDataSource方法重置DataSource，然后再保存一下数据。



### prepareAsync方法



```c++
void NuPlayer::GenericSource::prepareAsync() {
    ALOGV("prepareAsync: (looper: %d)", (mLooper != NULL));

    if (mLooper == NULL) {
        mLooper = new ALooper;
        mLooper->setName("generic");
        mLooper->start();

        mLooper->registerHandler(this);
    }

    sp<AMessage> msg = new AMessage(kWhatPrepareAsync, this);
    msg->post();
}

```

​	这个方法主要是用于发送kWhatPrepareAsync信息。并且在NuPlayer::GenericSource::onMessageReceived()方法中接受消息，并根据消息进行不同的操作。如何是kWhatPrepareAsync这个消息的话，就会执行GenericSource类中的onPrepareAsync，此方法如下：

```c++
void NuPlayer::GenericSource::onPrepareAsync() {
    ALOGV("onPrepareAsync: mDataSource: %d", (mDataSource != NULL));

    // delayed data source creation  延迟创建DataSource
    if (mDataSource == NULL) {
        // set to false first, if the extractor
        // comes back as secure, set it to true then.
        mIsSecure = false;

        if (!mUri.empty()) {
            const char* uri = mUri.c_str();
            String8 contentType;

            if (!strncasecmp("http://", uri, 7) || !strncasecmp("https://", uri, 8)) {
                mHttpSource = DataSource::CreateMediaHTTP(mHTTPService);
                if (mHttpSource == NULL) {
                    ALOGE("Failed to create http source!");
                    notifyPreparedAndCleanup(UNKNOWN_ERROR);
                    return;
                }
            }

            mDataSource = DataSource::CreateFromURI(
                   mHTTPService, uri, &mUriHeaders, &contentType,
                   static_cast<HTTPBase *>(mHttpSource.get()));
        } else {
            if (property_get_bool("media.stagefright.extractremote", true) &&
                    !FileSource::requiresDrm(mFd, mOffset, mLength, nullptr /* mime */)) {
                sp<IBinder> binder =
                        defaultServiceManager()->getService(String16("media.extractor"));
                if (binder != nullptr) {
                    ALOGD("FileSource remote");
                    sp<IMediaExtractorService> mediaExService(
                            interface_cast<IMediaExtractorService>(binder));
                    sp<IDataSource> source =
                            mediaExService->makeIDataSource(mFd, mOffset, mLength);
                    ALOGV("IDataSource(FileSource): %p %d %lld %lld",
                            source.get(), mFd, (long long)mOffset, (long long)mLength);
                    if (source.get() != nullptr) {
                        mDataSource = DataSource::CreateFromIDataSource(source);
                        if (mDataSource != nullptr) {
                            // Close the local file descriptor as it is not needed anymore.
                            close(mFd);
                            mFd = -1;
                        }
                    } else {
                        ALOGW("extractor service cannot make data source");
                    }
                } else {
                    ALOGW("extractor service not running");
                }
            }
            if (mDataSource == nullptr) {
                ALOGD("FileSource local");
                mDataSource = new FileSource(mFd, mOffset, mLength);
            }
            // TODO: close should always be done on mFd, see the lines following
            // DataSource::CreateFromIDataSource above,
            // and the FileSource constructor should dup the mFd argument as needed.
            mFd = -1;
        }

        if (mDataSource == NULL) {
            ALOGE("Failed to create data source!");
            notifyPreparedAndCleanup(UNKNOWN_ERROR);//上报处理结束
            return;
        }
    }

    if (mDataSource->flags() & DataSource::kIsCachingDataSource) {
        mCachedSource = static_cast<NuCachedSource2 *>(mDataSource.get());
    }

    // For cached streaming cases, we need to wait for enough
    // buffering before reporting prepared.
    mIsStreaming = (mCachedSource != NULL);

    // init extractor from data source
    status_t err = initFromDataSource();//多媒体文件格式探测

    if (err != OK) {
        ALOGE("Failed to init from data source!");
        notifyPreparedAndCleanup(err); 
        return;
    }

    if (mVideoTrack.mSource != NULL) {
        sp<MetaData> meta = doGetFormatMeta(false /* audio */);
        sp<AMessage> msg = new AMessage;
        err = convertMetaDataToMessage(meta, &msg);
        if(err != OK) {
            notifyPreparedAndCleanup(err);
            return;
        }
        notifyVideoSizeChanged(msg);//上报视频分辨率
    }

#ifdef MTK_DRM_APP  // OMA DRM v1 implementation: consume rights.
        mIsCurrentComplete = false;
        ConsumeRight(mDecryptHandle, mDrmManagerClient, mDrmProcName);
#endif
    //上报流状态
    notifyFlagsChanged(
            // FLAG_SECURE will be known if/when prepareDrm is called by the app
            // FLAG_PROTECTED will be known if/when prepareDrm is called by the app
            FLAG_CAN_PAUSE |
            FLAG_CAN_SEEK_BACKWARD |
            FLAG_CAN_SEEK_FORWARD |
            FLAG_CAN_SEEK);

    finishPrepareAsync();//上报函数调用正常结束

    ALOGV("onPrepareAsync: Done");
}
```

这个函数完成了格式探测和Metadata提取等动作。

这里我们重要注意initFromDataSource（）函数，这个函数主要是解析DataSource，并且将解析的数据进行保存。代码如下：

```c++
status_t NuPlayer::GenericSource::initFromDataSource() {
    sp<IMediaExtractor> extractor;
    CHECK(mDataSource != NULL);

    extractor = MediaExtractor::Create(mDataSource, NULL);

    if (extractor == NULL) {
        ALOGE("initFromDataSource, cannot create extractor!");
        return UNKNOWN_ERROR;
    }

#ifdef MTK_DRM_APP
    mDataSource->getDrmInfo(mDecryptHandle, &mDrmManagerClient);
    if (mDecryptHandle.get() != NULL && DecryptApiType::CONTAINER_BASED == mDecryptHandle->decryptApiType
        && RightsStatus::RIGHTS_VALID != mDecryptHandle->status) {
            sp<AMessage> msg = dupNotify();
            msg->setInt32("what", kWhatDrmNoLicense);
            msg->post();
    }
#endif
    mFileMeta = extractor->getMetaData();
    if (mFileMeta != NULL) {
        int64_t duration;
        if (mFileMeta->findInt64(kKeyDuration, &duration)) {
            mDurationUs = duration;
        }
    }

    int32_t totalBitrate = 0;

    size_t numtracks = extractor->countTracks();
    if (numtracks == 0) {
        ALOGE("initFromDataSource, source has no track!");
        return UNKNOWN_ERROR;
    }

    mMimes.clear();
	//读取多媒体文件的全部Track信息
    for (size_t i = 0; i < numtracks; ++i) {
        //mtkadd+ tricky to set flag before gettrack
        if (mIsMtkMusic) {
            sp<MetaData> mp3Meta = extractor->getTrackMetaData(i, MediaExtractor::kIncludeMp3LowPowerInfo);
            ALOGD("set mp3 low power");
        }

        sp<IMediaSource> track = extractor->getTrack(i);
        if (track == NULL) {
            continue;
        }

        sp<MetaData> meta = extractor->getTrackMetaData(i);
        if (meta == NULL) {
            ALOGE("no metadata for track %zu", i);
            return UNKNOWN_ERROR;
        }

        const char *mime;
        CHECK(meta->findCString(kKeyMIMEType, &mime));

        ALOGV("initFromDataSource track[%zu]: %s", i, mime);

        // Do the string compare immediately with "mime",
        // we can't assume "mime" would stay valid after another
        // extractor operation, some extractors might modify meta
        // during getTrack() and make it invalid.
        if (!strncasecmp(mime, "audio/", 6)) {
            if (mAudioTrack.mSource == NULL) {
                mAudioTrack.mIndex = i;
                mAudioTrack.mSource = track;
                mAudioTrack.mPackets =
                    new AnotherPacketSource(mAudioTrack.mSource->getFormat());

                if (!strcasecmp(mime, MEDIA_MIMETYPE_AUDIO_VORBIS)) {
                    mAudioIsVorbis = true;
                } else {
                    mAudioIsVorbis = false;
                }

                mMimes.add(String8(mime));
            }
        } else if (!strncasecmp(mime, "video/", 6)) {
            if (mVideoTrack.mSource == NULL) {
                mVideoTrack.mIndex = i;
                mVideoTrack.mSource = track;
                mVideoTrack.mPackets =
                    new AnotherPacketSource(mVideoTrack.mSource->getFormat());

                // video always at the beginning
                mMimes.insertAt(String8(mime), 0);
            }
        }
		//处理音频信息，并且保存
        mSources.push(track);
        int64_t durationUs;
        if (meta->findInt64(kKeyDuration, &durationUs)) {
            if (durationUs > mDurationUs) {
                mDurationUs = durationUs;
            }
        }

        int32_t bitrate;
        if (totalBitrate >= 0 && meta->findInt32(kKeyBitRate, &bitrate)) {
            totalBitrate += bitrate;
        } else {
            totalBitrate = -1;
        }
    }

    ALOGV("initFromDataSource mSources.size(): %zu  mIsSecure: %d  mime[0]: %s", mSources.size(),
            mIsSecure, (mMimes.isEmpty() ? "NONE" : mMimes[0].string()));

    if (mSources.size() == 0) {
        ALOGE("b/23705695");
        return UNKNOWN_ERROR;
    }

    // Modular DRM: The return value doesn't affect source initialization.
    (void)checkDrmInfo();

    mBitrate = totalBitrate;

    return OK;
}
```



### stop/start函数

stop函数相对简单，直接修改当前的状态：

```c++
void NuPlayer::GenericSource::stop() {
#ifdef MTK_DRM_APP
    setDrmPlaybackStatusIfNeeded(Playback::STOP, 0);
    mIsCurrentComplete = true;
#endif
    mStarted = false;
}
```

Start函数主要发送 kWhatStart 消息，代码如下：

```c++
void NuPlayer::GenericSource::start() {
    ALOGI("start");
    mStopRead = false;
    if (mAudioTrack.mSource != NULL) {
        postReadBuffer(MEDIA_TRACK_TYPE_AUDIO);
    }
    if (mVideoTrack.mSource != NULL) {
        postReadBuffer(MEDIA_TRACK_TYPE_VIDEO);
    }
    mStarted = true;
#ifdef MTK_DRM_APP
    setDrmPlaybackStatusIfNeeded(Playback::START, getLastReadPosition() / 1000);
    if (mIsCurrentComplete) {    // single recursive mode
        ConsumeRight(mDecryptHandle, mDrmManagerClient, mDrmProcName);
        mIsCurrentComplete = false;
    }
#endif
    (new AMessage(kWhatStart, this))->post();
}
```

发送消息后会在本类中的onMessageReceived方法中响应，而响应的方法也比较简单：

```c++
 case kWhatStart:
 case kWhatResume:
      {
          mBufferingMonitor->restartPollBuffering();
          break;
      }
```



### pause/resume函数

这两个函数跟start/pause类似，直接设置状态值：

```c++
void NuPlayer::GenericSource::pause() {
#ifdef MTK_DRM_APP
    setDrmPlaybackStatusIfNeeded(Playback::PAUSE, 0);
#endif
    mStarted = false;
}

void NuPlayer::GenericSource::resume() {
#ifdef MTK_DRM_APP
    setDrmPlaybackStatusIfNeeded(Playback::START, getLastReadPosition() / 1000);
#endif
    mStarted = true;

    (new AMessage(kWhatResume, this))->post();
}
```

resume方法主要发送kWhatResume消息，在本类中的onMessageReceived方法中响应，响应如下：

```c++
 case kWhatStart:
 case kWhatResume:
      {
          mBufferingMonitor->restartPollBuffering();
          break;
      }
```



### seekTo函数

seekTo是实现多媒体文件seek的主要函数，其实现跟kWhatSeek消息有关，代码如下：

```c++
status_t NuPlayer::GenericSource::seekTo(int64_t seekTimeUs, MediaPlayerSeekMode mode) {
    sp<AMessage> msg = new AMessage(kWhatSeek, this);
    msg->setInt64("seekTimeUs", seekTimeUs);
    msg->setInt32("mode", mode);

    sp<AMessage> response;
    status_t err = msg->postAndAwaitResponse(&response);
    if (err == OK && response != NULL) {
        CHECK(response->findInt32("err", &err));
    }

    return err;
}
```

实际响应是在onMessageReceived方法中，如下：

```c++
case kWhatSeek:
      {
          onSeek(msg);
          break;
      }
      
      
void NuPlayer::GenericSource::onSeek(const sp<AMessage>& msg) {
    int64_t seekTimeUs;
    int32_t mode;
    CHECK(msg->findInt64("seekTimeUs", &seekTimeUs));
    CHECK(msg->findInt32("mode", &mode));

    sp<AMessage> response = new AMessage;
    status_t err = doSeek(seekTimeUs, (MediaPlayerSeekMode)mode);
    response->setInt32("err", err);

    sp<AReplyToken> replyID;
    CHECK(msg->senderAwaitsResponse(&replyID));
    response->postReply(replyID);
}


status_t NuPlayer::GenericSource::doSeek(int64_t seekTimeUs, MediaPlayerSeekMode mode) {
    mBufferingMonitor->updateDequeuedBufferTime(-1ll);

    // If the Widevine source is stopped, do not attempt to read any
    // more buffers.
    //
    // TODO: revisit after widevine is removed.  May be able to
    // combine mStopRead with mStarted.
    if (mStopRead) {
        return INVALID_OPERATION;
    }
    if (mVideoTrack.mSource != NULL) {// 调整视频读取时间
        int64_t actualTimeUs;
        readBuffer(MEDIA_TRACK_TYPE_VIDEO, seekTimeUs, mode, &actualTimeUs);

        if (mode != MediaPlayerSeekMode::SEEK_CLOSEST) {
            seekTimeUs = actualTimeUs;
        }
        mVideoLastDequeueTimeUs = actualTimeUs;
    }

    if (mAudioTrack.mSource != NULL) {// 调整音频读取时间
        readBuffer(MEDIA_TRACK_TYPE_AUDIO, seekTimeUs);
        mAudioLastDequeueTimeUs = seekTimeUs;
    }
#ifdef MTK_DRM_APP
    setDrmPlaybackStatusIfNeeded(Playback::START, seekTimeUs / 1000);
    if (!mStarted) {
        setDrmPlaybackStatusIfNeeded(Playback::PAUSE, 0);
    }
#endif

    if (mSubtitleTrack.mSource != NULL) {
        mSubtitleTrack.mPackets->clear();
        mFetchSubtitleDataGeneration++;
    }

    if (mTimedTextTrack.mSource != NULL) {
        mTimedTextTrack.mPackets->clear();
        mFetchTimedTextDataGeneration++;
    }

    // If currently buffering, post kWhatBufferingEnd first, so that
    // NuPlayer resumes. Otherwise, if cache hits high watermark
    // before new polling happens, no one will resume the playback.
    mBufferingMonitor->stopBufferingIfNecessary();
    mBufferingMonitor->restartPollBuffering();

    return OK;
}
```



### disconnect

这个函数主要断开DataSource和GenericSource之间的关联，保证后续可用，代码如下：

```c++
void NuPlayer::GenericSource::disconnect() {
    sp<DataSource> dataSource, httpSource;
    {
        Mutex::Autolock _l(mDisconnectLock);
        dataSource = mDataSource;
        httpSource = mHttpSource;
    }

    if (dataSource != NULL) {
        // disconnect data source
        if (dataSource->flags() & DataSource::kIsCachingDataSource) {
            static_cast<NuCachedSource2 *>(dataSource.get())->disconnect();
        }
    } else if (httpSource != NULL) {
        static_cast<HTTPBase *>(httpSource.get())->disconnect();
    }
}
```



### getTrackCount、getTrackInfo、selectTrack和getSelectedTrack

这几个函数都是跟节目选择有关的，getTrackCount返回当前Source中包含的Track数目（一个Track可以是音频、视频、字幕或者文本），getTrackInfo则返回对应索引的详细信息。getSelectedTrack则返回当前选择或者正在播放的Track信息。selectTrack则用于选定特定的读取Track，也可用于取消读取。



### getFormat

这个接口用于获取音频或者视频格式，实现如下：

```c++
sp<AMessage> NuPlayer::Source::getFormat(bool audio) {
    sp<MetaData> meta = getFormatMeta(audio);

    if (meta == NULL) {
        return NULL;
    }

    sp<AMessage> msg = new AMessage;

    if(convertMetaDataToMessage(meta, &msg) == OK) {
        return msg;
    }
    return NULL;
}
```

也就是说可以看看getFormatMeta的实现逻辑，代码如下：

```c++
sp<MetaData> NuPlayer::GenericSource::getFormatMeta(bool audio) {
    sp<AMessage> msg = new AMessage(kWhatGetFormat, this);
    msg->setInt32("audio", audio);

    sp<AMessage> response;
    sp<RefBase> format;
    status_t err = msg->postAndAwaitResponse(&response);
    if (err == OK && response != NULL) {
        CHECK(response->findObject("format", &format));
        return static_cast<MetaData*>(format.get());
    } else {
        return NULL;
    }
}
```

主要是发送kWhatGetFormat消息，然后交给DataSource处理。



## 辅助信息获取及设置

这里面几个函数都比较简单，多数信息都是在prepareAsync函数中读取的。这里仅列出代码：

### getDuration

```c++
status_t NuPlayer::GenericSource::getDuration(int64_t *durationUs) {
    *durationUs = mDurationUs;
    return OK;
}
```

### isRealTime、isStreaming

```c++
virtual bool isRealTime() const {
    return false;
}

bool NuPlayer::GenericSource::isStreaming() const {
    return mIsStreaming;
}
```

### setOffloadAudio/setBuffers

```c++
void NuPlayer::GenericSource::setOffloadAudio(bool offload) {
    mBufferingMonitor->setOffloadAudio(offload);
}

status_t NuPlayer::GenericSource::setBuffers(
        bool audio, Vector<MediaBuffer *> &buffers) {
    if (mIsSecure && !audio && mVideoTrack.mSource != NULL) {
        return mVideoTrack.mSource->setBuffers(buffers);
    }
    return INVALID_OPERATION;
}
```

### feedMoreTSData

```c++
// 用于测试是否处理完所有数据
status_t NuPlayer::GenericSource::feedMoreTSData() {
    return OK;
}
```



















