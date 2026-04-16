---
name: quarterly-review-html-generator
description: 提取工作流水账和重点事项，直接生成带有高保真架构决策图排版样式的 HTML 格式季度回顾报告，支持直接导出 PDF。
---

# 核心指令 (Instructions)

当用户需要生成阶段性汇报时调用此 Skill。你的唯一输出是一段完整的、可独立运行的 HTML 代码。严禁在 HTML 代码块外部输出任何解释性 Markdown 文本。

## 1. 角色与处理规则
- **架构师语境**：使用“沉淀、演进、赋能、闭环、体系化、ROI”等词汇。
- **持续建设**：强调系统的“持续建设”和“迭代演进”，绝对禁止使用“重构”、“推翻”等否定前期基建的词汇。
- **思维升维**：以“Thinking (高阶思考)”替代传统的“反思 (Reflection)”，聚焦下一阶段的技术与业务架构推演。
- **精准降噪**：严格根据用户指定的 `[重点事项]` 从流水账中提取信息，抛弃无关细节。

## 2. 输入规范
- `core_goals`: 本季度的核心指标或愿景（用于副标题）。
- `timeline_events`: 3 个关键时间节点及其对应的动作（用于时间轴）。
- `initiatives`: 2-3 个核心卡片模块（如逻辑实现、方案争议等）。
- `action_items`: 具体的任务清单（包含：行动项、负责人、时间、状态）。
- `thinking`: 对下一步的顶层思考或核心共识。

