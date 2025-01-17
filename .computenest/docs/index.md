# 服务模版说明文档

## 服务说明

本文介绍基于SpringBoot软件包快速构建托管版单租计算巢服务，关于计算巢托管版可以参考[帮助文档](https://help.aliyun.com/zh/compute-nest/create-a-fully-managed-service?spm=a2c4g.11174283.0.i5)，
本示例对应的git地址为：[springboot-managed-demo](https://github.com/aliyun-computenest/springboot-managed-demo)
。根据该服务模板构建的服务默认包含三种套餐：

| 套餐名 | ECS规格族         | vCPU与内存          | 系统盘               | 公网带宽      |
|-----|----------------|------------------|-------------------|-----------|
| 低配版 | ecs.c6.large   | 内存型c6，2vCPU 4GiB | ESSD云盘 200GiB PL0 | 固定带宽1Mbps |
| 基础版 | ecs.c6.xlarge  | 内存型c6，4vCPU 8GiB | ESSD云盘 200GiB PL0 | 固定带宽1Mbps |
| 高配版 | ecs.c6.2xlarge | 内存型c6，4vCPU 8GiB | ESSD云盘 200GiB PL0 | 固定带宽1Mbps |

本示例会自动的构建计算巢服务，具体的服务构建流程为

1. 上传文件并构建计算巢文件部署物
2. 创建计算巢服务并关联文件部署物

创建过程大约持续2分钟，当服务变成待提交后构建成功

## 服务架构

本部署架构为单机ecs部署，通过公网ip 8080端口访问
![image.png](architecture.png)

## 服务构建计费说明

测试本服务构建无需任何费用，创建服务实例涉及的费用参考服务实例计费说明

## RAM账号所需权限

本服务需要对ECS、VPC等资源进行访问和创建操作，若您使用RAM用户创建服务实例，需要在创建服务实例前，对使用的RAM用户的账号添加相应资源的权限。添加RAM权限的详细操作，请参见[为RAM用户授权](https://help.aliyun.com/document_detail/121945.html)
。所需权限如下表所示。

| 权限策略名称                              | 备注                          |
|-------------------------------------|-----------------------------|
| AliyunECSFullAccess                 | 管理云服务器服务（ECS）的权限            |
| AliyunVPCFullAccess                 | 管理专有网络（VPC）的权限              |
| AliyunROSFullAccess                 | 管理资源编排服务（ROS）的权限            |
| AliyunComputeNestUserFullAccess     | 管理计算巢服务（ComputeNest）的用户侧权限  |
| AliyunComputeNestSupplierFullAccess | 管理计算巢服务（ComputeNest）的服务商侧权限 |

## 服务实例计费说明

由于本服务式托管版服务，资源属于服务商，所以创建服务实例的费用由服务商承担，主要涉及：

- 所选vCPU与内存规格
- 系统盘类型及容量
- 公网带宽

计费方式包括：

- 按量付费（小时）
- 包年包月

目前提供如下套餐：

| 套餐名 | ECS规格族         | vCPU与内存          | 系统盘               | 公网带宽      |
|-----|----------------|------------------|-------------------|-----------|
| 低配版 | ecs.c6.large   | 内存型c6，2vCPU 4GiB | ESSD云盘 200GiB PL0 | 固定带宽1Mbps |
| 基础版 | ecs.c6.xlarge  | 内存型c6，4vCPU 8GiB | ESSD云盘 200GiB PL0 | 固定带宽1Mbps |
| 高配版 | ecs.c6.2xlarge | 内存型c6，4vCPU 8GiB | ESSD云盘 200GiB PL0 | 固定带宽1Mbps |

## 服务实例部署流程

### 部署参数说明

| 参数项  | 说明                                                   |
|------|------------------------------------------------------|
| 实例密码 | 服务器登录密码,长度8-30，必须包含三项（大写字母、小写字母、数字、 ()`~!@#$%^&*_-+= |{}[]:;'<>,.?/ 中的特殊符号） |

### 部署步骤

0. 部署链接
   ![image.png](部署链接.png)
1. 单击部署链接，进入服务实例部署界面，根据界面提示，填写参数。
   ![image.png](部署参数.png)
2. 确认参数后点击**立即创建**。
   ![image.png](立即创建.png)
3. 租户创建出新的服务实例，处于待部署状态，需要服务商确认后才可以进入部署流程。
   ![image.png](待部署.png)
4. 服务商点击**部署**。
   ![image.png](服务商确认部署.png)
5. 随后该服务实例处于"部署中"状态。
   ![image.png](部署中.png)
6. 等待部署完成后就可以开始使用服务，进入服务实例详情点击visitUrl。
   ![image.png](使用服务.png)
5. 返回结果
   ![image.png](结果.png)

## 服务详细说明

本文通过将[代码](https://atomgit.com/flow-example/spring-boot)
构建后，将deploy.sh和application.jar打包成package.tgz，通过计算巢部署物上传为文件部署物，并分发生成SpringBootPackage，然后在模版中ALIYUN::
ECS::RunCommand执行命令

```
yum install -y java
mkdir -p /home/admin/application
cd /home/admin/application
wget '{{ computenest::file::springboot }}' -O package.tgz
tar xvf package.tgz
/bin/bash deploy.sh start
```

{{ computenest::file::springboot }} 为占位符，会由计算巢服务替换成文件部署物SpringBootPackage的http下载地址

templates/template.yaml主要由三部分组成

1. Parameters定义需要用户填写的参数，包括付费类型，实例规格和实例密码可用区参数

```
EcsInstanceType:
 Type: String
 Label:
   en: Instance Type
   zh-cn: 实例类型
 AssociationProperty: ALIYUN::ECS::Instance::InstanceType
 AssociationPropertyMetadata:
   InstanceChargeType: ${PayType}
 AllowedValues:
   - ecs.c6.large
   - ecs.c6.xlarge
   - ecs.c6.2xlarge
InstancePassword:
 NoEcho: true
 Type: String
 Description:
   en: Server login password, Length 8-30, must contain three(Capital letters, lowercase letters, numbers, ()`~!@#$%^&*_-+=|{}[]:;'<>,.?/ Special symbol in)
   zh-cn: 服务器登录密码,长度8-30，必须包含三项（大写字母、小写字母、数字、 ()`~!@#$%^&*_-+=|{}[]:;'<>,.?/ 中的特殊符号）
 AllowedPattern: '^[a-zA-Z0-9-\(\)\`\~\!\@\#\$\%\^\&\*\_\-\+\=\|\{\}\[\]\:\;\<\>\,\.\?\/]*$'
 Label:
   en: Instance Password
   zh-cn: 实例密码
 ConstraintDescription:
   en: Length 8-30, must contain three(Capital letters, lowercase letters, numbers, ()`~!@#$%^&*_-+=|{}[]:;'<>,.?/ Special symbol in)
   zh-cn: 长度8-30，必须包含三项（大写字母、小写字母、数字、 ()`~!@#$%^&*_-+=|{}[]:;'<>,.?/ 中的特殊符号）
 MinLength: 8
 MaxLength: 30
 AssociationProperty: ALIYUN::ECS::Instance::Password
# 可用区
ZoneId:
 Type: String
 AssociationProperty: ALIYUN::ECS::Instance:ZoneId
 Label:
   en: VSwitch Available Zone
   zh-cn: 可用区
VpcId:
 AssociationProperty: ALIYUN::ECS::VPC::VPCId
 Type: String
 Label:
   en: VPC ID
   zh-cn: 专有网络VPC实例ID
# 交换机实例ID
VSwitchId:
 AssociationProperty: ALIYUN::ECS::VSwitch::VSwitchId
 AssociationPropertyMetadata:
   VpcId: ${VpcId}
   ZoneId: ${ZoneId}
 Type: String
 Label:
   en: VSwitch ID
   zh-cn: 交换机实例ID
```

2. Resources定义需要开的资源，包括新开的vpc, vswitch和ecs实例, 以及执行命令的定义

```
SecurityGroup:
 Type: ALIYUN::ECS::SecurityGroup
 Properties:
   SecurityGroupName:
     Ref: ALIYUN::StackName
   VpcId:
     Ref: VpcId
   # 安全组入端口
   SecurityGroupIngress:
     - PortRange: 8080/8080
       Priority: 1
       SourceCidrIp: 0.0.0.0/0
       IpProtocol: tcp
       NicType: internet
InstanceGroup:
 Type: ALIYUN::ECS::InstanceGroup
 Properties:
   # 付费类型
   InstanceChargeType: PostPaid
   VpcId:
     Ref: VpcId
   VSwitchId:
     Ref: VSwitchId
   SecurityGroupId:
     Ref: SecurityGroup
   ZoneId:
     Ref: ZoneId
   ImageId: centos_7_8_x64_20G_alibase_20211130.vhd
   Password:
     Ref: InstancePassword
   InstanceType:
     Ref: EcsInstanceType
   SystemDiskCategory: cloud_essd
   SystemDiskSize: 200
   InternetMaxBandwidthOut: 1
   IoOptimized: optimized
   MaxAmount: 1
InstallPackage:
 Type: ALIYUN::ECS::RunCommand
 Properties:
   InstanceIds:
     Fn::GetAtt:
     - InstanceGroup
     - InstanceIds
   Type: RunShellScript
   Sync: true
   Timeout: 3600
   CommandContent:
     Fn::Sub:
       - |
         #!/bin/bash
         yum install -y java
         mkdir -p /home/admin/application
         cd /home/admin/application
         wget '{{ computenest::file::springboot }}' -O package.tgz
         tar xvf package.tgz
         /bin/bash deploy.sh start
       - AccountId:
           Ref: ALIYUN::TenantId
```

3. Outputs定义需要最终在计算巢概览页中对用户展示的输出

```
Outputs:
  VisitUrl:
    Description:
      en: VisitUrl.
      zh-cn: 访问页面。
    Value:
      Fn::Sub:
      - http://${Address}:8080
      - Address:
          Fn::Select:
          - 0
          - Fn::GetAtt:
            - InstanceGroup
            - PublicIps
```

## 服务配置

[创建代运维服务完成实例运维](https://help.aliyun.com/zh/compute-nest/create-a-hosted-operations-and-maintenance-service?spm=a2c4g.11186623.0.i24#task-2167552])

[创建包含变配功能的服务](https://help.aliyun.com/zh/compute-nest/use-cases/create-a-service-that-supports-specification-changes-and-change-the-specifications-of-a-service-instance?spm=a2c4g.11186623.0.i3])

[创建包含服务升级功能的服务](https://help.aliyun.com/zh/compute-nest/upgrade-a-service-instance?spm=a2c4g.11186623.0.i17#task-2236803)

## 服务交付

[自定义服务架构图](https://help.aliyun.com/zh/compute-nest/customize-a-service-architecture?spm=a2c4g.11186623.0.0.56e736bfyUdlFm])

[服务文档上线流程](https://help.aliyun.com/zh/compute-nest/use-cases/publish-documents-to-compute-nest?spm=a2c4g.313309.0.i0])

[将服务上架云市场并上到云市场售卖](https://help.aliyun.com/zh/compute-nest/publish-a-service-to-alibaba-cloud-marketplace?spm=a2c4g.11186623.0.i7])

## 其他

[实例代码源地址](https://atomgit.com/flow-example/spring-boot)

[软件包package.tgz构建流程参考](https://help.aliyun.com/document_detail/153848.html)
