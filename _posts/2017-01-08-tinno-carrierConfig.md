---
layout: post
title: android CarrierConfig配置
featured-img: sleek
mathjax: true
categories: note
---

### 如何配置VoLTE, ViLTE and VoWifi(IMS config for VoLTE, ViLTE and VoWifi)

**（1）VoLTE，ViLTE Feature**

| **Selection**             |               **Feature option**               |
| ------------------------- | :--------------------------------------------: |
| 打开VoLTE（Enable VoLTE） | MTK_IMS_SUPPORT=yes<br />MTK_VOLTE_SUPPORT=yes |
| 打开ViLTE（Enable ViLTE） |            MTK_VILTE_SUPPORT = yes             |

  如果要支持ViLTE，必须也要支持VoLTE。

<br/>

 **（2）VoWifi Feature**

|     **Selection**     |                      **Feature option**                      |
| :-------------------: | :----------------------------------------------------------: |
| 打开WFC (Enable WFC） | MTK_IMS_SUPPORT=yes<br />MTK_VOLTE_SUPPORT=yes<br />MTK_WFC_SUPPORT=yes<br />MTK_EPDG_SUPPORT=yes<br />MTK_FLIGHT_MODE_POWER_OFF_MD=no(No need after Android-M1) |
| 关闭WFC (Disable WFC) |                      MTK_WFC_SUPPORT=no                      |

   VoWifi有些module属于binary release，如果基线版本不支持VoWifi, 请使用**[Patch Release]模板**提交eservice给CPM申请open VoWifi release patch.(VoWifi feature has binary release module, if basic version does not support VoWifi, please submit eservice to CPM according to **[patch release template]** for VoWifi patch release.)

   **所有提到的WFC名称，等同于VoWifi**。**如果要支持VoWiFi，必须支持VoLTE。（VoWifi=WFC）
<br/>

<br/>

 **(3) Dual VoLTE (93 platform)**

   双VoLTE是否开启，是由AP project configuation控制的。（Single/Dual VoLTE is Switched by AP Project Configuration.）

| **L+W/Single VoLTE project** |   **L+L/Dual VoLTE project**   |
| :--------------------------: | :----------------------------: |
|  MTK_MULTI_PS_SUPPORT = yes  |   MTK_MULTI_PS_SUPPORT = yes   |
| MTK_MULTIPLE_IMS_SUPPORT =1  |  MTK_MULTIPLE_IMS_SUPPORT = 2  |
| MTK_PROTOCOL2_RAT_CONFIG=W/G | MTK_PROTOCOL2_RAT_CONFIG=L/W/G |

   operator定制project 如果要支持dual VoLTE，也是使用上述project config。（Operator Customization project also use above configuration to enable dual VoLTE.）

   另外，单卡项目, 对于93 modem使用单volte配置即可，但对于90/91/92 modem chip，需要set MTK_MULTI_PS_SUPPORT = no.

   备注：我司(MTK)目前没有双LTE+单IMS的组合测试（不确定是否有潜在问题），所以建议是单LTE+单IMS 或双LTE+双IMS。

 <br/>

  **(4) ViWifi：前提是打开ViLTE and VoWifi feature， 并且配置MTK_VIWIFI_SUPPORT=yes**

<br/> 

  **(5) 关闭IMS，要确保这些project config都是no**

**MTK_IMS_SUPPORT = no**

**MTK_VOLTE_SUPPORT = no**

**MTK_VILTE_SUPPORT = no**

**MTK_WFC_SUPPORT = no**

**MTK_VIWIFI_SUPPORT = no**

**MTK_UT_XCAP_SUPPORT = no**

**MTK_DYNAMIC_SBP_SUPPORT = no**

**MTK_CT_VOLTE_SUPPORT = no**



**2.针对某家operator，如何配置支持VoLTE、ViLTE和VoWifi (config volte, ViLTE and VoWifi for operator)**

   开启dynamic IMS switch 这个feature后（property persist.mtk_dynamic_ims_switch 值为1），需要根据运营商的mccmnc来配置config以支持VoLTE、ViLTE和VoWifi，MTK默认已经配置好了大部分的运营商, 厂商可以新增支持的运营商。（English: If support dynamic IMS switch feature（value of property persist.mtk_dynamic_ims_switch is 1） and some operator need support volte, ViLTE and VoWifi, must add mccmnc config. MTK has add configs for most of operators. Customer can add new config further.）

 

| **FeatureName** | **device config key (default value)**           | **available config key (default value)**                     |      |
| --------------- | ----------------------------------------------- | ------------------------------------------------------------ | ---- |
| **VoLTE**       | **config_device_volte_available (false)**       | **carrier_volte_available_bool (aosp is false, MtkCarrierConfigManager.java put as true)** |      |
| **ViLTE**       | **config_device_vt_available (false)**          | **carrier_vt_available_bool (aosp is false, MtkCarrierConfigManager.java put as true)** |      |
| **VoWifi**      | **config_device_wfc_ims_available** **(false)** | **carrier_wfc_ims_available_bool (aosp is false, MtkCarrierConfigManager.java put as true)** |      |

   注：ViWifi没有单独的配置项，跟ViLTE使用相同的config配置。

   如果需要新增支持这些feature的运营商，配置相应mccmnc的config文件并将相应的config key 设置成true即可。(Config related config files to support these features for new opeator.) 

   "carrier_xxx_available_bool" 三个key的AOSP默认值是false，但是MtkCarrierConfigManager.java将这三个key的默认值改成了true，所以如果贵司版本有使用MtkCarrierConfigManager.java，就可以忽略**available config key**的配置，只用关注device config key。

   **device config key和available config key区别**：前者指定平台是否支持运营商的VoLTE/ViLTE/VoWifi；后者指定运营商的VoLTE/ViLTE/VoWifi是否可用。比如config_device_volte_available = true并且carrier_volte_available_bool = true，isVolteEnabledByPlatform()这个判断函数才可能返回true，如果返回false代表这两个config没有配对。

   比如需要为40492新增支持VoLTE，ViLTE and VoWifi，就需要配置以下file:
   (1) device/mediatek/common/overlay/telephony/frameworks/base/core/res/res/values-mcc404-mnc92/config.xml

```xml
     <bool translatable="false" name="config_device_volte_available">true</bool>
     <bool translatable="false" name="config_device_vt_available">true</bool>
     <bool translatable="false" name="config_device_wfc_ims_available">true</bool>
```

   (2)/vendor/mediatek/proprietary/packages/apps/CarrierConfig/assets/carrier_config_40492.xml

```xml
     <boolean name="carrier_volte_available_bool" value="true" />
     <boolean name="carrier_vt_available_bool" value="true" />
     <boolean name="carrier_wfc_ims_available_bool" value="true" />
```

<br/>

###  VoWiFi白名单

​    【VoWifi白名单】针对90/91/92平台，VoWifi还有一处额外的“VoWifi白名单”配置文件需要注意：
vendor/mediatek/proprietary/packages/services/WifiOffload/res/values/arrays.xml
这个file里面配置了支持VoWifi的运营商，可以搜索MTK main log，会找到类似如下打印:

```reStructuredText
04-11 12:25:42.845471 1684 1684 D WifiOffloadService: notifyMalWfcSupported: simId: 0, supported= 0, isEnabled= 1
```

​    "supported= 0"代表这家运营商并没有列入白名单，需要在arrays.xml里面增加运营商的mccmnc。

​    PS:如果MTK main log中能搜到关键字“WifiOffloadService”，一定就是90/91/92平台之一。

```reStructuredText
  04-11 12:25:42.845471 1684 1684 D WifiOffloadService: notifyMalWfcSupported......
```

​    

​    **P0自93 modem配置方法有改变，请参考FAQ21175 IMS Config TelephonyWare Modifications (Changes since P)**



**3. 如何通过log确认device config有没有配置（how to check if config_device_xxx_available value for some operator？）**
    在AP main log中搜索"ImsConfigManager"，可以看到65507这家operator的支持VoLTE和VoWifi（WFC is VoWifi），不支持ViLTE (search "ImsConfigManager" in AP main log, as below log, 65507 config as config_device_volte_available on, config_device_vt_available off, config_device_wfc_ims_available on.)
   06-01 06:19:07.584491 1502 1502 D ImsConfigManager: DYNAMIC_IMS_SWITCH_TRIGGER phoneId:0, simState:LOADED
   06-01 06:19:07.584765 1502 1502 D ImsConfigManager: get MtkImsConfigImpl of phone 0
   06-01 06:19:07.603638 1502 1502 D ImsConfigManager: SIM loaded on phone 0 with mcc: 655 mnc: 7
   06-01 06:19:07.610780 1502 1502 D ImsConfigManager: check iccid:8986xxxxxx3112345678
   06-01 06:19:07.645687 1502 1502 D ImsConfigManager: Set res capability: volte = 1, vilte = 0, wfc = 1
   06-01 06:19:07.717272 1502 1502 D ImsConfigManager: DYNAMIC_IMS_SWITCH_COMPLETE phoneId:0, simState:LOADED
   以上log只有在开机或是热插拔卡等SIM卡信息都读上来之后才会打印。（these log only print when receive SIM state change with SIM loaded state.）

   PS: VoWifi = WFC

 

**4. 如何通过log确认available config有没有配置（how to check if carrier_xxx_available_bool value for some operator？）**

   **如果贵司版本有使用MtkCarrierConfigManager.java，就可以忽略这部分available config key的配置。(If your project use MtkCarrierConfigManager.java, please ignore config available config key.)**

​    **在AP radio log中搜索"isCarrierConfigSupport"，可以看到carrier_xxx_available_bool的支持状态 (search "ImsConfigManager" in AP radio log, it will indicate VoLTE, ViLTE(vt) and VoWifi(wfc) config status)**

   //VoLTE config as true

   06-05 16:03:34.868076 1780 1920 D MtkImsManager: Volte, isResourceSupport:true, isCarrierConfigSupport:true, isGbaValidSupport:true, isFeatureEnableByPlatformExt:true

   //ViLTE config as true
   06-05 16:03:34.892714 1806 1806 D MtkImsManager: Vt, isResourceSupport:true, isCarrierConfigSupport:true, isGbaValidSupport:true, isFeatureEnableByPlatformExt:true

   //VoWifi config as false
   06-05 16:03:34.896064 1780 1920 D MtkImsManager: Wfc, isResourceSupport:false, isCarrierConfigSupport:false, isGbaValidSupport:true, isFeatureEnableByPlatformExt:true

 

**5. 确认开关状态（How to confirm setting enabled or not？）**

  VoLTE开关默认是打开的。(VoLTE setting default value is true.）
  ViLTE开关默认是打开的。(ViLTE setting default value is true.）
  VoWIFI开关默认是关闭的，通过carrier config carrier_default_wfc_ims_enabled_bool可以定制某些operator默认打开:
  For example:
  vendor/mediatek/proprietary/packages/apps/CarrierConfig/assets/carrier_config_405861.xml

```xml
<boolean name="carrier_default_wfc_ims_enabled_bool" value="true" />
```

   ImsManager 和MtkImsManager也会在radio log打印相关log来印证AP层配置和设置的状态。如果config配置正确，仍然没有注册，还需要确认对应设置里面的开关有没有打开。(user load需要打开telephony log 才能看到相关的log。)

  //isResourceSupport:true 代表config_device_volte_available已经配置成true，enabled = true代表VoLTE开关是打开的。如果要注册VoLTE，available + enabled都应该是true。
  06-07 13:31:05.055478 1147 1147 D MtkImsManager: Volte, isResourceSupport:true, isCarrierConfigSupport:true, isGbaValidSupport:true, isFeatureEnableByPlatformExt:true
  06-07 13:31:05.061109 1147 1147 D ImsManager: updateVolteFeatureValue: available = true, enabled = true, nonTTY = true

  //isResourceSupport:false 代表config_device_vt_available 没有配置成true，enabled = true代表ViLTE开关是打开的。如果要注册ViLTE，available + enabled都应该是true。
  06-07 13:31:05.022622 1147 1147 D MtkImsManager: Vt, isResourceSupport:false, isCarrierConfigSupport:true, isGbaValidSupport:true, isFeatureEnableByPlatformExt:true
  06-07 13:31:05.033264 1147 1147 D MtkImsManager: updateVideoCallFeatureValue: available = false, enabled = true, nonTTY = true, data enabled = true

  //isResourceSupport:true代表config_device_wfc_ims_available已经配置为true，enabled = false代表WFC开关是关闭的。如果要注册VoWifi，available + enabled都应该是true。
  06-07 13:31:04.966629 1147 1147 D MtkImsManager: Wfc, isResourceSupport:true, isCarrierConfigSupport:true, isGbaValidSupport:true, isFeatureEnableByPlatformExt:true
  06-07 13:31:04.979197 1147 1147 D ImsManager: updateWfcFeatureAndProvisionedValues: available = true, enabled = false, mode = 2, roaming = false

  //isResourceSupport:false代表config_device_wfc_ims_available没有配置为true，enabled = false代表WFC开关是关闭的。如果要注册VoWifi，available + enabled都应该是true。
  06-07 13:31:05.076595 1147 1147 D MtkImsManager: Wfc, isResourceSupport:false, isCarrierConfigSupport:true, isGbaValidSupport:true, isFeatureEnableByPlatformExt:true
  06-07 13:31:05.086758 1147 1147 D ImsManager: updateWfcFeatureAndProvisionedValues: available = false, enabled = false, mode = 2, roaming = false

 

**6. How to check if AP set enable ims to modem？**
  (1)对于93平台，在MTK radio log中搜索"AT+EIMSCFG"， 这个AT后面跟着6个value值(search "AT+EIMSCFG" in radio log for 93 modem，it is followed by 6 values):
  volteEnable, vilteEnable, vowifiEnable, viwifiEnable, smsEnable, imsEnable
  such as
      06-01 06:18:43.997299 943 1000 I AT : [0] AT> AT+EIMSCFG=1,0,1,0,1,1 (RIL_CMD_READER_3, tid:512083367152)
  indicate:volte on, vilte off, vowifi on, viwifi off, sms on, ims enabled
  Details:AT+EIMSCFG=1(volte on),0(vilte off),1(vowifi on),0(viwifi off),1(sms on),1(ims enabled)


  (2)对于 90/91/92平台，在MTK radio log中搜索 "AT+EIMS"（search "AT+EIMS" in radio log for 90/91/92 modem）；
   AT+EIMSVOICE：Voice capability enable or not
   AT+EIMSCCP：Video capability enable or not
   AT+EIMSWFC：VoWifi enable or not
   AT+EIMSSMS：SMS over IMS capability
   AT+EIMSVOLTE: VoLTE enable or not
   AT+EIMS:enable/disable IMS functionality
  (2.1) If you can't see video call button, you can check if AT+EIMSCCP=1 sent in radio log first.
  (2.2) If Switch on VoWifi setting, you can see in radio log:
      04-11 12:13:54.989483 982 986 D RIL-OEM : data = AT+EIMSWFC=0, length = 12 //Vowifi off
      04-11 12:14:25.033981 982 986 D RIL-OEM : data = AT+EIMSWFC=1, length = 12 //Vowifi on
   (2.3) WifiOffloadService will transfer all related setting to RDS, these AT is contolled by RDS.
      04-11 12:25:40.896286 1684 1684 D WifiOffloadService: notifyMalUserProfile(0): mIsVolteEnabled: true, mIsVilteEnabled: false mIsWfcEnabled: true mFqdn:       mIsWifiEnabled: false mHasWiFiDisabledPending: false mWfcMode: 2 mDataRoamingEnabled: 1 mIsAllowTurnOffIms: false

 

**7. IMS register status**

  在MTK radio log中搜索"CIREGU"

// 如果是+CIREGU: 0 代表IMS没有注册上

09-04 09:37:14.405392 948  967 I AT : [0] AT< +CIREGU: 0 (RIL_URC_READER, tid:503816533232)

// +CIREGU: 1,d代表注册上了voice、Video、sms 三中capability over IMS，

// 第二个参数是按bit位代表capability能力的，0x01代表Voice, 0x04代表SMS，0x08代表video
01-01 08:06:44.224432 4013 4044 I AT : [0] AT< +CIREGU: 1,d (RIL_URC_READER, tid:527043679472)

  

  //也可以通过MTK radio log中搜索"handleFeatureCapabilityChanged"确认VoLTE、VoWifi的注册状态，true代表注册

08-30 11:27:33.560581 1175 1175 D MtkImsPhoneCallTracker: [MtkImsPhoneCallTracker] handleFeatureCapabilityChanged: VoLTE:false ViLTE:false VoWiFi:false ViWiFi:false UTLTE:false UTWiFi:false isVideoEnabledStateChanged=false

08-30 11:27:33.561725 1175 1175 D MtkImsPhoneCallTracker: [MtkImsPhoneCallTracker] handleFeatureCapabilityChanged: VoLTE:false ViLTE:false VoWiFi:false ViWiFi:false UTLTE:false UTWiFi:false isVideoEnabledStateChanged=false

08-30 11:27:38.815967 1175 1175 D MtkImsPhoneCallTracker: [MtkImsPhoneCallTracker] handleFeatureCapabilityChanged: VoLTE:true ViLTE:true VoWiFi:false ViWiFi:false UTLTE:true UTWiFi:false isVideoEnabledStateChanged=true


**8. 如果需要看到全部IMS Framework log，需要开启telephony log (open telephony log to obain full IMS Framework log).**

  **[Important]** **提交IMS 相关eService，请务必打开telephony log**

  具体enable telephony log开关方法（open telephony log steps）：
   1.在拨号盘输入*#*#3646633#*#* （Dialer input *#*#3646633#*#*）；
   2.切换到“Log and Debugging”选单，找到“Telephony Log Setting”这项点击进入（“Log and Debugging”-> “Telephony Log Setting”）；
   3.点击enable，会有“set succeeded. Please reboot phone”提示弹出（press enable，will notify “set succeeded, Please reboot phone"）
   4.重启手机（reboot phone）

   重启以后log设定会一直有效，除非下次更改设定或恢复出厂设置。(After reboot, This persist config will be enabled unless you reset this setting or do factory reset.)

<br/>

<br/>

<br/>

<br/>

[DESCRIPTION]
Android默认配置wifi calling开关是关闭的。
frameworks/base/telephony/java/android/telephony/CarrierConfigManager.java
sDefaults.putBoolean(KEY_CARRIER_DEFAULT_WFC_IMS_ENABLED_BOOL, false);
在Android Q版本上，针对客户及某些运营商要求开关默认开启的方法如下：

[SOLUTION]
以Vodafone号段28602为例
1.首先去vendor/mediatek/proprietary/packages/providers/TelephonyProvider/assets/carrier_list.textpb
查看28602对应的operator, 可以看到carrier_id是1736，
carrier_id {
    canonical_id: 1736
    carrier_name: "Vodafone"
    carrier_attribute {
    mccmnc_tuple: "28602"
}

2.在以下路径下找到对应的XML文件做具体配置，注意查找包含carrier_id的文件。
vendor/mediatek/proprietary/packages/apps/CarrierConfig/assets/
以28602为例，应该是以下文件：
carrier_config_carrierid_1736_Vodafone.xml
需要配置：
<boolean name="carrier_default_wfc_ims_enabled_bool" value="true"/>

3.如果找不到对应的包含carrier_id的文件，接着查找包含MNC/MCC的配置文件，
比如 carrier_config_mccmnc_31100.xml，就是对应MNC/MCC = 31100运营商的配置文件。

4.如果还是查找不到，说明android及MTK未添加如何配置，客户可根据步骤1,2,3自行添加XML配置文件。

<br/>

<br/>

<br/>

<br/>

## [FAQ23375] How to enable VoLTE/VoWIFI roaming

**描述：如何开启VoLTE/VoWIFI的数据漫游**

<br/>

**1.To enable VoLTE/VoWiFi roaming in LR13/LR12/LR11**
To enable VoLTE roaming,we should find the corresponding operator SBP ID
 and set below values in **\mcu\custom\protocol\common\ps\custom_imc_config.c**

[modem](http://gitcode.tinno.com/gitweb?p=mt6762q%2Fplatform%2Fmodem.git;a=tree;hb=a012d5f344622032f3ee9dd8bd9ef5cf8c308fe1) / [mcu/pcore/custom/modem/common/ps](http://gitcode.tinno.com/gitweb?p=mt6762q%2Fplatform%2Fmodem.git;hb=a012d5f344622032f3ee9dd8bd9ef5cf8c308fe1;f=mcu%2Fpcore%2Fcustom%2Fmodem%2Fcommon%2Fps)/**custom_iwlan_config.c**

```c
nvram_ims_profile_ptr->imc_config.ims_roaming_mode = 1 
```

<br/>

To enable VoWIFI roaming,we should find the corresponding operator SBP ID
 and set below values in **\mcu\custom\protocol\common\ps\custom_iwlan_config.c**

[modem](http://gitcode.tinno.com/gitweb?p=mt6762q%2Fplatform%2Fmodem.git;a=tree;hb=a012d5f344622032f3ee9dd8bd9ef5cf8c308fe1) / [mcu/pcore/custom/modem/common/ps](http://gitcode.tinno.com/gitweb?p=mt6762q%2Fplatform%2Fmodem.git;hb=a012d5f344622032f3ee9dd8bd9ef5cf8c308fe1;f=mcu%2Fpcore%2Fcustom%2Fmodem%2Fcommon%2Fps)/**custom_iwlan_config.c**

```c
cfg->wans_cfg.wans_ims_wlan_roaming_barring_enable = KAL_FALSE;  开启VoWiFi Romaing

 cfg->wans_cfg.wans_ims_roaming_barring_enable = KAL_FALSE; 开启IMS Romaing
 cfg->wans_cfg.wans_ims_wlan_roaming_barring_enable = KAL_FALSE;  开启VoWiFi Romaing
```



**2.To enable VoLTE/VoWiFi roaming in NR15/VMOLY**

In NR15/VMOLY MD version, volte romaming control has been moved to custom_iwlan_config.c
we should find the corresponding operator SBP ID and set below values in 

**\mcu\custom\protocol\common\ps\custom_iwlan_config.c**

[modem](http://gitcode.tinno.com/gitweb?p=mt6762q%2Fplatform%2Fmodem.git;a=tree;hb=a012d5f344622032f3ee9dd8bd9ef5cf8c308fe1) / [mcu/pcore/custom/modem/common/ps](http://gitcode.tinno.com/gitweb?p=mt6762q%2Fplatform%2Fmodem.git;hb=a012d5f344622032f3ee9dd8bd9ef5cf8c308fe1;f=mcu%2Fpcore%2Fcustom%2Fmodem%2Fcommon%2Fps)/**custom_iwlan_config.c**

```c
/* VoLTE roaming enable */
cfg->ipol_ans_cfg.ipol_ims_roaming_barring_enable = KAL_FALSE;

/* VoWiFi roaming enable */

 cfg->**ipol_ans_cfg.ipol_ims_wlan_roaming_barring_enable = KAL_FALSE;

```

<br/><br/><br/><br/>

### CarrierConfig xml文件中常用KEY

```xml
<!--功能：不显示漫游图标 -->
<string-array name="non_roaming_operator_string_array" num="3">
        <item value="26001"/>
        <item value="26002"/>
        <item value="26003"/>
</string-array>


<!--功能：当SIM卡上没有预装语音信箱号码时，指定运营商的默认语音信箱号码。当字符串为空时，不指定默认语音邮箱号码 -->
<string name="default_vm_number_string">132</string>

<!--MTK平台才有   功能：增加了在飞行模式下支持WFC（VoWiFi）启用 true问支持，反之不支持-->
<boolean name="wos_flight_mode_support_bool" value="true" />

<!--功能：表示默认数据帐户是否应该显示LTE或4G图标 -->
<boolean name="show_4g_for_lte_data_icon_bool" value="false"/>

<!--功能：如果LTE+图标可用,布尔指示这个图标是否显示 -->
<boolean name="hide_lte_plus_data_icon_bool" value="false" />

<!--功能：指示该运营商VoLTE是否可用 -->
<boolean name="carrier_volte_available_bool" value="true"/>

<!--功能：指示该运营商VoWiFi是否可用 -->
<boolean name="carrier_wfc_ims_available_bool" value="true" />

<!--功能：指示该运营商ViLte(视频电话)是否可用 -->
<boolean name="carrier_vt_available_bool" value="true" />

<!--功能：设置“Enhanced 4G LTE”或“Advanced Calling(高级呼叫)”模式切换的默认状态 true:开启-->
<boolean name="enhanced_4g_lte_on_by_default_bool" value="true" />

<!--功能：确定用户是否可以在calling preference中切换Wi-Fi preferred或Cellular preferred。一些运营商只支持Wi-Fi Calling ，不支持VoLTE。他们不需要“Cellular preferred”选项。在这种情况下，为preferred preference设置unedititable属性。 -->
<boolean name="editable_wfc_mode_bool" value="true"/>

<!--功能：在IMS home Network中WFC的默认模式 0:仅支持Wi-Fi   1:首选移动网络   2: prefer Wi-Fi（偏向WiFi）-->
<int name="carrier_default_wfc_ims_mode_int" value="1"/>

<!--功能：VoWIFI 开关在setting中的名字 “”代表使用默认的 “xxx”代表使用xxx作为开关名字 -->
<string name="vowifi_toggle_name_string">WiFi Call</string>

<!--功能：标示WFC名字的字符串 -->
<string name="wfc_name_string">WiFi Call</string>
    
<!--功能：布尔值指示如果可用VoWifi图标应该显示 -->
<boolean name="hide_vowifi_registration_icon_bool" value="false" />

<!--功能：确定当前设备是否允许在呼叫记录中记录紧急号码。(一些运营商要求“不”记录紧急呼叫，大概是为了避免从呼叫记录界面意外重拨的风险。这是一个好主意，所以这里默认为false。) -->
<boolean name="allow_emergency_numbers_in_call_log_bool" value="false" />

    
```

