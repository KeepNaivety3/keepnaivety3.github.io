---
title: Python feedparser 库介绍
date: 2024-01-15
categories: [编程, Python]
tags: [Python, RSS]
---

> 前言：大概有小半年没有写 Python 了，打算写一个 Python 脚本将几个 RSS 订阅汇集为一个来复健一下。因此，我打算在服务器上部署一个 Crontab 定期执行 RSS 汇集脚本。这里有部分专业词汇在第一次出现时给出我认为的中文翻译，后文中均用英文标识。
{: .prompt-info }

> Source: https://feedparser.readthedocs.io/en/latest/index.html

## 1. 基本功能

### 1.1. 介绍

**Universal Feed Parser** 是一个集下载解析 `消息来源(syndicated feed)` 的 Python 模块。它能处理 RSS 0.90, Netscape RSS 0.91, Userland RSS 0.91, RSS 0.92, RSS 0.93, RSS 0.94, RSS 1.0, RSS 2.0, Atom 0.3, Atom 1.0, CDF 和 JSON 格式的 `信息流(feeds)`。它还能解析几种流行的扩展模块，包括 Dublin Core 和 Apple 的 iTunes 扩展。

要使用 **Universal Feed Parser**，你需要 Python 3.8 及以上版本。**Universal Feed Parser** 不能独立运行，它被用作一个大型 Python 程序中的模块。

**Universal Feed Parser** 简单易用，他包含一个主要的公共函数 `parse`。`parse` 必须含有一个参数，并且支持处理多个参数。参数可以是 URL、本地文件或包含任何格式 feed 数据的原始字符串。

#### 1.1.1. 解析来自远程 URL 的信息流

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/atom10.xml')
>>> d['feed']['title']
'Sample Feed'
```

#### 1.1.2 解析来自本地文件的信息流

下面的示例假定您使用的是 Windows 操作系统，而且您已经在 `C:\incoming\atom10.xml` 中保存了一个源文件。

> Universal Feed Parser 可在任何能运行 Python 的平台上运行；请使用适合您平台的路径语法。
{: .prompt-tip }

```python
>>> import feedparser
>>> d = feedparser.parse(r'C:\incoming\atom10.xml')
>>> d['feed']['title']
'Sample Feed'
```

#### 1.1.3 解析来自字符串的信息流

```python
>>> import feedparser
>>> rawdata = """<rss version="2.0">
<channel>
<title>Sample Feed</title>
</channel>
</rss>"""
>>> d = feedparser.parse(rawdata)
>>> d['feed']['title']
'Sample Feed'
```

返回值为 Python Unicode 字符串 (除非它们不是 - 请参阅 [URL]字符编码检测 了解所有细节)

### 1.2. 常见 RSS 元素

RSS feeds (无论什么版本) 中最常见的元素包括`标题(title)`、`链接(link)`、`简介(description)`、`发布日期(publication date)`、`条目 ID(entry ID)`。`publication date` 来自 `pubDate` 元素，`entry ID` 来自 `GUID` 元素。

这条 RSS feeds 示例位于 https://feedparser.readthedocs.io/en/latest/examples/rss20.xml。

```xml
<?xml version="1.0" encoding="utf-8"?>
<rss version="2.0">
    <channel>
        <title>Sample Feed</title>
        <description>For documentation &lt;em&gt;only&lt;/em&gt;</description>
        <link>http://example.org/</link>
        <pubDate>Sat, 07 Sep 2002 00:00:01 GMT</pubDate>
        <!-- other elements omitted from this example -->
        <item>
            <title>First entry title</title>
            <link>http://example.org/entry/3</link>
            <description>Watch out for &lt;span style="background-image:
                url(javascript:window.location='http://example.org/')"&gt;nasty
                tricks&lt;/span&gt;</description>
            <pubDate>Thu, 05 Sep 2002 00:00:01 GMT</pubDate>
            <guid>http://example.org/entry/3</guid>
            <!-- other elements omitted from this example -->
        </item>
    </channel>
