# 股票信息查询机器人
## 项目简介
本项目的主要目的是构建以采用问询股票信息为应用背景的金融聊天机器人系统，实现的功能有：通过数据库查询帮助用户得到符合条件股票信息（价格筛选，位置筛选等），通过API查询得到AAPL（苹果）、ACN（爱僧哲）、TSLA（特斯拉）等数十种股票的实时价格、实时库存、一年中的最高最低价等功能，并且将机器人系统与微信集成。  
  
本项目主要涉及的技术有：数据库的查询与搜索、单轮多次增量查询技术、甄别否定实体技术、基于Rasa NLU的本地基础聊天机器人系统的构建、命名实体识别技术、通过正则表达式提取用户意图并组织回答、状态机的多轮多次查询技术等。
## 文件说明
### 主要项目文件包括：stock bot.ipynb，stocks.db，Ipynb_importer.py，config_spacy.yml，demo-rasa.json。  
stock bot.ipynb：包括了实现上述功能的全部代码。可以按照需求选择在控制台输出或者与微信集成。  
stocks.db：通过数据库技术创建的.db文件，用于存储股票价格、公司地理位置等信息。这里的数据是自造的，可通过stock bot.ipynb中的相关语句增添或删除信息。  
config_spacy.yml：spaCy的配置文件，包括language和pipeline。  
demo-rasa.json：rasa nlu的训练数据文件，这里使用的训练数据为结合实际场景的自造数据。  
Ipynb_importer.py：可通过该文件在jupyter notebook中调用其他.ipynb文件。  
## 项目内容及实现原理
### 数据库的查询与搜索
SQLite3 可使用 sqlite3 模块与 Python 进行集成，而且不需要单独安装该模块，因为 Python 2.5.x 以上版本默认自带了该模块。为了使用 sqlite3 模块，首先需要创建一个表示数据库的连接对象，然后可以有选择地创建光标对象，这将帮助我们执行所有的 SQL 语句。  
  
所以在本项目中我首先通过 conn = sqlite3.connect('stocks.db') 创建一个stock.db文件用于存储相关的股票信息。  
  
之后通过 c = conn.cursor() 创建一个 cursor，它将在 Python 数据库编程中用到。该方法接受一个单一的可选的参数 cursorClass。如果提供了该参数，则它必须是一个扩展自 sqlite3.Cursor 的自定义的 cursor 类。  
  
最后通过c.execute语句向数据库中填充数据。如下图所示：  
  
