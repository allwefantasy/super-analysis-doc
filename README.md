
# 🚀 Super Analysis 部署指南

本指南将帮助你部署 Super Analysis 系统，包括安装必要组件、配置服务和启动系统。

---

## 📦 安装 Super Analysis


```bash
pip install -U auto-coder
pip install -U super-analysis
```

---

## 🤖 部署 Deepseek 模型代理

在启动其他服务之前，我们需要先部署 Deepseek 模型：

```bash
byzerllm deploy --pretrained_model_type saas/openai \
--cpus_per_worker 0.001 \
--gpus_per_worker 0 \
--worker_concurrency 1000 \
--num_workers 1 \
--infer_params saas.base_url="https://api.deepseek.com/v1" saas.api_key=${MODEL_DEEPSEEK_TOKEN} saas.model=deepseek-chat \
--model deepseek_chat
```

注意：确保已设置环境变量 `MODEL_DEEPSEEK_TOKEN`。

---

## 🛠️ 部署 Byzer-SQL

参考 [安装与配置 Byzer-SQL 文档](./docs/4.3.1%20安装与配置%20Byzer-SQL.pdf) 完成部署。
根据 [Byzer-SQL 和大模型整合文档](./docs//4.2.1.3%20Byzer-SQL%20和大模型的整合.pdf) 中安装插件，然后注册 `deepseek_chat` 函数。

> 启动时需要在安装有 super-analysis 的 conda 环境中启动。

当 byzer-sql 部署完成后，注册账号为 `hello`，然后在 byzer-sql 控制台中执行：

```sql
!byzerllm setup single;

run command as LLM.`` where 
action="infer"
and reconnect="true"
and pretrainedModelType="saas/openai"
and udfName="deepseek_chat";
```

---

## 示例数据

下载电影数据集： https://www.kaggle.com/datasets/rounakbanik/the-movies-dataset/download?datasetVersionNumber=7

---

> 下面的指令都是在命令行里操作哈

## 📊 数据预处理和服务启动

1. 抽取电影数据集schema：

```bash
super-analysis.convert --data_dir /Users/allwefantasy/data/movice --doc_dir /Users/allwefantasy/data/movice/schemas/
```

你还可以添加 --include-rows-num 5 让系统在生成 schema 文档时同时提供一些示例数据。方便大模型更好的对这个表进行认知。


2. 启动 schema 文档知识库：

```bash
auto-coder.rag serve \
--model deepseek_chat --index_filter_workers 100 \
--tokenizer_path /Users/allwefantasy/Downloads/tokenizer.json \
--doc_dir /Users/allwefantasy/data/movice/schemas/ \
--port 8001
```

3. 下载 Byzer-SQL 文档并启动文档知识库：

```bash
git clone https://github.com/allwefantasy/llm_friendly_packages

auto-coder.rag serve \
--model deepseek_chat --index_filter_workers 100 \
--tokenizer_path /Users/allwefantasy/Downloads/tokenizer.json \
--doc_dir  /Users/allwefantasy/projects/llm_friendly_packages/github.com/allwefantasy \
--port 8002
```

4. 启动兼容 OpenAI Server 的分析服务：

```bash
super-analysis.serve --served-model-name deepseek_chat --port 8000 \
--schema-rag-base-url http://127.0.0.1:8001/v1 \
--context-rag-base-url http://127.0.0.1:8002/v1 \
--byzer-sql-url http://127.0.0.1:9003/run/script
```

你可以通过 `--sql-func-llm-model` 函数单独为 SQL 函数指定模型(比如配置一个速度极快的模型)。注意，同样的，你需要在 Byzer-SQL 中注册这个函数。

---

现在，Super Analysis 系统已经完全部署并启动。你可以开始使用 OpenAI SDK 进行测试和接口调用。具体测试和接口使用方法请参考 [openai_local_api.ipynb](./openai_local_api.ipynb)。

🎉 恭喜！你已经成功部署了 Super Analysis 系统。如有任何问题，请随时查阅文档或联系支持团队。