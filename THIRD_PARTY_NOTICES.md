# 第三方组件与模型说明

本项目集成多个第三方软件和模型。前端展示名称与开发依赖名称是不同层次；开发文档保留真实依赖，便于迁移、核查和遵守许可。

- 前端包含 Vue 3.5.39 和 Element Plus 2.14.2 的本地分发文件，原有文件头保留；对应 MIT 许可放在 `licenses/`。
- Python/GPU 依赖版本见 `environment/installed-packages.reference.txt`。该列表用于复核，不是面向所有 Jetson 的通用安装锁文件。
- 已收集到的 Python 依赖许可保留于 `licenses/python/`。
- 大模型权重、刀具与人员检测权重、动作识别权重未上传 GitHub；模型来源与哈希见 `src/models/`。
- 模型原始来源存在不同许可及部分需要进一步澄清的说明，已保留在元信息中。不能将本仓库视为对所有权重和训练数据授予商用或再分发权利。
- Qwen3.5-4B-AWQ 原模型许可文本保留在 `licenses/Qwen3.5-4B-AWQ-LICENSE.txt`，权重本身在参考运行镜像中。

参考来源：

- https://github.com/vuejs/core/blob/main/LICENSE
- https://github.com/element-plus/element-plus/blob/dev/LICENSE
- 具体模型的来源 URL、版本和许可说明见 `src/models/manifest.json`。

迁移包不包含演示视频、真实摄像头录像或业务截图。
