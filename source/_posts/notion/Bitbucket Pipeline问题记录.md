---
created: 2023-02-27 01:37:00
categories:
  - linux
tags:
  - bitbucket
  - pipeline
  - ci
  - cd
updated: 2024-04-03 03:32:00
date: 2023-02-27 00:00:00
title: Bitbucket Pipeline问题记录
cover: https://www.notion.so/images/page-cover/met_william_morris_1877_willow.jpg
id: 05c3d026-62ed-4b36-8187-768ebfde9ae6
---

# **Bitbucket Pipelines configuration reference**

<div style="width: 100%; margin-top: 4px; margin-bottom: 4px;"><div style="display: flex; background:white;border-radius:5px"><a href="https://support.atlassian.com/bitbucket-cloud/docs/bitbucket-pipelines-configuration-reference/"target="_blank"rel="noopener noreferrer"style="display: flex; color: inherit; text-decoration: none; user-select: none; transition: background 20ms ease-in 0s; cursor: pointer; flex-grow: 1; min-width: 0px; flex-wrap: wrap-reverse; align-items: stretch; text-align: left; overflow: hidden; border: 1px solid rgba(55, 53, 47, 0.16); border-radius: 5px; position: relative; fill: inherit;"><div style="flex: 4 1 180px; padding: 12px 14px 14px; overflow: hidden; text-align: left;"><div style="font-size: 14px; line-height: 20px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; min-height: 24px; margin-bottom: 2px;">Bitbucket Pipelines configuration reference | Bitbucket Cloud | Atlassian Support</div><div style="font-size: 12px; line-height: 16px; color: rgba(55, 53, 47, 0.65); height: 32px; overflow: hidden;">The complete configuration reference for Bitbucket Pipelines with all options/settings available in the Bitbucket Pipelines bitbucket-pipelines.yml</div><div style="display: flex; margin-top: 6px; height: 16px;"><img src="https://wac-cdn.atlassian.com/assets/img/favicons/atlassian/favicon.png"style="width: 16px; height: 16px; min-width: 16px; margin-right: 6px;"><div style="font-size: 12px; line-height: 16px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">https://support.atlassian.com/bitbucket-cloud/docs/bitbucket-pipelines-configuration-reference/</div></div></div></a></div></div>

# 无法在 `stage`中使用 `parallel`

<div style="width: 100%; margin-top: 4px; margin-bottom: 4px;"><div style="display: flex; background:white;border-radius:5px"><a href="https://support.atlassian.com/bitbucket-cloud/docs/parallel-step-options/"target="_blank"rel="noopener noreferrer"style="display: flex; color: inherit; text-decoration: none; user-select: none; transition: background 20ms ease-in 0s; cursor: pointer; flex-grow: 1; min-width: 0px; flex-wrap: wrap-reverse; align-items: stretch; text-align: left; overflow: hidden; border: 1px solid rgba(55, 53, 47, 0.16); border-radius: 5px; position: relative; fill: inherit;"><div style="flex: 4 1 180px; padding: 12px 14px 14px; overflow: hidden; text-align: left;"><div style="font-size: 14px; line-height: 20px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; min-height: 24px; margin-bottom: 2px;">Parallel step options | Bitbucket Cloud | Atlassian Support</div><div style="font-size: 12px; line-height: 16px; color: rgba(55, 53, 47, 0.65); height: 32px; overflow: hidden;">Run multiple pipeline steps concurrently or in parallel to reduce build time</div><div style="display: flex; margin-top: 6px; height: 16px;"><img src="https://wac-cdn.atlassian.com/assets/img/favicons/atlassian/favicon.png"style="width: 16px; height: 16px; min-width: 16px; margin-right: 6px;"><div style="font-size: 12px; line-height: 16px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">https://support.atlassian.com/bitbucket-cloud/docs/parallel-step-options/</div></div></div></a></div></div>

比如下面会报错

