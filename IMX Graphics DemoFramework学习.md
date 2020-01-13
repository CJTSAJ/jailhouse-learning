# High Level Overview
- DemoMain
  - 解析参数
  - 调用demoHost
  - 输出错误信息

- DemoHost
主要负责初始化和关闭host环境和运行main loop；main loop利用DemoAppManager控制DemoApp生命周期。

- DemoApp
一般需要特定的DemoHost支持，平台无关


# Demo application details

## Demo method overview
- Resized:
- Fixedupdate:
- Update:每次调用draw前会调用该函数
- Draw:渲染图像

- 在***_Register.cpp中注册demo类
- 

# Helper class overview

