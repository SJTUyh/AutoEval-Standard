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
