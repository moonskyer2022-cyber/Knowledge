# 医药知识图谱

一个面向临床决策辅助场景的本地知识图谱项目，基于 `Neo4j + FastAPI + HTML/CSS/JS` 实现，覆盖疾病推荐、药品详情、药物相互作用、禁忌查询、合并症分析和规则问答等核心能力。

这个项目适合两类读者：

- 面试官：可以快速看到一个完整的“数据清洗 -> 图谱建模 -> 图数据库导入 -> API 服务 -> 前端展示 -> 简单评估”闭环项目。
- 初学者：可以把它当作一个 Neo4j/FastAPI 入门样例，理解知识图谱项目是如何从 CSV 数据一步步落地成可查询系统的。

> **免责声明**
> 本项目数据与结论仅用于学习、演示与技术研究，不能替代执业医师、药师或正式临床指南。

## 项目亮点

- 使用 Neo4j 建模医药领域中的 `疾病 / 药品 / 方案 / 禁忌 / 不良反应 / ATC 分类` 等实体关系。
- 提供面向业务的 API，而不只是“把图数据库暴露出来”。
- 支持多药联用风险分析，能识别已知相互作用和同类重复用药。
- 提供简单的规则问答能力，演示“自然语言入口 + 图谱查询”的思路。
- 附带清洗脚本、导入脚本、评估脚本和前端页面，便于完整展示项目工程化思路。

## 我能学到什么

如果你是初学者，这个项目最值得学习的不是“医学知识本身”，而是下面这套方法：

1. 如何把表格数据抽象成图谱节点和关系。
2. 如何用 `Cypher` 写面向业务的查询。
3. 如何用 `FastAPI` 包装成可被前端和其他系统调用的接口。
4. 如何把“推荐、检索、风险提示、问答”做成一个统一的小系统。
5. 如何给一个数据项目补上 `scripts/`、`tests/`、`docs/` 这些基础工程结构。

## 功能概览

| 模块 | 能力说明 |
|------|------|
| 疾病推荐 | 根据疾病名或 ICD 返回治疗方案、药物、证据等级、适用人群等信息 |
| 药品详情 | 查询适应症、不良反应、禁忌、ATC 分类、别名 |
| 相互作用检测 | 分析多药联用风险，识别严重程度和建议 |
| 合并症分析 | 合并多个疾病的治疗方案，并检查交叉用药冲突 |
| 关联图谱可视化 | 展示疾病、方案、药品、不良反应等节点关系 |
| 规则问答 | 通过自然语言完成联用、推荐、适应症、禁忌类问题解析 |
| 回归评估 | 通过 `scripts/eval.py` 对推荐、相互作用、问答结果做简单校验 |

## 适合在面试中怎么介绍

你可以这样概括这个项目：

> 这是一个基于 Neo4j 的医药知识图谱项目。我先把药品、疾病、治疗方案、禁忌和不良反应等 CSV 数据清洗后导入图数据库，再通过 FastAPI 暴露推荐、药品详情、相互作用检查、合并症分析和问答接口，前端再把结果做成可视化页面。它体现的是一个完整的数据产品原型，而不是单点脚本。

如果你想更突出技术点，也可以这样说：

- 我做了领域建模，把关系型表格重新组织成图结构。
- 我把 Cypher 查询封装成业务 API，而不是只停留在数据库层。
- 我额外补了前端展示和评估脚本，让项目更接近“可演示、可说明”的作品。

## 系统架构

```text
raw CSV data
   ->
scripts/clean.py
   ->
data/clean/*.csv
   ->
scripts/import_neo4j.py
   ->
Neo4j graph database
   ->
api/*.py (FastAPI + Cypher)
   ->
web/index.html + web/js/app.js
```

## 技术栈

- 图数据库：`Neo4j 5`
- 后端：`FastAPI`
- 查询语言：`Cypher`
- 数据处理：`pandas`
- 前端：原生 `HTML / CSS / JavaScript`
- 测试与评估：`pytest`、自定义 `eval.py`

## 项目结构

```text
AI_Knowledeg/
├── api/                # FastAPI 后端与问答/图谱查询逻辑
├── web/                # 前端页面与交互脚本
├── data/raw/           # 原始 CSV 数据
├── data/clean/         # 清洗后的数据
├── scripts/            # 清洗、导入、评估、启动脚本
├── cypher/             # Neo4j schema 与查询定义
├── tests/              # 单元测试与评估用例
└── docs/               # 本体、说明文档
```

## 核心数据模型