![image](https://github.com/Wendellxx/stock-bot/blob/master/images/1.png)  
  
### RASA NLU
Rasa是一个基于多轮对话的框架，其中包含两个模块Rasa core与Rasa nlu。Rasa nlu是用来理解语义的，包括意图识别，实体识别，它会把用户的输入转换为结构化的数据。  
首先，我们需要定义一个对应的意图可能会出现的文本内容文件demo-rasa.json。根据股票查询的情景，我们可以确定其主要的意图和实体的类别，本次项目中主要包含了如下意图和实体类型：  
#### Intents：
·greet：表示打招呼，问好。  
·thanks: 表示致谢。  
·inquiry：表示询问。  
·stock_search：表示需要搜索符合某一条件的股票名。  
·company_search：表示需要搜索某一股票的相关信息。  
·info_search：表示需要搜索确定股票的某一类信息，比如latest Price、latest volume等。  
·bye：表示离开、告别。  
  
#### Entities:  
·location：表示相关股票公司所处的位置，用于搜索符合某一条件的股票名。  
·price: 表示股票的价格信息。  
·company：表示股票名。  
这里使用的训练数据为结合实际场景的自造数据，并转换为rasa nlu训练数据的格式。  
之后我们就开始通过rasa nlu进行数据训练：  
  
![image](https://github.com/Wendellxx/stock-bot/blob/master/images/3.png)   
  
其中相关函数的说明：  
  
![image](https://github.com/Wendellxx/stock-bot/blob/master/images/4.png)  
  
### 单轮多次增量查询技术以及甄别否定实体技术
#### 单次多轮增量查询
多轮增量查询是指，通过多次增加条件，将符合条件的范围不断缩小，最后得到符合条件的结果。  
  
我在本次项目中将该技术与数据库技术结合起来使用，通过多次的条件限制，在数据库中查询到符合所有条件的股票名并将结果回应给用户。具体情景如下图对话所示：  
  
![image](https://github.com/Wendellxx/stock-bot/blob/master/images/5.png)  
  
具体实现依赖于rasa nlu的实体识别功能，通过entities = interpreter.parse(message)["entities"] 解析出用户发送的message中所有的实体内容，并且将实体的value作为参数传递至find_stocks函数中，之后find_stocks可根据传递过来的参数值在数据库中进行检索，并将检索结构反馈给用户。 
  
#### 否定甄别技术
假设在实体之前有“not”或者“n’t”意味着用户想要排除这个实体。具体情景如下图对话所示：  
  
![image](https://github.com/Wendellxx/stock-bot/blob/master/images/12.png)  
  
该技术可由negated_ents函数实现，通过将用户的message分解为一个个单词的字符串传入该函数，将与需要否定的实体相同的实体挑选出来放入ents，并将句子划分为不同的chunk，最后通过搜索其中是否有否定词来判断该实体是否被否定了。  
### 基于spaCy的实体识别
#### 命名实体识别
命名实体识别是NLP里的一项很基础的任务，就是指从文本中识别出命名性指称项，为关系抽取等任务做铺垫。狭义上，是识别出人命、地名和组织机构名这三类命名实体（时间、货币名称等构成规律明显的实体类型可以用正则表达式等方式识别）。当然，在特定的领域中，会相应地定义领域内的各种实体类型。  
  
#### spaCy
处理原始文本是很困难的：大多数单词都是很少见的，而对于有些看起来完全不同的单词来说，它们的意思可能是一样的。相同的词在不同的顺序可以意味着完全不同的东西。在很多语言中将文本分割成类单词单元都是困难的。虽然从原始字符开始解决一些问题是可能的，但是最好使用语言知识来添加有用的信息。这正是spaCy的设计目的:输入原始文本，然后返回一个Doc对象，它带有各种注释。使用Spacy进行命名实体识别的一个好处是我们只需要应用nlp一次，整个后台管道都会返回对象。  
  
要想使用 Spacy和访问其不同的properties，需要先创建pipelines。 通过加载模型来创建一个pipeline。Spacy 提供了许多不同的模型 , 模型中包含了语言的信息：词汇表，预训练的词向量，语法和实体。  

### 通过正则表达式提取用户意图或做字符匹配
正则表达式(regular expression)描述了一种字符串匹配的模式（pattern），可以用来检查一个串是否含有某种子串、将匹配的子串替换或者从某个串中取出符合某个条件的子串等。  
  
构造正则表达式的方法和创建数学表达式的方法一样。也就是用多种元字符与运算符可以将小的表达式结合在一起来创建更大的表达式。正则表达式的组件可以是单个的字符、字符集合、字符范围、字符间的选择或者所有这些组件的任意组合。  
  
正则表达式是由普通字符（例如字符 a 到 z）以及特殊字符（称为"元字符"）组成的文字模式。模式描述在搜索文本时要匹配的一个或多个字符串。正则表达式作为一个模板，将某个字符模式与所搜索的字符串进行匹配。  
  
使用正则表达式可以直接匹配到用户信息中的某些字符，从而推测出用户的意图。但其实他最大用处还是提取一串字符中的某段重要信息。比如在本项目中将机器人系统和微信集成起来的过程中，用户发出的文本信息会变为 [用户昵称] ：[用户信息] （text）这种形式输出，通过使用正则表达式就可以把用户昵称和信息格式这些不重要的信息删去，从而留下用户想表达的信息。如下图所示：  
  
![image](https://github.com/Wendellxx/stock-bot/blob/master/images/8.png)  
  
### 状态机的多轮多次查询技术
![image](https://github.com/Wendellxx/stock-bot/blob/master/images/13.png)  
  
本次项目中我定义了如下状态：  
·INIT=0 ：表示初始状态，用户刚与机器人对话。  
·AUTHED=1：表示认证状态，要求用户登录并输入手机号。如果不输入数字则一直保持该状态。  
·CHOOSE_INFO=2：表示选择查询信息状态，用户可在该状态下输入想要查询股票的哪一方面信息。比如latest Price、latest volume等。  
·ORDERED=3：表示已返回查找信息状态，用户可在此时选择继续查找信息状态（CHOOSE_INFO=2）或者离开状态(LEAVE)。  
·LEAVE=4：表示离开状态，用户已经结束了本次查询。  
  
如何判断状态何时应该跳转则用到了意图识别。当用户发出的信息满足某一意图时，则跳转至相应状态。  
比如当用户处于CHOOSE_INFO=2状态时，如果下一条信息为“How about its highest price during last 52 weeks”，则判定它的意图为info_search并继续停留在CHOOSE_INFO=2状态；如果用户的下一条信息为“It’s ok, bye”，则判定它的意图为goodbye并跳转至LEAVE=4的状态。下图是将状态和意图与对话同时输出，可以更好地理解他们之间的关系。  
  
![image](https://github.com/Wendellxx/stock-bot/blob/master/images/9.png)   
  
### 通过wxpy将聊天机器人与微信集成
[具体实现可参考这个链接](https://github.com/youfou/wxpy)  



