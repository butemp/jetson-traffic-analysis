# Jetson Traffic Analysis

完整项目以迁移压缩包发布：[下载完整源码与部署包](jetson-traffic-analysis-source-20261116.zip)。解压后包含下文列出的全部源码和目录，便于离线搬运。

面向 Jetson 的交通视频分析与结构化辅助研判系统。此仓库整理自 **2026-11-16 正在运行的部署版本**，包含完整应用代码、静态前端、启动脚本和离线迁移工具。

> **这是一份源码与部署配置包，不是完整离线运行环境。** 另一台无网络 Jetson 还需要单独导入参考 Docker 镜像。模型权重和 GPU 依赖已在该镜像内，未上传 GitHub。

## 功能

- 网络视频源接入与离线录像上传、排队分析。
- 六类交通候选事件：拥堵、交通事故、逆行、非机动车、烟火、明火。
- 刀具与疑似打斗巡检，可在逐帧巡检中分别选择检测场景。
- 结构化研判：带时间戳的证据帧、短窗口时序回溯、画面内目标轨迹、风险提示和不确定性。
- 事件复核、告警处理、样本回流、报表、应急方案及人工审核流程。
- 上传前检查同名文件、复用服务器已有录像、传输进度与取消。
- 前端外观设置、标签排序，以及面向演示的功能名称展示。

模型结果属于候选线索。轨迹是画面内相对位置，不能直接换算真实车速或用于自动定责。精选样例不代表所有路段的识别准确率。

## 迁移范围与依赖

| 项目 | 说明 |
|---|---|
| 源设备 | Jetson AGX Orin Developer Kit，ARM64 |
| 源系统 | L4T R36.4.7 |
| 参考镜像 | `qjw/traffic-vlm-jetson:20261023` |
| 镜像内容 ID | 见 `environment/source-system.json` |
| 镜像大小 | 约 25.7 GB（未压缩） |
| 模型与加速库 | 在参考镜像内；不是单独 `pip install` 即可重建的普通 Python 环境 |
| 业务数据库、录像、账号、密钥 | 不包含，目标设备默认创建新的运行数据 |

目标设备须预先具备兼容的 JetPack/L4T、Docker Compose 和 NVIDIA Container Runtime。仓库不会联网安装系统驱动，也不会下载模型。其他 Jetson 型号、较小内存或不同系统版本需要重新验证，不能据此承诺直接兼容。

## 离线迁移步骤

### 1. 将源码带到目标设备

联网电脑可克隆仓库或下载 ZIP，再通过 U 盘、局域网复制到新 Jetson：

```bash
git clone https://github.com/butemp/jetson-traffic-analysis.git migration-download
cd migration-download
python3 -m zipfile -e jetson-traffic-analysis-source-20261116.zip .
cd jetson-traffic-analysis
```

### 2. 从旧 Jetson 导出运行镜像

在**旧 Jetson** 上执行，输出目录应位于至少有 30 GB 可用空间的外置磁盘：

```bash
sudo bash scripts/export-runtime.sh /media/USER/DISK/traffic-runtime.tar.gz
```

同时携带生成的 `.sha256` 文件。此操作不导出旧机器的摄像头密码、业务数据库和录像。

### 3. 在新 Jetson 导入镜像

```bash
sudo bash scripts/import-runtime.sh /media/USER/DISK/traffic-runtime.tar.gz
```

Docker 存储盘还需足够空间解包镜像。镜像不在本仓库，也不保证可从公开镜像仓库拉取。

### 4. 配置并启动

```bash
cp .env.example .env
sudo bash scripts/start.sh
docker compose ps
docker compose logs -f
```

首次启动需要初始化推理服务和检查加速引擎。全部服务就绪后，访问 `http://目标设备IP:8002/`。默认保持原部署的可信局域网免登录模式（`AUTH_DISABLED=1`）。本仓库不包含原有密码；启用登录时，启动脚本在目标设备生成新密码文件。

