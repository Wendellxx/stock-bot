# 股票信息查询机器人
## 项目简介
该项目的主要目的是构建以采用问询股票信息为应用背景的金融聊天机器人系统，本项目实现的功能有：通过数据库查询帮助用户得到符合条件股票信息（价格筛选，位置筛选等），通过API查询得到AAPL（苹果）、ACN（爱僧哲）、TSLA（特斯拉）等数十种股票的实时价格、实时库存、一年中的最高最低价等功能，并且将机器人系统与微信集成。  
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
![](https://github.com/Wendellxx//raw/master//1.gif)
