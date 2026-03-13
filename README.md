# API 自动化测试项目 (Python + Requests + Pytest)

## 1. 项目简介
本项目是一个基于 Python `requests` 库和 `pytest` 测试框架搭建的轻量级、高可维护的 API 自动化测试框架。
采用 **API Object 分层设计模式**，将接口定义、测试数据、测试逻辑完全解耦，旨在提高脚本的复用率和可读性。

## 2. 技术栈
- **语言**: Python 3.8+
- **HTTP 请求**: `requests`
- **测试框架**: `pytest`
- **数据驱动**: `yaml` / `json`
- **报告生成**: `pytest-html` 或 `allure-pytest` (可选)
- **日志管理**: `logging`

## 3. 目录结构说明

```
APITest_Project/
├── config/                 # [配置中心]
│   ├── __init__.py
│   ├── settings.py         # 全局常量 (超时时间、重试次数、日志级别)
│   └── env_config.yaml     # 多环境配置 (dev/test/prod 的 BaseURL, 账号等)
│
├── data/                   # [测试数据] (纯数据，无逻辑)
│   ├── login_data.yaml     # 登录测试数据
│   └── order_data.json     # 订单测试数据
│
├── apis/                   # [接口封装层] (核心：定义接口请求，不包含断言)
│   ├── __init__.py
│   ├── base_api.py         # 二次封装 Requests (统一处理 Token, 签名, 日志, 异常)
│   ├── user_api.py         # 用户模块接口定义 (如: login, register)
│   └── product_api.py      # 商品模块接口定义
│
├── cases/                  # [测试用例层] (核心：业务逻辑 + 断言)
│   ├── __init__.py
│   ├── conftest.py         # Pytest 钩子 (Fixture:  setup/teardown, 自动登录)
│   ├── test_login.py       # 登录模块测试用例
│   └── test_product.py     # 商品模块测试用例
│
├── utils/                  # [通用工具库]
│   ├── __init__.py
│   ├── db_util.py          # 数据库操作 (MySQL/Redis)
│   ├── excel_util.py       # Excel 读写 (如果需要)
│   └── logger.py           # 日志封装
│
├── reports/                # [测试报告] (运行时生成，建议.gitignore)
│   └── 2026-03-13_report.html
│
├── logs/                   # [运行日志] (运行时生成，建议.gitignore)
│   └── run.log
│
├── requirements.txt        # 依赖清单
├── pytest.ini              # Pytest 配置文件 (指定搜索路径、报告格式等)
├── .gitignore              # Git 忽略规则
└── README.md               # 项目说明文档
```

## 4. 快速开始

### 4.1 环境准备
确保已安装 Python 3.8+。

### 4.2 安装依赖
在项目根目录下执行：
```bash
pip install -r requirements.txt
```

### **4.3 配置环境**

编辑 `config/env_config.yaml`，修改当前测试环境的 `base_url` 和 测试账号信息：

```
dev:
  base_url: "https://dev-api.example.com"
  timeout: 10
  user: "test_user"
  password: "123456"
```

### **4.4 运行测试**

- **运行所有测试**:

  ```
  pytest
  ```

- **运行特定模块测试**:

  ```
  pytest cases/test_login.py
  ```

- **运行并生成 HTML 报告**:

  ```
  pytest --html=reports/report.html --self-contained-html
  ```

- **指定环境运行 (通过 marker 或 命令行变量)**:

  ```
  pytest --env=prod
  ```

## **5. 开发规范**

### **5.1 接口封装 (**`apis/`**)**

在 `apis/user_api.py` 中定义接口，**只负责发送请求和返回响应**，不做断言。

```
from apis.base_api import BaseApi

class UserApi(BaseApi):
    def login(self, username, password):
        """用户登录接口"""
        data = {"username": username, "password": password}
        return self.post("/api/v1/login", json=data)
```

### **5.2 测试用例 (**`cases/`**)**

在 `cases/test_login.py` 中调用接口并进行**断言**。

```
def test_login_success(user_api_fixture):
    # 调用封装好的接口
    response = user_api_fixture.login("admin", "123456")
    
    # 断言
    assert response.status_code == 200
    assert response.json()['code'] == 0
    assert "token" in response.json()['data']
```

### **5.3 数据驱动**

测试数据应存放在 `data/` 目录下的 YAML 文件中，通过 `@pytest.mark.parametrize` 读取。

## **6. 常见问题 (FAQ)**

- **Q: 如何切换测试环境？**
  A: 修改 `config/env_config.yaml` 中的激活环境标识，或在运行命令中传入 `--env` 参数。
- **Q: 接口需要 Token 怎么办？**
  A: 在 `apis/base_api.py` 中统一处理 Token 的自动注入，或在 `conftest.py` 中通过 fixture 前置登录获取 Token。

## **7. 联系方式**

- 作者: QA Team
- 更新日期: 2026-03-13

```
---

### 💡 核心设计亮点解析

1.  **`apis/` 层的引入 (关键点)**：
    *   很多初学者直接把 `requests.post` 写在 `cases` 里，导致一旦接口 URL 变了，要改几十个文件。
    *   **本结构优势**：接口变动（如 URL 路径变更、参数名变更）**只需修改 `apis/` 下的一个文件**，所有引用该接口的用例自动生效。

2.  **`base_api.py` 二次封装**：
    *   在这里统一处理 `requests.Session()`，实现**Cookie/Token 自动保持**。
    *   统一添加 `Headers` (如 `Content-Type`, `Authorization`)。
    *   统一打印请求日志（请求体、响应体、耗时），方便排查问题。

3.  **`pytest.ini` 的存在**：
    *   这是让项目“整洁”的关键。你可以在里面配置 `testpaths = cases`，这样运行 `pytest` 时它只扫描 `cases` 目录，不会误报 `apis` 或 `utils` 里的代码。

4.  **`conftest.py` 的作用**：
    *   用于定义 `fixture`。例如，可以定义一个 `user_api_fixture`，它在测试开始前自动调用登录接口获取 Token，并将这个带 Token 的 API 对象传递给测试用例，用例代码里完全不用关心登录过程。

这个结构既适合单人快速开发，也完全支持多人协作的大型项目。
```