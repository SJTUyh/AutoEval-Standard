# AutoEval-Standard

模型测评规范仓库。目录结构为 `{模型名称}/{测评基准名称}/`，每个基准目录包含两个文件：

| 文件 | 作用 |
| --- | --- |
| `baseline.json` | 目标得分：`median`（目标中位数）与 `tolerance` / `range`（允许波动范围），单位与得分一致（百分点）。`source` 记录模型卡出处与官方测评设置。 |
| `standard_config.json` | 测评基准的标准参数子集（`tool` + `subcommand` + `params`），只包含影响测评结论的字段。 |

## 使用方式

1. 读取 `standard_config.json`。
2. AutoEval-Benchmark-Adaptor 会根据 `tool` / `subcommand` 自动加载对应的内置配置模板，并用 `standard_config.json` 覆盖同名参数（对象逐键合并，数组与标量整体替换，`standard_config.json` 优先）。
3. 执行：

   ```bash
   autoeval-adaptor --config standard_config.json
   ```

4. 多次运行的得分中位数应落在 `baseline.json` 的 `range` 内；超出范围即判定为不符合测试规范。

`standard_config.json` 只锁定数据集、脚手架、采样参数、上下文与输出长度、尝试次数等标准参数。模型服务地址、模型名、并发数、`work_dir` 等运行参数由适配器模板提供默认值；其中 `work_dir` 等可通过 CLI（如 `--work-dir`）覆盖，模型服务信息可直接在 `standard_config.json` 中按实际环境补充。