`scripts/preflight.py` 会检查 ARM64、L4T、容器运行时和镜像 ID。版本不一致时不要将“能启动”当成完整验收。加速引擎在启动时检查 GPU、版本、模型校验值，不匹配时使用目标设备重新构建。

## 配置

编辑 `.env`，不要提交真实配置：

- `TRAFFIC_DATA_DIR`：本地持久化数据目录，默认 `./data`。SQLite 数据建议放目标设备本地磁盘。
- `WEB_PORT`、`VLLM_PORT`：网页与本机推理服务端口。
- `AUTH_DISABLED`：保持演示免登录或启用登录。
- `MAX_CONCURRENT_ANALYSIS`、`MAX_STREAMS`：任务与巡检并发上限，须按目标设备重新压测。
- `VLLM_GPU_UTIL`、`VLLM_KV_BYTES`、`VLLM_MAX_MODEL_LEN`：推理服务资源参数。
- 其他默认参数见 `deploy/runtime/config/app.env.defaults`、`src/config.py`。

摄像头地址与凭据在目标系统的数据源管理中重新配置。请勿将 `.env`、`data/`、模型权重、真实录像或密钥加入 Git。

## 压缩包内项目结构

```text
src/                         当前完整应用源码
  app.py                     Web/API 入口
  executor.py, worker.py      离线分析、任务队列与取消
  video.py, stream_api.py     视频输入、实时巡检会话
  streaming.py               时序检测与事件聚合
  stream_models.py            检测模型适配
  gpu_scheduler.py            共享模型与 GPU 调度
  vlm.py, local_vlm.py        本机推理服务调用与输出校验
  traffic_reasoning.py        证据回溯、目标关联、结构化研判
  db.py, alerts.py, review.py 数据、告警与复核
  emergency_plan.py           应急方案
  source_public.py            对外字段处理与展示名称
  static/                    网页代码、样式、离线前端依赖
  scripts/                   模型诊断、训练与导出参考脚本
  models/                    模型来源与校验元信息，不含权重
deploy/runtime/              容器启动脚本与引擎适配
scripts/                     离线导入/导出、预检与启动
environment/                 源系统、依赖版本和源码校验参考
licenses/                    第三方许可文本
docs/                        运行说明与迁移整理记录
```

运行方式采用“源码覆盖层”：将本仓库应用文件合并到参考镜像的项目目录，保留镜像自带模型及 `.vendor` 依赖。不要直接用不含模型的 `src/` 整目录覆盖挂载镜像内部项目目录，否则会遮住镜像中的模型文件。

## 二次开发与重新封装

修改 `src/` 后重启相应服务即可应用。也可以基于已导入的参考镜像构建新镜像：

```bash
sudo docker build --pull=false -t my-traffic-project:local .
```

`src/requirements-jetson.txt` 与 `environment/installed-packages.reference.txt` 用于核对版本，不是替换 Jetson GPU 软件栈的安装指令。

## 当前边界

- 六类交通采用采样帧，详细研判在后台生成；不宣称完整链路毫秒级处理。
- 当前轨迹为单视频短窗口关联，不是跨镜头身份追踪。
- 刀具与打斗组合模式仍可能出现误报，打斗专项演示应选择“仅疑似打斗”。
- 宿主机自动校时服务不在 Docker 应用内。目标设备须单独校准时间；不要复制旧机器的校时共享密钥。
- `data/` 默认全新，不包含原机事件、已生成报告或摄像头配置。
- 原机工作流已运行验证，但本迁移包尚未在另一台实体 Jetson 上完成整机验收。

## 文档与许可

- [运行与排障说明](OPERATIONS.md)
- [迁移整理记录](MIGRATION_CHANGES.md)
- [第三方组件说明](THIRD_PARTY_NOTICES.md)

本仓库未为原应用源码额外指定新的开源许可证。公开可见不等于新增授权；第三方组件和模型继续遵循各自原有许可。模型权重与演示视频未随仓库分发。
