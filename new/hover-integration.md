---
layout: default
nav-class: hover-integration
---

# Add a Hover integration

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
public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
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