# AutoEval-Standard

模型测评规范仓库。目录结构为 `{模型名称}/{测评基准名称}/`，每个基准目录固定包含三个文件：

| 文件 | 作用 |
| --- | --- |
| `baseline.json` | 目标得分：`median`（目标中位数）与 `tolerance` / `range`（允许波动范围），单位与得分一致（百分点）。`source` 记录模型卡出处与官方测评设置。 |
| `benchmark_config.json` | 一套完整、可直接交给 AutoEval-Benchmark-Adaptor 的启动配置（`tool` + `subcommand` + `params`）。 |
| `standard_config.json` | `benchmark_config.json` 中不可更改的标准参数子集，结构与 `benchmark_config.json` 相同，只含需要锁定的字段。 |

## 使用方式

1. 读取 `benchmark_config.json`。
2. 用 `standard_config.json` 覆盖同名参数（递归合并：对象逐键合并，数组与标量整体替换，`standard_config.json` 优先）。
3. 用合并后的配置执行 `autoeval-adaptor --config <合并结果>`。
4. 多次运行的得分中位数应落在 `baseline.json` 的 `range` 内；超出范围即判定为不符合测试规范。

`standard_config.json` 只锁定影响测评结论的参数（数据集、脚手架、采样参数、上下文与输出长度、尝试次数等）；主机地址、端口、并发数、`work_dir` 等环境相关参数保留在 `benchmark_config.json` 中，可按机器调整。

## 已知限制：MMMU-Pro 与远程 API

MMMU-Pro 的 `standard_config.json` 固定 `model_type=vllm_api_general_chat`。该模型配置会把数据集落盘的本地图片路径直接作为 `image_url` 发送，不会转换为内联 data URL；DeepSeek 等远程 OpenAI 兼容 API 只能访问 http(s) 或 data URL，因此请求在 warmup 阶段返回 HTTP 400（参考 AISBench `TINFER-RUNTIME-001`）。

在 Adaptor/AISBench 提供“本地图片转内联 data URL”之前：

- 该组合只适用于能直接读取本地文件路径的服务端；
- 远程 API 的失败结果不应作为模型能力结论或榜单成绩；
- 需要多模态支持时，应先获得维护者确认再调整标准锁定字段，不应在单次运行中绕过。

相关讨论：https://github.com/SJTUyh/AutoEval-Standard/issues/1
