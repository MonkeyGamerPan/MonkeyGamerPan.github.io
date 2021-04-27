---
layout: post
title: android CarrierConfig配置
featured-img: sleek
mathjax: true
categories: note
---

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
