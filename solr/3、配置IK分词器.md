
https://blog.csdn.net/yuanlaijike/article/details/79882684

solr自带

~~~xml
<fieldType name="text_smartcn" class="solr.TextField" positionIncrementGap="0">
    <analyzer type="index">
      <tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory"/>
    </analyzer>
    <analyzer type="query">
      <tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory"/>
    </analyzer>
  </fieldType>
~~~



ik

~~~xml
 <fieldType name="text_ik" class="solr.TextField">
    <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer"/>
  </fieldType>
~~~

