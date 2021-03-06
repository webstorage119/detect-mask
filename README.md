### 功能说明
* 为用户创建一个上海region的oss bucket（名称为：viapi-func-demo-${timestamp}-${random(0-100)}）
* 自动在bucket上创建source和target目录，并在source目录上创建一个触发器，对于新增的图片文件(.jpg,jpeg,.png). 会自动触发口罩识别功能
* 多人脸口罩检测功能
    * 调用检测服务，获取人脸数量，以及人脸的坐标
    * 口罩检测
        * 单个人脸时：调用人脸口罩
        * 多人脸时：逐一进行人脸裁剪并存入oss中（target目录下），并调用人脸口罩识别
        * 没有人脸： 不做处理
    * 把整体结果作为json文件写入到target目录中

* 备注：样例中的FC、FnF目前仅支持上海region

### 工作原理

* 资源创建： 通过 ROS 调用 FC 创建 OSS bucket，并在 bucket 上创建 source 和 target 文件夹
* 事件驱动：OSS 事件源可以直接自动触发函数的执行
* FNF 编排流程，会结合人脸识别、单人口罩识别、图片裁剪服务，实现多人脸的口罩识别并输出多张人脸图片和所属区域

### 文件说明
* template.yml 为模板
* detect-mask-flow.yaml 是工作流文件
    
### 操作步骤

#### 前置条件
* 开通如下服务
    * 视觉智能开放平台-人脸人体服务 [链接](https://vision.aliyun.com/facebody)
    * 函数计算（Function Compute, FC） [链接](https://www.aliyun.com/product/fc)
    * 函数工作流 （Function Flow, FnF） [链接](https://www.aliyun.com/product/fnf)
    * 对象存储服务（Object Storage Service，OSS） [链接](https://www.aliyun.com/product/oss)
    * 访问控制（RAM） [链接](https://buy.aliyun.com/ram)


* 本地安装应用部署工具Fun [链接](https://help.aliyun.com/document_detail/140283.html)
    
#### 编译打包发布
* 编译
    * 在根目录下执行：```IGNORE_TPL_VALIDATION=1 fun build```
* 打包
    * 拷贝工作流文件到 ```./.fun/build/artifacts/``` 目录下

        ```
        cp detect-mask-flow.yaml ./.fun/build/artifacts/
        ```

    * 在根目录下执行package命令

        ```
         ### 下面的 mask-test-demo 为发布ak有权限的bucket名称，可以替换为自有的bucket
         fun package --oss-bucket mask-test-demo template.yml
        ```

* 发布
    ```bash
      fun deploy --use-ros --stack-name test
    ```