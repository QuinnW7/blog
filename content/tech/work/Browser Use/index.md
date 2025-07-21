+++
date = '2025-07-11T10:58:00+08:00'
draft = false
title = '如何本地快速搭建Browser Use'
+++
本文介绍了browser Use的定义， 展示参考manus行为的demo搭建的过程
- 当前使用playwright操作浏览器
<!--more-->


“Browser Use”（或更具体的可能是 **Web Agent / Browser Automation**）是智能体领域一个非常热门且实用的方向。它意味着让AI Agent能够像人一样操作浏览器，完成订票、数据抓取、在线购物等复杂任务。

这个主题的写作难点在于，它横跨了前端技术（DOM）、后端架构（Orchestration）和AI模型（LLM），很容易写得散乱。

因此，**清晰的行文组织和叙事线索**至关重要。你要做的不是罗列技术，而是**讲一个引人入胜的故事**：我们如何赋予AI一双“眼睛”和一个“鼠标手”，让它驰骋于互联网的。

### 核心叙事线索：一个AI“上网高手”的诞生记

我们将文章组织成一个逐步进化的故事，从一个“瞎子”AI，一步步成长为能熟练操作浏览器的“数字员工”。

---

### 文章标题建议 (选一个或结合)

*   **直击痛点型**: 《让AI帮你点外卖：从0到1构建浏览器智能体(Browser Agent)》
*   **技术揭秘型**: 《解构Browser Agent：我们如何让LLM像人一样操作浏览器？》
*   **架构深度型**: 《企业级浏览器智能体架构：挑战、设计与实践》

---

### 行文结构与内容组织 (四段式进化论)

#### **第一幕：AI的“视力”问题 —— 为什么需要Browser Agent？(Why)**

**目标**: 吸引读者，建立问题场景，让读者明白这件事的价值和挑战。

1.  **开篇引入一个生动场景**:
    *   “想象一下，你对AI说：‘帮我查一下明天从上海到北京最便宜的机票，并在XX网站上预订’。在过去，这几乎是天方夜谭。AI无法‘看到’网页，更无法‘点击’按钮。它被困在了API的牢笼里。”
    *   引出核心矛盾：LLM拥有强大的推理能力，但缺乏与广大、非结构化的Web世界交互的能力。传统的API调用模式覆盖面太窄。

2.  **点明挑战 (The Challenges)**:
    *   **非结构化之痛**: 网页是为人类视觉设计的，充满了动态内容、复杂的DOM结构和广告干扰。
    *   **交互之难**: 点击、滚动、输入、处理弹窗... 这些对人来说轻而易举的动作，对程序来说却异常复杂。
    *   **状态之殇**: Web操作是带状态的（登录、购物车），如何让Agent拥有“会话记忆”？

3.  **提出我们的使命**:
    *   “本文将带你探索，我们如何通过构建一个强大的Browser Agent，为AI装上‘眼睛’和‘手’，打破数字世界的壁垒。”

#### **第二幕：赋予AI“眼睛”和“手” —— 核心技术组件拆解 (What & How)**

**目标**: 这是文章的技术核心。分模块、逐一介绍实现一个Browser Agent所需的关键组件，并配上架构图。

1.  **整体架构图先行**:
    *   先放一张清晰的High-Level架构图，让读者对全貌有个概念。图中应包含：**用户 -> 编排服务(Orchestrator) -> LLM -> 浏览器控制服务 -> 目标网站**。

2.  **组件一：浏览器控制器 (The "Hand")**
    *   **技术选型**: 介绍你是如何控制浏览器的。对比几种主流方案的优劣：
        *   **Playwright/Puppeteer**: 为什么选择它们？（功能强大、社区活跃、支持主流浏览器）。
        *   **Selenium**: 为什么可能不是首选？（相对老旧，API不够现代化）。
    *   **关键操作封装**: 你肯定不会直接暴露原始API给Agent。展示你是如何将底层操作封装成更语义化的行动（Action）的，如 `click(element)`, `type(text, element)`, `scroll(direction)`。

