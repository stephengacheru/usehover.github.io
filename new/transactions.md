---
layout: default
nav-class: transactions
---

# Create a Transaction

## Create and Send the Request
Create a request and launch the intent, for example, when a button is pressed: 

{% highlight java %}
Intent i = new Hover.Builder(this).request("send", amount, "Tsh", recipient).from(13);
startActivityForResult(i, 0);
{% endhighlight %}

To use `startActivityForResult()`, `this` must be an instance of an `Activity`, otherwise you should simply use `startActivty` (which will not tell you whether there were validation errors or whether your request is being processed). In most cases you must supply arguments to `Hover.Builder` which are passed on to the Mobile Money Service. The required arguments for each request are included on the [Supported Operators page](https://www.usehover.com/countries/). In most cases you supply extra arguments using the `extra()` method:

{% highlight java %}
Intent i = new Hover.Builder(this).request("pay_bill", amount, "Tsh").extra("who", recipient_business_no).extra("acct", acct_no).from(13);
{% endhighlight %}

For convenience the `request()` method optionally takes the `amount`, `currency`, and `recipient`. There are also convience methods `amount(amount, currency)` and `to(recipient)`.

## Check if Request Went Through

Implement `onActivityResult()` to get the outcome of the request (NOT confirmation of the transaction). If the `resultCode` is `RESULT_OK` then as far as the SDK can tell the request was accepted and is being processed by the Mobile Money Operator. Implement the [Transaction Broadcast Receiver](http://docs.usehover.com/new/transaction-results/) to receive the transaction confirmation. If the result code is `RESULT_CANCELED` then something went wrong and you should not expect the request to succeed. In this case the data intent returned will have a String Extra called `result` which will contain the error message. 

There are 2 types of failure: If there is a validation error then Hover does not make request to the Mobile Money Service. There will be no transaction and it will not count against your Hover credits. However, if the request failed after a request was actually made to the Mobile Money service then the data intent returned will include a `transaction_id` and the request will count against your Hover credits. This can happen because the user had insufficient balance, the amount sent was under the minimum amount required by the operator, or a host of other reasons. We recomend that in this case the `result` extra is shown to the user so that they can find out what went wrong.

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
