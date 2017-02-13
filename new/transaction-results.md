---
layout: default
nav-class: transaction-results
---

# Receive Transaction Confirmation

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
 
## Save proof of payment

Send the proof of payment to your servers for verification. Mobile networks are unreliable. If you can't reach your server, save the proof of payment and try again later.


# Transaction result details
The details about a transaction are simply String extras on the data intent. To get the confirmation code just write `data.getStringExtra("code")`.

All possible extras follow, please note that not all requests supply all extras. They depend on what information the Mobile Money service supplies.

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
