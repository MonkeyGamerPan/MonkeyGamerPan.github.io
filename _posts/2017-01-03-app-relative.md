---
layout: post
title: android app相关的知识点
featured-img: sleek
mathjax: true
categories: note
---

Dialer:

dialer中包括了通话界面以及来电界面

<br/>

来电界面：

  由AnswerFragment.java来控制界面，布局文件为fragment_incoming_call.xml文件。

  布局文件中：

```xml
<com.android.incallui.autoresizetext.AutoResizeTextView
          android:id="@id/contactgrid_contact_name"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:layout_marginBottom="8dp"
          android:layout_marginStart="24dp"
          android:layout_marginEnd="24dp"
          android:singleLine="true"
          android:textAppearance="@style/Dialer.Incall.TextAppearance.Large"
          android:textSize="@dimen/answer_contact_name_text_size"
          app:autoResizeText_minTextSize="@dimen/answer_contact_name_min_size"
          tools:ignore="Deprecated"
          tools:text="Jake Peralta"/>
```

用于显示来电号码以及来电名字。

<br/>

控制此控件的文件为：ContactGridManager.java

在此文件中的以下方法为控制方法：

其中包括了电话号码和名称以及联系人图片的刷新显示。

```java
private void updatePrimaryNameAndPhoto() {
    if (TextUtils.isEmpty(primaryInfo.name())) {
      contactNameTextView.setText(null);
    } else {

      Log.d("gang.pan","name: " + primaryInfo.name());
      Log.d("gang.pan","number: " + primaryInfo.number());
      Log.d("gang.pan","nameIsNumber: " + primaryInfo.nameIsNumber());
      Log.d("gang.pan","label: " + primaryInfo.label());
      Log.d("gang.pan","isLocalContact: " + primaryInfo.isLocalContact());
      
      contactNameTextView.setText(
          primaryInfo.nameIsNumber()
              ? PhoneNumberUtils.createTtsSpannable(primaryInfo.name())
              : primaryInfo.name());

      // Set direction of the name field
      int nameDirection = View.TEXT_DIRECTION_INHERIT;
      if (primaryInfo.nameIsNumber()) {
        nameDirection = View.TEXT_DIRECTION_LTR;
      }
      contactNameTextView.setTextDirection(nameDirection);
    }

    //add by gang.pan for testing
    contactNameTextView.setText(contactNameTextView.getText()+"操你");

    if (avatarImageView != null) {
      if (hideAvatar) {
        avatarImageView.setVisibility(View.GONE);
      } else if (avatarSize > 0 && updateAvatarVisibility()) {
        if (ConfigProviderComponent.get(context)
            .getConfigProvider()
            .getBoolean("enable_glide_photo", false)) {
          loadPhotoWithGlide();
        } else {
          loadPhotoWithLegacy();
        }
      }
    }

  }
```

