## 部署

- [安装 Serverless Devs Cli 开发者工具](https://www.serverless-devs.com/serverless-devs/install) ，并进行[授权信息配置](https://docs.serverless-devs.com/fc/config)
- 进行项目部署：`s deploy - y`

## Stable Diffusion启动优化
Stable Diffusion应用的镜像包含运行环境及模型文件，构建后约为15.5GB。运行SD应用时需要先拉取镜像，再加载模型。直到模型加载完成之前，应用都无法对外提供服务，存在较长的冷启动时间。因此，我们需要对Stable Diffusion应用进行优化, 思路和方法不限。

- 对镜像进行优化，制作更小的镜像
- 优化应用启动流程，跳过环境校验
- 优化模型加载方式，提升加载速度
- ...

## 如何二次开发

1. 构建镜像    
请参考[build-image.md](build-image.md)文档构建镜像。
限制：需要使用`v1-5-pruned-emaonly.safetensors` 模型文件。

2. 上传镜像    
使用`docker push`命令将镜像上传到镜像仓库，并将仓库设置为可公开拉取。以阿里云镜像服务ACR为例:  
   1. 注册[阿里云镜像服务](https://cr.console.aliyun.com/)
   2. 执行以下命令上传镜像
   ```bash
   docker login registry.${regionId}.aliyuncs.com 
   docker tag sd:v1 registry.${regionId}.aliyuncs.com/${repoName}/stable-diffusion:v1
   docker registry.${regionId}.aliyuncs.com/${repoName}/stable-diffusion:v1
   ```
3. 更新stable-diffusion.yaml  
将stable-diffusion.yaml文件中`image: registry-vpc.cn-hangzhou.aliyuncs.com/fc-stable-diffusion/stable-diffusion:v1` 改为上一步上传的镜像 `image: registry-vpc.${regionId}.aliyuncs.com/${repoName}/stable-diffusion:v1`

4. 部署应用  
执行 `s deploy -y ` 重新部署应用

## 常见问题

#### 1 应用部署后无法访问  
为了提升冷启动时间，我们提供了镜像加速服务，请关注控制台上的镜像加速状态，只有在 ready 才真正可用。

#### 2 刚启动是输入提示词偶尔会无法生成图片  
这个可能是因为模型本身还未加载，请注意查看左上角选择框里面包含模型内容，之后再操作。出图的时候会有一定的等待时间，这个是正常现象，耐心等待即可

#### 3 镜像构建失败，提示No Space Left on Device
磁盘空间不足。构建Stable Diffusion镜像需要大量磁盘空间，建议预留50GB磁盘空间。

#### 4 镜像构建慢
[更换（Pypi）pip源到国内镜像](https://developer.aliyun.com/article/652884)

#### 5 资费消耗
Stable Diffusion需要使用GPU计算资源，我们默认使用按量付费的模式，不用的时会自动释放资源，减少资费消耗