</rss>
```

`频道(channel)` 元素可从 `d.feed` 得到。

#### 1.2.1. 访问常见 channel 元素

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/rss20.xml')
>>> d.feed.title
'Sample Feed'
>>> d.feed.link
'http://example.org/'
>>> d.feed.description
'For documentation <em>only</em>'
>>> d.feed.published
'Sat, 07 Sep 2002 00:00:01 GMT'
>>> d.feed.published_parsed
(2002, 9, 7, 0, 0, 1, 5, 250, 0)
```

`项目(item)` 可以在列表 `d.entries` 中找到。访问列表中 `item` 的顺序与它们在原始 feed 中出现的顺序相同，因此第一个项目在 `d.entries[0]` 中。

#### 1.2.2. 访问常见 item 元素

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/rss20.xml')
>>> d.entries[0].title
'First item title'
>>> d.entries[0].link
'http://example.org/item/1'
>>> d.entries[0].description
'Watch out for <span>nasty tricks</span>'
>>> d.entries[0].published
'Thu, 05 Sep 2002 00:00:01 GMT'
>>> d.entries[0].published_parsed
(2002, 9, 5, 0, 0, 1, 3, 248, 0)
>>> d.entries[0].id
'http://example.org/guid/1'
```

> 您还可以使用 Atom 术语访问 RSS feeds 中的数据。详情请参阅[URL]内容规范化。
{: .prompt-tip }

### 1.3. 常见 Atom 元素

Atom feeds 通常比 RSS feeds 包含更多的信息 (因为相较之下有更多必要的元素)。但是最常见的元素仍然是 `title`, `link`, `subtitle`/`description`, `various dates`, 和 `ID`.

这条 `Atom feeds` 示例位于 https://feedparser.readthedocs.io/en/latest/examples/atom10.xml。

```xml
<?xml version="1.0" encoding="utf-8"?>
<feed xmlns="http://www.w3.org/2005/Atom"
    xml:base="http://example.org/"
    xml:lang="en">
    <title type="text">Sample Feed</title>
    <subtitle type="html">
        For documentation &lt;em&gt;only&lt;/em&gt;
</subtitle>
    <link rel="alternate" href="/" />
    <link rel="self"
        type="application/atom+xml"
        href="http://www.example.org/atom10.xml" />
    <rights type="html">
        &lt;p>Copyright 2005, Mark Pilgrim&lt;/p>&lt;
</rights>
    <id>tag:feedparser.org,2005-11-09:/docs/examples/atom10.xml</id>
    <generator
        uri="http://example.org/generator/"
        version="4.0">
        Sample Toolkit
</generator>
    <updated>2005-11-09T11:56:34Z</updated>
    <entry>
        <title>First entry title</title>
        <link rel="alternate"
            href="/entry/3" />
        <link rel="related"
            type="text/html"
            href="http://search.example.com/" />
        <link rel="via"
            type="text/html"
            href="http://toby.example.com/examples/atom10" />
        <link rel="enclosure"
            type="video/mpeg4"
            href="http://www.example.com/movie.mp4"
            length="42301" />
        <id>tag:feedparser.org,2005-11-09:/docs/examples/atom10.xml:3</id>
        <published>2005-11-09T00:23:47Z</published>
        <updated>2005-11-09T11:56:34Z</updated>
        <summary type="text/plain" mode="escaped">Watch out for nasty tricks</summary>
        <content type="application/xhtml+xml" mode="xml"
            xml:base="http://example.org/entry/3" xml:lang="en-US">
            <div xmlns="http://www.w3.org/1999/xhtml">Watch out for <span
                    style="background: url(javascript:window.location='http://example.org/')">
                    nasty tricks</span></div>
        </content>
    </entry>
</feed>
```

`channel` 元素可从 `d.feed` 得到。

#### 1.3.1. 访问常见 feed 元素

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/atom10.xml')
>>> d.feed.title
'Sample feed'
>>> d.feed.link
'http://example.org/'
>>> d.feed.subtitle
'For documentation <em>only</em>'
>>> d.feed.updated
'2005-11-09T11:56:34Z'
>>> d.feed.updated_parsed
(2005, 11, 9, 11, 56, 34, 2, 313, 0)
>>> d.feed.id
'tag:feedparser.org,2005-11-09:/docs/examples/atom10.xml'
```

`entries` 可以在列表 `d.entries` 中找到。访问列表中 `entries` 的顺序与它们在原始 feed 中出现的顺序相同，因此第一个项目在 `d.entries[0]` 中。

