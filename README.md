# SkillLearnBench 三组 Qwen 基线

固定上游版本 `726e1f79b96b0cb3d729d0e55a1c847e0b5353ae`。任务、原始 verifier、人工 Skill 与评分脚本由 prepare.py 从该版本获取。upstream-reference/ 提供同一版本的任务定义、verifier、评估源码和许可证，运行所需任务数据由 prepare.py 下载。覆盖地震板块计算 6 个实例、金融分析 6 个、加权 GDP 6 个和股票可视化 5 个，对比无 Skill、人工 Skill 与单轮生成 Skill。

## 运行

需要 Git、Python、Docker Desktop 和可调用的百炼北京区 API Key。以下 PowerShell 示例从本仓库运行，路径应为独立的空目录。

```powershell
python .\prepare.py --workspace 'E:\SkillLearnBench-Reproduction' --install
Set-Location 'E:\SkillLearnBench-Reproduction'
$env:DASHSCOPE_API_KEY = [Environment]::GetEnvironmentVariable('DASHSCOPE_API_KEY', 'User')
$env:SLB_STATE = 'qwen_budget_campaign'
$env:PYTHONUTF8 = '1'
.\.venv\Scripts\python.exe qwen
un_all.py
```

API Key 保存在本机环境中。新目录默认共享预算 20 元，小样本最多 3 元。三个分组在同一个金融实例完成求解及评分后才启动全量。预算耗尽时保留清单和账本。只启动一个调度器。进度见 `output/qwen_budget_campaign/manifest.json`。每次运行的轨迹、任务判分文件和评分请求响应见实验目录，聚合脚本为 `qwen/report_campaign.py`。

## 设置与口径

使用 qwen3.5-plus、temperature 0、关闭思考、单次输出上限 2048 token、任务最多 100 步。保留最近三轮完整交互，较旧交互留有限摘录。人工与生成 Skill 通过上下文注入，论文原版使用 CLI 自动加载。生成 Skill 按任务族复用。评分用同一模型族的独立请求，长轨迹在输入过大时使用头尾摘录，完整原始轨迹另存。官方判分在 Agent 执行结束后才复制进容器。

任务结果采用官方 verifier。没有 Skill 时质量指标记 N/A。显式 Skill 工具调用率和 shell 文件引用代理值分别统计。总 token 区分求解、技能生成和评分。费用账本采用公开无缓存单价和请求预留，实际账单以百炼控制台为准。

## 已交付结果

- [实验报告](results/实验报告.md)
- [69 组逐组指标](results/trials.csv)与[机器可读汇总](results/report.json)
- [失败分类及判分证据](results/failure_categories.csv)
- [原始日志、轨迹、模型响应及判分文件](results/raw-evidence.tar.gz)与[校验值](results/raw-evidence.sha256)

解压证据包后，可用逐组指标 run 字段中的 output/ 之后的路径定位记录。原始文件保留运行时路径。

### 排错与案例证据

- [小样本排错纪要](results/smoke排错纪要.md)
- [正式失败案例复盘](results/正式失败案例复盘.md)
