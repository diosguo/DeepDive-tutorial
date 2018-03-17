Deepdive 实战-从下载到跑路
Deepdive 实战-从下载到跑路
第零部分 下载安装和配置
1 下载
2 系统要求
3 安装
4 配置
第一部分 简单实例
0 基础工作
1 数据准备
1.1 载入原始数据
1.2 添加自然语言标记
1.3 抽取候选关系
1.3.1 抽取候选实体
1.3.2 抽取候选关系
1.4 特征提取
1.5 样本打标
2 模型构建
2.1 模型构建
2.2 因子图构建
第二部分 详细讲解
1 背景知识
1.1 因子图
1.2 可能的世界
1.3 边际推理
1.4 DeepDive中的推理
1.5 弱监督
1.6 DeepDive 系统概述和相关术语
1.7 知识库构建
2 构建DeepDive应用
2.1 写什么，怎么写
2.1.1 DeepDive应用的结构
2.1.2 在app.ddlog定义数据流
2.1.3 用Python编写用户定义函数（UDF）
2.1.4 使用DDlog制定统计模型
2.1.3.1 Variable declarations
2.2 怎么运行
2.2.1 Compiling a DeepDive application
2.2.2 Managing input data and data products
Inspecting schema for relations in the DeepDive app
Listing relations
Listing columns of a relation
Listing variable relations
Preparing the database for the DeepDive app
Initializing the database
Creating tables in the database
Organizing input data
Moving data in and out of the database
Loading data to the database
Unloading data products from the database
Running queries against the database
Running queries in DDlog
Browsing values
Joins, selection, and projections
Aggregation
Grouping
Ordering
Limiting (top k)
Using more than one rule
Saving results
Viewing the SQL
Running queries in SQL
Executing processes in the data flow
End-to-end execution
Granular execution scenarios
Partial execution
Stopping and resuming
Repeating
Skipping
Compiled processes and data flow
Execution plan
deepdive plan
deepdive do
Execution timestamps
deepdive mark
Built-in data flow nodes
Environment variables
Learning and inference with the statistical model
Getting the inference result
Inspecting the inference result
Grounding the factor graph
Learning the weights
Inference
Reusing weights
Keeping learned weights
Reusing learned weights
Managing kept weights
2.3 评估和调试
Debugging user-defined functions
Printing to the log
Executing UDFs within DeepDive's environment
第三部分 CookBook

本文介绍Deepdive的安装和配置，然后第二部分介绍了一个简单实例，在这个实例中，只要按照步骤一步步坐下来就可以实现，然后对实例进行详细的介绍，然后介绍如何对实例中的模块进行修改。其中主要有第三方分词工具Jieba和Stanford CoreNLP句法分析的结合。

第零部分 下载安装和配置

本文的数据和工具都来自于http://www.openkg.cn/dataset/cn-deepdive

1 下载

首先在上面的页面下载浙大汉化的支持中文的Deepdive，度盘资源。因为我们后面的数据也是中文的，所以需要这个版本。当然，学习到了后面章节的时候，我们学习了如何替换其中的各个模块后，就可以自己定义语言了。

2 系统要求

在写这篇文章的时候，Ubuntu 17.04下的支持并不好，虽然作者修改安装代码最终也装上了，但是不建议新手来处理。所以推荐Ubuntu 16.04的版本。

3 安装

我们下载到的Deepdive是一个.zip格式的压缩包，所以我们先要解压缩，图形界面下：右键->提取到此处（Extract Here） ；或者命令行下cd到Deepdive.zip文件的目录下，输入命令：

 

1
unzip Deepdive.zip
就可以得到解压后的文件夹，其目录结构如下：（其中带有绿色底纹的为目录）



我们现在需要先修改一下install.sh文件，把其第193行的xzvf改为xvf后，在CNdeepdive目录下执行

 

1
./install.sh


此过程需要联网

按照提示我们输入1,然后敲击回车键，deepdive就会开始安装，安装完成后，我们再安装postgres，输入6然后敲击回车键。正常是不会报错的，如果报错，请自行解决​​

然后执行下面的命令，安装自然语言处理的部分，很快就可以结束：

 

1
./nlp_setup.sh
4 配置

上面的安装过程完成后，会在你的主目录中创建一个local文件夹，我们要把这个其中的bin 目录添加到环境变量Path中。作为文本文件打开~/.bashrc，这个文件是隐藏的，可以在命令行中使用vi，或者gedit等各种你会用的文本编辑程序；或者在图形打开主目录后，按快捷键Ctrl+H显示隐藏文件后再编辑。在文件的末尾添加下面的内容：

 

1
export PATH="/home/(username)/local/bin:$PATH"
2
# 其中的(username)要替换成你当前登录的用户名
修改完成后保存并关闭文件，在终端中输入：

 

1
source ~/.bashrc
来使我们的修改生效。

好的，到这里Deepdive就已经安装完成了，打开终端执行deepdive我们可以看到Deepdive的版本信息和命令参数的Help。如下图：（后面还有很长）



第一部分 简单实例

这部分的数据和代码以及程序都在前面我们解压后的CNdeepdive/transaction中，你可以把这个目录复制到你喜欢的地方（比如~/Projects/transaction 这个位置是我一般用的，你可以自己定），然后再新建一个目录，可以叫做transaction2 我们后面的所有工作都是在这个transaction2目录中进行的，后面简称它为项目目录。

0 基础工作

我们的DeepDive的数据（包括输入，输出，中间数据）全都存在关系数据库中，强烈推荐Postgres（==说明其他类型的数据库也是支持的==，这个我们之前已经安装过了）我们配置db.url文件来设置本地数据库。在transaction2 目录中打开终端，执行下面命令：

 

1
echo "postgresql://localhost/deepdive_$USER" >db.url
2
#$符号是shell中用来表示变量的，所以这个蓝色部分大概就是 用户名@主机名：。。。。
3
#deepdive_spouse_$USER 我们这个项目的数据库名，也可以自定义。
1 数据准备

数据处理要有如下几个部分：

载入原始输入数据

添加自然语言处理（NLP)标记

抽取候选关系

为每个候选关系抽取特征

1.1 载入原始数据

我们控制Deepdive操作的方法，就是在我们的项目目录中新建app.ddlog文件，把操作写入到这个文件中，语法后面会讲到。

我们需要在把articles文件放到input文件夹里面去，按照deepdive官方的Tutorial的方法，无法实现不知道为什么。（这个文件在transaction/input中）

然后在app.ddlog 中添加如下内容：

 

1
aritcles(
2
    id      text,
3
    content text
4
).
其实就是为了标记我们的articles文档中的列名，然后存到数据库中。

==每次app.ddlog变动== 后我们都要进行编译操作，也就是在项目目录执行下面的命令：

 

1
deepdive compile
他会执行当前目录下的工作的编译。

编译完成后，我们要执行

 

1
deepdive do articles
这里do 后面应该和我们app.ddlog 中的名字相同，和input 文件夹中的文件名也相同，他们三个应该都是一致的。他会把input文件中的对应的文件导入postgresql数据库中。注意这个语句执行后，他会进入一个类似vi的界面让你审查他自动生成的处理代码是不是正确，这时候输入：wq 就可以保存并退出，继续执行后面的步骤。

执行完成后，可以执行下面代码在数据库中查询，看看是不是成功导入了。

 

1
deepdive query '?- articles(id, _).'
如果成功导入，那么他就会有几行数据，如果不成功，那么就会显示0 rows ，成功会显示下面这样

 

1
id     
2
------------
3
1201835868
4
1201835869
5
1201835883
6
1201835885
7
1201835927
8
1201845343
9
1201835928
10
1201835930
11
1201835934
12
1201841180
13
:
1.2 添加自然语言标记

deepdive默认用standford nlp进行文本处理，可以返回句子的分词、lemma、pos、NER

自然语言处理 Natural Language Processing

分词：首先是中文分词，在一句话中，我们要把词分出来，而不是光看单独的子。比如 ==我== ==今天== ==很== ==高兴== 选择合适的字组成合适的词来构成句子

lemma：词元，这个是指这个词实质上的含义，比如cat,cats他们有相同词元。

pos：词性标注，最基本的是动词、名词等等

NER：Named Entity Recognition，可以识别出地名、人名、组织等等

具体内容请自行学习自然语言处理。。。
现在我们有了文章了，所以下一步就是把文章拆分成更细致的东西sentences，所以，我们在数据库中需要有一个表来存储sentences的数据。同articles，我们要在app.ddlog 中新建一个表的定义：

 

1
sentences(
2
    doc_id          text,
3
    sentence_index  int,
4
    sentence_text   text,
5
    tokens          text[],
6
    lemmas          text[],
7
    pos_tags        text[],
8
    ner_tags        text[],
9
    doc_offsets     int[],
10
    dep_types       text[],
11
    dep_tokens      int[]
12
).
光有存储用的表的定义还不够，对吧，我们还需要对数据处理的方法，deepdive只是个框架，具体要怎么处理需要我们告诉他。所以我们要定义函数来处理articles让他变成sentences。

 

1
function nlp_markup over (
2
    doc_id text,
3
    content text
4
) returns rows like sentences
5
implementation "udf/nlp_markup.sh" handles tsv lines.
这里的语法我们记住这样用就可以了

function用来定义函数，后面nlp_markup 是函数名 over后面接的是参数表。

returns 说明了函数返回的形式，返回就像我们前面定义的sentences那样的一行。

最后一句说明了我们这个程序文件是udf/nlp_markup.sh ,输入是tsv的一行。
当然，光声明了函数是不行的，我们还要调用才会起作用对不啦，所以下面就是调用函数啦。

 

1
sentences+=nlp_markup(doc_id,content) :-
2
articles(doc_id,content).
上面的+= 其实和其他语言差不多，就是对于来源是articles中的每一行的doc_id 和content 我们都调用nlp_markup 然后结果添加到sentences表中。
 

