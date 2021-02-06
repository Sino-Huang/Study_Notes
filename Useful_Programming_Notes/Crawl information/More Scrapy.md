# More scrapy

[TOC]



## json response 

```python
import scrapy
import json


class QuoteSpider(scrapy.Spider):
    name = 'quote'
    allowed_domains = ['quotes.toscrape.com']
    page = 1
    start_urls = ['http://quotes.toscrape.com/api/quotes?page=1']

    def parse(self, response):
        data = json.loads(response.text)
        for quote in data["quotes"]:
            yield {"quote": quote["text"]}
        if data["has_next"]:
            self.page += 1
            url = f"http://quotes.toscrape.com/api/quotes?page={self.page}"
            yield scrapy.Request(url=url, callback=self.parse)
            
            
        

```

### the Firefox get headers directly but the format is weird 

```
{
	"Request Headers (381 B)": {
		"headers": [
			{
				"name": "Accept",
				"value": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8"
			},
			{
				"name": "Accept-Encoding",
				"value": "gzip, deflate"
			},
			{
				"name": "Accept-Language",
				"value": "en-US,en;q=0.5"
			},
			{
				"name": "Cache-Control",
				"value": "max-age=0"
			},
			{
				"name": "Connection",
				"value": "keep-alive"
			},
			{
				"name": "Host",
				"value": "quotes.toscrape.com"
			},
			{
				"name": "Upgrade-Insecure-Requests",
				"value": "1"
			},
			{
				"name": "User-Agent",
				"value": "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:85.0) Gecko/20100101 Firefox/85.0"
			}
		]
	}
}
```

```python
def parse_firefox_headers(headers:dict):
    output = {}
    for header in headers.values():
        for h_array in header.values():
            for item_dict in h_array:
                output[item_dict['name']] = item_dict['value']
    return output
```

```python
request_test = scrapy.Request(url='http://quotes.toscrape.com/api/quotes?page=1', headers= parse_firefox_headers(firefox_headers))
```



## Parse js script 

