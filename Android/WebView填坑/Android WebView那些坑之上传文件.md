###Android WebView那些坑之上传文件

最近公司项目需要在`WebView`上调用手机系统相册来上传图片，开发过程中发现在很多机器上无法正常唤起系统相册来选择图片。

解决问题之前我们先来说说`WebView`上传文件的逻辑：当我们在Web页面上点击选择文件的控件(`<input type="file">`)时，会回调`WebChromeClient`下的`openFileChooser()`（5.0及以上系统回调`onShowFileChooser()`）。这个时候我们在`openFileChooser`方法中通过`Intent`打开系统相册或者支持该`Intent`的第三方应用来选择图片。like this：
    
    public void openFileChooser(ValueCallback<Uri> valueCallback, String acceptType, String capture) {
    	uploadMessage = valueCallback;
       	openImageChooserActivity();
    }
    
    private void openImageChooserActivity() {
        Intent i = new Intent(Intent.ACTION_GET_CONTENT);
        i.addCategory(Intent.CATEGORY_OPENABLE);
        i.setType("image/*");
        startActivityForResult(Intent.createChooser(i, 
        			"Image Chooser"), FILE_CHOOSER_RESULT_CODE);
    }
    
最后我们在`onActivityResult()`中将选择的图片内容通过`ValueCallback`的`onReceiveValue`方法返回给`WebView`，然后通过js上传。代码如下：

	@Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == FILE_CHOOSER_RESULT_CODE) {
            Uri result = data == null || resultCode != RESULT_OK ? null : data.getData();
            if (uploadMessage != null) {
                uploadMessage.onReceiveValue(result);
                uploadMessage = null;
            }
        }
    }

> PS:`ValueCallbacks`是`WebView`组件通过`openFileChooser()`或者`onShowFileChooser()`提供给我们的，它里面包含了一个或者一组`Uri`,然后我们在`onActivityResult()`里将`Uri`传给`ValueCallbacks`的`onReceiveValue()`方法，这样`WebView`就知道我们选择了什么文件。

到这里你可能要问了，说了这么多还是没解释为什么在很多机型上无法唤起系统相册或者第三方app来选择图片啊？！这是因为为了最求完美的Google攻城狮们对`openFileChooser`做了多次修改，在5.0上更是将回调方法该为了`onShowFileChooser`。所以为了解决这一问题，兼容各个版本，我们需要对`openFileChooser()`进行重载，同时针对5.0及以上系统提供`onShowFileChooser()`方法：

	webview.setWebChromeClient(new WebChromeClient() {

            // For Android < 3.0
            public void openFileChooser(ValueCallback<Uri> valueCallback) {
                ***
            }

            // For Android  >= 3.0
            public void openFileChooser(ValueCallback valueCallback, String acceptType) {
                ***
            }

            //For Android  >= 4.1
            public void openFileChooser(ValueCallback<Uri> valueCallback, 
            		String acceptType, String capture) {
                ***
            }

            // For Android >= 5.0
            @Override
            public boolean onShowFileChooser(WebView webView, 
            		ValueCallback<Uri[]> filePathCallback, 
            		WebChromeClient.FileChooserParams fileChooserParams) {
                ***
                return true;
            }
        });
     
大家应该注意到`onShowFileChooser()`中的`ValueCallback`包含了一组`Uri(Uri[])`,所以针对5.0及以上系统，我们还需要对`onActivityResult()`做一点点处理。这里不做描述，最后我再贴上完整代码。

当处理完这些后你以为就万事大吉了？！当初我也这样天真，但当我们打好release包测试的时候却又发现没法选择图片了！！！真是坑了个爹啊！！！无奈去翻`WebChromeClient`的源码，发现`openFileChooser()`是系统API，我们的release包是开启了混淆的，所以在打包的时候混淆了`openFileChooser()`，这就导致无法回调`openFileChooser()`了。

    /**
     * Tell the client to open a file chooser.
     * @param uploadFile A ValueCallback to set the URI of the file to upload.
     *      onReceiveValue must be called to wake up the thread.a
     * @param acceptType The value of the 'accept' attribute of the input tag
     *         associated with this file picker.
     * @param capture The value of the 'capture' attribute of the input tag
     *         associated with this file picker.
     *
     * @deprecated Use {@link #showFileChooser} instead.
     * @hide This method was not published in any SDK version.
     */
    @SystemApi
    @Deprecated
    public void openFileChooser(ValueCallback<Uri> uploadFile, String acceptType, String capture) {
        uploadFile.onReceiveValue(null);
    }
    