#### 1.3.1. 访问常见 entries 元素

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/atom10.xml')
>>> d.entries[0].title
'First entry title'
>>> d.entries[0].link
'http://example.org/entry/3
>>> d.entries[0].id
'tag:feedparser.org,2005-11-09:/docs/examples/atom10.xml:3'
>>> d.entries[0].published
'2005-11-09T00:23:47Z'
>>> d.entries[0].published_parsed
(2005, 11, 9, 0, 23, 47, 2, 313, 0)
>>> d.entries[0].updated
'2005-11-09T11:56:34Z'
>>> d.entries[0].updated_parsed
(2005, 11, 9, 11, 56, 34, 2, 313, 0)
>>> d.entries[0].summary
'Watch out for nasty tricks'
>>> d.entries[0].content
[{'type': 'application/xhtml+xml',
'base': 'http://example.org/entry/3',
'language': 'en-US',
'value': '<div>Watch out for <span>nasty tricks</span></div>'}]
```

> 被解析的 `摘要(summary)` 和 `内容(contant)` 和原始 feed 中的内容不同。原始元素中包含危险的 HTML 标记已被净化，有关详细信息，请参阅[URL]净化。
{: .prompt-tip }

因为 Atom 的 `entries` 可以拥有多个 `content` 元素，因此 `d.entries[0].content` 是一个字典列表。每个词典包含每一个 `content` 元素的元数据。词典中最重要的两个值是 `content` 类型 `d.entries[0].content[0].type` 和 实际 content 值 `d.entries[0].content[0].value`。

你也可以从其他 Atom 元素中获得这种详细程度的信息。

### 1.4. 获取 Atom 元素的详细信息

几个 Atom 元素共享 Atom `content` 模型：`title`, `subtitle`, `rights`, `summary`, 和 `content` (Atom 0.3 也有一个共享此内容模型的 `info` 元素)。**Universal Feed Parser** 捕获所有包含这些元素的相关元数据，其中最重要的是 `content` 类型和值。

#### 1.4.1. Feed 元素中的详细信息

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/atom10.xml')
>>> d.feed.title_detail
{'type': 'text/plain',
'base': 'http://example.org/',
'language': 'en',
'value': 'Sample Feed'}
>>> d.feed.subtitle_detail
{'type': 'text/html',
'base': 'http://example.org/',
'language': 'en',
'value': 'For documentation <em>only</em>'}
>>> d.feed.rights_detail
{'type': 'text/html',
'base': 'http://example.org/',
'language': 'en',
'value': '<p>Copyright 2004, Mark Pilgrim</p>'}
>>> d.entries[0].title_detail
{'type': 'text/plain',
'base': 'http://example.org/',
'language': 'en',
'value': 'First entry title'}
>>> d.entries[0].summary_detail
{'type': 'text/plain',
'base': 'http://example.org/',
'language': 'en',
'value': 'Watch out for nasty tricks'}
>>> len(d.entries[0].content)
1
>>> d.entries[0].content[0]
{'type': 'application/xhtml+xml',
'base': 'http://example.org/entry/3',
'language': 'en-US'
'value': '<div>Watch out for <span> nasty tricks</span></div>'}
```

### 1.5. 不常见的 RSS 元素

这些元素并不常见，但是对于小众应用很有用，可能出现在任何 RSS feed 中。

RSS feed 能指定一个小图片被一些聚合器显示为图标(logo)。

#### 1.5.1. 访问 feed 图片

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/rss20.xml')
>>> d.feed.image
{'title': 'Example banner',
'href': 'http://example.org/banner.png',
'width': 80,
'height': 15,
'link': 'http://example.org/'}
```

feeds 和 entries 可以被分配给`多个分类(multiple categories)`，在某些版本的 RSS 中，类别可以与 域名 相关联，两者都是自由格式的字符串。因为一些历史原因，**Universal Feed Parser** 将 `multiple categories` 作为一个元组列表提供，而不是一个字典列表。

#### 1.5.2 访问 multiple categories

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/rss20.xml')
>>> d.feed.categories
[('Syndic8', '1024'),
('dmoz', 'Top/Society/People/Personal_Homepages/P/')]
```