1
#!/usr/bin/env bash
2
# A shell script that runs Bazaar/Parser over documents passed as input TSV lines
3
#
4
# $ deepdive env udf/nlp_markup.sh doc_id _ _ content _
5
##
6
set -euo pipefail
7
cd "$(dirname "$0")"
8
​
9
: ${BAZAAR_HOME:=$PWD/bazaar}
10
[[ -x "$BAZAAR_HOME"/parser/target/start ]] || {
11
    echo "No Bazaar/Parser set up at: $BAZAAR_HOME/parser"
12
    exit 2
13
} >&2
14
​
15
[[ $# -gt 0 ]] ||
16
    # default column order of input TSV
17
    set -- doc_id content
18
​
19
# convert input tsv lines into JSON lines for Bazaar/Parser
20
​
21
​
22
# start Bazaar/Parser to emit sentences TSV
23
tsv2json "$@" |
24
"$BAZAAR_HOME"/parser/run.sh -i json -k doc_id -v content
所有#开头的（除了#！）都是普通注释

参数的用法(

$0：调用文件使用的文件名，带有前面的路径,

$1-∞：传给脚本的各个参数,

$@,$*：这两个都表示传入的所有参数,

$#：表示传入参数的个数)

第一行指定了脚本的执行程序

第六行指定了一些程序的错误处理方式等（详见Shell相关文档）

第七行改变当前目录到nlp_markup.py 所在目录，也就是udf 目录

第九行设置了一个变量BAZZER_HOME 他的值是bazaar 的路径

第10-13行执行/parser/target/start 文件，如果有错会不正常退出，并提示

第15-17行检查输入参数的正确性，看参数个数是不是大于0个，如果没有参数，自己设定参数名

第23-24行把全部输入的参数用tsv2json 工具转换成json 格式，然后在执行parser/run.sh 并以刚才的json 作为参数输入。
现在我们应该知道，想对文本进行分词、词性标注等等自然语言处理，仅靠我们这点代码是不够的，所以我们有了bazaar 文件夹下的程序，对于这个文件我们不过多了解，但是现在我们要知道的是，如果我们想使用它，我们需要对它进行编译。来到bazaar/parser 目录下，然后在终端中执行下面代码

 

1
sbt/sbt stage
现在，这一步的准备工作完成了，可以回到我们的transaction 目录下去编译app.ddlog 并执行相应工作

 

1
deepdive compile
2
deepdive do transaction
这一步需要耐心等待，可能要很久很久很久。

（如果只做调试测试只用，那么把input文件夹中的articles删掉一些，可以只留2-3条，作者i3电脑可以在15分钟左右完成，然后在终端执行：deepdive redo articles 和deepdive redo sentences，这两个命令可以重新执行以前执行过的命令，删除以前的数据）

==请注意==上面这一步，如果报错，就是找不到进程啊，没有文件啊类似的错误可以使用下面的办法处理

 

1
#在执行前面的两条命令之前，进入superuser的模式，使用su进行一次登录
2
#具体步骤：按照下面的代码来
3
su (username)  #这里的username是你的用户名，就是终端前面‘@’这个符号前面的部分，不带括号
4
#它会提示你输入密码，登录即可。
5
#接下来在执行前面的
6
deepdive compile
7
deepdive do sentences
执行完成之后，我们可以使用下面的命令，查询处理结果：

 

1
deepdive query '
2
doc_id, index, tokens, ner_tags | 5
3
?- sentences(doc_id, index, text, tokens, lemmas, pos_tags, ner_tags, _, _, _).
4
'    

1.3 抽取候选关系

到这里，前面都是==通用的步骤==不论我们抽取什么样的关系，什么类型的实体，我们肯定都要先对我们的文章进行处理，分词啊，标记啊啥的。但是到了这一步，我们就是要按照我们的任务去安排了。

本文档是按照前面下载的deepdive中的transaction样例来演示的，所以现在我们的任务是抽取不同公司之间的交易关系。如果你要抽取和我这里不同的东西，just对号入座、按图索骥就可以了。

既然我们要抽取公司间的交易信息，所以我们是不是应该先得到我文本中的公司是谁，才能进一步知道他们有没有关系，这一步就是要抽取这些公司啦。一共分两步

抽取候选实体

得到实体间的候选关系

1.3.1 抽取候选实体

和前面的sentence一样，我们需要建立一个数据表，来存储我们接下来会的到公司（不然放在哪）,还有用来抽取实体的方法，并且要调用这个方法。所以我们要在app.ddlog中添加下面的内容，告诉Deepdive如何处理数据

 

1
company_mention(
2
    mention_id      text,
3
    mention_text    text,
4
    doc_id          text,
5
    sentence_index  int,
6
    begin_index     int,
7
    end_index       int
8
).
9
​
10
function map_company_mention over(
11
    doc_id          text,
12
    sentence_index   int,
13
    tokens          text[],
14
    ner_tags        text[]
15
)returns rows like company_mention
16
implementation "udf/map_company_mention.py" handles tsv lines.
17
​
18
company_mention += map_company_mention(
19
    doc_id, sentence_index, tokens, ner_tags
20
) :-
21
sentences(doc_id, sentence_index, _, tokens, _, _,ner_tags, _, _, _).
现在我们应该想了，处理方式我选好了，那么这个udf/map_company_mention.py应该怎么写才行啊，我要如何从这一堆文本里得到公司的名字。好！前面1.2 已经讲过了，我们nlp处理中有一个步骤是命名实体识别(NER)，这个东西会把每个词的实体识别出来，比如公司名字就应是属于ORG类的实体。所以我们只要在每个sentence中找到其中的ner_tags 为连续的ORG标记的就可以了。

好我们来看看map_company_mention中的代码

 

1
#!/usr/bin/env python
2
#encoding:utf-8
3
from deepdive import *
4
from transform import *
5
import re
6
​
7
@tsv_extractor
8
@returns(lambda
9
        mention_id       = "text",
10
        mention_text     = "text",
11
        doc_id           = "text",
12
        sentence_index   = "int",
13
        begin_index      = "int",
14
        end_index        = "int",
15
    :[])
16
def extract(
17
        doc_id         = "text",
18
        sentence_index = "int",
19
        tokens         = "text[]",
20
        ner_tags       = "text[]",
21
    ):
22
    """
23
    Finds phrases that are continuous words tagged with ORG.
24
    """
25
    num_tokens = len(ner_tags)
26
    # find all first indexes of series of tokens tagged as ORG
27
    first_indexes = (i for i in xrange(num_tokens) if ner_tags[i] == "ORG" and (i == 0 or ner_tags[i-1] != "ORG") and re.match(u'^[\u4e00-\u9fa5\u3040-\u309f\u30a0-\u30ffa-zA-Z]+$', unicode(tokens[i], "utf-8")) != None)
28
    for begin_index in first_indexes:
29
        # find the end of the ORG phrase (consecutive tokens tagged as ORG)
30
        end_index = begin_index + 1
31
        while end_index < num_tokens and ner_tags[end_index] == "ORG" and re.match(u'^[\u4e00-\u9fa5\u3040-\u309f\u30a0-\u30ffa-zA-Z]+$', unicode(tokens[end_index], "utf-8")) != None:
32
            end_index += 1
33
        end_index -= 1
34
        # generate a mention identifier
35
        mention_id = "%s_%d_%d_%d" % (doc_id, sentence_index, begin_index, end_index)
36
        mention_text = "".join(map(lambda i: tokens[i], xrange(begin_index, end_index + 1)))
37
        temp_text = link(mention_text, entity_dict)
38
        if temp_text == None or temp_text == '':
39
            continue
40
        if end_index - begin_index >= 25:
41
            continue
42
        # Output a tuple for each ORG phrase
43
        yield [
44
            mention_id,
45
            mention_text,
46
            doc_id,
47
            sentence_index,
48
            begin_index,
49
            end_index,
50
        ]
程序分析：

不常用的知识

python中以函数定义前，@标识符表示的是python中装饰器的意思

lambda关键字，用来定义一个匿名函数,可以参考这里：Python中的Lambda

x for x in range(100) if x%2 == 0 类似这样的用法，叫做列表解析

re.match()是用来匹配正则表达式

yield：用来创建生成器，具体点Python之yield

逻辑分析：

这里，先在句子中找到连续的命名实体标记ORG的连续的词。他们其实就是公司名，但可能有的ORG不是公司名，所以在第31-41行进行了简单的过滤。然后将每个公司名进行一次返回。

1.3.2 抽取候选关系

​	候选实体已经有了，就是文中出现的公司名，我们要找的是公司之间的交易关系。所以这里候选关系简单来说，就是把不同的公司名两两组合，最终得到的关系表其实就相当于对两个候选实体表进行笛卡尔积（当然，我们还需要一些简单的过滤的处理，比如两个公司名不能相同啊等等）

笛卡尔乘积是指在数学中，两个集合X和Y的笛卡尓积（Cartesian product），又称直积，表示为X×Y，第一个对象是X的成员而第二个对象是Y的所有可能有序对的其中一个成员

假设有

​	A：a1,a2,a3		B:b1,b2

​	A×B：（a1,b1),(a1,b2),(a2,b1),(a2,b2),(a3,b1),(a3,b2)
​	好了，现在定义一个表来存储我们的候选关系吧

 

1
transaction_candidate(
2
    p1_id       text,
3
    p1_name     text,
4
    p2_id       text,
5
    p2_name     text
6
)
​	这个候选关系同样是要经过我们函数处理的，所以输入的参数和表的Schema一样，这样定义一个函数：

 

1
function map_transaction_candidate over (
2
    p1_id       text,
3
    p1_name     text,
4
    p2_id       text,
5
    p2_name     text
6
) returns rows like transaction_candidate
7
implementation "udf/map_transaction_candidate.py" handles tsv lines.
这里的函数要怎么调用呢，首先我们的输入数据应该是候选实体，又因为deepdive处理笛卡尔积这种关系还是很简单的，直接使用数据库表的操作就可以。

 

1
transaction_candidate += map_transaction_candidate(p1, p1_name, p2, p2_name) :-
2
company_mention(p1, p1_name, same_doc, same_sentence, p1_begin, _),
3
company_mention(p2, p2_name, same_doc, same_sentence, p2_begin, _),
4
p1_name != p2_name,
5
p1_begin != p2_begin.
上面的处理，要求相关联的两个候选实体，要是在同一句话里的，不再同一句话中怎么知道他们两个有关系。而且两个名字不能一样，识别的时候也有可能一个部分识别了两个，所以开始位置也不能一样。
那现在来看看map_transaction_candidate.py的内容。

 

1
#!/usr/bin/env python
2
#encoding:utf-8
3
from deepdive import *
4
import re
5
​
6
@tsv_extractor
7
@returns(lambda
8
        p1_id       = "text",
9
        p1_name     = "text",
10
        p2_id           = "text",
11
        p2_name  = "text",
12
    :[])
13
def extract(
14
        p1_id       = "text",
15
        p1_name     = "text",
16
        p2_id           = "text",
17
        p2_name   = "text",
18
    ):
19
    if not(set(p1_name) <= set(p2_name) or set(p2_name) <= set(p1_name)):
20
        yield [
21
            p1_id,
22
            p1_name,
23
            p2_id,
24
            p2_name,
25
        ]
这里多了的内容是set对象，他是集合的意思

这里把字符串给进去，得到的集合是字符串中出现的字母、汉字、符号的组合（还被去重了，就是每个东西只有一个）

比如set('hello') 转换为字符串就是{'h','e','l','o'} 只有一个'l'哦

而他们之间的a <= b操作符用来判断，是不是a中的所有元素都在b中出现。

这里就是过滤掉，可能是同一个公司名字，但是有简略写法的可能。
这一切都尘埃落定，编译并执行这个操作吧

 

1
deepdive compile && deepdive do transaction_candidate

1.4 特征提取

​	对于前面提取出来的公司间的候选关系，我们肯定是要使用机器学习的算法，通过一些我们找的训练集，让计算机去分类，哪个关系可能有交易关系（哇塞，还有机器学习 ​​。。不然呢，人工分类吗，那直接让人看文章不是更方便 ​​)，所以特征首先要得到。

​	对于自然语言来说，他的特征就是上下文，就是上下文，一般重要的事说两遍。

​	所以定义一个特征表：

 

1
transaction_feature(
2
    p1_id       text,
3
    p2_id       text,
4
    feature     text
5
).
​	这个特征明显也是抽取出来的，所以要定义一个函数来抽取它。

 

1
function extract_transaction_features over (
2
    p1_id           text,
3
    p2_id           text,
4
    p1_begin_index  int,
5
    p1_end_index    int,
6
    p2_begin_index  int,
7
    p2_end_index    int,
8
    doc_id          text,
9
    sent_index      int,
10
    tokens          text[],
11
    lemmas          text[],
12
    pos_tags        text[],
13
    ner_tags        text[],
14
    dep_types       text[],
15
    dep_tokens      int[]
16
) returns rows like transaction_feature
17
implementation "udf/extract_transaction_features.py" handles tsv lines.
然后定义如何调用这个函数，输入对应的参数：

 

1
transaction_feature += extract_transaction_features(
2
p1_id, p2_id, p1_begin_index, p1_end_index, p2_begin_index, p2_end_index,
3
doc_id, sent_index, tokens, lemmas, pos_tags, ner_tags, dep_types, dep_tokens
4
) :-
5
company_mention(p1_id, _, doc_id, sent_index, p1_begin_index, p1_end_index),
6
company_mention(p2_id, _, doc_id, sent_index, p2_begin_index, p2_end_index),
7
sentences(doc_id, sent_index, _, tokens, lemmas, pos_tags, ner_tags, _, dep_types, dep_tokens).
老规矩，现在看看这个py脚本里的内容：

 

1
#!/usr/bin/env python
2
from deepdive import *
3
import ddlib
4
​
5
@tsv_extractor
6
@returns(lambda
7
        p1_id   = "text",
8
        p2_id   = "text",
9
        feature = "text",
10
    :[])
11
def extract(
12
        p1_id          = "text",
13
        p2_id          = "text",
14
        p1_begin_index = "int",
15
        p1_end_index   = "int",
16
        p2_begin_index = "int",
17
        p2_end_index   = "int",
18
        doc_id         = "text",
19
        sent_index     = "int",
20
        tokens         = "text[]",
21
        lemmas         = "text[]",
22
        pos_tags       = "text[]",
23
        ner_tags       = "text[]",
24
        dep_types      = "text[]",
25
        dep_parents    = "int[]",
26
    ):
27
    """
28
    Uses DDLIB to generate features for the spouse relation.
29
    """
30
    # Create a DDLIB sentence object, which is just a list of DDLIB Word objects
31
    sent = []
32
    for i,t in enumerate(tokens):
33
        sent.append(ddlib.Word(
34
            begin_char_offset=None,
35
            end_char_offset=None,
36
            word=t,
37
            lemma=lemmas[i],
38
            pos=pos_tags[i],
39
            ner=ner_tags[i],
40
            dep_par=dep_parents[i] - 1,  # Note that as stored from CoreNLP 0 is ROOT, but for DDLIB -1 is ROOT
41
            dep_label=dep_types[i]))
42
​
43
    # Create DDLIB Spans for the two person mentions
44
    p1_span = ddlib.Span(begin_word_id=p1_begin_index, length=(p1_end_index-p1_begin_index+1))
45
    p2_span = ddlib.Span(begin_word_id=p2_begin_index, length=(p2_end_index-p2_begin_index+1))
46
​
47
    # Generate the generic features using DDLIB
48
    for feature in ddlib.get_generic_features_relation(sent, p1_span, p2_span):
49
        yield [p1_id, p2_id, feature]
enumerate(iter)在迭代中的作用是返回集合iter的索引序号和其value的组合

这里调用了ddlib.Word()、ddlib.Span()、ddlib.get_generic_features_relation()东西，下面一个个看

ddlib.Word():其实这个是Python使用collections.namedtuple(需要先导入collections) 定义的带有名字的格式化的元组，类似于C语言中的struct(结构体)，其中一个有八个item，就和代码里写的一样。

ddlib.Span():这个和ddlib.Word()类型相似，都是类似结构体的东西，它内部存储的item也在代码里写了。

注意：Word、Span这两个都不是方法，只是定义了数据的存储结构而已。

ddlib.get_generic_features_relation()：这个就是普通函数了，它返回（用yield的方法）给定的句子中关于p1_span和p2_span的标准的特征（就是最普通的），注意这个函数参数的类型：sent是Word类型的，p?_span要求是Span类型的。其实这个函数还有最后一个参数，可以设置提取的特征的最大长度，默认是5，所以结果最长是5个特征的。

以上三个被调用的东西，都是Deepdive定义的比较简单的东西，供用户直接调用。

具体参见~/local/lib/python/ddlib/中的源代码文件。
现在编译并执行

 

1
deepdive compile && deepdive do transaction_feature
可以执行下面的语句来查看结果

 

1
deepdive query '| 20 ?- transaction_feature(_, _, feature).'
它显示的内容就是一个特征的内容。

比如下面：

 

1
feature                        
2
————————————————————————————
3
 WORD_SEQ_[郴州市 城市 建设 投资 发展 集团 有限 公司]
4
 LEMMA_SEQ_[郴州市 城市 建设 投资 发展 集团 有限 公司]
5
 NER_SEQ_[ORG ORG ORG ORG ORG ORG ORG ORG]
6
 POS_SEQ_[NR NN NN NN NN NN JJ NN]
7
 W_LEMMA_L_1_R_1_[为]_[提供]
8
 W_NER_L_1_R_1_[O]_[O]
9
 W_LEMMA_L_1_R_2_[为]_[提供 担保]
10
 W_NER_L_1_R_2_[O]_[O O]
11
 W_LEMMA_L_1_R_3_[为]_[提供 担保 公告]
12
 W_NER_L_1_R_3_[O]_[O O O]
13
 W_LEMMA_L_2_R_1_[公司 为]_[提供]
14
 W_NER_L_2_R_1_[ORG O]_[O]
15
 W_LEMMA_L_2_R_2_[公司 为]_[提供 担保]
16
 W_NER_L_2_R_2_[ORG O]_[O O]
17
##这个是我的注释，原先没有的哦
18
##所以你看，下面最长的就是左2右3，或者左3右2的格式，最长是五个。
19
 W_LEMMA_L_2_R_3_[公司 为]_[提供 担保 公告]
20
 W_NER_L_2_R_3_[ORG O]_[O O O]
21
 W_LEMMA_L_3_R_1_[有限 公司 为]_[提供]
22
 W_NER_L_3_R_1_[ORG ORG O]_[O]
23
 W_LEMMA_L_3_R_2_[有限 公司 为]_[提供 担保]
24
 W_NER_L_3_R_2_[ORG ORG O]_[O O]
25
​
26
(20 rows)

1.5 样本打标

​	对于监督学习，必然需要标注数据，那么已标注数据是怎么来的呢？当然正经的来说，应该是我们给这个系统提供大量的我们之前已经标注好了的数据，但是现在我们没有。所以我们可以对前面几步我们抽取出来的关系，利用一些先验的数据（比如人工标记的关系，还有先验的规则）对那些关系进行标记（标注出某些标记是已知的存在交易关系的，还有已知不存在交易关系的候选关系）

​	所以对于这里来说，我们同样需要数据库中有一个表，来存储我们的被标记数据。在app.ddlog中添加

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
​	其中的label用来表示当前关系的相关性，数字越大就越相关，正数为正相关，负数为负相关。前面解释了，这个被标记数据，是我们从已经抽取出来的候选关系中进行打标得到的，对吧，在我们打标操作之前，其实我们要把我们的候选关系放到这里，然后对它们打标（因为提前也不知道谁会被标记正负）。所以这里通过数据库的操作，把我们的候选关系的公司id直接拷贝过来（在app.ddlog中添加）

transcation_label(p1,p2,0,NULL) :- transaction_candidate(p1,_,p2,_).
​	好的，那么现在应该导入我们的先验数据了，这个其实就是我们知道的有交易关系的实体对，也就是两个公司的名字，同样用一个表来存储（在app.ddlog中添加），数据文件放到input/下面

transaction_dbdata(
    @key
    company1_name text,
    @key
    company2_name text
).
​	我们应该先导入先验数据才能够进行打标，对吧，现在表已经定义好了，我们还要进行真正的导入操作。所以终端执行下面命令：

 

1
deepdive compile && deepdive do transaction_dbdata
​	下面来定义我们的打标规则，如果候选的待打标的关系中的两个公司名，和我已知的两个公司名字相同，他们就应该是有关系的，应该被标上：那是相当有关系啊！ 。所以下面进行打标的操作（在app.ddlog中添加）：

transaction_label(p1,p2, 3, "from_dbdata") :-
    transaction_candidate(p1, p1_name, p2, p2_name), transaction_dbdata(n1, n2),
    [ lower(n1) = lower(p1_name), lower(n2) = lower(p2_name) ;
      lower(n2) = lower(p1_name), lower(n1) = lower(p2_name) ].
​	上面的数据库操作就是对于公司名字和先验数据中的公司名字相同的关系，被标记为相当有关系（给的label是3，这个数越大就说明关系就越大，或者说越有可能有关系），标记规则就写来自数据“from_dbdata"就是这样。后面的部分主要应用在英文文本，使结果大小写无关（在app.ddlog中添加）。

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
​	下面调用函数（在app.ddlog中添加）：

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
老规矩，看看这个udf/supervise_transaction.py

 

1
#!/usr/bin/env python
2
#encoding:utf-8
3
from deepdive import *
4
import random
5
from collections import namedtuple
6
​
7
TransactionLabel = namedtuple('TransactionLabel', 'p1_id, p2_id, label, type')
8
​
9
@tsv_extractor
10
@returns(lambda
11
        p1_id   = "text",
12
        p2_id   = "text",
13
        label   = "int",
14
        rule_id = "text",
15
    :[])
16
# heuristic rules for finding positive/negative examples of transaction relationship mentions
17
def supervise(
18
        p1_id="text", p1_begin="int", p1_end="int",
19
        p2_id="text", p2_begin="int", p2_end="int",
20
        doc_id="text", sentence_index="int", sentence_text="text",
21
        tokens="text[]", lemmas="text[]", pos_tags="text[]", ner_tags="text[]",
22
        dep_types="text[]", dep_token_indexes="int[]",
23
    ):
24
​
25
    # Constants
26
    TRANSLATION = frozenset(["转让", "交易", "卖出", "购买","收购","购入","拥有", "持有", "卖给", "买入", "获得"])
27
    STOCK = frozenset(["股权", "股份", "股"])
28
    TRANSLATION_COM = frozenset(["持股","买股","卖股"])
29
    TRANSLATION_AFTER = frozenset(["融资", "投资"])
30
    OTHERS_AFTER = frozenset(["产品", "委托", "贷款", "保险"])
31
    OTHERS_COM = frozenset(["购买", "提供", "申请", "销售"])
32
    CONF = frozenset(["对", "向"])
33
    OTHERS = frozenset(["销售产品", "提供担保","提供服务"])
34
​
35
​
36
    COMMAS = frozenset([":", "：","1","2","3","4","5","6","7","8","9","0","、", ";", "；"])
37
    MAX_DIST = 40
38
​
39
    # Common data objects
40
    p1_end_idx = min(p1_end, p2_end)
41
    p2_start_idx = max(p1_begin, p2_begin)
42
    p2_end_idx = max(p1_end,p2_end)
43
    intermediate_lemmas = lemmas[p1_end_idx+1:p2_start_idx]
44
    intermediate_ner_tags = ner_tags[p1_end_idx+1:p2_start_idx]
45
    tail_lemmas = lemmas[p2_end_idx+1:]
46
    transaction = TransactionLabel(p1_id=p1_id, p2_id=p2_id, label=None, type=None)
47
​
48
    # Rule: Candidates that are too far apart
49
    if len(intermediate_lemmas) > MAX_DIST:
50
        yield transaction._replace(label=-1, type='neg:far_apart')
51
​
52
    # Rule: Candidates that have a third company in between
53
    if 'ORG' in intermediate_ner_tags:
54
        yield transaction._replace(label=-1, type='neg:third_company_between')
55
​
56
    # Rule: Sentences that contain wife/husband in between
57
    
58
    if len(TRANSLATION_COM.intersection(intermediate_lemmas)) > 0:
59
        yield transaction._replace(label=1, type='pos:A持股B')
60
​
61
    if len(TRANSLATION.intersection(intermediate_lemmas)) > 0 and len(STOCK.intersection(tail_lemmas)) > 0:
62
        yield transaction._replace(label=1, type='pos:A购买B股权')
63
    
64
    if len(TRANSLATION_AFTER.intersection(intermediate_lemmas)) > 0:
65
        yield transaction._replace(label=1, type='pos:A增资B')
66
​
67
    if len(OTHERS_COM.intersection(intermediate_lemmas)) > 0 and len(OTHERS_AFTER.intersection(tail_lemmas)) > 0:
68
        yield transaction._replace(label=-1, type='neg:A购买B的产品')
69
​
70
    if len(CONF.intersection(intermediate_lemmas)) > 0 and len(OTHERS.intersection(tail_lemmas)) > 0:
71
        yield transaction._replace(label=-1, type='neg:A向B提供担保')
72
​
73
    # Rule: Sentences that contain and ... married
74
    #         (<P1>)(and)?(<P2>)([ A-Za-z]+)(married)
75
    if ("与" in intermediate_lemmas) and ("签署股权转让协议" in tail_lemmas):
76
        yield transaction._replace(label=1, type='pos:A和B签署股权转让协议')
77
        
78
    if len(COMMAS.intersection(intermediate_lemmas)) > 0:
79
        yield transaction._replace(label=-1, type='neg:中间有特殊符号')
80
​
81
​
上面我对源文件删掉了一些无效的注释

但是就这样其实可以看出来，这个代码中的程序就是一堆if..else..的结构，进行普通的规则的判断，然后给出标记和所使用的规则。整体的处理都是对于上下文（或者说特征的判断）

没有什么特殊的地方了，每个领域都有领域自身的一些能体现在文本上的规则。
因为上面的打标过程，可能多个规则都对同一个关系打标了，但是他们打标的值不同，所以这里要把这种同一个关系的都整合起来，让他们的label相加，得到最终的标记数据（在app.ddlog中添加）：

 transaction_label_resolved(p1_id, p2_id, SUM(vote)) :-
 transaction_label(p1_id, p2_id, vote, rule_id).
现在，打标的一系列操作我们都定义好了，接下来就执行打标吧，刺激不？在终端执行下面代码：

 

1
deepdive compile && deepdive do transaction_label_resolved

2 模型构建

通过前面的步骤，我们已经将需要用到的数据都准备好了，下面构建我们的模型。

2.1 模型构建

(1). 定义最终存储的表格，表后面的问好?表示此表是用户模式下的变量表，即需要推导关系的表。这里我们预测的是公司间是否存在交易关系。

@extraction
has_transaction?(
    p1_id text,
    p2_id text
).
(2). 根据1.5中的打标结果，把已经标记了的数据，输入到has_transaction表中。

has_transaction(p1_id, p2_id) = if l > 0 then TRUE
                  else if l < 0 then FALSE
                  else NULL end :- transaction_label_resolved(p1_id, p2_id, l).
此时变量表中的部分变量label已知，成为了先验变量。                  

(3). 最后编译执行决策表：

$ deepdive compile && deepdive do has_transaction
​    

2.2 因子图构建

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

p1_id	p2_id	expectation
1201778739_118_170_171	1201778739_118_54_60	0
1201778739_54_30_35	1201778739_54_8_11	0.035
1201759193_1_26_31	1201759193_1_43_48	0.07
1201766319_65_331_331	1201766319_65_159_163	0
1201761624_17_30_35	1201761624_17_9_14	0.188
1201743500_3_0_5	1201743500_3_8_14	0.347
1201789764_3_16_21	1201789764_3_75_76	0
1201778739_120_26_27	1201778739_120_29_30	0.003
1201752964_3_21_21	1201752964_3_5_10	0.133
1201775403_1_83_88	1201775403_1_3_6	0
1201778793_15_5_6	1201778793_15_17_18	0.984
1201773262_2_85_88	1201773262_2_99_99	0.043
1201734457_24_19_20	1201734457_24_28_29	0.081
1201752964_22_48_50	1201752964_22_9_10	0.013
1201759216_5_38_44	1201759216_5_55_56	0.305
1201755097_4_18_22	1201755097_4_52_57	1
1201750746_2_0_5	1201750746_2_20_26	0.034
1201759186_4_45_46	1201759186_4_41_43	0.005
1201734457_18_7_11	1201734457_18_13_18	0.964
1201759263_36_18_20	1201759263_36_33_36	0.002
至此，我们的交易关系抽取就基本完成了。更多详细说明请见http://deepdive.stanford.edu


第二部分 详细讲解

1 背景知识

1.1 因子图



上面就是一个因子图（Factor Graph）的例子（来自维基百科），因子图是一种概率图模型，Deepdive的概率推测（Probabilistic inference）就是在这个模型上执行的。图我们都知道，由节点(node)和弧或者叫边(edge)组成，因子图的节点由两种类型：变量（Variables）和因子（Factor），详细如下：

Variables：如果这种结点的值已知，它就可以当作一个证据变量（用来推断别的值）；这个值也可以是未知的，这事就叫做查询变量，就是我们需要进行预测得到的值，比如图中的​；

Factor：每个因子都可以连接到多个变量，并用因子函数定义它们之间的关系，比如图中的​ ；每个因子（或者说因子函数）都有一个权重值，来表示这个因子影响力的大小。换个说法，这个权重值表示了某个因子的可信程度，正数越大则越正确，负数越小则越错误（错误表示某种不可能，比如一个人的亲儿子同时是他的亲兄弟，这就是一个错误，所以这个因子的权重就应该是一个很小的负值）。因子函数的权重可以通过训练学习得到，也可以手动赋值（通过脚本或者app.ddlog）。

1.2 可能的世界

一个可能的世界（Possible  World）就是对因子图中的每个变量的取值，很多个可能的世界并不是等概率的，每个世界都有不同的概率。这个可能的世界，用我们的话来说就是一个“如果的世界”，就像下面这两句话：

如果他那天出门带伞了
如果他那天出门没带伞
假设我们不知道这个人到底带没带伞，而且天气预报说那天会下雨，这就是两个可能的世界，里面的事情都是我们假象的，既然是假设的那么就有了概率（也就是可能性的大小），这个人关注天气，生活细致，那么他带伞的概率就很大，不带伞的概率就很小。

可能世界的概率与因子图中所有因子函数的加权组合成比例，权重可以静态分配或自动学习。 如果训练的话，需要一些训练数据。 训练数据定义了一组可能的世界，训练过程，就是在给定的世界中，使概率达到最大的过程。

1.3 边际推理

边际推理（Marginal inference）就是要推断：一个变量取某个特定值的概率是多少（比如骰子6点向上的概率​​）

在这个任务中，可以使用全概率公式对这个变量取值的所有情况的概率的和（sum）来简单地表示这个我们要求的概率。

全概率公式：​  注意这里的事件​要相互独立且满足 ​

精确推理是因子图的一个难题，这个问题比较常用的方法是吉布斯抽样。这个解决过程首先要从随机的可能世界开始，然后对每一个变量（v）进行迭代更新，更新的时候要考虑到这个变量所连接的因子结点的因子函数和连接到这些因子结点的变量的取值（这算是前面变量v的马尔科夫毯）。

当对这些随机变量进行了很多次迭代后，我们可以计算变量某个特定值出现次数占总迭代次数的比例，这个比例就是对个变量的这个特定取值的概率（根据大数定律​​）。

Gibbs Sampling 吉布斯抽样

1.4 DeepDive中的推理

DeepDive允许用户自己定义建立因子图的推理规则，那么，规则可以表达下面这种意思，

”如果一次吃的多了，可能会消化不良。“
综合前面讲的因子图，也就是说，一个规则描述了一个因子结点的因子函数和这个结点所连接的变量。每一条规则都有权重（同样可以用户人工赋值或者由DeepDive计算出来），这个权重表示了这条规则的可信度。

这是一个微妙但非常重要的点。许多传统的机器学习算法，它通常假定的先验知识是精确的和孤立的进行预测，与之相反，deepdive进行联合推理：它同时决定所有事件的值。如果事件通过推理规则（直接或间接）连接，则允许事件相互影响。因此，一个事件的不确定性（约翰吸烟）可能影响另一事件的不确定性（约翰患有癌症）。随着事件之间的关系变得更加复杂，这个模型变得非常强大。例如，人们可以想象“约翰吸烟”的事件是否受到约翰是否有朋友吸烟的影响。在处理固有噪声信号，如人类语言时，这一点尤其有用。

下面是一个完整的DeepDive项目的流程与结构：



1.5 弱监督

大多数机器学习技术需要一组训练数据。收集训练数据的传统方法是人工标记一组文档。例如，对于婚姻关系，标记者可能把“Bill Clinton”和“Hillary Clinton”标记为一个正面的训练实例。这种方法在时间和金钱上都是很昂贵的，如果我们的语料很大，都无法为我们的算法提供足够多的处理数据。而且由于人类会犯错误，由此产生的训练数据很可能是充满噪声的。

生成训练数据的另一种方法是弱监督（Distant supervision）。弱监督中，我们使用一个已经存在的数据库，如Freebase或特定领域的数据库，收集我们要提取的关系的例子。然后我们使用这些例子来自动生成我们的训练数据。例如，Freebase中包含了Barack Obama和Michelle Obama已经结婚的事实。我们利用这个事实，然后将出现在同一个句子中的每一个“Barack Obama”和“Michelle Obama”标记为我们婚姻关系的一个正面的例子。这样我们就可以很容易地生成大量的（可能是嘈杂的）训练数据。运用弱监督来为某一特定关系找到正面的例子是容易的，==但是产生否定的例子可要难得多了==。

1.6 DeepDive 系统概述和相关术语

本文档讲DeepDive作为一个系统来概括。现在假设你已经熟悉一些一般概念，如推理和因子图、关系抽取和弱监督。下面介绍了执行DeepDive应用过程中的每一个步骤：

提取

DeepDive首先要对数据进行实体抽取、实体链接、特征抽取、弱监督和其他建立后面用来执行推断的变量的必要步骤，这些工作就是提取步骤。如果需要的话，还要用数据来生成用来学习因子权重的训练数据。这些提取中的小步骤都是通过定义提取器（extractor）来制定的（这个提取器也可以通过用户定义函数（user-defined functions)来指定，就是放在udf文件夹中的脚本）。提取结果存储在应用程序数据库中，然后将根据用户指定的规则构建因子图。

