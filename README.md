# project-workflows

GitHub Projectの運用を自動化するための再利用可能なワークフロー（Reusable Workflows）を提供するリポジトリです。

## 提供するワークフロー一覧

| ワークフロー | 目的 |
| --- | --- |
| [auto-assign-pr-creator.yml](.github/workflows/auto-assign-pr-creator.yml) | PR作成者を自動でAssigneeに設定 |
| [auto-label-subsystem.yml](.github/workflows/auto-label-subsystem.yml) | リポジトリTopicからサブシステムラベルを自動付与 |
| [auto-label-tag.yml](.github/workflows/auto-label-tag.yml) | リポジトリTopicからTAGラベルを自動付与 |
| [set-iteration-on-close.yml](.github/workflows/set-iteration-on-close.yml) | Issue/PR Close時に現在のIterationを自動設定 |
| [update-tracking-status.yml](.github/workflows/update-tracking-status.yml) | Tracking Statusを手動一括更新 |
| [update-renovate-tracking.yml](.github/workflows/update-renovate-tracking.yml) | Renovate PRのTracking Statusを定時/手動更新 |

## PR作成者の自動Assignee付与 (`auto-assign-pr-creator.yml`)

**目的**: PRが作成されたとき、そのPRの作成者を自動でAssigneeに設定します。

**動作の流れ**:

1. PRが作成される
2. PRの作成者を取得する
3. 作成者が既にAssigneeに含まれていないことを確認する
4. 作成者をAssigneeとして追加する

以下の内容で `.github/workflows/assign-pr-creator.yaml` を作成するだけで導入できます。

```yaml
name: Auto Assign PR Creator

on:
  pull_request:
    types: [opened]

jobs:
  assign:
    uses: ut-issl/github-project-workflows/.github/workflows/auto-assign-pr-creator.yml@main
```

> [!NOTE]
> この自動化は `GITHUB_TOKEN` のみで動作し、追加のSecrets設定は不要です。

## サブシステムラベル自動付与 (`auto-label-subsystem.yml`)

**目的**: IssueやPRが作成されたとき、リポジトリに設定されている **Topic** をもとに、対応するサブシステムラベル（`sys::CDH` など）を自動で付与します。

**動作の流れ**:

1. IssueまたはPRが作成される
2. リポジトリのTopicを取得する
3. Topic名からサフィックスを抽出し、マッピングテーブルに基づいてサブシステムラベルを特定する
4. 該当するラベルをIssue/PRに付与する

**Topicとラベルのマッピング**:

| Topicサフィックス | ラベル | 例 |
| --- | --- | --- |
| `cdh` | `sys::CDH` | `geox-cdh` |
| `thermal` | `sys::熱` | `geox-thermal` |
| `structure` | `sys::構造` | `geox-structure` |
| `comm` | `sys::通信` | `geox-comm` |
| `aocs` | `sys::姿勢` | `geox-aocs` |
| `power` | `sys::電源` | `geox-power` |
| `orbit` | `sys::軌道` | `geox-orbit` |
| `propulsion` | `sys::推進` | `geox-propulsion` |
| `ground` | `sys::地上` | `geox-ground` |
| `payload` | `sys::PL` | `geox-payload` |
| `elec` | `sys::電装` | `geox-elec` |
| `system` | `sys::システム` | `geox-system` |
| `sw-team`（完全一致） | `sys::SW` | `sw-team` |
| `se-team`（完全一致） | `sys::SE` | `se-team` |

以下の内容で `.github/workflows/auto-label-subsystem.yaml` を作成するだけで導入できます。

```yaml
name: Auto Label Subsystem

on:
  issues:
    types: [opened]
  pull_request:
    types: [opened]

jobs:
  auto-label:
    uses: ut-issl/github-project-workflows/.github/workflows/auto-label-subsystem.yml@main
```

