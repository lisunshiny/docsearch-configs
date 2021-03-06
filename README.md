# DocSearch configurations

This is the repository hosting the public [DocSearch](https://community.algolia.com/docsearch/) configurations.

DocSearch is composed of 3 different projects:
* The front-end JavaScript library: https://github.com/algolia/docsearch
* The scraper which browses & indexes web pages: https://github.com/algolia/docsearch-scraper
* The configurations for the scraper: https://github.com/algolia/docsearch-configs

If you want to run your own DocSearch instance on those configuration files, please get familiar with the [scraper setup guidelines](https://github.com/algolia/docsearch-scraper).

## Introduction

The DocSearch scraper will use a configuration file specifying:
 - the Algolia index name that will store the records resulting from the crawling
 - the URLs it needs to crawl
 - the URLs it shouldn't crawl
 - the (hierarchical) CSS selectors to use to extract the relevant content from your webpages
 - the CSS selectors to skip
 - An optional sitemap URL that will be crawled and then scraped
 - additional options you might provide to fine-tune the scraping

## How it works

Once you run the DocSearch scraper on a specific configuration, it will:
 - crawl all the URLs you specified (from the *start_urls* or the *sitemap*)
 - follow all the hyperlinks mentioned in the page, and continue the crawling there
 - stop the crawling as soon as you've reached a URL that is not specified in your configuration or affiliated to a start url
 - extract the content of every single crawled page following the logic you defined using the CSS selectors
 - push the resulting records to the Algolia index you configured

## Configuration format

A configuration file looks like:

```json
{
    "index_name": "stripe",
    "start_urls": [
        "https://stripe.com/docs"
    ],
    "stop_urls": [
        "https://stripe.com/docs/api"
    ],
    "selectors": {
      "lvl0": "#content header h1",
      "lvl1": "#content article h1",
      "lvl2": "#content section h3",
      "lvl3": "#content section h4",
      "lvl4": "#content section h5",
      "lvl5": "#content section h6",
      "text": "#content header p,#content section p,#content section ol"
    },
    "selectors_exclude": [
        ".method-list",
        "aside.note"
    ],
    // additional options
    [...]
}
```

### `index_name` ***Mandatory***

Name of the Algolia index where all the data will be pushed. If the `PREFIX` environment variable is defined, it will be prefixed
with it.

*This name must be equal to the configuration file name*

### `start_urls` ***Mandatory***
You can pass either a string or an array of urls. The crawler will go to each
page in order, following every link it finds on the page. It will only stop if
the domain is outside of the `allowed_domains` or if the link is blacklisted in
`stop_urls`.
Strings will be considered as regex.

Note that it currently does not follow 301 redirects.

### `selectors` ***Mandatory***

This object contains all the CSS selectors that will be used to create the
record hierarchy. It contains 6 levels (`lvl0`, `lvl1`, `lvl2`, `lvl3`, `lvl4`,
`lvl5`) and `text`. You should fill at least the three first levels for better
relevance.

A default config would be to target the page `title` or `h1` as `lvl0`, the `h2`
as `lvl1` and `h3` as `lvl2`. `text` is usually any `p` of text.

## Sitemap crawling ***Optional***

Our crawler offers you to crawl a site by discovering the URLs using Sitemaps. Thus, you will need to define the direct url(s) to your sitemap XML file, `sitemap_urls` , and then establish regex(s), `sitemap_urls_regexs` , which will match the URLs to crawl. Otherwise it will use the `start_urls` pattern in order to match the URLs available within the site map.

For sites that use Sitemap index files that point to other sitemap files, all those sitemaps will be followed.

###  `sitemap_urls`
A list of urls pointing to the sitemaps (or sitemap index) you want to crawl. Must be provided if you want to discover though sitemap.

###  `sitemap_urls_regexs`
A list of regular expression that will be applied to each URL from the sitemap. If the pattern match an URL, this link will be scrapped. If none regular expressions is defined, the start_urls will be taken as pattern.

###  `force_sitemap_urls_crawling`
Specifies if matched URL should not respect the same rules as the hyperlink crawled. If set to true, each URL will be scraped no matter if it suited the `start_urls` or `stop_urls`. Default is `force_sitemap_urls_crawling` disabled

#### Example
```json
[...]
"sitemap_urls": [
    "https://www.mySite.com/sitemap.xml"
  ],
"": [
    "/doc/"
  ],
"force_sitemap_urls_crawling": true,
[...]
```
Given this configuration, each webpage whose the URL contains '/doc/' will be scrapped even if they don't complied the `start_urls` or `stop_urls`


### Global selectors

It's possible to make a selector global which means that all records for the page will have
this value. This is useful when you have a title that is in the right sidebar and
the sidebar is placed after the content in the dom.

```json
"selectors": {
  "lvl0": {
    "selector": "#content header h1",
    "global": true
  }
}
```

### Xpath selector

By default `selectors` is considered to be a [css selector](https://www.w3schools.com/cssref/css_selectors.asp) but you can specify that a selector is a [xpath query](https://www.w3schools.com/xml/xpath_intro.asp).
This is useful when you want to do more complex selection like selecting the parent of a node.

```json
"selectors": {
  "lvl0": {
    "selector": "//li[@class=\"chapter active done\"]/../../a",
    "type": "xpath"
  }
}
```

### Default value

You have the possibility to add a default value. If the given selector doesn't match anything in a page
then for each record the default value will be set

```json
"selectors": {
  "lvl0": {
    "selector": "#content article h1",
    "default_value": "Documentation"
  }
}
```

### Strip Chars

You can override the default `strip_chars` per level

```json
"selectors": {
  "lvl0": {
    "selector": "#content article h1",
    "strip_chars": " .,;:"
  }
}
```

### `allowed_domains`

You can pass an array of strings. This is the whitelist of
domains the crawler will browse. If a link targets a page that is not in the
whitelist, the crawler will not follow it.

Default is the domain of the first element in the `start_urls`

### `stop_urls`

This is the blacklist of urls on which the crawler should stop. If a link in
a crawled webpage targets one the elements in the `stop_urls` list, the crawler
will not follow the link.

Note that you can use regexps as well as plain urls.

Note: It is sometimes needed to add `http://www.example.com/index.html` pages to
the `stop_urls` list if you set `http://www.example.com` as a `start_urls`, to
avoid duplicated content.

### `selectors_exclude`

By default, the `selectors` search is applied page-wide. If there are some parts
of the page that you do not want to include (like a header, sidebar or footer),
you can add them to the `selectors_exclude` key.

### `custom_settings`

This object is any custom Algolia settings you would like to pass to the index
settings.

### `min_indexed_level`

Lets you define the minimum level at which you want records to be indexed. For
example, with a `min_indexed_level: 1`, you will only index records that have at
least a `lvl1` field.

This is especially useful when the documentation is split into several pages,
but all pages duplicates the main title (see [this issue][1]).

### `js_render`

The HTML code that we crawl is sometimes generated using Javascript. In those
cases, the `js_render` option must be set to `true`. It will enable our
internal proxy (Selenium) to render pages before crawling them.

This parameter is optional and is set to `false` by default.

### `js_wait`

The `js_wait` parameter lets you change the default waiting time to render the
webpage with the Selenium proxy.

This parameter is optional and is set to `0`s by default.

### `use_anchors`

The `use_anchors` is need to be set to True for javascript doc when the hash is
used to route the query. Internally this will disable the canonicalize feature that
is removing the hash from the url.

This parameter is optional and is set to False by default.

### `strip_chars`

A list of character to remove from the text that is indexed.

Default is `" .,;:§¶"`

### `scrape_start_urls`

Default is `false`

### `remove_get_params`

Default is `false`

### `strict_redirect`

Default is `false`

### `nb_hits`

The number of object that should be indexed. Only used by the [`checker`](#checker).

Default is `0`.

## Possible issues

#### Duplicated content

It could happen that the crawled website returned duplicated data. Most of the time, this is because the crawled pages got the same urls with two different schemes.

If we have URLs like `http://website.com/page` and `http://website.com/page/` (notice the second one ending with `/`), the scrapper will consider them as different. This can be fixed by adding a regex to the `stop_urls` in the `config.json`:

```json
"stop_urls": [
  "/$"
]
```

In this attribute, you can also list the pages you want to skip:

```json
"stop_urls": [
  "http://website.com/page/"
]
```

#### Anchors

The scraper will also consider pages with anchors as different pages. Make sure you remove any hashsign from the urls you put in the stop & start URLs:

*Bad:*

```json
"stop_urls": [
  "http://website.com/page/#foo"
]
```

*Good:*

```json
"stop_urls": [
  "/$"
]
```

Or :

```json
"stop_urls": [
  "http://website.com/page/"
]
```