因子图的固化（Grounding，==不太清楚怎么翻译==） 

DeepDive用因子图进行推理。用户编写SQL查询来指示系统要创建哪些变量，这些查询通常涉及在提取步骤期间填充的表。因子图的变量结点根据用户指定的推理规则连接到因子结点，并定义了描述变量之间关系的因子函数。用户可以指定系数权重应该是用户给定的常数还是由系统学习。

固化是将因子图写入磁盘的过程，以便它可以用于执行推理。DeepDive把一个因子图存储为五个文件：一个变量文件，一个因子文件，一个弧文件，一个权重文件，和一个对这个系统有用的元数据文件。这些文件的格式是特殊的，以便它们可以被我们的==采样器==接受为输入。

权重的学习

deepdive可以通过弱监督从训练数据学习因子图的权重或由用户指定这些值，填充数据库提取阶段。学习这些权重的一般方法是极大似然法。

学习好的权重接下来会写入一个特定的数据库表，以便用户在校准过程中检查它们。

推理

最后一步是在因子图变量上进行边际推理，来学习他们在可能世界 中不同取值的概率。DeepDive使用自己的高通量的DimmWitted采样器（在MacBook上，比DeepDive默认的采样器快十倍）进行吉布斯抽样，遍历许多可能世界 并估计概率。采样器将固化的图（即前面说的五个文件）作为输入，连同一些参数来指定学习过程的参数。推理的结果被写入数据库。用户可以编写查询来分析结果。DeepDive还提供校准数据来评估推理的准确性。

