## Stable Diffusion 启动优化

Stable Diffusion 应用的镜像包含运行环境及模型文件，构建后约为 15.5GB。运行 SD 应用时需要先拉取镜像，再加载模型。直到模型加载完成之前，应用都无法对外提供服务，存在较长的冷启动时间。因此，我们需要对 Stable Diffusion 应用进行优化, 思路和方法不限。

- 对镜像进行优化，制作更小的镜像
- 优化应用启动流程，跳过环境校验
- 优化模型加载方式，提升加载速度
- ...

## 如何二次开发

请参考[build-image.md](build-image.md)文档重新构建镜像。

**限制：** 需要使用`v1-5-pruned-emaonly.safetensors` 模型文件。

### 注意事项

#### 1 镜像构建失败，提示 No Space Left on Device

磁盘空间不足。构建 Stable Diffusion 镜像需要大量磁盘空间，建议预留 50GB 磁盘空间。

#### 2 镜像构建慢

[更换（Pypi）pip 源到国内镜像](https://developer.aliyun.com/article/652884)

#### 3 刚启动是输入提示词偶尔会无法生成图片

这个可能是因为模型本身还未加载，请注意查看左上角选择框里面包含模型内容，之后再操作。出图的时候会有一定的等待时间，这个是正常现象，耐心等待即可

## 如何测试 Stable Diffusion 启动优化性能提升

可以有如下几种方式的任何一种来测试您的优化效果

### 1. 本地机器

好处: 无费用

劣势: 需要本地机器有 GPU 并且有一定的硬件规格条件运行 Stable Diffusion

### 2. 阿里云 GPU 实例

比如使用 ecs.gn6i 8c32g T4 实例作为测试机器

劣势: 有一定费用产生，GPU 实例暂无免费试用， 详情见[阿里云免费试用](https://free.aliyun.com/?product=9555927&spm=a2c22.12281978.0.0.74f91bfdVTvDQ4)

### 3. [阿里云函数计算](https://help.aliyun.com/document_detail/52895.html)

[【免费领用】函数计算 FC 免费领用](https://free.aliyun.com/?pipCode=fc&spm=a2c22.12281978.0.0.74f91bfdVTvDQ4)

1. 按照上文中二次开发方法重新构建镜像，然后上传镜像
   使用`docker push`命令将镜像上传到镜像仓库，并将仓库设置为可公开拉取。以阿里云镜像服务 ACR 为例:

   1. 注册[阿里云镜像服务](https://cr.console.aliyun.com/)
   2. 执行以下命令上传镜像

   ```bash
   docker login registry.${regionId}.aliyuncs.com
   docker tag sd:v1 registry.${regionId}.aliyuncs.com/${repoName}/stable-diffusion:v1
   docker registry.${regionId}.aliyuncs.com/${repoName}/stable-diffusion:v1
   ```

2. 更新 stable-diffusion.yaml  
   将 stable-diffusion.yaml 文件中`image: registry-vpc.cn-hangzhou.aliyuncs.com/fc-stable-diffusion/stable-diffusion:v1` 改为上一步上传的镜像 `image: registry-vpc.${regionId}.aliyuncs.com/${repoName}/stable-diffusion:v1`

3. 部署应用
   - [安装 Serverless Devs Cli 开发者工具](https://www.serverless-devs.com/serverless-devs/install) ，并进行[授权信息配置](https://docs.serverless-devs.com/fc/config)
   - 执行 `s deploy -y -t stable-diffusion.yaml` 部署应用
   - 部署完毕后，终端输出可以直接访问使用的域名，浏览器打开域名即可

您可以直接通过函数计算控制台的可观测或者自己通过打印日志获取模型启动时间

**注意：** 应用部署后无法访问，为了提升冷启动时间，我们提供了镜像加速服务，请关注控制台上的镜像加速状态，只有在 ready 才真正可用。
