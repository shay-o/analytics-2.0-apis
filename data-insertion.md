# Overview

The Data Insertion API supports HTTP POST and HTTP GET for submitting data to Adobe Experience Cloud servers.

The Data Insertion API provides a mechanism for server-side data collection and submission to Experience Cloud servers. Instead of using JavaScript beacons on each Web page to transmit visitor data to Experience Cloud servers, server-side data collection collects data based solely on Web browser requests and Web server responses.

While this data capture method cannot capture all data that is available via page-based beacons, it provides valuable insight into user activity on your Web pages without the overhead associated with attaching a JavaScript beacon to every Web page.

**Note**: Data Insertion API and Bulk Data Insertion API are both methods to submit server-side collection data to Adobe Analytics. Data Insertion API calls are made one event at a time. Bulk Data Insertion API accepts CSV formatted files containing event data, one event per row. If you are working on a new implementation of server-side collection, we recommend using Bulk Data Insertion API.

# Data Insertion Process
The Data Insertion API supports HTTP POST for submitting data to Adobe Experience Cloud servers.

**Note**: The Analytics response to each data insertion includes a status message (SUCCESS or FAILURE).

## HTTP POST
Use an HTTP POST to submit properly-formatted Data Insertion JSON to the Data Insertion URL. The Data Insertion URL differs from the standard JavaScript data submission URL. Adobe ClientCare can provide the domain name of the Adobe data collection servers where you should send data. For example:

http://namespace.sc.omtrdc.net/b/ss//7 [tbd]

http://namespace.sc.omtrdc.net/b/ss//7 [tbd]

Note: The "7" [tbd] code at the end of the URL indicates that the data submission requires JSON processing.

Upon receipt, Adobe servers perform basic tag validation of the data insertion. If it encounters an error, Adobe returns a Failure response. If the data insertion is successful, Adobe queues the data insertion request for processing by the standard Analytics Data Processing Engine. The engine processes these requests in the same way it processes data collected via JavaScript.

When using HTTP POST with the Data Insertion API, consider the following:

The Data Insertion API requires data in UTF-8 format. Specify the character encoding in the opening XML tag, as shown in XML Data Insertion Format.
Replace Ampersand (&), greater-than (>), and less-than (<) symbols with their HTML equivalents when passing them into a Analytics variable. For example, submit <evar1>News & Sports <local> </evar1> as <evar1>News &amp; Sports &lt;local&gt; </evar1>.
To submit data over an encrypted connection, the application must be configured to support HTTPS POST commands. Some tools that let you do this include:
PHP versions 4.3.0 and higher support OpenSSL, and can be used to POST data through SSL port 443. You can see an example of this in Sample Data Insertion (PHP).
You can use cURL to send data over an SSL connection. For information about HTTPS POST in Java, see http://java.sun.com/developer/technicalArticles/Security/secureinternet/.
Microsoft .Net Framework version 1.1 supports the HTTP header: Expect: 100-Continue in HTTP POST requests, but Experience Cloud servers reject POST data sent with this type of request. To avoid this, set ServicePointManager.Expect100Continue = False. For more information, see http://msdn.microsoft.com/library/en-us/cpref/html/frlrfsystemnetservicepointmanagerclassexpect100continuetopic.asp .

Upon receipt, Adobe servers perform basic tag validation of the data insertion. If it encounters an error, Adobe returns a Failure response. If the data insertion is successful, Adobe queues the data insertion request for processing by the standard Analytics Data Processing Engine. The engine processes these requests in the same way it processes data collected via JavaScript.

Note: The Data Insertion API requires data in UTF-8 format.


-----------------------------------------------------------------------------------------------------------------------------------------------------------
# Delayed Data

