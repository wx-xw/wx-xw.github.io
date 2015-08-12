---
layout: post
category: solr
tags: [solr,搜索]

---

## 定义域

### 示例

```xml
<field name="price" type="float" default="0.0" indexed="true" stored="true"/>
```

### 域属性

>域的所有属性已经在上文提到过了，域类型的定义的时候，会选择性的定义一些域属性的默认值，而在域定义的时候可以选择覆盖该值。

## 复制域

solr可以将多个域复制到同一个域中。

```xml
<copyField source="cat" dest="text" maxChars="30000" />
<copyField source="*_cat" dest="text" maxChars="30000" />
<copyField source="cat" dest="text_*" maxChars="30000" />
```


1. 如果dest的域中已经有值了，则会把source域中的值附加上去。dest的域需要设置multivalued属性为true。
2. maxChars是一个int型，如果需要控制索引域中单条数据的大小，避免索引过大，可以设置这个值。
3. 无论是source域还是dest域的名字都可以设置开始或者结束的通配符。
	1. source域使用通配符，dest域需要是一个指定的域，不能使用通配符。
	2. dest域使用通配符，source域需要是一个指定的域，不能使用通配符。

## 动态域

>与普通域类似，只是在name属性上使用通配符，这样可以增加schema的通用性。在不存在指定的域的时候才会选择使用动态域创建，以下边例子假设，如果已经存在一个名称为cost_i的域，则不在使用动态域了

```xml
<dynamicField name="*_i" type="int" indexed="true"  stored="true"/>
```

## 其他schema元素

### unique key

用来标识文档中的唯一键的，不是必须设置，但通常都会设置。特别在更新索引的时候用处很大。

- 复制域不能用来作为unique key
- 不能使用UIDUpdateProcessorFactory来自动生成
- 不能是多值

```xml
<uniqueKey>id</uniqueKey>
```

### 默认搜索域

如果使用Lucene的查询解析器，在搜索的时候没有指定某个域来搜索，则会使用defaultSearchField来搜索。
如果使用DisMax和Extended DisMax的查询解析器，并且没有指定qf参数的时候，也会使用这个域。
**<font color='red'>在solr3.6以及以上的版本，这个元素已经deprecated了，所以这个配置后期有可能被删除。</font>**

### 查询语法解析的默认操作符

solrQueryParser元素控制默认操作符，可以使AND或者OR。如果在查询中没有指定，则默认使用。使用Lucene的查询解析器，会使用这个元素。在DisMax或者Extended DisMax的查询解析器，如果没有指定，也是默认为OR，不使用这个元素。
**<font color='red'>在solr3.6以及以上的版本，这个元素已经deprecated了，所以最好在查询时候指定q.op参数</font>**

### similarity

一个Lucene中在搜索阶段给文本打分的类。

全局配置<similarity>，可以自定义一个打分实现，来适用特定的环境。

#### 无参的similarity

```xml
<similarity class="solr.DefaultSimilarityFactory"/>
```

#### 自定义参数的similarity

```xml
<similarity class="solr.DFRSimilarityFactory">
  <str name="basicModel">P</str>
  <str name="afterEffect">L</str>
  <str name="normalization">H2</str>
  <float name="c">7</float>
</similarity>
```

#### 特定域类型的similarity

```xml
<similarity class="solr.SchemaSimilarityFactory"/>
<fieldType name="text_ib">
   <analyzer/>
   <similarity class="solr.IBSimilarityFactory">
      <str name="distribution">SPL</str>
      <str name="lambda">DF</str>
      <str name="normalization">H2</str>
   </similarity>
</fieldType>
```