1.7 知识库构建

知识库的构建（Knowledge Base Construction，KBC）过程，就是用从数据（数据可以使文本、声音、视频、图标等）中提取出来的事实（或者说声明、断言）填充知识库。比如，我们实验室正在构建的药物和其不良反应的知识库，当然还有一些其他的方面。比如可以构建一个古生物知识库来了解恐龙这一物种在什么时间何地生存过，人际关系知识库来表达父母、配偶、兄弟姐妹等关系。DeepDive就是用来简化知识库构建的过程。

举一个具体的例子，我们可能想要从网络上的文本中抽取配偶关系，那么我们就可以构建一个DeepDive应用。下图就展示了DeepDive在应用中的作用。其输入是像“U.S President Barack Obama's wife Michelle Obama ...”这样的句子，它的输出是由三元组组成的一个叫做has_spouse的表，表里的每一项都表示了一组配偶关系。DeepDive同时还为这些和这些关系相关的概率，来表示这条关系可信程度

 
<center>Figure 1: An example KBC application</center>

KBC中的术语：KBC给它操作的对象起了些特别的名字，下面是比较重要的几个

实体（Entity）：一个实体就是现实世界中的一个对象，比如一个人，一个动物，地理位置，甚至一段时间都算。比如“奥巴马”就是一个实体。

Entity-level data are relational data over the domain of conceptual entities (as opposed to language-dependent mentions), such as relations in a knowledge base, e.g. "Barack Obama" in Freebase.

A mention is a reference to an entity, such as the word "Barack" in the sentence "Barack and Michelle are married".

Mention-level data are textual data with mentions of entities, such as sentences in web articles, e.g. sentences in New York Times articles.

实体链接（Entity Linking）：找到一个Mention所对应的实体。比如“美国总统”，“特朗普”，“川普”可能都是说的同一个实体“美国总统特朗普”，而实体链接的任务就是发现这些关系。

A mention-level relation is a relation among mentions rather than entities. For example, in a sentence "Barack and Michelle are married", the two mentions "Barack" and "Michelle" has a mention-level relation of has_spouse.

An entity-level relation is a relation among entities. For example, the entity "Barack Obama" and "Michelle Obama" has an entity-level relation of has_spouse.

上面这些概念之间的关系如下图所示：

 
<center>Figure 2: Data model for KBC</center>

KBC数据流（Data Flow）

在一个典型的KBC应用中，该系统的输入一般是一系列普通的文本，系统的输去是一个数据库（也就是知识库），这个知识库中包含了你需要的关系（实体级或者Mention级）。

就像前面第6部分的系统概述中所说，获取输出的步骤如下：

数据预处理

特征抽取

使用陈述性语言生成因子图

统计推断和学习

这里要特别说明的是：

在数据预处理步骤中，DeepDive接受文本格式的输入数据，把它们载入数据库，并且进行分词，断句等操作，得到每个句子中每个词的POS、NERtag等等特征。

在特征抽取步骤中，DeepDive运行开发者定义的extractor函数来把输入数据转换为被称作证据（Evidence）的关系信号。证据有两部分：

实体级或者Mention级的候选关系；

这些候选关系的（语言）特征。

下面的步骤，DeepDive使用Evidence来生成因子图。为了指示DeepDive如何构建这个因子图，开发者需要使用一个类似SQL的陈述性语言来指定推理规则。

再接下来，DeepDive自动在生成的因子图上执行学习和统计推理。在学习过程中，计算得到推理规则中指定的因子权重。这些权重直接地表示了规则的可信度。在推理过程中，会计算得到变量的联合概率，这个概率表示了特定“事实”为真的可能性。

推理结束后，其结果被存储在几个数据库表中。开发者可以通过SQL查询来获取结果，用校准图检查结果，或者执行错误分析来改进结果。

下图是一个例子，他展示了把句子“U.S President Barack Obama's wife Michelle Obama...”经过上面说的前三步处理的过程：

 
<center>Figure 3: Data flow for KBC</center>