> [!NOTE]
> この自動化は `GITHUB_TOKEN` のみで動作し、追加のSecrets設定は不要です。
> リポジトリのTopicは GitHub の Settings > General > Topics から設定できます。
> Topic名は
> [issl-autoproject の rules.yaml](https://github.com/ut-issl/issl-autoproject/blob/main/rules.yaml)
> と同じ命名規則に従ってください。

## TAGラベル自動付与 (`auto-label-tag.yml`)

**目的**: IssueやPRが作成されたとき、リポジトリに設定されている **Topic** をもとに、対応するTAGラベル（`tag::autonomy` など）を自動で付与します。

**動作の流れ**:

1. IssueまたはPRが作成される
2. リポジトリのTopicを取得する
3. `*-tag` パターンのTopicをマッピングテーブルに基づいてTAGラベルに変換する
4. 該当するラベルをIssue/PRに付与する

**Topicとラベルのマッピング**:

| Topic | ラベル |
| --- | --- |
| `autonomy-tag` | `tag::autonomy` |
| `formal-methods-tag` | `tag::formal-methods` |
| `inference-tag` | `tag::inference` |

以下の内容で `.github/workflows/auto-label-tag.yaml` を作成するだけで導入できます。

```yaml
name: Auto Label TAG

on:
  issues:
    types: [opened]
  pull_request:
    types: [opened]

jobs:
  auto-label:
    uses: ut-issl/github-project-workflows/.github/workflows/auto-label-tag.yml@main
```

> [!NOTE]
> この自動化は `GITHUB_TOKEN` のみで動作し、追加のSecrets設定は不要です。
> リポジトリのTopicは GitHub の Settings > General > Topics から
> `{tag名}-tag`（例: `autonomy-tag`）の形式で設定してください。

## Iteration自動付与 (`set-iteration-on-close.yml`)

**目的**: IssueやPRがCloseされたとき、そのIssue/PRが紐付いているGitHub Projectに「現在のIteration」を自動で設定します。

**動作の流れ**:

1. IssueまたはPRがCloseされる
2. GitHub Actionsが自動で起動する
3. そのIssue/PRが紐付いている全てのGitHub Projectを確認する
4. Iterationフィールドを持つProjectに対して、現在の日付に対応するIterationを自動設定する
5. Iterationフィールドがない、または現在のIterationが見つからないProjectはスキップされる

以下の内容で `.github/workflows/close-set-iteration.yaml` を作成するだけで導入できます。

```yaml
name: Set Iteration on Close

on:
  issues:
    types: [closed]
  pull_request:
    types: [closed]

jobs:
  set-iteration:
    uses: ut-issl/github-project-workflows/.github/workflows/set-iteration-on-close.yml@main
    secrets:
      ITERATION_AUTOMATION_APP_ID: ${{ secrets.ITERATION_AUTOMATION_APP_ID }}
      ITERATION_AUTOMATION_APP_PRIVATE_KEY: ${{ secrets.ITERATION_AUTOMATION_APP_PRIVATE_KEY }}
```

> [!NOTE]
> この自動化は、Organization Secretsに登録されたGitHub App
> （`ITERATION_AUTOMATION_APP_ID`, `ITERATION_AUTOMATION_APP_PRIVATE_KEY`）
> の認証情報を利用しています。
> 新たにリポジトリを追加する場合は、Organization Secretsの
> 「Repository access」に対象リポジトリを追加する必要があります。

## Tracking Status自動更新 (`update-tracking-status.yml`)

**目的**: GitHub Projectの「Tracking」フィールドを、アイテムの状態に応じて自動更新します。MTG終了時などに手動実行することで、ステータスの更新漏れを防ぎます。

**動作の流れ**:

1. リポジトリの Actions タブから「Update Tracking Status」ワークフローを手動実行する
2. Projectの「Status」フィールドと「Tracking」フィールドの値を基に、更新が必要なアイテムのみをクエリで取得する
3. 以下のルールに基づいてTrackingフィールドを更新する:
   - **Statusが「Done」のアイテム**: Trackingが「Closed」以外 → 「Closed」に更新
   - **Statusが「Done」以外のアイテム**: Trackingが「Needs Review」 → 「Tracked」に更新
   - **Statusが「Done」以外のアイテム**: Trackingが未設定 → 「Tracked」に更新
4. 更新件数と結果がログに出力される

**入力パラメータ**:

| パラメータ | 説明 | デフォルト |
| --- | --- | --- |
| `project-number` | 対象のGitHub Project番号 | （必須） |
| `dry-run` | 有効にすると変更を適用せず、更新対象のログ出力のみ行う | `false` |

以下の内容で `.github/workflows/tracking-update.yaml` を作成するだけで導入できます。

```yaml
name: Update Tracking Status

on:
  workflow_dispatch:
    inputs:
      project-number:
        description: "GitHub Project number to update"
        required: true
        type: number
      dry-run:
        description: "Dry run (log only, no changes)"
        required: false
        type: boolean
        default: false

jobs:
  update-tracking:
    uses: ut-issl/github-project-workflows/.github/workflows/update-tracking-status.yml@main
    with:
      project-number: ${{ inputs.project-number }}
      dry-run: ${{ inputs.dry-run }}
    secrets:
      ITERATION_AUTOMATION_APP_ID: ${{ secrets.ITERATION_AUTOMATION_APP_ID }}
      ITERATION_AUTOMATION_APP_PRIVATE_KEY: ${{ secrets.ITERATION_AUTOMATION_APP_PRIVATE_KEY }}
```

> [!NOTE]
> この自動化は、Organization Secretsに登録されたGitHub App
> （`ITERATION_AUTOMATION_APP_ID`, `ITERATION_AUTOMATION_APP_PRIVATE_KEY`）
> の認証情報を利用しています。

## Renovate PR Tracking自動更新 (`update-renovate-tracking.yml`)

**目的**: GitHub Project内のRenovate bot PRに対して、「Tracking」フィールドをStatusに応じて自動更新します。毎日定時に自動実行されるほか、手動でも実行できます。

**動作の流れ**:

1. 毎日 0:00 UTC（9:00 JST）に自動実行される（手動実行も可能）
2. Project内の全アイテムを取得し、Renovate bot（`renovate[bot]`）が作成したPRを抽出する
3. 以下のルールに基づいてTrackingフィールドを更新する:
   - **Statusが「Done」のPR**: Tracking → 「Closed」
   - **Statusが「Done」以外のPR**: Tracking → 「Tracked」
4. 既に正しい値が設定されているPRはスキップされる

**入力パラメータ**:

| パラメータ | 説明 | デフォルト |
| --- | --- | --- |
| `project-number` | 対象のGitHub Project番号 | （必須） |
| `dry-run` | 有効にすると変更を適用せず、更新対象のログ出力のみ行う | `false` |

以下の内容で `.github/workflows/renovate-tracking-update.yaml` を作成するだけで導入できます。

```yaml
name: Update Renovate PR Tracking

on:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:
    inputs:
      project-number:
        description: "GitHub Project number to update"
        required: false
        type: number
        default: "YOUR_PROJECT_NUMBER"
      dry-run:
        description: "Dry run (log only, no changes)"
        required: false
        type: boolean
        default: false

jobs:
  update-renovate-tracking:
    uses: ut-issl/github-project-workflows/.github/workflows/update-renovate-tracking.yml@main
    with:
      project-number: ${{ inputs.project-number || 'YOUR_PROJECT_NUMBER' }}
      dry-run: ${{ inputs.dry-run || false }}
    secrets:
      ITERATION_AUTOMATION_APP_ID: ${{ secrets.ITERATION_AUTOMATION_APP_ID }}
      ITERATION_AUTOMATION_APP_PRIVATE_KEY: ${{ secrets.ITERATION_AUTOMATION_APP_PRIVATE_KEY }}
```

> [!NOTE]
> この自動化は、Organization Secretsに登録されたGitHub App
> （`ITERATION_AUTOMATION_APP_ID`, `ITERATION_AUTOMATION_APP_PRIVATE_KEY`）
> の認証情報を利用しています。
