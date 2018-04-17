# Deepdive 实战-从下载到跑路

[TOC]

本文介绍Deepdive的安装和配置，然后第二部分介绍了一个简单实例，在这个实例中，只要按照步骤一步步坐下来就可以实现，然后对实例进行详细的介绍，然后介绍如何对实例中的模块进行修改。其中主要有第三方分词工具`Jieba`和`Stanford CoreNLP`句法分析的结合。

# 第零部分 下载安装和配置

**本文的数据和工具都来自于http://www.openkg.cn/dataset/cn-deepdive**

## 1 下载

首先在上面的页面下载浙大汉化的支持中文的Deepdive，度盘资源。因为我们后面的数据也是中文的，所以需要这个版本。当然，学习到了后面章节的时候，我们学习了如何替换其中的各个模块后，就可以自己定义语言了。

## 2 系统要求

在写这篇文章的时候，`Ubuntu 17.04`下的支持并不好，虽然作者修改安装代码最终也装上了，但是不建议新手来处理。所以推荐`Ubuntu 16.04`的版本。

## 3 安装

我们下载到的Deepdive是一个`.zip`格式的压缩包，所以我们先要解压缩，图形界面下：*右键->提取到此处（Extract Here）* ；或者命令行下`cd`到`Deepdive.zip`文件的目录下，输入命令：

```shell
unzip Deepdive.zip
```

就可以得到解压后的文件夹，其目录结构如下：（其中带有绿色底纹的为目录）

