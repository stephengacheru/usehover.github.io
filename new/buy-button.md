---
layout: default
nav-class: buy-button
---

# Create a Hover Buy Button

** Note that this is only available in 0.11.1 and up, which is still in alpha. Some services may not work with this version yet **

First [Add an Integration](http://docs.usehover.com/new/hover-integration/)

## Declarative XML

Add the button to your layout just like a normal Button:

{% highlight xml %}
<com.hover.sdk.buttons.BuyButton
            android:id="@+id/buy_something_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:textColor="@color/white"
            android:text="@string/buy_string" />
{% endhighlight %}

A few things to note here: due to issues on some devices, you must define`android:gravity` and `android:textColor`. They can be whatever you want, but they must be there. We also recomend setting a background color, the default is a dark blue. The default text for the button is "Buy", but you can set it to whatever you like.

## Customize Button Parameters

For the button to work you must call `setBuyParameters`. As input it takes a `HoverParameter` object which you get by using the `HoverParameter.Builder`.

{% highlight java %}
BuyButton buyButton = ((BuyButton) findViewById(R.id.hover_button));

buyButton.setBuyParameters(
	new HoverParameters.Builder(activtyInstance)
		.request("send", "100", "Ksh", "+254 55 555 5555")
		.from(8)
		.build());
{% endhighlight %}

See [Create a Transaction](http://docs.usehover.com/new/transactions/) for details about the parameters. The action specified for the buy button must be an action that sends money (Otherwise it wouldn't be a Buy button!).

## Specify Callbacks

The callback functions are not required, but allow you to be notified on successfully sent requests, validation errors, service errors and unexpected errors:

{% highlight java %}
private BuyButtonCallback mBtnCallbacks = new BuyButtonCallback() {
	@Override public void onError(Throwable throwable) {
		// Unexpected error
	}

	@Override public void onValidationError(String s) {
		// Validation error details: [Create page]()
	}

	@Override public void onServiceError(String s) {
		// Error during session. Possibly a low balance error or similar. Show the user the message so they can attempt to fix it.
	}

	@Override public void onServiceProcessing(String s, int i) {
		// Mobile Money operator is processing the request. The String s is the message from the operator. We recommend showing this message to the user so that they can see what is happening.
	}
};
  
  buyButton.setCallback(mBtnCallbacks, requestCode);
{% endhighlight %}

When the button is clicked it fires an Intent to Hover. If `requestCode` is specified (and the button is attached to an Activity) then you can catch the response by calling `onActivityResult` when your activity gets the result, which will subsequently call the appropriate callbacks:

{% highlight java %}
@Override
protected void onActivityResult (int requestCode, int resultCode, Intent data) {
	super.onActivityResult(requestCode, resultCode, data);
	buyButton.onActivityResult(requestCode, resultCode, data);
}
{% endhighlight %}

## Receive Transaction Result

Finally, [Receive the Result](http://docs.usehover.com/new/transaction-results/) of the buy action.
