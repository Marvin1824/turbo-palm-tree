---
title: 通过 GitHub Actions 自动化 Dependabot
intro: '如何使用 {% data variables.product.prodname_actions %} 来自动执行常见 {% data variables.product.prodname_dependabot %} 相关任务的示例。'
permissions: 'People with write permissions to a repository can configure {% data variables.product.prodname_actions %} to respond to {% data variables.product.prodname_dependabot %}-created pull requests.'
miniTocMaxHeadingLevel: 3
versions:
  fpt: '*'
  ghec: '*'
  ghes: '>3.2'
type: how_to
topics:
  - Actions
  - Dependabot
  - Version updates
  - Security updates
  - Repositories
  - Dependencies
  - Pull requests
shortTitle: Use Dependabot with actions
---

{% data reusables.dependabot.beta-security-and-version-updates %}
{% data reusables.dependabot.enterprise-enable-dependabot %}

## 关于 {% data variables.product.prodname_dependabot %} 与 {% data variables.product.prodname_actions %}

{% data variables.product.prodname_dependabot %} 创建拉动请求以保持依赖项的最新状态，并且当创建这些拉取请求时，您可以使用 {% data variables.product.prodname_actions %} 执行自动任务。 例如，获取其他构件、添加标签、运行测试或修改拉取请求。

## 响应事件

