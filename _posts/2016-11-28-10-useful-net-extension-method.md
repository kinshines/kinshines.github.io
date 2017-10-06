---
layout: post
title: 10个实用的.NET扩展方法
author: kinshines
date:   2016-11-28
categories: c#
permalink: /archivers/10-useful-net-extension-method
---

<p class="lead">扩展方法是C# 3.0的新特性，它可以扩展一个类的方法，也可以使得编码更简单，本文将介绍10个常用的扩展方法</p>

### 扩展方法的要求
* 类必须是 static
* 方法必须是 static
* 方法的第一个参数必须是以this 来声明
* 范围必须是pulic 

下面是10个常用的扩展方法
### 1. ToFileSize - Type: Long
    {% highlight java %}
    public static class MiscExtensions
    {
        public static string ToFileSize(this long size)
        {
            if (size < 1024) { return (size).ToString("F0") + " bytes"; }
            if (size < Math.Pow(1024, 2)) { return (size/1024).ToString("F0") + "KB"; }
            if (size < Math.Pow(1024, 3)) { return (size/Math.Pow(1024, 2)).ToString("F0") + "MB"; }
            if (size < Math.Pow(1024, 4)) { return (size/Math.Pow(1024, 3)).ToString("F0") + "GB"; }
            if (size < Math.Pow(1024, 5)) { return (size/Math.Pow(1024, 4)).ToString("F0") + "TB"; }
            if (size < Math.Pow(1024, 6)) { return (size/Math.Pow(1024, 5)).ToString("F0") + "PB"; }
            return (size/Math.Pow(1024, 6)).ToString("F0") + "EB";
        }
    }
    {% endhighlight %}

### 2. ToXmlDocument()/ToXDocument() - Type: XDocument or XmlDocument
    {% highlight java %}
    public static class XmlDocumentExtensions
{
    public static XmlDocument ToXmlDocument(this XDocument xDocument)
    {
        var xmlDocument = new XmlDocument();
        using (var xmlReader = xDocument.CreateReader())
        {
            xmlDocument.Load(xmlReader);
        }
        return xmlDocument;
    }
    public static XDocument ToXDocument(this XmlDocument xmlDocument)
    {
        using (var nodeReader = new XmlNodeReader(xmlDocument))
        {
            nodeReader.MoveToContent();
            return XDocument.Load(nodeReader);
        }
    }
    public static XmlDocument ToXmlDocument(this XElement xElement)
    {
        var sb = new StringBuilder();
        var xws = new XmlWriterSettings {OmitXmlDeclaration = true, Indent = false};
        using (var xw = XmlWriter.Create(sb, xws))
        {
            xElement.WriteTo(xw);
        }
        var doc = new XmlDocument();
        doc.LoadXml(sb.ToString());
        return doc;
    }
    public static Stream ToMemoryStream(this XmlDocument doc)
    {
        var xmlStream = new MemoryStream();
        doc.Save(xmlStream);
        xmlStream.Flush();//Adjust this if you want read your data 
        xmlStream.Position = 0;
        return xmlStream;
    }
}
    {% endhighlight %}

### 3. RemoveLast()/RemoveLastCharacter()/RemoveFirst()/RemoveFirstCharacter() - Type: String
{% highlight java %}
public static string RemoveLastCharacter(this String instr)
{
    return instr.Substring(0, instr.Length - 1);
}
public static string RemoveLast(this String instr, int number)
{
    return instr.Substring(0, instr.Length - number);
}
public static string RemoveFirstCharacter(this String instr)
{
    return instr.Substring(1);
}
public static string RemoveFirst(this String instr, int number)
{
    return instr.Substring(number);
}
{% endhighlight %}

### 4. Between() - Type: DateTime
{% highlight java %}
public static bool Between(this DateTime dt, DateTime rangeBeg, DateTime rangeEnd)
{
    return dt.Ticks >= rangeBeg.Ticks && dt.Ticks <= rangeEnd.Ticks;
}
{% endhighlight %}

### 5. CalculateAge() - Type: DateTime
{% highlight java %}
public static int CalculateAge(this DateTime dateTime)
{
    var age = DateTime.Now.Year - dateTime.Year;
    if (DateTime.Now < dateTime.AddYears(age))
        age--;
   return age;
}
{% endhighlight %}

