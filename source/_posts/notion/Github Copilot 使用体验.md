---
created: 2024-09-04 03:32:00
categories:
  - linux
  - jvm
  - life
tags:
  - linux
updated: 2024-09-04 07:12:00
date: 2024-09-04 00:00:00
title: Github Copilot 使用体验
id: 80e9f565-cce1-4023-b07a-50fed4f8ca0b
---

使用 GitHub Copilot 已经一年多了，来分享一下我的使用体验：

首先说优点：

## **代码优化助手**

有时你觉得一段代码写得有点啰嗦，但没有查阅相关文档，你并不知道是否有更好的写法。这时 Copilot 就能提供一些更简洁的写法。

例如，我之前写 GitHub Actions 时遇到过这样一个问题：下面的代码是部分 Actions 的代码，它的功能是 SSH 到 VPS 上执行一些固定的脚本。由于我有多台 VPS，每多一台 VPS 都要添加一段类似的代码，唯一的区别是 VPS 的端口号、IP 地址等变量。

原始代码：

```yaml
- name: Deploy to US
  uses: appleboy/ssh-action@v1.0.3
  env:
    REALITY_SERVER_NAME: "www.xxx.com"
  with:
    host: "1.1.1.1:2222"
    username: "root"
    key: ${{ secrets.SSH_VPS }}
    envs: REALITY_SERVER_NAME
    script: |
      scripts...

- name: Deploy to JP
  if: ${{ success() }}
  uses: appleboy/ssh-action@v1.0.3
  env:
    REALITY_SERVER_NAME: "www.xxx.com"
  with:
    host: "2.2.2.2:444"
    username: "root"
    key: ${{ secrets.SSH_VPS }}
    envs: REALITY_SERVER_NAME
    script: |
      scripts...
```

我中途研究了一下 `appleboy/ssh-action` 的用法，想看看它是否支持多台服务器，但最终发现它不符合我的需求。我还查了一下 GitHub Actions 目前是否支持 YAML 锚点。最后我问了 Copilot：“Is this github action code a bit redundant, is there room for optimization?”

Copilot 随后给出了修改后的代码：

```yaml
runs-on: ubuntu-latest
strategy:
  matrix:
    server:
      - {
          name: "server1",
          host: "1.1.1.1:2222",
          username: "root",
          reality_server_name: "www.xxx.com",
        }
      - {
          name: "server2",
          host: "2.2.2.2:4444",
          username: "root",
          reality_server_name: "www.xxx.com",
        }

steps:
  - name: Deploy to ${{ matrix.server.name }}
    uses: appleboy/ssh-action@v1.0.3
    env:
      REALITY_SERVER_NAME: ${{ matrix.server.reality_server_name }}
    with:
      host: ${{ matrix.server.host }}
      username: ${{ matrix.server.username }}
      key: ${{ secrets.SSH_VPS }}
      envs: REALITY_SERVER_NAME
      script: |
        scripts...
```

如果没有 copilot 或者你没有看过一遍 github actions 的所有文档的话，你是不知道还有 matrix 这种用法的。

## Demo 代码生成

有时候想用不熟悉的语言写个 Demo 程序，或者对框架和库不太熟悉的时候，Copilot 可以帮你快速写出示例代码。

## 冗余代码生成

copilot 特别适合生成一些 verbose code，特别是在写增删改查代码的时候，比如写一些 controller，生成一些 entity 代码。

接下来说一下缺点：

## 生成的代码可能包含一些不明显的 bug

我认为这是 Copilot 最大的缺点：它有时无法完全理解你的意图，生成的代码会包含一些不易察觉的 bug。比如以下代码（Kotlin，已简化）：

```kotlin
val a = 1;
val b = 2;

function add(x:Int, y:Int): Int {
   return x + y;
}
```

你本来意图是些下面这段代码：

```kotlin
val c = add (a, b)
```

但实际上 copilot 生成的这段代码

```kotlin
val c = add (a, a)
```

生成的代码会通过编译，如果你不去 review 一下你是很难发现这个 bug 的。

又或者下面这段代码：

```kotlin
val map = mutablMapOf<String,String>();

map.put("id" , a)
map.put("name" , b) -- 本来你想写的代码
map.put("username", b) -- copilot 生成的代码
```

本来要写的是 `name` 但是 copilot 生成的是 `username` ，这种 bug 也比较难发现。

## 数据库更新不及时

有时你想让 Copilot 生成某个框架的代码示例，但它只能给出旧版本的代码示例，因为 Copilot 的数据还没有包含最新版本的框架数据。

## 幻觉

这属于 LLM 的功能特性，而非 bug。例如，它生成的代码可能会调用一些不存在的方法，或者列出一些不存在的库。

## 插件支持

感觉 VS Code 上的 Copilot 比 JetBrains 上的好用，总觉得 JetBrains 上的 Copilot 有点笨。

总的来说，Copilot 有明显的优缺点。在使用 Copilot 生成的代码时，一定要仔细检查。如果使用得当，感觉可以提高不少效率。
