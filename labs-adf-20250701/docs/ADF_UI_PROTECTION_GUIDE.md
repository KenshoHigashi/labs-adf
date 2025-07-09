# Azure Data Factory UI 発行による上書き防止ガイド

## 問題

Azure Data Factory UI で発行すると、adf_publish ブランチが UI 側の設定で上書きされ、手動で管理している ARM テンプレートが失われる。

## 解決策

### 1. GitHub ブランチ保護設定（推奨）

#### 設定手順

1. GitHub リポジトリ → Settings → Branches
2. "Add rule" をクリック
3. Branch name pattern: `adf_publish`
4. 以下を有効化：
   - ☑ Restrict pushes that create files larger than 100MB
   - ☑ Require a pull request before merging
   - ☑ Require status checks to pass before merging
   - ☑ Require linear history
   - ☑ Include administrators

#### 効果

- ADF UI からの直接 push を防止
- 手動でのマージプロセスを強制
- 変更内容のレビューが可能

### 2. カスタム ARM テンプレート管理戦略

#### ディレクトリ構造

```
labs-adf/
├── custom-templates/           # 手動管理のARMテンプレート
│   ├── ARMTemplateForFactory.json
│   ├── ARMTemplateParametersForFactory.json
│   ├── ARMTemplateParametersForFactory.dev.json
│   ├── ARMTemplateParametersForFactory.stg.json
│   └── ARMTemplateParametersForFactory.prod.json
├── labs-adf-20250701/         # ADF UI生成（参照用のみ）
└── deployment/                # デプロイ用スクリプト
    ├── deploy.ps1
    └── deploy-env.ps1
```

#### デプロイスクリプト例

```powershell
# deploy-env.ps1
param(
    [Parameter(Mandatory=$true)]
    [ValidateSet("dev", "stg", "prod")]
    [string]$Environment,

    [Parameter(Mandatory=$true)]
    [string]$ResourceGroup
)

$templateFile = "custom-templates/ARMTemplateForFactory.json"
$parametersFile = "custom-templates/ARMTemplateParametersForFactory.$Environment.json"

az deployment group create `
    --resource-group $ResourceGroup `
    --template-file $templateFile `
    --parameters "@$parametersFile"
```

### 3. Git Hooks による自動保護

#### pre-receive hook 設定

```bash
#!/bin/bash
# .git/hooks/pre-receive

while read oldrev newrev refname; do
    if [[ $refname == "refs/heads/adf_publish" ]]; then
        echo "Error: Direct push to adf_publish branch is not allowed"
        echo "Please use pull request to merge changes"
        exit 1
    fi
done
```

### 4. CI/CD パイプライン統合

#### GitHub Actions 設定例

```yaml
# .github/workflows/adf-deploy.yml
name: ADF Deployment

on:
  push:
    branches: [main]
    paths: ["custom-templates/**"]

jobs:
  deploy-dev:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to Dev
        run: |
          az deployment group create \
            --resource-group "labs" \
            --template-file "custom-templates/ARMTemplateForFactory.json" \
            --parameters "@custom-templates/ARMTemplateParametersForFactory.dev.json"
```

### 5. 運用プロセス

#### 開発フロー

1. **開発**: ADF UI で開発・テスト
2. **エクスポート**: 完成したら ARM テンプレートをエクスポート
3. **カスタマイズ**: custom-templates ディレクトリで手動調整
4. **検証**: カスタムテンプレートでデプロイ検証
5. **マージ**: Pull Request で adf_publish ブランチに統合

#### 注意点

- ADF UI での発行は開発環境のみ使用
- 本番環境はカスタム ARM テンプレートのみでデプロイ
- 定期的に UI と ARM テンプレートの同期確認

## 情報ソース

- [Azure Data Factory CI/CD best practices](https://docs.microsoft.com/azure/data-factory/continuous-integration-deployment)
- [GitHub Branch Protection Rules](https://docs.github.com/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/about-protected-branches)
