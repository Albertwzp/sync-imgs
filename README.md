# sync-imgs

通过 GitHub Actions 构建仓库中各目录的 Docker 镜像，并推送到个人 Docker Hub。

## 目录约定

工作流会递归查找所有文件名为 `Dockerfile` 的文件。每个 Dockerfile 所在目录会作为该镜像的构建上下文。

例如：

```text
.
├── nginx/
│   └── Dockerfile
└── tools/
    └── kubectl/
        └── Dockerfile
```

以上目录会分别推送为：

```text
<dockerhub-username>/nginx
<dockerhub-username>/tools-kubectl
```

目录路径会转换为小写，并将非字母、数字字符替换为 `-`。如果转换后的镜像名重复，工作流会报错并停止，避免覆盖错误的镜像。

## Docker Hub 配置

在 GitHub 仓库的 **Settings > Secrets and variables > Actions** 中添加：

| Secret | 说明 |
| --- | --- |
| `DOCKERHUB_USERNAME` | Docker Hub 用户名 |
| `DOCKERHUB_TOKEN` | Docker Hub access token，不要使用账户密码 |

Docker Hub access token 可在 **Docker Hub > Account settings > Personal access tokens** 中创建，需要具备镜像仓库的读写权限。

## 运行方式

[构建工作流](.github/workflows/build-images.yml) 会在以下情况运行：

- 推送或合并到 `main` 分支时，仅构建本次提交变更中包含的 Dockerfile；
- 在 GitHub Actions 页面手动触发时，可填写 `folder` 参数，仅构建该文件夹内的 Dockerfile；留空则构建全部 Dockerfile。

每个 Dockerfile 对应一个独立的构建任务和一次镜像推送。

每个构建都会推送以下标签：

- 当前分支名；
- `sha-<commit>`；
- 默认分支额外推送 `latest`。

如果仓库中暂时没有 Dockerfile，发现任务会正常完成，并跳过构建任务。
