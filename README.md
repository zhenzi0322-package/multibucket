
<p align="center">
  <h1>Multibucket</h1>
  <a href="https://pypi.org/project/multibucket/"><img src="https://img.shields.io/pypi/v/multibucket.svg" alt="PyPI version"></a>
  <a href="https://pypi.org/project/multibucket/"><img src="https://img.shields.io/badge/Python-3.8~3.14-3776AB?logo=python&logoColor=white" alt="Python"></a>
  <a href="https://github.com/zhenzi0322-package/multibucket/blob/master/LICENSE"><img src="https://img.shields.io/pypi/l/multibucket.svg" alt="License"></a>
          <a href="https://tool.long920.cn/multibucket"><img src="https://app.readthedocs.org/projects/zhenzi0322-tool/badge/?version=latest" alt="Documentation Status"></a>
</p>

> 多对象存储桶`SDK`，支持`UCloud UFile`等云存储服务。

## 运行条件

- Python >= 3.8
- requests

## 安装

```bash
pip install multibucket
```

## 快速开始

```python
from multibucket import UCloudFileClient

# 初始化客户端
ufile = UCloudFileClient(
    public_key="your_public_key",
    private_key="your_private_key",
    bucket="your-bucket",
    region="cn-bj",
    timeout=30  # 全局超时时间（秒，可选）
)

# 上传文件
with open("image.jpg", "rb") as f:
    resp = ufile.upload("photos/image.jpg", f.read(), content_type="image/jpg")
print(resp.status_code)  # 200

# 上传本地文件（自动猜测 Content-Type）
success = ufile.upload_file("photos/image.jpg", "/path/to/image.jpg")
print(success)  # True/False

# 带重试的上传（默认重试 3 次）
success = ufile.upload_with_retry("backup/data.zip", data, retries=3)
print(success)  # True/False

# 获取文件 URL
url = ufile.get_path_url("photos/image.jpg")
print(url)  # https://your-bucket.cn-bj.ufileos.com/photos/image.jpg

# 生成分享链接（私有空间，默认 20 分钟有效）
share_url = ufile.get_shared_url("photos/image.jpg", expire_time=3600)
print(share_url)

# 判断文件是否存在
exists = ufile.is_exists("photos/image.jpg")
print(exists)  # True/False

# 获取文件内容到内存
data = ufile.get_content("photos/image.jpg")  # bytes
text = ufile.get_content("config.json", as_text=True)  # str

# 下载文件到本地
success = ufile.download("photos/image.jpg", "downloaded.jpg")
print(success)  # True/False

# 分片下载（支持进度回调）
success = ufile.multipart_download("videos/large.mp4", "large.mp4")

# 查询文件信息
resp = ufile.head("photos/image.jpg")
print(resp.headers["Content-Length"])

# 列出文件
resp = ufile.list(prefix="photos/", limit=50)
files = resp.json()["DataSet"]

# 删除文件
resp = ufile.delete("photos/image.jpg")
print(resp.status_code)  # 204

# 删除文件（简化版）
success = ufile.delete_file("photos/image.jpg")
print(success)  # True/False
```

## 高级功能

### 自定义域名（CDN）

```python
ufile = UCloudFileClient(
    public_key="your_public_key",
    private_key="your_private_key",
    bucket="your-bucket",
    region="cn-bj",
    domain_url="https://cdn.example.com"  # 自定义 CDN 域名
)
```

### 内网访问

```python
ufile = UCloudFileClient(
    public_key="your_public_key",
    private_key="your_private_key",
    bucket="your-bucket",
    region="cn-wlcb",
    internal=True  # 使用内网访问
)
```

### 分片上传（大文件）

```python
# 自动分片上传，支持进度回调
def progress_callback(part_number, total_parts, uploaded, total):
    print(f"进度: {part_number + 1}/{total_parts}")

result = ufile.multipart_upload(
    key="videos/large.mp4",
    file_path="/path/to/large.mp4",
    content_type="video/mp4",
    callback=progress_callback
)
```
