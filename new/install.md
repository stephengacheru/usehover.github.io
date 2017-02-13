---
layout: default
nav-class: install-hover
---

# Installing the Hover SDK

1. To include the SDK in your app, first add Hover to your app-level build.gradle dependencies:

{% highlight gradle %}
buildscript {
  repositories {
    mavenCentral()
    maven { url 'https://maven.fabric.io/public' }
    maven { url 'http://maven.usehover.com/releases' }
  }

  dependencies {
    classpath 'com.android.tools.build:gradle:2.2.2'
  }
}

repositories {
  mavenCentral()
  maven { url 'http://maven.usehover.com/releases' }
}

dependencies {
  compile('com.hover:android-sdk:0.10.4') { transitive = true; }
  compile 'com.android.support:appcompat-v7:25.1.0'
  compile 'com.android.support:support-v4:25.1.0'
}
{% endhighlight %}

2. Then add the following permissions to AndroidManifest.xml:

{% highlight xml %}
    <uses-permission android:name="android.permission.CALL_PHONE"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.RECEIVE_SMS"/>
    <uses-permission android:name="android.permission.READ_SMS"/>
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
    <uses-permission android:name="android.permission.BIND_ACCESSIBILITY_SERVICE"/>
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>

    <uses-feature android:name="android.hardware.telephony"/>
{% endhighlight %}

Finally, include your API token as application level meta-data in AndroidManifest.xml:

{% highlight xml %}
<application>
  ...
  <meta-data
     android:name="com.hover.ApiKey"  
     android:value="<YOUR_API_TOKEN>"/>
</application>
{% endhighlight %}

To find or create an API token visit your [Hover dashboard](https://www.usehover.com/apps)  and click on your app, or make a new one.