在上面的例子中：

在数据预处理时，句子被分词，之后进行词性标注、命名实体识别处理；

特征抽取过程中，DeepDive提取了这些内容：

提到的人和出现的位置；

配偶关系has_spouse的候选关系；

这些候选关系的feature（特征），比如Mention之间的词。

DeepDive使用用户定义的规则来构建因子图。

2 构建DeepDive应用

前面我们已经了解了DeepDive基本的工作流程，那么出了第一部分的例子，我们应该如何着手构建一个DeepDive的应用呢？

这里我们先要明白一点，没有任何一个DeepDive应用是一蹴而就的（甚至可以说是任何机器学习任务），一开始的准确率都不会很高很高，所以一般的情况是：我们先开发一个“垃圾”的初期版本，然后通过错误分析和调试迭代更新来提高质量。DeepDive中带着一打工具可以使用，可以让你尽量快地开发出一个可用的应用（当然最重要的还是使用工具的你，剑本无不同，只因御剑之人不同）。简单来说，构建一个DeepDive就分三步：写流程和函数、运行、分析和调试...然后不停循环迭代。

2.1 写什么，怎么写

2.1.1 DeepDive应用的结构

前面我们只介绍了DeepDive的运行过程，现在来具体说说，DeepDive的应用从文件的角度来看是什么样的。

先来谈谈我们见过的其他应用是什么样子的，Android的应用打眼一看是.apk后缀的文件，其内部结构其实和一般JAVA工程没有本质的区别（至少以前是这样，因为作者现在不学Android了）。现在说DeepDive，一个应用就是一个独立的文件夹，这个文件夹和该目录下所有文件（包括文件夹）的总和就是一个DeepDive应用。

这个文件夹下面的大体内容如下：

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
好，看完上面的结构，我们应该有了一个大概的认识，app.ddlog、db.rul、udf/中三部分的内容是要我们自己往里面写东西的。后面我们就主要围绕它们来介绍。

2.1.2 在app.ddlog定义数据流

这里，我们要用到的语言是专为DeepDive打造的叫做DDlog的语言，其语法类似Datalog ，我们在第一部分的实例中已经见识过了。这一小部分就只介绍如何定义数据流，其他部分在后面有介绍。

DDlog中，所有的语句都要用英文句号.结束，而注释用井号#开头。而且其代码先后顺序没有要求。

定义数据表Schema

首先要定义我们在DeepDive中读入的数据的结构（和数据文件的结构相同），和我们在应用中用到的结构（比如第一部分中的sentences表）。这里有统一的格式：

 

1
relation_name(
2
    column1 type1,
3
    column2 type2,
4
    ...
5
).
首先是表名，其内部结构在小括号内表示，每一个属性之间用逗号,分隔，最后一项后面没有逗号。还要记得，每个完整的定义后面都要带英文句号。其中的类型可以使底层数据库支持的任意格式，因为DeepDive会把这里定义的表映射到数据库上去。常用的类型已经在第一部分使用过了，更多的类型可以查看相应数据库支持的类型。比如在我们前面使用的PostgreSQL数据库中，支持的格式：PostgreSQL's types

<p id="normal-rule">基本的推导规则</p>

前面定义了数据，现在看看如何让我们的数据流动起来（就是让数据经过处理得到新的内容的部分，但是现在还是预处理的部分）

这里我们使用类似Datalog的语言来定义从某些关系中推导新关系的方法。比如下面就是通过关系R、S 来推导关系Q：

 

1
Q(x,y) :- R(x,y) , S(y)
具体来讲，比如关系R就是双亲，y是x的双亲，关系S就是男性。根据这两个关系，就能知道关系Q可以表示父亲 关系，y是x的父亲。这就是一个简单推理。

上面的规则中，Q(x,y)被称为Head Atom，R(x,y)、S(y)被称为Body Atom。x、y是规则中的变量。而上面整个句子可以看作一个表达式，下面给出了一些表达式的例子：

 

1
a(k int).
2
b(k int, p text, q text, r int).
3
c(s text, n int, t text).
4
​
5
Q("test", f(123), id) :- a(id), b(id, x,y,z), c(x || y,10,"foo").
6
​
7
R(TRUE, power(abs(x-z), 2)) :- a(x), b(y, _, _, _).
8
​
9
S(if l > 10 then TRUE
10
else if l < 10 then FALSE
11
else NULL end) :- R ( _, l).
从上面的例子我们可以看出，在Atom中，我们可以使用常量（比如“test”，True ），也可以使用函数（比如f(123)）还有前面提到的变量。而且还支持简单的if_then_else 结构。

在这里支持的常量类型有这些：

数字类型：123，-3.23

字符串类型：注意，句子中的引号和关闭小括号)之前都要加转义字符\否则就会出问题。

布尔类型：False、True

空类型：NULL

在表达式间可以使用操作符，就像例子中写的有两种情况expr op expr和op expr见下面：

二元操作符：

数字运算：+、-、*、/

字符串链接：||

一元操作符：

负号：-

布尔反：!

表达式可以嵌套，适当的时候要使用小括号来让运算顺序更清晰。

曾经在书上见过这样一句话，写代码的时候，永远不要考虑运算符优先级，在一切有可能混淆的地方，使用括号来规定运算顺序。
其实还有比较运算符：

等式：= 这里和常见语言语法有点差别，使用一个等号

不等于：!=

不等：<、<=、>=、>

空：IS NULL、IS NOT NULL

字符串比较：exprText LIKE exprPattern

函数调用和条件表达式在前面的例子中已经很清晰了，下面说说类型转换。

在学习其他编程语言的时候类型转换已经接触很多了，在DDlog中的用法这样：expr :: type:

 

1
num_occurs / total :: FLOAT
2
#否则会执行整数的除法，得到的结果还是整数
另外，在规则中，经常会有关系中的某些变量在推导过程中用不到，那么这时就有了占位符（Placeholder）。一个占位符就是一个下划线_，它只可以用在Body Atom中，表示这一列的变量用不到，就像下面这个：

Q(x, y, x + y) :- R(x, _, _, y, _, _, _).
#这里关系Q的推导，只和关系表R中的第一列和第四列有关系
条件

条件和Body Atiom同级，使用其中用到了的变量。相连接的条件用逗号分隔，不连接的条件用分号分隔，感叹号可以用来否定一个条件。条件可以用中括号[]任意嵌套，比如下面的例子：

Q(x) :- b(x, _, _, w), [ x + w = 100; [ !x > 50 , w < 10 ]].
析取（Disjunctions）

Disjunction可以使用多条相同Head Atom的规则来表示，也可以在一条规则中的Body Atom用分号分隔多个bodies。可以像下面这样：

Q(x, y) :- R(x, y), S(y).
Q(x, y) :- T(x, y).
#也可以像下边这样
Q(x, y) :- R(x, y), S(y); T(x, y).
#它们的意义相同

聚合函数（Aggregation）

就像普通的SQL语句一样，DDlog中也有聚合函数的存在，它可以在各个独立的取值上进行一些数学操作，比如求某列的最大值等等，DDlog中有如下几个聚合函数：

SUM

MAX

MIN

ARRAY_AGG

COUNT

下面的例子中，Q提取了R中第一列和第二列所有的值作为Q的第一列和第二列（相同的被视为同一组），然后Q的第三列是对应到R的每组中的第三列的最大值：

Q(a,b,MAX(c)) :- R(a,b,c).
去重

为了只从其他关系中得到不重复的元素，可以使用*:-操作符：

Q(x,y) *:- R(x, y).
上面的例子中，只有R中不重复的(x,y)才会插入到Q中。

量词

存在量词和全程量词

另外一个减少头部获得的三元组的方式是使用量词结构。在Body Atom和条件结构中，可以被量词EXISTS或者ALL约束，其语法如下：

body, ..., EXISTS[body, body, ...], body, ...
body, ...,    ALL[body, body, ...], body, ...
在量词内部的关系表示了对于同时在内部和外部出现的变量进行的约束。EXISTS表示的是：对于所约束的变量，必需存在一种满足约束内部关系的组合；ALL表示：对于所约束的变量，其所有组合都必需满足约束中的内容。这样就让我们可以定义更加复杂的推理工作，比如下面

P(a) :- R(a, _), EXISTS[S(a, _)].
Q(a) :- R(a, b),    ALL[S(a, c), c > b].
第一个例子，只要R中的第一列的值同样也出现在S的第一列，那么就可以加入P

第二个例子，要求R中第一列的值，在对应S中第一列的值和其相同，而对应的所有第二列的值都要大于R中第二列的值，只有满足这些条件，才能插入Q中。

条件量词

OPTIONAL量词，表示在其内部的条件可以不满足，照样加入到Head Atom中，就像SQL中的outer joins 一样，如果一个变量只在条件量词的内部出现，那么如果没有满足条件的值的情况下，Head中的对应值就是NULL。具体可以看下面的例子：

Q(a, c) :- R(a, b), OPTIONAL[S(a, c), c > b].
上面都是，Q的第一列和R的第一列相同，如果还存在S(a,c)，而且c>b，那么Q的第二列就为c的值。如果S中不存在c与a对应，或者说c不大于b，那么Q中的第二列的值就为NULL

2.1.3 用Python编写用户定义函数（UDF）

除了用DDlog写简单的推理规则，DeepDive还支持用户定义的函数，这叫做UDFs，被放置于DeepDive项目的udf/文件夹中，这些函数也用来进行数据的处理。UDF可以是任何一个接受制表符分隔的TSJ格式文件或者制表符分隔的标准输入流中的输入，然后他的输出也是同样的格式。TSJ格式要比单纯的每一行一个JSON对象，然后重复相同的名字要高效的多。

下面的内容展示了DeepDive推荐的如何写合适的Python脚本以及如何在DDlog中使用它。

在DDlog中使用UDF

首先定义一些数据模式，方便下面的例子使用：

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
假设我们想写一个函数来吧每个articles都给一个topic，下面是具体：

函数声明

函数的声明就像C、JAVA一样，需要有参数类型、返回值类型等：

function <function_name> over (<input_var_name> <input_var_type>,...)
    returns [(<output_var_name> <output_var_type>,...) | rows like <relation_name>]
    implementation "<executable_path>" handles tsj lines.
通过上面的语法，如果我们使用article关系中author和words属性来决定这个文章主题是什么，而脚本文件我们放在udf/classify.py。那么具体的声明就是这样：

function classify_articles over (id int, author text, words text[])
    returns (article_id int, topic text)
    implementation "udf/classify.py" handles tsj lines
需要注意的是，returns语句后面可以接输出的每个属性的组合，也可以直接使用之前定义好的模式（关系）：就像上面和下面的关系

function classify_articles over (id int, author text, words text[])
    returns rows like classification
    implementation "udf/classify.py" handles tsj lines.
函数的调用方法

函数的调用和赋值就好像我们前面说过的基本推理规则类似的格式，下面使用添加的方法使用函数返回的数据来填充classification关系。

classification += classify_articles(id, author, words) :-
    article(id, _, _, author, words).
:-后面的就是函数的参数名绑定，同样还有前面说过的占位符。

使用Python写UDF

DeepDive提供了一个用Python写用户定义函数的模板。实际上提供的是几个 Python 装饰器 来简化输入和输出的格式. 被调用的用来处理每一行输入数据的Python生成器需要被@tsj_extractor装饰（被放在def关键字前面的一行）。关于生成器可以参考我的一篇博客Python yield 。生成器中输入和输出中每个参数的类型可以通过定义函数名的参数默认值来给定，并且用@returns来装饰。他们定义了具体的输入输出操作，而在我们自己只需要写一个被调用的生成器（函数），通过参数获取输入，通过yield返回每一行结果。

下面是一个简单的例子，也就是上面我们提到的udf/classify.py分类的操作的程序：

 

1
#!/usr/bin/env python
2
from deepdive import *  # Required for @tsj_extractor and @returns
3
​
4
compsci_authors = [...]
5
bio_authors     = [...]
6
bio_words       = [...]
7
​
8
@tsj_extractor  # Declares the generator below as the main function to call
9
@returns(lambda # Declares the types of output columns as declared in DDlog
10
        article_id = "int",
11
        topic      = "text",
12
    :[])
13
def classify(   # The input types can be declared directly on each parameter as its default value
14
        article_id = "int",
15
        author     = "text",
16
        words      = "text[]",
17
    ):
18
    """
19
    Classify articles by assigning topics.
20
    """
21
    num_topics = 0
22
​
23
    if author in compsci_authors:
24
        num_topics += 1
