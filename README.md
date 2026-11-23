# LLM Wiki API

一个面向单个本地知识库的 HTTP API 草案。调用方提交 Markdown 原文，服务更新带出处的知识页。查询优先使用已有知识页，调用方可以把值得复用的回答保存成页面。接口定义见 [OpenAPI 3.1](openapi.yaml)。

| 方法 | 路径 | 结果 |
| --- | --- | --- |
| `POST` | `/v1/sources` | 归档原文，更新相关页面、索引和日志，返回来源 ID 和更新的页面 ID。 |
| `GET` | `/v1/sources/{sourceId}` | 读取原文和归档元数据。 |
| `GET` | `/v1/index` | 列出页面的 ID、标题、类型和摘要。 |
| `GET` | `/v1/pages/{pageId}` | 读取页面内容、引用和 `ETag`。 |
| `POST` | `/v1/query` | 根据已收录资料回答，返回 Markdown、逐条引用和待核问题。 |
| `PUT` | `/v1/pages/{pageId}` | 创建沉淀页或更新已有沉淀页，更新索引和日志。 |
| `POST` | `/v1/lint` | 返回断链、缺少引用和待核实的冲突等问题，追加检查日志。 |

## 从收录到沉淀

1. 调用 `POST /v1/sources`，提交 `title` 和 `markdown`。可用 `originUrl` 标注出处，服务仅把它当作元数据保存。此前已收录的资料有新版本时，可提交 `supersedesSourceId`。旧原文和旧引用仍可读取。
2. 调用 `POST /v1/query`，提交 `question`。正文中的 `[^1]` 对应 `citations` 的第一项，依此类推。引用通过 `sourceId` 和原文行号回查，可通过 `pageId` 查看对应的知识页。证据不足时返回 `insufficient_evidence`，并在 `openQuestions` 中列出待核问题。
3. 保存回答时，把回答用到的 `citations` 随 `title`、`summary` 和 `markdown` 提交给 `PUT /v1/pages/{pageId}`。创建使用 `If-None-Match: *`，更新使用 `GET` 返回的 `ETag` 作为 `If-Match`。页面在期间已变化时返回 `412`，调用方回读并合并后再保存。后续编辑前通过 `GET` 获取最新 `ETag`。

## 写入约定

- 原文按请求解码后的字符串保留，引用行号从 1 开始，以 LF 或 CRLF 划分行。`markdown`、`originUrl`、`supersedesSourceId` 三项完全相同时视为同一次收录，返回原收录结果，已归档标题沿用首次提交的值。
- `POST /v1/sources` 同步提交原文、页面、索引和日志。提交失败时保留上一个完整版本。若调用方超时，可重发同一请求确认结果。并行收录和页面更新以同一写入顺序提交。
- 合并知识页时，依据能区分出时间、版本或适用范围的，写明各自条件。仍需核实的分歧保留两边的出处。健康检查报告包含问题所在页面与证据，页面修正通过对应写入操作完成。
- `PUT` 的目标类型为 `note`。`source` 和 `topic` 页面由收录维护。查询读取一次提交的快照，健康检查持续读取同一快照直到报告生成。

当前范围为单个知识库和 UTF-8 Markdown。接口仍处于设计阶段，部署时由服务宿主负责访问控制。

设计参考 [Andrej Karpathy 的 LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)。
