---
layout: default
nav-class: transactions
---

# Make a transaction

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