25
        yield [article_id, "cs"]
26
​
27
    if author in bio_authors:
28
        num_topics += 1
29
        yield [article_id, "bio"]
30
    elif any (word for word in bio_words if word in words):
31
        num_topics += 1
32
        yield [article_id, "bio"]
33
​
34
    if num_topics == 0:
35
        yield [article_id, None]
这个函数简单地为每一个文章赋予了一个主题属性。首先看其作者信息。如果不知道作者具体信息，那么就在文章词汇里面找是否有我们提前准备的字典的词出现。最后如果都没有合适的，直接给一个None。

这里面需要注意的就是，如果想使用前面提到的两个装饰器，那么必须要导入DeepDive的Python包：

 

1
from deepdive import *
然后在@returns装饰器后面要带上输出的数据的数量和格式。

@tsj_extractor装饰器

这个装饰器是用户写的函数的第一个装饰器（最外层），它可以处理数据的输入，然后将列表作为返回值。同时这个装饰器也是用来告诉DeepDive需要调用哪一个函数（在一个Python文件中可以定义很多函数）。还有一点DeepDive的UDF不仅能处理TSJ格式，也能处理TSV格式，这就需要另一个装饰器@tsv_extractor，但是强烈推荐TSJ格式。

注意事项

一般来说，用户定义的生成器（函数）要放在程序文件的最后，除非在处理完所有输入数据还有其他任务要完成。而且，所有被装饰的函数所用到的第三方函数和变量，都要在装饰器之前来定义。

在生成器中，请不要使用普通的pring或者sys.stdout.write来进行命令行的输出，请使用print >> sys.stderr或者sys.stderr.write来代替，更详细的记录信息的问题请看后面的调试部分。

参数默认值和@returns装饰器

为了正确地解析每行TSJ数据为合适的Python数据类型，并把返回值正确的转换成TSJ格式，我们需要做Python程序中给出其对应的每个数据列的类型。这跟应该是和DDlog中声明的函数参数和返回值类型相同的。输入数据的类型可以直接在函数的参数默认值中给定，就像上面的例子中那样。输出数据的格式在@returns装饰器中定义，即可以是作为参数的一个name-type对列表，也可以是将数据类型作为默认参数的一个函数。使用lambda是个更好的选择，因为使用列表的话，需要很多其他的声明，这里lambda是匿名函数的意思，具体参考可以看Python中的Lambda。

如果是用列表的话，上面的例子中可以写成这样：

 

1
@returns([("article_id", "int"), ("topic", "text")])
可能同学会问，为什么这里不是用Python中的字典类型呢，那不是看起来更好？这是因为在Python中，它不会考虑到其内部的顺序，但是我们的输入数据，第一列第二列都是有顺序的，所以普通的字典不可以，这里只能使用列表类型。

使用函数默认值的方式的话，@returns装饰器中的函数永远不会被调用，所以我们直接给一个匿名函数就好，然后其函数体可以任意一个简单表达式，这里使用的一个空列表作为其返回值，当然用你想用的任意一个都可以。DeepDive也提供了一个像@returns一样的用来定义输入数据类型的装饰器：@over，你可以看见这个词和DDlog中的一样，但是非常不推荐这样，既然函数参数默认值已经可以实现了，何必要再搞点多余的东西呢，这个只要知道有这个即可。

运行和调试UDFs

当一个用户定义函数写好了以后，可以使用 deepdive do 或者 deepdive redo 命令来运行，第一次运行可以使用第一种，再运行就要使用第二种了。之前我们定义的提取类型关系的，就可以使用下面这条命令来执行。

deepdive redo classification
这条命令会使用udf/classify.py这个Python程序，给其TSJ格式作为其输入，然后将返回值添加到数据库的classification数据表中。

这一节主要是关于程序的编写，后面会专门讲到UDFs的运行和调试。

2.1.4 使用DDlog制定统计模型

每个DeepDive应用，都可以看作是一个统计推理问题，其输入是把普通的输入数据经过一系列处理后的东西。这一节有3部分：

介绍了如何声明DeepDive应用统计模型中的随机变量

如何定义随机变量的监督标记

如何为特定的特征和关系写下推理规则

2.1.3.1 Variable declarations

DeepDive要求用户自己在app.ddlog中声明变量关系 的名字和类型，其之和普通的关系语法有一点点差别，关系变量中存储了在概率推理时用到的随机变量。现在DeepDive支持布尔变量和类别变量。

布尔类型

在关系名字的后面加一个问号，说明这是一个包含了随机变量的变量关系，以此来和普通关系区分开。变量关系的列元素

A question mark after the relation name indicates that it is a variable relation containing random variables rather than a normal relation used for loading or processing data to be later used by the model. The columns of the variable relation serve as a key. The following is an example declaration of a relation of Boolean variables.

has_spouse?(p1_id text, p2_id text).

This declares a variable relation named has_spouse where each unique pair of (p1_id, p2_id) represents a different random variable in the model.

Categorical variables

DeepDive supports categorical variables, which take integer values ranging from 0 to a user-specified upper bound. The variable relation is declared similarly as a Boolean variable except that the declaration is followed by a Categorical(N) where N is the number of categories the variables can take, defining the size of the domain. Each variable can take values from 0, 1, ..., N-1. For instance, in the chunking example, a categorical variable of 13 possible categories is declared as follows:

tag?(word_id bigint) Categorical(13).

Scoping and supervision rules

After declaring a variable relation, its scope needs to be defined along with the supervision labels. That means, (a) all possible values for the variable relation's columns must be defined by deriving them from other relations, and (b) whether a random variable in the relation is true or false (Boolean), or which value it takes from its domain of categories (Categorical) must be defined using a special syntax. For these scoping and supervision rules, a syntax very similar to normal derivation rules is used for defining the variable relation, except that the head is followed by an = sign and an expression that corresponds to the random variable's label. For instance, in the spouse example, we scope the has_spouse variable by the following rule:

has_spouse(p1_id, p2_id) = NULL :-
    spouse_candidate(p1_id, _, p2_id, _).

This means all distinct p1_id and p2_id pairs found in the spouse_candidate relation is considered the scope of all has_spouse random variables in the model. By using a NULL expression on the right hand side, they are considered as unsupervised variables.

On the other hand, the following similar looking rule provides the supervision labels using a sophisticated expression.

has_spouse(p1_id, p2_id) = if l > 0 then TRUE
                      else if l < 0 then FALSE
                      else NULL end :- spouse_label_resolved(p1_id, p2_id, l).

This rule is basically doing a majority vote, turning aggregate numbers computed from spouse_label_resolved relation into Boolean labels.

Inference rules

Inference rules specify features for a variable and/or the correlations between variables. They are basically the templates for the factors in the factor graph, telling DeepDive how to ground them based on what input and derived data. Again, these rules extend the syntax of normal derivation rules and allow the type of the factor to be specified in the rule's head, preceded by a @weight declaration as shown below.

@weight(...)
FACTOR_HEAD :- RULE_BODY.

Here, RULE_BODY denotes a typical conjunctive query also used for normal derivation rules in DDlog. FACTOR_HEAD denotes the part where more than one variable relations can appear with a special syntax. Let's first look at the simplest case of describing one variable in the head of an inference rule.

Specifying features

In common cases, one wants to model the probability of a Boolean variable being true using a set of features. Expressing this kind of binary classification problem is very simple in DDlog. By writing a rule with just one variable relation in the head, DeepDive creates in the model aunary factor that connects to it. The weight of this unary factor is determined by a user-defined feature. For instance, in the spouse example, there is an inference rule specifying features for the has_spouse variables written as:

@weight(f)
has_spouse(p1_id, p2_id) :-
    spouse_candidate(p1_id, _, p2_id, _),
    spouse_feature(p1_id, p2_id, f).

This rule means that:

A factor should be created for each pair of person mentions found in the spouse_candidate relation and each of the corresponding features found in spouse_feature relation.

Each of those factors connects to a single has_spouse variable identified by a pair of mentions (p1_id, p2_id) originating from thespouse_candidate relation.

The feature f for a factor determines a weight (to be learned for this particular rule) that translates into the factor's potential, which in turn influences the probability of the connected has_spouse variable being true or not.

Specifying correlations

Now, in almost every problem, the variables are correlated with each other in a special way, and it is desirable to enrich the model with this domain knowledge. Such correlations can be modeled by creating certain types of factors that connect multiple correlated variables together. This is where a richer syntax in the FACTOR_HEAD comes into play. DDlog borrows a lot of syntax from Markov Logic Networks, and hence, first-order logic.

For example, the following rule in the smoke example correlates two variable relations.

@weight(3)
smoke(x) => cancer(x) :-
    person(x).

This rule expresses that if a person smokes, there is an implication that he/she will have cancer. Here, a constant 3 is used in the @weightto express some level of confidence in this rule, instead of learning the weight from the data (explained later).

Implication

Logical implication or consequence of two or more variables can be expressed using the following syntax.

@weight(...)  P(x) => Q(y)              :- RULE_BODY.
@weight(...)  P(x), Q(y) => R(z)        :- RULE_BODY.
@weight(...)  P(x), Q(y), R(z) => S(k)  :- RULE_BODY.

Disjunction

Logical disjunction of two or more variables can be expressed using the following syntax.

@weight(...)  P(x) v Q(y)         :- RULE_BODY.
@weight(...)  P(x) v Q(y) v R(z)  :- RULE_BODY.

Conjunction

Logical conjunction of two or more variables can be expressed using the following syntax.

@weight(...)  P(x) ^ Q(y)         :- RULE_BODY.
@weight(...)  P(x) ^ Q(y) ^ R(z)  :- RULE_BODY.

Equality

Logical equality of two variables can be expressed using the following syntax.

@weight(...)  P(x) = Q(y)  :- RULE_BODY.

Negation

Whenever a rule has to refer to a case when the Boolean variable is false (also referred to as a negative literal), then it can be negated using a preceding ! as shown below.

@weight(...)    P(x) => ! Q(x)  :- RULE_BODY.
@weight(...)  ! P(x) v  ! Q(x)  :- RULE_BODY.

Multinomial factors (STALE. NEEDS REVAMP.)

DeepDive has limited support for expressing correlations of categorical variables. The introduced syntax above can be used only for expressing correlations between Boolean variables. For categorical variables, DeepDive only allows the conjunction of the variables each taking a certain category value being true to be expressed using a special syntax shown below:

@weight(...)  Multinomial( P(x), Q(y) )        :- RULE_BODY.
@weight(...)  Multinomial( P(x), Q(y), R(z) )  :- RULE_BODY.

Multinomial takes only categorical variables as arguments*, and it can be thought as a compact representation of an equivalent model with Boolean variables corresponding to each category connected by a conjunction factor for every combination of category assignments.

For example, suppose a is a variable taking values 0, 1, 2, and b is a variable taking values 0, 1. Then, Multinomial(a, b) is equivalent to having factors between a and b that correspond to the following indicator functions.

I{a = 0, b = 0}

I{a = 0, b = 1}

I{a = 1, b = 0}

I{a = 1, b = 1}

I{a = 2, b = 0}

I{a = 2, b = 1}

Note that each of the factors above has a distinct weight, i.e., one weight for each possible assignment of variables in the Multinomialfactor. For more detail on how to specify Conditional Random Fields and perform Multi-class Logistic Regression using categorical factor, see the chunking example.

Because of this limitation, categorical variables and categorical factor support is likely to go away in a near future release, in favor of a more flexible way to express multi-class predictions and mutual exclusions. Emulate categorical variables with functional dependencies in the Boolean variable columns, i.e., tag?(@key word_id BIGINT, pos TEXT) rather than tag?(word_id BIGINT) Categorical(N), and replace Multinomial factor with conjunction of those Boolean variables with columns having functional dependencies.

Specifying weights

Each factor is assigned a weight, which represents the confidence in the correlation it expresses in the model. During statistical inference, these weights translate into their potentials that influence the probabilities of the connected variables. Factor weights are real numbers and only the relative magnitude to each other matters. Factors with larger weights have a greater impact on the connected variables than factors with smaller weights. Weights can be fixed to a constant manually, or they can be learned by DeepDive from the supervision labels at different granularity. Weights can be parameterized by some data originating from the data (in most time referred to as features), in which case factors with different parameter values will use different weights. In order to learn weights automatically, there must be enough training data available.

DDlog syntax for specifying weights for three different cases are shown below.

Q?(x TEXT).
data(x TEXT, y TEXT).

# Fixed weight (10 can be treated as positive infinite)
@weight(10)   Q(x) :- data(x, y).