[chompjs](https://github.com/Nykakin/chompjs) provides an API to parse JavaScript objects into a [`dict`](https://docs.python.org/3/library/stdtypes.html#dict).

```python
>>> import chompjs
>>> javascript = response.css('script::text').get()
>>> data = chompjs.parse_js_object(javascript)
>>> data
{'field': 'value', 'secondField': 'second value'}
```





## interact with js 

### use Splash 

#### Installation

Install scrapy-splash using pip:

```
$ pip install scrapy-splash
```

Scrapy-Splash uses [Splash](https://github.com/scrapinghub/splash) HTTP API, so you also need a Splash instance. Usually to install & run Splash, something like this is enough:

```
$ sudo docker pull scrapinghub/splash
$ sudo docker run --name splash -p 8050:8050 scrapinghub/splash
```

Check Splash [install docs](http://splash.readthedocs.org/en/latest/install.html) for more info.

https://github.com/scrapy-plugins/scrapy-splash



now you can go to http://localhost:8050/ 



create scrapy project 

```bash
scrapy startproject splashtest
cd splashtest
scrapy genspider quotes quotes.toscrape.com
```



spider/quotes.py

```python
import scrapy
import scrapy.utils.response as web
from scrapy_splash import SplashRequest
import time

class QuotesSpider(scrapy.Spider):
    name = 'quotes'


    def start_requests(self):
        url = 'https://angrybirds.fandom.com/wiki/Bad_Piggies_21-13?file=Angry_Birds_21-13_Bad_Piggies_3_Star_Walkthrough_%28Angry_Birds_Classic_21-13%29'
        request = SplashRequest(
            url=url,
            callback= self.parse
        )

        yield request

    def parse(self, response):
        time.sleep(5.0)
        web.open_in_browser(response)
        video = response.selector.xpath(f"//iframe[contains(@src, 'embed')]/@src").get()
        yield {"video" : video}


```



#### modify settings

```python
SPLASH_URL = 'http://localhost:8050'

DOWNLOADER_MIDDLEWARES = {
    'scrapy_splash.SplashCookiesMiddleware': 723,
    'scrapy_splash.SplashMiddleware': 725,
    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 810,
}

SPIDER_MIDDLEWARES = {
    'scrapy_splash.SplashDeduplicateArgsMiddleware': 100,
}

DUPEFILTER_CLASS = 'scrapy_splash.SplashAwareDupeFilter'
HTTPCACHE_STORAGE = 'scrapy_splash.SplashAwareFSCacheStorage'
```



1. Add the Splash server address to `settings.py` of your Scrapy project like this:

   ```
   SPLASH_URL = 'http://localhost:8050'
   ```

2. Enable the Splash middleware by adding it to `DOWNLOADER_MIDDLEWARES` in your `settings.py` file and changing HttpCompressionMiddleware priority:

   ```
   DOWNLOADER_MIDDLEWARES = {
       'scrapy_splash.SplashCookiesMiddleware': 723,
       'scrapy_splash.SplashMiddleware': 725,
       'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 810,
   }
   ```

   Order 723 is just before HttpProxyMiddleware (750) in default scrapy settings.

   HttpCompressionMiddleware priority should be changed in order to allow advanced response processing; see https://github.com/scrapy/scrapy/issues/1895 for details.

3. Enable `SplashDeduplicateArgsMiddleware` by adding it to `SPIDER_MIDDLEWARES` in your `settings.py`:

   ```
   SPIDER_MIDDLEWARES = {
       'scrapy_splash.SplashDeduplicateArgsMiddleware': 100,
   }
   ```

   This middleware is needed to support `cache_args` feature; it allows to save disk space by not storing duplicate Splash arguments multiple times in a disk request queue. If Splash 2.1+ is used the middleware also allows to save network traffic by not sending these duplicate arguments to Splash server multiple times.

4. Set a custom `DUPEFILTER_CLASS`:

   ```
   DUPEFILTER_CLASS = 'scrapy_splash.SplashAwareDupeFilter'
   ```

5. If you use Scrapy HTTP cache then a custom cache storage backend is required. scrapy-splash provides a subclass of `scrapy.contrib.httpcache.FilesystemCacheStorage`:

   ```
   HTTPCACHE_STORAGE = 'scrapy_splash.SplashAwareFSCacheStorage'
   ```

   If you use other cache storage then it is necesary to subclass it and replace all `scrapy.util.request.request_fingerprint` calls with `scrapy_splash.splash_request_fingerprint`.



## Meta and cb_kwargs

- `meta`

  A dict that contains arbitrary metadata for this request. This dict is empty for new Requests, and is usually  populated by different Scrapy components (extensions, middlewares, etc). So the data contained in this dict **depends on the extensions you have enabled**. See [Request.meta special keys](https://docs.scrapy.org/en/latest/topics/request-response.html#topics-request-meta) for a list of special meta keys recognized by Scrapy. <u>This dict is [shallow copied](https://docs.python.org/3/library/copy.html)</u> when the request is cloned using the `copy()` or `replace()` methods, and can also be accessed, in your spider, from the `response.meta` attribute.

  Those are:

  - [`dont_redirect`](https://docs.scrapy.org/en/latest/topics/downloader-middleware.html#std-reqmeta-dont_redirect)
  - [`dont_retry`](https://docs.scrapy.org/en/latest/topics/downloader-middleware.html#std-reqmeta-dont_retry)
  - [`handle_httpstatus_list`](https://docs.scrapy.org/en/latest/topics/spider-middleware.html#std-reqmeta-handle_httpstatus_list)
  - [`handle_httpstatus_all`](https://docs.scrapy.org/en/latest/topics/spider-middleware.html#std-reqmeta-handle_httpstatus_all)
  - [`dont_merge_cookies`](https://docs.scrapy.org/en/latest/topics/request-response.html#std-reqmeta-dont_merge_cookies)
  - [`cookiejar`](https://docs.scrapy.org/en/latest/topics/downloader-middleware.html#std-reqmeta-cookiejar)
  - [`dont_cache`](https://docs.scrapy.org/en/latest/topics/downloader-middleware.html#std-reqmeta-dont_cache)
  - [`redirect_reasons`](https://docs.scrapy.org/en/latest/topics/downloader-middleware.html#std-reqmeta-redirect_reasons)
  - [`redirect_urls`](https://docs.scrapy.org/en/latest/topics/downloader-middleware.html#std-reqmeta-redirect_urls)
  - [`bindaddress`](https://docs.scrapy.org/en/latest/topics/request-response.html#std-reqmeta-bindaddress)
  - [`dont_obey_robotstxt`](https://docs.scrapy.org/en/latest/topics/downloader-middleware.html#std-reqmeta-dont_obey_robotstxt)
  - [`download_timeout`](https://docs.scrapy.org/en/latest/topics/request-response.html#std-reqmeta-download_timeout)
  - [`download_maxsize`](https://docs.scrapy.org/en/latest/topics/settings.html#std-reqmeta-download_maxsize)
  - [`download_latency`](https://docs.scrapy.org/en/latest/topics/request-response.html#std-reqmeta-download_latency)
  - [`download_fail_on_dataloss`](https://docs.scrapy.org/en/latest/topics/request-response.html#std-reqmeta-download_fail_on_dataloss)
  - [`proxy`](https://docs.scrapy.org/en/latest/topics/downloader-middleware.html#std-reqmeta-proxy)
  - `ftp_user` (See [`FTP_USER`](https://docs.scrapy.org/en/latest/topics/settings.html#std-setting-FTP_USER) for more info)
  - `ftp_password` (See [`FTP_PASSWORD`](https://docs.scrapy.org/en/latest/topics/settings.html#std-setting-FTP_PASSWORD) for more info)
  - [`referrer_policy`](https://docs.scrapy.org/en/latest/topics/spider-middleware.html#std-reqmeta-referrer_policy)
  - [`max_retry_times`](https://docs.scrapy.org/en/latest/topics/request-response.html#std-reqmeta-max_retry_times)



- `cb_kwargs`

  A dictionary that contains arbitrary metadata for this request. Its contents will be passed to the Request’s callback as keyword arguments. It is empty for new Requests, which means by default callbacks only get a [`Response`](https://docs.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response) object as argument. This dict is [shallow copied](https://docs.python.org/3/library/copy.html) when the request is cloned using the `copy()` or `replace()` methods, and can also be accessed, in your spider, from the `response.cb_kwargs` attribute. In case of a failure to process the request, this dict can be accessed as `failure.request.cb_kwargs` in the request’s errback. For more information, see [Accessing additional data in errback functions](https://docs.scrapy.org/en/latest/topics/request-response.html#errback-cb-kwargs).

