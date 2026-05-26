# Test Automation Framework

一个基于 `pytest + requests + YAML + Allure` 的接口自动化测试项目，适合学习接口自动化框架的基本设计，也可以作为小型项目模板继续扩展。

项目当前已经包含：

- 单接口测试
- 业务场景测试
- YAML 数据驱动
- 接口关联提取
- 自定义动态参数
- Allure 测试报告
- 本地 Mock 服务
- 日志记录
- 数据库断言能力

## 技术栈

- Python 3.12+
- pytest
- requests
- PyYAML
- jsonpath
- allure-pytest
- Flask（用于本地 mock 服务）

## 项目结构

```text
Test-Automation-Framework-main/
├─ base/                   # 核心执行逻辑，请求编排、业务场景编排
├─ common/                 # 通用工具：发请求、断言、日志、读写 YAML、数据库连接
├─ conf/                   # 配置文件
├─ data/                   # 测试数据、SQL、CSV、Excel
├─ mock_server/            # 本地 mock 接口服务
├─ report/                 # 测试报告输出目录
├─ testcase/               # 测试用例
│  ├─ Single interface/    # 单接口测试
│  ├─ ProductManager/      # 商品管理相关接口测试
│  └─ Business interface/  # 业务场景测试
├─ conftest.py             # 全局前后置处理
├─ extract.yaml            # 接口关联提取结果
├─ pytest.ini              # pytest 配置
├─ run.py                  # 测试执行入口
└─ pyproject.toml          # 项目依赖配置
```

## 框架执行流程

这个项目的自动化测试流程可以概括为：

```text
启动 pytest
-> 读取测试文件
-> 读取 YAML 用例数据
-> 替换动态参数
-> 发送接口请求
-> 校验响应结果
-> 提取关联数据到 extract.yaml
-> 生成 Allure 报告
```

对应的核心文件如下：

- `run.py`：项目入口，调用 `pytest` 并生成报告
- `testcase/.../*.py`：测试入口文件
- `common/readyaml.py`：读取测试用例 YAML
- `base/apiutil.py`：统一处理请求、提取、断言
- `common/sendrequest.py`：底层发送 HTTP 请求
- `common/assertions.py`：统一断言逻辑
- `common/debugtalk.py`：动态参数函数
- `testcase/conftest.py`：登录前置、日志前后置

## 环境准备

### 1. 安装 Python

建议使用：

- Python 3.12 运行主项目
- Python 3.13 运行 `mock_server/api_server` 时也可以参考其独立配置

如果你只是在本仓库里学习和跑主测试流程，优先保证主项目的 Python 环境可用即可。

### 2. 安装依赖

使用 `uv`：

```bash
uv sync
```

或者使用 `pip`：

```bash
pip install -e .
```

如果你要单独启动 mock 服务，也可以进入对应目录安装依赖：

```bash
cd mock_server/api_server
pip install -e .
```

### 3. 安装 Allure 命令行工具

项目中 Python 依赖只包含 `allure-pytest`，如果你希望本地打开 Allure 页面，还需要额外安装 Allure CLI。

安装完成后，确保下面命令可用：

```bash
allure --version
```

## 配置说明

主配置文件在 `conf/config.ini`。

重点配置项：

```ini
[api_envi]
host = http://127.0.0.1:8787
```

这里的 `host` 默认指向本地 mock 服务，所以在没有真实后端环境时，也能先跑通整个流程。

报告类型配置：

```ini
[REPORT_TYPE]
type = allure
```

支持：

- `allure`
- `tm`

另外，`conf/setting.py` 中也定义了超时时间、报告目录、日志目录等基础配置。

## 先启动 Mock 服务

默认测试地址是 `http://127.0.0.1:8787`，所以执行测试前，建议先启动本地 mock 服务。

在 `mock_server/api_server` 目录下运行：

```bash
python base/flask_service.py
```

启动成功后，接口会监听：

```text
http://127.0.0.1:8787
```

项目里已经内置了一批模拟接口，例如：

- `/dar/user/login`
- `/dar/user/addUser`
- `/dar/user/updateUser`
- `/dar/user/deleteUser`
- `/dar/user/queryUser`
- `/coupApply/cms/goodsList`
- `/coupApply/cms/productDetail`
- `/coupApply/cms/placeAnOrder`

## 如何运行测试

### 方式一：运行入口脚本

```bash
python run.py
```

当 `REPORT_TYPE = allure` 时，脚本会：

1. 执行 `pytest`
2. 把结果输出到 `report/temp`
3. 复制 `environment.xml`
4. 调用 `allure serve ./report/temp`

### 方式二：直接运行 pytest

