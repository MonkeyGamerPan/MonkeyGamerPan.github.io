---
layout: post
title: android 网络类型对应
featured-img: sleek
mathjax: true
categories: [andriod,network]
---

## android 网络类型对应

int NETWORK_TYPE_UNKNOWN =0 Network type is unknown   

int NETWORK_TYPE_GPRS   =1 Current network is GPRS     移动或联通2G

int NETWORK_TYPE_EDGE   =2 Current network is EDGE     移动或联通2G

int NETWORK_TYPE_CDMA   =4 Current network is CDMA     Either IS95A or IS95B  电信2G

int NETWORK_TYPE_UMTS   =3 Current network is UMTS     联通3G

int NETWORK_TYPE_HSDPA  =8 Current network is HSDPA  3G

int NETWORK_TYPE_HSUPA =9 Current network is HSUPA  3G

int NETWORK_TYPE_HSPA   =10 Current network is HSPA  3G

int   NETWORK_TYPE_IDEN  =11 Current network is iDen

int NETWORK_TYPE_LTE   =13 Current network is LTE  4G（LTE就是俗称的3.9G）

int NETWORK_TYPE_EHRPD  =14 Current network is eHRPD     eHRPD是CDMA的演进技术，大概就是3.75g，如果你在中国，那就是电信使用的3g技术的一种演进技术。

int NETWORK_TYPE_HSPAP  =15 Current network is HSPAP  3G

int NETWORK_TYPE_GSM   =16 Current network is GSM      由欧洲开发的数字移动电话网络标准，它的开发目的是让全球各地共同使用一个移动电话网络标准，让用户使用一部手机就能行遍全球。

int NETWORK_TYPE_EVDO_0  =5 Current network is EVDO revision 0   电信3G

int NETWORK_TYPE_EVDO_A  =6 Current network is EVDO revision A   电信3G

int NETWORK_TYPE_EVDO_B =12 Current network is EVDO revision B     电信3G

int NETWORK_TYPE_1xRTT  =7 Current network is 1xRTT      电信2G

<br/>

android/frameworks/base/telephony/java/android/telephony/TelephonyManager.java

```java
@UnsupportedAppUsage
    public static int getNetworkClass(int networkType) {
        switch (networkType) {
            case NETWORK_TYPE_GPRS:
            case NETWORK_TYPE_GSM:
            case NETWORK_TYPE_EDGE:
            case NETWORK_TYPE_CDMA:
            case NETWORK_TYPE_1xRTT:
            case NETWORK_TYPE_IDEN:
                return NETWORK_CLASS_2_G;
            case NETWORK_TYPE_UMTS:
            case NETWORK_TYPE_EVDO_0:
            case NETWORK_TYPE_EVDO_A:
            case NETWORK_TYPE_HSDPA:
            case NETWORK_TYPE_HSUPA:
            case NETWORK_TYPE_HSPA:
            case NETWORK_TYPE_EVDO_B:
            case NETWORK_TYPE_EHRPD:
            case NETWORK_TYPE_HSPAP:
            case NETWORK_TYPE_TD_SCDMA:
                return NETWORK_CLASS_3_G;
            case NETWORK_TYPE_LTE:
            case NETWORK_TYPE_IWLAN:
            case NETWORK_TYPE_LTE_CA:
                return NETWORK_CLASS_4_G;
            default:
                return NETWORK_CLASS_UNKNOWN;
        }
    }
```



<br/><br/><br/>

## android手机制式

int PHONE_TYPE_CDMA——手机制式为CDMA，电信
int PHONE_TYPE_GSM——手机制式为GSM，移动和联通
int PHONE_TYPE_NONE——手机制式未知

<br/>

android/frameworks/base/telephony/java/android/telephony/TelephonyManager.java

```java
    @UnsupportedAppUsage
    public static int getPhoneType(int networkMode) {
        switch(networkMode) {
        case RILConstants.NETWORK_MODE_CDMA:
        case RILConstants.NETWORK_MODE_CDMA_NO_EVDO:
        case RILConstants.NETWORK_MODE_EVDO_NO_CDMA:
            return PhoneConstants.PHONE_TYPE_CDMA;

        case RILConstants.NETWORK_MODE_WCDMA_PREF:
        case RILConstants.NETWORK_MODE_GSM_ONLY:
        case RILConstants.NETWORK_MODE_WCDMA_ONLY:
        case RILConstants.NETWORK_MODE_GSM_UMTS:
        case RILConstants.NETWORK_MODE_LTE_GSM_WCDMA:
        case RILConstants.NETWORK_MODE_LTE_WCDMA:
        case RILConstants.NETWORK_MODE_LTE_CDMA_EVDO_GSM_WCDMA:
        case RILConstants.NETWORK_MODE_TDSCDMA_ONLY:
        case RILConstants.NETWORK_MODE_TDSCDMA_WCDMA:
        case RILConstants.NETWORK_MODE_LTE_TDSCDMA:
        case RILConstants.NETWORK_MODE_TDSCDMA_GSM:
        case RILConstants.NETWORK_MODE_LTE_TDSCDMA_GSM:
        case RILConstants.NETWORK_MODE_TDSCDMA_GSM_WCDMA:
        case RILConstants.NETWORK_MODE_LTE_TDSCDMA_WCDMA:
        case RILConstants.NETWORK_MODE_LTE_TDSCDMA_GSM_WCDMA:
        case RILConstants.NETWORK_MODE_LTE_TDSCDMA_CDMA_EVDO_GSM_WCDMA:
            return PhoneConstants.PHONE_TYPE_GSM;

        // Use CDMA Phone for the global mode including CDMA
        case RILConstants.NETWORK_MODE_GLOBAL:
        case RILConstants.NETWORK_MODE_LTE_CDMA_EVDO:
        case RILConstants.NETWORK_MODE_TDSCDMA_CDMA_EVDO_GSM_WCDMA:
            return PhoneConstants.PHONE_TYPE_CDMA;

        case RILConstants.NETWORK_MODE_LTE_ONLY:
            if (getLteOnCdmaModeStatic() == PhoneConstants.LTE_ON_CDMA_TRUE) {
                return PhoneConstants.PHONE_TYPE_CDMA;
            } else {
                return PhoneConstants.PHONE_TYPE_GSM;
            }
        default:
            return PhoneConstants.PHONE_TYPE_GSM;
        }
    }
```

<br/><br/><br/>

## SIM卡状态

获取SIM卡状态：int getSimState()
int SIM_STATE_ABSENT——SIM卡未找到
int SIM_STATE_NETWORK_LOCKED——SIM卡网络被锁定，需要Network PIN解锁
int SIM_STATE_PIN_REQUIRED——SIM卡PIN被锁定，需要User PIN解锁
int SIM_STATE_PUK_REQUIRED——SIM卡PUK被锁定，需要User PUK解锁
int SIM_STATE_READY——SIM卡可用
int SIM_STATE_UNKNOWN——SIM卡未知



































