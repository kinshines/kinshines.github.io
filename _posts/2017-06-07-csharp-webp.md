---
layout: post
title: C#中生成webp格式的图片
author: kinshines
date:   2017-06-07
categories: c#
permalink: /archivers/csharp-webp
---

<p class="lead">在将普通图片格式转换成webp格式时，之前一直是使用命令行的方式调用<a href="https://storage.googleapis.com/downloads.webmproject.org/releases/webp/index.html">cwebp.exe</a>，cwebp.exe<a href="https://storage.googleapis.com/downloads.webmproject.org/releases/webp/index.html">下载地址</a>.本文将尝试在C#中直接调用libwebp.dll的方式来生成webp格式的图片</p>

### 编译或下载libwebp.dll

由于谷歌只提供了libwebp.lib，目前最新的版本是0.6.0，需要我们自行编译成libwebp.dll以便C#调用，或者从以下地址直接下载：
[x86_64/libwebp.dll](https://s3.amazonaws.com/resizer-dynamic-downloads/webp/0.6.0/x86_64/libwebp.dll)
[x86/libwebp.dll](https://s3.amazonaws.com/resizer-dynamic-downloads/webp/0.5.2/x86/libwebp.dll)

### 调用libwebp.dll的中间类
{% highlight java %}
/////////////////////////////////////////////////////////////////////////////////////////////////////////////
/// Warper for WebP format in c#. (GPL) 
///////////////////////////////////////////////////////////////////////////////////////////////////////////// 
/// Main functions:
/// Save - Save a bitmap in WebP file.
/// Load - Load a WebP file in bitmap.
/// Decode - Decode WebP data (in byte array) to bitmap.
/// Encode - Encode bitmap to WebP (return a byte array). 
/// 
/// Another functions:
/// EncodeLossly - Encode bitmap to WebP with quality lost (return a byte array).
/// EncodeLossless - Encode bitmap to WebP without quality lost (return a byte array).
/////////////////////////////////////////////////////////////////////////////////////////////////////////////
using System;
using System.Drawing;
using System.Drawing.Imaging;
using System.IO;
using System.Runtime.InteropServices;


namespace WebP
{
    class clsWebP
    {
        /// <summary>Save bitmap to file in WebP format</summary>
        /// <param name="bmp">Bitmap with the WebP image</param>
        /// <param name="quality">Quality. 0 = minumin ... 100 = maximimun quality</param>
        /// <param name="pathFileName">The file to write</param>
        /// <returns>True if success; False otherwise</returns>
        public static bool Save(Bitmap bmp, int quality, string pathFileName)
        {
            byte[] dataWebP;

            try
            {
                //Encode in webP format
                if (!EncodeLossly(bmp, quality, out dataWebP)) return false;

                //Write webP file
                File.WriteAllBytes(pathFileName, dataWebP);

                return true;
            }
            catch (Exception ex) { return false; }
        }

        /// <summary>Read a WebP file</summary>
        /// <param name="pathFileName">WebP file to load</param>
        /// <param name="bmp">Bitmap with the WebP image</param>
        /// <returns>True if success; False otherwise</returns>
        public static bool Load(string pathFileName, out Bitmap bmp)
        {
            bool result;
            byte[] dataWebP;
            bmp = null;

            try
            {
                //Read webP file
                dataWebP = File.ReadAllBytes(pathFileName);

                result = Decode(dataWebP, out bmp);

                return result;
            }
            catch (Exception ex) { return false; }
        }

        /// <summary>Decode a WebP image</summary>
        /// <param name="webpData">the data to uncompress</param>
        /// <param name="bmp">Bitmap whit the image</param>
        /// <returns>True if success; False otherwise</returns>
        public static bool Decode(byte[] webpData, out Bitmap bmp)
        {
            int imgWidth;
            int imgHeight;
            IntPtr outputBuffer;
            int outputBufferSize;
            bmp = null;

            try
            {
                //Get image width and height
                GCHandle pinnedWebP = GCHandle.Alloc(webpData, GCHandleType.Pinned);
                IntPtr ptrData = pinnedWebP.AddrOfPinnedObject();
                UInt32 dataSize = (uint)webpData.Length;
                if (WebPGetInfo(ptrData, dataSize, out imgWidth, out imgHeight) != 1) return false;

                //Create a BitmapData and Lock all pixels to be written
                bmp = new Bitmap(imgWidth, imgHeight, PixelFormat.Format24bppRgb);
                BitmapData bmpData = bmp.LockBits(new Rectangle(0, 0, bmp.Width, bmp.Height), ImageLockMode.WriteOnly, bmp.PixelFormat);

                //Allocate memory for uncompress image
                outputBufferSize = bmpData.Stride * imgHeight;
                outputBuffer = Marshal.AllocHGlobal(outputBufferSize);

                //Uncompress the image
                outputBuffer = WebPDecodeBGRInto(ptrData, dataSize, outputBuffer, outputBufferSize, bmpData.Stride);

                //Write image to bitmap using Marshal
                byte[] buffer = new byte[outputBufferSize];
                Marshal.Copy(outputBuffer, buffer, 0, outputBufferSize);
                Marshal.Copy(buffer, 0, bmpData.Scan0, outputBufferSize);

                //Write image to bitmap using CopyMemory. Faster than Marshall, but only work in windows
                //CopyMemory(bmpData.Scan0, outputBuffer, (uint)outputBufferSize);

                //Unlock the pixels
                bmp.UnlockBits(bmpData);

                //Free memory
                pinnedWebP.Free();
                Marshal.FreeHGlobal(outputBuffer);

                return true;
            }
            catch (Exception ex) { return false; }
        }
        
        /// <summary>Write a WebP file in minimun size</summary>
        /// <param name="webpData">Bitmap to encode</param>
        /// <param name="quality">Quality. 0 = minumin ... 100 = maximimun quality</param>
        /// <param name="bmp">Bitmap with the image</param>
        /// <returns>True if success; False otherwise</returns>
        public static bool Encode(Bitmap bmp, int quality, out byte[] webpData)
        {
            byte[] lossly;
            byte[] lossless;
            webpData = null;
            
            try
            {
                //compress in two metods
                if (!EncodeLossly(bmp, quality, out lossly)) return false;
                if (!EncodeLossless(bmp, out lossless)) return false;

                if (lossly.Length >= lossless.Length)
                    webpData = lossless;
                else
                    webpData = lossly;

                return true;
            }
            catch (Exception ex) { return false; }
        }

        /// <summary>Lossly encoding image in bitmap</summary>
        /// <param name="bmp">Bitmap with the image</param>
        /// <param name="quality">Quality. 0 = minumin ... 100 = maximimun quality</param>
        /// <param name="webpData">Compress data</param>
        /// <returns>True if success; False otherwise</returns>
        public static bool EncodeLossly(Bitmap bmp, int quality, out byte[] webpData)
        {
            BitmapData bmpData;
            IntPtr unmanagedData;
            int size;
            webpData = null;

            try
            {
                bmpData = bmp.LockBits(new Rectangle(0, 0, bmp.Width, bmp.Height), ImageLockMode.ReadOnly, PixelFormat.Format24bppRgb);
                size = WebPEncodeBGR(bmpData.Scan0, bmp.Width, bmp.Height, bmpData.Stride, quality, out unmanagedData);

                //Copy image compress data to output array
                webpData = new byte[size];
                Marshal.Copy(unmanagedData, webpData, 0, size);

                //Unlock the pixels
                bmp.UnlockBits(bmpData);

                //Free memory
                WebPFree(unmanagedData);

                return true;
            }
            catch (Exception ex) { return false; }
        }

        /// <summary>Lossless encoding image in bitmap</summary>
        /// <param name="bmp">Bitmap with the image</param>
        /// <param name="webpData">Compress data</param>
        /// <returns>True if success; False otherwise</returns>
        public static bool EncodeLossless(Bitmap bmp, out byte[] webpData)
        {
            BitmapData bmpData;
            IntPtr unmanagedData;
            int size;
            webpData = null;

            try
            {
                //Get bmp data
                bmpData = bmp.LockBits(new Rectangle(0, 0, bmp.Width, bmp.Height), ImageLockMode.ReadOnly, PixelFormat.Format24bppRgb);

                //Compress the bmp data
                size = WebPEncodeLosslessBGR(bmpData.Scan0, bmp.Width, bmp.Height, bmpData.Stride, out unmanagedData);

                //Copy image compress data to output array
                webpData = new byte[size];
                Marshal.Copy(unmanagedData, webpData, 0, size);

                //Unlock the pixels
                bmp.UnlockBits(bmpData);

                //Free memory
                WebPFree(unmanagedData);

                return true;
            }
            catch (Exception ex) { return false; }
        }

        /// <summary>Validate the WebP image header and retrieve the image height and width. Pointers *width and *height can be passed NULL if deemed irrelevant</summary>
        /// <param name="data">Pointer to WebP image data</param>
        /// <param name="data_size">This is the size of the memory block pointed to by data containing the image data</param>
        /// <param name="width">The range is limited currently from 1 to 16383</param>
        /// <param name="height">The range is limited currently from 1 to 16383</param>
        /// <returns>1 if success, otherwise error code returned in the case of (a) formatting error(s).</returns>
        [DllImport("libwebp.dll", CallingConvention = CallingConvention.Cdecl)]
        private static extern int WebPGetInfo(IntPtr data, UInt32 data_size, out int width, out int height);

        /// <summary>Decode a WebP image pointed to by data</summary>
        /// <param name="data">Pointer to WebP image data</param>
        /// <param name="data_size">This is the size of the memory block pointed to by data containing the image data</param>
        /// <param name="width">The range is limited currently from 1 to 16383</param>
        /// <param name="height">The range is limited currently from 1 to 16383</param>
        /// <returns>output_buffer if function succeeds; NULL otherwise</returns>
        [DllImport("libwebp.dll", CallingConvention = CallingConvention.Cdecl)]
        private static extern IntPtr WebPDecodeBGR(IntPtr data, UInt32 data_size, ref int width, ref int height);

        /// <summary>Decode WEBP image pointed to by *data and returns BGR samples into a pre-allocated buffer</summary>
        /// <param name="data">Pointer to WebP image data</param>
        /// <param name="data_size">This is the size of the memory block pointed to by data containing the image data</param>
        /// <param name="output_buffer">Pointer to decoded WebP image</param>
        /// <param name="output_buffer_size">Size of allocated buffer</param>
        /// <param name="output_stride">Specifies the distance between scanlines</param>
        /// <returns>output_buffer if function succeeds; NULL otherwise</returns>
        [DllImport("libwebp.dll", CallingConvention = CallingConvention.Cdecl)]
        private static extern IntPtr WebPDecodeBGRInto(IntPtr data, UInt32 data_size, IntPtr output_buffer, int output_buffer_size, int output_stride);

        /// <summary>Lossless encoding images pointed to by *data in WebP format</summary>
        /// <param name="rgb">Pointer to RGB image data</param>
        /// <param name="width">The range is limited currently from 1 to 16383</param>
        /// <param name="height">The range is limited currently from 1 to 16383</param>
        /// <param name="output_stride">Specifies the distance between scanlines</param>
        /// <param name="quality_factor">Ranges from 0 (lower quality) to 100 (highest quality). Controls the loss and quality during compression</param>
        /// <param name="output">output_buffer with WebP image</param>
        /// <returns>Size of WebP Image</returns>
        [DllImport("libwebp.dll", CallingConvention = CallingConvention.Cdecl)]
        private static extern int WebPEncodeBGR(IntPtr rgb, int width, int height, int stride, float quality_factor, out IntPtr output);

        /// <summary>Lossless encoding images pointed to by *data in WebP format</summary>
        /// <param name="rgb">Pointer to RGB image data</param>
        /// <param name="width">The range is limited currently from 1 to 16383</param>
        /// <param name="height">The range is limited currently from 1 to 16383</param>
        /// <param name="output_stride">Specifies the distance between scanlines</param>
        /// <param name="quality_factor">Ranges from 0 (lower quality) to 100 (highest quality). Controls the loss and quality during compression</param>
        /// <param name="output">output_buffer with WebP image</param>
        /// <returns>Size of WebP Image</returns>
        [DllImport("libwebp.dll", CallingConvention = CallingConvention.Cdecl)]
        private static extern int WebPEncodeLosslessBGR(IntPtr rgb, int width, int height, int stride, out IntPtr output);

        [DllImport("libwebp.dll", CallingConvention = CallingConvention.Cdecl)]
        private static extern int WebPFree(IntPtr p);

        //Faster copy, but only work in windows
        //[DllImport("kernel32.dll", EntryPoint = "CopyMemory")]
        //private static extern void CopyMemory(IntPtr Destination, IntPtr Source, uint Length);
    }
}

{% endhighlight %}

### Console Demo
{% highlight java %}
class Program
    {
        static void Main(string[] args)
        {

            //Test JPG to WebP
            Bitmap bmp1 = new Bitmap("test.jpg");
            clsWebP.Save(bmp1, 80, "test.webp");

            //Test WebP to PNG
            Bitmap bmp2;
            clsWebP.Load("test.webp", out bmp2);
            bmp2.Save("test.png", ImageFormat.Png);

            //Test JPG to WebP in lossless mode. Using compress in memory
            byte[] webpImageData1;
            Bitmap bmp3 = new Bitmap("test.jpg");
            clsWebP.EncodeLossless(bmp3, out webpImageData1);
            File.WriteAllBytes("lossless.webp", webpImageData1);

            //Test JPG to WebP in lossly mode. Using encode in memory
            byte[] webpImageData2;
            Bitmap bmp4 = new Bitmap("test.jpg");
            clsWebP.EncodeLossly(bmp4, 80, out webpImageData2);
            File.WriteAllBytes("lossly.webp", webpImageData2);

            //Test WebP to PNG. Using decode in memory
            Bitmap bmp5;
            byte[] webpImageData3 = File.ReadAllBytes("lossless.webp");
            clsWebP.Decode(webpImageData3, out bmp5);
            bmp4.Save("test2.png", ImageFormat.Png);

            //Test WebP to pictureBox
            Bitmap bmp6;
            clsWebP.Load("test.webp", out bmp6);
            pictureBox.Image = bmp6;
        }
    }
{% endhighlight %}
