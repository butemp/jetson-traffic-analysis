# 运行与排障

## 常用操作

```bash
sudo docker compose ps
sudo docker compose logs --tail 100 web
sudo docker compose logs --tail 100 inference
sudo docker compose restart web
sudo docker compose stop
```

源码更新通过容器启动时的覆盖层生效。停止或重启前，先在页面等待正在执行的任务结束。`docker compose stop` 不删除 `data/`。

## 首次启动

1. 检查目标系统和参考镜像 ID。
2. 推理服务启动并通过健康检查。
3. Web 服务检查 GPU 与加速引擎签名，不匹配时构建本机引擎。
4. Web 初始化自己的运行目录、数据库和管理员密码文件。

默认端口是 8000（本机推理）与 8002（网页）。端口冲突时同时检查 `.env`、现有进程和其他 Compose 项目。

## 离线依赖缺失

若提示参考镜像不存在，请先导入旧 Jetson 导出的镜像。不要在离线设备反复执行拉取镜像或联网安装依赖。参考镜像已包含模型、经过部署使用的运行库和编译组件。

若目标系统没有 Docker Compose 或 NVIDIA Container Runtime，需要先从兼容 JetPack 的安装介质补齐宿主机依赖；这些不是应用代码的一部分。

## 加速引擎问题

`deploy/runtime/bin/engines.py` 对 GPU、计算能力、资源数量、推理库版本和模型哈希进行检查。不匹配会重建引擎。重建日志位于 `data/engines/`。

需要强制重建时，先停止 Web，随后执行：

```bash
sudo docker compose stop web
sudo docker compose run --rm --no-deps web build-engines
sudo docker compose up -d --pull never web
```

不要把“强制复用旧引擎”作为跨机迁移方案。参考：[NVIDIA 引擎兼容性说明](https://docs.nvidia.com/deeplearning/tensorrt/latest/inference-library/engine-compatibility.html)。

## 时间、数据和配置

首次启动前核对系统时间和时区。当前源码可以显示校时状态，但原机额外部署的宿主机校时服务与共享密钥没有包含在此仓库中。

本包以新数据库启动。若确需保留历史业务数据，应另外制定停机/一致性备份与凭据迁移方案。不要将业务数据库或摄像头认证信息提交到公开仓库。

## 模型和资源

内存大小、画面清晰度、并发数和上下文长度都会影响可运行性。此配置来自 AGX Orin，不代表所有 Jetson 型号都能使用相同并发配置。遇到资源不足时，先保留日志并降低并发，逐项验证，勿将异常结果当作正常无事件。

## 应用功能检查

- 检查 `/api/config`、`/api/vlm/status`。
- 使用短录像分别验证上传、分类、事件截图和取消。
- 对交通事件验证结构化研判和原始录像回看。
- 分别检查刀具、打斗专项；最后再测试组合模式。
- 按实际场景统计漏报、误报和端到端时延。

此包没有提供固定的准确率保证、自动事故定责或跨镜头长时跟踪保证。
