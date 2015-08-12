---
layout: post
category: solr
tags: [solr,搜索]

---

## 货币和汇率
>货币域类型提供对搜索时货币转换的支持，有以下特征
> 1. 小数查询
> 2. 范围查询
> 3. 函数范围查询
> 4. 排序
> 5. 货币编码或符号的解析
> 6. 对称以及不对称的汇率（当货币兑换有服务费的时候，不对称的汇率是有用的）

### 货币配置

```xml
<fieldType name="currency" class="solr.CurrencyField" precisionStep="8" defaultCurrency="USD" currencyConfig="currency.xml" />
```
在上边的例子中定义了name和class，并且定义了默认货币为美元，同时定义了一个汇率计算的文件“currency.xml”。

### 汇率

>两种方式提供汇率：FileExchangeRateProvider和OpenExchangeRatesOrgProvider

### FileExchangeRateProvider

这种是默认的配置方式，需要提供一个汇率的文件，属性名称为currencyConfig。可以命名为currency.xml，和schema.xml在同一个文件夹

```xml
<currencyConfig version="1.0">
  <rates>
    <!-- Updated from http://www.exchangerate.com/ at 2011-09-27 -->
    <rate from="USD" to="ARS" rate="4.333871" comment="ARGENTINA Peso" />
    <rate from="USD" to="AUD" rate="1.025768" comment="AUSTRALIA Dollar" />
    <rate from="USD" to="EUR" rate="0.743676" comment="European Euro" />
    <rate from="USD" to="CAD" rate="1.030815" comment="CANADA Dollar" />
    <!-- Cross-rates for some common currencies -->
    <rate from="EUR" to="GBP" rate="0.869914" />
    <rate from="EUR" to="NOK" rate="7.800095" />
    <rate from="GBP" to="NOK" rate="8.966508" />
    <!-- Asymmetrical rates -->
    <rate from="EUR" to="USD" rate="0.5" />
  </rates>
</currencyConfig>
```

### OpenExchangeRatesOrgProvider

这种方式代表可以从一个api中调用汇率，这样做比较实时

```xml
<fieldType name="currency" class="solr.CurrencyField" precisionStep="8"
           providerClass="solr.OpenExchangeRatesOrgProvider"
           refreshInterval="60"
ratesFileLocation="http://www.openexchangerates.org/api/latest.json?app_id=yourPersonalAppIdKey"/>
```
>refreshInterval属性是代表分钟的，所以60代表每个小时会下载一次汇率信息

## 时间日期

### 时间日期格式

YYYY-MM-DDThh:mm:ssZ
- YYYY：年
- MM：月
- DD：日
- hh：小时，24小时
- mm：分钟
- ss：秒钟
- Z：UTC

>不能设置时区，所有的时间都是标准时间

### 时间日期范围格式

- 2000-11：2000年11月整月
- 2000-11T13：2000年11月中，每天的下午1点到2点之间
- -0009：公元前10年
- [2000-11-01 TO 2014-12-01]：特定日期的时间范围查询
- [2014 TO 2014-12-01]：年初到特定日期
- [* TO 2014-12-01]：从最初的日期到特定日期

>不足：范围查询不支持嵌入时间表达式

### 时间表达式

#### 时间表达式语法

- NOW+2MONTHSNOW-1DAY：两个月后
- NOW-1DAY：一天前
- NOW/HOUR：当前小时
- NOW+6MONTHS+3DAYS/DAY：六个月三天后的起始，即为零点
- 1972-05-20T17:33:18.772Z+6MONTHS+3DAYS/DAY：1972年5月20日的六个月三天后的零点

#### 时间表达式的请求参数

- NOW
	- 这个参数时solr内部使用的，为了确保分布式的多节点的值的一致性
	- 这个参数可以被覆盖，由用户自定义数值，自定义数值需要精确到毫秒
	- `q=solr&fq=start_date:[* TO NOW]&NOW=1384387200000`

- TZ
	- 这个参数默认是标准时间
	- 这个参数可以被覆盖，有用户指定时区
	- 例子
		- 使用facet功能，聚合本月每天的数据量
		- 默认：http://localhost:8983/solr/my_collection/select?q=*:*&facet.range=my_date_field&facet=true&facet.range.start=NOW/MONTH&facet.range.end=NOW/MONTH%2B1MONTH&facet.range.gap=%2B1DAY
		- 使用洛杉矶时区：http://localhost:8983/solr/my_collection/select?q=*:*&facet.range=my_date_field&facet=true&facet.range.start=NOW/MONTH&facet.range.end=NOW/MONTH%2B1MONTH&facet.range.gap=%2B1DAY&TZ=America/Los_Angeles

```xml
<!-- 默认时区 -->
<int name="2013-11-01T00:00:00Z">0</int>
<int name="2013-11-02T00:00:00Z">0</int>
<int name="2013-11-03T00:00:00Z">0</int>
<int name="2013-11-04T00:00:00Z">0</int>
<int name="2013-11-05T00:00:00Z">0</int>
<int name="2013-11-06T00:00:00Z">0</int>
<int name="2013-11-07T00:00:00Z">0</int>
...
```

