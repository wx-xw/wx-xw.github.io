---
layout: post
category: solr
tags: [solr,搜索]

---

## schema的API


### API入口

- `/schema`：展示schema信息，添加或者删除或者替换域、动态域、复制域或者域类型
- `/schema/fields`：展示域信息
- `/schema/dynamicfields`：展示动态域信息
- `/schema/fieldtypes`：展示域类型信息
- `/schema/copyfields`：展示复制域信息
- `/schema/name`：展示schema名称
- `/schema/version`：展示schema版本
- `/schema/uniquekey`：展示唯一键信息
- `/schema/similarity`：展示全局的similarity信息
- `/schema/solrqueryparser/defaultoperator`：展示默认搜索操作符

### 修改schema

> 需要使用POST请求

- `add-field`：加入一个新域
- `delete-field`：删除一个域
- `replace-field`：使用至少一个不同的属性替换一个存在的域
- `add-dynamic-field`：加入一个动态域
- `delete-dynamic-field`：删除一个动态域
- `replace-dynamic-field`：使用至少一个不同的属性替换一个存在的动态域
- `add-field-type`：加入一个域类型
- `delete-field-type`：删除一个域类型
- `replace-field-type`：使用至少一个不同的属性替换一个存在的域类型
- `add-copy-field`：加入一个复制域
- `delete-copy-field`：删除一个复制域

#### 加入一个域

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field":{
     "name":"sell-by",
     "type":"tdate",
     "stored":true }
}' http://localhost:8983/solr/gettingstarted/schema
```

#### 删除一个域

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-field" : { "name":"sell-by" }
}' http://localhost:8983/solr/gettingstarted/schema
```

#### 替换一个域

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-field":{
     "name":"sell-by",
     "type":"date",
     "stored":false }
}' http://localhost:8983/solr/gettingstarted/schema
```

#### 添加一个动态域规则

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-dynamic-field":{
     "name":"*_s",
     "type":"string",
     "stored":true }
}' http://localhost:8983/solr/gettingstarted/schema
```

#### 删除一个动态域规则

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-dynamic-field":{ "name":"*_s" }
}' http://localhost:8983/solr/gettingstarted/schema
```

#### 替换一个动态域规则

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-dynamic-field":{
     "name":"*_s",
     "type":"text_general",
     "stored":false }
}' http://localhost:8983/solr/gettingstarted/schema
```

#### 加入一个新的域类型

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field-type" : {
     "name":"myNewTxtField",
     "class":"solr.TextField",
     "positionIncrementGap":"100",
     "analyzer" : {
        "charFilters":[{
           "class":"solr.PatternReplaceCharFilterFactory",
           "replacement":"$1$1",
           "pattern":"([a-zA-Z])\\\\1+" }],
        "tokenizer":{
           "class":"solr.WhitespaceTokenizerFactory" },
        "filters":[{
           "class":"solr.WordDelimiterFilterFactory",
           "preserveOriginal":"0" }]}}
}' http://localhost:8983/solr/gettingstarted/schema
```

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field-type":{
     "name":"myNewTextField",
     "class":"solr.TextField",
     "indexAnalyzer":{
        "tokenizer":{
           "class":"solr.PathHierarchyTokenizerFactory",
           "delimiter":"/" }},
     "queryAnalyzer":{
        "tokenizer":{
           "class":"solr.KeywordTokenizerFactory" }}}
}' http://localhost:8983/solr/gettingstarted/schema
```

#### 删除一个域类型

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-field-type":{ "name":"myNewTxtField" }
}' http://localhost:8983/solr/gettingstarted/schema
```

#### 替换一个域类型

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-field-type":{
     "name":"myNewTxtField",
     "class":"solr.TextField",
     "positionIncrementGap":"100",
     "analyzer":{
        "tokenizer":{
           "class":"solr.StandardTokenizerFactory" }}}
}' http://localhost:8983/solr/gettingstarted/schema
```

#### 加入一个复制域

| 参数 | 必要参数 | 描述 |
|:-|:-|:-|
| source | 是 | 源域 |
| dest | 是 | 目标域 |
| maxChars | 否 | 最大存储字符数 |

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-copy-field":{
     "source":"shelf",
     "dest":[ "location", "catchall" ]}
}' http://localhost:8983/solr/gettingstarted/schema
```

#### 删除一个复制域

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-copy-field":{ "source":"shelf", "dest":"location" }
}' http://localhost:8983/solr/gettingstarted/schema
```