{% data variables.product.prodname_dependabot %} 能够在其拉取请求和评论上触发 {% data variables.product.prodname_actions %} 工作流程；然而，由于 ["GitHub Action：Depabot PR 触发的工作流程将使用只读权限运行"](https://github.blog/changelog/2021-02-19-github-actions-workflows-triggered-by-dependabot-prs-will-run-with-read-only-permissions/)，某些事件的处理是不同的。

对于 {% data variables.product.prodname_dependabot %} (`github.actor == "dependabot[bot]"`) 使用 `pull_request`、`pull_request_review`、`pull_request_review_comment` 和 `push` 事件发起的工作流程，适用以下限制：

- `GITHUB_TOKEN` 具有只读权限。
- 密码不可访问。

更多信息请参阅“[保持 GitHub Actions 和工作流程安全：阻止 pwn 请求](https://securitylab.github.com/research/github-actions-preventing-pwn-requests/)”。

{% ifversion ghes > 3.2 %}
{% note %}

**Note:** Your site administrator can override these restrictions for {% data variables.product.product_location %}. For more information, see "[Troubleshooting {% data variables.product.prodname_actions %} for your enterprise](/admin/github-actions/advanced-configuration-and-troubleshooting/troubleshooting-github-actions-for-your-enterprise#troubleshooting-failures-when-dependabot-triggers-existing-workflows)."

If the restrictions are removed, when a workflow is triggered by {% data variables.product.prodname_dependabot %} it will have access to any secrets that are normally available. In addition, workflows triggered by {% data variables.product.prodname_dependabot %} can use the `permissions` term to increase the default scope of the `GITHUB_TOKEN` from read-only access.

{% endnote %}
{% endif %}

### 处理 `pull_request` 事件

如果您的工作流程需要访问具有写入权限的密码或 `GITHUB_TOKEN`，则有两个选项：使用 `pull_request_target` 或使用两个单独的工作流程。 我们将在本节中详述使用 `pull_request_target`，以及在 [处理 `push` 事件](#handling-push-events)“中使用下面两个工作流程。

下面是现在可能失败的 `pull_request` 工作流程的简单示例：

{% raw %}
```yaml
### This workflow now has no secrets and a read-only token
name: Dependabot Workflow
on:
  pull_request

jobs:
  dependabot:
    runs-on: ubuntu-latest
    # Always check the actor is Dependabot to prevent your workflow from failing on non-Dependabot PRs
    if: ${{ github.actor == 'dependabot[bot]' }}
    steps:
      - uses: actions/checkout@v2
```
{% endraw %}

您可以将 `pull_request` 替换为 `pull_request_target`，这用于复刻中的拉取请求，并明确检出拉取请求 `HEAD`。

{% warning %}

**警告：**使用 `pull_request_target` 替代 `pull_request` 会使您暴露在不安全的行为中。 我们建议您使用两种工作流程方法，如下面的“[处理 `push` 事件](#handling-push-events)”中所述。

{% endwarning %}

{% raw %}
```yaml
### This workflow has access to secrets and a read-write token
name: Dependabot Workflow
on:
  pull_request_target

permissions:
  # Downscope as necessary, since you now have a read-write token

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: ${{ github.actor == 'dependabot[bot]' }}
    steps:
      - uses: actions/checkout@v2
        with:
          # Check out the pull request HEAD
          ref: ${{ github.event.pull_request.head.sha }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```
{% endraw %}

还强烈建议您缩小授予 `GITHUB_TOKEN` 的权限范围，以避免泄露具有不必要特权的令牌。 更多信息请参阅“[`GITHUB_TOKEN`](/actions/reference/authentication-in-a-workflow#permissions-for-the-github_token) 的权限”。

### 处理 `push` 事件

因为没有等效于 `push` 事件的 `pull_request_target_` ，因此您必须使用两个工作流程：一个通过上传构件而结束不可信工作流程， 触发第二个下载构件并继续处理的可信任工作流程。

第一个工作流程执行任何不信任的工作：

{% raw %}
```yaml
### This workflow doesn't have access to secrets and has a read-only token
name: Dependabot Untrusted Workflow
on:
  push

jobs:
  check-dependabot:
    runs-on: ubuntu-latest
    if: ${{ github.actor == 'dependabot[bot]' }}
    steps:
      - uses: ...
```
{% endraw %}

第二个工作流程在第一个工作流程成功完成后执行受信任的工作：

{% raw %}
```yaml
### This workflow has access to secrets and a read-write token
name: Dependabot Trusted Workflow
on:
  workflow_run:
    workflows: ["Dependabot Untrusted Workflow"]
    types:
      - completed

permissions:
  # Downscope as necessary, since you now have a read-write token

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    steps:
      - uses: ...
```
{% endraw %}

### 手动重新运行工作流程

您还可以手动重新运行失败的 Dependabot 工作流程，它将以读写令牌运行并访问密码。 在手动重新运行失败的工作流程之前，您应始终检查更新的依赖项，以确保更改不会引入任何恶意或意外行为。

## 常用 Dependabot 自动化

以下是可以使用 {% data variables.product.prodname_actions %} 自动化的几个常见场景。

### 获取有关拉取请求的元数据

大量自动化需要了解拉取请求内容的信息：依赖项名称是什么，是否为生产依赖项，以及是否为主要、次要或补丁更新。

`dependabot/fetch-metadata` 操作为您提供所有这些信息：

{% raw %}
```yaml
name: Dependabot auto-label
on: pull_request_target

permissions:
  pull-requests: write
  issues: write
  repository-projects: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: ${{ github.actor == 'dependabot[bot]' }}
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@v1.1.1
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"
      # The following properties are now available:
      #  - steps.metadata.outputs.dependency-names
      #  - steps.metadata.outputs.dependency-type
      #  - steps.metadata.outputs.update-type      
```
{% endraw %}

更多信息请参阅 [`dependabot/fetch-metadata`](https://github.com/dependabot/fetch-metadata) 仓库。

### 标记拉取请求

如果您有基于 {% data variables.product.prodname_dotcom %} 标签的其他自动化或分类工作流程，则可以配置操作以根据提供的元数据分配标签。

例如，如果您想用标签标记所有生产依赖项更新：

{% raw %}
```yaml
name: Dependabot auto-label
on: pull_request_target

permissions:
  pull-requests: write
  issues: write
  repository-projects: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: ${{ github.actor == 'dependabot[bot]' }}
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@v1.1.1
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"
      - name: Add a label for all production dependencies
        if: ${{ steps.metadata.outputs.dependency-type == 'direct:production' }}
        run: gh pr edit "$PR_URL" --add-label "production"
        env:
          PR_URL: ${{github.event.pull_request.html_url}}
```
{% endraw %}

### 批准拉取请求

如果您想要自动批准 Dependabot 拉取请求，您可以在工作流程中使用 {% data variables.product.prodname_cli %}：

{% raw %}
```yaml
name: Dependabot auto-approve
on: pull_request_target

permissions:
  pull-requests: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: ${{ github.actor == 'dependabot[bot]' }}
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@v1.1.1
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"
      - name: Approve a PR
        run: gh pr review --approve "$PR_URL"
        env:
          PR_URL: ${{github.event.pull_request.html_url}}
          GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
```
{% endraw %}

### 在拉取请求上启用自动合并

如果您要自动合并拉取请求，可以使用 {% data variables.product.prodname_dotcom %} 的自动合并功能。 这样，当所有所需的测试和批准都成功满足时，拉取请求即可合并。 For more information on auto-merge, see "[Automatically merging a pull request"](/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/automatically-merging-a-pull-request)."

这是为所有补丁更新启用自动合并到 `my-dependency` 的示例：

{% raw %}
```yaml
name: Dependabot auto-merge
on: pull_request_target

permissions:
  pull-requests: write
  contents: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: ${{ github.actor == 'dependabot[bot]' }}
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@v1.1.1
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"
      - name: Enable auto-merge for Dependabot PRs
        if: ${{contains(steps.metadata.outputs.dependency-names, 'my-dependency') && steps.metadata.outputs.update-type == 'version-update:semver-patch'}}
        run: gh pr merge --auto --merge "$PR_URL"
        env:
          PR_URL: ${{github.event.pull_request.html_url}}
          GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
```
{% endraw %}

## 失败的工作流程运行故障排除

如果您的工作流程运行失败，请检查以下情况：

- 只有当正确的角色触发工作流程时，才运行工作流程。
- 您在为 `pull_request` 检出正确的 `ref`。
- 您没有尝试从 Dependabot 触发的 `pull_request`、`pull_request_review`、`pull_request_review_comment` 或 `push` 事件访问密码。
- 您没有尝试从 Dependabot 触发的 `pull_request`、`pull_request_review`、`pull_request_review_comment` 或 `push` 事件执行任何 `write` 操作。

有关编写和调试 {% data variables.product.prodname_actions %} 的信息，请参阅“[了解 GitHub Actions](/actions/learn-github-actions)”。