```xml
<!-- 洛杉矶时区 -->
<int name="2013-11-01T07:00:00Z">0</int>
<int name="2013-11-02T07:00:00Z">0</int>
<int name="2013-11-03T07:00:00Z">0</int>
<int name="2013-11-04T08:00:00Z">0</int>
<int name="2013-11-05T08:00:00Z">0</int>
<int name="2013-11-06T08:00:00Z">0</int>
<int name="2013-11-07T08:00:00Z">0</int>
...
```

#### DateRangeField

DateRangeField是TrieDateField的替代类，他们两个唯一不同在solr的xml活着solrj的返回个事的不同。DateRangeField返回String，TrieDateField返回Date。这也就意味着DateRangeField的索引更大一些。但是再搜索阶段，回比TrieDateField更快一些，特别是使用标准时间的时候。
这个类主要用于时间范围的查询，所以除了支持TrieDateField的基本语法外，同事还支持三种不同的范围查询：Intersects（默认），Contains，Winthin。
`fq={!field f=dateRange op=Contains}[2013 TO 2018]`

## 枚举

### 在schema.xml中定义一个枚举域

```xml
<fieldType name="priorityLevel" class="solr.EnumField" enumsConfig="enumsConfig.xml" enumName="severity"/>
<fieldType name="riskLevel" class="solr.EnumField" enumsConfig="enumsConfig.xml" enumName="risk" />
```
- enumsConfig：枚举值的配置文件
- enumName：枚举名，需要和枚举配置文件中的<enum/>标签中的名称匹配

### 用于定义枚举域的配置文件

```xml
<?xml version="1.0" ?>
<enumsConfig>
  <enum name="priority">
    <value>Not Available</value>
    <value>Low</value>
    <value>Medium</value>
    <value>High</value>
    <value>Urgent</value>
  </enum>
  <enum name="risk">
    <value>Unknown</value>
    <value>Very Low</value>
    <value>Low</value>
    <value>Medium</value>
    <value>High</value>
    <value>Critical</value>
  </enum>
</enumsConfig>
```

>不可以改变已存在的枚举的顺序，或者删除枚举值，除非重新创建索引。但是可以在最后加入新的枚举值

## 外部文件

### 外部文件域类型

**外部文件域类型使得在solr索引外部声明某个域的值成为可能。也即是说，solr从外部文件中取得某个域的值，而不是从索引文件中取得。**

>外部域不可被搜索，但是可以用于函数查询以及展示用。

一般情况，如果有一个域的更新频率远高于其他的域，例如：视频搜索的排序基于视频播放数，而播放数可以每小时都更新，而其他的属性，如视频标题，时长等则不需要经常更新，如果不适用外部文件域，则需要更新索引中的每个document，而适用外部文件域，则只需要更新外部文件即可。

```xml
<fieldType name="entryRankFile" keyField="pkId" defVal="0" stored="false" indexed="false" class="solr.ExternalFileField" valType="pfloat"/>
```

- keyField：在外部文件中定义的key，一般是索引的unique key
- defVal：默认值
- valType：只能是浮点数，所以属性值只能是pfloat，float或者tfloat。这个属性可以不设置

### 外部文件格式

- 必须在solr的索引目录中
- 名字需要有“external_”当做前缀，external_filename或者external_filename.*

>如果使用通配符.*，例如.txt，则solr会使用最新的一个文件，旧的会被删除掉，这是为了支持windows，因为windows下一旦一个文件被使用，则无法覆盖这个文件。

文件内容如下：

```bash
doc33=1.414
doc34=3.14159
doc40=42
```

>在文件中，key不需要唯一，并且也不需要排好序，但如果已经排好序，solr可以更快的载入它。

### 重新载入外部文件

当一个searcher重载或者当一个新的searcher生成的时候，需要定义一个事件监听器来重新载入外部文件。

```xml
<listener event="newSearcher" class="org.apache.solr.schema.ExternalFileFieldReloader"/>
<listener event="firstSearcher" class="org.apache.solr.schema.ExternalFileFieldReloader"/>
```

## 用例

这是一个比较通用的用例，需要特定域的属性或者域类型的支持。true和false代表特定属性在这个用例中必须设置的值，只有这样才能正确的实现功能。如果没有设置，则说明这个属性对于用例的功能没有影响。

| 用例 | indexed | stored | multiValued | omitNorms | termVectors | termPositions | docValues |
|--------|--------|--------|--------|--------|--------|--------|--------|
| 搜索 | true |||||||
| 展示 || true ||||||
| 唯一键 | true || false |||||
| 排序 | true[^7] || false | true[^1] ||| true[^7] |
| 域加权[^5] |||| false ||||
| 文本加权 |||| false ||||
| 高亮 | true[^4] | true ||| true[^2] | true[^3] ||
| faceting[^5] | true[^7] |||||| true[^7] |
| 多值并排序 ||| true |||||
| 域长度影响文本排序 |||| false ||||
| 相似性查询[^5] ||||| true[^6] |||


[^1]:建议但不是必须
[^2]:如果存在就会被使用，不是必须
[^3]:如果termVectors=true
[^4]:tokenizer必须在这个域中被定义，但是并不是一定要被索引
[^5]:分词，在其他章节介绍
[^6]:Term vectors并不是必须的，如果是true，则一个存储的域被分词。如果stored=false，term vectors是推荐的，但不强制。
[^7]:或者indexed或者docValues，必须为true，但不要求两个都为true。
