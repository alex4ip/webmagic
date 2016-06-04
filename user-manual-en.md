Webmagic Manual
========
> The new document addresses [http://webmagic.io/docs/](http://webmagic.io/docs/), this manual is no longer updated.


<div style="page-break-after:always"></div>

--------

--------

## Download and install

### Using maven

webmagic use maven dependency management, add the corresponding dependencies in the project can be used webmagic:

<dependency>
            <GroupId> us.codecraft </ groupId>
            <ArtifactId> webmagic-core </ artifactId>
            <Version> 0.4.2 </ version>
        </ Dependency>
<dependency>
            <GroupId> us.codecraft </ groupId>
            <ArtifactId> webmagic-extension </ artifactId>
            <Version> 0.4.2 </ version>
        </ Dependency>

#### Project structure

webmagic consists of two packages:

* ** Webmagic-core **

webmagic core part, only contains the basic modules and basic reptile extractor. webmagic-core goal is to become a textbook pages reptile-like implementation.

* ** Webmagic-extension **

webmagic expansion module that provides some of the more convenient tool written in reptiles. Including annotation format definition reptiles, JSON, distributed and other support.

webmagic also includes two expansion packs available, because these two packages are dependent on a relatively heavyweight tool, it pulled out from the main package, these packages need to download the source code after compile it yourself:

* ** Webmagic-saxon **

webmagic and Saxon binding module. Saxon is a XPath, XSLT analytical tools, webmagic rely Saxon to XPath2.0 parsing support.

* ** Webmagic-selenium **

webmagic and Selenium combined modules. Selenium is an analog browser page rendering tools, webmagic rely Selenium crawl dynamic pages.

In the project, you can according to need to rely on different packages.

### Does not use maven

Do not use maven users can download binary jar included with the package version (thanks [oschina](http://www.oschina.net/)):

git clone http://git.oschina.net/flashsword20/webmagic.git

In ** lib ** directory, there are all the project dependencies jar package, you can import directly in the IDE.

--------

## The first crawler

### Custom PageProcessor

PageProcessor is part webmagic-core, custom a PageProcessor to achieve their reptilian logic. The following is a piece of code to crawl osc blog:

```java
    public class OschinaBlogPageProcesser implements PageProcessor {

        private Site site = Site.me().setDomain("my.oschina.net")
           .addStartUrl("http://my.oschina.net/flashsword/blog");

        @Override
        public void process(Page page) {
            List<String> links = page.getHtml().links().regex("http://my\\.oschina\\.net/flashsword/blog/\\d+").all();
            page.addTargetRequests(links);
            page.putField("title", page.getHtml().xpath("//div[@class='BlogEntity']/div[@class='BlogTitle']/h1").toString());
            page.putField("content", page.getHtml().$("div.content").toString());
            page.putField("tags",page.getHtml().xpath("//div[@class='BlogTags']/a/text()").all());
        }

        @Override
        public Site getSite() {
            return site;

        }

        public static void main(String[] args) {
            Spider.create(new OschinaBlogPageProcesser())
                 .pipeline(new ConsolePipeline()).run();
        }
    }
```
Here to add a URL to be crawled through page.addTargetRequests() method, and by page.putField() to save the extraction results. page.getHtml(). xpath() is in accordance with the results of a rule extraction, where extraction support chained calls. After this call, toString() representation into a single String, all() is converted to a String list.

Spider is a crawler entrance class. Pipeline is the result of persistence and output interfaces, here ConsolePipeline show the results to the console.

The main method of execution, you can see the results in the console crawl. webmagic default there are three seconds crawl space, please be patient. You can change this value by site.setSleepTime (int). There are some changes to crawl site attributes.

#### Using annotations

webmagic-extension includes a method for the preparation of reptiles annotation mode, simply based on a POJO annotated increase to complete a reptile. The following blog is still crawling oschina piece of code that is identical in functionality and OschinaBlogPageProcesser:

```java
	@TargetUrl("http://my.oschina.net/flashsword/blog/\\d+")
	public class OschinaBlog {

	    @ExtractBy("//title")
	    private String title;

	    @ExtractBy(value = "div.BlogContent",type = ExtractBy.Type.Css)
	    private String content;

	    @ExtractBy(value = "//div[@class='BlogTags']/a/text()", multi = true)
	    private List<String> tags;

	    @Formatter("yyyy-MM-dd HH:mm")
	    @ExtractBy("//div[@class='BlogStat']/regex('\\d+-\\d+-\\d+\\s+\\d+:\\d+')")
	    private Date date;

	    public static void main(String[] args) {
	        OOSpider.create(
	        	Site.me().addStartUrl("http://my.oschina.net/flashsword/blog"),
				new ConsolePageModelPipeline(), OschinaBlog.class).run();
	    }
	}
```

This example defines a Model class, Model class field 'title', 'content', 'tags' attributes are to be extracted. Pipeline in this class can be multiplexed.

Annotation detail description see below in the comments webmagic-extension module.

<div style = "page-break-after: always"> </ div>

--------

## Module Details

## Webmagic-core

webmagic-core is the core framework reptiles, reptile includes only a core function of the functional modules. webmagic-core goal is to become a textbook pages reptile-like implementation.

This section is part of an excerpt from the author's blog
[Webmagic mechanism and design principle - how to develop a Java reptile](http://my.oschina.net/flashsword/blog/145796).

### Webmagic-core module division

webmagic-core reference module division scrapy, divided into Spider (crawler entire scheduling framework), Downloader (download page), PageProcessor (extract links and page analysis), Scheduler (URL management), Pipeline (off-line analysis and persistence) sections. Just scrapy achieve expansion through middleware, and webmagic is defined by these interfaces, and different implementations injection Spider main frame class to achieve expansion.

![Image](http://code4craft.github.io/images/posts/webmagic.png)
<div style = "page-break-after: always"> </ div>

#### Spider classes (core dispatch)

** Spider ** reptile entry classes, interfaces, call Spider uses a chain of API design, other features all through the interface injection Spider realized, the following is to start a more complex example of Spider.

```java
    Spider.create(sinaBlogProcessor)
	.scheduler(new FileCacheQueueScheduler("/data/temp/webmagic/cache/"))
	.pipeline(new FilePipeline())
	.thread(10).run();
```

Spider core processing process is very simple, as follows:

```java
    private void processRequest(Request request) {
        Page page = downloader.download(request, this);
        if (page == null) {
            sleep(site.getSleepTime());
            return;
        }
        pageProcessor.process(page);
        addRequest(page);
        for (Pipeline pipeline : pipelines) {
            pipeline.process(page, this);
        }
        sleep(site.getSleepTime());
    }
```
    
Spider also includes a method of test (String url), the only method to grab a separate page for testing the effect of extraction.
    
#### PageProcessor (page analysis and link extraction)

Page analysis is vertical crawlers need to customize parts. In webmagic-core, through the realization ** PageProcessor ** interfaces to implement custom crawlers. PageProcessor has two core methods: public void process (Page page) and public Site getSite().

* Public void process (Page page)

By ** Page ** object operation, and reptiles logic. Page objects include two of the most important methods: addTargetRequests() can add a URL to be crawled queue, put() can save the results for subsequent processing.
Page data can be obtained through Page.getHtml() and Page.getUrl().

* Public Site getSite()

** Site ** crawler domain objects define the start address, crawl space, coding and other information.

** Selector ** webmagic is to simplify the development of stand-alone modules page extraction, is the main focus of webmagic-core. Here we integrate CSS Selector, XPath and regular expressions, and can be chained extraction.

```java
    //content with other crawlers tools to extract text
    List<String> links = page.getHtml()
    .$("div.title")  //css select，Java while few in the $ symbol appears，But it looks like $ as the method name is legal
    .xpath("//@href")  //Extract links
    .regex(".*blog.*") //Regular filter match
    .all(); //Convert string list
```

webmagic comprises a body of the page for automatic extraction of class **SmartContentSelector**. 
I believe it will be used Evernote Clearly automatic text extraction technology is impressive. 
This technique is also called ** Readability **. Of course webmagic achieve Readability is still relatively rough, but there are still some learning value.

webmagic use of XPath parse the author of another open source project: Based Jsoup the XPath parser 
[Xsoup](https://github.com/code4craft/xsoup), Xsoup on XPath syntax of some extensions to support some custom function. 
Use these functions are XPath end add `/ name-of-function()`, 
for example: `" // div [@ class = 'BlogStat'] / regex ( '\\ d + - \\ d + - \\ d + \\ s + \\ d +: \\ d + ') "`.

<table>
    <tr>
        <td width = "100"> functions <td>
        <td> Description <td>
    <tr>
    <tr>
        <td width = "100"> text (n) <td>
        <td> n-th text node (0, it means all) <td>
    <tr>
        <tr>
        <td width = "100"> allText() <td>
        <td> all the text, including sub-node <td>
    <tr>
    <tr>
        <tr>
        <td width = "100"> tidyText() <td>
        <td> includes all text child nodes, and intelligently wrap <td>
    <tr>
    <tr>
        <td width = "100"> html() <td>
        <td> internal html (excluding the current tag itself) <td>
    <tr>
    <tr>
        <td width = "100"> outerHtml() <td>
        <td> external html (including the current tag itself) <td>
    <tr>
    <tr>
        <td width = "100"> regex (@ attr, expr, group) <td>
        <td> regular expressions, @ attr attribute is extracted (can be omitted), expr is an expression content, group capture group (can be omitted, the default is 0) <td>
    <tr>
<table>

Based Saxon, webmagic provides support XPath2.0 syntax. XPath2.0 syntax supports internal functions, logic control, 
is a complete language, if you are familiar with XPath2.0 grammar, it touches a try (the need to introduce ** webmagic-saxon ** package).

** Webmagic-samples ** bag with some of them as a site customized PageProcessor, for learning purposes.

#### Downloader (download page)

** Downloader ** webmagic is the interface in the download page, the main method:

* Public Page download (Request request, Task task)

** Request ** object encapsulates the URL to be crawled and other information, 
while Page contains the Html and other information pages are downloaded. Task is a wrapper task corresponding abstract interface Site information.

* Public void setThread (int thread)

Because Downloader generally involves connection pooling and other functions, these functions are closely related with the multi-threaded, so the definition of this method.

There are several Downloader implementation:

* HttpClientDownloader

** Apache HttpClient ** integrates the Downloader. Apache HttpClient (after 4.0 into HttpCompenent project) is a powerful Java http downloader, 
which supports custom HTTP header (for reptiles more useful is the User-agent, cookie, etc.), automatic redirect, connection multiplexing, cookie reservations, set many powerful agents.

* SeleniumDownloader

For some Javascript dynamically loaded web pages using only http Simulation tool, and can not get to the content of the page. 
This line of thought, there are two: one is unraveling analysis js logic, then reptiles to reproduce it; the other is: a built-in browser, 
direct access to the last page has been loaded. ** Webmagic-selenium ** Selenium integrated package to SeleniumDownloader, can crawl dynamic loading the page. 
Using selenium need to install some native tools, concrete steps can refer to the author's blog [use Selenium to crawl dynamic loading pages](http://my.oschina.net/flashsword/blog/147334)

#### Scheduler (URL management)

** Scheduler ** is webmagic management module, by implementing Scheduler can customize your URL Manager. Scheduler consists of two main methods:

* Public void push (Request request, Task task)

Crawl URL to be added to Scheduler. Request object is a URL of a package, including priorities, and a Map for storing data. 
Task still used to distinguish different tasks in a multiple task Scheduler utility can this distinction.

* Public Request poll (Task task)

Remove a request and subsequent execution in the Scheduler.

webmagic Scheduler There are currently three implementations:

* QueueScheduler

A simple memory queues, faster, and is thread safe.

* FileCacheQueueScheduler

Use queue file is saved, it can be used for lengthy download tasks, (manual stop or crash) when the task is stopped halfway, the next performance is still crawling resumes from the suspended URL.

* RedisScheduler

Use redis storage URL queue. By using the same redis storage server URL, webmagic can be easily deployed in a multi-machine, so as to achieve the effect of a distributed crawler.

#### Pipeline (subsequent processing and persistence)

** Pipeline ** is the final result of extraction output and persistence interface. It includes only one method:

* Public void process (ResultItems resultItems, Task task)

** ResultItems ** is an integrated object extraction results. By ResultItems.get (key) drawing result can be obtained. Task is also used to distinguish between objects of different tasks.

webmagic Pipeline includes the following implementation:

* ConsolePipeline

Direct output to the console, used for testing.

* FilePipeline

Output to a file, save each URL to a separate page to MD5 result URL as the file name. The constructor `public FilePipeline (String path)` define the storage path, use the following documents ** persistent classes, most of them use this method to specify a path **.

* JsonFilePipeline

In JSON output to a file (.json suffix), other FilePipeline same.

webmagic does not currently support persistence to the database, but in combination with other tools, persisted to the database is very easy. 
Here wish to look at [webmagic binding JFinal persisted to the database section of code](http://www.oschina.net/code/snippet_190591_23456). 
Because JFinal not currently support maven, so do not put this code in to webmagic-samples.

<div style = "page-break-after: always"> </ div>

-----

## Webmagic-extension

webmagic-extension is functional blocks in order to develop more convenient reptile achieved. These fully functional webmagic-core-based framework, including the preparation of notes in the form of reptiles, paging, distributed functions.

### Comment module

webmagic-extension modules including annotations. Why annotation mode?

Because PageProcessor of flexible, powerful, but does not address two questions:

* For a site, if you want to crawl URL in multiple formats, you must write in PageProcesser judgment logic, the code difficult to manage.
* Fetch a result there is no corresponding Model, does not comply with Java application development habits, with some very good framework can not be integrated.

Model class is annotated core itself is a POJO, the Model class for delivery, save the page to crawl the final result data. Annotations directly to extract data binding, to write and maintain.

By way of comment it is actually a PageProcessor implementation --ModelPageProcessor completed, so there is no impact on the webmagic-core code. Still crawling OschinaBlog program as an example:

```java
	@TargetUrl("http://my.oschina.net/flashsword/blog/\\d+")
	public class OschinaBlog {

	    @ExtractBy("//title")
	    private String title;

	    @ExtractBy(value = "div.BlogContent",type = ExtractBy.Type.Css)
	    private String content;

	    @ExtractBy(value = "//div[@class='BlogTags']/a/text()", multi = true)
	    private List<String> tags;

	    @Formatter("yyyy-MM-dd HH:mm")
	    @ExtractBy("//div[@class='BlogStat']/regex('\\d+-\\d+-\\d+\\s+\\d+:\\d+')")
	    private Date date;

	    public static void main(String[] args) {
	        OOSpider.create(
	        	Site.me().addStartUrl("http://my.oschina.net/flashsword/blog"),
				new ConsolePageModelPipeline(), OschinaBlog.class).run();
	    }
	}
```

Notes section includes the following:

* #### TargetUrl

"TargetUrl" indicates that the Model corresponds to crawl the URL, which contains two meanings: URL comply with this condition will be added to crawl queue; URL comply with this condition will be crawled this Model. ** SourceRegion ** TargetUrl can specify the URL of the extraction area (only supported XPath).

TargetUrl using regular expression matching URL "http://my.oschina.net/flashsword/blog/150039" format. webmagic regular expression is modified, representing the character only and do not represent any character "\ *" represents the, for example, "http". "". "" \ *. ": // \ * .oschina.net / \ * "represents the URL oschina all second-level domain under.

And there is a similar TargetUrl ** HelpUrl **, HelpUrl said: only crawl the URL link is used to extract, it is not content extraction. For example blog page text corresponds TargetUrl, and the list page corresponds HelpUrl.

* #### ExtractBy

* #### For the field

"ExtractBy" can be used for classes and fields. When used in the field defined field extraction rules. Extraction rules used by default [** XPath **](http://www.w3school.com.cn/xpath/), you can also choose to use CSS Selector, regular expression (by setting type).

ExtractBy several extended attributes. ** Multi ** indicates whether to draw a list, of course, is set to multi, you need a List field to accommodate it. ** Notnull ** it said that this field does not permit null, if null to discard the entire object.

* Class for ####

"ExtractBy" for the class, define a field extraction area. Still supported when used in multi class, multi indicates that a page can be extracted to multiple objects.

* #### ExtractByUrl

ExtractByUrl indicates extracting information from the URL, only supports regular expressions.

* #### ComboExtract

ComboExtract ExtractBy is a supplement, will support a combination of extraction rules with and or or form.

#### * Type conversion

webmagic annotation mode support for the drawing result type conversion, so that the result does not require extraction of type String, but can be any type. webmagic built basic types of support (need to ensure that the drawing result can be converted to the corresponding type).

```java
	    @ExtractBy("//ul[@class='pagehead-actions']/li[1]//a[@class='social-count js-social-count']/text()")
	    private int star;
```
Extraction result may be `java.util.Date` type, but need to specify the date format by:

```java
	    @Formatter("yyyy-MM-dd HH:mm")
	    @ExtractBy("//div[@class='BlogStat']/regex('\\d+-\\d+-\\d+\\s+\\d+:\\d+')")
	    private Date date;
```

You can also write a class that implements the `ObjectFormatter` interface, to conduct their own type resolution. To use your own class, you need to call `ObjectFormatters.put()` this class to register.

* #### AfterExtractor

AfterExtractor interface is a complement to the insufficient capacity to extract annotation mode. After realization AfterExtractor interface will be finished in the use of annotations populate fields ** ** call ** afterProcess() ** method, this method can directly access the decimated fields need to extract complementary fields, and even do some simple and the output of persistence operations (not very recommended). This section can refer to [webmagic binding JFinal persisted to the piece of code database](http://www.oschina.net/code/snippet_190591_23456).

* #### OOSpider
OOSpider entrance annotation type reptiles, called here **create()** method OschinaBlog this class was added to the extraction of the reptile, here it is can pass more than one class, for example:

```java
		OOSpider.create(
			Site.me().addStartUrl("http://www.oschina.net"),
			new ConsolePageModelPipeline(),
			OschinaBlog.clas,OschinaAnswer.class).run();
```

OOSpider will resolve according TargetUrl call different Model.

* #### PageModelPipeline
You can choose to define PageModelPipeline result output. Here new ConsolePageModelPipeline() is an implementation PageModelPipeline, the result will be output to the console.

PageModelPipeline now includes `ConsolePageModelPipeline`,`JsonFilePageModelPipeline`, `FilePageModelPipeline` three realized.

* #### Tab

Individual data processing tab (for example, a single press multiple pages) is a reptile troublesome issue. webmagic tab for the current solution is: In annotation mode, Model by implementing ** PagedModel ** interface, and the introduction of PagedPipeline to achieve as the first Pipeline. Specific reference webmagic-samples can grab the code Netease news:? ** Us.codecraft.webmagic.model.samples.News163 **.

About tab, there is an article for [some thoughts about reptiles implement paging](http://my.oschina.net/flashsword/blog/150039) a detailed description of webmagic tab implementation.
Currently distributed paging feature is not implemented if the implementation RedisScheduler distributed crawling, please do not use the paging function.
