# Azure Data Factory ARM テンプレート デプロイメントガイド

## 概要

このリポジトリには環境固有のデプロイメントに対応した Azure Data Factory (ADF)の ARM テンプレートが含まれています。

## テンプレート構造

### メインテンプレート

- `ARMTemplateForFactory.json` - メインの ARM テンプレート

### パラメータファイル

- `ARMTemplateParametersForFactory.json` - デフォルト/開発環境用
- `ARMTemplateParametersForFactory.dev.json` - 開発環境用
- `ARMTemplateParametersForFactory.stg.json` - ステージング環境用
- `ARMTemplateParametersForFactory.prod.json` - 本番環境用

## パラメータ説明

| パラメータ名                | 型     | 説明                                 | 例                                       |
| --------------------------- | ------ | ------------------------------------ | ---------------------------------------- |
| `factoryName`               | string | Data Factory の名前                  | `labs-adf-dev`                           |
| `managedIdentityResourceId` | string | 使用するマネージド ID のリソース ID  | `/subscriptions/.../labs_id_dev`         |
| `storageEndpoint`           | string | ストレージアカウントのエンドポイント | `https://storage.blob.core.windows.net/` |

## デプロイメント方法

### 1. パラメータファイルを使用したデプロイ

#### 開発環境

```bash
az deployment group create \
  --resource-group "labs" \
  --template-file "ARMTemplateForFactory.json" \
  --parameters "@ARMTemplateParametersForFactory.dev.json"
```

#### ステージング環境

```bash
az deployment group create \
  --resource-group "labs" \
  --template-file "ARMTemplateForFactory.json" \
  --parameters "@ARMTemplateParametersForFactory.stg.json"
```

#### 本番環境

```bash
az deployment group create \
  --resource-group "labs" \
  --template-file "ARMTemplateForFactory.json" \
  --parameters "@ARMTemplateParametersForFactory.prod.json"
```

### 2. コマンドライン引数によるデプロイ

```bash
az deployment group create \
  --resource-group "labs" \
  --template-file "ARMTemplateForFactory.json" \
  --parameters \
    factoryName="labs-adf-custom" \
    managedIdentityResourceId="/subscriptions/1e59e522-0988-4b3e-9d49-ba7db35962dd/resourceGroups/labs/providers/Microsoft.ManagedIdentity/userAssignedIdentities/labs_id_custom" \
    storageEndpoint="https://customstorage.blob.core.windows.net/"
```

## テンプレートの特徴

### シンプルな設計

- パラメータ数を最小限に抑制（3 つのみ）
- `managedIdentityResourceId`から自動的にクレデンシャル名を生成
- 環境固有の複雑なマッピングロジックを排除

### 柔軟性

- パラメータファイルとコマンドライン引数の両方に対応
- 新しい環境の追加が容易
- カスタムデプロイメントパターンのサポート

### 保守性

- 明確なパラメータ構造
- 環境ごとの設定の分離
- 自動化しやすい設計

## ベストプラクティス

1. **環境分離**: 各環境用に専用のパラメータファイルを使用
2. **命名規則**: 環境名をリソース名に含める (例: `labs-adf-dev`)
3. **リソース ID**: 完全なリソース ID を使用してリソースを一意に識別
4. **セキュリティ**: マネージド ID を使用して認証情報を安全に管理

## トラブルシューティング

### よくある問題

1. **マネージド ID が存在しない**: デプロイ前にマネージド ID リソースが作成されていることを確認
2. **ストレージアカウントアクセス権限**: マネージド ID にストレージアカウントへの適切な権限が付与されていることを確認
3. **リソースグループ**: 指定したリソースグループが存在することを確認

### デバッグ方法

```bash
# デプロイメントの検証（実際のデプロイメントは実行されない）
az deployment group validate \
  --resource-group "labs" \
  --template-file "ARMTemplateForFactory.json" \
  --parameters "@ARMTemplateParametersForFactory.dev.json"
```

## 情報ソース

- [Azure Data Factory ARM テンプレートリファレンス](https://docs.microsoft.com/azure/data-factory/quickstart-create-data-factory-resource-manager-template)
- [Azure Resource Manager テンプレートのベストプラクティス](https://docs.microsoft.com/azure/azure-resource-manager/templates/best-practices)
