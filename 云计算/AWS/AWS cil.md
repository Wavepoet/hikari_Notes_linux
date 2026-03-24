# AWS CLI

AWS CIL 是 AWS 官方提供的命令行工具，可以用来管理 AWS 资源。

## 安装

- debian/ubuntu:

```bash
sudo apt-get update
sudo apt-get install -y awscli

aws --version
```

- CentOS/RHEL/Fedora:

```bash
yum install -y awscli

aws --version
```

- Windows

```bash
 winget install Amazon.AWSCLI
```

## 配置安全凭证

依次配置Access Key、Secret Key、默认区域……

```bash
aws configure
```

## 基本使用

到这里AWS CLI便可以使用了。

-获取当前 IAM 用户的身份信息:

```bash
aws sts get-caller-identity
```

- aws iam list-users

```bash
aws iam list-users
```

- 查看 S3 存储桶:

```bash
aws s3 ls
```

- 下载 S3 中的文件到 Windows 桌面

```bash
aws s3 cp s3://your-bucket-name/test.txt ~\Desktop\
```

- 上传文件到存储桶：

```bash
aws s3 cp my-file.txt s3://my-bucket-name/
```

- 启动一个实例EC2：

```bash
aws ec2 start-instances --instance-ids i-1234567890abcdef0
```
