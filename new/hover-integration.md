---
layout: default
nav-class: hover-integration
---

# Add a Hover integration

In your `Activity`, add a Hover Integration. You may do this in `onCreate` or when the user presses a button ("Add Payment Method" for example). If you are targeting API 23+ You must use [Run Time Permissions](https://developer.android.com/training/permissions/requesting.html) to first ask for permission to `READ_PHONE_STATE` so that Hover can check the user's SIM card compatability. `HoverIntegration.add()` is necessary for the user to give you permission to use the mobile money service. It will also request that the user enable Accessibility Service if neccessary:

{% highlight java %}
private void addHoverIntegration() { 
	HoverIntegration.add(serviceID, com.hover.sdk.operators.Permission.NORMAL, hoverListener, context);
}

@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  ...
  
  if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_PHONE_STATE) != PackageManager.PERMISSION_GRANTED) {
    ActivityCompat.requestPermissions(this, new String[] { Manifest.permission.READ_PHONE_STATE }, 0);
  } else {
    addHoverIntegration();
  }
}

@Override
public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
  if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
        addHoverIntegration();
  } else {
        // Explain why you need the permission and ask again
      }
}
{% endhighlight %}

The first argument to `HoverIntegration.add()`must be the service ID of one or more of our supported Mobile Money Operators, which you can find [here](https://www.usehover.com/countries/).

The second argument is the permission level. This can be `TEST`, `NORMAL`, or `DIRECT`. TEST mode is for testing your integration in case you do not have access to the SIM card or network you wish your users to use. DIRECT will store your user's PIN so that they do not have to enter it every time to enable, for example, subscription payments. However, in most cases the user will still have to initiate the request due to the way that Android security works.

The third argument can be `null` or an implementation of `HoverIntegration.HoverListener` which provides callbacks for errors or success upon adding the integration. [See more here](http://docs.usehover.com/#hoverintegrationhoverlistener-interface). The final argument is the `Context`.

You may also ask permission to use a list of services. This is useful if you wish to support multiple services in the same country or across multiple countries. This will check if the user has the proper SIM card to support one or more of the services in your list. If so, it will ask the user to choose one of the services supported by their SIM card. The ID of the service they choose will be returned in the `onSuccess()` callback.

{% highlight java %}
HoverIntegration.add(new int[] {8, 13}, Permission.NORMAL, hoverListener, context);
{% endhighlight %}