## 3. 输出结构 (HTML Template)
你必须严格使用以下 HTML 和 CSS 模板生成最终内容。只需将 `{...}` 中的占位符替换为用户输入的实际提炼内容即可，保持 CSS 样式不变，以确保完美的视觉呈现。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>阶段回顾报告</title>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; background-color: #f8f9fa; color: #333; line-height: 1.6; margin: 0; padding: 40px; }
        .page-container { max-width: 960px; margin: 0 auto; background: #fff; padding: 40px 50px; box-shadow: 0 4px 20px rgba(0,0,0,0.05); border-radius: 8px; }
        
        /* 标题区 */
        h1 { font-size: 28px; color: #2c3e50; margin-bottom: 5px; font-weight: 600; }
        .subtitle { font-size: 14px; color: #7f8c8d; margin-bottom: 40px; border-bottom: 1px solid #eee; padding-bottom: 20px; }
        
        /* 模块标题 */
        .section-header { display: flex; align-items: center; font-size: 18px; font-weight: bold; margin: 30px 0 20px 0; }
        .section-header::before { content: ''; display: inline-block; width: 12px; height: 12px; margin-right: 10px; border-radius: 2px; }
        .c-brown::before { background-color: #a0816b; }
        .c-pink::before { background-color: #d16b8a; }
        .c-purple::before { background-color: #7b68ee; }
        .c-green::before { background-color: #46a069; }
        .c-blue::before { background-color: #3476d2; }

        /* 时间轴 */
        .timeline-container { display: flex; justify-content: space-between; position: relative; padding-top: 20px; margin-bottom: 40px; }
        .timeline-line { position: absolute; top: 0; left: 0; width: 100%; height: 2px; background-color: #a0816b; }
        .timeline-item { flex: 1; padding-right: 20px; position: relative; }
        .timeline-item::before { content: ''; position: absolute; top: -24px; left: 0; width: 8px; height: 8px; background: #a0816b; border-radius: 50%; border: 3px solid #fff; }
        .time-date { font-weight: bold; color: #444; margin-bottom: 10px; font-size: 15px; }
        .time-desc { font-size: 13px; color: #666; margin: 0; padding-left: 15px; position: relative; }
        .time-desc li { margin-bottom: 5px; list-style-type: disc; }

        /* 卡片网格 */
        .card-grid { display: flex; gap: 20px; margin-bottom: 40px; }
        .card { flex: 1; padding: 20px; border-radius: 8px; font-size: 14px; }
        .card h3 { margin-top: 0; font-size: 15px; display: flex; align-items: center; }
        .card ul { padding-left: 20px; margin-bottom: 0; }
        .card li { margin-bottom: 8px; }
        
        .card-pink { background-color: #fcf4f6; border: 1px solid #f6dde4; color: #c0392b; }
        .card-pink h3 { color: #d16b8a; }
        .card-purple { background-color: #f7f5fb; border: 1px solid #e6e0f6; color: #4a235a; }
        .card-purple h3 { color: #7b68ee; }
        .card-green { background-color: #f2fbf5; border: 1px solid #d5ecd8; color: #145a32; }
        .card-green h3 { color: #46a069; }

        /* 数据表格 */
        table { width: 100%; border-collapse: collapse; margin-bottom: 40px; font-size: 14px; }
        th { background-color: #faeaea; color: #a94442; text-align: left; padding: 12px 15px; font-weight: 600; border-bottom: 2px solid #f2d7d7; }
        td { padding: 12px 15px; border-bottom: 1px solid #eee; color: #333; }
        tr:nth-child(even) { background-color: #fafafa; }

        /* 核心共识 Box */
        .consensus-box { background-color: #f0f7ff; border: 1px solid #cce0ff; border-radius: 8px; padding: 20px; display: flex; align-items: flex-start; gap: 15px; }
        .consensus-box .icon { font-size: 20px; color: #3476d2; margin-top: 2px; }
        .consensus-content { font-size: 14px; color: #2b579a; margin: 0; font-weight: 500; }
        .highlight { color: #3476d2; font-weight: bold; }

    </style>
</head>
<body>
    <div class="page-container">
        <h1>{报告大标题，例如：阶段效能演进与架构持续建设决策}</h1>
        <div class="subtitle">{核心目标：提取用户的核心目标作为副标题}</div>

        <div class="section-header c-brown">版本发布与持续建设节奏</div>
        <div class="timeline-container">
            <div class="timeline-line"></div>
            <div class="timeline-item">
                <div class="time-date">{节点1时间}</div>
                <ul class="time-desc">
                    <li>{动作描述点1}</li>
                    <li>{动作描述点2}</li>
                </ul>
            </div>
            <div class="timeline-item">
                <div class="time-date">{节点2时间}</div>
                <ul class="time-desc">
                    <li>{动作描述点1}</li>
                    <li>{动作描述点2}</li>
                </ul>
            </div>
            <div class="timeline-item">
                <div class="time-date">{节点3时间}</div>
                <ul class="time-desc">
                    <li>{动作描述点1}</li>
                    <li>{动作描述点2}</li>
                </ul>
            </div>
        </div>

        <div class="card-grid">
            <div class="card card-pink">
                <h3>{事项标题 A，如：日常开发逻辑与实现}</h3>
                <ul>
                    <li>{提炼要点 1：突出动作}</li>
                    <li>{提炼要点 2：突出价值/收益}</li>
                </ul>
            </div>
            <div class="card card-purple">
                <h3>{事项标题 B，如：接口层或 A2A 协议方案}</h3>
                <ul>
                    <li>{提炼要点 1：突出架构演进}</li>
                    <li>{提炼要点 2：突出系统沉淀}</li>
                </ul>
            </div>
        </div>

        <div class="card-grid">
            <div class="card card-green">
                <h3>{事项标题 C，如：效能提升识别}</h3>
                <ul>
                    <li>{提炼要点 1：定量收益}</li>
                    <li>{提炼要点 2：定性目标完成度}</li>
                </ul>
            </div>
        </div>

        <div class="section-header c-pink">关键行动项与推进列表</div>
        <table>
            <thead>
                <tr>
                    <th>行动项</th>
                    <th>负责人</th>
                    <th>时间节点</th>
                    <th>状态</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>{行动项详情 1}</td>
                    <td>{对应负责人}</td>
                    <td>{日期/节点}</td>
                    <td>{进行中/待推进/已完成}</td>
                </tr>
                <tr>
                    <td>{行动项详情 2}</td>
                    <td>{对应负责人}</td>
                    <td>{日期/节点}</td>
                    <td>{状态}</td>
                </tr>
            </tbody>
        </table>

        <div class="section-header c-blue">阶段高阶思考 (Thinking) 与核心共识</div>
        <div class="consensus-box">
            <div class="icon">◎</div>
            <p class="consensus-content">
                <span class="highlight">核心架构推演：</span>{将基于当前基建的顶层思考输出于此。必须体现对下一阶段持续建设的主线规划，避免使用纠错式词汇}
            </p>
        </div>
    </div>
</body>
</html>