3.  **组件二：网页“阅读器” (The "Eyes")**
    *   **核心问题**: 如何将复杂的网页DOM，转换成LLM能理解的、简洁的文本信息？
    *   **你的解决方案 (这部分是展现技术深度的关键)**:
        *   **方案A (简单暴力)**: 直接提取 `<body>` 的 `innerText`。**缺点**: 丢失结构，信息太多。
        *   **方案B (基于可交互元素)**: 提取所有可点击、可输入的元素（`<a>`, `<button>`, `<input>`），并给它们打上ID。例如：“页面上有以下元素：[1: 登录按钮], [2: 搜索输入框], ...”。
        *   **方案C (高级，结合视觉)**: 使用多模态模型（如GPT-4V）直接“看”截图，或者利用工具（如`set-of-mark`）在截图上打标记。
    *   **对比与权衡**: 分析不同方案在成本、准确率、信息密度上的trade-off。

4.  **组件三：决策大脑 (The "Brain")**
    *   **Prompt设计**: 展示你的核心Prompt模板。它需要包含：
        *   **任务目标 (Objective)**: “你的任务是预订一张机票”。
        *   **当前页面信息 (Observation)**: “你现在在XX网站首页，页面上有[1:出发地输入框], [2:目的地输入框]...”。
        *   **可用行动 (Available Actions)**: `click(id)`, `type(id, text)`。
        *   **思考链 (Chain of Thought)**: 要求模型输出思考过程和下一步行动。
    *   **模型选型**: 为什么选择某个特定的LLM？（如GPT-4o的函数调用能力，还是Claude 3的超长上下文处理能力？）

#### **第三幕：实战演练 —— 一次“订机票”的旅程 (Case Study)**

**目标**: 让理论落地，通过一个完整的例子，把所有组件串起来，展示你的系统是如何工作的。

1.  **设定任务**: `“帮我在X程网上，查找明天从上海到北京的机票”`
2.  **分步展示 (Step-by-Step)**:
    *   **Step 1: 打开首页**: `Action: goto("https://www.ctrip.com")`
    *   **Step 2: 观察与思考**:
        *   `Observation`: “页面上有[1:出发城市], [2:到达城市], [3:查询按钮]...”
        *   `Thought`: “我需要先在出发城市输入‘上海’，然后在到达城市输入‘北京’，最后点击查询按钮。”
        *   `Action`: `type(1, "上海")`
    *   **Step 3: 执行与反馈**: 编排服务调用Playwright执行`type`操作，并返回新的页面状态。
    *   **...循环这个过程**: 直到任务完成或失败。用流程图或序列图来展示这个`Observe -> Think -> Act`的循环。

#### **第四幕：从“能用”到“好用” —— 挑战与未来展望 (Challenges & Future)**

**目标**: 展示你作为资深工程师的思考深度，不仅满足于实现功能，更关注生产环境的健壮性和未来的发展。

1.  **我们踩过的坑 (The "Gotchas")**:
    *   **动态内容与等待**: 如何处理AJAX加载和页面SPA路由跳转？（分享你的等待策略）。
    *   **错误处理与恢复**: 如果一次点击失败了，或者页面出现了预料之外的弹窗，Agent如何自愈或重试？
    *   **性能与成本**: 控制一个浏览器实例（尤其是带头模式）非常耗费资源，如何进行池化管理和优化？LLM的调用成本如何控制？
    *   **对抗与检测**: 很多网站有反爬虫机制，如何让Agent的行为更像“人”？

2.  **未来展望 (The Future)**:
    *   **多模态融合**: 视觉将是关键，让Agent直接“看”懂页面。
    *   **自我学习与反思**: Agent能否从失败的经验中学习，自动优化其操作策略？
    *   **更复杂的长期任务**: 如何让Agent处理需要数天才能完成的任务（如监控商品降价并自动下单）？

