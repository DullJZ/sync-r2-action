# GitHub Action: 通过 rclone 同步文件到 R2

本 Action 使用 `rclone` 将你的 GitHub 仓库工作空间中的文件（通过包含过滤器筛选，默认为常见图片类型）同步或复制到 Cloudflare R2 存储桶。

## 功能特性

* 使用 `rclone` 进行高效的云存储操作。
* 可配置的源目录和目标目录。
* 支持 `sync`（镜像源，删除目标中多余文件）和 `copy`（仅添加/更新文件）两种模式。
* 允许指定自定义的 `rclone` 包含过滤器。
* 允许传递额外的 `rclone` 标志。
* 使用提供的 R2 凭证即时配置 `rclone`。

## 输入参数

| 输入参数 (Input)       | 描述                                                                 | 是否必须 | 默认值                                       |
| ---------------------- | -------------------------------------------------------------------- | -------- | -------------------------------------------- |
| `r2_access_key_id`     | Cloudflare R2 Access Key ID。                                        | 是       | N/A                                          |
| `r2_secret_access_key` | Cloudflare R2 Secret Access Key。                                    | 是       | N/A                                          |
| `r2_endpoint`          | Cloudflare R2 Endpoint URL (例如: `https://<ACCOUNT_ID>.r2...`).     | 是       | N/A                                          |
| `r2_bucket_name`       | Cloudflare R2 存储桶的名称。                                         | 是       | N/A                                          |
| `source_dir`           | 仓库工作空间内要同步的源目录。                                       | 否       | `.`                                          |
| `destination_dir`      | R2 存储桶内的目标目录（前缀）。留空表示根目录。                        | 否       | `''`                                         |
| `include_filter`       | 空格分隔的 rclone include 过滤器模式列表。                           | 否       | `*.jpg *.jpeg *.png *.gif *.svg *.webp`       |
| `sync_mode`            | 操作模式：`copy` 或 `sync`。                                         | 否       | `copy`                                       |
| `rclone_flags`         | 直接传递给 rclone 命令的附加标志 (例如：`--dry-run`)。               | 否       | `--progress`                                 |

**注意:** R2 凭证应作为[加密的 Secrets](https://docs.github.com/zh/actions/security-guides/encrypted-secrets) 存储在使用本 Action 的仓库中。

## 使用示例

```yaml
name: 部署图片到 R2

on:
  push:
    branches: [ main ]
    paths:
      - 'assets/images/**' # 仅当图片改变时运行

jobs:
  sync_to_r2:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout 代码
        uses: actions/checkout@v4

      - name: 同步图片到 R2
        uses: DullJZ/sync-r2-action@v1
        with:
          # 使用 Secrets 传递凭证
          r2_access_key_id: ${{ secrets.R2_ACCESS_KEY_ID }}
          r2_secret_access_key: ${{ secrets.R2_SECRET_ACCESS_KEY }}
          r2_endpoint: ${{ secrets.R2_ENDPOINT }} # 也将你的 R2 endpoint 作为 Secret 存储
          r2_bucket_name: ${{ secrets.R2_BUCKET_NAME }}

          # 可选：指定源和目标目录
          source_dir: './assets/images'
          destination_dir: 'website-images' # 上传到存储桶内的 'website-images/' 前缀下

          # 可选：使用 sync 模式替代 copy
          # sync_mode: 'sync'

          # 可选：自定义包含过滤器 (如果不仅仅是图片)
          # include_filter: '*.jpg *.png *.css'

          # 可选：添加额外的 rclone 标志
          # rclone_flags: '--dry-run --verbose'