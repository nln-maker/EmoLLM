# EmoLLM RAG

## **模块目的** 

根据用户的问题，检索对应信息以增强回答的专业性, 使EmoLLM的回答更加专业可靠。检索内容包括但不限于以下几点：
- 心理学相关理论
- 心理学方法论
- 经典案例
- 客户背景知识

## **环境准备**

```python

langchain==0.1.13
langchain_community==0.0.29
langchain_core==0.1.33
langchain_openai==0.0.8
langchain_text_splitters==0.0.1
FlagEmbedding==1.2.8
unstructured==0.12.6
```

```python

cd rag
pip3 install -r requirements.txt

```

## **使用指南** 

### 准备数据

- txt数据：放入到 src.data.txt 目录下
- json 数据：放入到 src.data.json 目录下

会根据准备的数据构建vector DB，最终会在 data 文件夹下产生名为 vector_db 的文件夹包含 index.faiss 和 index.pkl

如果已经有 vector DB 则会直接加载对应数据库


### 配置 config 文件

根据需要改写 config.config 文件：

```python

# 存放所有 model
model_dir = os.path.join(base_dir, 'model')

# embedding model 路径以及 model name
embedding_path = os.path.join(model_dir, 'embedding_model')
embedding_model_name = 'BAAI/bge-small-zh-v1.5'


# rerank model 路径以及 model name
rerank_path = os.path.join(model_dir, 'rerank_model')
rerank_model_name = 'BAAI/bge-reranker-large'


# select num: 代表rerank 之后选取多少个 documents 进入 LLM
select_num = 3

# retrieval num： 代表从 vector db 中检索多少 documents。（retrieval num 应该大于等于 select num）
retrieval_num = 10

# 智谱 LLM 的 API key。目前 demo 仅支持智谱 AI api 作为最后生成
glm_key = ''

# Prompt template: 定义
prompt_template = """
	你是一个拥有丰富心理学知识的温柔邻家温柔大姐姐艾薇，我有一些心理问题，请你用专业的知识和温柔、可爱、俏皮、的口吻帮我解决，回复中可以穿插一些可爱的Emoji表情符号或者文本符号。\n

	根据下面检索回来的信息，回答问题。

	{content}

	问题：{query}
"""
```

### 调用

```python
cd rag/src
python main.py
```


## **数据集**

- 经过清洗的QA对: 每一个QA对作为一个样本进行 embedding
- 经过筛选的TXT文本
	- 直接对TXT文本生成embedding (基于token长度进行切分)
	- 过滤目录等无关信息后对TXT文本生成embedding (基于token长度进行切分)
	- 过滤目录等无关信息后, 对TXT进行语意切分生成embedding
	- 按照目录结构对TXT进行拆分，构架层级关系生成embedding

数据集合构建的详情，请参考 [qa_generation_README](https://github.com/SmartFlowAI/EmoLLM/blob/ccfa75c493c4685e84073dfbc53c50c09a2988e3/scripts/qa_generation/README.md)

## **相关组件**

### [BCEmbedding](https://github.com/netease-youdao/BCEmbedding?tab=readme-ov-file)

- [bce-embedding-base_v1](https://hf-mirror.com/maidalun1020/bce-embedding-base_v1): embedding 模型，用于构建 vector DB
- [bce-reranker-base_v1](https://hf-mirror.com/maidalun1020/bce-reranker-base_v1): rerank 模型，用于对检索回来的文章段落重排

### [Langchain](https://python.langchain.com/docs/get_started)

LangChain 是一个开源框架，用于构建基于大型语言模型（LLM）的应用程序。LangChain 提供各种工具和抽象，以提高模型生成的信息的定制性、准确性和相关性。

### [FAISS](https://faiss.ai/)

Faiss是一个用于高效相似性搜索和密集向量聚类的库。它包含的算法可以搜索任意大小的向量集。由于langchain已经整合过FAISS，因此本项目中不在基于原生文档开发[FAISS in Langchain](https://python.langchain.com/docs/integrations/vectorstores/faiss)


### [RAGAS](https://github.com/explodinggradients/ragas)

RAG的经典评估框架，通过以下三个方面进行评估:

- Faithfulness: 给出的答案应该是以给定上下文为基础生成的。
- Answer Relevance: 生成的答案应该可以解决提出的实际问题。
- Context Relevance: 检索回来的信息应该是高度集中的，尽量少的包含不相关信息。

后续增加了更多的评判指标，例如：context recall 等


## **方案细节**

### RAG具体流程

- 根据数据集构建vector DB
- 对用户输入的问题进行embedding
- 基于embedding结果在向量数据库中进行检索
- 对召回数据重排序
- 依据用户问题和召回数据生成最后的结果

**Noted**: 当用户选择使用RAG时才会进行上述流程

### 后续增强

- 将RAGAS评判结果加入到生成流程中。例如，当生成结果无法解决用户问题时，需要重新生成
- 增加web检索以处理vector DB中无法检索到对应信息的问题
- 增加多路检索以增加召回率。即根据用户输入生成多个类似的query进行检索


