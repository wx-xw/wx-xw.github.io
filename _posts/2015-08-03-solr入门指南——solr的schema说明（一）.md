---
layout: post
category: solr
tags: [solr,搜索]

---

#solr的schema说明（一）
###&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;——solr的域类型

##域类型定义以及属性
- 名称（必须）
- 域类型的java实现类（必须）
- 如果域类型是文本域，可以定义一种分词方式
- 域类型的属性，根据不同的域实现类，一些属性是不同的，一些属性是必须的

###域类型在schema.xml中定义
- 在schema.xml中定义
- 使用名字为fieldType的xml标签定义
- 域类型的实现类需要是存在的正确实现的类
- 如果实现类只以solr当包的开头，schema.xml解析的时候会自动找org.apache.solr.schema包和org.apache.solr.analysis包（solr.TextField=org.apache.solr.schema.TextField）

```xml
<fieldType name="text_general" class="solr.TextField" positionIncrementGap="100">
  <analyzer type="index">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
    <!-- in this example, we will only use synonyms at query time
    <filter class="solr.SynonymFilterFactory" synonyms="index_synonyms.txt" ignoreCase="true" expand="false"/> -->
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
  <analyzer type="query">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
    <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt"
ignoreCase="true" expand="true"/>
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
</fieldType>
```

###域类型属性
>不同的类，会存在一些不同的属性。

```xml
<fieldType name="date" class="solr.TrieDateField" sortMissingLast="true" omitNorms="true"/>
```

###域类型通用属性
| 属性 | 描述 | 值 |
|--------|--------|--------|
| name | 域类型的名称。这个值在域定义的“type”属性中使用。</br>强烈建议，但是不是强制性，这个值只以字母数字以及下划线组成并且不能以数字开头 ||
| class | 类名称，用于存储以及索引数据。如果以“solr.”开头，则solr可以寻找特定的包来找相应的类，但如果是使用第三方的类，则需要写全类的路径名 ||
| positionIncrementGap | 用于多值的域，在多个值之间声明一个距离，避免将多个值当成一个词组 | 整形 |
| autoGeneratePhraseQueries | 用于文本类型域。如果是true，solr会自动将相邻的term组成一个词组，如果是false，必须用双引号将term括起来，才能当做一个词组 | true或false |
| docValuesFormat | 在使用这个域类型的域中使用DocValuesFormat，需要有额外声明，例如在solrconfig.xml声明SchemaCodecFactory | n/a |
| postingsFormat | 与docValuesFormat类似 | n/a |

###域默认属性
| 属性 | 描述 | 值 |
|--------|--------|--------|
| indexed | 如果true，域可以用于检索到数据 | true或false |
| stored | 如果true，域的真实数据可以被展示 | true或false |
| dovValues | 如果true，这个域的值将放入列模式的数据结构中 | true或false |
| sortMissingFirst</br>sortMissingLast | 当排序域不存在的时候，用来控制文档的位置。solr3.5，这个配置用于所有的数字域中 | true或false |
| multiValued | 如果true，则说明一个单一的文档的这个域中可能存在多个值 | true或false |
| omitNorms | 如果true，则删除这个域的关联性（这个将删除一些长度信息以及索引时期这个域的权重，并且将节省一些内存）。所有的原始域类型（不分词的）都默认为true，例如：int，float，data，bool以及string。只有全文的域或者需要索引时加权重的域需要norms | true或false |
| omitTermFreqAndPositions | 如果true，删除这个域上的term的频率，位置以及payloads，可以节约一些存储空间。默认对除了文本域外所有域都为true | true或false |
| omitPositions | 和omitTermFreqAndPositions很想，只是保留了term频率。 | true或false |
| termVectors<br>termPositions<br>termOffsets<br>termPayloads | 这些属性设置为true可以加速高亮或者其他的辅助功能的效率，但是对于倒排索引所占空间的增长是可观的，一般是使用不到的 | true或false |
| required | 默认是false，如果是true，则这个域的值是必须的 | true或false |

##solr自带的域类型
| 属性 | 描述 |
|--------|--------|
| BinaryField | 二进制数据 |
| BoolField | 包含true或false，“1”、“t”或者“T”为首字母的值，则认为是true，其他都为false |
| CollationField | 为排序以及范围查询提供unicode校对功能，如果用ICU4J，则选择ICUCollationField更好一些 |
| CurrencyField | 支持货币以及汇率建立索引 |
| DateRangeField | 支持对于时间范围建立索引 |
| ExternalFileField |  |
| EnumField | 如果存在一系列的值需要排序，而又无法使用简单的字典或者数字排序的时候 |
| ICUCollationField | 为排序以及范围查询提供unicode校对功能 |
| LatLonType | 纬度经度坐标对，纬度在前 |
| PointType |  |
| PreAnalyzedField |  |
| RandomSortField | 没有值。如果用这个域排序，则返回的结果是一个随机顺序的结果 |
| SpatialRecursivePrefixTreeFieldType | 接受纬度，逗号，经度或者其他的WKT格式的搜索 |
| StrField | String |
| TextField | Text，经常是多个词 |
| TrieDateField | Date域。精确到毫秒级别。precisionStep=0，可以增加时间排序的效率并且减小索引的大小。默认precisionStep=8，可以增加范围查询的效率 |
| TrieDoubleField | Double域，64位IEEE浮点数。precisionStep=0，可以增加数值排序的效率并且减小索引的大小。默认precisionStep=8，可以增加范围查询的效率 |
| TrieField | 如果使用这个域类型，则“type”属性必须声明，值为：integer，long，float，double，date。precisionStep作用同上 |
| TrieFloatField | Float域，32位IEEE浮点数。precisionStep作用同上 |
| TrieIntField | Integer域。32位有符号整形。precisionStep作用同上 |
| TrieLongField | Long域。64位有符号整形。precisionStep作用同上 |
| UUIDField | 通用唯一识别码（Universally Unique Identifier） |
