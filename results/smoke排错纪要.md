# 小样本验证与排错纪要

正式全量前，按低耗配置对金融分析实例 1 跑三组。三组均生成了 `/root/answers.json`，官方 verifier 能判分，且评分记录均已保存。无 Skill 与 one-shot 的答案质量失败属于任务结果，人工 Skill 通过。小样本记录单独保留，正式 69 组不包含这些结果。

| 配置 | 官方结果 | 关键检查或现象 | 求解 token |
|---|---|---|---:|
| none | FAIL | `test_answer_quality` 显示持仓题的 `q2_answer` 为 3457，期望 2451 | 98,386 |
| human_authored | PASS | 先尝试的 `/root/scripts/search_fund.py` 不存在，后续完成答案，全部检查通过 | 198,094 |
| b1-one-shot | FAIL | 遇到系统 Python 包安装限制、`python: not found`、TSV 混合类型告警，后续写出答案。官方 q2 同样为 3457 对 2451 | 103,442 |

## 排查记录

1. **模型和 Skill 链路。** 比较三组 `agent/qwen.txt` 的 `skill_context`、工具调用和模型响应，核对无 Skill 为空、人工与生成组有内容。通过 `skill_source.json` 回查实际来源。
2. **路径与 Python。** 轨迹区分了找不到脚本、解释器命令失败和任务结果失败。人工组最终完成写答。one-shot 在遇到环境错误后也生成了答卷，官方最终停在 q2 数值检查。环境告警本身无法解释 3457 与 2451 的差值。
3. **独立评分接口。** 上游 Skill 质量脚本使用 Responses 接口，本地适配器最初只转接 Chat Completions，人工组评分收到 403。修复后把文本评分请求转接到同一百炼接口，补齐评分而不重新求解。评分请求与响应保存在 `metrics/judge_raw/`。
4. **恢复与去重。** 恢复期间出现重叠的 one-shot 尝试和输出目录识别冲突。正式实验使用的 smoke 记录以最终归档的 manifest 为准，保留有效结果和已有生成 Skill。该历史调度错误与 verifier 的答案错误分别记录。
5. **进入全量的条件。** 三组均有轨迹、官方 reward、检查明细以及完整指标。通过条件针对评估链路，允许模型答错。上述条件满足后才启动 23 × 3 组正式实验。

## 如何复核

解压 [raw-evidence.tar.gz](raw-evidence.tar.gz)，查看 `output/qwen_budget_campaign/manifest.json` 的 `smoke` 节点。三组对应的目录如下，目录下的 `trials/` 包含 `agent/qwen.txt`、`verifier/ctrf.json` 和 `verifier/reward.txt`，同级的 `metrics/` 包含指标文件。

| 配置 | 归档内目录 |
|---|---|
| none | `output/qwen_smoke/20261118-110259-7825de` |
| human_authored | `output/qwen_smoke/20261118-110626-6f6bce` |
| b1-one-shot | `output/qwen_smoke/20261118-111815-f25d3d` |

原始归档主要保留有效尝试及其评分，历史控制器报错来自本地排错记录。关于 q2 的原因及证据边界见[正式失败案例复盘](正式失败案例复盘.md)。
