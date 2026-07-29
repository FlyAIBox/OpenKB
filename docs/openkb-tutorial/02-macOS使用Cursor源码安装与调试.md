# macOS 使用 Cursor 从源码安装与调试 OpenKB

> 项目地址：[VectifyAI/OpenKB](https://github.com/VectifyAI/OpenKB)  
> 适用场景：修改 OpenKB 的 Python 后端、CLI 或 Knowledge Workbench 前端。  
> 本文以 macOS、Cursor、Python 3.11 和 OpenKB `main` 分支为例。

## 1. 源码安装与普通安装的区别

普通安装：

```bash
python -m pip install openkb
```

安装的是 PyPI 发布包，适合直接使用。源码可编辑安装：

```bash
git clone https://github.com/VectifyAI/OpenKB.git
cd OpenKB
python -m pip install -e .
```

其中 `-e` 表示 editable。安装后，Python 仍从当前仓库读取 `openkb/`
源码，因此修改 `.py` 文件后通常不需要重新执行 `pip install`。

开发时还需要测试、代码检查和 Web API 依赖，建议一次安装完整：

```bash
python -m pip install -e ".[dev,web]"
```

这比只执行 `pip install -e .` 多安装：

- `pytest`、`ruff`、`mypy` 等开发工具；
- FastAPI、Uvicorn 和文件上传组件；
- `openkb-web` 和 `openkb-api` 命令所需依赖。



## 2. 准备 macOS 开发环境



### 2.1 安装基础工具

先确认 Git、Python、Node.js 和 npm：

```bash
git --version
python3 --version
node --version
npm --version
```

推荐使用 Python 3.11 或 3.12。若尚未安装，可以通过 Homebrew 安装：

```bash
brew install git python@3.11 node
```

安装 Cursor 后，在命令面板中执行：

```text
Shell Command: Install 'cursor' command in PATH
```

之后可以从终端打开项目：

```bash
cursor .
```



### 2.2 克隆仓库

```bash
mkdir -p ~/code
cd ~/code
git clone https://github.com/VectifyAI/OpenKB.git
cd OpenKB
cursor .
```

如果已经克隆，不要重复克隆，直接进入现有目录：

```bash
cd ~/code/OpenKB
git status
cursor .
```



## 3. 创建独立 Python 环境

不要在 Conda `base` 或系统 Python 中直接开发。下面两种方式任选一种。

### 3.1 使用 venv

在仓库根目录执行：

```bash
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip setuptools wheel
python -m pip install -e ".[dev,web]"
```



### 3.2 使用 Conda

```bash
conda create -n openkb-dev python=3.11 -y
conda activate openkb-dev
cd ~/code/OpenKB
python -m pip install --upgrade pip setuptools wheel
python -m pip install -e ".[dev,web]"
```

验证当前命令确实来自开发环境：

```bash
python --version
python -m pip --version
which python
which openkb
which openkb-web
python -m pip show openkb
```

`pip show openkb` 中的 `Editable project location` 应指向当前 OpenKB
仓库。还可以直接确认 Python 实际加载的源码位置：

```bash
python -c "import openkb; print(openkb.__file__)"
```

输出应类似：

```text
/Users/你的用户名/code/OpenKB/openkb/__init__.py
```



## 4. 在 Cursor 中选择解释器

1. 按 `Command + Shift + P` 打开命令面板；
2. 运行 `Python: Select Interpreter`；
3. 选择仓库中的 `.venv/bin/python`，或选择 `openkb-dev` Conda 环境；
4. 新建 Cursor 终端，运行 `which python` 再次确认。

如果列表中没有目标解释器，选择 `Enter interpreter path`，然后指定：

```text
项目绝对路径/.venv/bin/python
```

调试器、测试发现和终端必须尽量使用同一个环境。否则常见现象是终端能
导入 OpenKB，但按 F5 后出现 `ModuleNotFoundError`。

## 5. 配置模型凭据和开发知识库



### 5.1 创建本地环境变量

复制示例文件：

```bash
cp .env.example .env
```

编辑仓库根目录的 `.env`：

```dotenv
LLM_API_KEY=your_llm_api_key

# 使用 OpenAI 兼容网关时取消注释：
# OPENAI_API_BASE=https://your-api-endpoint.example/v1

# REST API 创建和查找开发知识库的位置：
OPENKB_KB_ROOT=/Users/你的用户名/code/OpenKB/kbs
```

仓库已经忽略 `.env` 和 `kbs/`，但提交前仍应运行 `git status`，确保真实
API Key 没有进入版本控制。

### 5.2 创建开发知识库

```bash
mkdir -p kbs/dev-kb
cd kbs/dev-kb
openkb init
```

初始化后可以编辑：

```text
kbs/dev-kb/.openkb/config.yaml
```

例如：

```yaml
model: gpt-5.4
language: zh
pageindex_threshold: 20
```

回到仓库根目录：

```bash
cd ../..
```



## 6. 运行和调试 CLI



### 6.1 在终端运行

CLI 根据当前工作目录定位知识库，因此先进入开发知识库：

```bash
cd kbs/dev-kb
openkb status
openkb add /path/to/document.pdf
openkb query "请概括文档的核心内容"
openkb chat
```

源码可编辑安装后，修改 `openkb/` 中的 Python 文件，停止并重新运行命令
即可加载新代码，无需再次安装。

### 6.2 用 Cursor 断点调试

在 Cursor 左侧打开 **Run and Debug**，创建 `.vscode/launch.json`，加入：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "OpenKB CLI：status",
      "type": "debugpy",
      "request": "launch",
      "module": "openkb",
      "args": ["status"],
      "cwd": "${workspaceFolder}/kbs/dev-kb",
      "envFile": "${workspaceFolder}/.env",
      "console": "integratedTerminal",
      "justMyCode": false
    },
    {
      "name": "OpenKB CLI：query",
      "type": "debugpy",
      "request": "launch",
      "module": "openkb",
      "args": ["query", "请概括知识库中的核心内容"],
      "cwd": "${workspaceFolder}/kbs/dev-kb",
      "envFile": "${workspaceFolder}/.env",
      "console": "integratedTerminal",
      "justMyCode": false
    }
  ]
}
```

使用方法：

1. 在 `openkb/cli.py` 或调用链上的模块左侧单击设置断点；
2. 在调试配置中选择对应命令；
3. 按 F5；
4. 使用 Variables、Call Stack 和 Debug Console 检查状态。

要调试其他子命令，只需复制配置并修改 `args`。例如调试文件导入：

```json
"args": ["add", "/absolute/path/to/document.pdf"]
```

传给 `args` 的本地文件最好使用绝对路径，避免被 `cwd` 影响。

## 7. 运行和调试 REST API



### 7.1 终端启动

在仓库根目录执行：

```bash
source .venv/bin/activate
openkb-web --host 127.0.0.1 --port 7566 --reload
```

检查服务：

```text
http://127.0.0.1:7566/docs
```

也可以使用等价入口：

```bash
python -m openkb.api --host 127.0.0.1 --port 7566 --reload
```

`--reload` 适合日常开发：Python 文件改变后 Uvicorn 会重启服务。

### 7.2 Cursor 断点配置

在 `launch.json` 的 `configurations` 数组中增加：

```json
{
  "name": "OpenKB REST API",
  "type": "debugpy",
  "request": "launch",
  "module": "openkb.api",
  "args": ["--host", "127.0.0.1", "--port", "7566"],
  "cwd": "${workspaceFolder}",
  "envFile": "${workspaceFolder}/.env",
  "console": "integratedTerminal",
  "justMyCode": false
}
```

F5 启动后，可以在 `openkb/api.py` 的路由或业务模块中设置断点，再从
Swagger、Workbench 或 `curl` 发起请求。

断点调试配置没有添加 `--reload`，因为自动重载会创建子进程，使单步调试
和进程生命周期更难判断。修改代码后停止调试并重新按 F5 即可。

一个不调用 LLM 的基础检查：

```bash
curl http://127.0.0.1:7566/api/v1/kbs
```

如果设置了 `OPENKB_API_TOKEN`，请求必须增加：

```bash
curl \
  -H "Authorization: Bearer $OPENKB_API_TOKEN" \
  http://127.0.0.1:7566/api/v1/kbs
