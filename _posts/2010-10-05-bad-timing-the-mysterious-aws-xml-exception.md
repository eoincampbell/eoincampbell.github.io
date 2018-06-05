---
layout: post
title: Bad Timing & The Mysterious AWS XML Exception
date: 2010-10-05 14:25:06.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- ".NET"
- C#
- Coding
tags:
- Amazon
- AWS
- Exception
- S3
- x-amz-date
- XML
meta:
  _edit_last: '1'
  _sexybookmarks_permaHash: 5a2802a7e103a427ccdced544eec412c
  _sexybookmarks_shortUrl: http://tinyurl.com/37wz22h
  _thumbnail_id: '210'
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p><a href="http://trycatch.me/bad-timing-the-mysterious-aws-xml-exception/number_13/" rel="attachment wp-att-210"><img src="{{ site.baseurl }}/assets/number_13-150x150.jpg" alt="Bad Luck" title="Bad Luck" width="150" height="150" class="size-thumbnail wp-image-210" /></a><br />
Server Time Mis-configured, Amazon Web Services S3 .NET SDK has a bug in it, Vague XMLExceptions that don't make sense.</p>
<p>Sometimes karma is just going to get you. There's no point fighting it. A series of events &amp; issues just come together in cosmic bliss guaranteed to completely wreck your weekend.</p>
<p>We rolled out some new code for a new client last week. One of the "big stories" for both us and the client was a migration away from in-house content storage to a cloud based solution. We had opted to use Amazon S3 for the file storage part and after ~6 weeks of project development, rigourous QA &amp; Regression testing, and a bit of a stressful production release we were good to go. Our Biz/Mkt team had seen it and they were happy. Last minute checks of the production service were done, all looked good, and home we went.</p>

{% raw %}
System.ApplicationException: Error Occured in `S3Storage.Exists` --->
System.Xml.XmlException: Root element is missing.
at System.Xml.XmlTextReaderImpl.Throw(Exception e)
at System.Xml.XmlTextReaderImpl.ThrowWithoutLineInfo(String res)
{% endraw %}

<p>Vague &amp; unhelpful on a Saturday afternoon. After a visit to the office, many hours of debugging, testing on multiple servers &amp; pcs with the most bizarre  and unhelpful results, one bright spark noticed that the clock on the server was wrong. So we fixed the time. </p>
<blockquote><p><strong><em>And it "magically" started working</em></strong>.</p></blockquote>
<p> *sigh* This kind of software voodoo drives me nuts so I took a few minutes today to figure out what the heck had actually happened.</p>
<p>So why would an incorrectly set system clock possibly manifest itself as a `Root element is missing` <code>XmlException</code>. A little context first. The exception was occuring in our code on the AWS SDK's S3Client <code>GetObjectMetadata</code> method</p>
<p><em>Note: some code omitted &amp; replaced with "..." for brevity</em></p>

{% highlight csharp %}
var mdRequest = new GetObjectMetadataRequest() {
    BucketName = targetBucket,
    Key = targetFileName
};

//Exception Occurs Here
var mdResponse = s3client.GetObjectMetadata(mdRequest);
{% endhighlight %}

<p>Time to break out .NET Reflector and go through the SDK Code.</p>


{% highlight csharp %}
public GetObjectMetadataResponse GetObjectMetadata(
	GetObjectMetadataRequest request)
{
    ...
    this.ConvertGetObjectMetadata(request);
    return this.Invoke<GetObjectMetadataResponse>(request);
}

private void ConvertGetObjectMetadata(
	GetObjectMetadataRequest request)
{
    ...
    this.AddS3QueryParameters(request, request.BucketName);
}

private void AddS3QueryParameters(
	S3Request request, string destinationBucket)
{
    ...
    webHeaders["x-amz-date"] =
        AmazonS3Util.FormattedCurrentTimestamp;
}
{% endhighlight %}

<p>As we follow the chain of calls, we find this little gem, the webservice sets it's own custom header called <strong><em>x-amz-date</em></strong> with the current time from the local system running the App. Well that's some progress. Some quick googling and we find that this timestamp requirement is mandatory for REST calls to the service and that it must be within 15 minutes of the Amazon System Time when the request is received otherwise a <code>RequestTimeTooSkewed</code> error status will be thrown back.</p>
<p><strong><a href="http://docs.amazonwebservices.com/AmazonS3/2006-03-01/index.html?RESTAuthentication.html">http://docs.amazonwebservices.com/AmazonS3/2006-03-01/index.html?RESTAuthentication.html</a></strong></p>
<p>After some quick testing on my local machine, this does indeed seem to be case. 14 minute difference. No problem. 16 minute difference. Still the XML Exception. As a side note, it turns out our PDC was serving time to our entire production domain of about 14 minutes &amp; 59 seconds out of whack with the real world and lost another second over the weekend. That's just bad luck.</p>
<p>Alas, the docs would appear to be incorrect (at least from the aspect of the the .NET Library behavior). In fact there's a pretty horrid bug in the internal exception handling of the AWS SDK for .NET. When attempting to call the <code>GetResponse</code> in the main invoke method, it throws an exception and dumps you down into the generic catch all WebExceptions block.</p>

{% highlight csharp %}
private T Invoke(S3Request userRequest) where T: S3Response, new()
{
    ...
    try
    {
        httpResponse = request.GetResponse() as HttpWebResponse;
    }
    catch (WebException exception)
    {
        using (HttpWebResponse response2 =
            exception.Response as HttpWebResponse)
        {
            flag = ProcessRequestError(request, exception,
                response2, requestAddr, out headers, this.myType);
	}
    }
}

private static bool ProcessRequestError( ... )
{
    ...
    using (StreamReader reader = new StreamReader(errorResponse.GetResponseStream(), Encoding.UTF8))
    {
        responseBody = reader.ReadToEnd();
    }
    ...
    if ((statusCode == HttpStatusCode.InternalServerError) || (statusCode == HttpStatusCode.ServiceUnavailable))
    {
        return true;
    }
    ...
    using (XmlTextReader reader2 = new XmlTextReader(
        new StringReader(Transform(responseBody, "S3Error", t))))
    {
        ...
        throw new AmazonS3Exception( ... ); 
        //without original WebException
    }
}
{% endhighlight %}
<p>This is where it all goes pear shaped. The <code>WebException</code> thrown is actually a 403 Access Forbidden. But the <code>ProcessRequestError</code> method doesn't account for this. It'll trigger the retry flag on an InternalServerError or ServiceUnavailable status code but otherwise it will continue on, and attempt to parse the response body into an S3Error. In our case the response body is <code>string.Empty</code>, an <code>XmlException</code> gets thrown and the original <code>WebException</code> gets lost in the ether</p>
<p><strong>*Wanders off to find a way to log bugs with the AWS for .NET Dev team*</strong></p>


***Eoin Campbell***