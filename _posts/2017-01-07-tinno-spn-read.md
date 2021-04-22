---
layout: post
title: android 系统读取SIM卡运营商SPN
featured-img: sleek
mathjax: true
categories: note
---

# android系统读取SIM卡运营商SPN



##  SPN(Service Provider Name) 就是当前发行SIM卡的运营商的名称，可以从以下两个路径获取：

##     1、从SIM文件系统读取     2、从配置文件读取


    我们本节就来分析该字串的读取过程。

# 一、从SIM读取SPN过程

一般来说，SIM上保存有当前SIM的发行运营商名称，也就是SPN，该字串可以存储在SIM的EF_SPN(0x6F46)、EF_SPN_CPHS(0x6f14)、EF_SPN_SHORT_CPHS(0x6f18)三个地址上，在SIMRecords初始化时通过getSpnFsm()从SIM中读取出来并保存。下面来看读取SPN的过程：

```java
@SIMRecords.java
    protected void fetchSimRecords() {
      getSpnFsm(true, null);
    }
```

请注意，此时的getSpnFsm()的**start参数为true**，而且mSpnState为初始化值：GetSpnFsmState.IDLE

```java
private void getSpnFsm(boolean start, AsyncResult ar) {
      byte[] data;
      if (start) {
        if(mSpnState == GetSpnFsmState.READ_SPN_3GPP ||
            mSpnState == GetSpnFsmState.READ_SPN_CPHS ||
            mSpnState == GetSpnFsmState.READ_SPN_SHORT_CPHS ||
            mSpnState == GetSpnFsmState.INIT) {
          mSpnState = GetSpnFsmState.INIT;
          return;
        } else {
          //mSpnState默认为IDLE，然后修改为INIT
          mSpnState = GetSpnFsmState.INIT;
        }
      }
      switch(mSpnState){
        case INIT:
          //初始化SPN
          setServiceProviderName(null);
          //从SIM的EF_SPN读取SPN
          mFh.loadEFTransparent(EF_SPN, obtainMessage(EVENT_GET_SPN_DONE));
          mRecordsToLoad++;
          //mSpnState修改为READ_SPN_3GPP
          mSpnState = GetSpnFsmState.READ_SPN_3GPP;
          break;
        case READ_SPN_3GPP:
        case READ_SPN_CPHS:
        case READ_SPN_SHORT_CPHS:
        default:
          mSpnState = GetSpnFsmState.IDLE;
      }
    }
```

在上面的过程中，将会从EF_SPN中读取当前SPN，并且将mSpnState置为**READ_SPN_3GPP**。
 当读取完毕后，在handleMessage()中读取反馈：

   

```java
 public void handleMessage(Message msg) {
      try {
        switch (msg.what) {
          case EVENT_GET_SPN_DONE:
            isRecordLoadResponse = true;
            ar = (AsyncResult) msg.obj;
            getSpnFsm(false, ar);
            break;
        }
      } catch (RuntimeException exc) {
      } finally {
        if (isRecordLoadResponse) {
          onRecordLoaded();
        }
      }
    }
然后再次进入getSpnFsm()中处理，此时的mSpnState状态为READ_SPN_3GPP，而start为false，所以直接进入switch语句判断：

    private void getSpnFsm(boolean start, AsyncResult ar) {
      byte[] data;
      if (start) {
      }
 
 
      switch(mSpnState){
       case INIT:
          break;
        case READ_SPN_3GPP:
          if (ar != null && ar.exception == null) {
            data = (byte[]) ar.result;
            //设置mSpnDisplayCondition显示标志位，如果通过EF_SPN没有取到，则认为SpnDisplayCondition=-1
            mSpnDisplayCondition = 0xff & data[0];
            setServiceProviderName(IccUtils.adnStringFieldToString( data, 1, data.length - 1));
            //将当前的SPN写入系统属性
            setSystemProperty(PROPERTY_ICC_OPERATOR_ALPHA, getServiceProviderName());
            mSpnState = GetSpnFsmState.IDLE;
          } else {
            mFh.loadEFTransparent( EF_SPN_CPHS, obtainMessage(EVENT_GET_SPN_DONE));
            mRecordsToLoad++;
            mSpnState = GetSpnFsmState.READ_SPN_CPHS;
            mSpnDisplayCondition = -1;
          }
          break;
        case READ_SPN_CPHS:
        case READ_SPN_SHORT_CPHS:
        default:
          mSpnState = GetSpnFsmState.IDLE;
      }
    }
```

 

如果此时从SIM读取的SPN不为空，则会通过adnStringFieldToString()将数据转换为字串后，通过**setServiceProviderName()**保存，同时也要存储在PROPERTY_ICC_OPERATOR_ALPHA的系统属性中，并且重置mSpnState为IDLE；
    另外，这里的mSpnDisplayCondition是SPN的第一位数据，在显示SPN时用来判定显示规则。
    **如果SIM中的SPN为空，则再去读取SIM中的EF_SPN_CPHS分区**，和上面流程相同，该请求会通过handleMessage()再次发送给getSpnFsm()内部，只不过这次进入**READ_SPN_CPHS**分支处理：

   

