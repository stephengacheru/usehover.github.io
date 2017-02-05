---
layout: default
nav-class: overview
---

# Welcome to Hover

**Hover is currently in Alpha and may be unstable. If you have issues please get in touch with us.**

## Introduction

Hover is an Android SDK which allows an Android app to use supported Mobile Money wallets to pay for goods and services in-app. A developer can initiate a payment when a User presses a button or takes another action, or can create a subscription. Hover will deal with initiating the transaction, including getting the user's permission, and returning the confirmation back to the app. This means that apps can treat a Mobile Money payment just as they would a credit card payment.


## Using the SDK

### 1. Add a Hover Integration

In your `Activity`, add a Hover Integration. You may do this in `onCreate` or when the user presses a button ("Add Payment Method" for example). If you are targeting API 23+ You must use [Run Time Permissions](https://developer.android.com/training/permissions/requesting.html) to first ask for permission to `READ_PHONE_STATE` so that Hover can check the user's SIM card compatability. `HoverIntegration.add()` is necessary for the user to give you permission to use the mobile money service. It will also request that the user enable Accessibility Service if neccessary:

{% highlight java %}
@Override
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	...
	
	if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_PHONE_STATE) != PackageManager.PERMISSION_GRANTED) {
		ActivityCompat.requestPermissions(this, new String[] { Manifest.permission.READ_PHONE_STATE }, 0);
	} else {
		HoverIntegration.add("vodacom_tanzania", hoverListener, this);
	}
}

@Override
public void onRequestPermissionsResult(int requestCode,	String permissions[], int[] grantResults) {
	if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
    		HoverIntegration.add("vodacom_tanzania", hoverListener, this);
	} else {
    		// Explain why you need the permission and ask again
    	}
}
{% endhighlight %}

