# 第一次抓取演练 | First fetch & serve drill

从零到跑通三档用法，按顺序来：

1. 装并验二进制：`curl -L -o lightpanda https://github.com/lightpanda-io/browser/releases/download/nightly/lightpanda-x86_64-linux && chmod a+x ./lightpanda`，先跑 `./lightpanda version` 确认能执行（报 `cannot execute` 多半是 libc 不对，换 glibc 底的系统/镜像）。
2. 先关遥测再干活：`export LIGHTPANDA_DISABLE_TELEMETRY=true`。
3. 一次性抓取试水：`./lightpanda fetch --obey-robots --dump markdown https://demo-browser.lightpanda.io/campfire-commerce/`，能出 Markdown 正文说明 JS 执行与 DOM 都通了；再试 `--dump structured_data` 看结构化数据输出。
4. 起 CDP 服务换脚本上：`./lightpanda serve --host 127.0.0.1 --port 9222`，把现有 Puppeteer 脚本的连接方式换成 `puppeteer.connect({ browserWSEndpoint: 'ws://127.0.0.1:9222' })` 跑一遍原有流程，对比内存与耗时。
5. 复杂站点先小批量：拿目标站点跑 10 个页面试水，确认没有缺失的 Web API 再放量；凡是要真截图的环节，提前从流程里剔除或留给真 Chromium。