```java
- stage: &deploy
        name: Deploy
        deployment: default
        parallel:
          steps:
            - step:
                name: Deploy store backend
                oidc: true
                caches:
                  - node
                script:
                  - npm i
                  - export CDK_DEPLOY_REGION=$AWS_REGION
                  - export ENVIRONMENT=$BITBUCKET_DEPLOYMENT_ENVIRONMENT
                  - export AWS_WEB_IDENTITY_TOKEN_FILE=$(pwd)/web-identity-token
                  - echo $BITBUCKET_STEP_OIDC_TOKEN > $(pwd)/web-identity-token
                  - npx cdk deploy test-$BITBUCKET_DEPLOYMENT_ENVIRONMENT-store-stack --require-approval never
            - step:
                name: Deploy store step function
                oidc: true
                caches:
                  - node
                script:
                  - npm i
                  - export CDK_DEPLOY_REGION=$AWS_REGION
                  - export ENVIRONMENT=$BITBUCKET_DEPLOYMENT_ENVIRONMENT
                  - export AWS_WEB_IDENTITY_TOKEN_FILE=$(pwd)/web-identity-token
                  - echo $BITBUCKET_STEP_OIDC_TOKEN > $(pwd)/web-identity-token
                  - npx cdk deploy test-$BITBUCKET_DEPLOYMENT_ENVIRONMENT-store-sfn-solana-stack --require-approval never
```