The first argument to `HoverIntegration.add()`must be one of our supported Mobile Money Operators, which you can find [here](https://www.usehover.com/countries/).

The second argument can be `null` or an implementation of `HoverIntegration.HoverListener` which provides callbacks for errors or success upon adding the integration. [More on it later](http://docs.usehover.com/#hoverintegrationhoverlistener-interface). The final argument is the `Context`.

You may also ask permission to use any Mobile Money available to the user. This is useful if you wish to support multiple operators in the same country or across multiple countries. This will ask the user to choose one of the operators supported by their SIM card. This will also automatically detect changes of the SIM card.

{% highlight java %}
HoverIntegration.add(hoverListener, this);
{% endhighlight %}

In this case the `hoverListener` `onSuccess` callback can be used to find out which operator they chose and, for convenience, what country it is from and what currency it uses.

### 2. Create a request

Create a request and launch the intent, for example, when a button is pressed:  

{% highlight java %}
Intent i = new Hover.Builder(this).request("send", amount, "Tsh", recipient).from("vodacom_tanzania");
startActivityForResult(i, 0);
{% endhighlight %}

To use `startActivityForResult`, `this` must be an `Activity`, otherwise you should simply use `startActivty` (which will not tell you whether the request is being processed or not). In most cases you must supply arguments to `Hover.Builder` which are passed on to the Mobile Money Service. The required arguments for each request are included on the Supported Operators page. In most cases you supply extra arguments using the `extra()` method:

{% highlight java %}
Intent i = new Hover.Builder(this).request("pay_bill", amount, "Tsh").extra("who", recipient_business_no).extra("acct", acct_no).from("vodacom_tanzania");
{% endhighlight %}

For convenience the `request()` method optionally takes the `amount`, `currency`, and `recipient`. There are also convience methods `amount(amount, currency)` and `to(recipient)`.

### 3. Use the result 

Implement `onActivityResult` to get the outcome of the request (NOT confirmation of the transaction, see below). If the `resultCode` is `RESULT_OK` then as far as the SDK can tell the request was accepted and is being processed by the Mobile Money Operator. See the next step to receive the final result. If the result code is `RESULT_CANCELED` then something went wrong and you should not expect the request to succeed. In this case the data intent returned will have a String Extra called `result` which will contain the error message. There are 2 types of failure: If the Intent that started the activity was improperly formed then a request was never made to the Mobile Money Operator. There will be no transaction and it will not count against your Hover transaction count. However, if the request failed after a request was actually made to the Mobile Money Operator then the data intent returned will include a `transaction_id` and the request will count against your Hover transaction count. This can happen because the user had insufficient balance, the amount sent was under the minimum amount required by the operator, or a host of other reasons. We recomend that in this case the `result` extra is shown to the user so that they can find out what went wrong.

{% highlight java %}
@Override
protected void onActivityResult (int requestCode, int resultCode, Intent data) {
  if (resultCode == Activity.RESULT_OK) {
    Log.i("Example", data.getStringExtra("response_message") + " contains the message from the operator");
    Log.i("Example", data.getStringExtra("transaction_id") + " contains the id of the pending transaction");
  } else if (resultCode == Activity.RESULT_CANCELED) {
    Log.i("Example", data.getStringExtra("result") + " contains the error that occured");
    if (data.hasExtra("transaction_id") // If a request was actually made to the Mobile Money Operator
      Log.i("Example", data.getStringExtra("transaction_id") + " contains the id of the failed transaction");
  }
}
{% endhighlight %}

### 4. Receive the transaction result

Add a `BroadcastReceiver` which receives intents with the action `YOUR.PACKAGE.NAME.CONFIRMED_TRANSACTION` to your Android Manifest. **Make sure exported is false** otherwise another app could spoof successful transactions:

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

Create the Receiver itself and use the intent as you need:

{% highlight java %}
public class TransactionReceiver extends BroadcastReceiver {
	public TransactionReceiver() { }

	@Override
	public void onReceive(Context context, Intent intent) {
	  intent.getStringExtra("code");
	}
}
{% endhighlight %}

The intent received will contain the details of the transaction, such as the confirmation code, timestamp, and full response message from the operator. See [below](http://docs.usehover.com/#transaction-result-details) for details.
 
### 5. Save proof of payment

Send the proof of payment to your servers for verification. Mobile networks are unreliable. If you can't reach your server, save the proof of payment and try again later.

## HoverIntegration.HoverListener Interface

Implement this interface to receive the call backs:

{% highlight java %}
public interface HoverListener {
  void onSIMError();
  void onUserDenied();
  void onSuccess(String operator, String countryName, String currency);
}
{% endhighlight %}

If on `SIMError` or `UserDenied` are called, then adding the integration has failed. For the former, the user does not have a SIM that supports the integration requested or possibly any SIM at all. For the latter, they did not give permission to use the integration requested. In this case you may show a message explaining why it is neccessary and ask them again by remaking the same call to `HoverIntegration.add()`.

`onSuccess` passes back the operator, country, and currency. This is especially useful if you asked for permission to use any mobile money available to the user, which they choose.

## Testing Your Integration
To test a specific operator replace your call to `HoverIntegration.add()` with:
{% highlight java %}
HoverIntegration.test("operatorSlug", hoverListener, context);
{% endhighlight %}
Your call to `Hover.Builder()...from("operatorSlug");` must also specify the same operator.

This will not actually contact the operator or transfer real money, allowing you to test without the appropriate SIM card or network connection. `OnActivityResult` will return successful if your request had the appropriate parameters. Your `TransactionReceiver` will also receive an `Intent` containing the details of the transaction which are possible to supply, including an ID, the parameters you sent, as well as the timestamp and a status of "test". **You cannot use the `test()` method in production, doing so will result in an IllegalState Exception.**

## Transaction result details
The details about a transaction are simply String extras on the data intent. To get the confirmation code just write `data.getStringExtra("code")`.

All possible extras follow, please note that not all requests supply all extras. They depend on what the Mobile Money Operator supplies in the message.

Extra | Description
--- | ---
`operator_name` |
`action` | Should match the action slug by to the request. e.g. "send"
`action_id` | Should match the action id from our supported operators page
`code` | Transaction confirmation code, many MMO's supply these even for balance checks etc.
`currency` | 
`amount` | Amount of money sent or recieved (note that balance is a separate field)
`balance` | User's balance at completion of this transaction
`who` | Recipient or sender
`request_timestamp` | Time user initiated transaction (Unix time)
`response_timestamp` | Time at which the operator confirmed (The timestamp supplied in the confirmation message, Unix time)
`response_message` | Full confirmation message from the operator

## More about verifying payments

The Hover Android SDK:

  * Presents UI to get permission and confirm payment from the user.
  * Coordinates payment with mobile money service.
  * Returns a proof of payment to your app.
  
Your code:

  * Triggers a payment, either when a user interaction happens or on a schedule (e.g. for a subscription).
  * Receives proof of payment from the Hover Android SDK.
  * Sends proof of payment to your servers for verification.
  * Provides the user their goods or services.