```



## 8. 运行和调试 Knowledge Workbench 前端

源码仓库中的 `openkb/web/` 是构建产物，默认不提交 Git。仅安装 Python
源码并不会自动生成这部分静态文件。

### 8.1 前后端热更新开发

打开两个 Cursor 终端。

终端 1：启动 Python API：

```bash
cd ~/code/OpenKB
# 使用 venv才需要这个命令，Conda方式不需要
source .venv/bin/activate
openkb-web --host 127.0.0.1 --port 7566 --reload
```

终端 2：启动 Vite：

```bash
cd ~/code/OpenKB/frontend
npm install
npm run dev
```

浏览器打开：

```text
http://127.0.0.1:5173/
```

Vite 会把 `/api` 请求代理到 `http://127.0.0.1:7566`。此模式下：

- 修改 React/TypeScript/CSS 后，页面会热更新；
- 修改 Python 后端后，Uvicorn 会自动重启；
- API 仍可在 `http://127.0.0.1:7566/docs` 单独检查。

前端断点可使用 Cursor/浏览器的 JavaScript 调试器。也可以先在浏览器开发者
工具中检查 Console 和 Network；SSE 查询或聊天问题重点查看 Network 中对应
请求的响应事件。

### 8.2 构建由 FastAPI 直接提供的前端

要验证发布形态，执行：

```bash
cd frontend
npm install
npm run build
cd ..
openkb-web --host 127.0.0.1 --port 7566
```

构建结果写入：

```text
openkb/web/
```

然后打开：

```text
http://127.0.0.1:7566/
```

此时前端和 API 使用同一个端口。若改了前端源码，需要重新运行
`npm run build`；若只改 Python 源码，不需要重建前端。

## 9. 运行测试和质量检查

在仓库根目录执行：

```bash
pytest
ruff check .
ruff format --check .
mypy openkb
```

修改前端后执行：

```bash
cd frontend
npm run i18n:check
npm run lint
npm run build
```

调试单个 Python 测试：

```bash
pytest tests/test_cli.py -q
pytest tests/test_cli.py::test_name -q
```

若要在失败处进入 Python 调试器：

```bash
pytest tests/test_cli.py -x --pdb
```

Cursor 也可以在测试函数左侧使用 **Debug Test**，前提是已经选择正确的
Python 解释器并成功发现测试。

## 10. 推荐的日常开发流程

```bash
# 1. 进入仓库并激活环境
cd ~/code/OpenKB
source .venv/bin/activate

# 2. 同步主分支
git switch main
git pull --ff-only

# 3. 创建开发分支
git switch -c fix/short-description

# 4. 修改代码并运行针对性测试
pytest tests/test_target.py -q
ruff check .

# 5. 查看实际变更
git status
git diff
```

不要把 `.env`、真实文档、生成的知识库或 `openkb/web/` 构建产物提交到
仓库。提交前还应运行与改动范围匹配的完整测试。

## 11. 常见问题

