# Supervisord REST API 接口文档

本文档描述了 Go-Supervisord 提供的 REST API 接口，这些接口用于管理和监控 supervisord 进程。

## 概述

Supervisord REST API 提供了程序管理和 supervisor 系统控制两大类接口：

- 程序管理接口：用于启动、停止、查看程序状态和日志
- 系统控制接口：用于重启、关闭 supervisord 系统

## 接口基础信息

- **Base URL**: `http://<supervisor-host>:<port>`
- **Content-Type**: `application/json` (除特殊说明外)
- **响应格式**: JSON

## 程序管理接口

### 1. 获取所有程序状态

**接口**: `GET /program/list`

**描述**: 获取所有被管理程序的状态信息

**请求示例**:
```bash
GET /program/list
```

**响应示例**:
```json
[
  {
    "name": "myprogram",
    "group": "mygroup",
    "description": "My program description",
    "start": 1640995200,
    "stop": 1640995300,
    "now": 1640995400,
    "state": 20,
    "statename": "RUNNING",
    "spawnerr": "",
    "exitstatus": 0,
    "logfile": "/path/to/logfile",
    "stdout_logfile": "/path/to/stdout",
    "stderr_logfile": "/path/to/stderr",
    "pid": 12345
  }
]
```

**字段说明**:
- `name`: 程序名称
- `group`: 程序组名称
- `statename`: 程序状态 (STARTING, RUNNING, STOPPED, FATAL等)
- `pid`: 进程ID
- 其他字段详见 `ProcessInfo` 结构体

### 2. 启动单个程序

**接口**: `POST /program/start/{name}` 或 `PUT /program/start/{name}`

**描述**: 启动指定名称的程序

**路径参数**:
- `name`: 程序名称

**请求示例**:
```bash
POST /program/start/myprogram
```

**响应示例**:
```json
{
  "success": true
}
```

### 3. 停止单个程序

**接口**: `POST /program/stop/{name}` 或 `PUT /program/stop/{name}`

**描述**: 停止指定名称的程序

**路径参数**:
- `name`: 程序名称

**请求示例**:
```bash
POST /program/stop/myprogram
```

**响应示例**:
```json
{
  "success": true
}
```

### 4. 批量启动程序

**接口**: `POST /program/startPrograms` 或 `PUT /program/startPrograms`

**描述**: 批量启动多个程序

**请求体**: 程序名称数组

**请求示例**:
```bash
POST /program/startPrograms
Content-Type: application/json

["program1", "program2", "program3"]
```

**响应示例**:
```text
Success to start the programs
```

### 5. 批量停止程序

**接口**: `POST /program/stopPrograms` 或 `PUT /program/stopPrograms`

**描述**: 批量停止多个程序

**请求体**: 程序名称数组

**请求示例**:
```bash
POST /program/stopPrograms
Content-Type: application/json

["program1", "program2", "program3"]
```

**响应示例**:
```text
Success to stop the programs
```

### 6. 读取程序标准输出日志

**接口**: `GET /program/log/{name}/stdout`

**描述**: 读取指定程序的标准输出日志

**路径参数**:
- `name`: 程序名称

**注意**: 此接口在当前实现中为空函数，可能需要进一步实现

## 系统控制接口

### 1. 关闭 supervisord

**接口**: `POST /supervisor/shutdown` 或 `PUT /supervisor/shutdown`

**描述**: 关闭整个 supervisord 系统

**请求示例**:
```bash
POST /supervisor/shutdown
```

**响应示例**:
```text
Shutdown...
```

### 2. 重载配置

**接口**: `POST /supervisor/reload` 或 `PUT /supervisor/reload`

**描述**: 重新加载 supervisord 配置文件

**请求示例**:
```bash
POST /supervisor/reload
```

**响应示例**:
```json
{
  "success": false
}
```

## 错误处理

### 错误响应格式

- 400 Bad Request: 请求格式错误
  ```text
  not a valid request
  ```

- 其他错误: 通过 `success: false` 表示操作失败

### 常见错误原因

1. 程序不存在或未配置
2. supervisord 未运行
3. 权限不足
4. 配置文件错误

## 使用示例

### JavaScript 客户端示例

```javascript
// 获取程序列表
function listPrograms() {
  fetch('/program/list')
    .then(response => response.json())
    .then(data => {
      console.log('Programs:', data);
    });
}

// 启动程序
function startProgram(programName) {
  fetch(`/program/start/${programName}`, {
    method: 'POST'
  })
  .then(response => response.json())
  .then(data => {
    console.log('Start result:', data.success);
  });
}

// 批量停止程序
function stopMultiplePrograms(programNames) {
  fetch('/program/stopPrograms', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(programNames)
  })
  .then(response => response.text())
  .then(data => {
    console.log('Stop result:', data);
  });
}
```

### cURL 示例

```bash
# 获取程序列表
curl -X GET http://localhost:9001/program/list

# 启动程序
curl -X POST http://localhost:9001/program/start/myprogram

# 停止程序
curl -X POST http://localhost:9001/program/stop/myprogram

# 批量启动程序
curl -X POST http://localhost:9001/program/startPrograms \
  -H "Content-Type: application/json" \
  -d '["program1", "program2"]'

# 重载配置
curl -X POST http://localhost:9001/supervisor/reload

# 关闭 supervisord
curl -X POST http://localhost:9001/supervisor/shutdown
```

## 注意事项

1. **权限控制**: API接口的访问控制需要通过 supervisord 的 `[inet_http_server]` 配置实现
2. **并发操作**: 多个API调用可能导致状态不一致，建议串行执行关键操作
3. **超时设置**: 程序启动/停止操作可能需要较长时间，建议客户端设置适当的超时时间
4. **监控**: 建议定期调用 `/program/list` 接口来监控程序状态

## 配置启用

要使用REST API，需要在 supervisord.conf 中启用HTTP服务器：

```ini
[inet_http_server]
port=127.0.0.1:9001
username=admin
password=password

[supervisorctl]
serverurl=http://127.0.0.1:9001
```

完成配置后重启 supervisord 即可使用REST API接口。