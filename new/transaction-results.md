---
layout: default
nav-class: transaction-results
---

# Use the result

## 1. Initiate a session

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

## 2. Receive the transaction result

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
 
## 3. Save proof of payment

Send the proof of payment to your servers for verification. Mobile networks are unreliable. If you can't reach your server, save the proof of payment and try again later.

### HoverIntegration.HoverListener Interface

Implement this interface to receive the callbacks:

{% highlight java %}
public interface HoverListener {
  void onSIMError();
  void onUserDenied();
  void onSuccess(String operator, String countryName, String currency);
}
{% endhighlight %}

If on `SIMError` or `UserDenied` are called, then adding the integration has failed. For the former, the user does not have a SIM that supports the integration requested or possibly any SIM at all. For the latter, they did not give permission to use the integration requested. In this case you may show a message explaining why it is neccessary and ask them again by remaking the same call to `HoverIntegration.add()`.

`onSuccess` passes back the operator, country, and currency. This is especially useful if you asked for permission to use any mobile money available to the user, which they choose.

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