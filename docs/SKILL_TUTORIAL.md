# BrowserPilot 技能 (Skill) 与 Drill 脚本编写指南

本文档旨在介绍如何在 BrowserPilot 中编写自定义的 **技能 (Skill)**，以及如何配置确定性批处理的 **Drill 脚本 (JSON)**。

---

## 1. 什么是技能 (Skill)

在 BrowserPilot 中，**技能 (Skill)** 是一种引导 AI 执行复杂业务流程的 Prompt 拓展机制：
1. 它保存在客户端设置的 `"skills"` 模块中。
2. 任务启动时，系统会将对应的 Skill 描述文本拼接到 AI 的 Prompt 里。
3. 技能描述通常由两部分组成：
   * **前置引导与业务决策逻辑**：用自然语言告诉 AI 应该满足哪些筛选条件、如何判定合格性、使用何种沟通策略。
   * **后置 Drill 批处理脚本**：给出一个具体的 JSON 脚本模板，指示 AI 在前置准备完成后，必须调用 `execute_drill` 工具来进入无 AI 参与的确定性高速批处理阶段。

---

## 2. Drill 脚本架构 (Drill DSL)

Drill 脚本是一个声明式的 JSON 结构，用于指示浏览器自动执行“在主页面提取条目列表 -> 逐个对条目进行深度页面操作”的流程。

### 脚本结构
脚本主要由两部分管道组成：
* **`steps` (页面级管道)**：在主页面上运行一次，用于页面滚动、触发加载和数据提取。
* **`item_steps` (条目级循环管道)**：对 `steps` 中提取出的每一个数据条目，循环执行的一系列操作（如开启新标签页、抓取详情、AI 判断、发送消息）。

```json
{
  "task_name": "任务名称",
  "loop_count": 1,
  "steps": [
    // 页面级操作（滚动、提取）
  ],
  "item_steps": [
    // 循环级操作（导航、提取、AI判断、动作执行）
  ]
}
```

---

## 3. 页面级步骤 (steps) 可用动作

### 1. `infinite_scroll` (无限滚动)
让页面不断向下滚动以加载更多动态条目。
* **`selector_value`**：页面加载的卡片 CSS 选择器（例如 `".job-card-wrap"`）。
* **`selector_options`**：
  * `"max_unchanged"`：页面元素数量连续无变化多少次后停止滚动，默认 `2`。
  * `"wait_ms"`：每次滚动的延迟等待时间，默认 `2000`。

### 2. `extract_list` (提取数据列表)
提取页面卡片信息并存入条目上下文（`itemContext`）。
* **`selector_value`**：卡片元素的 CSS 选择器（例如 `".job-card-wrap"`）。
* **`record_fields`**：需要提取的字段名称与其提取路径表达式的映射。
  * 提取子元素属性：`"href": "a.job-name@href"`
  * 提取卡片自身属性：`"href": "@href"`（即选择器留空）
  * 提取子元素文本：`"title": "span.name"`

> [!WARNING]
> **关于 N/A 容错**：若指定选择器未匹配到卡片子元素，对应的字段值会被记录为 `"N/A"`。后续的 `navigate` 或逻辑判断需注意防范空路径或 N/A 错误。

---

## 4. 条目级步骤 (item_steps) 可用动作

对 `extract_list` 提取出的每一个条目，引擎会顺序执行以下动作序列：

### 1. `open_tab` (打开新标签页)
* 在浏览器中新建一个空白标签页，并将标签页 ID 绑定到当前条目的执行上下文中。

### 2. `navigate` (导航)
* **`url`**：静态网页地址。
* **`url_from`**：动态 URL 来源。指定 `itemContext` 中的字段名（例如 `"href"`）。
* **`url_prefix`**：可选的前缀。会自动与 `url_from` 的值进行拼接（例如 `"https://www.zhipin.com"`）。

### 3. `wait_for` (等待页面元素)
* **`selector_value`**：等待其出现的 CSS 选择器。
* **`selector_options`**：
  * `"wait_ms"`：如果不填 `selector_value`，则在此处进行固定时间的硬等待（毫秒）。

### 4. `fetch_content` (抓取文本内容)
* **`selector_value`**：目标内容的 CSS 选择器。
* **`selector_fallbacks`**：可选。主选择器找不到时，备用的 CSS 选择器列表。
* **`store_as`**：存入条目上下文中的键名（例如 `"jd_content"`）。

### 5. `ask_ai` (调用 AI 判定)
在后台调用大模型对提取到的页面内容进行智能分析。
* **`content_from`**：输入内容对应的 `store_as` 键名。
* **`instruction`**：判定提示词。**必须指示 AI 返回带有所需字段的 JSON 对象**。
* **`ai_result_key`**：判定结果的对象前缀名（例如 `"judge"`）。通过 `{judge.match}` 和 `{judge.greeting}` 引用 AI 的决策。

### 6. `abort_item` (中断条件)
根据上一阶段的决策过滤不需要的条目。
* **`condition`**：表达式判定条件（例如 `"{judge.match} == false"`）。
* **`then`**：条件成立时的收尾动作（通常设为 `"close_tab"` 避免标签页残留）。

### 7. `click` (点击元素)
* **`playwright_api`**：指定为 `"locator"`。
* **`selector_value`**：按钮或点击目标的 CSS 选择器。

