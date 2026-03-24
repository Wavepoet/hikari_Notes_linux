# AWS产品概述

## 计算服务

- Amazon EC2 (Elastic Compute Cloud):

提供弹性计算云，可动态调整大小的虚拟服务器（实例）。

- Amazon Lightsail:

Lambda是一种无服务器计算服务，允许运行代码而无需预置或管理底层操作系统，按代码实际执行的毫秒数计费。

- Amazon ECR (Elastic Container Registry)：

完全托管的 Docker 容器注册表，可存储和管理容器镜像。

## 存储服务

- Amazon S3 (Simple Storage Service):

对象存储服务，提供极高的持久性和扩展性。常用于存储静态文件、系统备份、日志归档以及作为数据湖的底层存储。

- Amazon EBS (Elastic Block Store):

EBS是弹性块存储，为EC2实例提供持久性存储。

## 数据库服务

- Amazon RDS (Relational Database Service):

托管的关系型数据库服务，支持 MySQL、PostgreSQL 等主流引擎。

- Amazon DynamoDB：

完全托管的无服务器键值 (Key-Value) 和文档数据库，能够处理海量并发请求并提供个位数毫秒级响应。

## 网络服务

- Amazon CloudFront：

AWS的CDN服务。

- Amazon VPC (Virtual Private Cloud)：

允许在逻辑隔离的自定义虚拟网络中启动 AWS 资源。

- Elastic Load Balancing (ELB):

自动在多个可用区的目标（如 EC2 实例、容器、IP 地址）之间分配传入的应用程序流量，确保应用的高可用性。

- Amazon Route 53:

高可用且可扩展的云 DNS Web 服务。

## 安全、身份与访问控制

- AWS IAM (Identity and Access Management):

用于精细化控制对 AWS 服务和资源的访问,以用户和组的形式管理权限。每一个用户默认都是没有任何权限的。
