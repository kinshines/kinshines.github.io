---
layout: post
title: Java-C#双语配套AES加解密
author: kinshines
date:   2016-11-19
categories: c#
permalink: /archivers/java-csharp-aes
---

<p class="lead">AES(Advanced Encryption Standard)加密作为对称密钥加密中最流行的算法之一，应用在重要的数据接口传输中，然而在对接双方分别使用Java和C#作为编程语言时，对接过程往往会出现各种问题，下面分别是这两种语言的实现，已实现加解密互通。</p>

这里采用的加解密使用base64转码方法，ECB模式，PKCS5Padding填充，无向量，密码必须是16位，否则会报错哈

模式：Java的ECB对应C#的System.Security.Cryptography.CipherMode.ECB

填充方法：Java的PKCS5Padding对应C#System.Security.Cryptography.PaddingMode.PKCS7

编码指定采用UTF-8

### Java

{% highlight java %}

package nb.tmall.util;

import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;

import javax.crypto.*;
import javax.crypto.spec.SecretKeySpec;

import sun.misc.*;

@SuppressWarnings("restriction")
public class EncryptUtil {

    public static String aesEncrypt(String str, String key) throws Exception {
        if (str == null || key == null) return null;
        Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
        cipher.init(Cipher.ENCRYPT_MODE, new SecretKeySpec(key.getBytes("utf-8"), "AES"));
        byte[] bytes = cipher.doFinal(str.getBytes("utf-8"));
        return new BASE64Encoder().encode(bytes);
    }

    public static String aesDecrypt(String str, String key) throws Exception {
        if (str == null || key == null) return null;
        Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
        cipher.init(Cipher.DECRYPT_MODE, new SecretKeySpec(key.getBytes("utf-8"), "AES"));
        byte[] bytes = new BASE64Decoder().decodeBuffer(str);
        bytes = cipher.doFinal(bytes);
        return new String(bytes, "utf-8");
    }
}

{% endhighlight %}


### C\#

{% highlight java %}

using System;
using System.Security.Cryptography;
using System.Text;

namespace CSharp.Util.Security
{
   
        /// <summary>
        ///  AES 加密
        /// </summary>
        /// <param name="str"></param>
        /// <param name="key"></param>
        /// <returns></returns>
        public static string AesEncrypt(string str, string key)
        {
            if (string.IsNullOrEmpty(str)) return null;
            Byte[] toEncryptArray = Encoding.UTF8.GetBytes(str);

            System.Security.Cryptography.RijndaelManaged rm = new System.Security.Cryptography.RijndaelManaged
            {
                Key = Encoding.UTF8.GetBytes(key),
                Mode = System.Security.Cryptography.CipherMode.ECB,
                Padding = System.Security.Cryptography.PaddingMode.PKCS7
            };

            System.Security.Cryptography.ICryptoTransform cTransform = rm.CreateEncryptor();
            Byte[] resultArray = cTransform.TransformFinalBlock(toEncryptArray, 0, toEncryptArray.Length);

            return Convert.ToBase64String(resultArray, 0, resultArray.Length);
        }

        /// <summary>
        ///  AES 解密
        /// </summary>
        /// <param name="str"></param>
        /// <param name="key"></param>
        /// <returns></returns>
        public static string AesDecrypt(string str, string key)
        {
            if (string.IsNullOrEmpty(str)) return null;
            Byte[] toEncryptArray = Convert.FromBase64String(str);

            System.Security.Cryptography.RijndaelManaged rm = new System.Security.Cryptography.RijndaelManaged
            {
                Key = Encoding.UTF8.GetBytes(key),
                Mode = System.Security.Cryptography.CipherMode.ECB,
                Padding = System.Security.Cryptography.PaddingMode.PKCS7
            };

            System.Security.Cryptography.ICryptoTransform cTransform = rm.CreateDecryptor();
            Byte[] resultArray = cTransform.TransformFinalBlock(toEncryptArray, 0, toEncryptArray.Length);

            return Encoding.UTF8.GetString(resultArray);
        }
    }
}

{% endhighlight %}

### 后记

在Java中的AES实现里，可以看到 key 的简单实现是：

        new SecretKeySpec(key.getBytes("utf-8"), "AES")
      
但是网上搜到很多的实现方案如下：

{% highlight java %}
        KeyGenerator kgen = KeyGenerator.getInstance("AES");
        SecureRandom sr = SecureRandom.getInstance("SHA1PRNG");
        sr.setSeed(seed);
        kgen.init(128, sr);
        SecretKey sKey = kgen.generateKey();
{% endhighlight %}

这里实际上是调用了Java内部的密钥生成器来生成密钥，输入一个任意位数的seed，可以生成一个128bit的密钥，但是在C#中没有类似的实现，因此如果Java方使用了以上的密钥生成器来生成的密钥，则需要将密钥生成器生成的密钥告知C#方，双方才能正常通信。

当然，简单的处理方式如同上文，即Java方不要使用密钥生成器，而是直接使用双方约定好的16位的密钥