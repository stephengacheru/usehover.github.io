---
layout: default
nav-class: overview
---

# Welcome to Hover

**Hover is currently in Alpha and may be unstable. If you have issues please get in touch with us.**

## Introduction

Hover is an Android SDK which allows an Android app to use supported Mobile Money wallets to pay for goods and services in-app. A developer can initiate a payment when a User presses a button or takes another action, or can create a subscription. Hover will deal with initiating the transaction, including getting the user's permission, and returning the confirmation back to the app. This means that apps can treat a Mobile Money payment just as they would a credit card payment.


## Using the SDK

Using Hover consists of just a few steps:

### 0. Installation

Hover uses Maven, so installation is as simple as any other Android library.

### 1. Integrate a Mobile Money or Telecom Service

Choose one or more services to integrate from our [list](https://www.usehover.com/countries/). Hover will check that the user's SIM card works with at least one of the services you choose. The user will be asked to give you permission to use the matching mobile money service.

### 2. Create a transaction request

Once you have permission you can make any request supported by the service. This consists in building an Intent and calling `startActivity()` or `startActivityForResult()` for Hover to execute it. If the service requires a PIN to execute the request, then Hover will present the user with a dialog to enter their PIN. This is stored encrypted on the user's device, never leaves, and is deleted as soon as it is used. That way you, the developer, does not have to worry about security.

### 3. Check the progress 

Once Hover makes the request it will return a result indicating whether the service is processing the request or not. Most services take a short time to process. Once they have finished, Hover returns the result asynchronously (see next step). If there are validation problems with the request Hover will return an error here instead of sending the request on to the service. Hover will also return an error here if the service returns an error before completing the request. Note that this step is only available if you start Hover from an Activity using `startActivityForResult()`, which is therefore highly recommended.

### 4. Receive the transaction result

Hover receives and processes the result from the service asynchronously then sends a broadcast with the confirmation message. The broadcast contains the request details, such as the transaction code from the service, the timestamp, and other request specific details. Once this broadcast has been received you can release the good or service that the user paid for, show them their balance, or whatever else a successful request should trigger.

### 5. Save proof of payment

Send the proof of payment to your servers for verification. Mobile networks are unreliable. If you can't reach your server, save the proof of payment and try again later.


## More about verifying payments

The Hover Android SDK:

  * Presents UI to get permission and confirm payment from the user.
  * Coordinates payment with mobile money service.
  * Returns a proof of payment to your app.
  
Your code:

  * Triggers a payment, either when a user interaction happens or on a schedule (e.g. for a subscription).
  * Receives proof of payment from the Hover Android SDK.
  * Provides the user their goods or services.
  * Sends proof of payment to your servers for verification.