# Unknown weight, to be learned from the data, but not depending on any variable.
# All factors created by this rule will have the same weight.
@weight("?")  Q(x) :- data(x, y).

# Unknown weight, each to be learned from the data per different values of y.
@weight(y)    Q(x) :- data(x, y).
2.2 怎么运行

2.2.1 Compiling a DeepDive application

The first step to run a DeepDive application is to compile it. We describe here the use of the different deepdive commands, why and where it compiles and which files can be interesting to look at.

How to compile

To compile a DeepDive application, simply run the following command within the DeepDive app.

deepdive compile

​	All subsequent deepdive do commands will run what has been compiled, so an app has to be compiled at least once initially before executing any part of it.

What is compiled

All compiled output is created under the run/ directory of the app:

run/dataflow.svg

A data flow diagram showing the dependencies between processes. This file contains a graph of the dataflow and helps better understand the dependencies among relations, rules and user-defined functions used to produce them. It can be opened in a web browser (Chrome or Safari work particularly well for it).

run/Makefile

A Makefile that mirrors the dependencies, used for generating execution plans for running certain parts of the data flow.

run/process/**/run.sh

Shell scripts that contain what actually should be run for every process. These scripts will be run by certain deepdive do commands.

run/LATEST.COMPILE/ and run/compiled/

Each compilation step creates a unique timestamped directory under run/ to store all the intermediate representation used for compilation (*.json). In particular, the latest compiled version can be found in run/LATEST.COMPILE/. For instance, the deepdive.conf used for the latest compilation can be found at run/LATEST.COMPILE/deepdive.conf.

config.json

The compiled final configuration object.

code-*.json

The compiled object for generating actual code.

compile.log

The log file left by the latest compilation step.

If you keep track of your DeepDive application with Git, it may be important to add a /run line in .gitignore.

When to compile

The following files in a DeepDive application are considered as the input to deepdive compile:

app.ddlog

deepdive.conf

schema.json

deepdive compile should be rerun whenever changes to these files are to be reflected on the execution done by deepdive do. Otherwise, the modifications in app.ddlog won't be considered, and simply the last compiled code will execute.

Why compile

Compiling a DeepDive application presents two main advantages. First, it helps to run the application faster by extracting the correct dependencies between the different operations. Second, it provides many checks for the application and helps detecting errors even before running DeepDive.

Compile-time checks

All the extractors and tables declarations can be written in any order in app.ddlog: the whole dataflow is extracted during the compilation and many checks are made. All these checks can also be performed individually by the command deepdive check. In particular, the following checks are made:

input_extractors_well_defined: checks sanity of all defined extractors.

input_schema_wellformed: checks if schema variables are well formed.

compiled_base_relations_have_input_data: all the tables declared that are not filled by an extractor must have input data in input/ that can be loaded automatically.

compiled_dependencies_correct: check that the dependencies are correct. In particular, in the application includes extractors in deepdive.conf, each output relation must be output by exactly one extractor.

compiled_input_output_well_defined: checks if all inputs and outputs of compiled processes are well-defined.

compiled_output_uniquely_defined: checks if all outputs in the compiled documents are defined by one process.

2.2.2 Managing input data and data products

DeepDive provides a handful of commands and conventions to manage input data to an application as well as the data produced by it. The actual processing of the data is discussed in a page about executing the DeepDive application.

Inspecting schema for relations in the DeepDive app

There are a few handy ways to inspect the relational schema declared for a DeepDive app. Once the application is compiled, the following commands allow you to access the schema.

Listing relations

deepdive relation list

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

Listing columns of a relation

deepdive relation columns articles

id:text
content:text

Listing variable relations

deepdive relation variables

has_spouse

Preparing the database for the DeepDive app

The database for the DeepDive application is configured through the db.url file or can be overridden with the DEEPDIVE_DB_URLenvironment variable.

Initializing the database

To initialize the database in a clean state, also dropping it if necessary, run:

deepdive db init

Creating tables in the database

To make sure an empty table named foo is created in the database as declared in app.ddlog, run:

deepdive create table foo

It is also possible to create any table with a particular column-type definitions, after another existing table such as bar, or using the result of a SQL query. A view can be created given a SQL query. This command ensures any existing table or view with the same name is dropped, and a new empty one is created and exists.

deepdive create table foo x:INT y:TEXT ...

deepdive create table foo like bar

deepdive create table foo as 'SELECT ...'

deepdive create view bar as 'SELECT ...'

To create a table only if it does not exist, the following command can be used instead:

deepdive create table-if-not-exists foo ...

Organizing input data