RSS feed 的每个条目都可以有一个 `附件(enclosure)`，这是一个令人愉快的误称，它只是一个指向外部文件 (通常是音乐或视频文件，但任何类型的文件都可以被 "附件")。这种元素曾经很少见，但最近由于播客的兴起而逐渐流行起来。一些客户端 (例如苹果的 iTunes) 会自动下载 enclosure；另一些客户端 (例如网页端的Bloglines ~~已经寄了~~) 会简单地将每个 `enclosure` 视为一个链接。

RSS 规范规定，每个 `item` 最多只能有一个 `enclosure`。但是，Atom 的 `entries` 可能包含多个 `enclosure`，因此 **Universal Feed** 解析器会捕获所有 `enclosure`，并以列表形式提供。

#### 1.5.3 访问 enclosures

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/rss20.xml')
>>> e = d.entries[0]
>>> len(e.enclosures)
1
>>> e.enclosures[0]
{'type': 'audio/mpeg',
'length': '1069871',
'href': 'http://example.org/audio/demo.mp3'}
```

#### 1.5.4 访问 feed cloud

没有人确切知道云是什么。(我也不知道啥是feed cloud)

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/rss20.xml')
>>> d.feed.cloud
{'domain': 'rpc.example.com',
'port': '80',
'path': '/RPC2',
'registerprocedure': 'pingMe',
'protocol': 'soap'}
```

> 有关访问 RSS 元素的更多示例，请参阅注释示例 [URL]RSS 1.0、RSS 2.0 和带命名空间的 RSS 2.0。
{: .prompt-tip }

### 1.5. 不常见的 Atom 元素

这些元素并不常见，但是对于小众应用很有用，可能出现在任何 Atom feed 中。

除作者外，每个 Atom feed 或 `entries` 还可以有任意数量的 `贡献者(contributors)`。**Universal Feed Parser** 会以列表形式提供这些 `contributors`。

#### 1.5.1. 访问 contributors

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/atom10.xml')
>>> e = d.entries[0]
>>> len(e.contributors)
2
>>> e.contributors[0]
{'name': 'Joe',
'href': 'http://example.org/joe/',
'email': 'joe@example.org'}
>>> e.contributors[1]
{'name': 'Sam',
'href': 'http://example.org/sam/',
'email': 'sam@example.org'}
```

除了备用链接外，每个 Atom feed 或 `entries` 还可以有任意数量的其他链接。每个链接都由其 `type` 属性加以区分，即 MIME 风格的内容类型和其 rel 属性。

#### 1.5.2. 访问 multiple links

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/atom10.xml')
>>> e = d.entries[0]
>>> len(e.links)
4
>>> e.links[0]
{'rel': 'alternate',
'type': 'text/html',
'href': 'http://example.org/entry/3'}
>>> e.links[1]
{'rel': 'related',
'type': 'text/html',
'href': 'http://search.example.com/'}
>>> e.links[2]
{'rel': 'via',
'type': 'text/html',
'href': 'http://toby.example.com/examples/atom10'}
>>> e.links[3]
{'rel': 'enclosure',
'type': 'video/mpeg4',
'href': 'http://www.example.com/movie.mp4',
'length': '42301'}
```

> 有关访问 Atom 元素的更多示例，请参阅注释示例 [URL]Atom 1.0 和 Atom 0.3。
{: .prompt-tip }

### 1.6. 测试存在性

现实中的 feed 可能缺少元素，甚至是规范要求的元素。在获取元素值之前，应始终测试元素是否存在。永远不要假设元素存在。

要测试元素是否存在，可以使用标准的 Python 字典函数。