### 8. `type_slowly` (人性化模拟键入)
* **`selector_value`**：输入框的 CSS 选择器。
* **`action_value`**：待输入的文本（可使用占位符，例如 `"{judge.greeting}"`）。
* **`selector_options`**：
  * `"min_delay_ms"`：字间最小延迟，默认 `30`。
  * `"max_delay_ms"`：字间最大延迟，默认 `100`。

### 9. `close_tab` (关闭标签页)
* 关闭当前条目的详情标签页，释放浏览器内存。

---

## 5. 完整实战模板：`boss_auto_apply` (Boss直聘自动搜索筛选与批量投递)

下面是一个经过完整验证的 Boss 直聘自动筛选投递技能示例：

### 5.1 前置人设与决策策略
这部分作为 Skill 的前置 Markdown 描述输入到应用设置中，用于告诉 AI 如何处理前置筛选以及在何时触发 Drill 脚本。

```markdown
**在前置筛选阶段（步骤1~4），严禁使用写死的 CSS 选择器，必须通过 `browser_snapshot` 和视觉图像定位元素并操作。**

---

## 阶段一：自主感知与前置筛选（Aria 快照/视觉定位）

1. **导航至首页**：使用 `browser_navigate` 导航至 `https://www.zhipin.com`。
2. **搜索职位**：
   - 调用 `browser_snapshot` 获取页面快照，识别搜索框元素并用 `browser_type` 输入职位关键字。
   - 识别搜索按钮并用 `browser_click` 点击，或发送 `Enter` 键触发搜索。
3. **筛选城市与工作区域**：
   - 在搜索结果页，点击城市选择下拉菜单。
   - **地级市与县级市级联筛选**：若目标城市是县级市（例如“公主岭”），先在城市搜索框中输入并选择其上级地级市（例如“长春”）。待页面加载后，在“工作区域”或区域筛选栏中，找到并点击选择该县级市（例如“公主岭”）。
4. **筛选薪资及其它条件**：
   - 识别“薪资待遇”等筛选下拉菜单，点击并选择符合要求的薪资区间。

---

## 阶段二：触发 Drill 自动化批处理

当所有前置筛选条件全部选择完毕且搜索结果刷新后，**你必须立即下发 `execute_drill` 工具调用**，触发底层的高效并发脚本执行引擎。

请输出以下 exact 格式的 JSON 动作调用（注意替换 `tabId` 为当前活动 Tab 的 ID）：
```

### 5.2 JSON 脚本
以下是上述技能中，AI 应该自动生成的 `execute_drill` 脚本实体：

```json
{
  "tabId": "T1",
  "execute_drill": {
    "script": {
      "task_name": "Boss直聘自动化投递任务",
      "loop_count": 1,
      "steps": [
        {
          "step_name": "无限滚动加载工作列表",
          "action": "infinite_scroll",
          "selector_value": ".job-card-wrap",
          "selector_options": {
            "max_unchanged": "3",
            "wait_ms": "2000"
          }
        },
        {
          "step_name": "提取所有职位卡片链接",
          "action": "extract_list",
          "selector_value": ".job-card-wrap",
          "record_fields": {
            "href": "a.job-name@href"
          }
        }
      ],
      "item_steps": [
        {
          "step_name": "开启详情标签页",
          "action": "open_tab"
        },
        {
          "step_name": "访问职位详情页",
          "action": "navigate",
          "url_from": "href",
          "url_prefix": "https://www.zhipin.com"
        },
        {
          "step_name": "等待详情页面加载",
          "action": "wait_for",
          "selector_value": ".job-detail"
        },
        {
          "step_name": "抓取职位描述",
          "action": "fetch_content",
          "selector_value": ".job-sec-text",
          "selector_fallbacks": [".job-detail", ".job-description"],
          "store_as": "jd_content"
        },
        {
          "step_name": "AI评估匹配度与生成打招呼语",
          "action": "ask_ai",
          "content_from": "jd_content",
          "instruction": "请仔细分析该职位的描述（JD），判断其是否符合我的简历要求，并生成最合适的打招呼内容。必须以 JSON 格式返回，包含两个字段：\"match\"（布尔值，是否符合并决定投递）、\"greeting\"（字符串，打招呼的开场白）。格式示例：{\"match\": true, \"greeting\": \"您好，我对该职位非常感兴趣...\"}",
          "ai_result_key": "judge"
        },
        {
          "step_name": "过滤不匹配项并关闭Tab",
          "action": "abort_item",
          "condition": "{judge.match} == false",
          "then": "close_tab"
        },
        {
          "step_name": "点击立即沟通",
          "action": "click",
          "playwright_api": "locator",
          "selector_value": ".btn-container .btn-startchat"
        },
        {
          "step_name": "等待对话输入框加载",
          "action": "wait_for",
          "selector_value": ".input-area",
          "selector_options": {
            "wait_ms": "3000"
          }
        },
        {
          "step_name": "逐字输入打招呼语",
          "action": "type_slowly",
          "selector_value": ".input-area",
          "action_value": "{judge.greeting}",
          "selector_options": {
            "min_delay_ms": "30",
            "max_delay_ms": "100"
          }
        },
        {
          "step_name": "点击发送按钮",
          "action": "click",
          "playwright_api": "locator",
          "selector_value": ".send-message"
        },
        {
          "step_name": "完成关闭详情页",
          "action": "close_tab"
        }
      ]
    }
  }
}
```