### 6. ToReadableTime() - Type: DateTime
{% highlight java %}
public static string ToReadableTime(this DateTime value)
{
    var ts = new TimeSpan(DateTime.UtcNow.Ticks - value.Ticks);
    double delta = ts.TotalSeconds;
    if (delta < 60)
    {
        return ts.Seconds == 1 ? "one second ago" : ts.Seconds + " seconds ago";
    }
    if (delta < 120)
    {
        return "a minute ago";
    }
    if (delta < 2700) // 45 * 60
    {
        return ts.Minutes + " minutes ago";
    }
    if (delta < 5400) // 90 * 60
    {
        return "an hour ago";
    }
    if (delta < 86400) // 24 * 60 * 60
    {
        return ts.Hours + " hours ago";
    }
    if (delta < 172800) // 48 * 60 * 60
    {
        return "yesterday";
    }
    if (delta < 2592000) // 30 * 24 * 60 * 60
    {
        return ts.Days + " days ago";
    }
    if (delta < 31104000) // 12 * 30 * 24 * 60 * 60
    {
        int months = Convert.ToInt32(Math.Floor((double)ts.Days / 30));
        return months <= 1 ? "one month ago" : months + " months ago";
    }
    var years = Convert.ToInt32(Math.Floor((double)ts.Days / 365));
    return years <= 1 ? "one year ago" : years + " years ago";
}
{% endhighlight %}

### 7. WorkingDay()/IsWeekend()/NextWorkday() - Type: DateTime
{% highlight java %}
public static bool WorkingDay(this DateTime date)
{
    return date.DayOfWeek != DayOfWeek.Saturday && date.DayOfWeek != DayOfWeek.Sunday;
}
public static bool IsWeekend(this DateTime date)
{
    return date.DayOfWeek == DayOfWeek.Saturday || date.DayOfWeek == DayOfWeek.Sunday;
}
public static DateTime NextWorkday(this DateTime date)
{
    var nextDay = date;
    while (!nextDay.WorkingDay())
    {
        nextDay = nextDay.AddDays(1);
    }
    return nextDay;
}

{% endhighlight %}

### 8. Next() - Type: DateTime
{% highlight java %}
public static DateTime Next(this DateTime current, DayOfWeek dayOfWeek)
{
    int offsetDays = dayOfWeek - current.DayOfWeek;
    if (offsetDays <= 0)
    {
        offsetDays += 7;
    }
    DateTime result = current.AddDays(offsetDays);
    return result;
}

{% endhighlight %}

### 9. str.ToStream()/stream.ToString()/CopyTo() - Type: String/Stream
{% highlight java %}
public static Stream ToStream(this string str)
{
    byte[] byteArray = Encoding.UTF8.GetBytes(str);
    //byte[] byteArray = Encoding.ASCII.GetBytes(str);
    return new MemoryStream(byteArray);
}
public static string ToString(this Stream stream)
{
    var reader = new StreamReader(stream);
    return reader.ReadToEnd();
}
/// <summary>
/// Copy from one stream to another.
/// Example:
/// using(var stream = response.GetResponseStream())
/// using(var ms = new MemoryStream())
/// {
///     stream.CopyTo(ms);
///      // Do something with copied data
/// }
/// </summary>
/// <param name="fromStream">From stream.</param>
/// <param name="toStream">To stream.</param>
public static void CopyTo(this Stream fromStream, Stream toStream)
{
    if (fromStream == null)
        throw new ArgumentNullException("fromStream");
    if (toStream == null)
        throw new ArgumentNullException("toStream");
    var bytes = new byte[8092];
    int dataRead;
    while ((dataRead = fromStream.Read(bytes, 0, bytes.Length)) > 0)
        toStream.Write(bytes, 0, dataRead);
}

{% endhighlight %}

### 10. Has<T>()/Is<T>()/Add<T>()/Remove<T>() - Type: Enum
{% highlight java %}
public static bool Has<T>(this System.Enum type, T value)
{
    try
    {
        return (((int)(object)type & (int)(object)value) == (int)(object)value);
    }
    catch
    {
        return false;
    }
}
public static bool Is<T>(this System.Enum type, T value)
{
    try
    {
        return (int)(object)type == (int)(object)value;
    }
    catch
    {
        return false;
    }
}
public static T Add<T>(this System.Enum type, T value)
{
    try
    {
        return (T)(object)(((int)(object)type | (int)(object)value));
    }
    catch (Exception ex)
    {
        throw new ArgumentException(
            string.Format(
                "Could not append value from enumerated type '{0}'.",
                typeof(T).Name
                ), ex);
    }
}
public static T Remove<T>(this System.Enum type, T value)
{
    try
    {
        return (T)(object)(((int)(object)type & ~(int)(object)value));
    }
    catch (Exception ex)
    {
        throw new ArgumentException(
            string.Format(
                "Could not remove value from enumerated type '{0}'.",
                typeof(T).Name
                ), ex);
    }
}

{% endhighlight %}