![yglVy.png](https://s1.ax2x.com/2018/02/17/yglVy.png)

我们现在需要先修改一下**install.sh**文件，把其第193行的**xzvf**改为**xvf**后，在*CNdeepdive*目录下执行

```shell
./install.sh
```

![ygSll.png](https://s1.ax2x.com/2018/02/17/ygSll.png)

**此过程需要联网**

按照提示我们输入1,然后敲击回车键，deepdive就会开始安装，安装完成后，我们再安装postgres，输入6然后敲击回车键。正常是不会报错的，如果报错，请自行解决:laughing:

然后执行下面的命令，安装自然语言处理的部分，很快就可以结束：

```
./nlp_setup.sh
```

## 4 配置

上面的安装过程完成后，会在你的主目录中创建一个*local*文件夹，我们要把这个其中的*bin* 目录添加到环境变量`Path`中。作为文本文件打开`~/.bashrc`，这个文件是隐藏的，可以在命令行中使用**vi**，或者**gedit**等各种你会用的文本编辑程序；或者在图形打开主目录后，按快捷键`Ctrl+H`显示隐藏文件后再编辑。在文件的末尾添加下面的内容：

```shell
export PATH="/home/(username)/local/bin:$PATH"
# 其中的(username)要替换成你当前登录的用户名
```

修改完成后保存并关闭文件，在终端中输入：

```shell
source ~/.bashrc
```

来使我们的修改生效。

好的，到这里Deepdive就已经安装完成了，打开终端执行`deepdive`我们可以看到Deepdive的版本信息和命令参数的Help。如下图：（后面还有很长）

![ywP6O.png](https://s1.ax2x.com/2018/02/17/ywP6O.png)

# 第一部分 简单实例

这部分的数据和代码以及程序都在前面我们解压后的*CNdeepdive/transaction*中，你可以把这个目录复制到你喜欢的地方（比如*~/Projects/transaction* 这个位置是我一般用的，你可以自己定），然后再新建一个目录，可以叫做*transaction2* 我们后面的所有工作都是在这个**transaction2**目录中进行的，后面简称它为项目目录。

## 0 基础工作

我们的DeepDive的数据（包括输入，输出，中间数据）全都存在关系数据库中，强烈推荐Postgres（==说明其他类型的数据库也是支持的==，这个我们之前已经安装过了）我们配置db.url文件来设置本地数据库。在*transaction2* 目录中打开终端，执行下面命令：

```shell
echo "postgresql://localhost/deepdive_$USER" >db.url
#$符号是shell中用来表示变量的，所以这个蓝色部分大概就是 用户名@主机名：。。。。
#deepdive_spouse_$USER 我们这个项目的数据库名，也可以自定义。
```

## 1 数据准备

数据处理要有如下几个部分：

- 载入原始输入数据
- 添加自然语言处理（NLP)标记
- 抽取候选关系
- 为每个候选关系抽取特征

### 1.1 载入原始数据

我们控制Deepdive操作的方法，就是在我们的项目目录中新建`app.ddlog`文件，把操作写入到这个文件中，语法后面会讲到。

我们需要在把articles文件放到input文件夹里面去，按照deepdive官方的Tutorial的方法，无法实现不知道为什么。（这个文件在transaction/input中）

然后在`app.ddlog` 中添加如下内容：

```
aritcles(
	id		text,
	content	text
).
```

其实就是为了标记我们的articles文档中的列名，然后存到数据库中。

==每次app.ddlog变动== 后我们都要进行编译操作，也就是在项目目录执行下面的命令：

```shell
deepdive compile
```

他会执行当前目录下的工作的编译。

编译完成后，我们要执行

```shell
deepdive do articles
```

这里`do` 后面应该和我们`app.ddlog` 中的名字相同，和`input` 文件夹中的文件名也相同，他们三个应该都是一致的。他会把input文件中的对应的文件导入postgresql数据库中。注意这个语句执行后，他会进入一个类似vi的界面让你审查他自动生成的处理代码是不是正确，这时候输入`：wq` 就可以保存并退出，继续执行后面的步骤。

执行完成后，可以执行下面代码在数据库中查询，看看是不是成功导入了。

```shell
deepdive query '?- articles(id, _).'
```

如果成功导入，那么他就会有几行数据，如果不成功，那么就会显示`0 rows` ，成功会显示下面这样

```
id     
------------
1201835868
1201835869
1201835883
1201835885
1201835927
1201845343
1201835928
1201835930
1201835934
1201841180
:
```

### 1.2 添加自然语言标记

deepdive默认用standford nlp进行文本处理，可以返回句子的分词、lemma、pos、NER

> 自然语言处理 Natural Language Processing
>
> 分词：首先是中文分词，在一句话中，我们要把词分出来，而不是光看单独的子。比如 ==我== ==今天== ==很== ==高兴== 选择合适的字组成合适的词来构成句子
>
> lemma：词元，这个是指这个词实质上的含义，比如cat,cats他们有相同词元。
>
> pos：词性标注，最基本的是动词、名词等等
>
> NER：Named Entity Recognition，可以识别出地名、人名、组织等等
>
> 具体内容请自行学习自然语言处理。。。

现在我们有了文章了，所以下一步就是把文章拆分成更细致的东西sentences，所以，我们在数据库中需要有一个表来存储sentences的数据。同articles，我们要在`app.ddlog` 中新建一个表的定义：

```
sentences(
	doc_id			text,
	sentence_index 	int,
	sentence_text	text,
	tokens			text[],
	lemmas 			text[],
	pos_tags		text[],
	ner_tags		text[],
	doc_offsets		int[],
	dep_types		text[],
	dep_tokens		int[]
).
```

光有存储用的表的定义还不够，对吧，我们还需要对数据处理的方法，deepdive只是个框架，具体要怎么处理需要我们告诉他。所以我们要定义函数来处理articles让他变成sentences。

```
function nlp_markup over (
	doc_id text,
	content text
) returns rows like sentences
implementation "udf/nlp_markup.sh" handles tsv lines.
```

> 这里的语法我们记住这样用就可以了
>
> function用来定义函数，后面`nlp_markup` 是函数名 over后面接的是参数表。
>
> returns 说明了函数返回的形式，返回就像我们前面定义的sentences那样的一行。
>
> 最后一句说明了我们这个程序文件是`udf/nlp_markup.sh` ,输入是tsv的一行。

当然，光声明了函数是不行的，我们还要调用才会起作用对不啦，所以下面就是调用函数啦。

```
sentences+=nlp_markup(doc_id,content) :-
articles(doc_id,content).
```

> 上面的`+=` 其实和其他语言差不多，就是对于来源是articles中的每一行的`doc_id` 和`content` 我们都调用`nlp_markup` 然后结果添加到sentences表中。

```shell
#!/usr/bin/env bash
# A shell script that runs Bazaar/Parser over documents passed as input TSV lines
#
# $ deepdive env udf/nlp_markup.sh doc_id _ _ content _
##
set -euo pipefail
cd "$(dirname "$0")"

: ${BAZAAR_HOME:=$PWD/bazaar}
[[ -x "$BAZAAR_HOME"/parser/target/start ]] || {
    echo "No Bazaar/Parser set up at: $BAZAAR_HOME/parser"
    exit 2
} >&2

[[ $# -gt 0 ]] ||
    # default column order of input TSV
    set -- doc_id content

# convert input tsv lines into JSON lines for Bazaar/Parser


# start Bazaar/Parser to emit sentences TSV
tsv2json "$@" |
"$BAZAAR_HOME"/parser/run.sh -i json -k doc_id -v content
```

> 所有#开头的（除了#！）都是普通注释
>
> 参数的用法(
>
> - \$0：调用文件使用的文件名，带有前面的路径,
> - \$1-∞：传给脚本的各个参数,
> - \$@,\$*：这两个都表示传入的所有参数,
> - \$#：表示传入参数的个数)
>
> 第一行指定了脚本的执行程序
>
> 第六行指定了一些程序的错误处理方式等（详见Shell相关文档）
>
> 第七行改变当前目录到`nlp_markup.py` 所在目录，也就是`udf` 目录
>
> 第九行设置了一个变量`BAZZER_HOME` 他的值是`bazaar` 的路径
>
> 第10-13行执行`/parser/target/start` 文件，如果有错会不正常退出，并提示
>
> 第15-17行检查输入参数的正确性，看参数个数是不是大于0个，如果没有参数，自己设定参数名
>
> 第23-24行把全部输入的参数用`tsv2json` 工具转换成`json` 格式，然后在执行`parser/run.sh` 并以刚才的`json` 作为参数输入。

现在我们应该知道，想对文本进行分词、词性标注等等自然语言处理，仅靠我们这点代码是不够的，所以我们有了`bazaar` 文件夹下的程序，对于这个文件我们不过多了解，但是现在我们要知道的是，如果我们想使用它，我们需要对它进行编译。来到`bazaar/parser` 目录下，然后在终端中执行下面代码

```shell
sbt/sbt stage
```

现在，这一步的准备工作完成了，可以回到我们的`transaction` 目录下去编译`app.ddlog` 并执行相应工作

```shell
deepdive compile
deepdive do transaction
```

这一步需要耐心等待，可能要很久很久很久。

（如果只做调试测试只用，那么把input文件夹中的articles删掉一些，可以只留2-3条，作者i3电脑可以在15分钟左右完成，然后在终端执行：`deepdive redo articles` 和`deepdive redo sentences`，这两个命令可以重新执行以前执行过的命令，删除以前的数据）

==请注意==上面这一步，如果报错，就是找不到进程啊，没有文件啊类似的错误可以使用下面的办法处理

```shell
#在执行前面的两条命令之前，进入superuser的模式，使用su进行一次登录
#具体步骤：按照下面的代码来
su (username)  #这里的username是你的用户名，就是终端前面‘@’这个符号前面的部分，不带括号
#它会提示你输入密码，登录即可。
#接下来在执行前面的
deepdive compile
deepdive do sentences
```

执行完成之后，我们可以使用下面的命令，查询处理结果：

```
deepdive query '
doc_id, index, tokens, ner_tags | 5
?- sentences(doc_id, index, text, tokens, lemmas, pos_tags, ner_tags, _, _, _).
'    
```



### 1.3 抽取候选关系

到这里，前面都是==通用的步骤==不论我们抽取什么样的关系，什么类型的实体，我们肯定都要先对我们的文章进行处理，分词啊，标记啊啥的。但是到了这一步，我们就是要按照我们的任务去安排了。

本文档是按照前面下载的*deepdive*中的**transaction**样例来演示的，所以现在我们的任务是抽取不同公司之间的交易关系。如果你要抽取和我这里不同的东西，just对号入座、按图索骥就可以了。

既然我们要抽取公司间的交易信息，所以我们是不是应该先得到我文本中的公司是谁，才能进一步知道他们有没有关系，这一步就是要抽取这些公司啦。一共分两步

- 抽取候选实体
- 得到实体间的候选关系

#### 1.3.1 抽取候选实体

和前面的*sentence*一样，我们需要建立一个数据表，来存储我们接下来会的到公司（不然放在哪）,还有用来抽取实体的方法，并且要调用这个方法。所以我们要在`app.ddlog`中添加下面的内容，告诉*Deepdive*如何处理数据

```scheme
company_mention(
	mention_id		text,
	mention_text	text,
	doc_id			text,
	sentence_index	int,
	begin_index		int,
	end_index		int
).

function map_company_mention over(
	doc_id			text,
	sentence_index	 int,
	tokens			text[],
	ner_tags		text[]
)returns rows like company_mention
implementation "udf/map_company_mention.py" handles tsv lines.

company_mention += map_company_mention(
	doc_id, sentence_index, tokens, ner_tags
) :-
sentences(doc_id, sentence_index, _, tokens, _, _,ner_tags, _, _, _).
```

现在我们应该想了，处理方式我选好了，那么这个`udf/map_company_mention.py`应该怎么写才行啊，我要如何从这一堆文本里得到公司的名字。好！前面**1.2** 已经讲过了，我们*nlp*处理中有一个步骤是命名实体识别(NER)，这个东西会把每个词的实体识别出来，比如公司名字就应是属于`ORG`类的实体。所以我们只要在每个*sentence*中找到其中的`ner_tags` 为连续的`ORG`标记的就可以了。

好我们来看看`map_company_mention`中的代码

```python
#!/usr/bin/env python
#encoding:utf-8
from deepdive import *
from transform import *
import re

@tsv_extractor
@returns(lambda
        mention_id       = "text",
        mention_text     = "text",
        doc_id           = "text",
        sentence_index   = "int",
        begin_index      = "int",
        end_index        = "int",
    :[])
def extract(
        doc_id         = "text",
        sentence_index = "int",
        tokens         = "text[]",
        ner_tags       = "text[]",
    ):
    """
    Finds phrases that are continuous words tagged with ORG.
    """
    num_tokens = len(ner_tags)
    # find all first indexes of series of tokens tagged as ORG
    first_indexes = (i for i in xrange(num_tokens) if ner_tags[i] == "ORG" and (i == 0 or ner_tags[i-1] != "ORG") and re.match(u'^[\u4e00-\u9fa5\u3040-\u309f\u30a0-\u30ffa-zA-Z]+$', unicode(tokens[i], "utf-8")) != None)
    for begin_index in first_indexes:
        # find the end of the ORG phrase (consecutive tokens tagged as ORG)
        end_index = begin_index + 1
        while end_index < num_tokens and ner_tags[end_index] == "ORG" and re.match(u'^[\u4e00-\u9fa5\u3040-\u309f\u30a0-\u30ffa-zA-Z]+$', unicode(tokens[end_index], "utf-8")) != None:
            end_index += 1
        end_index -= 1
        # generate a mention identifier
        mention_id = "%s_%d_%d_%d" % (doc_id, sentence_index, begin_index, end_index)
        mention_text = "".join(map(lambda i: tokens[i], xrange(begin_index, end_index + 1)))
        temp_text = link(mention_text, entity_dict)
        if temp_text == None or temp_text == '':
            continue
        if end_index - begin_index >= 25:
            continue
        # Output a tuple for each ORG phrase
        yield [
            mention_id,
            mention_text,
            doc_id,
            sentence_index,
            begin_index,
            end_index,
        ]
```

> 程序分析：
>
> - 不常用的知识
>   - python中以函数定义前，@标识符表示的是python中**装饰器**的意思
>
>   - lambda关键字，用来定义一个匿名函数,可以参考这里：[Python中的Lambda](http://blog.csdn.net/cx943024256/article/details/79569853)
>
>   - x for x in range(100) if x%2 == 0 类似这样的用法，叫做**列表解析**
>
>   - re.match()是用来匹配**正则表达式**
>
>   - yield：用来创建**生成器**，具体点[Python之yield](http://blog.csdn.net/cx943024256/article/details/79005309)
>
> - 逻辑分析：
>   - 这里，先在句子中找到连续的命名实体标记ORG的连续的词。他们其实就是公司名，但可能有的ORG不是公司名，所以在第31-41行进行了简单的过滤。然后将每个公司名进行一次返回。

#### 1.3.2 抽取候选关系

​	候选实体已经有了，就是文中出现的公司名，我们要找的是公司之间的交易关系。所以这里候选关系简单来说，就是把不同的公司名两两组合，最终得到的关系表其实就相当于对两个候选实体表进行笛卡尔积（当然，我们还需要一些简单的过滤的处理，比如两个公司名不能相同啊等等）

> 笛卡尔乘积是指在数学中，两个[集合](https://baike.baidu.com/item/%E9%9B%86%E5%90%88)*X*和*Y*的笛卡尓积（Cartesian product），又称[直积](https://baike.baidu.com/item/%E7%9B%B4%E7%A7%AF)，表示为*X*×*Y*，第一个对象是*X*的成员而第二个对象是*Y*的所有可能[有序对](https://baike.baidu.com/item/%E6%9C%89%E5%BA%8F%E5%AF%B9)的其中一个成员
>
> 假设有
>
> ​	A：a1,a2,a3		B:b1,b2
>
> ​	A×B：（a1,b1),(a1,b2),(a2,b1),(a2,b2),(a3,b1),(a3,b2)

​	好了，现在定义一个表来存储我们的候选关系吧

```
transaction_candidate(
	p1_id		text,
	p1_name 	text,
	p2_id		text,
	p2_name 	text
)
```

​	这个候选关系同样是要经过我们函数处理的，所以输入的参数和表的Schema一样，这样定义一个函数：

```
function map_transaction_candidate over (
	p1_id		text,
	p1_name		text,
	p2_id		text,
	p2_name		text
) returns rows like transaction_candidate
implementation "udf/map_transaction_candidate.py" handles tsv lines.
```

这里的函数要怎么调用呢，首先我们的输入数据应该是候选实体，又因为**deepdive**处理笛卡尔积这种关系还是很简单的，直接使用数据库表的操作就可以。

```
transaction_candidate += map_transaction_candidate(p1, p1_name, p2, p2_name) :-
company_mention(p1, p1_name, same_doc, same_sentence, p1_begin, _),
company_mention(p2, p2_name, same_doc, same_sentence, p2_begin, _),
p1_name != p2_name,
p1_begin != p2_begin.
```

> 上面的处理，要求相关联的两个候选实体，要是在同一句话里的，不再同一句话中怎么知道他们两个有关系。而且两个名字不能一样，识别的时候也有可能一个部分识别了两个，所以开始位置也不能一样。

那现在来看看`map_transaction_candidate.py`的内容。

```python
#!/usr/bin/env python
#encoding:utf-8
from deepdive import *
import re

@tsv_extractor
@returns(lambda
        p1_id       = "text",
        p1_name     = "text",
        p2_id           = "text",
        p2_name  = "text",
    :[])
def extract(
        p1_id       = "text",
        p1_name     = "text",
        p2_id           = "text",
        p2_name   = "text",
    ):
    if not(set(p1_name) <= set(p2_name) or set(p2_name) <= set(p1_name)):
        yield [
            p1_id,
            p1_name,
            p2_id,
            p2_name,
        ]
```

> 这里多了的内容是set对象，他是集合的意思
>
> 这里把字符串给进去，得到的集合是字符串中出现的字母、汉字、符号的组合（还被去重了，就是每个东西只有一个）
>
> 比如set('hello') 转换为字符串就是{'h','e','l','o'} 只有一个'l'哦
>
> 而他们之间的**a <= b**操作符用来判断，是不是**a**中的所有元素都在**b**中出现。
>
> 这里就是过滤掉，可能是同一个公司名字，但是有简略写法的可能。

这一切都尘埃落定，编译并执行这个操作吧

```shell
deepdive compile && deepdive do transaction_candidate
```



### 1.4 特征提取

​	对于前面提取出来的公司间的候选关系，我们肯定是要使用机器学习的算法，通过一些我们找的训练集，让计算机去分类，哪个关系可能有交易关系（哇塞，还有机器学习 :laughing:。。不然呢，人工分类吗，那直接让人看文章不是更方便 :joy:)，所以特征首先要得到。

​	对于自然语言来说，他的特征就是**上下文**，就是**上下文**，一般重要的事说两遍。

​	所以定义一个特征表：

```
transaction_feature(
	p1_id		text,
	p2_id		text,
	feature		text
).
```

​	这个特征明显也是抽取出来的，所以要定义一个函数来抽取它。

```
function extract_transaction_features over (
	p1_id			text,
	p2_id			text,
	p1_begin_index	int,
	p1_end_index	int,
	p2_begin_index 	int,
	p2_end_index	int,
	doc_id			text,
	sent_index		int,
	tokens			text[],
	lemmas			text[],
	pos_tags		text[],
	ner_tags		text[],
	dep_types		text[],
	dep_tokens		int[]
) returns rows like transaction_feature
implementation "udf/extract_transaction_features.py" handles tsv lines.
```

然后定义如何调用这个函数，输入对应的参数：

```
transaction_feature += extract_transaction_features(
p1_id, p2_id, p1_begin_index, p1_end_index, p2_begin_index, p2_end_index,
doc_id, sent_index, tokens, lemmas, pos_tags, ner_tags, dep_types, dep_tokens
) :-
company_mention(p1_id, _, doc_id, sent_index, p1_begin_index, p1_end_index),
company_mention(p2_id, _, doc_id, sent_index, p2_begin_index, p2_end_index),
sentences(doc_id, sent_index, _, tokens, lemmas, pos_tags, ner_tags, _, dep_types, dep_tokens).
```

老规矩，现在看看这个py脚本里的内容：

```python
#!/usr/bin/env python
from deepdive import *
import ddlib

@tsv_extractor
@returns(lambda
        p1_id   = "text",
        p2_id   = "text",
        feature = "text",
    :[])
def extract(
        p1_id          = "text",
        p2_id          = "text",
        p1_begin_index = "int",
        p1_end_index   = "int",
        p2_begin_index = "int",
        p2_end_index   = "int",
        doc_id         = "text",
        sent_index     = "int",
        tokens         = "text[]",
        lemmas         = "text[]",
        pos_tags       = "text[]",
        ner_tags       = "text[]",
        dep_types      = "text[]",
        dep_parents    = "int[]",
    ):
    """
    Uses DDLIB to generate features for the spouse relation.
    """
    # Create a DDLIB sentence object, which is just a list of DDLIB Word objects
    sent = []
    for i,t in enumerate(tokens):
        sent.append(ddlib.Word(
            begin_char_offset=None,
            end_char_offset=None,
            word=t,
            lemma=lemmas[i],
            pos=pos_tags[i],
            ner=ner_tags[i],
            dep_par=dep_parents[i] - 1,  # Note that as stored from CoreNLP 0 is ROOT, but for DDLIB -1 is ROOT
            dep_label=dep_types[i]))

    # Create DDLIB Spans for the two person mentions
    p1_span = ddlib.Span(begin_word_id=p1_begin_index, length=(p1_end_index-p1_begin_index+1))
    p2_span = ddlib.Span(begin_word_id=p2_begin_index, length=(p2_end_index-p2_begin_index+1))

    # Generate the generic features using DDLIB
    for feature in ddlib.get_generic_features_relation(sent, p1_span, p2_span):
        yield [p1_id, p2_id, feature]
```

> *enumerate(iter)*在迭代中的作用是返回集合*iter*的索引序号和其value的组合
>
> 这里调用了`ddlib.Word()`、`ddlib.Span()`、`ddlib.get_generic_features_relation()`东西，下面一个个看
>
> `ddlib.Word()`:其实这个是*Python*使用`collections.namedtuple`(**需要先导入collections**) 定义的带有名字的格式化的元组，类似于C语言中的`struct`(结构体)，其中一个有八个*item*，就和代码里写的一样。
>
> `ddlib.Span()`:这个和*ddlib.Word()*类型相似，都是类似结构体的东西，它内部存储的*item*也在代码里写了。
>
> **注意**：`Word`、`Span`这两个都不是方法，只是定义了数据的存储结构而已。
>
> `ddlib.get_generic_features_relation()`：这个就是普通函数了，它返回（用*yield*的方法）给定的句子中关于*p1_span*和*p2_span*的标准的特征（就是最普通的），注意这个函数参数的类型：*sent*是Word类型的，*p?_span*要求是Span类型的。其实这个函数还有最后一个参数，可以设置提取的特征的最大长度，默认是5，所以结果最长是5个特征的。
>
> 以上三个被调用的东西，都是Deepdive定义的比较简单的东西，供用户直接调用。
>
> 具体参见`~/local/lib/python/ddlib/`中的源代码文件。

现在编译并执行

```shell
deepdive compile && deepdive do transaction_feature
```

可以执行下面的语句来查看结果

```shell
deepdive query '| 20 ?- transaction_feature(_, _, feature).'
```

它显示的内容就是一个特征的内容。

比如下面：

```python
feature                        
————————————————————————————
 WORD_SEQ_[郴州市 城市 建设 投资 发展 集团 有限 公司]
 LEMMA_SEQ_[郴州市 城市 建设 投资 发展 集团 有限 公司]
 NER_SEQ_[ORG ORG ORG ORG ORG ORG ORG ORG]
 POS_SEQ_[NR NN NN NN NN NN JJ NN]
 W_LEMMA_L_1_R_1_[为]_[提供]
 W_NER_L_1_R_1_[O]_[O]
 W_LEMMA_L_1_R_2_[为]_[提供 担保]
 W_NER_L_1_R_2_[O]_[O O]
 W_LEMMA_L_1_R_3_[为]_[提供 担保 公告]
 W_NER_L_1_R_3_[O]_[O O O]
 W_LEMMA_L_2_R_1_[公司 为]_[提供]
 W_NER_L_2_R_1_[ORG O]_[O]
 W_LEMMA_L_2_R_2_[公司 为]_[提供 担保]
 W_NER_L_2_R_2_[ORG O]_[O O]
##这个是我的注释，原先没有的哦
##所以你看，下面最长的就是左2右3，或者左3右2的格式，最长是五个。
 W_LEMMA_L_2_R_3_[公司 为]_[提供 担保 公告]
 W_NER_L_2_R_3_[ORG O]_[O O O]
 W_LEMMA_L_3_R_1_[有限 公司 为]_[提供]
 W_NER_L_3_R_1_[ORG ORG O]_[O]
 W_LEMMA_L_3_R_2_[有限 公司 为]_[提供 担保]
 W_NER_L_3_R_2_[ORG ORG O]_[O O]

(20 rows)
```



### 1.5 样本打标

​	对于监督学习，必然需要标注数据，那么已标注数据是怎么来的呢？当然正经的来说，应该是我们给这个系统提供大量的我们之前已经标注好了的数据，但是现在我们没有。所以我们可以对前面几步我们抽取出来的关系，利用一些先验的数据（比如人工标记的关系，还有先验的规则）对那些关系进行标记（标注出某些标记是已知的存在交易关系的，还有已知不存在交易关系的*候选关系*）

​	所以对于这里来说，我们同样需要数据库中有一个表，来存储我们的被标记数据。在**app.ddlog**中添加

```
transaction_label(
    @key
    @references(relation="has_transaction", column="p1_id", alias="has_transaction")
    p1_id   text,
    @key
    @references(relation="has_transaction", column="p2_id", alias="has_transaction")
    p2_id   text,
    @navigable
    label   int,
    @navigable
    rule_id text
).
```

​	其中的**label**用来表示当前关系的相关性，数字越大就越相关，正数为正相关，负数为负相关。前面解释了，这个被标记数据，是我们从已经抽取出来的候选关系中进行打标得到的，对吧，在我们打标操作之前，其实我们要把我们的候选关系放到这里，然后对它们打标（因为提前也不知道谁会被标记正负）。所以这里通过数据库的操作，把我们的候选关系的公司id直接拷贝过来（在**app.ddlog**中添加）

```
transcation_label(p1,p2,0,NULL) :- transaction_candidate(p1,_,p2,_).
```

​	好的，那么现在应该导入我们的先验数据了，这个其实就是我们知道的有交易关系的实体对，也就是两个公司的名字，同样用一个表来存储（在**app.ddlog**中添加），数据文件放到`input/`下面

```
transaction_dbdata(
    @key
    company1_name text,
    @key
    company2_name text
).
```

​	我们应该先导入先验数据才能够进行打标，对吧，现在表已经定义好了，我们还要进行真正的导入操作。所以**终端执行**下面命令：

```shell
deepdive compile && deepdive do transaction_dbdata
```

​	下面来定义我们的打标规则，如果候选的待打标的关系中的两个公司名，和我已知的两个公司名字相同，他们就应该是有关系的，应该被标上：*那是相当有关系啊！* 。所以下面进行打标的操作（在**app.ddlog**中添加）：

```
transaction_label(p1,p2, 3, "from_dbdata") :-
    transaction_candidate(p1, p1_name, p2, p2_name), transaction_dbdata(n1, n2),
    [ lower(n1) = lower(p1_name), lower(n2) = lower(p2_name) ;
      lower(n2) = lower(p1_name), lower(n1) = lower(p2_name) ].
```

​	上面的数据库操作就是对于公司名字和先验数据中的公司名字相同的关系，被标记为相当有关系（给的label是3，这个数越大就说明关系就越大，或者说越有可能有关系），标记规则就写来自数据“from_dbdata"就是这样。后面的部分主要应用在英文文本，使结果大小写无关（在**app.ddlog**中添加）。

```
function supervise over (
    p1_id text, p1_begin int, p1_end int,
    p2_id text, p2_begin int, p2_end int,
    doc_id         text,
    sentence_index int,
    sentence_text  text,
    tokens         text[],
    lemmas         text[],
    pos_tags       text[],
    ner_tags       text[],
    dep_types      text[],
    dep_tokens     int[]
) returns (
    p1_id text, p2_id text, label int, rule_id text
)
implementation "udf/supervise_transaction.py" handles tsv lines.
```

​	下面调用函数（在**app.ddlog**中添加）：

```
transaction_label += supervise(
p1_id, p1_begin, p1_end,
p2_id, p2_begin, p2_end,
doc_id, sentence_index, sentence_text,
tokens, lemmas, pos_tags, ner_tags, dep_types, dep_token_indexes
) :-
transaction_candidate(p1_id, _, p2_id, _),
company_mention(p1_id, p1_text, doc_id, sentence_index, p1_begin, p1_end),
company_mention(p2_id, p2_text, _, _, p2_begin, p2_end),
sentences(
    doc_id, sentence_index, sentence_text,
    tokens, lemmas, pos_tags, ner_tags, _, dep_types, dep_token_indexes
).
```

老规矩，看看这个`udf/supervise_transaction.py`

```python
#!/usr/bin/env python
#encoding:utf-8
from deepdive import *
import random
from collections import namedtuple

TransactionLabel = namedtuple('TransactionLabel', 'p1_id, p2_id, label, type')

@tsv_extractor
@returns(lambda
        p1_id   = "text",
        p2_id   = "text",
        label   = "int",
        rule_id = "text",
    :[])
# heuristic rules for finding positive/negative examples of transaction relationship mentions
def supervise(
        p1_id="text", p1_begin="int", p1_end="int",
        p2_id="text", p2_begin="int", p2_end="int",
        doc_id="text", sentence_index="int", sentence_text="text",
        tokens="text[]", lemmas="text[]", pos_tags="text[]", ner_tags="text[]",
        dep_types="text[]", dep_token_indexes="int[]",
    ):

    # Constants
    TRANSLATION = frozenset(["转让", "交易", "卖出", "购买","收购","购入","拥有", "持有", "卖给", "买入", "获得"])
    STOCK = frozenset(["股权", "股份", "股"])
    TRANSLATION_COM = frozenset(["持股","买股","卖股"])
    TRANSLATION_AFTER = frozenset(["融资", "投资"])
    OTHERS_AFTER = frozenset(["产品", "委托", "贷款", "保险"])
    OTHERS_COM = frozenset(["购买", "提供", "申请", "销售"])
    CONF = frozenset(["对", "向"])
    OTHERS = frozenset(["销售产品", "提供担保","提供服务"])


    COMMAS = frozenset([":", "：","1","2","3","4","5","6","7","8","9","0","、", ";", "；"])
    MAX_DIST = 40

    # Common data objects
    p1_end_idx = min(p1_end, p2_end)
    p2_start_idx = max(p1_begin, p2_begin)
    p2_end_idx = max(p1_end,p2_end)
    intermediate_lemmas = lemmas[p1_end_idx+1:p2_start_idx]
    intermediate_ner_tags = ner_tags[p1_end_idx+1:p2_start_idx]
    tail_lemmas = lemmas[p2_end_idx+1:]
    transaction = TransactionLabel(p1_id=p1_id, p2_id=p2_id, label=None, type=None)

    # Rule: Candidates that are too far apart
    if len(intermediate_lemmas) > MAX_DIST:
        yield transaction._replace(label=-1, type='neg:far_apart')

    # Rule: Candidates that have a third company in between
    if 'ORG' in intermediate_ner_tags:
        yield transaction._replace(label=-1, type='neg:third_company_between')

    # Rule: Sentences that contain wife/husband in between
    
    if len(TRANSLATION_COM.intersection(intermediate_lemmas)) > 0:
        yield transaction._replace(label=1, type='pos:A持股B')

    if len(TRANSLATION.intersection(intermediate_lemmas)) > 0 and len(STOCK.intersection(tail_lemmas)) > 0:
        yield transaction._replace(label=1, type='pos:A购买B股权')
    
    if len(TRANSLATION_AFTER.intersection(intermediate_lemmas)) > 0:
        yield transaction._replace(label=1, type='pos:A增资B')

    if len(OTHERS_COM.intersection(intermediate_lemmas)) > 0 and len(OTHERS_AFTER.intersection(tail_lemmas)) > 0:
        yield transaction._replace(label=-1, type='neg:A购买B的产品')

    if len(CONF.intersection(intermediate_lemmas)) > 0 and len(OTHERS.intersection(tail_lemmas)) > 0:
        yield transaction._replace(label=-1, type='neg:A向B提供担保')

    # Rule: Sentences that contain and ... married
    #         (<P1>)(and)?(<P2>)([ A-Za-z]+)(married)
    if ("与" in intermediate_lemmas) and ("签署股权转让协议" in tail_lemmas):
        yield transaction._replace(label=1, type='pos:A和B签署股权转让协议')
        
    if len(COMMAS.intersection(intermediate_lemmas)) > 0:
        yield transaction._replace(label=-1, type='neg:中间有特殊符号')


```

> 上面我对源文件删掉了一些无效的注释
>
> 但是就这样其实可以看出来，这个代码中的程序就是一堆`if..else..`的结构，进行普通的规则的判断，然后给出标记和所使用的规则。整体的处理都是对于上下文（或者说**特征**的判断）
>
> 没有什么特殊的地方了，每个领域都有领域自身的一些能体现在文本上的规则。

因为上面的打标过程，可能多个规则都对同一个关系打标了，但是他们打标的值不同，所以这里要把这种同一个关系的都整合起来，让他们的**label**相加，得到最终的标记数据（在**app.ddlog**中添加）：

```
 transaction_label_resolved(p1_id, p2_id, SUM(vote)) :-
 transaction_label(p1_id, p2_id, vote, rule_id).
```

现在，打标的一系列操作我们都定义好了，接下来就执行打标吧，刺激不？在终端执行下面代码：

```shell
deepdive compile && deepdive do transaction_label_resolved
```

这里如果执行报错找不到文件的话，将`udf/tranform.py`中的变量`ENTITY_NAME`中存储的文件名改为绝对路径即可。


## 2 模型构建

通过前面的步骤，我们已经将需要用到的数据都准备好了，下面构建我们的模型。

### 2.1 模型构建

(1). 定义最终存储的表格，表后面的问好`?`表示此表是用户模式下的变量表，即需要推导关系的表。这里我们预测的是公司间是否存在交易关系。

    @extraction
    has_transaction?(
        p1_id text,
        p2_id text
    ).

(2). 根据**1.5**中的打标结果，把已经标记了的数据，输入到`has_transaction`表中。

    has_transaction(p1_id, p2_id) = if l > 0 then TRUE
                      else if l < 0 then FALSE
                      else NULL end :- transaction_label_resolved(p1_id, p2_id, l).

此时变量表中的部分变量label已知，成为了先验变量。                  

(3). 最后编译执行决策表：

    $ deepdive compile && deepdive do has_transaction
​    

### 2.2 因子图构建


(1). 指定特征

将每一对has_transaction中的实体对和特征表连接起来，通过特征factor的连接，全局学习这些特征的权重。在app.ddlog中定义：

    @weight(f)
    has_transaction(p1_id, p2_id) :-
        transaction_candidate(p1_id, _, p2_id, _),
        transaction_feature(p1_id, p2_id, f).

(2). 指定变量间的依赖性

我们可以指定两张变量表间遵守的规则，并给这个规则以权重。比如c1和c2有交易，可以推出c2和c1也有交易。这是一条可以确保的定理，因此给予较高权重：

     @weight(3.0)
     has_transaction(p1_id, p2_id) => has_transaction(p2_id, p1_id) :-
        transaction_candidate(p1_id, _, p2_id, _).


变量表间的依赖性使得deepdive很好地支持了多关系下的抽取。

(3). 最后，编译，并生成最终的概率模型：

    $ deepdive compile && deepdive do probabilities

查看我们预测的公司间交易关系概率：

    $ deepdive sql "SELECT p1_id, p2_id, expectation FROM has_transaction_label_inference ORDER BY random() LIMIT 20"
​    

------

| p1\_id                    | p2\_id                   | expectation |
| ------------------------- | ------------------------ | ----------- |
| 1201778739\_118\_170\_171 | 1201778739\_118\_54\_60  | 0           |
| 1201778739\_54\_30\_35    | 1201778739\_54\_8\_11    | 0.035       |
| 1201759193\_1\_26\_31     | 1201759193\_1\_43\_48    | 0.07        |
| 1201766319\_65\_331\_331  | 1201766319\_65\_159\_163 | 0           |
| 1201761624\_17\_30\_35    | 1201761624\_17\_9\_14    | 0.188       |
| 1201743500\_3\_0\_5       | 1201743500\_3\_8\_14     | 0.347       |
| 1201789764\_3\_16\_21     | 1201789764\_3\_75\_76    | 0           |
| 1201778739\_120\_26\_27   | 1201778739\_120\_29\_30  | 0.003       |
| 1201752964\_3\_21\_21     | 1201752964\_3\_5\_10     | 0.133       |
| 1201775403\_1\_83\_88     | 1201775403\_1\_3\_6      | 0           |
| 1201778793\_15\_5\_6      | 1201778793\_15\_17\_18   | 0.984       |
| 1201773262\_2\_85\_88     | 1201773262\_2\_99\_99    | 0.043       |
| 1201734457\_24\_19\_20    | 1201734457\_24\_28\_29   | 0.081       |
| 1201752964\_22\_48\_50    | 1201752964\_22\_9\_10    | 0.013       |
| 1201759216\_5\_38\_44     | 1201759216\_5\_55\_56    | 0.305       |
| 1201755097\_4\_18\_22     | 1201755097\_4\_52\_57    | 1           |
| 1201750746\_2\_0\_5       | 1201750746\_2\_20\_26    | 0.034       |
| 1201759186\_4\_45\_46     | 1201759186\_4\_41\_43    | 0.005       |
| 1201734457\_18\_7\_11     | 1201734457\_18\_13\_18   | 0.964       |
| 1201759263\_36\_18\_20    | 1201759263\_36\_33\_36   | 0.002       |


至此，我们的交易关系抽取就基本完成了。更多详细说明请见<http://deepdive.stanford.edu>



# 第二部分 详细讲解

## 1 背景知识

### 1.1 因子图

![factor graph](https://upload.wikimedia.org/wikipedia/commons/3/32/Factorgraph.jpg)

上面就是一个因子图（Factor Graph）的例子（来自维基百科），因子图是一种**概率图模型**，Deepdive的**概率推测（Probabilistic inference）**就是在这个模型上执行的。图我们都知道，由节点(node)和弧或者叫边(edge)组成，因子图的节点由两种类型：变量（Variables）和因子（Factor），详细如下：

- Variables：如果这种结点的值已知，它就可以当作一个证据变量（用来推断别的值）；这个值也可以是未知的，这事就叫做查询变量，就是我们需要进行预测得到的值，比如图中的$X_1,X_2,X_3$；
- Factor：每个因子都可以连接到多个变量，并用因子函数定义它们之间的关系，比如图中的$f_1,f_2,f_3,f_4$ ；每个因子（或者说因子函数）都有一个权重值，来表示这个因子影响力的大小。换个说法，这个权重值表示了某个因子的可信程度，正数越大则越正确，负数越小则越错误（错误表示某种不可能，比如一个人的亲儿子同时是他的亲兄弟，这就是一个错误，所以这个因子的权重就应该是一个很小的负值）。因子函数的权重可以通过训练学习得到，也可以手动赋值（通过脚本或者`app.ddlog`）。

### 1.2 可能的世界

一个可能的世界（Possible  World）就是对因子图中的每个变量的取值，很多个可能的世界并不是等概率的，每个世界都有不同的概率。这个可能的世界，用我们的话来说就是一个“如果的世界”，就像下面这两句话：

> 如果他那天出门带伞了
> 如果他那天出门没带伞

假设我们不知道这个人到底带没带伞，而且天气预报说那天会下雨，这就是两个可能的世界，里面的事情都是我们假象的，既然是假设的那么就有了概率（也就是可能性的大小），这个人关注天气，生活细致，那么他带伞的概率就很大，不带伞的概率就很小。

可能世界的概率与因子图中所有因子函数的加权组合成比例，权重可以静态分配或自动学习。 如果训练的话，需要一些训练数据。 训练数据定义了一组可能的世界，训练过程，就是在给定的世界中，使概率达到最大的过程。

### 1.3 边际推理 

边际推理（Marginal inference）就是要推断：一个变量取某个特定值的概率是多少（比如骰子6点向上的概率:laughing:）

在这个任务中，可以使用全概率公式对这个变量取值的所有情况的概率的和（sum）来简单地表示这个我们要求的概率。

- 全概率公式：$P(B)=\sum\limits_{i=1}^nP(A_i)P(B|A_i)$  注意这里的事件$A_i$要相互独立且满足 $\sum P(A_i)=1$

精确推理是因子图的一个难题，这个问题比较常用的方法是吉布斯抽样。这个解决过程首先要从随机的可能世界开始，然后对每一个变量（v）进行迭代更新，更新的时候要考虑到这个变量所连接的因子结点的因子函数和连接到这些因子结点的变量的取值（这算是前面变量v的马尔科夫毯）。

当对这些随机变量进行了很多次迭代后，我们可以计算变量某个特定值出现次数占总迭代次数的比例，这个比例就是对个变量的这个特定取值的概率（根据大数定律:laughing:）。

- Gibbs Sampling 吉布斯抽样

### 1.4 DeepDive中的推理

DeepDive允许用户自己定义建立因子图的[推理规则](http://deepdive.stanford.edu/writing-model-ddlog)，那么，规则可以表达下面这种意思，

>  ”如果一次吃的多了，可能会消化不良。“

综合前面讲的因子图，也就是说，一个规则描述了一个因子结点的因子函数和这个结点所连接的变量。每一条规则都有权重（同样可以用户人工赋值或者由DeepDive计算出来），这个权重表示了这条规则的可信度。

这是一个微妙但非常重要的点。许多传统的机器学习算法，它通常假定的先验知识是精确的和孤立的进行预测，与之相反，deepdive进行联合推理：它同时决定所有事件的值。如果事件通过推理规则（直接或间接）连接，则允许事件相互影响。因此，一个事件的不确定性（约翰吸烟）可能影响另一事件的不确定性（约翰患有癌症）。随着事件之间的关系变得更加复杂，这个模型变得非常强大。例如，人们可以想象“约翰吸烟”的事件是否受到约翰是否有朋友吸烟的影响。在处理固有噪声信号，如人类语言时，这一点尤其有用。

下面是一个完整的DeepDive项目的流程与结构：

![aPr2E.png](https://s1.ax2x.com/2018/02/26/aPr2E.png)

### 1.5 弱监督

大多数机器学习技术需要一组训练数据。收集训练数据的传统方法是人工标记一组文档。例如，对于婚姻关系，标记者可能把“Bill Clinton”和“Hillary Clinton”标记为一个正面的训练实例。这种方法在时间和金钱上都是很昂贵的，如果我们的语料很大，都无法为我们的算法提供足够多的处理数据。而且由于人类会犯错误，由此产生的训练数据很可能是充满噪声的。

生成训练数据的另一种方法是弱监督（Distant supervision）。弱监督中，我们使用一个已经存在的数据库，如Freebase或特定领域的数据库，收集我们要提取的关系的例子。然后我们使用这些例子来自动生成我们的训练数据。例如，Freebase中包含了Barack Obama和Michelle Obama已经结婚的事实。我们利用这个事实，然后将出现在同一个句子中的每一个“Barack Obama”和“Michelle Obama”标记为我们婚姻关系的一个正面的例子。这样我们就可以很容易地生成大量的（可能是嘈杂的）训练数据。运用弱监督来为某一特定关系找到正面的例子是容易的，==但是产生否定的例子可要难得多了==。

### 1.6 DeepDive 系统概述和相关术语

本文档讲DeepDive作为一个系统来概括。现在假设你已经熟悉一些一般概念，如推理和因子图、关系抽取和弱监督。下面介绍了执行DeepDive应用过程中的每一个步骤：

- 提取
  - DeepDive首先要对数据进行实体抽取、实体链接、特征抽取、弱监督和其他建立后面用来执行推断的变量的必要步骤，这些工作就是**提取**步骤。如果需要的话，还要用数据来生成用来学习因子权重的训练数据。这些提取中的小步骤都是通过定义**提取器（extractor）**来制定的（这个提取器也可以通过**用户定义函数（user-defined functions)**来指定，就是放在`udf`文件夹中的脚本）。提取结果存储在应用程序数据库中，然后将根据用户指定的规则构建因子图。
- 因子图的固化（Grounding，==不太清楚怎么翻译==） 
  - DeepDive用因子图进行推理。用户编写SQL查询来指示系统要创建哪些变量，这些查询通常涉及在提取步骤期间填充的表。因子图的变量结点根据用户指定的推理规则连接到因子结点，并定义了描述变量之间关系的因子函数。用户可以指定系数权重应该是用户给定的常数还是由系统学习。
  - 固化是将因子图写入磁盘的过程，以便它可以用于执行推理。DeepDive把一个因子图存储为五个文件：一个变量文件，一个因子文件，一个弧文件，一个权重文件，和一个对这个系统有用的元数据文件。这些文件的格式是特殊的，以便它们可以被我们的==采样器==接受为输入。
- 权重的学习
  - deepdive可以通过弱监督从训练数据学习因子图的权重或由用户指定这些值，填充数据库提取阶段。学习这些权重的一般方法是极大似然法。
  - 学习好的权重接下来会写入一个特定的数据库表，以便用户在校准过程中检查它们。
- 推理
  - 最后一步是在因子图变量上进行边际推理，来学习他们在*可能世界* 中不同取值的概率。DeepDive使用自己的高通量的DimmWitted采样器（在MacBook上，比DeepDive默认的采样器快十倍）进行吉布斯抽样，遍历许多*可能世界* 并估计概率。采样器将固化的图（即前面说的五个文件）作为输入，连同一些参数来指定学习过程的参数。推理的结果被写入数据库。用户可以编写查询来分析结果。DeepDive还提供校准数据来评估推理的准确性。

### 1.7 知识库构建

知识库的构建（Knowledge Base Construction，KBC）过程，就是用从**数据**（数据可以使文本、声音、视频、图标等）中提取出来的**事实**（或者说声明、断言）填充知识库。比如，我们实验室正在构建的药物和其不良反应的知识库，当然还有一些其他的方面。比如可以构建一个古生物知识库来了解恐龙这一物种在什么时间何地生存过，人际关系知识库来表达父母、配偶、兄弟姐妹等关系。DeepDive就是用来简化知识库构建的过程。

举一个具体的例子，我们可能想要从网络上的文本中抽取配偶关系，那么我们就可以构建一个DeepDive应用。下图就展示了DeepDive在应用中的作用。其输入是像“U.S President Barack Obama's wife Michelle Obama ...”这样的句子，它的输出是由**三元组**组成的一个叫做`has_spouse`的表，表里的每一项都表示了一组配偶关系。DeepDive同时还为这些和这些关系相关的**概率**，来表示这条关系可信程度

![alcza.png](https://s1.ax2x.com/2018/02/27/alcza.png) 
<center>**Figure 1: An example KBC application**</center>

- KBC中的术语：KBC给它操作的对象起了些特别的名字，下面是比较重要的几个
  - 实体（Entity）：一个实体就是现实世界中的一个对象，比如一个人，一个动物，地理位置，甚至一段时间都算。比如“奥巴马”就是一个实体。
  - **Entity-level data** are relational data over the domain of conceptual entities (as opposed to language-dependent mentions), such as relations in a knowledge base, e.g. ["Barack Obama" in Freebase](http://www.freebase.com/m/02mjmr).
  - A **mention** is a reference to an entity, such as the word "Barack" in the sentence "Barack and Michelle are married".
  - **Mention-level data** are textual data with mentions of entities, such as sentences in web articles, e.g. sentences in New York Times articles.
  - 实体链接（Entity Linking）：找到一个Mention所对应的实体。比如“美国总统”，“特朗普”，“川普”可能都是说的同一个实体“美国总统特朗普”，而实体链接的任务就是发现这些关系。
  - A **mention-level relation** is a relation among mentions rather than entities. For example, in a sentence "Barack and Michelle are married", the two mentions "Barack" and "Michelle" has a mention-level relation of `has_spouse`.
  - An **entity-level relation** is a relation among entities. For example, the entity "Barack Obama" and "Michelle Obama" has an entity-level relation of `has_spouse`.

上面这些概念之间的关系如下图所示：

![Data Model](http://deepdive.stanford.edu/images/walkthrough/datamodel.png) 
<center>**Figure 2: Data model for KBC**</center>

- KBC数据流（Data Flow）

在一个典型的KBC应用中，该系统的输入一般是一系列普通的文本，系统的输去是一个数据库（也就是知识库），这个知识库中包含了你需要的关系（实体级或者Mention级）。

就像前面第6部分的系统概述中所说，获取输出的步骤如下：

1. 数据预处理
2. 特征抽取
3. 使用陈述性语言生成因子图
4. 统计推断和学习

这里要特别说明的是：

1. 在数据预处理步骤中，DeepDive接受文本格式的输入数据，把它们载入数据库，并且进行分词，断句等操作，得到每个句子中每个词的`POS`、`NERtag`等等特征。
2. 在特征抽取步骤中，DeepDive运行开发者定义的`extractor`函数来把输入数据转换为被称作**证据（Evidence）**的关系信号。证据有两部分：
   - 实体级或者Mention级的候选关系；
   - 这些候选关系的（语言）特征。
3. 下面的步骤，DeepDive使用Evidence来生成因子图。为了指示DeepDive如何构建这个因子图，开发者需要使用一个类似SQL的陈述性语言来指定推理规则。
4. 再接下来，DeepDive自动在生成的因子图上执行学习和统计推理。在学习过程中，计算得到推理规则中指定的因子权重。这些权重直接地表示了规则的可信度。在推理过程中，会计算得到变量的联合概率，这个概率表示了特定“事实”为真的可能性。

推理结束后，其结果被存储在几个数据库表中。开发者可以通过SQL查询来获取结果，用校准图检查结果，或者执行错误分析来改进结果。

下图是一个例子，他展示了把句子“U.S President Barack Obama's wife Michelle Obama...”经过上面说的前三步处理的过程：

![Data Flow](http://deepdive.stanford.edu/images/walkthrough/dataflow.png) 
<center>**Figure 3: Data flow for KBC**</center>

在上面的例子中：

1. 在数据预处理时，句子被分词，之后进行词性标注、命名实体识别处理；
2. 特征抽取过程中，DeepDive提取了这些内容：
   - 提到的人和出现的位置；
   - 配偶关系`has_spouse`的候选关系；
   - 这些候选关系的`feature`（特征），比如Mention之间的词。
3. DeepDive使用用户定义的规则来构建因子图。

## 2 构建DeepDive应用

前面我们已经了解了DeepDive基本的工作流程，那么出了第一部分的例子，我们应该如何着手构建一个DeepDive的应用呢？

这里我们先要明白一点，没有任何一个DeepDive应用是一蹴而就的（甚至可以说是任何机器学习任务），一开始的准确率都不会很高很高，所以一般的情况是：我们先开发一个“垃圾”的初期版本，然后通过错误分析和调试迭代更新来提高质量。DeepDive中带着一打工具可以使用，可以让你尽量快地开发出一个可用的应用（当然最重要的还是使用工具的你，剑本无不同，只因御剑之人不同）。简单来说，构建一个DeepDive就分三步：写流程和函数、运行、分析和调试...然后不停循环迭代。

### 2.1 写什么，怎么写

#### 2.1.1 DeepDive应用的结构

前面我们只介绍了DeepDive的运行过程，现在来具体说说，DeepDive的应用从文件的角度来看是什么样的。

先来谈谈我们见过的其他应用是什么样子的，*Android*的应用打眼一看是`.apk`后缀的文件，其内部结构其实和一般*JAVA*工程没有本质的区别（至少以前是这样，因为作者现在不学Android了）。现在说DeepDive，一个应用就是一个独立的**文件夹**，这个文件夹和该目录下所有文件（包括文件夹）的总和就是一个DeepDive应用。

这个**文件夹**下面的大体内容如下：

```
-A_DeepDive_Application(文件名自己取，任意合法目录名都行)
--app.ddlog            (这个文件由用户编写，其内容是指挥DeepDive如何工作的，包括定义上一节说的规则)
--deepdive.conf		   (app.ddlog中没有的DeepDive配置，也可用HOCON语法定义规则，但更推荐用app.ddlog)
--db.url			   (里面是配置的数据库的URL，若有环境变量$DEEPDIVE_DB_URL将会忽略这个文件)
--input/			   (存放待处理的所有数据)
----[init.sh]		   (可以没有。可执行文件，控制数据如何写入数据库)
--udf/				   (存放用户定义函数的目录，User Defined Functions的缩写)
--run/				   (存放DeepDive运行时产生的记录和结果,由DeepDive生成)
----LATEST/               最后一次运行记录
----FINISHED/             最后一次运行成功的记录
----ABORTED/		      最后一次运行失败的记录
--[schema.json|schema.sql]  （设置数据库底层数据表的数据定义语言，若DeepDive中写了，这个文件会被忽略）
```

好，看完上面的结构，我们应该有了一个大概的认识，`app.ddlog`、`db.rul`、`udf/`中三部分的内容是要我们自己往里面写东西的。后面我们就主要围绕它们来介绍。

#### 2.1.2 在app.ddlog定义数据流

这里，我们要用到的语言是专为DeepDive打造的叫做**DDlog**的语言，其语法类似*Datalog* ，我们在第一部分的实例中已经见识过了。这一小部分就只介绍**如何定义数据流**，其他部分在后面有介绍。

**DDlog**中，所有的语句都要用英文句号`.`结束，而注释用井号`#`开头。而且其代码先后顺序没有要求。

- 定义数据表Schema

  首先要定义我们在DeepDive中读入的数据的结构（和数据文件的结构相同），和我们在应用中用到的结构（比如第一部分中的`sentences`表）。这里有统一的格式：

  ```ddlog
  relation_name(
  	column1 type1,
  	column2 type2,
  	...
  ).
  ```

  首先是表名，其内部结构在小括号内表示，每一个属性之间用逗号`,`分隔，最后一项后面没有逗号。还要记得，每个完整的定义后面都要带**英文句号**。其中的类型可以使底层数据库支持的任意格式，因为DeepDive会把这里定义的表映射到数据库上去。常用的类型已经在第一部分使用过了，更多的类型可以查看相应数据库支持的类型。比如在我们前面使用的PostgreSQL数据库中，支持的格式：[PostgreSQL's types](http://www.postgresql.org/docs/9.1/static/datatype.html#DATATYPE-TABLE)

- <p id="normal-rule">基本的推导规则</p>

  前面定义了数据，现在看看如何让我们的数据流动起来（就是让数据经过处理得到新的内容的部分，但是现在还是预处理的部分）

  这里我们使用类似**Datalog**的语言来定义从某些关系中推导新关系的方法。比如下面就是通过关系`R`、`S` 来推导关系`Q`：

  ```DDlog
  Q(x,y) :- R(x,y) , S(y)
  ```

  具体来讲，比如关系`R`就是双亲，y是x的双亲，关系`S`就是男性。根据这两个关系，就能知道关系`Q`可以表示*父亲* 关系，y是x的父亲。这就是一个简单推理。

  上面的规则中，`Q(x,y)`被称为**Head Atom**，`R(x,y)`、`S(y)`被称为**Body Atom**。x、y是规则中的变量。而上面整个句子可以看作一个**表达式**，下面给出了一些表达式的例子：

  ```DDlog
  a(k int).
  b(k int, p text, q text, r int).
  c(s text, n int, t text).

  Q("test", f(123), id) :- a(id), b(id, x,y,z), c(x || y,10,"foo").

  R(TRUE, power(abs(x-z), 2)) :- a(x), b(y, _, _, _).

  S(if l > 10 then TRUE
  else if l < 10 then FALSE
  else NULL end) :- R ( _, l).
  ```

  从上面的例子我们可以看出，在**Atom**中，我们可以使用常量（比如“test”，*True* ），也可以使用函数（比如*f(123)*）还有前面提到的变量。而且还支持简单的*if_then_else* 结构。

  在这里支持的常量类型有这些：

  - 数字类型：123，-3.23
  - 字符串类型：注意，句子中的引号和关闭小括号`)`之前都要加转义字符`\`否则就会出问题。
  - 布尔类型：False、True
  - 空类型：NULL

  在表达式间可以使用操作符，就像例子中写的有两种情况`expr op expr`和`op expr`见下面：

  - 二元操作符：
    - 数字运算：`+`、`-`、`*`、`/`
    - 字符串链接：`||`
  - 一元操作符：
    - 负号：`-`
    - 布尔反：`!`

  表达式可以嵌套，适当的时候要使用小括号来让运算顺序更清晰。

  > 曾经在书上见过这样一句话，写代码的时候，永远不要考虑运算符优先级，在一切有可能混淆的地方，使用括号来规定运算顺序。

  其实还有比较运算符：

  - 等式：`=` 这里和常见语言语法有点差别，使用一个等号
  - 不等于：`!=`
  - 不等：`<`、`<=`、`>=`、`>`
  - 空：`IS NULL`、`IS NOT NULL`
  - 字符串比较：`exprText LIKE exprPattern`

  函数调用和条件表达式在前面的例子中已经很清晰了，下面说说类型转换。

  在学习其他编程语言的时候类型转换已经接触很多了，在DDlog中的用法这样：`expr :: type`:

  ```DDlog
  num_occurs / total :: FLOAT
  #否则会执行整数的除法，得到的结果还是整数
  ```

  另外，在规则中，经常会有关系中的某些变量在推导过程中用不到，那么这时就有了**占位符（Placeholder）**。一个占位符就是一个下划线`_`，它只可以用在**Body Atom**中，表示这一列的变量用不到，就像下面这个：

  ```
  Q(x, y, x + y) :- R(x, _, _, y, _, _, _).
  #这里关系Q的推导，只和关系表R中的第一列和第四列有关系
  ```

- 条件

  条件和**Body Atiom**同级，使用其中用到了的变量。相连接的条件用逗号分隔，不连接的条件用分号分隔，感叹号可以用来否定一个条件。条件可以用中括号`[]`任意嵌套，比如下面的例子：

  ```
  Q(x) :- b(x, _, _, w), [ x + w = 100; [ !x > 50 , w < 10 ]].
  ```

- 析取（Disjunctions）

  **Disjunction**可以使用多条相同**Head Atom**的规则来表示，也可以在一条规则中的**Body Atom**用分号分隔多个bodies。可以像下面这样：

  ```
  Q(x, y) :- R(x, y), S(y).
  Q(x, y) :- T(x, y).
  #也可以像下边这样
  Q(x, y) :- R(x, y), S(y); T(x, y).
  #它们的意义相同

  ```

- 聚合函数（Aggregation）

  就像普通的SQL语句一样，DDlog中也有聚合函数的存在，它可以在各个独立的取值上进行一些数学操作，比如求某列的最大值等等，DDlog中有如下几个聚合函数：

  - `SUM`
  - `MAX`
  - `MIN`
  - `ARRAY_AGG`
  - `COUNT`

  下面的例子中，Q提取了R中第一列和第二列所有的值作为Q的第一列和第二列（相同的被视为同一组），然后Q的第三列是对应到R的每组中的第三列的最大值：

  ```
  Q(a,b,MAX(c)) :- R(a,b,c).
  ```


- 去重

  为了只从其他关系中得到不重复的元素，可以使用`*:-`操作符：

  ```
  Q(x,y) *:- R(x, y).
  ```

  上面的例子中，只有R中不重复的(x,y)才会插入到Q中。

- 量词

  - 存在量词和全程量词

    另外一个减少头部获得的三元组的方式是使用量词结构。在Body Atom和条件结构中，可以被量词`EXISTS`或者`ALL`约束，其语法如下：

    ```
    body, ..., EXISTS[body, body, ...], body, ...
    body, ...,    ALL[body, body, ...], body, ...
    ```

    在量词内部的关系表示了对于同时在内部和外部出现的变量进行的约束。`EXISTS`表示的是：对于所约束的变量，必需存在一种满足约束内部关系的组合；`ALL`表示：对于所约束的变量，其所有组合都必需满足约束中的内容。这样就让我们可以定义更加复杂的推理工作，比如下面

    ```
    P(a) :- R(a, _), EXISTS[S(a, _)].
    Q(a) :- R(a, b),    ALL[S(a, c), c > b].
    ```

    第一个例子，只要R中的第一列的值同样也出现在S的第一列，那么就可以加入P

    第二个例子，要求R中第一列的值，在对应S中第一列的值和其相同，而对应的所有第二列的值都要大于R中第二列的值，只有满足这些条件，才能插入Q中。

  - 条件量词

    `OPTIONAL`量词，表示在其内部的条件可以不满足，照样加入到Head Atom中，就像SQL中的*outer joins* 一样，如果一个变量只在条件量词的内部出现，那么如果没有满足条件的值的情况下，Head中的对应值就是`NULL`。具体可以看下面的例子：

    ```
    Q(a, c) :- R(a, b), OPTIONAL[S(a, c), c > b].
    ```

    上面都是，Q的第一列和R的第一列相同，如果还存在S(a,c)，而且c>b，那么Q的第二列就为c的值。如果S中不存在c与a对应，或者说c不大于b，那么Q中的第二列的值就为`NULL`

#### 2.1.3 用Python编写用户定义函数（UDF）

除了用DDlog写简单的推理规则，DeepDive还支持用户定义的函数，这叫做**UDFs**，被放置于DeepDive项目的`udf/`文件夹中，这些函数也用来进行数据的处理。UDF可以是任何一个接受制表符分隔的**TSJ**格式文件或者制表符分隔的标准输入流中的输入，然后他的输出也是同样的格式。TSJ格式要比单纯的每一行一个JSON对象，然后重复相同的名字要高效的多。

下面的内容展示了DeepDive推荐的如何写合适的Python脚本以及如何在DDlog中使用它。

- 在DDlog中使用UDF

  首先定义一些数据模式，方便下面的例子使用：

  ```
  article(
      id     int,
      url    text,
      title  text,
      author text,
      words  text[]
  ).

  classification(
      article_id int,
      topic      text
  ).
  ```

  假设我们想写一个函数来吧每个articles都给一个`topic`，下面是具体：

- 函数声明

  函数的声明就像C、JAVA一样，需要有参数类型、返回值类型等：

  ```
  function <function_name> over (<input_var_name> <input_var_type>,...)
      returns [(<output_var_name> <output_var_type>,...) | rows like <relation_name>]
      implementation "<executable_path>" handles tsj lines.
  ```

  通过上面的语法，如果我们使用article关系中`author`和`words`属性来决定这个文章主题是什么，而脚本文件我们放在`udf/classify.py`。那么具体的声明就是这样：

  ```
  function classify_articles over (id int, author text, words text[])
      returns (article_id int, topic text)
      implementation "udf/classify.py" handles tsj lines
  ```

  需要注意的是，returns语句后面可以接输出的每个属性的组合，也可以直接使用之前定义好的模式（关系）：就像上面和下面的关系

  ```
  function classify_articles over (id int, author text, words text[])
      returns rows like classification
      implementation "udf/classify.py" handles tsj lines.
  ```


- 函数的调用方法

  函数的调用和赋值就好像我们前面说过的[基本推理规则](#normal-rule)类似的格式，下面使用添加的方法使用函数返回的数据来填充`classification`关系。

  ```
  classification += classify_articles(id, author, words) :-
      article(id, _, _, author, words).
  ```

  `:-`后面的就是函数的参数名绑定，同样还有前面说过的占位符。


- 使用Python写UDF

  DeepDive提供了一个用Python写用户定义函数的模板。实际上提供的是几个 [Python 装饰器](https://www.python.org/dev/peps/pep-0318/) 来简化输入和输出的格式. 被调用的用来处理每一行输入数据的Python生成器需要被`@tsj_extractor`装饰（被放在`def`关键字前面的一行）。关于生成器可以参考我的一篇博客[Python yield](http://blog.csdn.net/cx943024256/article/details/79005309) 。生成器中输入和输出中每个参数的类型可以通过定义函数名的参数默认值来给定，并且用`@returns`来装饰。他们定义了具体的输入输出操作，而在我们自己只需要写一个被调用的**生成器**（函数），通过参数获取输入，通过`yield`返回每一行结果。

  下面是一个简单的例子，也就是上面我们提到的`udf/classify.py`分类的操作的程序：

  ```python
  #!/usr/bin/env python
  from deepdive import *  # Required for @tsj_extractor and @returns

  compsci_authors = [...]
  bio_authors     = [...]
  bio_words       = [...]

  @tsj_extractor  # Declares the generator below as the main function to call
  @returns(lambda # Declares the types of output columns as declared in DDlog
          article_id = "int",
          topic      = "text",
      :[])
  def classify(   # The input types can be declared directly on each parameter as its default value
          article_id = "int",
          author     = "text",
          words      = "text[]",
      ):
      """
      Classify articles by assigning topics.
      """
      num_topics = 0

      if author in compsci_authors:
          num_topics += 1
          yield [article_id, "cs"]

      if author in bio_authors:
          num_topics += 1
          yield [article_id, "bio"]
      elif any (word for word in bio_words if word in words):
          num_topics += 1
          yield [article_id, "bio"]

      if num_topics == 0:
          yield [article_id, None]
  ```

  这个函数简单地为每一个文章赋予了一个主题属性。首先看其作者信息。如果不知道作者具体信息，那么就在文章词汇里面找是否有我们提前准备的字典的词出现。最后如果都没有合适的，直接给一个`None`。

  这里面需要注意的就是，如果想使用前面提到的两个装饰器，那么必须要导入DeepDive的Python包：

  ```python
  from deepdive import *
  ```

  然后在`@returns`装饰器后面要带上输出的数据的数量和格式。

  - `@tsj_extractor`装饰器

    这个装饰器是用户写的函数的第一个装饰器（最外层），它可以处理数据的输入，然后将列表作为返回值。同时这个装饰器也是用来告诉DeepDive需要调用哪一个函数（在一个Python文件中可以定义很多函数）。还有一点DeepDive的UDF不仅能处理**TSJ**格式，也能处理**TSV**格式，这就需要另一个装饰器`@tsv_extractor`，但是强烈推荐**TSJ**格式。


  - 注意事项

    一般来说，用户定义的生成器（函数）要放在程序文件的最后，除非在处理完所有输入数据还有其他任务要完成。而且，所有被装饰的函数所用到的第三方函数和变量，都要在装饰器之前来定义。

    在生成器中，**请不要**使用普通的`pring`或者`sys.stdout.write`来进行命令行的输出，请**使用**`print >> sys.stderr`或者`sys.stderr.write`来代替，更详细的记录信息的问题请看后面的[调试部分](http://deepdive.stanford.edu/debugging-udf)。


  - 参数默认值和`@returns`装饰器

    为了正确地解析每行TSJ数据为合适的Python数据类型，并把返回值正确的转换成TSJ格式，我们需要做Python程序中给出其对应的每个数据列的类型。这跟应该是和DDlog中声明的函数参数和返回值类型相同的。输入数据的类型可以直接在函数的参数默认值中给定，就像上面的例子中那样。输出数据的格式在`@returns`装饰器中定义，即可以是作为参数的一个`name-type`对列表，也可以是将数据类型作为默认参数的一个函数。使用`lambda`是个更好的选择，因为使用列表的话，需要很多其他的声明，这里`lambda`是匿名函数的意思，具体参考可以看[Python中的Lambda](http://blog.csdn.net/cx943024256/article/details/79569853)。

    如果是用列表的话，上面的例子中可以写成这样：

    ```python
    @returns([("article_id", "int"), ("topic", "text")])
    ```

    可能同学会问，为什么这里不是用Python中的字典类型呢，那不是看起来更好？这是因为在Python中，它不会考虑到其内部的顺序，但是我们的输入数据，第一列第二列都是有顺序的，所以普通的字典不可以，这里只能使用列表类型。

    使用函数默认值的方式的话，`@returns`装饰器中的函数永远不会被调用，所以我们直接给一个匿名函数就好，然后其函数体可以任意一个简单表达式，这里使用的一个空列表作为其返回值，当然用你想用的任意一个都可以。DeepDive也提供了一个像`@returns`一样的用来定义输入数据类型的装饰器：`@over`，你可以看见这个词和DDlog中的一样，但是非常不推荐这样，既然函数参数默认值已经可以实现了，何必要再搞点多余的东西呢，这个只要知道有这个即可。

- 运行和调试UDFs

  当一个用户定义函数写好了以后，可以使用 `deepdive do` 或者 `deepdive redo` 命令来运行，第一次运行可以使用第一种，再运行就要使用第二种了。之前我们定义的提取类型关系的，就可以使用下面这条命令来执行。

  ```
  deepdive redo classification
  ```

  这条命令会使用`udf/classify.py`这个Python程序，给其TSJ格式作为其输入，然后将返回值添加到数据库的`classification`数据表中。

  这一节主要是关于程序的编写，后面会专门讲到UDFs的运行和调试。

#### 2.1.4 使用DDlog制定统计模型

每个DeepDive应用，都可以看作是一个统计推理问题，其输入是把普通的输入数据经过一系列处理后的东西。这一节有3部分：

1. 介绍了如何声明DeepDive应用统计模型中的随机变量
2. 如何定义随机变量的监督标记
3. 如何为特定的特征和关系写下推理规则

##### 2.1.3.1 变量的声明

DeepDive要求用户自己在`app.ddlog`中声明*变量关系* 的名字和类型，其之和普通的关系语法有一点点差别，**关系变量**中存储了在概率推理时用到的随机变量。现在DeepDive支持布尔变量和类别变量。

- 布尔类型

  在关系名字的后面加一个问号，说明这是一个包含了随机变量的变量关系，以此来和普通关系区分开。变量关系的列元素看作是键，下面是个简单例子：

  ```
  has_spouse?(p1_id text, p2_id text).
  ```

  上面定义了一个叫做`has_spouse`的变量关系，其每个唯一的p1、p2的组合都代表了模型中一个不同的随机变量。

- 类型变量

  DeepDive支持类型变量，用从零开始的整数来表示不同的变量，他和布尔类型变量关系的语法之间就差了一个`Categorical(N)`后缀，N代表了类别的数量，比如下面的例子，就定义了`tag`就是一个有13种类型的类别变量关系：

  ```
  tag?(word_id bigint) Categorical(13).
  ```

##### 2.1.3.2 确定范围和监督的规则

在声明了一个变量关系后，要一起定义他的范围和监督数据的标签。这说明了变量关系的几个特点：

- 变量关系每一列的所有可能取值都是从其他已经定义好的关系中取出来的；
- 无论随机变量的取值是`TRUE`还是`FALSE`，或是某一个类别，他都需要一个特定的语法来定义

基于上面两个规则，定义变量关系的语法和普通的推理规则有很大的类似，比如下面的例子：

```
has_spouse(p1_id, p2_id) = NULL :-
    spouse_candidate(p1_id, _, p2_id, _).
```

上面就是，定义每一个在`spouse_candidate`中不同的`p1_id`、`p2_id`的组合都对应到`has_spouse`中的一个变量，等号右边的`NULL`表示现在添加的是非监督数据。`:-`表示其数据来源符号，和前面规则定义部分的相似。

下面是另一种情况，下面添加的是监督数据还配合了简单的表达式：

```
has_spouse(p1_id, p2_id) = if l > 0 then TRUE
                      else if l < 0 then FALSE
                      else NULL end :- spouse_label_resolved(p1_id, p2_id, l).

```

上面的条件表达式可以看作是一个规则，这个结果根据`l`的大小得来。这同样是把连续的浮点数概率转换为布尔类型的方法。

##### 2.1.3.3 推理规则

推理规则可以给定变量的特征或者是变量之间的关系，其最基本的就是我们前面所提到的**因子图**中的因子，它可以指导DeepDive如何分析数据。而推理模型这里对其基本语法进行了扩充，这允许在规则定义的前一行指定因子的类型，就像下面给定权重：

```
@weight(...)
FACTOR_HEAD :- RULE_BODY.
```

`RULE_BODY`表示的是一个普通的联合查询，就像我们之前在DDlog中定义的一样。

`FACTOR_HEAD`表示的是一种特殊的语法结构，它可以包含多个**变量关系**后面我们会主要介绍。

- 指定特征

  一般情况下，我们都想用一组特征来构建布尔变量为`true`的概率模型，这件事在DDlog很容易就能实现。在`FACTOR_HEAD`的部分写上一个变量关系，DeepDive会为其在模型中创建一个一元的因子，并且把相关的节点进行连接。每个一元因子的权重都可以是认为指定的特征，比如下面的例子：

  ```
  @weight(f)
  has_spouse(p1_id, p2_id) :-
      spouse_candidate(p1_id, _, p2_id, _),
      spouse_feature(p1_id, p2_id, f).
  ```

  上面的例子：

  - 为每个`spouse_candidate`中的组合创建一个因子，其特征都来自于`spouse_feature`
  - 每个因子都连接到一个`has_spouse`变量
  - 每个因子的特征`f`以后会通过学习来决定权重，权重值决定了因子所代表的可能性大小，也就是决定了`has_spouse`中变量的取值为`TRUE`还是`FALSE`


- 指定关系

  现如今，很多问题中变量之间都是由某些神秘的联系，我们可以通过领域知识来丰富模型处理这些联系。比如吸烟的人患有肺癌的几率可能会大。这种联系在模型中定义，需要先创建特定类型的因子，这个因子还要联系起多个相关的变量。DDlog借用了**Markov Logic Networks**和**First-Order Logic**的语法，具体定义如下：

  ```
  @weight(3)
  smoke(x) => cancer(x) :-
      person(x).
  ```

  上面吸烟的例子，可以在DeepDive官网找到（[传送门](http://deepdive.stanford.edu/example-smoke)）

  上面就表示了如果吸烟，那么同样暗示了会患有肺癌。

  权重为3表示了这个规则的可信程度，越大的正数说明约可信，这里使用常数就不需要像前面说的从数据中学习了。
  - 隐含（implication）

    隐含关系可以如下面几种语法定义，关于逻辑隐含可以看维基中的定义（[传送门](http://en.wikipedia.org/wiki/Truth_table#Logical_implication)）

    ```
    @weight(...)  P(x) => Q(y)              :- RULE_BODY.
    @weight(...)  P(x), Q(y) => R(z)        :- RULE_BODY.
    @weight(...)  P(x), Q(y), R(z) => S(k)  :- RULE_BODY.
    ```

    ​

  - 逻辑或

    逻辑或就是这样定义，当两个关系都不成立的话，就为`FALSE`，其余情况为`TRUE`

    <center>Logical disjunction</center>

  | *p*  | *q*  | *p* ∨ *q* |
  | ---- | ---- | --------- |
  | T    | T    | T         |
  | T    | F    | T         |
  | F    | T    | T         |
  | F    | F    | F         |

  ​	可以按照如下方式定义

  ```
  @weight(...)  P(x) v Q(y)         :- RULE_BODY.
  @weight(...)  P(x) v Q(y) v R(z)  :- RULE_BODY.
  ```
  - 逻辑与

    与逻辑或类似，不过是只有当两个关系都存在时才为`TRUE`

  <center>Logical Conjunction</center>

  | *p*  | *q*  | *p* ∧ *q* |
  | ---- | ---- | --------- |
  | T    | T    | T         |
  | T    | F    | F         |
  | F    | T    | F         |
  | F    | F    | F         |

  ​	按照如下语法定义：

  ```
  @weight(...)  P(x) ^ Q(y)         :- RULE_BODY.
  @weight(...)  P(x) ^ Q(y) ^ R(z)  :- RULE_BODY.
  ```

  - 等价

    逻辑等价只有再两个关系都存在或者都不存在为`TRUE`

  <center>Logical Equality</center>

  | *p*  | *q*  | *p* ↔ *q* |
  | ---- | ---- | --------- |
  | T    | T    | T         |
  | T    | F    | F         |
  | F    | T    | F         |
  | F    | F    | T         |

  ​	按照如下语法定义：

  ```
  @weight(...)  P(x) = Q(y)  :- RULE_BODY.
  ```
  - 取反

  		当我们需要某个变量为`FALSE`才能操作的情况，就可以再关系名前面加上一个感叹号`!`，定义如下：

  ```
  @weight(...)    P(x) => ! Q(x)  :- RULE_BODY.
  @weight(...)  ! P(x) v  ! Q(x)  :- RULE_BODY.
  ```
  - Multinomial factors (STALE. NEEDS REVAMP.)

    前面的几种关系都是使用的布尔类型的变量关系，那么之前我们提到的类别变量如何处理呢？

    DeepDive对类别型变量只有很有限的支持，

DeepDive has limited support for expressing correlations of categorical variables. The introduced syntax above can be used only for expressing correlations between Boolean variables. For categorical variables, DeepDive only allows the conjunction of the variables each taking a certain category value being true to be expressed using a special syntax shown below:

- f

  - d

    ```
    @weight(...)  Multinomial( P(x), Q(y) )        :- RULE_BODY.
    @weight(...)  Multinomial( P(x), Q(y), R(z) )  :- RULE_BODY.
    ```

    `Multinomial`只接受类别变量作为参数，

`Multinomial` takes only categorical variables as arguments*, and it can be thought as a compact representation of an equivalent model with Boolean variables corresponding to each category connected by a conjunction factor for every combination of category assignments.

For example, suppose `a` is a variable taking values 0, 1, 2, and `b` is a variable taking values 0, 1. Then, `Multinomial(a, b)` is equivalent to having factors between `a` and `b` that correspond to the following indicator functions.

- I{`a` = 0, `b` = 0}
- I{`a` = 0, `b` = 1}
- I{`a` = 1, `b` = 0}
- I{`a` = 1, `b` = 1}
- I{`a` = 2, `b` = 0}
- I{`a` = 2, `b` = 1}

Note that each of the factors above has a distinct weight, i.e., one weight for each possible assignment of variables in the `Multinomial`factor. For more detail on how to specify Conditional Random Fields and perform Multi-class Logistic Regression using categorical factor, see [the chunking example](http://deepdive.stanford.edu/example-chunking).

* Because of this limitation, categorical variables and categorical factor support is likely to go away in a near future release, in favor of a more flexible way to express multi-class predictions and mutual exclusions. Emulate categorical variables with functional dependencies in the Boolean variable columns, i.e., `tag?(@key word_id BIGINT, pos TEXT)` rather than `tag?(word_id BIGINT) Categorical(N)`, and replace `Multinomial` factor with conjunction of those Boolean variables with columns having functional dependencies.
* Specifying weights

Each factor is assigned a *weight*, which represents the confidence in the correlation it expresses in the model. During [statistical inference](http://deepdive.stanford.edu/inference), these weights translate into their potentials that influence the probabilities of the connected variables. Factor weights are real numbers and only the relative magnitude to each other matters. Factors with larger weights have a greater impact on the connected variables than factors with smaller weights. Weights can be fixed to a constant manually, or [they can be learned by DeepDive](http://deepdive.stanford.edu/ops-model#learning-the-weights) from the supervision labels at different granularity. Weights can be parameterized by some data originating from the data (in most time referred to as *features*), in which case factors with different parameter values will use different weights. In order to learn weights automatically, there must be [enough training data available](http://deepdive.stanford.edu/relation_extraction).

DDlog syntax for specifying weights for three different cases are shown below.

```
Q?(x TEXT).
data(x TEXT, y TEXT).

# Fixed weight (10 can be treated as positive infinite)
@weight(10)   Q(x) :- data(x, y).

# Unknown weight, to be learned from the data, but not depending on any variable.
# All factors created by this rule will have the same weight.
@weight("?")  Q(x) :- data(x, y).

# Unknown weight, each to be learned from the data per different values of y.
@weight(y)    Q(x) :- data(x, y).
```

### 2.2 怎么运行

#### 2.2.1 编译DeepDive应用

要想运行DeepDive应用，那么必须要先编译它，就和Java或者C语言类似。下面会介绍几个DeepDive命令的用途和语法，和他们相关的文件。

##### 2.2.1.1 如何编译

要编译DeepDive应用，只要像下面这样再终端中使用命令就行：

```shell
deepdive compile
```

所有的`do`或者`redo`这类的命令，都是在编译的基础上执行的，所以必须要先编译，**每次DDlog修改后也要编译才能生效**。

##### 2.2.1.2 什么东西被编译了

所有编译出来的文件，都在应用目录中的`run/`目录下。主要文件如下：

- `run/dataflow.svg`

  处理中的数据流图，包含了数据流和各个关系、规则、函数和处理过程的依赖关系，可以直接用浏览器打开。

- `run/Makefile`

  依赖关系的镜像，用来后面生成特定数据流或操作的`plan`

- `run/process/**/run.sh`

  包含了每个处理过程需要执行的Shell脚本，实际上就是在执行`deepdive do ...`命令时候执行。

- `run/LATEST.COMPILE/` and `run/compiled/`

  每个编译步骤都会在`run/`中创建一个目录，其名字是编译的日期时间戳，其中存储的是一堆`.json`文件。这些文件是编译的中间文件。而且有个特殊的文件夹`run/LATEST.COMPILE`是最近一次编译的版本的对应文件夹的快捷方式，同样也可以找到`deepdive.conf`文件

  - `config.json`

    编译的最终配置信息

  - `code-*.json`

    用来生成实际的代码的对象

  - `compile.log`

    最后一个编译步骤的记录信息，可以用来检查错误

结合上面所说的，`run/`文件夹就是运行中的中间文件，所以在代码托管工具中，没有必要托管这个文件。

##### 2.2.1.3 什么时候需要编译

下面这三个文件，是DeepDive中编译的三个相关文件:

- `app.ddlog`
- `deepdive.conf`
- `schema.json`

所以，如果这三个文件中任意一个发生改变，都要重新执行`deepdive compile` 进行编译，然后其更改才会生效，否则还会按照上次编译的情况执行 `deepdive do`. 

##### 2.2.1.4 为什么要编译

编译有两个好处：

- 通过编译得到的不同操作之间的关系，可以优化程序运行的速度
- 可以在运行之前就能识别到错误

##### 2.2.1.5 编译时检查的项目

所有的`extractor`和数据表可以在`app.ddlog`中按照任意的顺序定义：整个数据流程是在编译的时候确定的，同时还会做很多检查，如果这些检查不通过就会报错。但是，我们也输入命令`deepdive check`进行检查，下面是几个检查项目：

- `input_extractors_well_defined`: 检查`extractor`是不是都定义完整.
- `input_schema_wellformed`: 检查数据表schema的变量定义是否正确.
- `compiled_base_relations_have_input_data`: 没有使用函数或规则填充的数据表是不是都有对应的输入文件，而且这个文件必须是在`input/`文件夹中，而且命名规则可以让DeepDive自动载入，这个在下一节会讲。
- `compiled_dependencies_correct`: 检查各个处理步骤之间的依赖是否正确，实际上就是检查`deepdive.conf`文件中的`extractor`是不是其输入值都是有效的输入（来源于数据表或者其他`extractor`）
- `compiled_input_output_well_defined`: 检查编译的输入输出文件是否正常
- `compiled_output_uniquely_defined`: 检查编译的输出是否都来自于同一进程

#### 2.2.2 管理输入的数据和产生的数据

DeepDive提供了一系列方便使用的命令来管理输入和自己产生的数据，实际的处理数据的介绍在后面会提到。

#####  2.2.2.1 检查关系模式（Schema）

==OPENKG 下载的有问题，没有`deepdive relation这个命令`==

下面是几种检查关系模式的方法，但是要注意，这些命令只有才进行了前面的编译过程才有效。

- 显示所有关系（数据表）

  ```
  deepdive relation list
  ```

  其结果可能是（按照第一部分实例的结果）：

  ```
  articles
  has_spouse
  num_people
  person_mention
  sentences
  spouse_candidate
  spouse_feature
  spouse_label
  spouse_label__0
  spouse_label_resolved
  spouses_dbpedia
  ```

- 列出某个关系的各个列名及类型

  ```
  deepdive relation columns articles
  ```

  其结果为：

  ```
  id:text
  content:text

  ```

- 显示变量关系

  ```
  deepdive relation variables
  ```

  其结果为：

  ```
  has_transaction
  ```

##### 2.2.2.2 数据库准备工作

为每个DeepDive应用对应的数据库在`db.url`文件中定义，或者可以在环境变量`DEEPDIVE_DB_URL`中定义。

- 初始化数据库

  把数据库初始化为什么都没有的状态，也可以用来删掉以前的东西

  ```
  deepdive db init
  ```

- 新建数据表

  一般来说，在`app.ddlog`中定义的表都会自动创建，当然也可以手动这么做：

  ```\
  deepdive create table foo
  ```

  除了上面这种简单的创建，还能具体定义表中每一列的类型，或者可以参考别的数据表来新建，还能新建视图，如果名字冲突的话，会删掉以前的。

  按照给定的列类型新建表：

  ```
  deepdive create table foo x:INT y:TEXT ...
  ```

  参考别的数据表新建：

  ```
  deepdive create table foo like bar
  ```

  通过查询新建表：

  ```
  deepdive create table foo as 'SELECT ...'
  ```

  同过查询新建视图：

  ```
  deepdive create view bar as 'SELECT ...'
  ```

  如果你想名字冲突时不删掉以前的，可以按照下面这样做，如果表名不存在才新建：

  ```
  deepdive create table-if-not-exists foo ...
  ```

##### 2.2.2.3 组织输入数据

就像我们前面所说的，所有的输入数据都要放到`input/`文件夹下，DeepDive依赖一个基于名字的获取方式，比如`app.ddlog`中定义一个`foo`关系，而且存在一个文件`input/foo.extension`，那么他就会自动载入数据，而这个`extension`扩展名可以是以下格式中的一种：

​	 `tsv`, `csv`, `sql` `tsv.bz2`, `csv.bz2`, `tsv.gz`, `csv.gz`, `tsv.sh`, `csv.sh`. 

这里的扩展名说明了用什么格式来处理数据，而比如`tsv.sh`这种格式的文件，是一个可运行的脚本，它输出的每一行都是用TAB分隔的，DeepDive会执行它来获取输入数据。

##### 2.2.2.4 从数据库中取出或者输入数据

- 数据载入数据库

  为某个关系载入数据，可以执行下面命令:

  ```
  deepdive load foo
  ```

  为某个关系从特定的一个或几个文件中载入数据可以参考下面命令：

  ```
  deepdive load foo  source.tsv
  ```

  ```
  deepdive load foo  /tmp/source-1.tsv /data/source-2.tsv.bz2
  ```

  如果一个文件里面只包含数据表中特定列的数据，可以这样输入：

  ```
  deepdive load 'foo(x,y)'  only-x-y.csv
  ```

  ​

If the destination to load is a [variable relation](http://deepdive.stanford.edu/writing-model-ddlog#variable-relations) and no columns are explicitly specified, then the sources are expected to provide an extra column at the right end that corresponds to each row's supervision label.

- 从数据库导出产生的数据

  从一个关系表中把数据导出到文件，可以执行下面的命令：

  ```
  deepdive unload bar  bar-1.tsv /data/bar-2.csv.bz2
  ```

##### 2.2.2.5 对数据库进行查询

- 运行DDlog的查询（这个是DeepDive自己定的语法）

  使用`deepdive query`可以方便的运行查询，来查看DDlog中定义的数据库。一个完整的DDlog查询以一个表达式列表开始（也可以没有，这样会显示全部匹配数据），以`?-`分隔表达式和被查询的数据表

  想要查询至少要现有数据，所以请在DeepDive执行过后才进行查询。

  **查询要使用单引号包裹起来，注意其中要有一个英文句号结束**


- 值查询

  为了浏览某个关系中的值，可以在关系想查询的列的位置放置变量，其他不想关注的地方用占位符`_`代替，就可以进行查询，比如下面的例子：

  ```
  deepdive query '?- transaction_candidate(_, name1, _, name2).'
  ```

- 查询中的选择、连接、投影

  其实这里讲的就是，同时在多个关系的变量中满足某种特定条件的查询表达。比如下面这个查询就表示，查询一对夫妻，其中的一个人的相关文档中出现了“总统”这个词

  ```
  deepdive query '
      name1, name2 ?-
          spouse_candidate(p,name1,_,name2),
          person_mention(p,_,doc,_,_,_),
          articles(doc, content),
          content LIKE "%President%".
      '
  ```

- 聚合

  就像SQL查询一样，这里同样支持聚合函数`COUNT`、`SUM`等等，使用方法如下：`COUNT(1)`是用来统计符合规则的三元组的数量

  ```
  deepdive query 'COUNT(1) ?- articles(_, content), content LIKE "%President%".'
  ```

- 分组

  如果在表达式中使用了上面提到的聚合函数，那么查询的结果就会自动按照表达式中其他项的值进行分组，比如下面的查询就会按照`doc`进行分组：

  ```
  deepdive query '
      doc, COUNT(1) ?-
          spouse_candidate(p,_,_,_),
          person_mention(p,_,doc,_,_,_),
          articles(doc, content).
      '
  ```


- 排序

  排序的语法，就和SQL不一样了，这里对要排序的属性前面加上`@order_by(...)`标记。比如下面的查询：

  ```
  deepdive query '
      doc, @order_by("DESC") COUNT(1) ?-
          spouse_candidate(p,_,_,_),
          person_mention(p,_,doc,_,_,_),
          articles(doc, content).
      '
  ```

  `@order_by`可以接受两个参数，其语法如下

  ```
  @order_by("ASC"|"DESC"[,priority])
  ```

  其中`"ASC"`和`"DESC"`分别代表升序和降序，可选的整数`priority`确定了多个排序各自的优先级，其值越小则优先级越高。比如这几个例子：`@order_by("DESC", -1)`, `@order_by("DESC")`, `@order_by(priority=-1)` 

- 个数限制（取前k个）

  在查询表达式之后，`?-`符号之前插入`| number`来设置要显示的数量限制。比如下面只显示前十的`doc`

  deepdive query '

      doc, @order_by("DESC") COUNT(1) | 10 ?-
          spouse_candidate(p,_,_,_),
          person_mention(p,_,doc,_,_,_),
          articles(doc, content).
      '

- 使用多条规则

  有时候在我们的复杂查询里，一条规则的表达能力是不够的，所以DeepDive支持在查询中定义临时规则，比如下面的例子，规则的语法和前面讲的推理规则相同，定义在查询之前，同样也包含在单引号中。

  ```
  deepdive query '
      num_candidates_by_doc(doc, COUNT(1)) :-
          spouse_candidate(p,_,_,_),
          person_mention(p,_,doc,_,_,_),
          articles(doc, content).

      @order_by num_candidates, COUNT(1) ?- num_candidates_by_doc(doc, num_candidates).
      '
  ```


- 保存结果

  在查询的最后面（单引号的外面）加入一个额外的部分

  ```
  format=(tsv|csv) > filename.(tsv|csv)
  ```

  上面语法表示可以选择tsv或者csv格式来保存结果到文件，看下面例子：

  ```
  deepdive query '
      doc,name1,name2 ?-
          spouse_candidate(p1,name1,_,name2),
          person_mention(p1,_,doc,_,_,_).
      ' format=csv  >candidates-docs.csv
  ```

- 查看查询的SQL形式

  在`deepdive query`后面加上参数`-n`，就可以查看后面查询的SQL形式，从这里我们可以知道，虽然自定义了查询语法，但是DeepDive实际上还是使用SQL进行数据库查询，只是简化了其形式。比如下面：

  ```
  deepdive query -n '?- transaction_candidate(_, name1, _, name2).'
  ```

  其返回的结果为：

  ```sql
  SELECT R0.p1_name AS "name1", R0.p2_name AS "name2"
  FROM transaction_candidate R0
  ```

- 运行SQL查询

  运行SQL查询有两种方式，第一种是打开sql的命令行，进行操作，另一种是直接执行查询

  - 第一种

    ```
    deepdive sql
    ```

    执行这条命令，可以打开当前DeepDive工程的数据库SQL命令行。

  - 第二种

    在参数中给出SQL查询语句，直接查询：

    ```
    deepdive sql "SELECT doc_id, COUNT(*) FROM sentences GROUP BY doc_id"
    ```

  - 保存查询结果

    SQL方式同样支持两种格式（TSV，CSV），使用如下命令操作：

    ```
    deepdive sql eval "SELECT doc_id, COUNT(*) FROM sentences GROUP BY doc_id" format=tsv

    deepdive sql eval "SELECT doc_id, COUNT(*) FROM sentences GROUP BY doc_id" format=csv header=1
    ```


#### 2.2.3 执行处理操作

在执行完前面说的编译操作后，被编译好的处理过程可以很灵活地执行。DeepDive提供了一系列命令来对执行进行精确地控制，下面这些都是可以同过DeepDive地命令做到地事情：

- 执行整个数据流的操作
- 执行数据流中的一部分操作.
- 在任何一个阶段停止运行，然后再某一时刻恢复执行
- 使用不同参数重复执行某个过程
- 同过读入外部数据来跳过时空代价特别大的过程

##### 2.2.3.1 完整地执行整个过程

从头到尾完整地运行DeepDive应用，编译后使用下面命令就行：

```
deepdive run
```

这条命令有如下功能:

1. 按照之前地配置初始化应用.
2. 运行所有和规则、关系相关地过程，并且适当时候运行用户定义程序.
3. 按照之前统计模型的推理规则构建基础的因子图
4. 运行权重学习和推理，来计算每个变量的边缘概率
5. 为调试程序生成校准曲线和数据.

##### 2.2.3.2 部分执行的情况

部分执行的情况经常出现在我们的开发中，因为没有什么东西是一蹴而就的，我们的应用也是一步一步构建的。接下来这一节的命令都可以在DeepDive项目文件夹中运行，如果你想得到关于命令的帮助，只需要加一个`help`参数就可以，比如想知道`deepdive do`的用法，可以执行下面命令：

```
deepdive help do
```

- 部分执行

  只运行数据流中定义的一小部分是最常见的情况，下面的命令可以只执行完`TARGET`就停止运行，不在执行后面的步骤：

  ```shell
  deepdive do TARGET...
  #Exapmle
  deepdive do articles
  ```

  执行完上面的命令后，将会在一个文本编辑器（类似vi)中显示接下来要进行的处理工作，直接键入`:wq`可以保存并退出编辑器，接下来就会自动执行真正的处理过程。

  执行下面命令可以显示当前可用的`TARGET`

  ```
  deepdive do
  ```

  ​下一节关于`TARGET`名字会有更详细的介绍

- 终止和恢复

  在运行程序的任何时候，都可以使用快捷键`Ctrl-C`来终止当前的进程，当进程发生错误的时候，也会停止。

  当再次运行DeepDive中的过程的时候，会继续从上次已经完成的部分继续运行：

  ```
  deepdive do TARGET...
  ```

- 重复

  重复执行数据流的某部分也是很常见的一种情况，DeepDive运行用户将某个过程标记为**未执行**来让他们可以被重复执行，比如下面的例子，就是重复的执行：

  ```
  deepdive mark todo init/app calibration weights
  ```

  ```
  deepdive do        init/app calibration weights
  ```

  当然，DeepDive提供了一种更简单的方法，就是我们在第一部分提到过的`deepdive redo TARGET...`来重复执行某一部分，比如：

  ```
  deepdive redo init/app calibration weights
  ```

- 跳过

  跳过某个处理过程并从外部数据导入也是一个很容易就能实现的操作，这尤其适合遇到需要很长很长时间才能执行完的过程。比如，这里有个关系`bar_derived_from_foo`是从关系`foo`中抽取得到的，而`foo`需要很长实际的运算才能得到，而且我们已经有一个外部文件包含了`foo`的内容，那么现在就不用运行`foo`了，直接按照下面的命令操作就可以：

  ```
  deepdive create table foo

  deepdive load foo /some/data/source.tsv

  deepdive mark new foo

  deepdive do bar_derived_from_foo
  ```

  > 之前不知道这个方法的时候，都是同过直接写程序操作数据库，或者是定义一个UDF来读取外部文件写入数据库。事实证明，DeepDive想的还是很周到的。

##### 2.2.3.3 编译好的处理和数据流

为了更好的运行，DeepDive之前在编译的时候生成了一个以**data**,**model**,**process**为节点和边的图，边表示了节点直接的依赖关系。

- 数据节点要和关系或者数据库中的表相关.
- 模型的节点都代表了统计学习和推理的人工的部分.
- 处理节点表示了一个计算单元.
- 连接处理节点和数据（或者模型）节点的边表示这个处理接受这个节点的输入（或输出到一个节点）.
- 连接两个处理节点的边表示其中一个依赖另一个的输出（相当于有个隐含的数据节点，作为中介）.

不论哪个应用，DeepDive都会向数据流图中编译几个内建的处理过程，这些是必要的初始化、统计学习推理和校准相关的处理。除此之外，就都是我们之前在`app.ddlog`中定义的处理了。DeepDive应用中，除了可以配置`app.ddlog`还可以配置`deepdive.conf`,但是后者已经不推荐使用了，其功能和前者相似，感兴趣的读者可以参考[extractors in deepdive.conf](http://deepdive.stanford.edu/configuration#extraction-and-extractors). 

下面这个连接是官方教程：[tutorial example](http://deepdive.stanford.edu/example-spouse)的数据流图，[数据流图，点击查看](http://deepdive.stanford.edu/images/spouse/dataflow.svg)。你也可以在你的DeepDive项目目录中的 `run/dataflow.svg`在浏览器中打开查看你的数据流图。

- 执行的计划（Plan）

  DeepDive同过编译好的数据流图来找到各个过程的正确执行顺序，图中的每个节点（或一组节点）都可以作为前面所说的`TARGET`来执行，之后DeepDive就会列举出来每个需要执行的过程的执行计划，官方叫做*execution plan* 

  - deepdive计划deepdive plan

    执行计划，可以使用`deepdive plan`命令来查询，比如下面例子，就是查询`/data/sentences`的计划

    ```
    deepdive plan data/sentences
    ```

    DeepDive的输出就像是下面，可能一些小细节不同:

    ```
    # execution plan for sentences

    : ## process/init/app ##########################################################
    : # Done: 2016-02-01T20:56:50-0800 (2d 23h 55m 9s ago)
    : process/init/app/run.sh
    : mark_done process/init/app
    : ##############################################################################
    :
    : ## process/init/relation/articles ############################################
    : # Done: 2016-02-01T20:56:55-0800 (2d 23h 55m 4s ago)
    : process/init/relation/articles/run.sh
    : mark_done process/init/relation/articles
    : ##############################################################################
    :
    : ## data/articles #############################################################
    : # Done: 2016-02-01T20:56:55-0800 (2d 23h 55m 4s ago)
    : # no-op
    : mark_done data/articles
    : ##############################################################################

    ## process/ext_sentences_by_nlp_markup #######################################
    # Done: N/A
    process/ext_sentences_by_nlp_markup/run.sh
    mark_done process/ext_sentences_by_nlp_markup
    ##############################################################################

    ## data/sentences ############################################################
    # Done: N/A
    # no-op
    mark_done data/sentences
    ##############################################################################

    ```

    一个执行计划本质上就是一个`Shell`脚本，他会调用相应的处理程序，在计划中的注释里，如果某个任务已经完成，那么他会在`Done:`后面接上时间戳，表示在该时间已经完成过了。

  - deepdive do

    前面说了，`deepdive do`可以执行数据流图中的节点，会在文本编辑器中显示执行计划，这就说明了，我们可以修改其生成的计划，达到我们特殊的目的，虽然这个操作貌似不太常用。

    ```
    deepdive do data/sentences
    ```

- 运行的时间戳

  如果你仔细看上面列出来的计划就会发现，在每个过程执行完成后，都会有执行一个`mark_done`命令，它可以把对应的处理过程标记为**已完成**的状态。这个命令实际上是修改`run/`文件夹中的`*process/name*.done`文件来标记某个过程的状态是完成了还是没完成。

  - 下面是DeepDive处理过程的几种状态标记

    DeepDive提供了一个`deepdive mark`命令来标记过程的时间戳，以方便我们可以重复执行或者跳过某一个任务，有五个可用的标记： 

    - `done`

      如果强制标记为`done`，指定的处理任务及其依赖的所有任务，都可以被跳过.

    - `todo-from-scratch`

      指定的任务及其之前的任务，都要被重新执行。

    - `todo`

      指定的任务及其之后的任务，都可以重新执行。

    - `new`

      指定的任务之后的所有任务，都可以重新执行（不包括指定的）

    - `all-new`

      依赖与指定节点及指定节点的祖先节点，都可以重复执行，但是不包括被指定的节点 

    例如，我想重新执行某些任务:

    ```
    deepdive mark todo process/init/relation/articles

    ```

    下面看看`deepdive plan`返回的结果，用来重新执行：

    ```
    # execution plan for sentences

    : ## process/init/app ##########################################################
    : # Done: 2016-02-01T20:56:50-0800 (2d 23h 55m 39s ago)
    : process/init/app/run.sh
    : mark_done process/init/app
    : ##############################################################################

    ## process/init/relation/articles ############################################
    # Done: 2016-02-01T20:56:55-0800 (2d 23h 55m 34s ago)
    process/init/relation/articles/run.sh
    mark_done process/init/relation/articles
    ##############################################################################

    ## data/articles #############################################################
    # Done: 2016-02-01T20:56:55-0800 (2d 23h 55m 34s ago)
    # no-op
    mark_done data/articles
    ##############################################################################

    ## process/ext_sentences_by_nlp_markup #######################################
    # Done: N/A
    process/ext_sentences_by_nlp_markup/run.sh
    mark_done process/ext_sentences_by_nlp_markup
    ##############################################################################

    ## data/sentences ############################################################
    # Done: N/A
    # no-op
    mark_done data/sentences
    ##############################################################################
    ```


    ​```

- 内建的数据流节点

  DeepDive在编译的时候会加入几个内建的处理过程（前面说过了），这些过程主要用于辅助其他处理的执行，有下面几个：

  - `process/init/app`

    这个过程初始话数据库，如果存在脚本`input/init.sh`的话，也会执行它。这个脚本主要是用来确保输入数据、代码、库等内容都下载好了。

    所有不依赖于其他过程的处理，都会自动依赖这个初始化过程，这样就可以保证，所有的处理都是在初始化完成之后进行。

  - `process/init/relation/*R*`

    关系表的初始化过程，就像我们前面管理输入数据的部分说过的。这些过程都会自动创建相应的关系表。

  - `process/grounding/*`

    这个空间下的所有处理都是用来**构建因子图**的。

  - `process/model/*`

    这个空间下的所有处理**学习和推理任务**的。

  - `model/*`

    所有人为连接到统计推理模型，却又不属于数据库的都放在了这里：

    - `model/factorgraph`

      指出`run/model/factorgraph/`下的二进制因子图文件

    - `model/weights`

      指出`run/model/weights`中的包含学习的到模型权重的文本文件

    - `model/probabilities`

      指出`run/model/probabilities`存储计算出的边缘概率的文件

    - `model/calibration-plots`

      指出`/run/model/calibration-plots/`中的校准图和数据文件

  - `data/model/*`

    和统计推理相关的并且要输入到数据库中的数据处理都在这里。比如： `data/model/weights`and `data/model/probabilities` 存储统计和学习推理的结果。

##### 2.2.3.4 环境变量

有几个**系统环境变量**可以影响到用户定义程序（UDFs）的执行：

- `DEEPDIVE_NUM_PROCESSES`

  这个环境变量可以控制并行运行UDF进程的数量，默认值是处理器数量减一，最小值是1。

- `DEEPDIVE_NUM_PARALLEL_UNLOADS` and `DEEPDIVE_NUM_PARALLEL_LOADS`

  控制输出数据和载入数据进程的并行数量，默认是1.

- `DEEPDIVE_AUTOCOMPILE`

  这个变量控制`deepdive do `命令是否在源文件（`app.ddlog`，`deepdive.conf`）发生改变时自动执行编译。默认值是`DEEPDIVE_AUTOCOMPILE=true`，这样就可以减少忘记`deepdive compile`的情况。当然也可以修改为`false`，省的你不想编译的时候就意外编译了。

- `DEEPDIVE_INTERACTIVE`

  控制是否`deepdive do`命令需要和用户交互，也就是说在真正执行任务之前是不是需要人工检查**plan**。默认值是`false`，在源文件变化时不会询问是否要重新编译，也不会要求检查和编辑执行计划。

- `DEEPDIVE_PLAN_EDIT`

  控制是否要听过修改**plan**的机会（需要在交互模式）

- `VISUAL` and `EDITOR`

  选择使用哪一个文本编辑器（比如显示*plan* )，默认值是`vi`

#### 2.2.4 基于概率模型进行学习和推理

每个DeepDive应用，执行前面定义任何一个处理过程都是为了后面构建用来推理的统计模型提供数据。正好DeepDive提供了几个命令简化统计模型上的操作，包括：**创建(grounding)**、**参数估计(learning)**、**计算概率(inference)**、**模型参数的再利用**

##### 2.2.4.1 获取推理结果

可以使用下面的命令得到推理结果，一般都是我们在DDlog中定义的随机变量的边缘概率：

```
deepdive do probabilities
```

这条命令将会执行所有必须的数据处理过程，之后创建统计模型并且在其上学习并推理，最后将每个变量的概率存入数据库。

##### 2.2.4.2 检查推理结果

为了方便用户查看推理结果，DeepDive创建了一个数据库**视图**来关联每个抽取到的关系及其概率，比如下面的SQL查询就是用来检查`has_transaction`的关系变量的概率：

```
deepdive sql "SELECT * FROM has_transaction_inference"
```

其显示的数据表就像下面类似：

```
                      p1_id                       |                      p2_id                       | expectation
--------------------------------------------------+--------------------------------------------------+-------------
 7b29861d-746b-450e-b9e5-52db4d17b15e_4_5_5       | 7b29861d-746b-450e-b9e5-52db4d17b15e_4_0_0       |       0.988
 ca1debc9-1685-4555-8eaf-1a74e8d10fcc_7_25_25     | ca1debc9-1685-4555-8eaf-1a74e8d10fcc_7_30_31     |       0.972
 34fdb082-a6ef-4b54-bd17-6f8f68acb4a4_15_28_28    | 34fdb082-a6ef-4b54-bd17-6f8f68acb4a4_15_23_23    |       0.968
 7b29861d-746b-450e-b9e5-52db4d17b15e_4_0_0       | 7b29861d-746b-450e-b9e5-52db4d17b15e_4_5_5       |       0.957
 a482785f-7930-427a-931f-851936cd9bb1_2_34_35     | a482785f-7930-427a-931f-851936cd9bb1_2_18_19     |       0.955
 a482785f-7930-427a-931f-851936cd9bb1_2_18_19     | a482785f-7930-427a-931f-851936cd9bb1_2_34_35     |       0.955
 93d8795b-3dc6-43b9-b728-a1d27bd577af_5_7_7       | 93d8795b-3dc6-43b9-b728-a1d27bd577af_5_11_13     |       0.949
 e6530c2c-4a58-4076-93bd-71b64169dad1_2_11_11     | e6530c2c-4a58-4076-93bd-71b64169dad1_2_5_6       |       0.946
 5beb863f-26b1-4c2f-ba64-0c3e93e72162_17_35_35    | 5beb863f-26b1-4c2f-ba64-0c3e93e72162_17_29_30    |       0.944
 93d8795b-3dc6-43b9-b728-a1d27bd577af_3_5_5       | 93d8795b-3dc6-43b9-b728-a1d27bd577af_3_0_0       |        0.94
 216c89a9-2088-4a78-903d-6daa32b1bf41_13_42_43    | 216c89a9-2088-4a78-903d-6daa32b1bf41_13_59_59    |       0.939
 c3eafd8d-76fd-4083-be47-ef5d893aeb9c_2_13_14     | c3eafd8d-76fd-4083-be47-ef5d893aeb9c_2_22_22     |       0.938
 70584b94-57f1-4c8c-8dd7-6ed2afb83031_20_6_6      | 70584b94-57f1-4c8c-8dd7-6ed2afb83031_20_1_2      |       0.938
 ac937bee-ab90-415b-b917-0442b88a9b87_5_7_7       | ac937bee-ab90-415b-b917-0442b88a9b87_5_10_10     |       0.934
 942c1581-bbc0-48ac-bbef-3f0318b95d28_2_35_36     | 942c1581-bbc0-48ac-bbef-3f0318b95d28_2_18_19     |       0.934
 ec0dfe82-30b0-4017-8c33-258e2b2d7e35_36_29_29    | ec0dfe82-30b0-4017-8c33-258e2b2d7e35_36_33_34    |       0.933
 74586dd9-55af-4bb4-9a95-485d5cef20d7_34_8_8      | 74586dd9-55af-4bb4-9a95-485d5cef20d7_34_3_4      |       0.933
 70bebfae-c258-4e9b-8271-90e373cc317e_4_14_14     | 70bebfae-c258-4e9b-8271-90e373cc317e_4_5_5       |       0.933
 ca1debc9-1685-4555-8eaf-1a74e8d10fcc_7_30_31     | ca1debc9-1685-4555-8eaf-1a74e8d10fcc_7_25_25     |       0.928
 ec0dfe82-30b0-4017-8c33-258e2b2d7e35_36_15_15    | ec0dfe82-30b0-4017-8c33-258e2b2d7e35_36_33_34    |       0.927
 f49af9ca-609a-4bdf-baf8-d8ddd6dd4628_4_20_21     | f49af9ca-609a-4bdf-baf8-d8ddd6dd4628_4_15_16     |       0.923
 ec0dfe82-30b0-4017-8c33-258e2b2d7e35_16_9_9      | ec0dfe82-30b0-4017-8c33-258e2b2d7e35_16_4_5      |       0.923
 93d8795b-3dc6-43b9-b728-a1d27bd577af_3_23_23     | 93d8795b-3dc6-43b9-b728-a1d27bd577af_3_0_0       |       0.921
 5530e6a9-2f90-4f5b-bd1b-2d921ef694ef_2_18_18     | 5530e6a9-2f90-4f5b-bd1b-2d921ef694ef_2_10_11     |       0.918
[...]
```

如何使用推理结果进行系统的调校和修改，等下一部分会讲到更多。

下面会更详细地DeepDive提供地统计模型地操作命令。

##### 2.2.4.3 因子图的构建

之前在DDlog中定义的推理规则，实际上在DeepDive中就是一个用来执行统计推理的数据结构，这个数据结构就叫做**因子图**。而构建（Grounding）就是将因子图这个数据结构按照特定有的[格式](http://deepdive.stanford.edu/factor_graph_schema)固化在一系列文件中的过程，可以在终端中使用下面命令来启动这个过程：

```
deepdive model ground
```

上面的命令可以看作是一个内建处理的快捷执行方式：

```
deepdive redo process/grounding/variable_assign_id process/grounding/combine_factorgraph
```

这个过程会在`run/model/grounding/`目录下为每个变量和因子都创建一系列文件，他们之后会合并到`run/model/factorgraph/`目录下的统一的因子图中，之后会在其基础上使用**DimmWitted推理引擎**来进行学习和推理。比如下面的例子中显示出来的就是一个构建的因子图的文件列表：

```
find run/model/grounding -type f
```

```
run/model/grounding/factor/inf_imply_has_spouse_has_spouse/factors.part-1.bin.bz2
run/model/grounding/factor/inf_imply_has_spouse_has_spouse/nedges.part-1
run/model/grounding/factor/inf_imply_has_spouse_has_spouse/nfactors.part-1
run/model/grounding/factor/inf_imply_has_spouse_has_spouse/weights.part-1.bin.bz2
run/model/grounding/factor/inf_imply_has_spouse_has_spouse/weights_count
run/model/grounding/factor/inf_imply_has_spouse_has_spouse/weights_id_begin
run/model/grounding/factor/inf_imply_has_spouse_has_spouse/weights_id_exclude_end
run/model/grounding/factor/inf_istrue_has_spouse/factors.part-1.bin.bz2
run/model/grounding/factor/inf_istrue_has_spouse/nedges.part-1
run/model/grounding/factor/inf_istrue_has_spouse/nfactors.part-1
run/model/grounding/factor/inf_istrue_has_spouse/weights.part-1.bin.bz2
run/model/grounding/factor/inf_istrue_has_spouse/weights_count
run/model/grounding/factor/inf_istrue_has_spouse/weights_id_begin
run/model/grounding/factor/inf_istrue_has_spouse/weights_id_exclude_end
run/model/grounding/factor/weights_count
run/model/grounding/variable/has_spouse/count
run/model/grounding/variable/has_spouse/id_begin
run/model/grounding/variable/has_spouse/id_exclude_end
run/model/grounding/variable/has_spouse/variables.part-1.bin.bz2
run/model/grounding/variable_count
```

##### 2.2.4.4 权重学习

DeepDive会在前面建立的因子图上学习权重，比如从弱监督数据打标过的变量中学习统计模型的参数的最大似然估计值。**DimmWitted推理引擎**使用随机梯度下降的**吉布斯采样**来学习权重。

下面的命令使用前面的因子图执行学习（如果前面没有建立，会自己建立）

```
deepdive model learn
```

它和下面这条命令实际上是相同的：

```
deepdive redo process/model/learning data/model/weights
```

**DimmWitted**把学习的的权重存储在`run/model/weights/`目录下的文件中，为了方便起见，DeepDive会把学习的的权重载入数据库并且新建几个视图以供下面命令使用：

```
deepdive do data/model/weights
```

这将会创建一个权重的综合的视图，叫做： `dd_inference_result_weights_mapping`. 有了这个视图，就可以很容易得到每个推理规则和它们的参数值，下面是一个简单的示例：

```
deepdive sql "SELECT * FROM dd_inference_result_weights_mapping"
```

```
    weight    |                      description
--------------+---------------------------------------------------------------
      1.80754 | inf_istrue_has_spouse--INV_NGRAM_1_[wife]
      1.45959 | inf_istrue_has_spouse--NGRAM_1_[wife]
     -1.33618 | inf_istrue_has_spouse--STARTS_WITH_CAPITAL_[True_True]
      1.30884 | inf_istrue_has_spouse--INV_NGRAM_1_[husband]
      1.22097 | inf_istrue_has_spouse--NGRAM_1_[husband]
     -1.00449 | inf_istrue_has_spouse--W_NER_L_1_R_1_[O]_[O]
     -1.00062 | inf_istrue_has_spouse--NGRAM_1_[,]
           -1 | inf_imply_has_spouse_has_spouse-
     -0.94185 | inf_istrue_has_spouse--IS_INVERTED
     -0.91561 | inf_istrue_has_spouse--INV_STARTS_WITH_CAPITAL_[True_True]
     0.896492 | inf_istrue_has_spouse--NGRAM_2_[he wife]
     0.835013 | inf_istrue_has_spouse--INV_NGRAM_1_[he]
    -0.825314 | inf_istrue_has_spouse--NGRAM_1_[and]
     0.805815 | inf_istrue_has_spouse--INV_NGRAM_2_[he wife]
    -0.781846 | inf_istrue_has_spouse--INV_W_NER_L_1_R_1_[O]_[O]
      0.75984 | inf_istrue_has_spouse--NGRAM_1_[he]
     -0.74405 | inf_istrue_has_spouse--INV_NGRAM_1_[and]
     0.701149 | inf_istrue_has_spouse--INV_NGRAM_1_[she]
    -0.645765 | inf_istrue_has_spouse--INV_NGRAM_1_[,]
       0.6105 | inf_istrue_has_spouse--INV_NGRAM_2_[husband ,]
     0.585621 | inf_istrue_has_spouse--INV_NGRAM_2_[she husband]
     0.583075 | inf_istrue_has_spouse--INV_NGRAM_2_[and he]
     0.581042 | inf_istrue_has_spouse--NGRAM_1_[she]
     0.540534 | inf_istrue_has_spouse--NGRAM_2_[husband ,]
[...]
```

##### 2.2.4.5 推理

在学习了权重之后，DeepDive会利用他们来计算每个变量的边缘概率。实际上使用的**DimmWitted**的吉布斯采样来近似计算每个变量在*可能的世界* 中每个可能取值的概率，可以使用如下代码执行：

```
deepdive model infer

```

其实和下面的这个数据流处理时相同的：

```
deepdive redo process/model/inference data/model/probabilities

```

实际上，如果分开执行学习和推理过程，会重复地把因子图读入内存，所以**DimmWitted**会在学习后执行推理，所以，除非要重用学习到的权重，不然你就可以略过推理的部分，正常来说执行这个命令会没有什么实际用处。

**DimmWitted**把推理到的概率存储在`run/model/proabilities/`目录中的文本文件中。就像前面刚刚说的一样，DeepDive会把计算出的概率存到数据库中，并创建对应的视图。

- 重用权重

  最常见的重用权重的情况时在一个数据集上学习权重，并且在另一个数据集上进行推理，就想传统机器学习的在训练集上训练模型，在另一个新的数据集上进行测试。

  这一部分就可以解决每次运行DeepDive需要时间特别长，只要把训练好的权重和因子图保存下来就可以了。

  1. 在一个小数据集上学习权重.
  2. 保存学习的的权重.
  3. 在大数据集上重用刚刚保存的权重.
  
  DeepDive提供了几个命令来管理和重用权重，如下：

  - 保存权重

  保存当前情况下的权重以便将来使用，可以使用下面的命令，给定一个名字来存储：

  ```
  deepdive model weights keep FOO

  ```

  执行后就会把数据库中的权重保存到`snapshot/model/weights/FOO/`目录中，然后我们就可以在后面的重用部分用到。这里名字`FOO`是可以不写的，不写的话就会用当前的时间戳来代替名字。

  - 重用权重

  给定前面设置的名字或者时间戳，执行下面的命令就可以重用权重:

  ```
  deepdive model weights reuse FOO

  ```

  上面命令把位于 `snapshot/model/weights/FOO/`目录的数据导入到数据库中，然后重新执行必要将权重导入因子图的构建过程。这里的名字同样可以省略，DeepDive会使用最新保存的数据。

  后续的命令就可以不再学习而直接进行推理的步骤.

  ```
  deepdive model infer

  ```

  - 管理权重

  DeepDive还提供了一些命令来管理保存下来的权重数据。

  列出所有保存的权重:

  ```
  deepdive model weights list

  ```

  删除一个组指定的权重:

  ```
  deepdive model weights drop FOO

  ```

  删除所有之前载入的权重，以便重新学习:

  ```
  deepdive model weights init
  ```

### 2.3 评估和调试

#### 2.3.1 调试用户定义函数（UDF）

写代码的时候不可能毫无错误，那么鉴于之前UDF定义的格式，如何才能方便地调试呢？肯定不是把代码粘贴出来单独执行啦，而且有的时候DeepDive使用的环境和我们系统默认环境是不一样的。DeepDive用户定义函数的语言可以是任何一种使用标准输入输出的可执行语言。下面就说说两个方面：首先是在屏幕上输出信息，另外就是不进入项目的数据流而单独调试UDF。

##### 2.3.1.1 显示日志信息

**请注意**DeepDive的UDF的所有标准输出都被转换为`TSJ`或者`TSV`格式并存到数据库里，所以像Python中的`print`这样的语句是不可以作为我们的屏幕信息输出的，这样会破坏我们的数据。所以正确的用法是把屏幕信息使用标准错误流`standard error`中进行输出，比如下面一个Python语言的例子，其他语言也类似：

```
#!/usr/bin/env python
from deepdive import *
import sys

@tsj_extractor
@returns( ... )
def extract( ... ):
    ...
    print >>sys.stderr, 'This prints some_object to logs :', some_object
    ...

```

在脚本的执行过程中，所有写到标准错误流中的信息，都会显示在屏幕上，并且存入`run/LATEST/run.log`文件中，作为执行日志。

##### 2.3.1.2 在DeepDive环境中运行UDF

为了辅助调试UDFs中的错误，DeepDive提供了一个包装命令来在实际的运行环境中执行UDF。

比如说，我们有一个Python脚本：`udf/fn.py`，其中导入了DeepDive的`deepdive.ddlib`包，如果我们直接在系统中执行这个脚本，就会报错，因为找不到这个`deepdive`模块。

```
python udf/fn.py

```

```
Traceback (most recent call last):
  File "udf/fn.py", line 2, in <module>
    from deepdive import *
ImportError: No module named deepdive

```

为了避免这个问题，调试程序的时候请使用`deepdive env`命令，这样执行脚本就好像在DeepDive工作流中执行似的，会载入对应的库。

```
deepdive env python udf/fn.py

```

这样执行的话，程序会接受`TSJ`格式每一行标准输入，并在标准输出中，输出对应格式的`TSJ`行，当然也会显示错误信息（包括要显示的Log信息），此时就像调试一个普通的Python程序一样了。

### 2.3.2 Calibration调校

DeepDive的一个重要特性就是他的迭代是工作流，在执行了**概率推理**过程后，评估结果并且反馈到系统中来提高准确率是十分重要的，所以DeepDive提供了**校准图**来帮助我们完成这个过程。

#### 定义一个Holdout集 Defining a holdout set

为了最大限度地利用校准功能，用户需要定义一个**Holdout比例（holdout fraction)**或者一个用户定义的**Holdout查询**来把一部分的证明数据作为训练数据。DeepDive使用Holdout变量来估计其准确率，默认是不是用的Holdout的。

##### Holdout 比例

当在`deepdive.conf`文件中定义了`holdout_fraction`，DeepDive会在证据数据（已经被打标的数据）中随机抽取指定比例的变量作为训练数据。其定义如下：

```
deepdive.calibration.holdout_fraction: 0.25

```

##### 自定义 Holdout 查询

DeepDive也提供了使用SQL查询定义Holdout数据的功能，要将使用的数据插入的`dd_graph_variables_holdout`表中，而且必须插入`dd_id`列。

比如查询定义的Holdout可以在`deepdive.conf`中这样定义：

```
deepdive.calibration.holdout_query: """
    INSERT INTO dd_graph_variables_holdout(variable_id)
    SELECT dd_id
    FROM mytable
    WHERE predicate
"""
```

如果定义了`holdout_query`的话，那么将会忽略`holdout_fraction`的定义。

#### 检查概率和权重

为了提高预测的准确性，检查每个变量的概率和节点的权重有很大作用。DeepDive在数据库中创建了叫做`dd_inference_result_weights_mapping`的视图，其中存储的是按照绝对值排序的权重和因子名字。这个视图的Schema如下：

```
View "public.dd_inference_result_weights_mapping"
   Column    |       Type       | Modifiers
-------------+------------------+-----------
 id          | bigint           |
 isfixed     | integer          |
 initvalue   | real             |
 cardinality | text             |
 description | text             |
 weight      | double precision |

```

说明:

- **id**: 每个权重的唯一标识符
- **initial_value**: 权重的初始值
- **is_fixed**: 这个权重是否是固定的，在学习过程中是否可以改变
- **cardinality**: 因子的技术. 详情可以参见[categorical factors](http://deepdive.stanford.edu/example-chunking).
- **description**: 权重的说明，其格式为：[推理规则的名字]-[在推理中设定的权重值]
- **weight**: 学习出的权重值

##### 校准的数据和图

执行下面命令，可以让DeepDive为每个在模型中定义的**变量关系**生成校准用的数据文件。

```
deepdive do model/calibration-plots

```

其在下面的位置生成了`tsv`文件，一共有十行五列，如下所示:

```
run/model/calibration-plots/[variable_name].tsv

```

```
[bucket_from] [bucket_to] [num_predictions] [num_true] [num_false]

```

DeepDive把所有的推理结果分了是个部分，每个部分都和从0.0-1.0范围的概率相关联。其后面三列我们做个详细的解释

- `num_predictions` 是这一部分所包含的变量的数量，包括不带标记的Holdout或者是查询的变量，简单来说也就是$num_predictions=num_holdout+num_unknown_var$.
- `num_true` 存储了当前部分的Holdout变量中逻辑取值为`true`的数量。当变量的概率都普遍偏高时，这个数字也会变高，而概率低的话，这个数字会变小。也就是说，当变量的概率很高的时候，会被当作`true`。还要注意的时，这里和下面属性统计的数量只包含Holdout变量。
- `num_false` 是当前部分的Holdout变量中逻辑取值为`False`的数量。当变量普遍取值偏高时，这个数字应该偏小，反之则相反。

DeepDive同样为每个变量生成了一个图像文件，他是紧接着校准数据生成出来的，也在`run/model/calibration-plots/`目录中。

```
run/model/calibration-plots/[variable_name].png

```

一个普通的校准图就像是下面这个样子，你也可以看看你的项目中的图像是什么样的：

![A calibration plot from the spouse example](http://deepdive.stanford.edu/images/spouse/has_spouse.png)

##### 如何理解校准图

下面介绍校准图文件中的三个图的意思：

- **The accuracy plot (a)** 展示了识别出正确预测的比例，同样是分了十个部分，也就是从左到右十个点。理想情况下，红色的折线应该和蓝色的虚线你和，这种情况表示我们的系统的到的结果和变量的概率值是线性相关的。而在完美的定义中，0概率的没有`true`的情况，而100%概率的全是`true`的情况。关于准确性的定义为：`num_holdout_true` / `num_holdout_total`.

- **Plots (b) and (c)** 这两幅图分别显示了在测试集和全部数据集上的预测情况。他们的理想情况都应该是最高点形成一个U形，将预测的概率两极化，这也是机器学习中的目标，可以更好的进行分类。因为在概率在0.4-0.6之间的变量，说明我们的系统也不敢确定它到底是个啥，这就需要更多的特征来对其进行分类了。

**注意**：如果你单纯地按照本教程地第一部分的例子来做，可能你会得不到这些校准数据，你还需要按照前面说的，在`deepdive.conf`文件中设置Holdout数据。

##### 校准数据的作用

根据校准数据我们可以了解当前系统的性能，并且根据其特点找到性能不好的缺陷在哪里，主要有以下几种:

- **Not enough features:** 如果你的校准图中预测的大部分都落在0.4-0.6之间，那么最有可能的就是没有足够的特征。系统根据仅有的特征很难做出判断，这时候，可以取看看概率在其中的变量和数据，检查其中是否还有你之前没有发现的正（或负）特征。
- **Not enough positive evidence:** 如果你的系统结果中，正确识别出来的关系很少的话，那么你就应该想想是不是正例太少了，导致系统觉得不能凭空给你构造大概率的结果。当然还有一种可能是，没有正确使用正例中的特征。
- **Not enough negative evidence:** 和上面的情况相反，会出现负例不足的情况，其表现为低概率的变量特别少（和实际相比），那么就是负例不够的问题了，或者是没有正确使用负例的特征。你可以使用这里的方法来生成负例，这是个技术活：[negative evidence](http://deepdive.stanford.edu/generating_negative_examples)
- **Weight learning does not converge:** 如果你的预测结果看起来是一团糟，额，查看日志文件中的梯度值是不是很大（1000+），如果是的话，那么要恭喜你了，权重训练没有收敛，解决办法就是增加迭代次数并减小学习率。
- **Weight learning converges to a local optimum:** 如果权重没有收敛到全局最优，你可以尝试增加学习率，使用慢衰减，具体看看DimmWitted采样的内容：[DimmWitted sampler documentation](http://deepdive.stanford.edu/sampler)。

#### 召回误差

召回率（Recall）在信息检索中又可以称为查全率，是检索出的相关信息量占系统中相关信息总量的比例。其误差主要来源于两个方面：

- **Event candidates are not recognized in the text.** 在这种情况下，就是没有为特定事件创建变量，而且你在校准图中根本发现不了。比如说，我们的文本中有个“冻北达学”或者“东北大 学”这类的内容，我们很难把`东北大学`这个实体识别出来，因为其中的拼写错误，英文中还存在大小写的问题。除非有一个超全的数据库，然而这不现实。
- **Events fall below a confidence cutoff.** 如果我们只对大概率事件感兴趣，那么校准图的中间部分就应该看作是召回误差。比如说，我们认为在可信度为90%以上的公司为正确的识别结果，那么在90%以下的公司的缺失，就会产生召回误差。

### 使用Mindbender浏览DeepDive数据。

这一节介绍如何简单地使用交互式的搜索界面来浏览DeepDive的输入数据以及它产生的数据。在此之前，你的DeepDive应用必须要像前面说的那样，在DDlog中写好。

目前，这个功能只支持**PostrgreSQL v9.3+**，因为它需要依赖`to_json()`方法的支持。

#### 快速入门

##### 1. 标记DDlog中的关系

首先，你要在DDlog中关系定义的地方添加一些标记才能进行浏览。标记是额外的信息，可以放在关系名前面，也可以放在列名前面，使用`@`符开始，就像：`@extraction`、`@references(relation="foo", column="bar")`。这里有个官方的DDlog的例子，你可以去看看：[传送门](https://github.com/HazyResearch/mindbender/blob/master/examples/spouse_example/app.ddlog).

1. 关系之间的联系可以使用标记`@key`、`@references`来标记其中有关系的列。比如下面的例子中表达的意思就是，`entity`关系中的`doc_id`是和`document`关系中的`id`相关联的。

   ```
   document( @key  id text, ... ).
   entity( ...,  @references(relation="documents", column="id")  doc_id text ).

   ```

2. 所有像查看的关系，必须要由 `@extraction`或者`@source`进行标记才能浏览。

   - 所有同过`@references`和`@key`联系起来的关系，在显示中会用嵌套的方式显示出来。
   - 一个`@extraction`标记的关系，应该有使用`@references`标记的列来和`@source`标记的关系相关联。

3. 所有关系中可显示的列，在这里都是可以进行全文搜索的。

4. `@searchable`标记用来表示会在搜索结果中重点显示出来。

5. `@navigable`和`@searchable`标记的列用来主要用来分面导航。

##### 2. 输入搜索的索引

在前面对`Schema`进行标记后，就要把想要浏览的数据添加到搜索的索引当中，才可以浏览。这个功能基于后面这个工具实现：[Elasticsearch](https://www.elastic.co/products/elasticsearch).

要实现现在这个步骤，在DeepDive项目目录中打开终端执行下面命令：

```
mindbender search update

```

##### 3. 启动用户界面

执行完前面要求的所有步骤后，我们就可以启动一个用户界面了，执行下面的命令

```
mindbender search gui

```

你会看到终端界面上有提示信息：

![IU35J.png](https://s1.ax2x.com/2018/04/09/IU35J.png)

当然，你可能看见的是`http://你的计算机名或者（localhost):8000`，然后在浏览器中打开这个地址，你就能看到三个按钮，选择**Search**就能看到DeepDive项目中的数据了。或者直接使用下面地址：<http://localhost:8000/#/search>

![Screenshot of DeepDive's GUI for Browsing Data](http://deepdive.stanford.edu/images/browsing_screenshot.png)

[Elasticsearch manual on query string syntax](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html#query-string-syntax),这个页面有关于当前工具的查询语法的介绍，想深入了解可以看下。

------

#### DeepDive中使用的经典模式

DeepDive应用，在DDlog的代码中的关系一般都会是下面几种关系之一：

1. 存放源数据（输入数据）的关系
   - 带有NLP标记的语料库
   - 词典，受控词表
   - 本体，知识库，已知的关系
2. 存放抽取的到的数据的关系
   - 候选实体
   - 提到的关系，实体链接
   - 特征
   - 监督的标注
3. 存放预测数据的关系（变量关系）
   - 期望DeepDive预测出来的数据

比如在前面第一部分中，我们用到的关系有:

1. Source
   - `articles`
   - `sentences`
   - `transaction_dbpedia`
2. Extraction
   - `company_mention`
   - `transaction_candidate`
   - `transaction_feature`
   - `transaction_label`
3. Prediction
   - `has_transaction`

#### DDlog中用于预览的注释

DeepDive允许关系和列可以被任意数量的`@name(arguments)`格式的标记所标记。这里的`arguments`参数是可选的，如果没有参数就要按照`@tag_name`这样来写。而其中的参数有两种书写格式：`@foo(1,"two", 3.45)`和`@bar(number=1,string="two",double=3.45)`，他们的区别就是参数值之前是否有参数名称.这里可用的类型就是*整数*、*浮点类型*、*字符串*。

##### 在DDlog中标记不同关系之间的关联

###### 用`@key`标记列

每个关系中都可以有一个或者多个列带有`@key`标记。如果关系中只有一个列标记为`@key`，那么这一列就是主键（就像数据库中似的）。当有多个列带有`@key`标记，那么这些列的组合值就作为键。

###### 用`@references(relation, [column], [alias])`标记列

如果在一个关系中的某些列是使用的另一个关系中的某个键值，那么这个列就可以是所有`@references`标记。如果目标关系中，有多个`@key`标记的话，那么这里需要添加列名这个参数了。而如果在一个关系中，有多个列同时引用了同一个关系中的同一组数据,可以通过`alias`参数给他们一个相同的别名，来表示是同一次引用的多列数据。

##### 标记为抽取或者可搜索

DDlog中标记的一个主要应用是用来搜索，Mindbender支持DeepDive项目中的所有数据的全文搜索。其后台是[Elasticsearch](https://baike.baidu.com/item/elasticsearch/3411206?fr=aladdin)引擎,DDlog中的标记指示了存储在关系数据库中的DeepDive数据怎样输出转换为Elasticsearch中的文档模型。

###### `@extraction([label])` 标记关系

存储被抽取出的数据的关系，应该带有`@extraction`标记。在搜索界面中，抽取的关系被看作是一个搜索的类型，而其中的每一组数据都看作是一个搜索结果的单位。如果有多个抽取关系之间用`@reference`标记链接，那么只要最主要的关系带有这个标记就行。比如说：`company_mention`、`has-transaction`和`transaction_feature`都是抽取关系，但是只要`person_mention`和`has_transaction`带有`@extraction`标记就行了，因为`transaction_feature`中包含了与`has_spouse`相关的额外信息。

###### `@searchable([relation, ...])` 标记列

Any column annotated with `@searchable` that's directly or indirectly associated with an `@extraction` relation (or a relation that corresponds to a searchable type) is indexed for searching and appears in the result highlighted. Optionally, names of the `@extraction` relations can be specified as arguments to limit the search indexes the column participates in.

###### `@source([label])` 标记关系

A relation that holds the input/source data to DeepDive should be annotated as `@source` with optional label. The `@source` relations themselves become a searchable type in the search interface as well. Every `@extraction` relation is supposed to record its provenance in column(s) that `@references` one of the `@source` relations. There are two reasons why `@source` relations are explicitly annotated. First, since the typically larger `@source` relations may change less frequently than the `@extraction`s derived from them as the DeepDive application evolves, we can avoid reindexing a large part of the data that stays mostly invariant. Specifically, Elasticsearch's parent-child mapping is used to decouple the frequently changing `@extraction`s from their `@source`s. Second, given how the searchable data to be indexed is determined from the annotations, `@source` relations provide natural breaking points while following the association links between relations. In other words, because independent `@extraction`s can reference a `@source` relation as its provenance, to make each `@extraction` searchable, we may end up collecting data for all other `@extraction`s from the same `@source`.

##### Annotation for aggregation / faceted search

###### `@navigable([relation, ...])` columns

Similar to `@searchable`, any column annotated with `@navigable` that's directly or indirectly associated with a relation that corresponds to a searchable type is indexed for faceted navigation. Upon every search, handful of significant terms or value ranges found for the column will appear with their approximate counts to help narrowing down the search result. For example, if `label` column is annotated as `@navigable`, every search result will show how many were distantly supervised as positive and negative examples among the result and quickly drill down to them.

##### Annotation for presentation

Under development. Use [custom templates](http://deepdive.stanford.edu/browsing#customizing-presentation) for the moment.

In addition to the `@searchable` columns, typical `@extraction`s record extra provenance details with respect to the `@source`, and special presentation of the data is often necessary for human consumption. For example, `person_mention` are text spans in sentences each of which is represented by an array of indexes to the tokens of the source sentence. A sensible presentation of such array of indexes is highlighting the corresponding words in the original sentence text with different colors rather than just showing the raw numbers. Similarly, tables, figures, images, and other data types all have a similar issue: extra columns of `@extraction`s typically hold extra detail that can be visualized in data-dependent ways.

#### Customizing presentation

##### Presentation templates

How a browsable relation is presented in the GUI can be fully customized by creating an HTML-like template under the DeepDive application. For example, relation `has_spouse` is rendered using a template at `./mindbender/search-template/has_spouse.html` when it exists. It is in fact an [AngularJS template](https://docs.angularjs.org/guide/templates) where the object to be rendered is accessible with scope variable `extraction` or `source`, depending on whether the browsable relation is an extraction or source. For extractions, `source` variable may also point to its provenance source object.

Here's an example for rendering `has_spouse` in the spouse example as highlighted text spans with two colors. The `mindtagger-word-array` and `mindtagger-highlight-words`are AngularJS directives provided by [Mindtagger, a tool for labeling data](http://deepdive.stanford.edu/labeling).

```
<a mb-search-link="{{extraction.p1.mention_text}}"><strong>{{extraction.p1.mention_text}}</strong></a>
--
<a mb-search-link="{{extraction.p2.mention_text}}"><strong>{{extraction.p2.mention_text}}</strong></a>
<br> E=<strong>{{extraction.expectation}}</strong>
<br><small class="text-muted">See all
    <a mb-search-link='feature:* "{{extraction.p1_id}}@{{extraction.p2_id}}"'>features</a>
    /
    <a mb-search-link='label:*   "{{extraction.p1_id}}@{{extraction.p2_id}}"'>labels</a>
</small>

<small class="text-muted">
    in <a mb-search-only="sentences" mb-search-link="{{extraction.p1.doc_id}} sentence_index:{{extraction.p1.sentence_index}}">sentence {{extraction.p1.sentence_index}}</a>
    in <a mb-search-only="articles"  mb-search-link="{{extraction.p1.doc_id}}">article({{extraction.p1.doc_id}})</a>:
</small>
<div mindtagger-word-array="source.tokens">
    <mindtagger-highlight-words from="extraction.p1.begin_index" to="extraction.p1.end_index" with-style="background-color: yellow;"/>
    <mindtagger-highlight-words from="extraction.p2.begin_index" to="extraction.p2.end_index" with-style="background-color: cyan;"/>
</div>

```

Without the template, the GUI would render by default a `has_spouse` in a hard-to-read format:

![`has_spouse` without a Presentation Template](http://deepdive.stanford.edu/images/browsing_without_presentation_template.png)

This is improved as shown below by creating the template above in the application:

![`has_spouse` with a proper Presentation Template](http://deepdive.stanford.edu/images/browsing_with_presentation_template.png)

##### AngularJS extensions

To define further AngularJS extensions, such as directives or filters to be used in the templates, define a `mindbender.extensions` module in [`./mindbender/extensions.coffee`](https://github.com/HazyResearch/deepdive/blob/master/examples/spouse/mindbender/extensions.coffee) or `./mindbender/extensions.js`. This can be very useful if you need to share some template fragments or scripts across many browsable relations.

```
###
   To define your AngularJS extensions for the custom templates in Mindbender,
   your own JavaScript/CoffeeScript code can be put at mindbender/extensions.js
   or mindbender/extensions.coffee file under the working directory where you
   start `mindbender gui`.  All files under the mindbender/ directory will be
   available under the same relative path, e.g., an AngularJS template stored
   in mindbender/foo.html will be available as "mindbender/foo.html".  Hence,
   you can use arbitrary number of files to define your extensions.
###

angular.module "mindbender.extensions", [
    "mindbender.search"
]

.directive "mbSearchLink", ($location, DeepDiveSearch) ->
    link: ($scope, $element, $attrs) ->
        $scope.$watch (-> $attrs.mbSearchLink), ->
            params = _.extend {}, DeepDiveSearch.params
            params.s = $attrs.mbSearchLink
            params.t = $attrs.mbSearchOnly
            params.p = 1
            $element.attr "href", "#/search/#{
                unless DeepDiveSearch.index? then ""
                else encodeURIComponent DeepDiveSearch.index}?#{(
                "#{encodeURIComponent k}=#{encodeURIComponent v ? ""}" for k,v of params
            ).join("&")}"

.directive "mbView", (DeepDiveSearch) ->
    scope:
        mbView: "="
        mbViewType: "@"
    link: ($scope, $element, $attrs) ->
        # TODO use hit and https://github.com/HazyResearch/mindbender/blob/master/gui/frontend/src/search/search.html#L92-L96
        $element.attr "href", "#/view/#{encodeURIComponent DeepDiveSearch.index}/#{
            encodeURIComponent $scope.mbViewType}/?id=#{
                encodeURIComponent $scope.mbView}#{
                    unless DeepDiveSearch.routing? then ""
                    else "&routing=#{DeepDiveSearch.routing}"
                }"



.filter "uri", () ->
    (s) -> encodeURIComponent s


```



### Labeling DeepDive data with Mindtagger

This document describes a common data labeling task one has to perform while developing DeepDive applications and introduces a graphical user interface tool dedicated for accelerating such task. In this document we use the terms annotating, marking, and tagging data interchangeably with labeling data.

To assess the performance of a DeepDive application, one typically computes precision and recall of the results produced by the DeepDive application. Let's define some terms before we move on. A *relevant* set is a set of information one wants to extact, e.g., entities or relationships. DeepDive's *positive* prediction is a set of information extracted by a DeepDive application whose assigned expectation value is greater than a certain threshold.*Precision* is a ratio of how much of the relevant set is represented in the DeepDive's positive prediction. Precision is 1 if DeepDive makes no mistakes in its positive prediction by assigning high expectations only to those that are also in the relevant set (i.e., no false positives). As DeepDive assigns high expectation values to ones that are not in the relevant set, precision decreases (i.e., more false postivies were introduced). On the other hand, *recall* is a ratio of how much of the DeepDive's positive prediction set is represented in the relevant set. Recall is 1 if DeepDive does not miss any information in the relevant set (i.e., no false negatives). Recall decreases as DeepDive either assigns low expectation to ones in the relevant set or failes to capture them during the candidate extraction steps.

Because the relevant set of a corpus is usually not known in advance, there must be a human input to inspect a sample of DeepDive's predictions as well as the corpus and to answer whether each of them is relevant or not to estimate the true value. Estimating the precision can be as simple as randomly sampling predictions with expectation higher than a certain threshold, then counting how many were also in the relevant set. However, estimating the recall accurately can be very costly since one needs to find enough number of relevant information sampled from the entire corpus.

In this document, we show how the precision estimation for a DeepDive application can be done in a systematic and convenient way using a graphical user interface.

#### Mindtagger: A tool for labeling data

*Mindtagger* is a general data annotation tool that provides a customizable, interactive graphical user interface. An annotation task takes a list of items and a template that defines how each item should be rendered and what annotations can be added as inputs.

Mindtagger provides an interactive interface for users to look at the rendered items and quickly go through each of them to manually leave annotations. These annotations collected over time can be then outputted in various forms (SQL, CSV/TSV, and JSON) to be used outside of the tool such as augmenting the ground truth of a DeepDive application. Note that Mindtagger currently does not help with the sampling part but only supports the labeling task for the precision/recall estimation. Therefore, producing the right samples for the correct estimation is a Mindtagger user's responsibility.

#### Use case 1: Measuring precision of the spouse example

In the following few steps, we explain how Mindtagger can help the user to perform a precision estimation task using the [spouse example in our tutorial](http://deepdive.stanford.edu/example-spouse). All commands and path names used in the rest of this section is consistent with the [code under `examples/spouse/`](https://github.com/HazyResearch/deepdive/blob/master/examples/spouse/).

```
cd examples/spouse/

```

##### 1.1. Prepare the data items to inspect

First, you need to grab a reasonable number of samples for the precision task. Using the SQL query shown below, you can get a hundred positive predictions with threshold 0.9 from `has_spouse` relationship with the array of words of the sentence the relationship was mentioned and two text spans mentioning the two persons involved.

```
-- labeling/has_spouse-precision/sample-has_spouse.sql file

SELECT hsi.p1_id
     , hsi.p2_id
     , s.doc_id
     , s.sentence_index
     , hsi.label
     , hsi.expectation
     , s.tokens
     , pm1.mention_text AS p1_text
     , pm1.begin_index  AS p1_start
     , pm1.end_index    AS p1_end
     , pm2.mention_text AS p2_text
     , pm2.begin_index  AS p2_start
     , pm2.end_index    AS p2_end

  FROM has_spouse_inference hsi
     , person_mention             pm1
     , person_mention             pm2
     , sentences                  s

 WHERE hsi.p1_id          = pm1.mention_id
   AND pm1.doc_id         = s.doc_id
   AND pm1.sentence_index = s.sentence_index
   AND hsi.p2_id          = pm2.mention_id
   AND pm2.doc_id         = s.doc_id
   AND pm2.sentence_index = s.sentence_index
   AND       expectation >= 0.9

 ORDER BY random()
 LIMIT 100


```

Let's keep the result of the above SQL query in a file named `has_spouse.csv`.

```
deepdive sql <"labeling/has_spouse-precision/sample-has_spouse.sql" >"has_spouse.csv"

```

The lines in the [generated `has_spouse.csv`](https://github.com/HazyResearch/deepdive/blob/master/examples/spouse/labeling/has_spouse-precision/has_spouse.csv) should look similar to the following:

```
p1_id,p2_id,doc_id,sentence_index,label,expectation,tokens,p1_text,p1_start,p1_end,p2_text,p2_start,p2_end
b5d4091a-4fb0-4afb-8d49-65e985f1be21_65_13_13,b5d4091a-4fb0-4afb-8d49-65e985f1be21_65_11_11,b5d4091a-4fb0-4afb-8d49-65e985f1be21,65,,0.994,"{Danny,was,on,the,ground,in,a,pickup,truck,"","",while,Mickey,and,Stan,flew,above,"","",issuing,instructions,on,where,to,change,the,co,se,.}",Stan,13,13,Mickey,11,11
248c8a3a-ea00-454d-b018-968e410c190f_41_16_16,248c8a3a-ea00-454d-b018-968e410c190f_41_18_18,248c8a3a-ea00-454d-b018-968e410c190f,41,,0.98,"{``,While,I,'m,as,entertained,as,anyone,by,this,personal,back-and-forth,about,the,history,of,Donald,and,Carly,'s,career,"","",'',he,said,"","",``,for,the,55-year-old,construction,worker,out,in,that,audience,tonight,who,does,n't,have,a,job,"","",who,ca,n't,fund,his,child,'s,education,--,I,got,ta,tell,you,the,truth,--,they,could,care,less,about,your,career,'',___,FIGHTING,FOR,ATTENTION,ON,A,CROWDED,STAGE,With,a,record,number,of,debate,participants,"","",they,could,n't,all,stand,out,.}",Donald,16,16,Carly,18,18
bdff3acf-88a1-473a-b3d7-542ec6cf20a1_66_9_9,bdff3acf-88a1-473a-b3d7-542ec6cf20a1_66_7_7,bdff3acf-88a1-473a-b3d7-542ec6cf20a1,66,,0.996,"{Lucy,and,Simon,do,deliver,Chloé,to,Thomas,and,Pierre,"","",but,they,say,there,is,one,more,returned,person,they,want,.}",Pierre,9,9,Thomas,7,7
8fe3c398-184d-4c31-b8ce-31bde7445d87_2_17_20,8fe3c398-184d-4c31-b8ce-31bde7445d87_2_13_15,8fe3c398-184d-4c31-b8ce-31bde7445d87,2,,0.995,"{She,was,born,October,2,"","",1963,in,West,Helena,"","",Arkansas,to,Arvey,Lee,Turner,and,Dorothy,Ann,Taylor,Turner,.}",Dorothy Ann Taylor Turner,17,20,Arvey Lee Turner,13,15
fd868d84-0c8b-4f60-8050-77013669a6d5_18_4_5,fd868d84-0c8b-4f60-8050-77013669a6d5_18_1_2,fd868d84-0c8b-4f60-8050-77013669a6d5,18,,1,"{05,Randeep,Hooda,and,Lisa,Haydon,:,Soon,after,the,release,of,her,successful,film,Queen,"","",model-turned-actor,Lisa,Haydon,and,actor,Randeep,Hooda,were,snapped,in,Mumbai,late,in,the,night,.}",Lisa Haydon,4,5,Randeep Hooda,1,2
[...]

```

##### 1.2. Prepare the Mindtagger configuration and template

In order to use these sampled data items with Mindtagger, you need to create two more files that define a task for Mindtagger: a configuration and a template. Mindtagger configuration that looks like below should go into [the `mindtagger.conf` file](https://github.com/HazyResearch/deepdive/blob/master/examples/spouse/labeling/has_spouse-precision/mindtagger.conf). You can specify the path to the file holding the data items as well as the column names that are the keys, e.g. `p1_id` and `p2_id` in this example.

```
title: Labeling task for estimating has_spouse precision
items: {
    file: has_spouse.csv
    key_columns: [p1_id, p2_id]
}
template: template.html


```

As shown in the configuration, the template for the task is set to `template.html`. A Mindtagger template is a collection of HTML fragments decorated with Mindtagger-specific directives that control how the data items are rendered and which tags and GUI elements should be available during the task. For the precision task at hand, creating [a `template.html`file with contents similar to the following](https://github.com/HazyResearch/deepdive/blob/master/examples/spouse/labeling/has_spouse-precision/template.html) will do the job.

```
<mindtagger mode="precision">

  <template for="each-item">
    <strong title="item_id: {{item.id}}">{{item.p1_text}} -- {{item.p2_text}}</strong>
    with expectation <strong>{{item.expectation | number:3}}</strong> appeared in:
    <blockquote>
        <big mindtagger-word-array="item.tokens" array-format="postgres">
            <mindtagger-highlight-words from="item.p1_start" to="item.p1_end" with-style="background-color: yellow;"/>
            <mindtagger-highlight-words from="item.p2_start" to="item.p2_end" with-style="background-color: cyan;"/>
        </big>
    </blockquote>

    <div>
      <div mindtagger-item-details></div>
    </div>
  </template>

  <template for="tags">
    <span mindtagger-adhoc-tags></span>
    <span mindtagger-note-tags></span>
  </template>

</mindtagger>

```

The template is mostly self explanatory, and there are some JavaScript-like expressions appearing here and there. In fact, Mindtagger GUI is implemented with [AngularJS](https://angularjs.org/), and standard AngularJS expressions and directives can be used to extend the template. In addition to the standard interface, Mindtagger provides several domain-specific directives and variables scoped within the template which are useful for creating the desired interface for data annotation. For example, `mindtagger-word-array` directive makes it very simple to render an array of words which is a commonly used data representation across many text and NLP-based DeepDive applications. Also the column values of each item are available through the `item` variable.

##### 1.3. Launch Mindtagger

The user can now launch Mindtagger by passing the path to your `mindtagger.conf` file as a command-line argument such as follows:

```
mindbender tagger labeling/*/mindtagger.conf

```

If the default port (8000) is already used by other tasks, the user can specify an alternative port, say 12345 as follows:

```
PORT=12345 mindbender tagger labeling/*/mindtagger.conf

```

##### 1.4. Mark each prediction as correct or not

Now, the user can point the browser to the URL displayed in the output of Mindtagger (or `http://localhost:8000` if that doesn't work). The following screenshot shows the annotation task in progress. ![Screenshot of Mindtagger precision task in progress](http://deepdive.stanford.edu/images/mindtagger_screenshot.png)

For the precision mode, Mindtagger provides dedicated buttons colored green/red for marking whether each item is relevant or not which will be linked to the value for `is_correct` tag. One should inspect each item, and mark each item by pushing the right button. (Hint: Mindtagger supports keyboard shortcuts to mark responses and move between items. Press ? key for details.)

Ad-hoc tags can also be added to each item, e.g., for marking the reason or type of error. These tags will be very useful later for clustering the false positive errors based on their type.

##### 1.5. Count annotations

While inspecting every item, one can quickly check how many items were labeled with each tag using the "Tags" dropdown on the top-right corner.

![Screenshot of tags frequency display in Mindtagger](http://deepdive.stanford.edu/images/mindtagger_screenshot_tags.png)

##### 1.6. Export annotations

Suppose one wants to augment the ground truth with the `is_correct` tags marked on each item throughout this task. Using Mindtagger's export tags feature ("Export" on the top-right), one can download the tag data as SQL with `UPDATE` or `INSERT` statements, as well as CSV/TSV or JSON.

![Screenshot of exporting tags from Mindtagger](http://deepdive.stanford.edu/images/mindtagger_screenshot_export.png)

The downloaded SQL file will look like the following, ready to fill in the `is_correct` column of the `has_spouse_labeled`table in the database:

```
UPDATE "has_spouse_labeled" SET ("is_correct") = (FALSE)    WHERE "p1_id" = 'b5d4091a-4fb0-4afb-8d49-65e985f1be21_65_13_13' AND "p2_id" = 'b5d4091a-4fb0-4afb-8d49-65e985f1be21_65_11_11';
UPDATE "has_spouse_labeled" SET ("is_correct") = (FALSE)    WHERE "p1_id" = '248c8a3a-ea00-454d-b018-968e410c190f_41_16_16' AND "p2_id" = '248c8a3a-ea00-454d-b018-968e410c190f_41_18_18';
UPDATE "has_spouse_labeled" SET ("is_correct") = (FALSE)    WHERE "p1_id" = 'bdff3acf-88a1-473a-b3d7-542ec6cf20a1_66_9_9'   AND "p2_id" = 'bdff3acf-88a1-473a-b3d7-542ec6cf20a1_66_7_7';
UPDATE "has_spouse_labeled" SET ("is_correct") = (TRUE) WHERE "p1_id" = '8fe3c398-184d-4c31-b8ce-31bde7445d87_2_17_20'  AND "p2_id" = '8fe3c398-184d-4c31-b8ce-31bde7445d87_2_13_15';
UPDATE "has_spouse_labeled" SET ("is_correct") = (FALSE)    WHERE "p1_id" = 'fd868d84-0c8b-4f60-8050-77013669a6d5_18_4_5'   AND "p2_id" = 'fd868d84-0c8b-4f60-8050-77013669a6d5_18_1_2';
UPDATE "has_spouse_labeled" SET ("is_correct") = (FALSE)    WHERE "p1_id" = '38e9afdb-8323-4815-bcdc-40f54a69f0c8_54_13_14' AND "p2_id" = '38e9afdb-8323-4815-bcdc-40f54a69f0c8_54_10_11';
UPDATE "has_spouse_labeled" SET ("is_correct") = ('UNKNOWN')    WHERE "p1_id" = 'd30a9e45-3b43-4082-a6c2-ae4f1db5fbf7_10_4_4'   AND "p2_id" = 'd30a9e45-3b43-4082-a6c2-ae4f1db5fbf7_10_6_6';
[...]

```

For other ad-hoc tags, exporting into different formats may be more useful.

#### Use Case 2: Inspecting Features

After inspecting a number of predictions for estimating precision, user may start to wonder which features for the probabilistic inference were produced for each item. In this second example, we will show how one can extend the first example to include the features related to each prediction to get a better sense of what's going on. This is not a task for simply estimating a quality measure of the result, but rather a more elaborate error analysis task to understand which features are doing well/poorly and to gain more insights for debugging/improving the DeepDive application. Using ad-hoc tags will be much more important in this task since they will help to prioritize fixing more common source of errors.

##### 2.1. Prepare mentions with relevant features

The following SQL query can be used to include the features related to the prediction along with their weights.

```
deepdive sql eval "
SELECT hsi.*
     , f.features
     , f.weights
FROM (

SELECT hsi.p1_id
     , hsi.p2_id
     , s.doc_id
     , s.sentence_index
     , hsi.label
     , hsi.expectation
     , s.tokens
     , pm1.mention_text AS p1_text
     , pm1.begin_index  AS p1_start
     , pm1.end_index    AS p1_end
     , pm2.mention_text AS p2_text
     , pm2.begin_index  AS p2_start
     , pm2.end_index    AS p2_end

  FROM has_spouse_inference hsi
     , person_mention             pm1
     , person_mention             pm2
     , sentences                  s

 WHERE hsi.p1_id          = pm1.mention_id
   AND pm1.doc_id         = s.doc_id
   AND pm1.sentence_index = s.sentence_index
   AND hsi.p2_id          = pm2.mention_id
   AND pm2.doc_id         = s.doc_id
   AND pm2.sentence_index = s.sentence_index
   AND       expectation >= 0.9

 ORDER BY random()
 LIMIT 100

) hsi, (
SELECT p1_id
     , p2_id
     , ARRAY_AGG(feature ORDER BY abs(weight) DESC) AS features
     , ARRAY_AGG(weight  ORDER BY abs(weight) DESC) AS weights
  FROM dd_weights_inf_istrue_has_spouse dd1
     , dd_inference_result_weights dd2
     , spouse_feature
 WHERE feature = dd_weight_column_0
   AND dd1.id = dd2.id
 GROUP BY p1_id,p2_id
) f
WHERE hsi.p1_id          = f.p1_id
  AND hsi.p2_id          = f.p2_id

" format=csv header=1 >has_spouse-with_features.csv

```

Running it will generate [a `has_spouse-with_features.csv`](https://github.com/HazyResearch/deepdive/blob/master/examples/spouse/labeling/has_spouse-precision-with_features/has_spouse-with_features.csv)that looks like below:

```
p1_id,p2_id,doc_id,sentence_index,label,expectation,tokens,p1_text,p1_start,p1_end,p2_text,p2_start,p2_end,features,weights
22adb358-5ea8-40bf-ad95-b5a7e1810bea_27_11_12,22adb358-5ea8-40bf-ad95-b5a7e1810bea_27_9_9,22adb358-5ea8-40bf-ad95-b5a7e1810bea,27,,0.996,"{Judge,Stephenson,is,preceded,in,death,by,his,parents,Paul,and,Carroll,Stephenson,.}",Carroll Stephenson,11,12,Paul,9,9,"{INV_NER_SEQ_[O],INV_NGRAM_1_[and],INV_POS_SEQ_[CC],INV_WORD_SEQ_[and],INV_LEMMA_SEQ_[and],IS_INVERTED,INV_STARTS_WITH_CAPITAL_[True_True],INV_BETW_D_[conj_and],INV_BETW_L_[conj_and],INV_BETW_[conj_and],INV_W_NER_L_1_R_1_[O]_[O],INV_LENGTHS_[3_0],""INV_W_NER_L_2_R_1_[O O]_[O]"",INV_W_LEMMA_L_1_R_1_[parent]_[.],""INV_W_LEMMA_L_2_R_1_[he parent]_[.]"",""INV_W_NER_L_3_R_1_[O O O]_[O]"",""INV_W_LEMMA_L_3_R_1_[by he parent]_[.]""}","{2.94252,-2.52698,2.01443,1.98219,1.98219,-1.41634,-0.949953,0.334665,0.330707,0.324957,-0.282554,-0.212128,-0.181359,-0.024521,-0.0220253,0.00138261,0}"
8afc4b5a-e433-46c9-8290-4f541a7f8f16_17_15_16,8afc4b5a-e433-46c9-8290-4f541a7f8f16_17_13_13,8afc4b5a-e433-46c9-8290-4f541a7f8f16,17,,0.997,"{Or,some,folks,prefer,Rose,Lake,"","",Mich.,"","",including,Sharon,Kellermeier,"","",Dave,and,Ellen,Nowak,"","",and,Brent,and,Pam,Cousino,.}",Ellen Nowak,15,16,Dave,13,13,"{INV_NER_SEQ_[O],INV_NGRAM_1_[and],INV_POS_SEQ_[CC],INV_WORD_SEQ_[and],INV_LEMMA_SEQ_[and],IS_INVERTED,INV_STARTS_WITH_CAPITAL_[True_True],""INV_W_LEMMA_L_1_R_1_[,]_[,]"",INV_W_NER_L_1_R_1_[O]_[O],INV_LENGTHS_[2_0],""INV_W_NER_L_1_R_3_[O]_[O O O]"",""INV_W_NER_L_2_R_1_[PERSON O]_[O]"",""INV_W_NER_L_2_R_3_[PERSON O]_[O O O]"",""INV_BETW_L_[conj_and conj_and]"",""INV_W_NER_L_3_R_1_[PERSON PERSON O]_[O]"",""INV_W_NER_L_3_R_3_[PERSON PERSON O]_[O O O]"",""INV_W_LEMMA_L_1_R_2_[,]_[, and]"",""INV_W_NER_L_2_R_2_[PERSON O]_[O O]"",""INV_W_NER_L_1_R_2_[O]_[O O]"",""INV_W_NER_L_3_R_2_[PERSON PERSON O]_[O O]"",""INV_W_LEMMA_L_3_R_1_[Sharon Kellermeier ,]_[,]"",""INV_BETW_D_[conj_and Kellermeier conj_and]"",""INV_W_LEMMA_L_2_R_2_[Kellermeier ,]_[, and]"",""INV_W_LEMMA_L_3_R_3_[Sharon Kellermeier ,]_[, and Brent]"",""INV_W_LEMMA_L_3_R_2_[Sharon Kellermeier ,]_[, and]"",""INV_W_LEMMA_L_2_R_3_[Kellermeier ,]_[, and Brent]"",""INV_BETW_[conj_and Kellermeier conj_and]"",""INV_W_LEMMA_L_2_R_1_[Kellermeier ,]_[,]"",""INV_W_LEMMA_L_1_R_3_[,]_[, and Brent]""}","{2.94252,-2.52698,2.01443,1.98219,1.98219,-1.41634,-0.949953,0.620401,-0.282554,-0.260218,0.240517,-0.20124,-0.182793,0.121314,-0.117852,-0.0994847,0.0461265,-0.044868,-0.0407692,-0.0034324,0,0,0,0,0,0,0,0,0}"
[...]

```

##### 2.2. Modify template to render features

Now, we can use the following [Mindtagger template](https://github.com/HazyResearch/deepdive/blob/master/examples/spouse/labeling/has_spouse-precision-with_features/template-with_features.html) to enumerate the extra feature information.

```
<mindtagger mode="precision">

  <template for="each-item">
    <strong title="item_id: {{item.id}}">{{item.p1_text}} -- {{item.p2_text}}</strong>
    with expectation <strong>{{item.expectation | number:3}}</strong> appeared in:
    <blockquote>
        <big mindtagger-word-array="item.tokens" array-format="postgres">
            <mindtagger-highlight-words from="item.p1_start" to="item.p1_end" with-style="background-color: yellow;"/>
            <mindtagger-highlight-words from="item.p2_start" to="item.p2_end" with-style="background-color: cyan;"/>
        </big>
    </blockquote>

    <div class="row" ng-if="item.features">
      <!-- Enumerate features with weights (leveraging AngularJS a bit more)-->
      <div class="col-sm-offset-1 col-sm-10">
        <table class="table table-striped table-condensed table-hover">
          <thead><tr>
              <th class="col-sm-1">Weight</th>
              <th>Feature</th>
          </tr></thead>
          <tbody>
            <tr ng-repeat="feature in item.features | parsedArray:'postgres' track by $index">
              <td class="text-right">{{(item.weights | parsedArray:'postgres')[$index] | number:6}}</td>
              <th>{{feature}}</th>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div>
      <div mindtagger-item-details></div>
    </div>
  </template>

  <template for="tags">
    <span mindtagger-adhoc-tags></span>
    <span mindtagger-note-tags></span>
  </template>

</mindtagger>

```

##### 2.3. Browse the predictions with features and weights

Now, the user can see all the features next to their learned weights shown below. By looking at the features through this task, the user can discover some features that work well or not as well and can come up with a more data-driven idea for improving the application's performance for the next iteration.

![img](http://deepdive.stanford.edu/images/mindtagger_screenshot_with_features.png)



### Monitoring statistics of DeepDive data with Dashboard

*Dashboard* serves as an interface for viewing and analyzing the results of your DeepDive application runs. It provides a structure to organize various report templates that compute app-specific metrics and snapshot of reports produced after each run.

This document is composed of two main sections. First, we explain how to use Dashboards with build-in and user-specified templates from the [user interface](http://deepdive.stanford.edu/dashboard#user-interface) as well as defining tasks and trends. Then we will dive in to an [advanced topic](http://deepdive.stanford.edu/dashboard#advanced-topics)which is more directed to terminal lovers. We explain how to build all these reports without using the user interface, which is less intuitive but fully programmable giving more flexibility.

#### User interface

In the user interface, Dashboard provides built-in [report templates](http://deepdive.stanford.edu/dashboard#report-templates) (as well as an ability to [create your own templates](http://deepdive.stanford.edu/dashboard#writing-custom-templates)) which generate visual reports for the resulting data from your DeepDive application run. Reports are generated by running Dashboard [snapshots](http://deepdive.stanford.edu/dashboard#snapshots) which capture and retain the data relevant for displaying the reports. [Snapshot configurations](http://deepdive.stanford.edu/dashboard#configuring-snapshots)can be created to specify which reports should be generated for a particular Dashboard snapshot.

Once a snapshot has been run, you will be able to view the resulting reports. Additionally, you can run [tasks](http://deepdive.stanford.edu/dashboard#tasks) on values from a report for further analysis.

To analyze the results of your DeepDive application runs over time, you can specify to track certain report values and view a visual representation of how they change over time using [Trends](http://deepdive.stanford.edu/dashboard#trends).

Below are two example reports based on the built-in templates: ![Dashboard Custom Report](http://deepdive.stanford.edu/images/dashboard/calibration_plot_report.png)

![Dashboard Formatted Report](http://deepdive.stanford.edu/images/dashboard/top_positive_features_report.png)

##### Launching Dashboard

To bring up a GUI for Dashboard, simply run:

```
mindbender dashboard

```

Let's observe that running a search GUI for [browsing data](http://deepdive.stanford.edu/browsing) will also let you see Dashboard (using the dropdown menu on the top-left).

##### Report templates

Report templates, accessible from the "Configure Templates" page in the GUI, define the data displayed on a report and the appearance of the report. A number of general report templates come with Dashboard, but you may also [create your own](http://deepdive.stanford.edu/dashboard#writing-custom-templates).

###### Template types

There are two types of report templates:

- Formatted: Defined by a single SQL query. This template type simply runs SQL query on your data and displays the results as a table on the report. Formatted template also gives an option to include charts on the report, where the X and Y axes correspond to column names from the SQL query.
- Custom: Defined by a shell script which allows you to manipulate your data as needed. There are a number of helpful [built-in shell commands](http://deepdive.stanford.edu/dashboard#custom-template-commands) you can use to perform actions such as run queries and generate custom charts.

###### Parameters

Parameters allow you to include variables in report templates which can be set when the snapshot runs. A common use case for parameters is when you want to run the same report template on different data sources from your DeepDive application run, or want to make a quick series of snapshots involving a varying number of data items.

The name of a parameter corresponds to the name of the variable in the report template. For example, if the parameter name is "doc_id", the corresponding template variable is also `$doc_id`.

###### Nested templates

You may have noticed that the built-in report template names use slashes (`/`) to indicate a hierarchical structure. While this is a helpful strategy for organizing your report templates, the slash also has the effect of "nesting" the report template.

Nested report templates automatically inherit parameters from their ancestor report templates and are shown in grey on a nested report template.

More information on nested templates is documented in the advanded [section](http://deepdive.stanford.edu/dashboard#extending-report-templates)

##### Snapshots

A Snapshot captures all of the data required to generate the reports associated with its snapshot configuration. Prior to running a snapshot, you must specify its snapshot configuration.

###### Configuring snapshots

A snapshot configuration consists of a list of report templates and parameter values for each report template. Snapshot configurations are made from the "Run Snapshot" tab.

A default snapshot configuration is included with Dashboard. You may modify this configuration or create your own by clicking the "Add Configuration" button. You may also copy the contents of an existing configuration to a new configuration using the "Copy Configuration" button from an existing configuration.

To add a report template to the snapshot configuration, click the "Add Template" button and select the report template of interest from the dropdown menu. You can add additional report templates to the configuration using these same steps. If you want to remove a template from the snapshot configuration, click the red "X" next to the report template dropdown.

When a report template is selected, corresponding parameters for that template will appear directly below. You should fill in these parameters with the values relevant to your interests and consistent with the DeepDive application structure. Some parameters may already have beeen filled in if the default value was specified in the report template. For built-in report templates, you can refer to the parameter's "Description" field for a description of what value the parameter should take.

Once you have finished adding the report templates and their corresponding parameter values which comprise the snapshot configuration, save the configuration by clicking the "Update Configuration" button.

A configuration can be deleted by clicking the "Delete Configuration" button from an existing configuration.

![Running a Dashboard snapshot](http://deepdive.stanford.edu/images/dashboard/run_snapshot.png)

###### Running snapshots

To run a snapshot, first load its snapshot configuration by selecting the snapshot name from the dropdown on the *Run Snapshots* page. This will show you the list of report templates and corresponding parameter values which will be used to create the snapshot.

Click the "Run Snapshot" button at the bottom of the snapshot configuration to run it.

##### Viewing reports

Once a snapshot has been made, the reports corresponding to the report templates used in the snapshot configuration will be available from the "View Snapshots" tab.

The name of the snapshot corresponds to the date the snapshot was made followed by the run number for that day. For example, snapshot "20150714-5" was ran on July 14, 2015, and was the fifth snapshot run of that day.

To view the reports associated with a snapshot, click on the snapshot name. On the page for that individual snapshot, the reports are shown in a navigation menu on the left. Any nested reports are displayed hierarchically - you can use the small arrow to the left of the report names to collapse and expand the reports nested beneath it.

To view a report, click the report's name from the navigation menu. The report will display a data table or chart(s), depending on how the report template was written.

At the top right of the report are three buttons:

- Details: View the raw settings used to generate this report.
- Template: Go to the report template for this report.
- Tasks: Opens the interface for running tasks on this report. See the [tasks documentation](http://deepdive.stanford.edu/dashboard#tasks).

![Dashboard Custom Report](http://deepdive.stanford.edu/images/dashboard/supervision_report.png)

##### Tasks

Tasks allow you to perform further analysis on the data displayed in a report. For example, you can display the number of mentions you extracted from a text over time.

Tasks are configured and displayed in a very similar manner as reports but with one major difference: the parameters for a task are supplied, or *bound*, by the user interaction from a report. The type of the data you are binding to a parameter must match the type specified in the task template.

Templates currently allow for 3 type specifications: int, float, and string.

If no type is specified on the task template, any data value can be bound to the parameter.

###### Configuring task templates

Dashboard contains built-in task templates but you can also write your own task templates.

Task templates are configured in the same way as report templates, from the "Configure Templates" page. Refer to the [documentation on writing custom templates](http://deepdive.stanford.edu/dashboard#writing-custom-templates) for more information.

###### Running tasks

Tasks are ran from a report page. To initiate a Task, click on any data value contained in a data table or chart tooltip within a report. Doing so will display a menu of all relevant tasks which accept a parameter of the type of the data value you clicked on.

Next to each Task is its corresponding list of parameters which should be supplied interactively by the user. Parameters in bold are those for which the data value selected can be bound to. To bind the data value you clicked on to a task parameter, simply click on the parameter. The bound value will display next to the parameter. You can continue this process for any additional parameters. Parameter bindings can be un-bound by clicking the parameter you wish to remove.

![Select values from the report to use as task input](http://deepdive.stanford.edu/images/dashboard/task_input.png)

At any time, you can also open the Task control interface by clicking the "Tasks" button at the top right of the report. The Task control interface will display any currently selected task, as well as the current parameter bindings. You can edit any of the parameter bindings by cliking on them. You can also move the parameter bindings around by dragging them.

When you are ready to run the Task, click "Run Task" from the task control interface. When complete, the Task's output will be available from the report navigation menu directly below the report you ran the task on.

![Verify the task template parameters and run the task](http://deepdive.stanford.edu/images/dashboard/task_control.png)

##### Trends

As you run more and more snapshots, you may be interested in seeing how certain data values from your reports change over time. Trends provides a simple mechanism which you can accomplish this.

Tracking data values can be accomplished via the `report-value` command. For example, if you wish to track a precision value from a report, assuming the precision is stored in a `$precision` variable, simply include the following in the report template:

```
precision=...  # computed somehow

report-value precision="$precision"

```

Assuming the report is called "my_report", this command will store the precision in a trend named "my_report/precision", which is accessible from Trends.

###### Trends overview

To view an overview of the changes in all trends for all time, visit the "Trends" page. Numeric trends will show small charts indicating the changes in values over time, and non-numeric trends will display a color band where each color uniquely identifies a categorical report value. Hovering over any of the data points will reveal an exact data value and the time of the snapshot which captured it.

![Trends](http://deepdive.stanford.edu/images/dashboard/trends.png)

###### Comprehensive trend view

To view a trend in more detail, click on it from the Trends overview page. This page will display a larger version of the chart shown in the overview with additional options to limit the time frame and whether to display null trend values. Limiting the time frame can be accomplished by moving either end of the slider at the bottom of the page. The number of snapshots and time range of those snapshots the view has been limited to are shown above the slider.

Toggling the Null Values button will add/remove snapshots from the chart which either did not track the trend at that point in time, or the value of the trend was `null`.

Clicking on a data point or colored band block will take you to the report in the relevant snapshot for that data value.

![Trend](http://deepdive.stanford.edu/images/dashboard/trend.png)

###### Dashboard trends

You may want to make a few important trends easily viewable from the homepage of Dashboard for quick monitoring. To do so, click the "Add to Dashboard" button on the comprehensive trend view page to add a smaller version of the chart to the homepage of Dashboard. You can toggle this button to remove the chart from the homepage at any time.

![Dashboard](http://deepdive.stanford.edu/images/dashboard/homepage.png)

##### Writing custom templates

In addition to the built-in templates you can also write your own templates.

To start a new template, click the "Add Template" button from the "Configure Templates" page. You can also click the "Copy Template" button from an existing template to create a new template with the same contents of the existing template.

If you are creating a formatted template, you simply need to ensure that the output of the template is a single SQL query.

For custom templates, you can use [markdown](https://help.github.com/articles/markdown-basics/) and shell commands to run any desired commands and provide custom output for the report. To make writing custom templates easier, we have included a number of built-in shell commands which are described in the [Custom Template Commands](http://deepdive.stanford.edu/dashboard#custom-template-commands)section below.

You will also need to specify whether the template is for a Report or Task. If you are creating a Task template, you additionally need to specify the *scope* of the template, which is the report (or set of reports if selecting a hierarchical report name) that the task is valid for.

#### Advanced topics

Here, we explore the details of Dashboard. In particular how to write specific commands and scripts such that many built-in and user-defined reports can be run automatically.

More info can be found on the [Mindbender repo](https://github.com/HazyResearch/mindbender/tree/master/dashboard#readme).

##### DeepDive snapshots

After each run of a DeepDive application, you can run

```
mindbender snapshot

```

This command is used to produce a *snapshot* under `snapshot/` of the target app with a unique name beginning with a timestamp, e.g., `snapshot/20150206-1/`.

- **Reports**. Each snapshot will contain a set of reports that summarize various aspects about the data products of the DeepDive run or the code that produced them.
  - The set of reports to be produced are controlled by a *snapshot configuration* (described in the next section).
  - `reports/` directory that contains all the individually produced reports.
  - `README.md` merges those of the individual reports.
  - `reports.json` aggregation all the important values reported by individual reports in a machine-friendlier JSON format.
- **Files**. Alongside the reports, copies of important DeepDive artifacts such as the `application.conf` and extractor code are automatically preserved for future reference under the `files/` directory.
  - The list of files to keep can be enumerated in `snapshot-files` next to the `snapshot/`directory.

##### Dashboard snapshot configuration

The list of reports each new snapshot produce should be configured in a *snapshot configuration*.

- A dashboard snapshot configuration is a plain text file named `snapshot-*.conf` that resides at the root of the DeepDive app.
- Any text after a `#` character is ignored, so comments can be written there.
- Each of its line begins with either:
  - `section`, followed by a section title, or
  - `report`, followed by a name referring to a *report template* (described in the next section), and zero or more named parameters separated by white space.
- Each time a new snapshot is created, a copy of this snapshot configuration will be retained in the snapshot.

Here's an example snapshot configuration that produces four reports.

```
#### dashboard snapshot configuration

# show statistics about the corpus stored in table "sentences"
report corpus   table=sentences column_document=doc_id column_sentence=sent_id

section "Variables"
report variable/inference    variable=has_spouse.is_correct
report variable/supervision  variable=has_spouse.is_correct top_positive=10 top_negative=10
report variable/feature      variable=has_spouse.is_correct
report variable/candidate    variable=has_spouse.is_correct

```

##### Report template

Each report in a snapshot is produced by a *report template*.

- Structure

  : a report template is a carefully structured directory on the file system.

  - It resides under either:
    - `snapshot-template/` of the target DeepDive app (app-specific), or
    - `dashboard/snapshot-template/` of Mindbender's source tree (built-in).
  - It is referred from the snapshot configuration with the path name relative to `snapshot-template/`.
  - It must contain one or more of the following:
    - An executable file named `report.sh`.
    - An executable document `README.md.in`.
    - One or more *nested report templates*(described later).
  - It may contain as many extra files as it needs.
    - All files in the template will be cloned to the report instance.
    - Any file named `*.in` will be considered as executable documents and automatically compiled at production time.

- Parameters

  : it takes zero or more named parameters (i.e.,

   

  ```
  name=value
  ```

   

  pairs).

  - The parameters should be declared in the `report.params` file.

  - Each `report` line in the snapshot configuration provides value bindings for these parameters.

  - This allows a template to be instantiated with different sets of parameters to produce multiple reports.

  - The

     

    ```
    report.params
    ```

     

    file must be formatted in the following way:

    - Each line declares a parameter,

      - whose name may contain only alphanumeric letters and underscore (`_`) and must begin with an alphabet or underscore.

    - A required parameter is declared by a line beginning with

       

      ```
      required
      ```

      , followed by

      - the parameter name, and
      - a description surrounded by quotes.

    - An optional parameter is declared by a line beginning with

       

      ```
      optional
      ```

      , followed by

      - the parameter name,
      - the default value surrounded by quotes, and
      - a description surrounded by quotes.

- **Executables**: a report template must contain at least one executable that produces its contents.

  - A

     

    ```
    report.sh
    ```

     

    shell script can invoke as many other programs as it needs to produce various output files in the report as well as the

     

    ```
    README.md
    ```

     

    and

     

    ```
    report.json
    ```

    .

    - Several utilities are provided to simplify the common tasks from writing valid JSON to querying data in the underlying database of the DeepDive app.

  - An executable document, named `*.in`, is a text file mixed with shell scripts.

    - Shell script fragments must be surrounded by `<$` and `$>`, e.g.,

      ```
      There are <$ deepdive sql "SELECT COUNT(DISTINCT doc_id) FROM sentences" $> documents in the corpus.

      ```

    - After executing every mixed in shell script in the order they appear, their output will be interpolated with the normal text, and written to a file named without the trailing `.in`, e.g., `README.md.in` will produce a `README.md` after execution.

    - Keeping the reporting logic in executable documents may be simpler than `report.sh` if it mainly computes values to be presented in between a chunk of text, e.g., as for `README.md`.

    - It can be viewed as an inverted shell script, where normal text is written to the output in between running the fragments of shell scripts.

###### How each report is produced

Suppose a line in the snapshot configuration refers to report template `variable/inference` with a named parameter `variable=has_spouse.is_correct`. Following steps are taken to produce the report.

1. Instantiation

   . A

    

   report instance

    

   (or simply

    

   report

   ) directory is created under

    

   ```
   reports/
   ```

    

   of the snapshot, e.g.,

    

   ```
   reports/variable/inference/
   ```

   .

   - If the same report template is instantiated more than once (most likely with different parameters), the path of the report will be suffixed with a unique serial number, to isolate each report instance from others, e.g., `reports/variable/inference-2/`, `reports/variable/inference-3/`, etc.
   - All files in the template directory for `variable/inference` are cloned into the report instance, preserving the structure within the template.

2. Parameters

   . All parameters given by the snapshot configuration are checked against the

    

   ```
   report.params
   ```

   specification

   - If there are `report.params` in the parent directories of the report template, parameters declared in them will also be checked. For example, `variable/report.params` as well as `variable/inference/report.params` will be used.
   - Finally, all parameter values are recorded in `report.params.json` in JSON format as well as `.report.params.sh` in a shell script.

3. Execution

   . All

    

   ```
   report.sh
   ```

    

   scripts and executable documents (

   ```
   *.in
   ```

   ) in the report are executed under a controlled environment:

   - All named parameters for the report are declared as environment variables.
   - Current working directory is set to the report directory. This localizes file accesses with the scripts and executable documents, and therefore simplifies reading files in the template as well as generating new ones without having to refer to any global path or variables.

###### What each report contains after production

- `README.md` -- a human-readable content of the report in Markdown syntax is expected to be produced.

- `report.json` -- a machine-readable content of the report in JSON format may be produced as well to easily keep track of important values.

- ```
  report.params.json
  ```

   

  -- a machine-readable file that records all parameters used for producing the report in JSON format.

  - `.report.params.sh` -- all parameter bindings stored in a shell script, so it can be easily loaded.
  - `.report.params.*` -- other by-products of checking parameters.

- `.report.id` -- a unique identifier (within a snapshot) for the report is generated and stored in this file.

- `*` -- rest of the files are either cloned from the report template, or generated by an executable in the report.

###### Extending report templates

Report templates are carefully designed to be easily extensible. Since it is often necessary to augment part of an existing report with app-specific metrics or sample data, the *nested report template* design tries to enable this with minimal user intervention, avoiding repetition of existing report templates as much as possible.

- **Nested report templates**. A report template may be nested under another template. When instantiating the parent template from the snapshot configuration,

  - All nested ones will be instantiated with the same set of parameters. All `report.params`specifications found along the path to each nested one will be used to check and supply default values for the parameters.
  - App-specific as well as built-in nested templates will all be instantiated. Therefore, a built-in report template can be easily extended from a DeepDive app by adding app-specific nested templates. For example, `variable/new-metric`in the app extends the built-in `variable`template.

- **Ordering nested templates**. The instantiation order of the nested templates can be specified in a special file named `reports.order`.

  - Each line of the file should contain a glob pattern matching nested report templates under it.

  - At most one of the line may be wildcard `*`, which denotes the position for the rest of the paths not explicitly mentioned.

  - For example, `variable/reports.order` has the following lines, which orders the summary at the top and the built-in templates in a particular order at the bottom, so any app-specific ones appear first:

    ```
    summary
    *
    quality
    inference
    supervision
    feature
    candidate

    ```

  - Currently, there's a limitation that nested `reports.order` have no effect, and only the one at the top of the template directly mentioned from the snapshot configuration is taken into account.

For example, consider the "Variables" section of the previous snapshot configuration:

```
###### dashboard snapshot configuration (instantiating each template individually)
section "Variables"
report variable/inference    variable=has_spouse.is_correct
report variable/supervision  variable=has_spouse.is_correct top_positive=10 top_negative=10
report variable/feature      variable=has_spouse.is_correct
report variable/candidate    variable=has_spouse.is_correct

```

This could be rewritten as a single line as shown below:

```
###### dashboard snapshot configuration (instantiating group of nested templates)
section "Variables"
report variable              variable=has_spouse.is_correct top_positive=10 top_negative=10

```

Furthermore, when more templates are added to `variable`, they will also get produced automatically without adding numerous lines for every instantiation of `variable` in the snapshot configuration.

### Utilities for report templates

Several utilities are provided to the executables in report templates to simplify the writing of new report templates.

###### For producing JSON

`report-values` command can be used for augmenting named values to the `report.json` file without dealing with JSON parsing and formatting. For example, suppose `report.json`already had the following content:

```
{
  "a": "foo",
  "b": "bar"
}

```

Simply running the command below will update `report.json`to have what follows:

```
report-values x=1 y=2.34 b=true c=bar d='[1,"2","three"]'

```

```
{
  "a": "foo",
  "b": true,
  "x": 1,
  "y": 2.34,
  "c": "bar",
  "d": [
    1,
    "2",
    "three"
  ]
}

```

As shown in this example, values passed as arguments can be a valid JSON formatted string or they will be treated as a normal string.

###### Running SQL queries

`deepdive sql` command runs a SQL query against the underlying database for the current DeepDive app, and outputs the result in a tab-separated format.

###### Including CSV/TSV data in HTML or Markdown

`html-table-for` command formats a given CSV or TSV file into an HTML table that can be included in Markdown documents. For example, the following executable document runs a SQL query to retrieve 10 sample candidates and presents a table. Note the extra `CSV HEADER` arguments to `deepdive sql` for producing a CSV format compatible with this command.

```
<!-- README.md.in -->

####### 10 Most Frequent Candidates
Here are the 10 most frequent candidates extracted by DeepDive:
<$
deepdive sql "
    SELECT words, COUNT(*) AS count
    FROM candidate_table
    GROUP BY words
    ORDER BY count DESC
    LIMIT 10
" format=csv header=1 >top_candidates.csv

html-table-for top_candidates.csv
$>

```

It will produce a table that looks like the following:

> #### 10 Most Frequent Candidates
>
> Here are the 10 most frequent candidates extracted by DeepDive:
>
> | words | count |
> | ----- | ----- |
> | foo   | 987   |
> | bar   | 654   |
> | ...   | ...   |

###### Logging messages

`report-log` and `report-warn` commands shows given arguments as timestamped messages in the log. `report-warn`will start the message with a `WARNING:` sign to make it stand out.

The following command will print the next two lines below:

```
report-log "Computing something..."
report-warn "Something went wrong!"

```

```
2015-02-21 06:28:12 localhost   Computing something...
2015-02-21 06:28:13 localhost   WARNING: Something went wrong!

```

###### Displaying charts

The `<chart>` tag can be used to display a chart (rendered using [Highcharts](http://www.highcharts.com/)) within a custom report template. The attribute values for the `<chart>` tag are outlined below:

```
<chart
        highcharts-options="{}"  # Optional highcharts configurations
        data-file="FILE"         # JSON data file from which to render the chart
        data="JSON DATA"         # Raw JSON data from which to render the chart
        type="bar|scatter"       # The type of the chart
        x-axis="X_AXIS_NAME"     # The name of the column from the JSON file to use for the X axis
        x-label="X_AXIS_LABEL"   # Text label for the X axis
        y-axis="Y_AXIS_NAME"     # The name of the column from the JSON file to use for the Y axis
        y-label="Y_AXIS_LABEL"   # Text label for the Y axis
></chart>

```

Either the `data-file` or `data` attribute must be specified, but not both. The format of the JSON data can be in either row-major or column-major order, e.g.:

- Row-major order

  ```
  {
    "headers": [
      "num_candidates",
      "num_features"
    ],
    "data": [
      [
        1,
        3
      ],
      [
        2,
        5
      ],
      [
        3,
        10
      ],
      [
        4,
        54
      ]
    ]
  }

  ```

- Column-major order

  ```
  {
    "num_candidates": [
      1,
      2,
      3,
      4
    ],
    "num_features": [
      3,
      5,
      10,
      54
    ]
  }

  ```

An example use of the `<chart>` tag is given below:

```
<chart
        highcharts-options="{ chart: { width: 500, height: 400 }, title: { text: 'Candidates vs. Features' } }"
        data-file="candidates_features_data"
        type="scatter"
        x-axis="num_candidates"
        x-label="Number of Candidates"
        y-axis="num_features"
        y-label="Number of Features"
></chart>
```

#### Generating negative evidence

In common relation extraction applications, we often need to generate negative examples in order to train a good system. Without negative examples the system will tend to classify all the variables as positive. However for relation extraction, one cannot easily find enough golden-standard negative examples (ground truth). In this case, we often need to use distant supervision to generate negative examples.

Negative evidence generation can be somewhat an art. The following are several common ways to generate negative evidence, while there might be some other ways undiscussed:

- incompatible relations
- domain-specific knowledge
- random sampling

##### Incompatible relations

Incompatible relations are other relations that are always, or usually, conflicting with the relation we want to extract. Say that we have entities `x` and `y`, the relation we want to extract is `A` and one incompatible relation to `A` is `B`, we have:

```
B(x,y) => not A(x,y)

```

As an example, if we want to generate negative examples for "spouse" relation, we can use relations incompatible to spouse, such as parent, children or siblings: If `x` is the parent of `y`, then `x` and `y` cannot be spouses.

##### Domain specific rules

Sometimes we can make use of other domain-specific knowledge to generate negative examples. The design of such rules are largely dependent on the application.

For example, for spouse relation, one possible domain-specific rule that uses temporal information is that "people that do not live in the same time cannot be spouse". Specifically, if a person `x` has `birth_date` later than `y`'s `death_date`, then `x` and `y` cannot be spouses.

##### Random samples

Another way to generate negative evidence is to randomly sample a small proportion among all the variables (people mention pairs in our spouse example), and mark them as negative evidence.

This is likely to generate some false negative examples, but if statistically variables are much more likely to be false, the random sampling will work.

For example, most people mention pairs in sentences are not spouses, so we can randomly sample a small proportion of mention pairs and mark them as false examples of spouse relation.

To see an example of how we generate negative evidence in DeepDive, refer to the [example application tutorial](http://deepdive.stanford.edu/example-spouse#1-3-extracting-candidate-relation-mentions).















第三部分 CookBook
