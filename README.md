# Pay One Context

用 Markdown 持续整理资料和问答的 LLM Wiki。读者从[索引](wiki/index.md)进入，Agent 按[维护约定](AGENTS.md)归档原文、更新主题并维护链接。

## 使用接口

将仓库目录打开为 Agent 的工作区，再发出下列请求。

| 操作 | 可以对 Agent 说 | 结果 |
| --- | --- | --- |
| 收录 | 「把这份资料收进 Wiki」并附上文件、链接或原文 | 来源页、更新过的主题和待核事项。 |
| 查询 | 「查 Wiki 里关于某主题的结论，附出处」 | 答案、适用范围和来源链接。 |
| 沉淀 | 「把刚才的结论存进 Wiki」 | 更新过的主题页链接。 |
| 检查 | 「检查 Wiki 的断链、过期结论和冲突」 | 问题位置、已修正项、待核事项和日志记录。 |

原文放在 [`raw/`](raw/)，来源摘要放在 [`wiki/sources/`](wiki/sources/)，综合结论放在 [`wiki/topics/`](wiki/topics/)。通过[日志](wiki/log.md)查看历次更新。需要收录内部或个人资料时，请使用独立的本地副本或私有仓库。

设计参考 [Andrej Karpathy 的 LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)。
