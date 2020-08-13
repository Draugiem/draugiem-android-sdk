Draugiem.lv Android SDK
===========
Draugiem.lv Android SDK for authorisation and payments


Setup steps
===============
### 0. Create your draugiem.lv application, if you haven't yet.

Navigate to [your draugiem developer page](https://www.draugiem.lv/applications/dev/myapps/) and create a new application.
Fill in the details. You should end up with something like this:

![App creation form](https://raw.githubusercontent.com/aigarssilavs/DraugiemSDK/master/Documents/appCreationForm.png)

1. Take note of your application API key, (aplikācijas API atslēga: `068411db50ed4d0de895d4405461f112` in the example).
2. Generate hash code from you release APK (the same way) and add it to the field on the same page:

`keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64`

### 1. Setup SDK in your application

To authorize your application's user in Draugiem you would need to include Draugiem SDK dependency:

```
implementation 'lv.draugiem:AndroidSDK:1.0.1'
```
    
Instantiate Draugiem SDK in your activity:
```
private DraugiemAuth draugiemSDK = new DraugiemAuth(APP, this);
```

To implement authorize:
1. Add click listener and authorization from cache
```
mAuthorizeButton.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                draugiemSDK.authorize(authCallback);
            }
        });
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
        // open Draugiem in Google Play
        try {
            startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("market://details?id=com.draugiem2")));
        } catch (android.content.ActivityNotFoundException anfe) {
            startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("http://play.google.com/store/apps/details?id=com.draugiem2")));
        }
    }
};
```

You can find sample app here https://github.com/Draugiem/draugiem-android-sdk-sampleApp
