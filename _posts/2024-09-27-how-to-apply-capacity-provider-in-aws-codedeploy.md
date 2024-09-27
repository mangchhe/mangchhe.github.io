---
layout: post
title: How to Apply ECS Capacity Provider in AWS CodeDeploy
categories: devops
tags: devops
---

최근 CodeDeploy를 통해 ECS 서비스를 배포하면서, 기본 용량 공급자(Capacity Provider)로 설정된 FARGATE_SPOT이 아닌 FARGATE로 태스크가 생성되는 문제를 경험했다.

이 문제는 ECS 클러스터에 설정된 Capacity Provider가 배포 시 제대로 반영되지 않았고 다음 이슈로 인해 해결책을 얻을 수 있었다.

> [https://github.com/aws/containers-roadmap/issues/695](https://github.com/aws/containers-roadmap/issues/695)
> 
> AWS CodeDeploy uses the AppSpec file as the source of truth for the capacity provider strategy

배포 시 AppSpec 파일에 명시적으로 용량 공급자(Capacity Provider)를 FARGATE_SPOT으로 설정하여 해결할 수 있었다.

```yml
{
  "version": 1,
  "Resources": [
    {
      "TargetService": {
        "Type": "AWS::ECS::Service",
        "Properties": {
          "TaskDefinition": "arn:aws:ecs:us-west-2:<MyAccountNumber>:task-definition/<MyTaskDefinitionName>:<MyTaskDefinitionVersion>",
          "LoadBalancerInfo": {
            "ContainerName": "<MyContainerName>",
            "ContainerPort": <MyContainerPort>
          },
          "CapacityProviderStrategy": [
            {
                "CapacityProvider": "<MyCapacityProviderNameFromECSCluster>",
            	"Base": 0,
                "Weight": 1
            }
          ]
        }
      }
    }
  ]
}
```

```sh
aws deploy create-deployment \
    --application-name {CODEDEPLOY_APP_NAME} \
    --deployment-group-name {CODEDEPLOY_DEPLOYMENT_GROUP} \
    --deployment-config-name CodeDeployDefault.ECSAllAtOnce \
    --revision "{\"revisionType\":\"AppSpecContent\",\"appSpecContent\":{\"content\":{APPSPEC_CONTENT}}}"
```

> [https://github.com/aws/containers-roadmap/issues/713](https://github.com/aws/containers-roadmap/issues/713)
> [https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-resources.html](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-resources.html)