By using the Data Insertion API, you can submit delayed data \(for example, from an offline application, to Adobe data collection servers.

**Note:** Data from delayed submissions is available to Data Warehouse and Ad hoc analysis.

When submitting delayed data, you must send visitor data in the order it was received, \(see [Sequential Data Requirements](c_sequential_data.md#)\) but you do not need to send data for different visitors in any specific order. For example, you can submit all data for visitor A before data for visitor B, even if some or all of visitor B data occurred before visitor A data.

Submitting delayed data requires an additional `<timestamp>` tag in the XML. Analytics discards data without a timestamp.

**Note:** 

To view time-stamped hits, Adobe ClientCare must enable time-stamp support on the report suite, in addition to the time-stamp variable that you set. Time-stamped hits sent to a report suite that has not been enabled do not appear in the report.

Format the timestamp data, including time zone, using either the Unix timestamp format or the ISO-8601 format. For example, using the ISO-8601 format, a timestamp for a visitor event that occurred on November 17, 2013 at 5:33 PM Mountain Time \(GMT-7:00:00\) is as follows:

`2013-11-17T17:33:22-07` 

Consider the following when submitting delayed \(time-stamped\) data:

-   While a report suite is collecting time-stamped data, it cannot collect data through the standard JavaScript beacon method.
-   You must enable time stamp support on a per-report-suite basis.
-   You can also use Data Sources to send this type of data to the Experience Cloud via FTP. For more information, see the [Data Sources User Guide](https://marketing.adobe.com/resources/help/en_US/sc/datasources/).

 -----------------------------------------------------------------------------------------------------------------------------------------------------------

# HTTP 1.1

Adobe data collection servers support HTTP 1.1.

Using HTTP 1.1, you can send multiple POST requests in a single session. However, when using HTTP 1.1, be aware of the following:

-   While a report suite is collecting time-stamped data, it cannot collect data through the standard JavaScript beacon method.
-   Adobe data collection servers limit the number of HTTP requests per HTTP session. To account for this, make sure your application can detect closed sessions and respond appropriately.
-   Adobe data collection servers observe HTTP 1.1 rules when chunking content. Make sure your application properly interprets the chunking numbers provided in the Online Marketing Suite server response to your data insertion. For example, in the following HTTP 1.1 response, the numbers `68` and `0` are chunking numbers, which represent the number of bytes to read before the next chunk.
-   The data insertion load balancers are optimized for small packets. As a result, the load balancer will frequently break a persistent HTTP connection and cause the server to appear unresponsive. Your insertion process should check for a failed connection before upload and re-establish the connection as necessary.


```
TTP/1.1 200 OK Date: Thu, 07 Dec 2008 15:48:51 GMT Expires: Wed, 06 Dec 2008 15:48:51 GMT Last-Modified: Fri, 08 Dec 2008 15:48:51 GMT Cache-Control: no-cache, no-store, must-revalidate, max-age=0, proxy-revalidate, no-transform, private Pragma: no-cache ETag: "457837E3-649A-4BBB9314" Vary: * P3P: policyref="/w3c/p3p.xml", CP="NOI DSP COR NID PSA OUR IND COM NAV STA" Transfer-Encoding: chunked Content-Type: text/xml  68 <?xml version="1.0" encoding="UTF-8"?> <status>FAILURE</status> <reason>NO pagename OR pageurl</reason> 0 
```

 -----------------------------------------------------------------------------------------------------------------------------------------------------------

# Limitations

There are some limitations to server-side data collection when compared to client-side data collection using JavaScript.

-   Server-side data collection cannot collect Web browser configuration data.
-   Server-side data collection cannot collect ClickMap data.
-   Server-side data collection loses cached page navigation data because using a Web browser's back button or reloading a cached page does not generate additional server requests or responses. This limits pathing data available for Analytics reports.
-   Some bots do not run JavaScript when requesting a page, and do not request the page's image tags. However, server-side data collection includes requests from those bots and spiders.
-   You must have control over the servers from which you want to collect data. If you use a Content Delivery Network \(CDN\) such as Akamai\* to deliver web pages, the server-side data collection described here cannot collect data for those pages.
-   If you control most of your Web site pages, but use some third-party services \(for example, surveys or shopping cart systems\), server-side data collection cannot collect data from the third-party pages. You should use JavaScript tagging on third-party pages that request content from your Web servers to gain visibility into those systems.
-   Server-side data collection alone cannot provide cross-domain tracking of site visitors. However, you can use the `<visitorID>` tag in conjunction with client-side JavaScript data collection to pass additional information to the server. This is the only way that cross-domain tracking is possible when using a `<visitorID>`.
-   Server-side data collection can co-exist with a client-side \(JavaScript\) implementation only if clients submit server-side data in near real-time. If clients have a JavaScript beacon implementation, and want to collect server-side data and submit it in batches, they should use Data Sources, not the Data Insertion API.

 -----------------------------------------------------------------------------------------------------------------------------------------------------------

# Sequential Data Requirements

 

To ensure accurate visit number values, visit counts, and event attribution on persisting variables, applications must send visitor data in sequential order for every visitor. Data for one visitor may be submitted before or between data for another visitor, but each visitor's data must be received by Adobe servers sequentially, based on the time-stamp of the hit.

In some situations, it is possible to see two hits with the same time-stamp. For example, two hits might have the same time-stamp if an auto-playing video fires during the same second that the page loads. If two hits for one visitor have the same time-stamp, and order matters for those hits \(one contains variables that should persist to the next one\), you can change the time-stamp of the second hit by one second so we sort the two correctly.

 

The Analytics data processing engine closes a visit after 30 minutes of inactivity. For time-stamped data, if the difference between time-stamps is greater than 30 minutes, the visit number is incremented and a new visit is counted.

 -----------------------------------------------------------------------------------------------------------------------------------------------------------

# Visitor Identification

To track site visitors, each visitor must have a unique *visitor ID*. Ideally, this is a persistent cookie with an extended expiration period.

Use the `visitorID` tag, the combination of IP address and `userAgent`, to submit visitor ID information to Adobe data collection servers.

If the data insertion includes a `visitorID`, the Analytics Data Processing Engine assumes that persistent cookies are enabled for the visitor's Web browser. If the data insertion identifies visitors using IP address/User Agent, Analytics displays persistent cookies for that visitor as disabled.

**Note:** To avoid processing problems, make sure that the same visitor ID is not specified on a large number of server calls. For example, setting the same visitor ID for all unauthenticated visitors is not recommended.

The visitor ID is critical for linking a visitor's activities captured via JavaScript with those submitted through tagless data collection, or a third-party application. For example, to link a visitor's browsing activity with transactions in a third-party shopping cart application, you must pass the visitor ID to the shopping cart application. The shopping cart application uses the visitor ID when uploading transaction data via the Data Insertion API so Analytics can link the transaction data to the visitor's browsing activity \(previously captured via JavaScript\).

## Custom Visitor IDs

Typically, the client systems generate unique visitor identifiers that you can use as the `visitorID` value. We recommend generating your own visitor ID, in both the XML and in the JavaScript so that the values match.

If you are generating your own visitor ID, it can be up to 100 alpha-numeric characters and cannot contain a hyphen.

## Reading the s\_vi Cookie

When sending data that is connected to a web visitor, you need to use the same visitor ID in both JavaScript on your page and in data sent by the data insertion API. To accomplish this, you could pass the visitor ID in a hidden form element to a server-side application that uses the data insertion API.

The Analytics visitor ID is contained in the `s_vi` cookie, and can be retrieved on the client using the cookie read function:

```
s.c_r('s_vi');
```

The visitor ID that is returned is formatted similar to the following:

```

[CS]v1|1234567890123456-6543210987654321[CE]
```

When passing the visitor id to the data insertion API, pass just the values between the pipe '|' character and \[CE\]. For example:

```

<visitorid>1234567890123456-6543210987654321</visitorid>
```

## Experience Cloud ID Service

If you have implemented the Visitor ID service, functions are provided to simplify reading visitor IDs. See [Experience Cloud ID Service Library Reference](https://marketing.adobe.com/resources/help/en_US/mcvid/mcvid_library.html).

Make sure you check for both the Experience Cloud ID and the Analytics Visitor ID, and send both IDs \(if present\) to make sure any data sent is associated with the existing Analytics visitor profile.

-----------------------------------------------------------------------------------------------------------------------------------------------------------

# XML Encoding

 

XML data sent to the data insertion API must be in UTF-8 and the special XML characters need to be replaced with entities:

|Character|XML Entity|
|---------|----------|
|&|&amp;|
|<|&lt;|
|\>|&gt;|

-----------------------------------------------------------------------------------------------------------------------------------------------------------

# POST Response Codes

The Data Insertion API supports these response messages to HTTP POST operations.

The following table lists all supported XML tags, along with their associated JavaScript variable equivalents and HTTP header variable equivalents, where applicable.

 

ponse messages can help you understand and correct the problem.

| POST Response | Description |
|-----------------|---------------|
| `<?xml version="1.0" encoding="UTF-8"?><?xml version="1.0" encoding="UTF-8"?><status>SUCCESS</status>` | Success \(general\) |
| `<?xml version="1.0" encoding="UTF-8"?><?xml version="1.0" encoding="UTF-8"?><status>FAILURE</status>` | Failure \(general\) |
| `<?xml version="1.0" encoding="UTF-8"?><?xml version="1.0" encoding="UTF-8"?><status>FAILURE</status> <reason>NO account</reason>` | Failure \(missing required report suite id\) |
| `<?xml version="1.0" encoding="UTF-8"?><?xml version="1.0" encoding="UTF-8"?><status>FAILURE</status> <reason>NO pagename OR pageurl</reason>` | Failure \(missing required page name or page URL\) |
| `<?xml version="1.0" encoding="UTF-8"?><?xml version="1.0" encoding="UTF-8"?><status>FAILURE</status> <reason>NO visitorid OR ipaddress</reason>` | Failure \(missing required visitor id or IP address\) |
| `<?xml version="1.0" encoding="UTF-8"?><?xml version="1.0" encoding="UTF-8"?><status>FAILURE</status> <reason>Syntax Error</reason>` | Failure \(syntax error: includes malformed XML, non-encoded reserved characters, etc.\) |

-----------------------------------------------------------------------------------------------------------------------------------------------------------

# Supported XML Tags and Query Variables

When processing HTTP POST data submissions, only values in supported XML tags are processed. When submitting HTTP GET data submissions, only specified query string and HTTP header variables are processed.

The following table lists all supported XML tags, along with their associated query string variable equivalents and HTTP header variable equivalents, where applicable. The JavaScript variable column is provided for reference. Details about each of these variables is provided in the [Analytics Variables](http://microsite.omniture.com/t2/help/en_US/sc/implement/?f=sc_variables) topic in the *Analytics Implementation Guide*.

To specify that a given row should be considered a page view, that row must contain either `<pageName>` or `<pageURL>`. To specify that a given row should be considered a link event, that row must include `<linkType>` and either `<linkName>` or `<linkURL>`. If a given row contains both `<linkType>` and either `<pageName>` or `<pageURL>`, Adobe Analytics will automatically clear out `<pageName>` and `<pageURL>` by setting them to `null`. Also, if a given row does not contain the minimum fields required to fully identify the row as either a page view or a link event, that row will be dropped.

Every data insertion must also include one of `<visitorID>`, `<marketingCloudVisitorID>` or `<IPaddress>`.

**Note:** XML tags are case-insensitive, capitalization is used only to assist readability.

|XML Tag \(POST\)|Query String Parameter \(GET\)|JavaScript Variable|HTTP Header Var|Description|
|----------------|------------------------------|-------------------|---------------|-----------|
| `<browserHeight>` |`bh` | N/A | N/A | Browser height in pixels \(For example, 768\). |
| `<browserWidth>` |`bw` | N/A | N/A | Browser width in pixels \(For example, 1024\). |
| `<campaign>` |`v0` | campaign | N/A | The campaign tracking code associated with the page. |
| `<channel>` |`ch` | channel | N/A | The page title or bread crumb. |
| `<colorDepth>` |`c` | N/A | N/A | Monitor color depth in bits \(For example, 24\). |
| `<connectionType>` |`ct` | N/A | N/A | Visitor's connection type \("lan" or "modem"\). |
| `<contextData>` |`c.\[key\]` | contextData | N/A | Key-values pairs are specified in one of the following formats: `<my.a>red</my.a>`  or: `<my><a>red</a></my>`  Each of these examples result in a context data value of `my.a = red`. Multiple key-value pairs can be specified. In the query string, this context data variable would appear as `c.my.a=red` |
| `<cookiesEnabled>` |`k` | N/A | N/A | Whether the visitor supports first party session cookies \(Y or N\). |
|`<customerPerspective>`|`cp`|customerPerspective|N/A| Whether a hit is considered a background or foreground hit. <list><li>0 - Foreground (Default)</li><li>1 - Background </li></list> See [Context Aware Sessions](https://marketing.adobe.com/resources/help/en_US/reference/vrs-mobile-visit-processing.html) for more detail|
| `<currencyCode>` |`cc` | currencyCode | N/A | Revenue currency code For example, `USD`. |
| `<eVar*\#*>` For example, `<eVar2>`. |`v*\#*` For example, `v2`. | eVar1 - eVar75 | N/A | Analytics eVar. |
| `<events>` |`events` | events | N/A | A list of Analytics events. Multiple events are separated by a comma. <events\>event1</events\> <events\>event1,event2</events\> |
| `fallbackVisitorId` |`fid` | N/A | N/A | A fallback cookie `s_fid` is set when the primary cookie (`AMCV_` or `S_vi`) is unavailable. It has a 2 year expiration and is used as the fallback identification method going forward. |
| `<hier*n*>` For example, `<hier2>`. |`h#` For example, `h2` | hier1 - hier5 | N/A | A hierarchy string. |
| `<homePage>` |`hp` | N/A | N/A | Whether the current page is the visitor's homepage \(Y or N\). |
|`<imsregion>` |`aamlh` | N/A | N/A | Use this tag to ensure data is forwarded to the right Audience Manager regional data collection center when using the data insertion API for both collecting Analytics data and forwarding that data to Audience Manager. The Analytics tracking server/API end point must also be set to forward to the correct Audience Manager instance before Analytics data will forward to Audience Manager. Use the [getLocationHint](https://marketing.adobe.com/resources/help/en_US/mcvid/mcvid-getlocationhint.html) function of the VisitorAPI to retrieve the correct `imsregion` value for the user. The `marketingCloudVisitorID` tag is also required when using this tag. |
| `<ipaddress>` | N/A | N/A | X-Forwarded-For | The visitor's IP address. |
| `<javaEnabled>` |`v` | N/A | N/A | Whether the visitor has Java enabled \(Y or N\). |
| `<javaScriptVersion>` |`j` | N/A | N/A | JavaScript version. For example, 1.3. |
| `<language>` |`N/A` | N/A | Accept-Language | The browser's supported language. For example, "en-us". |
| `<linkName>` |`pev2` | linkName | N/A | Name of link. |
| `<linkType>` |`pe` | linkType | N/A | Type of link \("d", "e", or "o"\). |
| `<linkURL>` |`pev1` | linkURL | N/A | The link's HREF. For custom links, page values are ignored. |
|`<list*n*>` For example, `<list2>`. |`l#` | list1 - list3 | N/A | A delimited list of values that are passed into a variable, then reported as individual line items for reporting. |
| `marketingcloudorgid` |`mcorgid` | N/A | N/A | The Experience Cloud Organization ID; it identifies the organization within the Adobe Experience Cloud. |
| `<pageName>` |`pageName` | pageName | N/A | The Web page name. |
| `<pageType>` |`pageType` | pageType | N/A | The Web page type. This is only used on 404 error pages. Set pageType to "Error Page" for when a 404 error is detected. |
| `<pageURL>` |`g` | pageURL | N/A | The Web page URL For example, http://www.mysite.com/index.html. |
| `<plugins>` |`p` | N/A | N/A | Semicolon separated list of Netscape plug-in names. |
| `<products>` |`products` | products | N/A | List of all products on the page. Separate products with a comma. For example: Sports;Ball;1;5.95, Toys; Top;1:1.99. |
| `<prop*n*>` For example, `<prop2>`  |`c#` For example, `c2` | prop1 - prop75 | N/A | Analytics property name. |
| `<purchaseID>` |`purchaseID` | purchaseID | N/A | Purchase ID number. |
| `<referrer>` |`r` | N/A | N/A | The URL of the page referrer. |
| `<reportSuiteID>` |Contained in the URL. See [HTTP GET Sample](../sample_code/r_sample_http_get.md#).| s\_account | N/A | Specifies the report suites where you want to submit data. Separate multiple report suite IDs with a comma. |
| `<resolution>` |`s` | N/A | N/A | Monitor resolution For example, 1280x1024. |
| `<scXmlVer>` | N/A | N/A | N/A | Analytics XML request version number. For example, 1.0. |
| `<server>` |`server` | server | N/A | The Web server serving the page. |
| `<state>` |`state` | state | N/A | The visitor's U.S. state. |
| `<timestamp>` |`ts` | timestamp | N/A | The time and date on which the data was collected. |
| `<timezone>` | Part of the `t` parameter | N/A | N/A | XML POST: Visitor's time zone offset from GMT in hours. For example, -8. GET: The visitor's time zone is included in the `t` query string parameter, which contains the local time. The `t` parameter is in the following format: ```dd/mm/yyyy hh:mm:ss D OFFSET``` Where D is a number in the range `0-6` specifying the day of the week, and `OFFSET` represents: ```offset from GMT in hours * 60 * - 1``` For example: ```09/23/2013 14:00:00 1 420``` |
| `<transactionID>` |`xact` | transactionID | N/A | Common value used to tie multi-channel user activities together for reporting purposes. For more information, see the [Data Sources User Guide](http://microsite.omniture.com/t2/help/en_US/sc/datasources/oms_sc_data_sources.pdf). |
| `<userAgent>` | N/A | N/A | User-Agent | The visitor's browser type and OS. |
| `<visitorID>` |`vid` | Automatic, or `visitorID` is using custom IDs. | N/A | Visitor's Analytics ID. See [Visitor Identification](../overview/c_visitor_id.md#). |
| `<marketingCloudVisitorID>` |`mid` | N/A | N/A | Experience Cloud ID. See [Visitor Identification](../overview/c_visitor_id.md#) and the [Experience Cloud Visitor ID Service](http://microsite.omniture.com/t2/help/en_US/mcvid/). |
| `<zip>` |`zip` | zip | N/A | The visitor's zip code. |

-----------------------------------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------------------------------------------------
# Sample Code

 

-   **[HTTP POST Sample](../sample_code/r_sample_http_post.md)**  
 This sample displays the structure of an HTTP POST request and response using the Data Insertion API.
-   **[HTTP GET Sample](../sample_code/r_sample_http_get.md)**  
 This sample displays the structure of an HTTP GET request and response using the Data Insertion API.
-   **[Data Insertion Sample \(PHP\)](../sample_code/r_sample_php.md)**  
 This PHP sample illustrates how to connect to Adobe data collection servers and record a page view.
-   **[Data Insertion Sample \(Java\)](../sample_code/r_sample_java.md)**  
 This Java sample illustrates how to use Java classes to send an XML request.
-   **[Data Insertion Sample \(Python\)](../sample_code/r_sample_python.md)**  
 This Python sample illustrates how to access the Data Insertion API.

-----------------------------------------------------------------------------------------------------------------------------------------------------------

# HTTP POST Sample

This sample displays the structure of an HTTP POST request and response using the Data Insertion API.

**Note:** Do not copy-and-paste these code samples. They use generic values that you must replace with valid data appropriate to your application.

The *\[rsid\]* parameter identifies the report suite where you want to submit the data. Other values, such as `PageURL`, are sample values only.

## HTTP 1.0 POST Request

```
POST /b/ss//6 HTTP/1.0
Host: [namespace].sc.omtrdc.net
Content-Length: 593

<?xml version=1.0 encoding=UTF-8?>
<request>
   <sc_xml_ver>1.0</sc_xml_ver>
   <pageURL>apps.sillystring.com/summary.do</pageURL>
   <referrer>http://apps.sillystring.com/summary.do</referrer>
   <ipAddress>192.168.10.1</ipAddress>
   <pageName>summary</pageName>
   <eVar2>14911</eVar2>
   <userAgent>Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0; SLCC1; .NET CLR 2.0.50727; Media Center PC 5.0; .NET CLR 3.0.04506)</userAgent>
   <prop10>Brazil</prop10>
   <visitorID>1286556420966514130</visitorID>
   <timestamp>2009-03-05T01:00:01-05</timestamp>
    <reportSuiteID>[rsid]</reportSuiteID>
</request>
```

## HTTP POST Response

```
HTTP/1.1 200 OK
Date: Wed, 13 May 2009 16:26:47 GMT
X-C: ms-3.7.2
Expires: Tue, 12 May 2009 16:26:47 GMT
Last-Modified: Thu, 14 May 2009 16:26:47 GMT
Cache-Control: no-cache, no-store, must-revalidate, max-age=0, proxy-revalidate, no-transform, private
Pragma: no-cache
ETag: "4A0AF4C7-08E1-37C7F492"
Vary: *
P3P: policyref="/w3c/p3p.xml", CP="NOI DSP COR NID PSA OUR IND COM NAV STA"
xserver: www79
Connection: close
Content-Type: text/xml
<?xml version="1.0" encoding="UTF-8"?>
<status>SUCCESS</status>
```

## HTTP 1.1 POST Request

```
POST /b/ss//6 HTTP/1.1
Host: [namespace].sc.omtrdc.net
Keep-Alive: timeout=15
Connection: Keep-Alive
Content-Length: 593

<?xml version=1.0 encoding=UTF-8?>
<request>
   <sc_xml_ver>1.0</sc_xml_ver>
   <pageURL>apps.sillystring.com/summary.do</pageURL>
   <referrer>http://apps.sillystring.com/summary.do</referrer>
   <ipAddress>192.168.10.1</ipAddress>
   <pageName>summary</pageName>
   <eVar2>14911</eVar2>
   <userAgent>Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0; SLCC1; .NET CLR 2.0.50727; Media Center PC 5.0; .NET CLR 3.0.04506)</userAgent>
   <prop10>Brazil</prop10>
   <visitorID>1286556420966514130</visitorID>
   <timestamp>2009-03-05T01:00:01-05</timestamp>
   <reportSuiteID>[rsid]</reportSuiteID>
</request>
```

## HTTP 1.1 POST Response

```
HTTP/1.1 200 OK
Date: Wed, 13 May 2009 16:25:12 GMT
X-C: ms-3.7.2
Expires: Tue, 12 May 2009 16:25:12 GMT
Last-Modified: Thu, 14 May 2009 16:25:12 GMT
Cache-Control: no-cache, no-store, must-revalidate, max-age=0, proxy-revalidate, no-transform, private
Pragma: no-cache
ETag: "4A0AF468-4DF2-33AE9089"
Vary: *
P3P: policyref="/w3c/p3p.xml", CP="NOI DSP COR NID PSA OUR IND COM NAV STA"
xserver: www116
Keep-Alive: timeout=15
Connection: Keep-Alive
Content-Length: 40
Content-Type: text/xml

<?xml version="1.0" encoding="UTF-8"?>
<status>SUCCESS</status>
```

**Parent topic:** [Sample Code](../sample_code/c_sample_code.md)

-----------------------------------------------------------------------------------------------------------------------------------------------------------
# Data Insertion Sample \(PHP\)

This PHP sample illustrates how to connect to Adobe data collection servers and record a page view.

```
<?/
 **
 * @file
 * Example code to send one pageview to Omniture via the Data Insertion API
 *   and check for a valid response.
 * 
 * @author Omniture, an Adobe Company <clientcare@omniture.com>
 * @copyright 2007-2010 Adobe Systems, Inc. All Rights Reserved.
 */ 
// NOTE: Contact an ClientCare to identify the correct namespace and domain for your company.

$namespace = "namespace";
$domain    = "sc.omtrdc.net";
$host      = $namespace.".".$domain;
$rsid      = "rsid";
$vid       = "";
$ip        = "10.0.0.1";
$page_url  = "";
$pageName  = "Test Page"; 
$timestamp = "2008-10-21T17:33:22-07";

// create opening XML tags
$xml  = "<?xml version=1.0 encoding=UTF-8?>\n";
$xml .= "<request>\n";
$xml .= " <scXmlVer>1.0</scXmlVer>\n"; 

// add tags for required elements
$xml .= $rsid ? " <reportSuiteID>$rsid</reportSuiteID>\n":""; 

//The timestamp line of code can only be used when an Omniture representitive has enabled timestamp support for your organization.
$xml .= $timestamp ? " <timestamp>$timestamp</timestamp>\n":"";

$xml .= $vid ? " <visitorID>$vid</visitorID>\n":"";
$xml .= $ip ? " <ipAddress>$ip</ipAddress>\n":""; 

$xml .= $page_url ? " <pageURL>$page_url</pageURL>\n":"";
$xml .= $pageName ? " <pageName>$pageName</pageName>\n":""; 

// close the XML request
$xml .= "</request>\n"; 

// Create POST, Host and Content-Length headers
$head = "POST /b/ss//6 HTTP/1.0\n";
$head .= "Host: $host\n";
$head .= "Content-Length: ".(string)strlen($xml)."\n\n"; 

// combine the head and XML
$request = $head.$xml; 

$fp=fsockopen($host,80,$errno,$errstr,30); 
// Use this function in place of the call above if you have PHP 4.3.0 or
//   higher and have compiled OpenSSL into the build.
//
// $fp = pfsockopen("ssl://".$host, 443, $errno, $errstr);
// 
if( $fp ) { 

// send data
fwrite($fp,$request); 

// get response
$response="";
while( !feof($fp) ){
	$response .= fgets($fp,1028);
}
fclose($fp); 

// display results
echo "RESULTS:\n";
print_r($response);
echo "\n"; 

// check for errors
if( preg_match("/status\>FAILURE\<\/status/im",$response) ){
	/*
	* TODO:
	* write $request and $response to log file for investigation
	* and retries
	*/
	echo "<h1>Failure</h1>\n";
	echo "<p>Note the reason tag in the response, fix and try again.</h1>\n";
}
} else {
echo "<H1>Couldn't open port to Analytics servers</H1>\n"; 
if(!$rsid){
	echo "<H4>No report suite specified</H4>\n";
} else {
	echo "<p>$errstr ($errno)</p>\n";
	/*
	* TODO:
	* write $request and $errstr to log file for investigation
	* and retries
	*/
}
}
?>
```

**Parent topic:** [Sample Code](../sample_code/c_sample_code.md)

-----------------------------------------------------------------------------------------------------------------------------------------------------------

# Data Insertion Sample \(Java\)

This Java sample illustrates how to use Java classes to send an XML request.

This example sets the IP address, page name, and custom variables.

```java
DataSubmission.java
package com.omniture;

import java.util.ArrayList;
import java.util.Iterator;

public class DataInsertion
{
    private String rptSuiteID             = null;
    private String visitorID             = null;
    private StringBuffer buff            = null;
    private ArrayList<String> request    = new ArrayList<String>();
   
    public DataInsertion( String rsid, String vid )
    {
       this.rptSuiteID         = rsid;
       this.visitorID             = vid;
    }
   
    public String toString()
    {
       buff                    = new StringBuffer();
       buff.append( "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n" );
       buff.append( "<request>\n" );
       buff.append( "<sc_xml_ver>1.0</sc_xml_ver>\n" );
       
       if( this.rptSuiteID != null )
       buff.append( this.tagify( "reportsuiteid", this.rptSuiteID ) );
       
       if( this.visitorID != null )
       buff.append( this.tagify( "visitorid", this.visitorID ) );
       
       Iterator iter = this.request.iterator();
       
       while( iter.hasNext() )
       {
          buff.append( iter.next() );
       }
       buff.append( "</request>\n" );
       
       return buff.toString();
    }
    
    private String tagify( String name, String value )
    {
       return "<" + name + ">" + value + "</" + name + ">\n";
    }

    private String tagify( String name, char value )
    {
       return "<" + name + ">" + value + "</" + name + ">\n";
    }
   
    public void set( String tag, String value )
    {
       this.request.add( this.tagify(tag, value) );
    }
   
    public void set( String tag, char value )
    {
       this.request.add( this.tagify(tag, value) );
    }
   
}
```

## DataSubmission.java

```
package com.omniture;

import java.io.BufferedReader;
import java.io.DataOutputStream;
import java.io.InputStreamReader;
import java.net.URL;
import java.net.URLConnection;

public class DataInsertionRequest
{
    public static void main(String[] args) throws Exception
    {
       DataInsertion di = new DataInsertion( "test", "123456" );
          di.set( "ipaddress", "127.0.0.1" );
          di.set( "pagename", "Test Page" );
          di.set( "channel", "Tests" );
          di.set( "prop1", "All" );
          di.set( "events", "event1" );
          di.set( "evar2", "Test Page" );
          di.set( "evar3", "Test Page" );
          di.set( "evar4", "Test Page" );
          di.set( "hier1", "Test|Test Page|All" );
       
          URL url = null;
             URLConnection urlConn         = null;
             DataOutputStream printout     = null;
             BufferedReader input        = null;
             String u            = "http://namespace.sc.omtrdc.net/b/ss/ll/6";
             String tmp             = null;
       
             url = new URL( u );
             urlConn = url.openConnection();
             urlConn.setDoInput( true );
             urlConn.setDoOutput( true );
             urlConn.setUseCaches( false );
             urlConn.setRequestProperty( "Content-Type", "application/x-www-form-urlencoded" );

             printout             = new DataOutputStream(urlConn.getOutputStream());
          
             printout.writeBytes( di.toString() );
             printout.flush();
             printout.close();
   
             input         = new BufferedReader( new InputStreamReader( urlConn.getInputStream( ) ) );
       
             System.out.println( di );
             while( null != ( ( tmp = input.readLine() ) ) )
             {
                System.out.println( tmp );
             }
             printout.close();
             input.close();
    }

}
```

**Parent topic:** [Sample Code](../sample_code/c_sample_code.md)

-----------------------------------------------------------------------------------------------------------------------------------------------------------

# Data Insertion Sample \(PHP\)

This PHP sample illustrates how to connect to Adobe data collection servers and record a page view.

```
<?/
 **
 * @file
 * Example code to send one pageview to Omniture via the Data Insertion API
 *   and check for a valid response.
 * 
 * @author Omniture, an Adobe Company <clientcare@omniture.com>
 * @copyright 2007-2010 Adobe Systems, Inc. All Rights Reserved.
 */ 
// NOTE: Contact an ClientCare to identify the correct namespace and domain for your company.

$namespace = "namespace";
$domain    = "sc.omtrdc.net";
$host      = $namespace.".".$domain;
$rsid      = "rsid";
$vid       = "";
$ip        = "10.0.0.1";
$page_url  = "";
$pageName  = "Test Page"; 
$timestamp = "2008-10-21T17:33:22-07";

// create opening XML tags
$xml  = "<?xml version=1.0 encoding=UTF-8?>\n";
$xml .= "<request>\n";
$xml .= " <scXmlVer>1.0</scXmlVer>\n"; 

// add tags for required elements
$xml .= $rsid ? " <reportSuiteID>$rsid</reportSuiteID>\n":""; 

//The timestamp line of code can only be used when an Omniture representitive has enabled timestamp support for your organization.
$xml .= $timestamp ? " <timestamp>$timestamp</timestamp>\n":"";

$xml .= $vid ? " <visitorID>$vid</visitorID>\n":"";
$xml .= $ip ? " <ipAddress>$ip</ipAddress>\n":""; 

$xml .= $page_url ? " <pageURL>$page_url</pageURL>\n":"";
$xml .= $pageName ? " <pageName>$pageName</pageName>\n":""; 

// close the XML request
$xml .= "</request>\n"; 

// Create POST, Host and Content-Length headers
$head = "POST /b/ss//6 HTTP/1.0\n";
$head .= "Host: $host\n";
$head .= "Content-Length: ".(string)strlen($xml)."\n\n"; 

// combine the head and XML
$request = $head.$xml; 

$fp=fsockopen($host,80,$errno,$errstr,30); 
// Use this function in place of the call above if you have PHP 4.3.0 or
//   higher and have compiled OpenSSL into the build.
//
// $fp = pfsockopen("ssl://".$host, 443, $errno, $errstr);
// 
if( $fp ) { 

// send data
fwrite($fp,$request); 

// get response
$response="";
while( !feof($fp) ){
	$response .= fgets($fp,1028);
}
fclose($fp); 

// display results
echo "RESULTS:\n";
print_r($response);
echo "\n"; 

// check for errors
if( preg_match("/status\>FAILURE\<\/status/im",$response) ){
	/*
	* TODO:
	* write $request and $response to log file for investigation
	* and retries
	*/
	echo "<h1>Failure</h1>\n";
	echo "<p>Note the reason tag in the response, fix and try again.</h1>\n";
}
} else {
echo "<H1>Couldn't open port to Analytics servers</H1>\n"; 
if(!$rsid){
	echo "<H4>No report suite specified</H4>\n";
} else {
	echo "<p>$errstr ($errno)</p>\n";
	/*
	* TODO:
	* write $request and $errstr to log file for investigation
	* and retries
	*/
}
}
?>
```

**Parent topic:** [Sample Code](../sample_code/c_sample_code.md)

-----------------------------------------------------------------------------------------------------------------------------------------------------------

# Data Insertion Sample \(Python\)

This Python sample illustrates how to access the Data Insertion API.

```
import httplib

xml='<?xml version="1.0" encoding="UTF-8"?>
<request> <scXmlVer>1.0</scXmlVer>
<reportSuiteID>Corp1_rs</reportSuiteID>
<timestamp>2010-03-20T10:33:22-07</timestamp> <visitorID>169</visitorID> <ipAddress>10.0.0.1</ipAddress> <pageName>Test Page</pageName></request>'

conn = httplib.HTTPConnection("namespace.sc.omtrdc.net:80")
conn.request("POST", "/b/ss//6",xml )
response = conn.getresponse()
print(response.status, response.reason)
print(response.read())
```

**Parent topic:** [Sample Code](../sample_code/c_sample_code.md)

