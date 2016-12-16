---
layout: default
---

# Build your first integration

## 1. Install Hover

Add the Hover SDK to your app-level build.gradle dependencies

{% highlight gradle %}
buildscript {
  repositories {
    mavenCentral()
    maven { url 'http://maven.usehover.com/releases' }
  }

  dependencies {
    classpath 'com.android.tools.build:gradle:2.1.0'
  }
}

repositories {
  mavenCentral()
  maven { url 'http://maven.usehover.com/releases' }
}

dependencies {
  compile('com.hover:android-sdk:0.8.8@aar') { transitive = true; }
}
{% endhighlight %}

Add required permissions and API key to your Android manifest

{% highlight xml %}
<uses-permission android:name="android.permission.CALL_PHONE"/>
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
<uses-permission android:name="android.permission.RECEIVE_SMS"/>
<uses-permission android:name="android.permission.READ_SMS"/>
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
<uses-permission android:name="android.permission.BIND_ACCESSIBILITY_SERVICE"/>
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
<uses-permission android:name="android.permission.INTERNET"/>

<uses-feature android:name="android.hardware.telephony"/>
{% endhighlight %}

{% highlight xml %}
<meta-data
   android:name="com.hover.ApiKey"  
   android:value="cad46be34a2934a362d9e0b74da4b62c"/>
{% endhighlight %}

## 2. Create a buy button

![Samsung phone with buy button](/assets/samsung_buy_button.png)

Hover's **Buy Button** offers the fastest way to integrate mobile money payments into your app. You can add the button directly into the XML for your view:

{% highlight xml %}
<com.hover.sdk.BuyButton
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
{% endhighlight %}

or in Java:

{% highlight java %}
BuyButton buyButton = new BuyButton(context);
layout.addView(buyButton);
{% endhighlight %}

<div class="spacer">&nbsp;</div>

You must pass additional parameters to customize the button's behavior:

{% highlight java %}
BuyBtnCallback callback = new BuyBtnCallback() {
  @override
  public void onError(String message) { }

  @override
  public void onSimError(String message) { }

  @override
  public void onUserDenied() { }

  @override
  public void onRequestMade(String operatorResponseMessage, String requestId) { }

  BuyParameters buyParams = new BuyParameters.Builder(context)
      .setAmount("100", "Ksh")
      .setCallback(callback);

  buyButton.setBuyParameters(buyParams);
};
{% endhighlight %}

See the [full API documentation](javascript:void(0);) for more information about these parameters.

<div id="interactive-wrapper">
  <h4 id="try-it-now">Try it now!</h4>
  <p>Test the buy button on your device and keep this window open. You'll get a notification here when a transaction is successful:</p>
  <figure class="highlight">
    <pre><div class="loading">waiting</div></pre>
  </figure>
</div>

When the button is pressed Hover will ask the user for permission. Once granted the transaction will be initiated. If the request is made you will get the `onRequestMade` callback, which tells you that the payment provider is processing your payment. 

## 3. Receive confirmation

Add a `BroadcastReceiver` which receives intents with the action `YOUR.PACKAGE.NAME.CONFIRMED_TRANSACTION` to your Android Manifest.

{% highlight xml %}
<receiver
    android:name=".TransactionReceiver"
    android:enabled="true"
    android:exported="false">
    <intent-filter>
        <action android:name="your.package.name.CONFIRMED_TRANSACTION"/>
    </intent-filter>
</receiver>
{% endhighlight %}

**Warning:** You must set `android:exported="false"` otherwise another app could spoof successful transactions:

Create the receiver and use the intent:

{% highlight java %}
public class TransactionReceiver extends BroadcastReceiver {
  public TransactionReceiver() { }

  @Override
  public void onReceive(Context context, Intent intent) {
    intent.getStringExtra("code");
  }
}
{% endhighlight %}

The intent received will contain the details of the transaction, such as the confirmation code, timestamp, and full response message from the operator. See the full [API documentation](javascript:void(0);) for details and a full list of available extras.
