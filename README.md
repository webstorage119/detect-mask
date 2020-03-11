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

### 文件说明
* template.yml 为模板
* viapi-detect-mask-demo-flow-serial.yaml 是工作流文件
    * 由于目前对外的viapi的免费qps只有1~2，所以多人的口罩识别使用串行处理
    * 流程图参照： ![图片](./workflow.svg)
    
### 操作步骤

#### 前置条件
* 开通服务
    * OSS
    * 函数计算
    * 函数工作流
    * RAM
    
#### 编译打包发布
* 编译
    * 在根目录下执行：```fun build```
* 打包
    * 拷贝工作流文件到 ```./.fun/build/artifacts/``` 目录下

        ```
        cp viapi-detect-mask-demo-flow-serial.yaml ./.fun/build/artifacts/ ``` 
        ```

    * 到 ```.fun/build/artifacts/```下执行package命令

        ```
         cd .fun/build/artifacts/
         ### 下面的viapi-test为发布ak有权限的bucket名称，可以替换为自有的bucket
         fun package --oss-bucket viapi-test -t template.yml
        ```

* 发布
    * 方式1： 把生成的template.packaged.yml 转成为json格式，然后存放到oss中，生成oss的http链接后，
        可以通过如下格式拼接链接，直接唤起ros创建
        ```
        ### 格式： https://rosnext.console.aliyun.com/cn-shanghai/stacks/create?templateUrl=${json文件链接}&step=1&hideTemplateSelector=true
        样例：https://rosnext.console.aliyun.com/cn-shanghai/stacks/create?templateUrl=http%3A%2F%2Fviapi-demo.oss-cn-shanghai.aliyuncs.com%2Ffunction%2FdetectMaskDemo2.json&step=1&hideTemplateSelector=true
        ```