> **Limitations**

    - The maximum number of steps you can within a stage is:
    	- 10 steps for workspaces on the [Free plan](https://www.atlassian.com/software/bitbucket/pricing).
    	- 100 steps for workspaces on a [Standard or Premium plan](https://www.atlassian.com/software/bitbucket/pricing).
    - Parallel stages are not supported.
    - A stage can't include parallel steps.
    - A stage can't contain manually triggered steps, but you can configure a stage to be manually triggered.
    - A stage can't contain conditional steps, but you can configure a stage to be conditional.
    - Disabling artifact downloads inside a stage is not supported.
    - Steps can't override any property also set on a stage.
    - Partially completed deployment stages can't be continued if another change was subsequently deployed to the same environment.

# 传递文件变量

Bitbucket pipeline 可以通过`Repository variables`来传递变量，但是如果变量包含一些特殊字符比如换行符，bitbucket 就不能很好的处理，对于这种情况我们可以将变量用 base64 编码一下，在 pipeline 中再解码就可以解决这问题了。

```text
cat file.txt | base64

// in pipeline file
echo ${YOUR_ENV} | base64 -d > file.txt
```

# Condition 用法

Documentation

<div style="width: 100%; margin-top: 4px; margin-bottom: 4px;"><div style="display: flex; background:white;border-radius:5px"><a href="https://support.atlassian.com/bitbucket-cloud/docs/step-options/#Condition"target="_blank"rel="noopener noreferrer"style="display: flex; color: inherit; text-decoration: none; user-select: none; transition: background 20ms ease-in 0s; cursor: pointer; flex-grow: 1; min-width: 0px; flex-wrap: wrap-reverse; align-items: stretch; text-align: left; overflow: hidden; border: 1px solid rgba(55, 53, 47, 0.16); border-radius: 5px; position: relative; fill: inherit;"><div style="flex: 4 1 180px; padding: 12px 14px 14px; overflow: hidden; text-align: left;"><div style="font-size: 14px; line-height: 20px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; min-height: 24px; margin-bottom: 2px;">Step options | Bitbucket Cloud | Atlassian Support</div><div style="font-size: 12px; line-height: 16px; color: rgba(55, 53, 47, 0.65); height: 32px; overflow: hidden;">Options for defining pipeline steps, including the required script property</div><div style="display: flex; margin-top: 6px; height: 16px;"><img src="https://wac-cdn.atlassian.com/assets/img/favicons/atlassian/favicon.png"style="width: 16px; height: 16px; min-width: 16px; margin-right: 6px;"><div style="font-size: 12px; line-height: 16px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">https://support.atlassian.com/bitbucket-cloud/docs/step-options/#Condition</div></div></div></a></div></div>

```yaml
- step: &deployStepFunction
    name: Deploy store step function
    oidc: true
    caches:
      - node
    condition:
      changesets:
        includePaths:
          - "lib/sfn.ts"
          - "sfn/**"
          - "bin/**"
    script:
      - npm i
      - export CDK_DEPLOY_REGION=$AWS_REGION
      - export ENVIRONMENT=$BITBUCKET_DEPLOYMENT_ENVIRONMENT
      - export AWS_WEB_IDENTITY_TOKEN_FILE=$(pwd)/web-identity-token
      - echo $BITBUCKET_STEP_OIDC_TOKEN > $(pwd)/web-identity-token
      - npx cdk deploy test-$BITBUCKET_DEPLOYMENT_ENVIRONMENT-store-sfn-solana-stack --require-approval never
```

## 限制

- A step within the stage can't contain a condition. The condition must be defined on the stage.

例如，下面这段就会引发上述错误。

```yaml
image: node:18

pipelines:
  definitions:
    - stage: &deploy
        name: Deploy to Default
        deployment: default
        steps:
          - step:
              name: Install
              caches:
                - node
              script:
                - npm install
          - step:
              oidc: true
              name: Deploy Admin Stack
              condition:
                changesets:
                  includePaths:
                    - "lib/admin.ts"
              script:
                - export CDK_DEPLOY_REGION=$AWS_REGION
                - export ENVIRONMENT=$BITBUCKET_DEPLOYMENT_ENVIRONMENT
                - export AWS_WEB_IDENTITY_TOKEN_FILE=$(pwd)/web-identity-token
                - echo $BITBUCKET_STEP_OIDC_TOKEN > $(pwd)/web-identity-token
                - npx cdk deploy "project-${ENVIRONMENT}-admin-stack" --require-approval never
          - step:
              oidc: true
              name: Deploy Backend Stack
              condition:
                changesets:
                  includePaths:
                    - "lib/backend.ts"
              script:
                - export CDK_DEPLOY_REGION=$AWS_REGION
                - export ENVIRONMENT=$BITBUCKET_DEPLOYMENT_ENVIRONMENT
                - export AWS_WEB_IDENTITY_TOKEN_FILE=$(pwd)/web-identity-token
                - echo $BITBUCKET_STEP_OIDC_TOKEN > $(pwd)/web-identity-token
                - npx cdk deploy "project-${ENVIRONMENT}-backend-stack" --require-approval never

  branches:
    master:
      - stage:
          <<: *deploy
          name: Deploy to demo
          deployment: demo
    local:
      - stage:
          <<: *deploy
          name: Deploy to charles
          deployment: charles
```

# Parallel

<div style="width: 100%; margin-top: 4px; margin-bottom: 4px;"><div style="display: flex; background:white;border-radius:5px"><a href="https://support.atlassian.com/bitbucket-cloud/docs/parallel-step-options/#Parallel"target="_blank"rel="noopener noreferrer"style="display: flex; color: inherit; text-decoration: none; user-select: none; transition: background 20ms ease-in 0s; cursor: pointer; flex-grow: 1; min-width: 0px; flex-wrap: wrap-reverse; align-items: stretch; text-align: left; overflow: hidden; border: 1px solid rgba(55, 53, 47, 0.16); border-radius: 5px; position: relative; fill: inherit;"><div style="flex: 4 1 180px; padding: 12px 14px 14px; overflow: hidden; text-align: left;"><div style="font-size: 14px; line-height: 20px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; min-height: 24px; margin-bottom: 2px;">Parallel step options | Bitbucket Cloud | Atlassian Support</div><div style="font-size: 12px; line-height: 16px; color: rgba(55, 53, 47, 0.65); height: 32px; overflow: hidden;">Run multiple pipeline steps concurrently or in parallel to reduce build time</div><div style="display: flex; margin-top: 6px; height: 16px;"><img src="https://wac-cdn.atlassian.com/assets/img/favicons/atlassian/favicon.png"style="width: 16px; height: 16px; min-width: 16px; margin-right: 6px;"><div style="font-size: 12px; line-height: 16px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">https://support.atlassian.com/bitbucket-cloud/docs/parallel-step-options/#Parallel</div></div></div></a></div></div>

# 内存问题

<div style="width: 100%; margin-top: 4px; margin-bottom: 4px;"><div style="display: flex; background:white;border-radius:5px"><a href="https://support.atlassian.com/bitbucket-cloud/docs/databases-and-service-containers/"target="_blank"rel="noopener noreferrer"style="display: flex; color: inherit; text-decoration: none; user-select: none; transition: background 20ms ease-in 0s; cursor: pointer; flex-grow: 1; min-width: 0px; flex-wrap: wrap-reverse; align-items: stretch; text-align: left; overflow: hidden; border: 1px solid rgba(55, 53, 47, 0.16); border-radius: 5px; position: relative; fill: inherit;"><div style="flex: 4 1 180px; padding: 12px 14px 14px; overflow: hidden; text-align: left;"><div style="font-size: 14px; line-height: 20px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; min-height: 24px; margin-bottom: 2px;">Databases and service containers | Bitbucket Cloud | Atlassian Support</div><div style="font-size: 12px; line-height: 16px; color: rgba(55, 53, 47, 0.65); height: 32px; overflow: hidden;">Bitbucket Pipelines allows you to run multiple Docker containers from your build pipeline.</div><div style="display: flex; margin-top: 6px; height: 16px;"><img src="https://wac-cdn.atlassian.com/assets/img/favicons/atlassian/favicon.png"style="width: 16px; height: 16px; min-width: 16px; margin-right: 6px;"><div style="font-size: 12px; line-height: 16px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">https://support.atlassian.com/bitbucket-cloud/docs/databases-and-service-containers/</div></div></div></a></div></div>

`pip` 和 `step` 共享内存

```sql
default:
   - step:
       services:
         - redis
         - mysql
         - docker
       script:
         - echo "This step is only allowed to consume 2048 MB of memory"
         - echo "Services are consuming the rest. docker 512 MB, redis 512 MB, mysql 1024 MB"
definitions:
  services:
    redis:
      image: redis:3.2
      memory: 512
    docker:
      memory: 512  # reduce memory for docker-in-docker from 1GB to 512MB
    mysql:
      image: mysql:5.7
      # memory: 1024  # default value
      variables:
        MYSQL_DATABASE: my-db
        MYSQL_ROOT_PASSWORD: $password
```

# OIDC Connect

<div style="width: 100%; margin-top: 4px; margin-bottom: 4px;"><div style="display: flex; background:white;border-radius:5px"><a href="https://support.atlassian.com/bitbucket-cloud/docs/deploy-on-aws-using-bitbucket-pipelines-openid-connect/"target="_blank"rel="noopener noreferrer"style="display: flex; color: inherit; text-decoration: none; user-select: none; transition: background 20ms ease-in 0s; cursor: pointer; flex-grow: 1; min-width: 0px; flex-wrap: wrap-reverse; align-items: stretch; text-align: left; overflow: hidden; border: 1px solid rgba(55, 53, 47, 0.16); border-radius: 5px; position: relative; fill: inherit;"><div style="flex: 4 1 180px; padding: 12px 14px 14px; overflow: hidden; text-align: left;"><div style="font-size: 14px; line-height: 20px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; min-height: 24px; margin-bottom: 2px;">Deploy on AWS using Bitbucket Pipelines OpenID Connect | Bitbucket Cloud | Atlassian Support</div><div style="font-size: 12px; line-height: 16px; color: rgba(55, 53, 47, 0.65); height: 32px; overflow: hidden;">Use Bitbucket Pipelines OpenID Connect to deploy your builds on AWS</div><div style="display: flex; margin-top: 6px; height: 16px;"><img src="https://wac-cdn.atlassian.com/assets/img/favicons/atlassian/favicon.png"style="width: 16px; height: 16px; min-width: 16px; margin-right: 6px;"><div style="font-size: 12px; line-height: 16px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">https://support.atlassian.com/bitbucket-cloud/docs/deploy-on-aws-using-bitbucket-pipelines-openid-connect/</div></div></div></a></div></div>

<div style="width: 100%; margin-top: 4px; margin-bottom: 4px;"><div style="display: flex; background:white;border-radius:5px"><a href="https://support.atlassian.com/bitbucket-cloud/docs/integrate-pipelines-with-resource-servers-using-oidc/"target="_blank"rel="noopener noreferrer"style="display: flex; color: inherit; text-decoration: none; user-select: none; transition: background 20ms ease-in 0s; cursor: pointer; flex-grow: 1; min-width: 0px; flex-wrap: wrap-reverse; align-items: stretch; text-align: left; overflow: hidden; border: 1px solid rgba(55, 53, 47, 0.16); border-radius: 5px; position: relative; fill: inherit;"><div style="flex: 4 1 180px; padding: 12px 14px 14px; overflow: hidden; text-align: left;"><div style="font-size: 14px; line-height: 20px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; min-height: 24px; margin-bottom: 2px;">Integrate Pipelines with resource servers using OIDC | Bitbucket Cloud | Atlassian Support</div><div style="font-size: 12px; line-height: 16px; color: rgba(55, 53, 47, 0.65); height: 32px; overflow: hidden;">Use Bitbucket Pipelines OpenID Connect Provider (OIDC IDP) to allow your pipelines to access your resource server</div><div style="display: flex; margin-top: 6px; height: 16px;"><img src="https://wac-cdn.atlassian.com/assets/img/favicons/atlassian/favicon.png"style="width: 16px; height: 16px; min-width: 16px; margin-right: 6px;"><div style="font-size: 12px; line-height: 16px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">https://support.atlassian.com/bitbucket-cloud/docs/integrate-pipelines-with-resource-servers-using-oidc/</div></div></div></a></div></div>

# Artifact

不支持环境变量名 , 比如:

```javascript
        script:
          - apt-get update && apt-get install -y zip
          - python3 -m pip install --upgrade pip
          - pip install poetry
          - poetry export --only $FUNCTION -f requirements.txt -o ./$FUNCTION/requirements.txt --without-hashes --without-urls
          - cd $FUNCTION
          - zip -r $FUNCTION.zip .

        artifacts:
          - $FUNCTION/$FUNCTION.zip
```

##

<div style="width: 100%; margin-top: 4px; margin-bottom: 4px;"><div style="display: flex; background:white;border-radius:5px"><a href="https://support.atlassian.com/bitbucket-cloud/docs/use-artifacts-in-steps/"target="_blank"rel="noopener noreferrer"style="display: flex; color: inherit; text-decoration: none; user-select: none; transition: background 20ms ease-in 0s; cursor: pointer; flex-grow: 1; min-width: 0px; flex-wrap: wrap-reverse; align-items: stretch; text-align: left; overflow: hidden; border: 1px solid rgba(55, 53, 47, 0.16); border-radius: 5px; position: relative; fill: inherit;"><div style="flex: 4 1 180px; padding: 12px 14px 14px; overflow: hidden; text-align: left;"><div style="font-size: 14px; line-height: 20px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; min-height: 24px; margin-bottom: 2px;">Pipeline artifacts | Bitbucket Cloud | Atlassian Support</div><div style="font-size: 12px; line-height: 16px; color: rgba(55, 53, 47, 0.65); height: 32px; overflow: hidden;">Artifacts are files that are produced by a step. Once you've defined them in your Bitbucket Pipeline configuration, you can share or export them.</div><div style="display: flex; margin-top: 6px; height: 16px;"><img src="https://wac-cdn.atlassian.com/assets/img/favicons/atlassian/favicon.png"style="width: 16px; height: 16px; min-width: 16px; margin-right: 6px;"><div style="font-size: 12px; line-height: 16px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">https://support.atlassian.com/bitbucket-cloud/docs/use-artifacts-in-steps/</div></div></div></a></div></div>

# Multiple Lines Script

```sql
pipelines:
  definitions:
    - step: &deploy
        oidc: true
        caches:
          - node
        name: CDK Deploy
        script:
          - npm install
          - export AWS_ROLE_ARN=$AWS_ROLE_ARN
          - export AWS_WEB_IDENTITY_TOKEN_FILE=$(pwd)/web-identity-token
          - echo $BITBUCKET_STEP_OIDC_TOKEN > $(pwd)/web-identity-token
          - >
            if [ -z "$STACK_NAME" ]; then
              echo "STACK_NAME is not set";
              npx cdk deploy --all --require-approval never
            else
              echo "STACK_NAME is set to '$STACK_NAME'";
              npx cdk deploy $STACK_NAME --require-approval never
            fi
```
