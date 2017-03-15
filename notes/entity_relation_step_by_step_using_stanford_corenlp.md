# guide

> 这是基于[stanford-CoreNLP](http://stanfordnlp.github.io/CoreNLP/download.html)的实体关系抽取DEMO手册。
该手册包含三大部分的内容：DEMO的框架结构、详细实现细节以及安装使用说明。

# framework

> 该DEMO主要分为两块：实体关系抽取，数据反馈模型改进。
实体关系抽取包括nlp的常见任务（segment,pos-tagger,ner,parser,er），模型改进部分根据线上操作人员的反馈，
更新额外的自定义模型进行训练，并更新nlp任务所用到的模型。

实体关系抽取的流程如下：
1. 分词
2. 词性标注
3. 命名体识别
4. 关系抽取

各步骤是一个依次顺下的关系。而模型改进部分流程如下：
1. 指定目标句子
2. 根据反馈的结果添加相应预料到模型中（反馈的结果包括实体-关系，分词以及词性，命名体识别的相关修正结果，根据这些
结果，形成新的标注语料添加到自定义预料中，比如如果由于分词错误引起关系提取错误，首先需要将分词的正确结果加入分词
的训练预料，然后将改词填入外部词典保证分词的正确性，然后将词性的标注结果添加到词性训练预料（如有必要），然后将
命名体的正确结果加入到NER的训练预料，然后将实体-关系的结果加入到实体关系抽取的预料）
3. 重新训练待更新的模型
4. 重新载入训练好的模型

# details

## CoreNLP relate

### segment

#### 分词训练方法
        ```
        CRFClassifier crf = new CRFClassifier(props);
        SeqClassifierFlags flags = new SeqClassifierFlags(props);
        String serializeTo = flags.serializeToText;
        crf.train();
        crf.serializeClassifier(serializeTo);
        ```
> **注意**，props是训练所需配置的参数集合，训练分词模型需要准备好如下几个事项：
> 训练预料，"props"中的"trainFile"字段，已经切分好的文本，以行存储，比如：“我 爱 北京 天安门 。”
> 词典，"props"中的"setDictionary"字段，除了dict-chris6.ser.gz，还可以添加自定义的词典（人名、地名、机构名）
> 其他特征词典，"props"中的"sighanCorporaDict"字段，比如单字特征以及非词特征等
> 其他属性字段请参考 "edu/stanford/nlp/sequences/SeqClassifierFlags.java"中释义

#### 分词调用方法
        ```
        Properties props = PropertiesUtils.asProperties(
                        "model", "data/segment/ctb8.seg.ser.gz",
                        "sighanCorporaDict", "edu/stanford/nlp/models/segmenter/chinese",
                        "serDictionary", "edu/stanford/nlp/models/segmenter/chinese/dict-chris6.ser.gz",
                        "sighanPostProcessing", "true"
                );
        CRFClassifier<CoreLabel> segmenter = new CRFClassifier<CoreLabel>(props);
        segmenter.loadClassifierNoExceptions("data/segment/ctb8.seg.ser.gz", props);
        String sample1 = "我爱北京天安门"
        List<String> segmented1 = segmenter.segmentString(sample1);
        System.out.println(segmented1);
        ```

### pos-tagger

### ner

# user guide

# note

> 本DEMO暂针对中文使用。