通过这样的行文结构，你的文章将不再是技术的堆砌，而是一个逻辑清晰、引人入ชม的故事，既有广度（覆盖所有组件），又有深度（探讨具体实现和挑战），还能展现你的思考高度。


















# 背景
作为Deep Research三大工具之一,另两个是Code和Search

# 原理
要让 AI Agent 完成任务，可以直接让 AI 浏览网页，像人一样去理解页面。
核心原理：观察-思考-行动（Observation-Thought-Action, OTA）循环，所有智能体的浏览器操作都可以归结为一个持续循环的框架，这与强化学习中的智能体与环境交互的模式非常相似。
- 观察 (Observation-O): 智能体需要“看到”并“理解”当前浏览器的界面。这是输入环节。
- 思考 (Thought-T): 基于当前的观察结果和最终的总任务目标，智能体（其核心是LLM）进行推理和规划，决定下一步应该做什么。这是决策环节。
- 行动 (Action-A): 智能体执行上一步决策出的具体操作，例如点击按钮、输入文字、滚动页面等。这是输出环节。

# 业界调研
主流程：理解任务 -> 制定计划 -> 浏览网页 -> 观察和理解内容 -> 提取信息 -> 综合分析 -> 得出结论
## Manus



# 技术方案
## 流程图
![img.png](img.png)

# DockerFile

```bash
# 1. 使用更优的 node:slim 基础镜像
FROM node:22-bookworm-slim

# 2. 环境变量
ENV PW_VERSION=1.44.0
ENV DEBIAN_FRONTEND=noninteractive

# -- 用户和路径配置 --
ENV APP_USER=pwuser
ENV APP_HOME=/home/${APP_USER}

# -- VNC 和 Web VNC 配置 --
ENV VNC_RESOLUTION=1280x720
ENV NOVNC_PORT=6901
# 【变化】移除了 VNC_PASSWORD 环境变量，因为它不再被使用

# -- Playwright 服务配置 --
ENV PW_PORT=3000
ENV PW_BROWSER="chromium"
ENV PW_HEADLESS="false"

# 3. 精细的镜像瘦身
RUN echo 'path-exclude /usr/share/doc/*' > /etc/dpkg/dpkg.cfg.d/01_nodoc && \
    echo 'path-exclude /usr/share/man/*' >> /etc/dpkg/dpkg.cfg.d/01_nodoc

# 4. 创建用户
RUN useradd --create-home --shell /bin/bash --uid 1001 ${APP_USER}

# 5. 安装系统依赖
USER root
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    supervisor \
    # 【变化】移除了 tigervnc-common，因为它只包含不再需要的 vncpasswd
    tigervnc-standalone-server \
    novnc websockify \
    fluxbox xterm fonts-liberation \
    wget ca-certificates \
    && \
    echo "===== Installing Google Chrome Stable =====" && \
    wget -q -O /tmp/chrome.deb "https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb" && \
    apt-get install -y /tmp/chrome.deb && \
    npm config set cache /tmp/npm_cache_root --global && \
    npx --yes playwright@${PW_VERSION} install-deps && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /var/cache/apt/* /tmp/* /var/tmp/* /root/.npm

# 6. 复制所有配置文件和脚本
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY start-playwright-server.js ${APP_HOME}/start-playwright-server.js
# 【变化】移除了所有与 entrypoint.sh 相关的行
RUN chown ${APP_USER}:${APP_USER} ${APP_HOME}/start-playwright-server.js

# 7. 以非 root 用户身份安装项目级的 Playwright
USER ${APP_USER}
WORKDIR ${APP_HOME}
RUN npm install playwright@${PW_VERSION} && \
    npx playwright install && \
    rm -rf /tmp/* ~/.npm
# 切换回 root 用户来运行进程管理器
USER root
# 8. 暴露端口，并设置默认命令
EXPOSE ${NOVNC_PORT}
EXPOSE ${PW_PORT}
# 【变化】移除了 ENTRYPOINT，CMD 现在直接启动 supervisord
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
```









