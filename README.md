Draugiem.lv Android SDK
===========
Android SDK for authorisation and payments via https://draugiem.lv.


How to set up SDK?
===============
### 0. Create your draugiem.lv application, if you haven't yet.

Navigate to your Draugiem developer page ([Latvian](https://www.draugiem.lv/applications/dev/myapps/) and [English](https://www.frype.com/applications/dev/myapps/)) and create a new application.
Fill in the details. You should end up with something like this:

![App creation form](https://user-images.githubusercontent.com/100644/90143432-48812200-dd86-11ea-8139-14985381aac6.png)

1. Take note of your _Application API key_ (Latvian: _Aplikācijas API atslēga_; `0360f381d755cd966e3c4a98c5664e63` in the example).
2. Generate hash code for your keystores using this command:

* for debug build: `keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64`
* release build: `keytool -exportcert -alias <YOUR_RELEASE_KEY_ALIAS> -keystore <YOUR_RELEASE_KEY_PATH> | openssl sha1 -binary | openssl base64`

The results would look like this: `xC/AvLAFNbAR61n4E2+VNQzEKyI=`. Add those to the "Android key hash" field on the app settings page.

**Important**! After you add full information, you should write to Draugiem support (api@draugiem.lv) to get your app approved. If you plan to support users' payments via Draugiem wallet, ask the support to enable it.

### 1. Setup SDK in your application

To authorize your application's user in Draugiem you would need to include Draugiem SDK dependency:

```
implementation 'lv.draugiem:AndroidSDK:1.0.2-SNAPSHOT'
```
    
Instantiate Draugiem SDK in your activity with the aforementioned application API key:
```
private DraugiemAuth draugiemSDK = new DraugiemAuth(<app API key>, this);
```

### 2. Set up authorisation:
1. Add click listener and authorization from cache
```
mAuthorizeButton.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                draugiemSDK.authorize(authCallback);
            }
        });
```
If you were logged in the past, you may get user and token via this call:
```
draugiemSDK.authorizeFromCache(authCallback);
```
2. Add onActivityResult() to the same activity
```
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (draugiemSDK.onActivityResult(requestCode, resultCode, data)) {
        //some of draugiem intentns were captured
        if (resultCode == RESULT_CANCELED) {
            Log.e("FF", "LKDJH");
        }
    }
}
```
3. Add authorization callback
```
authCallback = new AuthCallback() {
    @Override
    public void onLogin(User u, String apikey) {
        // in case of successfull login, you recieve user object
    }

    @Override
    public void onError() {
        // something went wrong
    }

    @Override
    public void onNoApp() {
        // user hasn't Draugiem app installed -> open in Google Play
        try {
            startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("market://details?id=com.draugiem2")));
        } catch (android.content.ActivityNotFoundException anfe) {
            startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("http://play.google.com/store/apps/details?id=com.draugiem2")));
        }
    }
};
```
4. You can always logout: ```draugiemSDK.logout()```

### 3. Set up payments:

0. Make sure that Draugiem support has enabled payments for your app: in the left menu of app settings you should have **Paid services** line.
1. There are 2 types of payments: fixed price (could be paid with SMS or Draugiem wallet) and with dynamic price (price should be passed dynamically).

![Two types of payments](https://user-images.githubusercontent.com/100644/90149895-b41abd80-dd8d-11ea-96b1-5db4e10d23c0.png)

2. Initiate payment with `DraugiemAuth.getTransactionId()` by passing payment ID and price (A. `null` if this payment has fixed price or B. euro cents, if this payment has dynamic price). You will get transcation ID in a callback.
```
draugiemSDK.getTransactionId(<payment ID>, <price>, new TransactionCallback() {
    @Override
    public void onTransaction(final int id, String url) {
        // here you get transcation ID and url with iframe if no Draugiem app installed
    }
    @Override
    public void onError(int code, String error) {
        // TODO show error;
    }
});
```
3. Delegate the wallet/SMS purchase to Dragiem app via `DraugiemAuth.payment()`:
```
draugiemSDK.payment(<transaction ID>, new PaymentCallback(){
    @Override
    public void onSuccess() {
        Toast.makeText(MainActivity.this, "Payment succeeded", Toast.LENGTH_LONG).show();
    }

    @Override
    public void onError(String error) {
        // error occurred
    }

    @Override
    public void onNoApp() {
        // user hasn't Draugiem app installed -> open in Google Play
        try {
            startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("market://details?id=com.draugiem2")));
        } catch (android.content.ActivityNotFoundException anfe) {
            startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("http://play.google.com/store/apps/details?id=com.draugiem2")));
        }
    }
    @Override
    public void onPossibleSms() {
        draugiemSDK.checkTransaction(<transaction ID>, 5, new TransactionCheckCallback(){
            @Override
            public void onOk() {
                Toast.makeText(MainActivity.this, "Payment succeeded", Toast.LENGTH_LONG).show();
                //TODO add service
            }

            @Override
            public void onFailed() {
                Toast.makeText(MainActivity.this, "Payment failed", Toast.LENGTH_LONG).show();
            }

            @Override
            public void onStopChecking() {
                Toast.makeText(MainActivity.this, "Stop checking transaction", Toast.LENGTH_LONG).show();
            }
        });
    }

    @Override
    public void onUserCanceled() {
        // user canceled
    }
});
```

Don't forget to pass the result intent inside `onActivityResult()` the same way it was done for authorisation.