#### 1.6.1 测试元素是否存在

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/atom10.xml')
>>> 'title' in d.feed
True
>>> 'ttl' in d.feed
False
>>> d.feed.get('title', 'No title')
'Sample feed'
>>> d.feed.get('ttl', 60)
60
```

## 2. 高级功能

### 2.1. 日期分析

不同的 feed 类型和版本使用的日期格式大相径庭。**Universal Feed Parser** 会尝试自动检测任何日期元素中使用的日期格式，并将其解析为一个 UTC 标准 Python 元组格式(exp: (2004, 1, 1, 19, 48, 21, 3, 1, 0))，正如 Python [time 模块](https://docs.python.org/3/library/time.html) 中记录的那样。

以下元素被解析为日期：

- `feed.updated` -> `feed.updated_parsed`
- `entries[i].published` -> `entries[i].published_parsed`
- `entries[i].updated` -> `entries[i].updated_parsed`
- `entries[i].created` -> `entries[i].created_parsed`
- `entries[i].expired` -> `entries[i].expired_parsed`

#### 2.1.1. 日期格式的历史

略

#### 2.1.2. 认可的日期格式

略

#### 2.1.3. 支持其他日期格式

**Universal Feed Parser** 支持多种不同的日期格式，但可能还有更多格式尚未得到支持。如果你发现了其他日期格式，可以使用 `registerDateHandler` 注册来支持它们。它只接受一个参数，即一个回调函数。回调函数应接收一个参数（字符串）并返回一个值 (UTC 标准 Python 元组格式)。

#### 2.1.4. 注册第三方日期处理程序

```python
import feedparser
import re

_my_date_pattern = re.compile(
    r'(\d{,2})/(\d{,2})/(\d{4}) (\d{,2}):(\d{2}):(\d{2})')

def myDateHandler(aDateString):
    """parse a UTC date in MM/DD/YYYY HH:MM:SS format"""
    month, day, year, hour, minute, second = \
        _my_date_pattern.search(aDateString).groups()
    return (int(year), int(month), int(day), \
        int(hour), int(minute), int(second), 0, 0, 0)

feedparser.registerDateHandler(myDateHandler)
d = feedparser.parse(...)
```

略

### 2.2. 净化

大多数 feed 都会在 feed 元素中嵌入 HTML 标记。有些 feed 甚至嵌入其他类型的标记，例如 SVG 或 MathML。由于许多 feed 聚合器使用网页或者网页组件来显示内容，因此 **Universal Feed Parser** 会对嵌入的标记进行净化，以去除可能带来安全风险的内容。

默认情况下，会对这些元素进行净化：

- `entries[i].content`
- `entries[i].summary`
- `entries[i].title`
- `feed.info`
- `feed.rights`
- `feed.subtitle`
- `feed.title`

> 如果内容声明为 text/plain，则不会对其进行净化，这是为了避免数据丢失。建议检查内容类型，例如 `entries[i].summary_detail.type`。如果是 text/plain，则未进行消毒处理，需要在渲染内容前应执行 HTML 转义。
{: .prompt-tip }

#### 2.2.1 HTML 净化

https://feedparser.readthedocs.io/en/latest/html-sanitization.html#html-sanitization

#### 2.2.2. SVG 净化

https://feedparser.readthedocs.io/en/latest/html-sanitization.html#svg-sanitization

#### 2.2.3. MathML 净化

https://feedparser.readthedocs.io/en/latest/html-sanitization.html#mathml-sanitization

#### 2.2.4. CSS 净化

https://feedparser.readthedocs.io/en/latest/html-sanitization.html#css-sanitization

#### 2.2.5. 使用白名单，不要使用黑名单

经常有人问我，为什么 **Universal Feed Parser** 对 HTML 和 CSS 净化如此苛刻。为了说明这个问题，这里列出了一份潜在危险的 HTML 标记和属性的不完整列表：

- script，可能包含恶意脚本
- applet, embed 和 object，可自动下载和执行恶意代码
- meta，可包含恶意重定向
- onload, onunload 和所有其他 on* 属性，可包含恶意脚本
- style, link 和 style 属性，可包含恶意脚本

style? 是的，CSS 定义可以包含可执行代码。

略 https://feedparser.readthedocs.io/en/latest/html-sanitization.html#whitelist-don-t-blacklist

#### 2.2.6. 禁用 HTML 净化

虽然不推荐使用，但可以通过向 `feedparser.parse()` 传递 `sanitize_html=False` 来禁用通用 feed 解析器的 HTML 净化功能。当传递此标记时，您将负责手动清除 Feed 中的 HTML。

[How to consume RSS safely](https://web.archive.org/web/20080826033749/http://diveintomark.org/archives/2003/06/12/how_to_consume_rss_safely): Explains the platypus attack.

### 2.3. 格式化内容

**Universal Feed Parser** 可解析多种不同类型的 feed：Atom, CDF, 以及九种不同格式的 RSS。你不必被迫学习这些格式之间的差异。**Universal Feed Parser** 会尽最大努力确保你能以相同的方式处理所有馈送，无论其格式或版本如何。

你可以使用 RSS 关键词访问 Atom feed 中的基本要素。

#### 2.3.1 以 RSS feed 方式访问 Atom feed

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/atom10.xml')
>>> d['channel']['title']
'Sample Feed'
>>> d['channel']['link']
'http://example.org/'
>>> d['channel']['description']
'For documentation <em>only</em>
>>> len(d['items'])
1
>>> e = d['items'][0]
>>> e['title']
'First entry title'
>>> e['link']
'http://example.org/entry/3'
>>> e['description']
'Watch out for nasty tricks'
>>> e['author']
'Mark Pilgrim (mark@example.org)'
```