All input data for a DeepDive application should be kept under the input/ directory. DeepDive relies on a naming convention and assumes data for a relation *foo* declared in app.ddlog (and not output by an extractor) exists at path input/*foo*.*extension* where *extension* can be one of tsv, csv, sql tsv.bz2, csv.bz2, tsv.gz, csv.gz, tsv.sh, csv.sh. This indicates in what format it is serialized as well as how it is compressed, or whether it's a shell script that emits such data or a file containing the data itself. For example, in the spouse example, the input/articles.tsv.sh is a shell script that produces lines with tab-separated values for the "articles" relation.

Moving data in and out of the database

Loading data to the database

To load such input data for a relation foo, run:

deepdive load foo

To load data from a particular source such as source.tsv or multiple sources /tmp/source-1.tsv, /data/source-2.tsv.bz2 instead, they can be passed over as extra arguments:

deepdive load foo  source.tsv

deepdive load foo  /tmp/source-1.tsv /data/source-2.tsv.bz2

When a data source provides only particular columns, they can be specified as follows:

deepdive load 'foo(x,y)'  only-x-y.csv

If the destination to load is a variable relation and no columns are explicitly specified, then the sources are expected to provide an extra column at the right end that corresponds to each row's supervision label.

Unloading data products from the database

To unload data produced for a relation to files, such as bar into two sinks dump-1.tsv and /data/dump-2.csv.bz2, assuming you have the table populated in the database by executing some data processing, run:

deepdive unload bar  bar-1.tsv /data/bar-2.csv.bz2

This will unload partitions of the rows of relation bar to the given sinks in parallel.

Running queries against the database

Running queries in DDlog

It is possible to write simple queries against the database in DDlog and run them using the deepdive query command. A DDlog query begins with an optional list of expressions, followed by a separator ?-, then a typical body of a conjunctive query in DDlog. Following are examples of actual queries that can be used against the data produced by DeepDive for the spouse example after running deepdive runcommand at least once.

Browsing values

To browse values in a relation, variables can be placed at the columns of interest, such as name1 and name2 for the second and fourth columns of the spouse_candidate relation, and the wildcard _ can be used for the rest to ignore them as follows:

deepdive query '?- spouse_candidate(_, name1, _, name2).'

Joins, selection, and projections

Finding values across multiple relations that satisfy certain conditions can be expressed in a succinct way. For example, finding the names of candidate pairs of spouse mentions in a document that contains a certain keyword, such as "President" can be written as:

deepdive query '
    name1, name2 ?-
        spouse_candidate(p,name1,_,name2),
        person_mention(p,_,doc,_,_,_),
        articles(doc, content),
        content LIKE "%President%".
    '

Aggregation

Computing aggregate values such as counts and sums can be done using aggregate functions. Counting the number of tuples is as easy as just adding a COUNT(1) expression before the separator. For example, to count the number of documents that contain the word "President" is written as:

deepdive query 'COUNT(1) ?- articles(_, content), content LIKE "%President%".'

Grouping

If expressions using aggregate functions are mixed with other values, the aggregates are grouped by the other ones. This makes it easy to break down a single aggregate value into parts, such as number of candidates per document as shown below.

deepdive query '
    doc, COUNT(1) ?-
        spouse_candidate(p,_,_,_),
        person_mention(p,_,doc,_,_,_),
        articles(doc, content).
    '

Ordering

Ordering the results by certain ways is also easily expressed by adding @order_by annotations before the value to use for ordering. For example, the following modified query shows the documents with the most number of candidates first:

deepdive query '
    doc, @order_by("DESC") COUNT(1) ?-
        spouse_candidate(p,_,_,_),
        person_mention(p,_,doc,_,_,_),
        articles(doc, content).
    '

@order_by takes two arguments: 1) dir parameter taking either the "ASC" or "DESC" for ascending (default) or descending order and 2) priority parameter taking a number for deciding the priority for ordering when there are multiple @order_by expressions (smaller the higher priority). For example, @order_by("DESC", -1), @order_by("DESC"), @order_by(priority=-1) are all recognized.

Limiting (top k)

By putting a number after the expressions separated by a pipe character, e.g., | 10, the number of tuples can be limited. For example, the following modified query shows the top 10 documents containing the most candidates:

deepdive query '
    doc, @order_by("DESC") COUNT(1) | 10 ?-
        spouse_candidate(p,_,_,_),
        person_mention(p,_,doc,_,_,_),
        articles(doc, content).
    '

Using more than one rule

Sometimes one rule is not enough to express, so a query may define multiple temporary relations first. For example, to produce a histogram of the number of candidates per document, the counts must be counted, so two rules are necessary as shown below.

deepdive query '
    num_candidates_by_doc(doc, COUNT(1)) :-
        spouse_candidate(p,_,_,_),
        person_mention(p,_,doc,_,_,_),
        articles(doc, content).

    @order_by num_candidates, COUNT(1) ?- num_candidates_by_doc(doc, num_candidates).
    '

Saving results

By providing the extra format=tsv or format=csv argument, the resulting tuples can be easily saved into a file. For example, the following command saves the names of candidates with their document id as a comma-separated file named candidates-docs.csv.

deepdive query '
    doc,name1,name2 ?-
        spouse_candidate(p1,name1,_,name2),
        person_mention(p1,_,doc,_,_,_).
    ' format=csv  >candidates-docs.csv

Viewing the SQL

Using the -n flag will display the SQL query to be executed instead of showing the results from executing it.

deepdive query -n '?- ...'

Running queries in SQL

deepdive sql

This command opens a SQL prompt for the underlying database configured for the application.

Optionally, the SQL query can be passed as a command line argument to run and print its result to standard output. For example, the following command prints the number of sentences per document:

deepdive sql "SELECT doc_id, COUNT(*) FROM sentences GROUP BY doc_id"

To get the result as tab-separated values (TSV), or comma-separated values (CSV), use the following commands:

deepdive sql eval "SELECT doc_id, COUNT(*) FROM sentences GROUP BY doc_id" format=tsv

deepdive sql eval "SELECT doc_id, COUNT(*) FROM sentences GROUP BY doc_id" format=csv header=1
Executing processes in the data flow
After compiling a DeepDive application, each compiled process in the data flow defined by the application can be executed with great flexibility. DeepDive provides a core set of commands that supports precise control of the execution. The following tasks are a few examples that are easily doable in DeepDive's vocabulary:

Executing the complete data flow.

Executing a fragment of the complete data flow.

Stopping execution at any point and resuming from where it stopped.

Repeating certain processes with different parameters.

Skipping expensive processes by loading their output from external data sources.

End-to-end execution

To simply run the complete data flow in an application from the beginning to the end, use the following command under any subdirectory of the application:

deepdive run

This command will:

Initialize the application as well as its configured database.

Run all processes that correspond to the normal derivation rules as well as the rules with user-defined functions.

Ground the factor graph according to the inference rules defining the model for statistical inference.

Perform weight learning and inference to compute marginal probability of every variable.

Generate calibration plots and data for debugging.

Granular execution scenarios

There are several execution scenarios that frequently arise while developing a DeepDive application. Any of the commands shown in this page can be run under any subdirectory of a DeepDive application. To see all options for each command, the online help message can be seen with the deepdive help command. For example, the following shows detailed usage of deepdive do command:

deepdive help do

Partial execution

Running only a small part of the data flow defined in an application is the most common scenario. The following command allows the execution to stop at given TARGETs instead of continuing to the end.

deepdive do TARGET...

It will present a shell script in a text editor that enumerates the processes to be run for the given TARGETs. By saving a final plan and quiting the editor, the actual execution starts.

Valid TARGET names are shown when no argument is given.

deepdive do

Refer to the next section for more detail about these TARGET names.

Stopping and resuming

The execution started can be interrupted at any time (with Ctrl-C or ^C) or aborted by an error, then resumed later.

deepdive do TARGET...

...
^C
...

deepdive do TARGET...

DeepDive will resume the execution from the last unfinished process, skipping what has already been done.

Repeating

Repeating certain parts of the data flow is another common scenario. DeepDive provides a way to mark certain processes as not done so they can be repeated. For example, the sequence of two commands below is basically what deepdive run does, i.e., repeats the end-to-end execution.

deepdive mark todo init/app calibration weights

deepdive do        init/app calibration weights

DeepDive provides a shorthand deepdive redo for easier repeating:

deepdive redo init/app calibration weights

Skipping

Skipping certain parts of the data flow and starting from data manually loaded is also easily doable. This can be useful to skip certain processes that take very long. For example, suppose relation bar_derived_from_foo is derived from relation foo, and foo takes excessive amount of time to compute and therefore has been saved at /some/data/source.tsv. Then the following sequence of commands skips all processes that foo depends on, assumes it is new, and executes the processes that derive bar_derived_from_foo from foo.

deepdive create table foo

deepdive load foo /some/data/source.tsv

deepdive mark new foo

deepdive do bar_derived_from_foo

Compiled processes and data flow

For execution, DeepDive produces a compiled data flow graph that consists of primarily data, model, and process nodes and edges representing dependencies between them.

Data nodes correspond to relations or tables in the database.

Model nodes denote artifacts for statistical learning and inference.

Process nodes represent a unit of computation.

An edge between a process and data/model nodes mean the process takes as input or produces such node.

An edge between two processes denotes one is dependent on another while hiding the details about the intermediary nodes.

DeepDive compiles a few built-in processes into the data flow graph that are necessary for initialization, statistical learning and inference, and calibration. The rest of the processes correspond to the rules for deriving relations according to the DDlog program and the extractors in deepdive.conf. Below is a data flow graph compiled for the tutorial example, which can be found at run/dataflow.svg for any application.

Execution plan

DeepDive uses the compiled data flow graph to find the correct order of processes to execute. Any node or set of nodes in the data flow graph can be set as the target for execution, and DeepDive enumerates all necessary processes as an execution plan.

deepdive plan

This execution plan can be seen using the deepdive plan command. For example, when a plan for data/sentences is asked using the following command:

deepdive plan data/sentences

DeepDive gives an output that looks like:

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

An execution plan is basically a shell script that invokes the actual run.sh compiled for each process. Processes that have been already marked as done in the past are commented out (with :), and exactly when they were done is displayed in the comments.

deepdive do

DeepDive provides a deepdive do command that takes as input the target nodes in the data flow graph, presents the execution plan in an editor, then executes the final plan. Because of the provided chance to modify the generated execution plan, the user has complete control over what is executed, and override or skip certain processes if needed.

deepdive do data/sentences

Execution timestamps

After a process finishes its execution, the execution plan includes a mark_done command to mark the executed process as done. The command touches a *process/name*.done file under the run/ directory to record a timestamp. These timestamps are then used for determining which processes have finished and which others have not.

deepdive mark

DeepDive provides a deepdive mark command to manipulate such timestamps to repeat or skip certain parts of the data flow. It allows a given node to be marked as:

done

So all processes depending on the process including itself can be skipped.

todo-from-scratch

So all processes from the first process in the execution plan can be repeated.

todo

So all processes that depend on the node including itself can be repeated.

new

So all processes that depend on the node can be repeated (not including itself).

all-new

So all processes that depend on the node or any of its ancestor can be repeated (not including itself).

For example, if we mark a process as to be repeated using the following command:

deepdive mark todo process/init/relation/articles

Then deepdive plan will give an output like below to repeat the processes already marked as done in the past:

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


Built-in data flow nodes

DeepDive adds several built-in processes to the compiled data flow to ensure necessary steps are performed before and after the user-defined processes.

process/init/app

This process initializes the application's database and executes the input/init.sh script if available, which can be used for ensuring input data is downloaded and libraries and code required by UDFs are set up correctly.

All processes that are not dependent on any other automatically becomes dependent on this process to ensure every process is executed after the application is initialized.

process/init/relation/*R*

Such process creates the table R in the database and loads data from input/*R*.*. These processes are automatically created and added to the data flow for every relation that is not derived from any other relation nor output by any process.

process/grounding/*

All processes for grounding the factor graph are put under this namespace.

process/model/*

All processes for learning and inference are put under this namespace.

model/*

All artifacts related to the statistical inference model that do not belong to the database are put under this namespace, e.g.:

model/factorgraph

Denotes factor graph binary files under run/model/factorgraph/.

model/weights

Denotes text files that contain learned weights of the model under run/model/weights/.

model/probabilities

Denotes files holding the computed marginal probabilities under run/model/probabilities/

model/calibration-plots

Denotes calibration plot images and data files under run/model/calibration-plots/.

data/model/*

All data relevant to statistical inference that go into the database are put under this namespace. For example, data/model/weightsand data/model/probabilities correspond to tables and views that keep the learning and inference results.

Environment variables

There are several environment variables that can be tweaked to influence the execution of processes for UDFs.

DEEPDIVE_NUM_PROCESSES

Controls the number of UDF processes to run in parallel. This defaults to one less than the number of processors, minimum one.

DEEPDIVE_NUM_PARALLEL_UNLOADS and DEEPDIVE_NUM_PARALLEL_LOADS

Controls the number of processes to run in parallel for unloading from and loading to the database. These default to one.

DEEPDIVE_AUTOCOMPILE

Controls whether deepdive do should automatically compile the app whenever a source file changes, such as app.ddlog or deepdive.conf. By default, it is DEEPDIVE_AUTOCOMPILE=true, i.e., automatically compiling to reduce the friction of manually running deepdive compile. However, it can be set to DEEPDIVE_AUTOCOMPILE=false to prevent unexpected recompilations.

DEEPDIVE_INTERACTIVE

Controls whether deepdive do should be interactive or not, asking to review the execution plan and allow editing before actual execution. By default, it is DEEPDIVE_INTERACTIVE=false, not asking whether to recompile the app upon source change, nor providing a chance to review or edit the execution plan.

DEEPDIVE_PLAN_EDIT

Controls whether a chance to edit the execution plan is provided (when set to true) or not (when false) in interactive mode.

VISUAL and EDITOR

Decides which editor to use. Defaults to vi.

Learning and inference with the statistical model
For every DeepDive application, executing any data processing it defines is ultimately to supply with necessary bits in the construction of the statistical model declared in DDlog for joint inference. DeepDive provides several commands to streamline operations on the statistical model, including its creation (grounding), parameter estimation (learning), and computation of probabilities (inference) as well as keeping and reusing the parameters of the model (weights).

Getting the inference result

To simply get the inference results, i.e., the marginal probabilities of the random variables defined in DDlog, use the following command:

deepdive do probabilities

This takes care of executing all necessary data processing, then creates a statistical to perform learning and inference, and loads all probabilities of every variable into the database.

Inspecting the inference result

For viewing the inference result, DeepDive creates a database view that corresponds to each variable relation (using a _inference suffix). For example, the following SQL query can be used for inspecting the probabilities of the variables in relation has_spouse:

deepdive sql "SELECT * FROM has_spouse_inference"

It shows a table that looks like below where the expectation column holds the inferred marginal probability for each variable:

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

To better understand the inference result for debugging, please refer to the pages about calibration, Dashboard, labeling, and browsing data.

The next several sections describe further detail about the different operations on the statistical model supported by DeepDive.

Grounding the factor graph

The inference rules written in DDlog give rise to a data structure called factor graph DeepDive uses to perform statistical inference.Grounding is the process of materializing the factor graph as a set of files by laying down all of its variables and factors in a particular format. This process can be performed using the following command:

deepdive model ground

The above can be viewed as a shorthand for executing the following built-in processes:

deepdive redo process/grounding/variable_assign_id process/grounding/combine_factorgraph

Grounding generates a set of files for each variable and factor under run/model/grounding/. They are then combined into a unified factor graph under run/model/factorgraph/ to be easily consumed by the DimmWitted inference engine for learning and inference. For example, below shows a typical list of files holding a grounded factor graph:

find run/model/grounding -type f

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

Learning the weights

DeepDive learns the weights of the grounded factor graph, i.e., estimates the maximum likelihood parameters of the statistical model from the variables that were assigned labels via distant supervision rules written in DDlog. DimmWitted inference engine uses Gibbs samplingwith stochastic gradient descent to learn the weights.

The following command performs learning using the grounded factor graph (or grounds a new factor graph if needed):

deepdive model learn

This is equivalent to executing the following targets:

deepdive redo process/model/learning data/model/weights

DimmWitted outputs the learned weights as a text file under run/model/weights/. For convenience, DeepDive loads the learned weights into the database and creates several views for the following target:

deepdive do data/model/weights

This will create a comprehensive view of the weights named dd_inference_result_weights_mapping. The weights corresponding to each inference rule and by their parameter value can be easily accessed using it. Below shows a few example of learned weights:

deepdive sql "SELECT * FROM dd_inference_result_weights_mapping"

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

Inference

After learning the weights, DeepDive uses them with the grounded factor graph to compute the marginal probability of every variable. DimmWitted's high-speed implementation of Gibbs sampling is used for performing a marginal inference by approximately computing the probablities of different values each variable can take over all possible worlds.

deepdive model infer

This is equivalent to executing the following nodes in the data flow:

deepdive redo process/model/inference data/model/probabilities

In fact, because performing inference as a separate process from learning incurs unnecessary overhead of reloading the factor graph into memory again, DimmWitted also performs inference immediately after learning the weights. Therefore unless previously learned weights are being reused, hence skipping the learning part, the following command that performs just the inference has no effect:

DimmWitted outputs the inferred probabilities as a text file under run/model/probabilities/. As shown in the first section, DeepDive loads the computed probabilities into the database and creates views for convenience.

Reusing weights

A common use case is to learn the weights from one dataset then performing inference on another, i.e., train model on one dataset and test it on new datasets.

Learn the weights from a small dataset.

Keep the learned weights.

Reuse the kept weights for inference on a larger dataset.

DeepDive provides several commands to support the management and reuse of such learned weights.

Keeping learned weights

To keep the currently learned weights for future reuse, say under a name FOO, use the following command:

deepdive model weights keep FOO

This dumps the weights from the database into files at snapshot/model/weights/FOO/ so they can be reused later. The name FOO is optional, and a generated timestamp is used instead when no name is specified.

Reusing learned weights

To reuse a previously kept weights, under a name FOO, use the following command:

deepdive model weights reuse FOO

This loads the weights at snapshot/model/weights/FOO/ back to the database, then repeats necessary grounding processes for including the weights into the grounded factor graph. The name FOO is optional, and the most recently kept weights are used when no name is specified.

A subsequent command for performing inference reuses these weights without learning.

deepdive model infer

Managing kept weights

DeepDive provides several more commands to manage the kept weights.

To list the names of kept weights, use:

deepdive model weights list

To drop a particular weights, use:

deepdive model weights drop FOO

To clear any previously loaded weights to learn new ones, use:

deepdive model weights init
2.3 评估和调试

Debugging user-defined functions
Many things can go wrong in user-defined functions (UDFs), so debugging support is important for the user to write the code and easily verify that it works as expected. UDFs can be implemented in any programming language as long as they take the form of an executable that reads from standard input and writes to standard output. Here are some general tips for printing information to the log and running the UDFs in limited ways to help work through issues without needing to run the entire data flow of the DeepDive application.

Printing to the log

Remember that the standard output of a UDF is already reserved for TSJ or TSV formatted data that gets loaded into the database. Therefore when a typical print statement is used for debugging, it won't appear anywhere in the log but just mangle the TSJ output stream and ultimately fail the UDF execution or corrupt its output. The correct way to print log statements is to print to the standard error. Below is an example in Python.

#!/usr/bin/env python
from deepdive import *
import sys

@tsj_extractor
@returns( ... )
def extract( ... ):
    ...
    print >>sys.stderr, 'This prints some_object to logs :', some_object
    ...

During execution of the script, anything written to standard error appears in the console as well as in the file named run.log under the run/LATEST/ directory.

Executing UDFs within DeepDive's environment

To assist with debugging issues in UDFs, DeepDive provides a wrapper command to directly execute it within the same environment it uses for the actual execution.

Suppose a Python UDF at udf/fn.py imports deepdive and ddlib as suggested in the guide for writing UDFs. When run normally as a Python script, it will give an error that looks like this:

python udf/fn.py

Traceback (most recent call last):
  File "udf/fn.py", line 2, in <module>
    from deepdive import *
ImportError: No module named deepdive

Instead, by prefixing the command with deepdive env, they can be executed as if they were executed in the middle of DeepDive's data flow.

deepdive env python udf/fn.py

This will take TSJ rows from standard input and print TSJ rows to standard output as well as debug logs to standard error. It can therefore be debugged just like a normal Python program.

第三部分 CookBook
