---
layout: post
title: Android 使用Ksoap2向WebService发送byte[]
author: kinshines
date:   2017-08-22
categories: android
permalink: /archivers/android-ksoap2-webservice
---

<p class="lead">Android 开发过程中与Web接口通信时，Server端规定了WebService，虽然不理解这个年代为什么还有人在使用WebService这即将淘汰的技术。在我的印象中，基于Http协议的Web Api早已成主流，通信的数据格式也基本上统一是json了，不过没办法，这个既然人家已经定好了，也只能按照要求搞下去了， </p>



### build.gradle
首先，Android开发中访问WebService主要用到的Java类库是 [Ksoap2](http://ksoap2.sourceforge.net/)

因此主要的build.gradle文件中的dependencies添加依赖

            compile 'com.google.code.ksoap2-android:ksoap2-android:3.6.2'

### Main.xml

{% highlight xml %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android" xmlns:tools="http://schemas.android.com/tools"
android:layout_width="match_parent"
android:layout_height="match_parent" >

<Button
android:id="@+id/button1″
style="?android:attr/buttonStyleSmall"
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:layout_alignParentBottom="true"
android:layout_alignParentLeft="true"
android:layout_alignParentRight="true"
android:text="Button" />

</LinearLayout>
{% endhighlight %}

### MainActivity.java 

{% highlight java %}
package com.example.webview;

import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

import org.ksoap2.SoapEnvelope;
import org.ksoap2.serialization.MarshalBase64;
import org.ksoap2.serialization.SoapObject;
import org.ksoap2.serialization.SoapPrimitive;
import org.ksoap2.serialization.SoapSerializationEnvelope;
import org.ksoap2.transport.HttpTransportSE;

import android.annotation.SuppressLint;
import android.app.Activity;
import android.os.Bundle;
import android.view.Menu;
import android.view.View;
import android.view.View.OnClickListener;
import android.webkit.WebView;
import android.widget.Button;
import android.widget.Toast;

@SuppressLint(“SdCardPath")
public class MainActivity extends Activity {
InputStream is=null;
byte[] array;
@Override

public void onCreate(Bundle savedInstanceState) {

super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);
Button b1= (Button) findViewById(R.id.button1);
String path="/sdcard/BB-sst-animation.gif";
try{
is = new FileInputStream(path);
if (path != null) {
try {
array=streamToBytes(is);
} finally {
is.close();
}
}
}catch (Exception e){
e.printStackTrace();
try {
throw new IOException(“Unable to open R.raw.");
} catch (IOException e1) {
e1.printStackTrace();
}
}

b1.setOnClickListener(new OnClickListener() {

@Override
public void onClick(View v) {
final String SOAP_ACTION = NAMESPACE+ METHOD_NAME;
final String METHOD_NAME="ByteArrayToFile";
final String NAMESPACE ="http://tempuri.org/";
final String URL ="Your WSDL URL";
try{
SoapObject so=new SoapObject(NAMESPACE, METHOD_NAME);
so.addProperty(“BArray", array);
SoapSerializationEnvelope sse=new SoapSerializationEnvelope(SoapEnvelope.VER11);
new MarshalBase64().register(sse);
sse.dotNet=true;
sse.setOutputSoapObject(so);
HttpTransportSE htse=new HttpTransportSE(URL);
htse.call(SOAP_ACTION, sse);
SoapPrimitive response=(SoapPrimitive) sse.getResponse();
String str=response.toString();
System.out.println(“"+str);
Toast.makeText(getApplicationContext(), str, Toast.LENGTH_LONG).show();
} catch (Exception e) {
e.printStackTrace();
}
}
});
}
public static byte[] streamToBytes(InputStream is) {
ByteArrayOutputStream os = new ByteArrayOutputStream(1024);
byte[] buffer = new byte[1024];
int len;
try {
while ((len = is.read(buffer)) >= 0) {
os.write(buffer, 0, len);
}
} catch (java.io.IOException e) {
}
return os.toByteArray();
}
@Override
public boolean onCreateOptionsMenu(Menu menu) {
getMenuInflater().inflate(R.menu.activity_main, menu);
return true;
}

}
{% endhighlight %}

### 说明

因为要向该WebService接口发送byte[]数据，代码中要添加：

        new MarshalBase64().register(sse);

另外，如果WebService的提供方是由.net实现的，代码还需要添加：

        sse.dotNet=true;