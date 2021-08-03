---
title: 使用CloudWatch来监控Nginx日志
date: 2021-08-03 16:30:49
tags: 
- CloudWatch 
- Nginx
categories:
- AWS
---
 
> 场景：在使用Nginx的服务器上通过将日志文件推送到CloudWatch Log Group来实现日志监控同时创建Query Insights控件来实现Dashboard

## IAM权限配置

1. 在EC2的Policy Group中添加CloudWatchAgentPolicy

   ``` JSON
   {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:PutMetricData",
                "ec2:DescribeVolumes",
                "ec2:DescribeTags",
                "logs:PutLogEvents",
                "logs:DescribeLogStreams",
                "logs:DescribeLogGroups",
                "logs:CreateLogStream",
                "logs:CreateLogGroup"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ssm:GetParameter"
            ],
            "Resource": "arn:aws:ssm:*:*:parameter/AmazonCloudWatch-*"
        }
        ]
    }
   ```

## EC2 Nginx服务器端安装CloudWatch Agent

### 安装CloudWatch Agent

1. 使用Yum安装

    ``` bash
    sudo yum install amazon-cloudwatch-agent
    ```

2. 使用rpm安装

    ``` bash
    wget https://dl.grafana.com/oss/release/grafana-7.5.9-1.x86_64.rpm

    sudo yum install grafana-7.5.9-1.x86_64.rpm
    ```

### 配置CloudWatch Agent

1. 使用Wizard

   ``` bash
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
   ```

2. 手动配置

    ``` JSON
    {
     "agent": {
      "run_as_user": "root"
     },
     "logs": {
      "logs_collected": {
       "files": {
        "collect_list": [
         {
          "file_path": "/var/log/nginx/access.log",
          "log_group_name": "access.log",
          "log_stream_name": "{instance_id}"
         }
        ]
       }
      }
     }
    }
    ```

### 启动CloudWatch Agent

1. 重启

   ``` bash
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file://opt/aws/amazon-cloudwatch-agent/bin/config.json -s
   ```

2. 查看状态

   ``` bash
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
   ```

3. 停止

   ``` bash
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a stop
   ```