```bash
pytest -s -v ./testcase
```

如果你想手动生成 Allure 结果：

```bash
pytest -s -v --alluredir=./report/temp ./testcase --clean-alluredir
```

再手动打开报告：

```bash
allure serve ./report/temp
```

## 用例是怎么组织的

项目采用 Python 测试文件 + YAML 数据文件分离的方式。

以单接口测试为例：

- 测试入口：`testcase/Single interface/test_debug_api.py`
- 测试数据：`testcase/Single interface/addUser.yaml`

测试函数通过 `pytest.mark.parametrize` 读取 YAML 中的多条数据，把每个场景拆成一个独立用例执行。

## YAML 用例格式

一个典型的 YAML 用例分为两部分：

- `baseInfo`：接口公共信息
- `testCase`：具体测试场景

示例：

```yaml
- baseInfo:
    api_name: 新增用户
    url: /dar/user/addUser
    method: POST
    header:
      Content-Type: application/x-www-form-urlencoded;charset=UTF-8
  testCase:
    - case_name: 正常新增用户
      data:
        username: testadduser
        password: test123456
        role_id: 123456789
        dates: "2023-12-31"
        phone: 13800000000
        token: ${get_extract_data(token)}
      validation:
        - contains: { "status_code": 200 }
        - contains: { "msg": "新增成功" }
```

## 接口关联是怎么实现的

项目通过 `extract.yaml` 实现接口之间的数据传递。

例如：

1. 登录接口执行后，从响应中提取 `token`
2. 把 `token` 写入 `extract.yaml`
3. 其它接口通过 `${get_extract_data(token)}` 读取并复用这个值

登录前置在 `testcase/conftest.py` 中定义，整个测试 session 开始时会先自动登录。

## 动态参数能力

`common/debugtalk.py` 中封装了一些动态函数，可以在 YAML 中直接调用，例如：

```yaml
token: ${get_extract_data(token)}
timeStamp: ${timestamp()}
orderNumber: ${get_extract_data(orderNumber)}
```

常见用途：

- 读取上一个接口提取的数据
- 生成时间戳
- 读取 CSV 数据
- 做简单加密处理

## 断言能力

项目当前支持的断言类型主要有：

- `contains`：包含断言
- `eq`：相等断言
- `ne`：不相等断言
- `rv`：任意字段值断言
- `db`：数据库断言

示例：

```yaml
validation:
  - contains: { "status_code": 200 }
  - eq: { "msg": "登录成功" }
```

## 业务场景测试

除了单接口测试，项目还支持业务链路测试。

例如 `testcase/Business interface/BusinessScenario.yml` 中演示了：

1. 获取商品列表
2. 获取商品详情
3. 提交订单
4. 订单支付
5. 校验订单状态

这个流程通过提取 `goodsIds`、`orderNumber`、`userId` 等数据，把多个接口串联起来。

## 日志与报告

### 日志

日志输出目录：

```text
logs/
```

项目会自动按日期生成日志文件，并清理过期日志。

### 报告

Allure 原始结果目录：

```text
report/temp/
```

Allure HTML 报告目录：

```text
report/allureReport/
```

TMReport 目录：

```text
report/tmreport/
```

## 初学者推荐阅读顺序

如果你刚接触这个项目，建议按下面顺序阅读源码：

1. `testcase/Single interface/test_debug_api.py`
2. `testcase/Single interface/addUser.yaml`
3. `common/readyaml.py`
4. `base/apiutil.py`
5. `common/sendrequest.py`
6. `common/assertions.py`
7. `testcase/conftest.py`
8. `common/debugtalk.py`

这样最容易看懂一个测试用例是如何从 YAML 一步步执行到最终断言完成的。

## 常见问题

### 1. 为什么运行时报连接失败？

先检查 `conf/config.ini` 中的 `host` 是否可访问。  
如果你使用默认配置，需要先启动本地 mock 服务。

### 2. 为什么没有打开 Allure 报告？

通常是因为本机没有安装 Allure CLI，或者 `allure` 命令没有加入环境变量。

### 3. 为什么 token 为空？

先确认登录接口是否执行成功，再检查 `extract.yaml` 中是否成功写入了 `token`。

### 4. 为什么报告里有旧数据？

项目启动时会自动清理 `report/temp` 和 `extract.yaml`，但如果你是手动执行部分命令，也可以先手动确认目录是否已清空。

## 后续可以扩展的方向

- 接口重试机制
- 更完善的数据库校验
- 多环境切换
- 更严格的 schema 校验
- CI/CD 集成
- 钉钉或企业微信通知
- 自定义测试标签和分层运行

## License

本项目遵循仓库中的 `LICENSE` 文件。