```java
 private void getSpnFsm(boolean start, AsyncResult ar) {
      byte[] data;
      if (start) {
      }

      switch(mSpnState){
        case INIT:
          break;
        case READ_SPN_3GPP:
          break;
        case READ_SPN_CPHS:
          if (ar != null && ar.exception == null) {
            data = (byte[]) ar.result;
            setServiceProviderName(IccUtils.adnStringFieldToString(data, 0, data.length));
            setSystemProperty(PROPERTY_ICC_OPERATOR_ALPHA, getServiceProviderName());
            mSpnState = GetSpnFsmState.IDLE;
          } else {
            mFh.loadEFTransparent( EF_SPN_SHORT_CPHS, obtainMessage(EVENT_GET_SPN_DONE));
            mRecordsToLoad++;
            mSpnState = GetSpnFsmState.READ_SPN_SHORT_CPHS;
          }
          break;
        case READ_SPN_SHORT_CPHS:
        default:
          mSpnState = GetSpnFsmState.IDLE;
      }
    }
```


**与上面读取EF_SPN类似，如果读取成功就保存，否则再去读取EF_SPN_SHORT_CPHS，而读取后的结果一样在getSpnFsm()中处理**：

​    

```java
private void getSpnFsm(boolean start, AsyncResult ar) {
      byte[] data;
      if (start) {
      }
 
 
      switch(mSpnState){
        case INIT:
          break;
        case READ_SPN_3GPP:
          break;
        case READ_SPN_CPHS:
          break;
        case READ_SPN_SHORT_CPHS:
          if (ar != null && ar.exception == null) {
            data = (byte[]) ar.result;
            setServiceProviderName(IccUtils.adnStringFieldToString(data, 0, data.length));
            setSystemProperty(PROPERTY_ICC_OPERATOR_ALPHA, getServiceProviderName());
          }else {
            if (DBG) log("No SPN loaded in either CHPS or 3GPP");
          }
          mSpnState = GetSpnFsmState.IDLE;
        default:
          mSpnState = GetSpnFsmState.IDLE;
      }
    }
```



 遇上面流程类似，读取成功就保存，不成功就不再处理。

 经过上面的过程，就将SIM中的SPN读取并保存起来了。

# 二、从配置文件读取SPN过程

**Android原始代码中，无论在SIM的三个文件分区有没有查询到SPN，系统都会继续尝试从配置文件中读取SPN，如果读取成功，则覆盖刚才SIM中读取的值，如果配置文件读取失败，就使用上面的SIM中的SPN。
    开发者可以将所有预置的SPN存入spn-conf.xml这个文件中（不同平台该文件的存储位置不同），在编译时候就会将其拷贝到out的system\etc\目录中，以供系统读取。我们来挑选几条该文件中的项来看一下**：

```xml
@spn-conf.xml        
<spnOverride numeric="46000" spn="CHINA MOBILE"/>        
<spnOverride numeric="46001" spn="CHN-UNICOM"/>        
<spnOverride numeric="46002" spn="CHINA MOBILE"/>        
<spnOverride numeric="46003" spn="CHINA TELECOM"/>        
<spnOverride numeric="46007" spn="CHINA MOBILE"/>        
<spnOverride numeric="46008" spn="CHINA MOBILE"/>        
<spnOverride numeric="46009" spn="CHN-UNICOM"/>
```

 这些项是针对中国区的SPN，我们看到，每一项都包含两个元素，PLMN和SPN，**我们可以用当前SIM所驻留的网络的PLMN号码来匹配查找当前的SPN字串**。
下面我们来看如何将该文件读取到SPN中。
 SIMRecords对象在初始化时，在构造方法里面创建了一个SpnOverride对象：

​    @SIMRecords.java
​    public SIMRecords(UiccCardApplication app, Context c, CommandsInterface ci) {
​      super(app, c, ci);
​      mSpnOverride = new SpnOverride();
​    }


   **这里的SpnOverride作用就是读取系统预置的SPN列表，我们先来看其初始化流程：**

​     public SpnOverride () {
​      mCarrierSpnMap = new HashMap<String, String>();
​      loadSpnOverrides();
​    }
​    private void loadSpnOverrides() {
​      FileReader spnReader;
​      //PARTNER_SPN_OVERRIDE_PATH ="etc/spn-conf.xml"
​      final File spnFile = new File(Environment.getRootDirectory(), PARTNER_SPN_OVERRIDE_PATH);
 
 
​      try {
​        spnReader = new FileReader(spnFile);
​      } catch (FileNotFoundException e) {
​        Rlog.w(LOG_TAG, "Can not open " + Environment.getRootDirectory() + "/" + PARTNER_SPN_OVERRIDE_PATH);
​        return;
​      }
 
 
​      try {
​        //解析spn-conf.xml文件
​        XmlPullParser parser = Xml.newPullParser();
​        parser.setInput(spnReader);
​        XmlUtils.beginDocument(parser, "spnOverrides");
​        while (true) {
​          XmlUtils.nextElement(parser);
​          String name = parser.getName();
​          if (!"spnOverride".equals(name)) {
​            break;
​          }
 
 
​          String numeric = parser.getAttributeValue(null, "numeric");
​          String data   = parser.getAttributeValue(null, "spn");
​          mCarrierSpnMap.put(numeric, data);
​        }
​        spnReader.close();
​      } catch (XmlPullParserException e) {
​        Rlog.w(LOG_TAG, "Exception in spn-conf parser " + e);
​      } catch (IOException e) {
​        Rlog.w(LOG_TAG, "Exception in spn-conf parser " + e);
​      }
​    }