#### 一个post请求，多个命令

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field-type":{
     "name":"myNewTxtField",
     "class":"solr.TextField",
     "positionIncrementGap":"100",
     "analyzer":{
        "charFilters":[{
           "class":"solr.PatternReplaceCharFilterFactory",
           "replacement":"$1$1",
           "pattern":"([a-zA-Z])\\\\1+" }],
        "tokenizer":{
           "class":"solr.WhitespaceTokenizerFactory" },
        "filters":[{
           "class":"solr.WordDelimiterFilterFactory",
           "preserveOriginal":"0" }]}},
   "add-field" : {
      "name":"sell-by",
      "type":"myNewTxtField",
      "stored":true }
}' http://localhost:8983/solr/gettingstarted/schema
```

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field":{
     "name":"shelf",
     "type":"myNewTxtField",
     "stored":true },
  "add-field":{
     "name":"location",
     "type":"myNewTxtField",
     "stored":true },
  "add-copy-field":{
     "source":"shelf",
      "dest":[ "location", "catchall" ]}
}' http://localhost:8983/solr/gettingstarted/schema
```

同样的命令可以使用数组形式

```bash
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field":[
     { "name":"shelf",
       "type":"myNewTxtField",
       "stored":true },
     { "name":"location",
       "type":"myNewTxtField",
       "stored":true }]
}' http://localhost:8983/solr/gettingstarted/schema
```

#### 跨节点的schema修改

当使用SolrCloud模式，在一个节点上修改schema会最终同步到在集群中的所有节点。可以使用`updateTimeoutSecs`参数来指定更新schema的超时时间。

这个参数可以使得客户端程序可以知道修改schema后是否在特定时间内所有的节点都更新了schema，如果有节点没有更新，则会抛出异常。

如果不指定`updateTimeoutSecs`参数，默认则是把schema的配置更新到zookeeper后就返回。而其他节点会从zookeeper中更新配置信息，但是客户端是不清楚是否所有的节点更新成功的。

### 展示schema信息

#### 展示整个schema

`GET /collection/schema`

> 输出schema的所有信息

| 路径参数 | 描述 |
|:-|:-|
| collection | collection或者core名称 |

| 请求参数 | 类型 | 是否必需 | 默认值 | 描述 |
|:-|:-|:-|:-|:-|
| wt | string | 否 | json | 返回的数据格式，默认json，可选值：json，xml，schema.xml |

```bash
curl http://localhost:8983/solr/gettingstarted/schema?wt=json
curl http://localhost:8983/solr/gettingstarted/schema?wt=xml
curl http://localhost:8983/solr/gettingstarted/schema?wt=schema.xml
```

#### 列出域

`GET /collection/schema/fields`
`GET /collection/schema/fields/fieldname`

> 输出所有域或者特定域的信息

| 路径参数 | 描述 |
|:-|:-|
| collection | collection或者core名称 |
| fieldname | 特定的域名称 |

| 请求参数 | 类型 | 是否必需 | 默认值 | 描述 |
|:-|:-|:-|:-|:-|
| wt | string | 否 | json | 返回的数据格式，默认json，可选值：json，xml |

```bash
curl http://localhost:8983/solr/gettingstarted/schema/fields?wt=json

```

#### 列出动态域

`GET /collection/schema/dynamicfields`
`GET /collection/schema/dynamicfields/name`

> 输出所有动态域或者特定动态域的信息

| 路径参数 | 描述 |
|:-|:-|
| collection | collection或者core名称 |
| name | 特定的动态域名称 |

| 请求参数 | 类型 | 是否必需 | 默认值 | 描述 |
|:-|:-|:-|:-|:-|
| wt | string | 否 | json | 返回的数据格式，默认json，可选值：json，xml |

```bash
curl http://localhost:8983/solr/gettingstarted/schema/dynamicfields?wt=json
```

#### 列出域类型

`GET /collection/schema/fieldtypes`
`GET /collection/schema/fieldtypes/name`

> 输出所有域类型或者特定域类型的信息

| 路径参数 | 描述 |
|:-|:-|
| collection | collection或者core名称 |
| name | 特定的域类型名称 |

| 请求参数 | 类型 | 是否必需 | 默认值 | 描述 |
|:-|:-|:-|:-|:-|
| wt | string | 否 | json | 返回的数据格式，默认json，可选值：json，xml |

```bash
curl http://localhost:8983/solr/gettingstarted/schema/fieldtypes?wt=json
```

#### 列出复制域

`GET /collection/schema/copyfields`

> 输出所有复制域的信息

| 路径参数 | 描述 |
|:-|:-|
| collection | collection或者core名称 |

| 请求参数 | 类型 | 是否必需 | 默认值 | 描述 |
|:-|:-|:-|:-|:-|
| wt | string | 否 | json | 返回的数据格式，默认json，可选值：json，xml |

```bash
curl http://localhost:8983/solr/gettingstarted/schema/copyfields?wt=json
```

#### 展示schema名称

`GET /collection/schema/name`

> 输出schema的名称

| 路径参数 | 描述 |
|:-|:-|
| collection | collection或者core名称 |

