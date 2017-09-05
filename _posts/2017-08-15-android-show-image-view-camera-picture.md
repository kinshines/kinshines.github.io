---
layout: post
title: Android使用ImageView通过Camera拍摄照片的 file path 呈现图像
author: kinshines
date:   2017-08-15
categories: android
permalink: /archivers/android-show-image-view-from-camera-picture-file-path
---

今天在用Android Studio 调试项目时，想通过ImageView显示刚拍摄的照片，但是一直显示空白，刚开始通过stackoverflow找的的简单解决方案是：

{% highlight java %}
    Bitmap bitmap=BitmapFactory.decodeFile(imageUri.getPath());
    imageView.setImageBitmap(bitmap);
{% endhighlight %}

但仍然无效，近一步搜索得知是摄像机拍摄的照片尺寸过大导致的，需要使用以下方法调整显示大小方可正常显示

{% highlight java %}
    Bitmap bitmap=decodeSampledBitmap(imageUri.getPath());
    imageView.setImageBitmap(bitmap);
{% endhighlight %}

decodeSampledBitmap 及相关函数定义如下：
{% highlight java %}
    private int calculateInSampleSize(
            BitmapFactory.Options options, int reqWidth, int reqHeight) {
        // Raw height and width of image
        final int height = options.outHeight;
        final int width = options.outWidth;
        int inSampleSize = 1;

        if (height > reqHeight || width > reqWidth) {

            final int halfHeight = height / 2;
            final int halfWidth = width / 2;

            // Calculate the largest inSampleSize value that is a power of 2 and keeps both
            // height and width larger than the requested height and width.
            while ((halfHeight / inSampleSize) > reqHeight
                    && (halfWidth / inSampleSize) > reqWidth) {
                inSampleSize *= 2;
            }
        }

        return inSampleSize;
    }

    private Bitmap decodeSampledBitmap(String pathName,
                                       int reqWidth, int reqHeight) {

        // First decode with inJustDecodeBounds=true to check dimensions
        final BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeFile(pathName, options);

        // Calculate inSampleSize
        options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);

        // Decode bitmap with inSampleSize set
        options.inJustDecodeBounds = false;
        return BitmapFactory.decodeFile(pathName, options);
    }

    //I added this to have a good approximation of the screen size:
    private Bitmap decodeSampledBitmap(String pathName) {
        Display display = getWindowManager().getDefaultDisplay();
        Point size = new Point();
        display.getSize(size);
        int width = size.x;
        int height = size.y;
        return decodeSampledBitmap(pathName, width, height);
    }
{% endhighlight %}


最后别忘了还要在manifest文件声明权限噢

                <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