**这个对象在初始化时就将"etc/spn-conf.xml"文件加载进来，并进行XML解析，把每一项存入mCarrierSpnMap的HashMap中。然后该对象提供了两个查询SPN的方法：**

​    

```java
public boolean containsCarrier(String carrier) {
      //查询是否包含某个运营商的SPN
      return mCarrierSpnMap.containsKey(carrier);
    }
    public String getSpn(String carrier) {
      //获取某个运营商的SPN
      return mCarrierSpnMap.get(carrier);
    }


  然后我们接着上一节的介绍，当SIM中的SPN被读取之后，就会在SIMRecords中的handleMessage()消息中收到EVENT_GET_SPN_DONE的消息：
    public void handleMessage(Message msg) {
      try {
        switch (msg.what) {
          case EVENT_GET_SPN_DONE:
            isRecordLoadResponse = true;
            ar = (AsyncResult) msg.obj;
            getSpnFsm(false, ar);
            break;
        }
      } catch (RuntimeException exc) {
      } finally {
        if (isRecordLoadResponse) {
          onRecordLoaded();
        }
      }
    }
    public void handleMessage(Message msg) {
      try {
        switch (msg.what) {
          case EVENT_GET_SPN_DONE:
            isRecordLoadResponse = true;
            ar = (AsyncResult) msg.obj;
            getSpnFsm(false, ar);
            break;
        }
      } catch (RuntimeException exc) {
      } finally {
        if (isRecordLoadResponse) {
          onRecordLoaded();
        }
      }
    }
```

前面我们分析过，在getSpnFsm()中将会对Modem的返回结果进行解析，如果读取成功，就会将SPN字串保存起来，现在我们继续来看如果保存之后，将会进入finally的处理当中，也就是onRecordLoaded()方法：

```java
protected void onRecordLoaded() {
      mRecordsToLoad -= 1;
      if (mRecordsToLoad == 0 && mRecordsRequested == true) {
        onAllRecordsLoaded();
      } else if (mRecordsToLoad < 0) {
        mRecordsToLoad = 0;
      }
    }
```


 	这里的mRecordsToLoad**表明当前需要读取的SIM信息条数**(SIMRecords初始化过程中需要读取大量的SIM数据)，每向Modem发送一条读取的指令，该计数就会加1，当一条记录读取完毕后该计数就会减1，当所有记录全部读取完毕，就会进入onAllRecordsLoaded()的处理：

```java
protected void onAllRecordsLoaded() {
      setLocaleFromUsim();
      if (mParentApp.getState() == AppState.APPSTATE_PIN || mParentApp.getState() == AppState.APPSTATE_PUK) {
        mRecordsRequested = false;
        return ;
      }
 
 
      String operator = getOperatorNumeric();
      if (!TextUtils.isEmpty(operator)) {
        //保存当前的PLMN
        setSystemProperty(PROPERTY_ICC_OPERATOR_NUMERIC, operator);
        final SubscriptionController subController = SubscriptionController.getInstance();
        subController.setMccMnc(operator, subController.getDefaultSmsSubId());
      } else {
      }
 
 
      if (!TextUtils.isEmpty(mImsi)) {
        setSystemProperty(PROPERTY_ICC_OPERATOR_ISO_COUNTRY, MccTable.countryCodeForMcc(Integer.parseInt(mImsi.substring(0,3))));
      } else {
        log("onAllRecordsLoaded empty imsi skipping setting mcc");
      }
      //设置当前的语音信箱
      setVoiceMailByCountry(operator);
      //读取配置文件中的SPN
      setSpnFromConfig(operator);
      //将通知发送出来
      mRecordsLoadedRegistrants.notifyRegistrants( new AsyncResult(null, null, null));
    }
 我们来看setSpnFromConfig()的方法：

    private void setSpnFromConfig(String carrier) {
      if (mSpnOverride.containsCarrier(carrier)) {
        //用配置文件中的SPN来作为最终的SPN
        setServiceProviderName(mSpnOverride.getSpn(carrier));
        SystemProperties.set(PROPERTY_ICC_OPERATOR_ALPHA, getServiceProviderName());
      }
    }


```


​	这里就用上mSpnOverride这个对象了，前面我们分析过，他的作用就是把spn-conf.xml文件中的SPN信息解析出来，保存到HashMap中，现在我们需要根据当前的MCC/MNC去该HashMap中寻找匹配的SPN值，并把其作为最终的SPN保存起来。
 以上就是整个SPN的读取流程，下面用一张逻辑图来展示上述过程：
![img](../assets/img/androidPIC/SPN)