| 请求参数 | 类型 | 是否必需 | 默认值 | 描述 |
|:-|:-|:-|:-|:-|
| wt | string | 否 | json | 返回的数据格式，默认json，可选值：json，xml |

```bash
curl http://localhost:8983/solr/gettingstarted/schema/name?wt=json
```

#### 展示schema版本

`GET /collection/schema/version`

> 输出schema的版本

| 路径参数 | 描述 |
|:-|:-|
| collection | collection或者core名称 |

| 请求参数 | 类型 | 是否必需 | 默认值 | 描述 |
|:-|:-|:-|:-|:-|
| wt | string | 否 | json | 返回的数据格式，默认json，可选值：json，xml |

```bash
curl http://localhost:8983/solr/gettingstarted/schema/version?wt=json
```

#### 列出唯一键

`GET /collection/schema/uniquekey`

> 输出schema中的唯一键

| 路径参数 | 描述 |
|:-|:-|
| collection | collection或者core名称 |

| 请求参数 | 类型 | 是否必需 | 默认值 | 描述 |
|:-|:-|:-|:-|:-|
| wt | string | 否 | json | 返回的数据格式，默认json，可选值：json，xml |

```bash
curl http://localhost:8983/solr/gettingstarted/schema/uniquekey?wt=json
```

#### 展示全局的similarity

`GET /collection/schema/similarity`

> 输出schema中的全局similarity

| 路径参数 | 描述 |
|:-|:-|
| collection | collection或者core名称 |

| 请求参数 | 类型 | 是否必需 | 默认值 | 描述 |
|:-|:-|:-|:-|:-|
| wt | string | 否 | json | 返回的数据格式，默认json，可选值：json，xml |

```bash
curl http://localhost:8983/solr/gettingstarted/schema/similarity?wt=json
```

#### 取得默认的搜索操作符

`GET /collection/schema/solrqueryparser/defaultoperator`

> 输出schema中的默认搜索操作符

| 路径参数 | 描述 |
|:-|:-|
| collection | collection或者core名称 |

| 请求参数 | 类型 | 是否必需 | 默认值 | 描述 |
|:-|:-|:-|:-|:-|
| wt | string | 否 | json | 返回的数据格式，默认json，可选值：json，xml |

```bash
curl http://localhost:8983/solr/gettingstarted/schema/solrqueryparser/defaultoperator?wt=json
```

## schema

### 整体架构

```xml
<schema>
  <types>
  <fields>
  <uniqueKey>
  <copyField>
</schema>
```

> types和fields是可选参数，即使选择把域类型和域的定义都放到顶层，也是可以解析成功的，加上types和fields为了逻辑更加清晰。

### 选择适当的数字类型

一般情况，使用基本数字类型，例如：TrieIntField, TrieLongField, TrieFloatField, and TrieDoubleField，直接设置precisionStep为0就好。

如果需要频繁的范围查询，则使用默认的precisionStep值，或者设置precisionStep为8，8就是默认值。这样可以提高范围查询的速度，但是会增加一定的存储空间。

## DocValues

DocValues是一种记录域值的方式，在排序、faceting，高亮等情况下，他的效率会高于传统的索引的效率。

### 为何DocValues

solr建立索引的方法是倒排索引。这种结构在搜索的时候非常快，但是再排序，faceting以及高亮的时候效率不太好。

例如：solr在facet阶段，会查找在文档中的每一个term来建立结果，而这些域都是在内存中，这样会降低读取的效率。

在Lucene4.0后引入了DocValues概念，DocValues域是一个列模式的域，在索引阶段，已经做好了文档到值的映射。这种实现相比fieldCache，降低了内存的使用，同时还使得faceting，排序以及聚集（grouping）更快。

### 怎样用DocValues

激活DocValues模式，只需要在域定义的时候添加`docValues="true"`。

- 如果在没有激活DocValues的情况下已经建立了索引，那么想要DocValues生效，必须完全重建索引。
- 特定域类型才可以使用DocValues
	- StrStrField和UUIDField
		- 如果是单值模式（multi-valued为false），Lucene将使用SORTED类型
		- 如果是多值模式，Lucene将使用SORTED_SET类型
	- 所有的Trie*数值域类型以及EnumField
		- 如果是单值模式（multi-valued为false），Lucene将使用NUMERIC类型
		- 如果是多值模式，Lucene将使用SORTED_SET类型

```xml
<field name="manu_exact" type="string" indexed="false" stored="false"
docValues="true" />
```

默认情况，DocValues会载入内存一部分，剩余部分放在磁盘上读取，但是也可以将其全部载入内存，只要提供就`docValuesFormat="Memory"`可以了

```xml
<fieldType name="string_in_mem_dv" class="solr.StrField" docValues="true" docValuesFormat="Memory" />
```