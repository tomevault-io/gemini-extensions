## moviepilotlite

> During you interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again.

# Instructions

During you interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again. 

You should also use the `.cursorrules` file as a scratchpad to organize your thoughts. Especially when you receive a new task, you should first review the content of the scratchpad, clear old different task if necessary, first explain the task, and plan the steps you need to take to complete the task. You can use todo markers to indicate the progress, e.g.
[X] Task 1
[ ] Task 2
Also update the progress of the task in the Scratchpad when you finish a subtask.
Especially when you finished a milestone, it will help to improve your depth of task accomplishment to use the scratchpad to reflect and plan.
The goal is to help you maintain a big picture as well as the progress of the task. Always refer to the Scratchpad when you plan the next step.

# Tools

Note all the tools are in python. So in the case you need to do batch processing, you can always consult the python files and write your own script.

## LLM

You always have an LLM at your side to help you with the task. For simple tasks, you could invoke the LLM by running the following command:
```bash
cursor-llm --prompt "What is the capital of France?" --provider "anthropic"
```

The LLM API supports multiple providers:
- OpenAI (default, model: gpt-4o)
- DeepSeek (model: deepseek-chat)
- Anthropic (model: claude-3-sonnet-20240229)
- Gemini (model: gemini-pro)
- Local LLM (model: Qwen/Qwen2.5-32B-Instruct-AWQ)

But usually it's a better idea to check the content of the file and use the APIs in the `cursor_agent/tools/llm_api.py` file to invoke the LLM if needed.

## Web browser

You could use the web scraper to fetch web pages:
```bash
cursor-scrape --max-concurrent 3 URL1 URL2 URL3
```
This will output the content of the web pages.

## Search engine

You could use the search engine to find information:
```bash
cursor-search "your search keywords"
cursor-search --max-results 5 "specific search"
```
This will output the search results in the following format:
```
URL: https://example.com
Title: This is the title of the search result
Snippet: This is a snippet of the search result
```
If needed, you can further use the `web_scraper.py` file to scrape the web page content.

# Lessons

## User Specified Lessons

- You have a python venv in ./venv.
- Include info useful for debugging in the program output.
- Read the file before you try to edit it.
- Use LLM to perform flexible text understanding tasks. First test on a few files. After success, make it parallel.

## Cursor learned

- Flutter web: browsers forbid setting the `Cookie` request header from JS for `fetch`/XHR (including `CachedNetworkImage` / `HttpFileService`).跨域时不要用全局 `withCredentials: true`（会触发 CORS 失败 → Dio `XMLHttpRequest onError`）；仅在 `Uri.base` 与 `dio.options.baseUrl` 同源时对 Dio 使用 `BrowserHttpClientAdapter(withCredentials: true)`。非代理图仍走 `CachedNetworkImage`；`flutter_cache_manager` 的 Web `HttpFileService` 用 `BrowserClient(withCredentials: false)` 以免拖垮 CORS。
- MoviePilot v2：`/api/v1/system/img`、`/api/v1/system/cache/image` 使用 `verify_resource_token`，只认 Cookie名 `MoviePilot` 里的 **resource** JWT（`purpose=resource`），与登录 `access_token`（authentication）不同；Web 上对代理图用 `ApiClient.fetchResourceProxyImage`（Dio `extra.withCredentials: true` + `skipUnauthorizedHandling` 避免 403 误登出）。跨站时若服务端 Set-Cookie 为 `SameSite=Lax`，浏览器可能仍不带该 Cookie，需同源反代或服务端改为 `SameSite=None; Secure`。
- iOS: if `UIBackgroundModes` includes `processing`, App Store requires `BGTaskSchedulerPermittedIdentifiers` listing every BGTask identifier the app registers. If the app does not use `BGTaskScheduler`, remove `processing` instead of adding placeholder identifiers.
- For website image paths, always use the correct relative path (e.g., 'images/filename.png') and ensure the images directory exists
- For search results, ensure proper handling of different character encodings (UTF-8) for international queries
- Add debug information to stderr while keeping the main output clean in stdout for better pipeline integration
- When using seaborn styles in matplotlib, use 'seaborn-v0_8' instead of 'seaborn' as the style name due to recent seaborn version changes
- Available command-line tools after installation:
  - `cursor-agent`: Initialize directory with agent capabilities
  - `cursor-llm`: LLM interaction (OpenAI, DeepSeek, Anthropic, Gemini, Local)
  - `cursor-scrape`: Web scraping with JavaScript support
  - `cursor-search`: Search engine integration
  - `cursor-update`: Update to latest version
  - `cursor-verify`: Verify setup

## Testing Lessons

- Use pytest fixtures in conftest.py for common test data and setup
- Mock environment variables with autouse fixture to ensure consistent test environment
- Test both success and failure cases for each command
- Verify exact function call arguments using assert_called_once_with
- Use temporary directories (tmp_path) for file operation tests
- Keep mock data realistic and consistent with actual API responses
- Test CLI argument parsing with sys.argv mocking
- Add proper cleanup in teardown to restore original state
- Test file operations should use Path objects for cross-platform compatibility
- Mock external API calls and file operations to make tests reliable and fast

# Scratchpad

## Done: 已安装插件页（PluginPage）与插件市场同风格 bottom floating bar + visibleItems 缓存 + palette 预加载限制 + await loadFromCache

## Current Task: Dashboard 页面编辑功能（隐藏/显示 + 换位）

[ ] 理清 DashboardConfig / DashboardOrder 与 displayedWidgets 的映射关系
[ ] 设计并实现 Dashboard 编辑 UI（从 AppSettingPage 进入）
[ ] 将编辑结果写回 /api/v1/user/config/Dashboard 与 /api/v1/user/config/DashboardOrder
[ ] 确保隐藏状态下不会触发对应组件的定时请求（复用 _loadDataBasedOnConfig 逻辑）
[ ] 本地持久化与远端配置保持兼容

Previous: Fix App Store ITMS-90771（已完成）

---
> Source: [singleton-altman/MoviePilotLite](https://github.com/singleton-altman/MoviePilotLite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-20 -->
