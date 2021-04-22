---
layout: post
title: android 系统根据SIM卡自动选择语言
featured-img: sleek
mathjax: true
categories: note
---

# android根据SIM卡自动选择语言

在android/frameworks/opt/telephony/src/java/com/android/internal/telephony/MccTable.java中：

​    updateMccMncConfiguration(Context context, String mccmnc):此方法会在每次插卡开机SIM卡准备好了之后，系统就会调用。













## android无SIM卡默认语言设置

​	使用的是alps\device\公司名字\项目名字\项目名字.mk 中的PRODUCT_LOCALES请添加语言的时候在PRODUCT_LOCALES中添加。例如v500使用的是：android/device/tinno/v500/vnd_v500.mk

```makefile
PRODUCT_LOCALES := en_US zh_CN zh_TW es_ES pt_BR ru_RU fr_FR de_DE tr_TR vi_VN ms_MY in_ID th_TH it_IT ar_EG hi_IN bn_IN ur_PK fa_IR pt_PT nl_NL el_GR hu_HU tl_PH ro_RO cs_CZ ko_KR km_KH iw_IL my_MM pl_PL es_US bg_BG hr_HR lv_LV lt_LT sk_SK uk_UA de_AT da_DK fi_FI nb_NO sv_SE en_GB hy_AM zh_HK et_EE ja_JP kk_KZ sr_RS sl_SI ca_ES
```

​	例如我司自己内部的项目代号aubest52 那么添加语言路径是（device\mediatek\aubest52\full_aubest52.mk中的PRODUCT_LOCALES后面添加语言）



### 设置默认语言

​	如果想把某种语言设置为开机默认语言，只需把这个语言的代码放到(Android L PRODUCT_LOCALES后面第一个即可),（Android KK MTK_PRODUCT_LOCALES的第一个即可)。









# MTK问题

#### 添加语言后语言列表没有显示或者显示空白和乱码

​	在MTK_PRODUCT_LOCALES（KK及以前版本，L上是PRODUCT_LOCALES）中添某种语言代码，却没有在setting语言列表中找到该语言选项或者出现空白和乱码，出现这样的情况可以按照如下方法排查。

### 一、检查添加的语言代码是否正确

Android使用语言_区域来确定一种语言，比如en_US,zh_CN，前面两位表示语言，后面两位表示区域，语言和区域中间使用_隔开，多种语言中间用空格分隔。

语言代码遵循ISO_639-1标准，可以参考维基百科：ISO_639-1

http://zh.wikipedia.org/wiki/ISO_639-1

语言代码遵循ISO_3166-1标准，可以参考维基百科：ISO_3166-1

http://zh.wikipedia.org/wiki/ISO_3166-1

Note： Java中使用了几个过时的语言代码，与ISO_639-1中的不一样，见下表，因此在添加下面几种语言的时候需要额外注意：希伯来语，印尼语，意地绪语。

|                    | ISO_639-1 | Android/Java |
| ------------------ | --------- | ------------ |
| 希伯来语(Hebrew)   | he        | iw           |
| 印尼语(Indonesian) | id        | in           |
| 意地绪语(Yiddish)  | yi        | ji           |

### 二、检查framework是否有对应的value文件夹

如果添加的语言代码是正确的，列表种还是没有，请检查framework的res下是否有相应的values-xx-rYY文件夹，例如

JB2、JB3在ProjectConfig.mk文件MTK_PRODUCT_LOCALES处加上bn_IN,ur_PK后，setting语言列表却找不到这2个语言，那是因为

frameworks/base/core/res/res/下缺少文件values-bn-rIN和values-ur-rPK,需要新建并在其里面新建文件arrays.xml(KK和L上是strings.xml)，内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

  <!-- Do not translate. -->
  <integer-array name="maps_starting_lat_lng">
    <item>20593684</item>
    <item>78962880</item>
  </integer-array>
  <!-- Do not translate. -->
  <integer-array name="maps_starting_zoom">
    <item>3</item>
  </integer-array>

</resources>
```



### 三、语言列表中出现空白或者乱码

这是由于缺少字库或者字库添加不正确造成的，可以参考FAQ04513

如果按照上面步骤检查后仍有问题，请联系MTK技术人员解决。