解决方案也很简单，直接不混淆`openFileChooser()`就好了。

	-keepclassmembers class * extends android.webkit.WebChromeClient{
   		public void openFileChooser(...);
	}

支持关于上传文件的所有坑都填完了，最后附上完整源码：
(源码地址:[https://github.com/BaronZ88/WebViewSample](https://github.com/BaronZ88/WebViewSample))

    public class MainActivity extends AppCompatActivity {
    
        private ValueCallback<Uri> uploadMessage;
        private ValueCallback<Uri[]> uploadMessageAboveL;
        private final static int FILE_CHOOSER_RESULT_CODE = 10000;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
    
            WebView webview = (WebView) findViewById(R.id.web_view);
            assert webview != null;
            WebSettings settings = webview.getSettings();
            settings.setUseWideViewPort(true);
            settings.setLoadWithOverviewMode(true);
            settings.setJavaScriptEnabled(true);
            webview.setWebChromeClient(new WebChromeClient() {
    
                // For Android < 3.0
                public void openFileChooser(ValueCallback<Uri> valueCallback) {
                    uploadMessage = valueCallback;
                    openImageChooserActivity();
                }
    
                // For Android  >= 3.0
                public void openFileChooser(ValueCallback valueCallback, String acceptType) {
                    uploadMessage = valueCallback;
                    openImageChooserActivity();
                }
    
                //For Android  >= 4.1
                public void openFileChooser(ValueCallback<Uri> valueCallback, String acceptType, String capture) {
                    uploadMessage = valueCallback;
                    openImageChooserActivity();
                }
    
                // For Android >= 5.0
                @Override
                public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, WebChromeClient.FileChooserParams fileChooserParams) {
                    uploadMessageAboveL = filePathCallback;
                    openImageChooserActivity();
                    return true;
                }
            });
            String targetUrl = "file:///android_asset/up.html";
            webview.loadUrl(targetUrl);
        }
    
        private void openImageChooserActivity() {
            Intent i = new Intent(Intent.ACTION_GET_CONTENT);
            i.addCategory(Intent.CATEGORY_OPENABLE);
            i.setType("image/*");
            startActivityForResult(Intent.createChooser(i, "Image Chooser"), FILE_CHOOSER_RESULT_CODE);
        }
    
        @Override
        protected void onActivityResult(int requestCode, int resultCode, Intent data) {
            super.onActivityResult(requestCode, resultCode, data);
            if (requestCode == FILE_CHOOSER_RESULT_CODE) {
                if (null == uploadMessage && null == uploadMessageAboveL) return;
                Uri result = data == null || resultCode != RESULT_OK ? null : data.getData();
                if (uploadMessageAboveL != null) {
                    onActivityResultAboveL(requestCode, resultCode, data);
                } else if (uploadMessage != null) {
                    uploadMessage.onReceiveValue(result);
                    uploadMessage = null;
                }
            }
        }
    
        @TargetApi(Build.VERSION_CODES.LOLLIPOP)
        private void onActivityResultAboveL(int requestCode, int resultCode, Intent intent) {
            if (requestCode != FILE_CHOOSER_RESULT_CODE || uploadMessageAboveL == null)
                return;
            Uri[] results = null;
            if (resultCode == Activity.RESULT_OK) {
                if (intent != null) {
                    String dataString = intent.getDataString();
                    ClipData clipData = intent.getClipData();
                    if (clipData != null) {
                        results = new Uri[clipData.getItemCount()];
                        for (int i = 0; i < clipData.getItemCount(); i++) {
                            ClipData.Item item = clipData.getItemAt(i);
                            results[i] = item.getUri();
                        }
                    }
                    if (dataString != null)
                        results = new Uri[]{Uri.parse(dataString)};
                }
            }
            uploadMessageAboveL.onReceiveValue(results);
            uploadMessageAboveL = null;
        }
    }

源码地址:https://github.com/BaronZ88/WebViewSample

> 如果你喜欢我的文章，就关注下我的**知乎专栏**或者在 GitHub 上添个 Star 吧！
>   
> * 知乎专栏：[https://zhuanlan.zhihu.com/baron](https://zhuanlan.zhihu.com/baron)  
> * GitHub：[https://github.com/BaronZ88](https://github.com/BaronZ88)
        
  







