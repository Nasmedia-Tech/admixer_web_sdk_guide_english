# admixer_web_sdk_guide_english
ADMIXER WEB SDK GUIDE (javascript)
* Please copy the script below, and paste it where you want the ads to be shown.
-----------------------------------
### A. Script Guide
```html
<script type="text/javascript" src="//scr.nsmartad.com/admixer/v3/admixer.js"></script>
<script type="text/javascript">
    new admixer_ad({
        media_key : "AdMixer Media Key",
        adunit :[
            {   //default setting
                adunit_id : "Adunit ID 1",
                display_id : "display_admixer_1" //display area
            },
            {   //default setting & additional setting
                adunit_id : "Adunit ID 2",
                display_id : "display_admixer_2",
                ad_change : true, // Set the refresh in adunit settings
                close_btn : true, // Whether to use ad close button (applied to Adunit, not fullscreen. [default: false - Close button not exposed])
                passback_fn : function(){
                    // If the Admixer advertisement is no-fill, add the passback processing function (do not add it if passback is unnecessary)
                }
            }
        ],
        coppa : 0, // Tag for the service targeting children (0: not targeting children, 1: targeting children)
        admixer_log : true, // Log check setting (default : false[unset],), for log checking, not for loading test ad
        admixer_log_display_id : "display_test_log" // Div ID for the test log to be displayed
    });
</script>
<!-- Insert the div tag into the area where the adunit will be exposed. -->
<div id="display_admixer_1"></div>
<div id="display_admixer_2"></div>
<!-- Insert the div tag in the area where the test log will be displayed. -->
<div id="display_test_log"></div>
```
* __media_key__ : media key issued by the AdMixer platform. (Required)
  - You can find the code at the page 'Media > My media'.
* __adunit__ : Enter the number of Adunits you want to expose. (Required)
* __adunit_id__ : adunit id issued by the AdMixer platform.
  - You can find the code at the page ‘Media > My media > Media info’. (Required)
* __display_id__ : div id into the area where the adunit will be exposed (Required)
* ad_change : Settings for whether to re-request ads when the page has not been changed..
  - true:setting, false:not set
  - default : false
- If set to false, the advertisement will not be recalled until the page is switched.
- If the value is omitted or set to true, the ad is recalled according to the request period set in the ad setting..
  - At the page ‘Media > My Media > Media info’, you can set the request period by clicking the ‘ad setting’ button and ‘ad change’
* close_btn : Ad close button. Use it when you no longer want to expose your ads.
  - true:using the close button, false:unused
  - default : false
  - If set to false or not entered, the close button is not exposed.
  - Applies to Adunit, not Fullscreen (unconditionally exposed for Fullscreen)
* passback_fn : If the AdMixer ad has a nofill response, fill in the passback processing function
  - If you don't need a passback, don't fill it out.
* coppa : Tag for the service targeting children 
  - 0: not targeting children, 1: targeting children
  - default : 0
  - If the item is set to false, it is considered that it is not content for children.
  - If the item is set to true, ads that violate the 'family policy' will be excluded and sent.
* admixer_log : Log check setting
  - true:set, false:unset
  - default : false
  - If you set up admixer_log, the page displays 2 things like below, making it easier to test.
    1) 'Next Ad' button: You can force ads in the next order regardless of the request cycle.
    2) Log: Displays the number of ad calls, receipts, receipts, and failures.
* admixer_log_display_id : Div ID for the test log to be displayed.
  - If admixer_log (log check setting) is true, fill it out.
-------------------------------
## B. When using WEB SDK through WEB VIEW in Android app
* Set javascript to work in webview.
* You must enable local storage capabilities in Webview.
* Prevents landing within the ad area when a new window is displayed in the webview area (_blank processing).
* When you click on an ad, the app link that connects to the app may not work, so separate processing is required.
```java
[WebviewSetting].setJavaScriptEnabled(true);
[WebviewSetting].setDomStorageEnabled(true);
[WebviewSetting].setCacheMode(WebSettings.LOAD_NO_CACHE);
[WebviewSetting].setSupportMultipleWindows(true);
// _blank processing (new window)
[WebviewSetting].setWebChromeClient(new WebChromeClient() {
    @Override
    public boolean onCreateWindow(WebView view, boolean dialog, boolean userGesture, Message resultMsg) {
        WebView newWebView = new WebView(view.getContext());
        WebView.WebViewTransport transport = (WebView.WebViewTransport)resultMsg.obj;
        transport.setWebView(newWebView);
        resultMsg.sendToTarget();
        return true;
    }
});
// App Link Processing
[WebviewSetting].setWebViewClient(new WebViewClient(){
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        if ( view == null || url == null) {
            // Failed to process
            return false;
        }
        if ( url.startsWith("http:") || url.startsWith("https:") ) {
            // HTTP/HTTPS requests are handled internally.
            view.loadUrl(url);
        } else {
            Intent intent;
            try {
                intent = Intent.parseUri(url, Intent.URI_INTENT_SCHEME);
            } catch (URISyntaxException e) {
                // Failed to process
                return false;
            }
            try {
                view.getContext().startActivity(intent);
            } catch (ActivityNotFoundException e) {
                // For Intent Scheme, connect to Market if the app is not installed
                if ( url.startsWith("intent:") && intent.getPackage() != null) {
                    url = "market://details?id=" + intent.getPackage();
                        view.getContext().startActivity(new Intent(Intent.ACTION_VIEW,Uri.parse(url) ));
                        return true;
                } else {
                    // Failed to process
                    return false;
                }
            }
        }
        return true;
    }
});
```
* If you need to move the previous page when you click the Back button, set it as below.
```java
public boolean onKeyDown (int keyCode, KeyEvent event) {
    // move to the previous page
    if (keyCode == KeyEvent.KEYCODE_BACK) {
        // If you have a previous page, go to the previous page
        if ([Webview].canGoBack()){
            [Webview].goBack();
            return false;
        }
    }
    return super.onKeyDown(keyCode, event);
}
```