```text
Drug  -[TREATS]->              Disease
Drug  -[HAS_DOSAGE_FOR]->      Disease
Drug  -[INTERACTS_WITH]-       Drug
Drug  -[CONTRAINDICATED_FOR]-> Condition
Drug  -[CAUSES]->              AdverseEffect
Drug  -[HAS_ALIAS]->           Alias
Drug  -[BELONGS_TO_ATC]->      AtcClass
Plan  -[TARGETS]->             Disease
Plan  -[INCLUDES]->            Drug
Disease -[SUBCLASS_OF]->       Disease
AtcClass -[SUBCLASS_OF]->      AtcClass
```

更详细的本体说明见 [docs/ONTOLOGY.md](docs/ONTOLOGY.md)。

## 典型使用场景

### 1. 疾病治疗方案推荐

输入疾病名或 ICD，例如：

- `高血压`
- `E11`

系统返回：

- 对应治疗方案
- 方案中的药物组合
- 治疗线别
- 证据等级
- 适用人群和简要说明

### 2. 多药联用风险检查

输入多个药名，例如：

- `阿司匹林 + 氯吡格雷`
- `缬沙坦 + 厄贝沙坦`

系统可以返回：

- 是否存在已知相互作用
- 风险严重程度
- 处理建议
- 是否存在同类 ATC 重复用药

### 3. 药品信息查询

输入药品名后，可以查看：

- 适应症
- 不良反应
- 禁忌与慎用条件
- ATC 分类
- 别名

### 4. 自然语言问答

示例问题：

- `阿司匹林和氯吡格雷能一起吃吗？`
- `高血压推荐什么方案？`
- `二甲双胍的适应症是什么？`

## 快速开始

### 1. 准备 Neo4j

可以用 Docker：

```powershell
docker compose up -d
```

也可以使用 Neo4j Desktop，只要保证 `.env` 中的连接信息正确即可。

### 2. 安装依赖

```powershell
pip install -r requirements.txt
```

### 3. 清洗并导入数据

```powershell
python scripts/clean.py
python scripts/import_neo4j.py --full
```

### 4. 启动服务

```powershell
.\scripts\start_web.ps1
```

### 5. 打开页面

- 前端界面：`http://127.0.0.1:8000`
- API 文档：`http://127.0.0.1:8000/docs`
- 健康检查：`http://127.0.0.1:8000/health`

## API 概览

| 端点 | 方法 | 说明 |
|------|------|------|
| `/recommend` | GET | 疾病治疗方案推荐 |
| `/drug` | GET | 药品详情查询 |
| `/interactions` | GET | 相互作用检测（逗号分隔药名） |
| `/interactions/check` | POST | 相互作用检测（JSON） |
| `/contraindications` | GET | 禁忌查询 |
| `/comorbidity` | POST | 合并症方案分析 |
| `/graph` | GET | 关联图谱子图数据 |
| `/qa` | POST | 规则问答接口 |
| `/stats` | GET | 图谱统计信息 |

## 数据规模

当前示例数据大致包括：

- 15 种药品
- 10 种疾病
- 9 套治疗方案
- 10 组药物相互作用
- 15 条禁忌记录
- 25 条不良反应
- 14 条用法用量

这个规模不大，但非常适合作为知识图谱原型项目和教学项目。

## 测试与评估

### 单元测试

```powershell
pytest tests/ -v
```

### 简单端到端评估

在 API 已运行、数据已导入的前提下：

```powershell
python scripts/eval.py
```

当前评估主要覆盖：

- 推荐结果是否返回关键药物
- 相互作用结果是否命中预期严重程度
- 问答模块是否识别正确意图

## 如果你是初学者，建议这样读代码

推荐阅读顺序：

1. `README.md`
2. `docs/ONTOLOGY.md`
3. `data/raw/*.csv`
4. `scripts/clean.py`
5. `scripts/import_neo4j.py`
6. `api/cypher.py`
7. `api/main.py`
8. `api/qa.py`
9. `web/index.html` 和 `web/js/app.js`

这样会更容易理解“数据是怎么变成图谱，再怎么变成接口和页面”的。

## 项目边界与改进方向

这个项目当前更偏“演示型原型”，而不是生产级临床系统。它已经适合用于：

- 面试作品展示
- Neo4j/FastAPI 学习
- 小型知识图谱课程作业
- GraphRAG 前置练习

如果要继续升级，可以考虑：

- 引入更完整的标准医学编码体系，例如 RxNorm、SNOMED CT
- 扩展更多疾病、药品和患者分层信息
- 增加权限控制、日志和异常监控
- 将规则问答升级为 `LLM + GraphRAG`
- 补齐更系统的测试、部署和 CI 流程

## 仓库建议描述

如果你要在 GitHub 仓库顶部填写一句简短介绍，建议用这句：

> 基于 Neo4j 和 FastAPI 的医药知识图谱项目，支持治疗方案推荐、药物相互作用检测、禁忌查询与规则问答。

