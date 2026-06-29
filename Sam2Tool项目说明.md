# Sam2Tool 项目说明

## 项目定位

Sam2Tool 是从 OgSpirit 中拆出来的独立 SAM2 工具前端仓库。当前仓库只保留 SAM2 这一个功能，不包含登录、用户系统、历史记录、平台首页、工具列表和其他插件。

这个仓库实现的是浏览器端交互界面，包括视频上传入口、首帧展示、前景/背景点选、生成流程控制、进度展示、序列帧展示、ZIP 下载入口、循环帧段预览和精灵图导出。

## 当前仓库实现内容

```text
Sam2Tool/
├── index.html
├── package.json
├── vite.config.ts
├── tsconfig.json
├── postcss.config.js
├── .env.example
├── README.md
└── src/
    ├── main.tsx
    ├── styles.css
    ├── vite-env.d.ts
    ├── components/
    │   ├── ToolWorkspace.tsx
    │   ├── FrameGallery.tsx
    │   └── LoopControlPanel.tsx
    ├── hooks/
    │   └── useSam2.ts
    ├── services/
    │   └── sam2Api.ts
    └── utils/
        └── frameProcessor.ts
```

主要文件职责：

| 文件 | 说明 |
| --- | --- |
| `src/main.tsx` | React 入口，渲染单工具页面 |
| `src/styles.css` | 全局样式 |
| `src/components/ToolWorkspace.tsx` | 主工作区，负责上传、标注、生成、下载等页面交互 |
| `src/components/FrameGallery.tsx` | 展示生成后的序列帧 |
| `src/components/LoopControlPanel.tsx` | 负责循环帧段选择、循环预览、精灵图导出 |
| `src/hooks/useSam2.ts` | 前端业务状态管理，串联上传、点选、生成、轮询和结果展示 |
| `src/services/sam2Api.ts` | 封装前端对 SAM2 后端 API 的请求 |
| `src/utils/frameProcessor.ts` | 前端侧帧处理辅助逻辑 |

## 后端位置

SAM2 后端不在当前前端仓库中。后端原本封装在 AutoDL 机器的数据盘里，可以从数据盘恢复。

历史后端相关目录：

```text
/root/autodl-tmp/
├── tmp/                   # 临时文件目录
├── pip_cache/             # pip 缓存目录
├── conda_envs/            # conda 环境目录
├── segment-anything-2/    # SAM2 源码
└── sam2-api/              # SAM2 API 服务代码
```

核心后端目录：

```text
/root/autodl-tmp/sam2-api/
```

SAM2 源码目录：

```text
/root/autodl-tmp/segment-anything-2/
```

历史启动命令：

```bash
cd /root/autodl-tmp/sam2-api
source /root/autodl-tmp/miniforge3/etc/profile.d/conda.sh
conda activate sam2_project
unset http_proxy https_proxy all_proxy
uvicorn api_server:app --host 0.0.0.0 --port 6006
```

这套后端的作用是接收前端上传的视频，完成视频拆帧、SAM2 点选推理、mask 生成、透明序列帧导出、ZIP 打包，并向前端返回图片和下载地址。

## 前后端连接方式

前端默认请求：

```text
http://localhost:6006/sam2/api/v1
```

对应配置在：

```text
src/services/sam2Api.ts
```

如果后端部署在其他服务器，需要在前端仓库根目录创建 `.env.local`：

```text
VITE_SAM2_API_URL=http://后端地址/sam2/api/v1
```

修改 `.env.local` 后需要重启前端：

```powershell
npm run dev
```

## 后端接口契约

前端期望后端提供以下接口，路径均相对于 `VITE_SAM2_API_URL`：

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| `POST` | `/upload` | 上传视频并创建任务 |
| `POST` | `/tasks/:taskId/click` | 添加前景或背景点 |
| `POST` | `/tasks/:taskId/undo` | 撤销上一个点 |
| `POST` | `/tasks/:taskId/reset` | 清空点 |
| `POST` | `/tasks/:taskId/generate` | 开始生成序列帧 |
| `GET` | `/tasks/:taskId` | 查询任务状态 |
| `GET` | `/tasks/:taskId/frame/0` | 获取第一帧 |
| `GET` | `/tasks/:taskId/mask` | 获取当前 mask 预览 |
| `GET` | `/tasks/:taskId/frames/:index` | 获取指定结果帧 |
| `GET` | `/tasks/:taskId/download` | 下载 ZIP 结果包 |

## 前端启动方式

```powershell
git clone https://github.com/Lincode-hormony/Sam2Tool.git
cd Sam2Tool
npm install
npm run dev
```

默认访问：

```text
http://127.0.0.1:3000
```

如果 3000 端口被占用：

```powershell
npm run dev -- --port 3001
```

## 完整运行条件

完整运行需要同时满足：

1. 前端仓库正常启动。
2. AutoDL 数据盘中的 `/root/autodl-tmp/sam2-api/` 后端已恢复并启动。
3. 后端可以访问 SAM2 源码、模型文件和运行环境。
4. 前端的 `VITE_SAM2_API_URL` 指向可访问的后端 API 地址。

如果只启动前端，没有启动后端，页面可以打开，但上传视频和生成结果相关功能无法完成。
