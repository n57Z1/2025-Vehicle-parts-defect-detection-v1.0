同时还含有Web端(Vue)、小程序(Uniapp)、以及后端(FastApi)完整代码

## 小程序端

采用HBuilderX开发工具进行开发，基于Uniapp框架实现。小程序端的测试运行只需要直接将代码导入HBuilderX（内置小程序开发的各类插件）中即可进行实时的调试与开发。


## Web端

Web端基于Vue-admin-temple进行开发，**注意node环境最好采用14.15.4版本，高版本可能会出现环境问题**<br />
1. 在完成node环境的配置后，进入mask-web目录下，打开cmd，执行下面的命令
```
//下载模块到本地
npm install
```
2. 在当前目录(./mask-web)下，执行下面的命令可直接启动项目(默认81端口)
```
npm run serve
```

## 服务端

服务端接口采用FastApi开发，其中mask-server使用到YOLOv5，oldPhotoRectify-server使用到Bringing-Old-Photos-Back-to-Life和colorization两个开源项目（链接可点击下方**涉及开源项目**进行跳转）。由于模型大小较大，上传代码未包含训练得到的模型。<br />
常用python库已在requirement.txt中给出，可通过执行以下命令进行安装（注：由于本项目主要为本人大二期间完成，因此大部分库的版本较早）
```
pip install -r requirement.txt
```
服务端代码的运行，可直接进入server目录下，执行下面命令运行（注：上传代码中未包含模型，因此无法完成检测，但包含完整代码，且服务端可正常运行）
```
python main.py
```

---

## 涉及开源项目：
- [![目标检测算法](https://img.shields.io/badge/-YOLOv5-blue)](https://github.com/ultralytics/yolov5)
- [![图像修复](https://img.shields.io/badge/-Bringing--Old--Photos--Back--to--Life-orange)](https://github.com/microsoft/Bringing-Old-Photos-Back-to-Life)
- [![图像上色](https://img.shields.io/badge/-colorization-1E88B0)](https://github.com/richzhang/colorization)
- [![前端模板框架](https://img.shields.io/badge/-vue--admin--template-green)](https://github.com/PanJiaChen/vue-admin-template)