#### 2.3.2 以 Atom feed 方式访问 RSS feed

```python
>>> import feedparser
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/rss20.xml')
>>> d.feed.subtitle_detail
{'type': 'text/html',
'base': 'http://feedparser.org/docs/examples/rss20.xml',
'language': None,
'value': 'For documentation <em>only</em>'}
>>> len(d.entries)
1
>>> e = d.entries[0]
>>> e.links
[{'rel': 'alternate',
'type': 'text/html',
'href': 'http://example.org/item/1'}]
>>> e.summary_detail
{'type': 'text/html',
'base': 'http://feedparser.org/docs/examples/rss20.xml',
'language': 'en',
'value': 'Watch out for <span>nasty tricks</span>'}
>>> e.updated_parsed
(2002, 9, 5, 0, 0, 1, 3, 248, 0)
```

> 有关 **Universal Feed Parser** 如何对不同格式的内容进行规范化处理的更多示例，请参阅注释示例。
{: .prompt-tip }

### 2.4. 命名空间处理

略

### 2.5. 相对链接解析

略

### 2.6. feed 类型和版本检测

**Universal Feed Parser** 会自动检测所解析馈送的类型和版本。不同版本的 RSS 之间有许多微妙或不太微妙的区别，应用程序可能会选择以不同的方式处理不同的源类型。

#### 2.6.1. 访问 feed 版本

```python
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/atom10.xml')
>>> d.version
'atom10'
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/atom03.xml')
>>> d.version
'atom03'
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/rss20.xml')
>>> d.version
'rss20'
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/rss20dc.xml')
>>> d.version
'rss20'
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/rss10.rdf')
>>> d.version
'rss10'
```

以下是 `version` 中可能返回的已知信息源类型和版本的完整列表：

略 https://feedparser.readthedocs.io/en/latest/version-detection.html#accessing-feed-version

如果 feed 类型完全未知，`version` 将为空字符串。

### 2.7. 字符编码检测

略

### 2.8. Bozo 检测

**Universal Feed Parser** 可以解析 feed，无论其是否为格式规范的 XML。不过，由于某些应用程序可能希望拒绝或警告用户非格式良好的 feed，因此 **Universal Feed Parser** 会在检测到 feed 格式不佳时设置 `bozo` 位。感谢 Tim Bray 提出这一术语。

#### 2.8.1. 检测格式不规范的 feed

```python
>>> d = feedparser.parse('https://feedparser.readthedocs.io/en/latest/examples/atom10.xml')
>>> d.bozo
0
>>> d = feedparser.parse('http://feedparser.org/tests/illformed/rss/aaa_illformed.xml')
>>> d.bozo
1
>>> d.bozo_exception
<xml.sax._exceptions.SAXParseException instance at 0x00BAAA08>
>>> exc = d.bozo_exception
>>> exc.getMessage()
"expected '>'\\n"
>>> exc.getLineNumber()
6
```

除了这个例子 (不完整的结束标记) 之外，XML 文档不符合格式的原因还有很多，请参阅[URL]字符编码检测，了解其他一些绊导致 Bozo 的情况。

## 3. HTTP 功能

### 3.1. ETag 和 Last-Modified 标头

略

### 3.2. 用户代理和引用标头

略

### 3.3. HTTP 重定向

略

### 3.4. 受密码保护的 feed

略

### 3.5. 其他 HTTP 标头

略