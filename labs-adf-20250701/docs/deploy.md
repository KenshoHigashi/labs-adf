# デプロイ

## パラメーター直指定

### dev

```
az deployment group create `
  --resource-group "labs" `
  --template-file "ARMTemplateForFactory.json" `
  --parameters `
    factoryName="labs-adf-20250701" `
    subscriptionId="1e59e522-0988-4b3e-9d49-ba7db35962dd" `
    resourceGroupName="labs" `
    credentialName="labs_id_dev" `
    storageEndpoint="https://labsstorageaccountpdev.blob.core.windows.net/"
```

### stg

```
az deployment group create `
  --resource-group "labs" `
  --template-file "ARMTemplateForFactory.json" `
  --parameters `
    factoryName="labs-adf-20250701" `
    subscriptionId="1e59e522-0988-4b3e-9d49-ba7db35962dd" `
    resourceGroupName="labs" `
    credentialName="labs_id_stg" `
    storageEndpoint="https://labsstorageaccountpstg.blob.core.windows.net/"
```

### prod

```
az deployment group create `
  --resource-group "labs" `
  --template-file "ARMTemplateForFactory.json" `
  --parameters `
    factoryName="labs-adf-20250701" `
    subscriptionId="1e59e522-0988-4b3e-9d49-ba7db35962dd" `
    resourceGroupName="labs" `
    credentialName="labs_id_prod" `
    storageEndpoint="https://labsstorageaccountpprod.blob.core.windows.net/"
```

## パラメーターファイル使用

### dev

```
az deployment group create `
  --resource-group "labs" `
  --template-file "ARMTemplateForFactory.json" `
  --parameters "@ARMTemplateParametersForFactory.dev.json"
```

### stg

```
az deployment group create `
  --resource-group "labs" `
  --template-file "ARMTemplateForFactory.json" `
  --parameters "@ARMTemplateParametersForFactory.stg.json"
```

### prod

```
az deployment group create `
  --resource-group "labs" `
  --template-file "ARMTemplateForFactory.json" `
  --parameters "@ARMTemplateParametersForFactory.prod.json"
```

# 実行

```
az datafactory pipeline create-run `
 --factory-name "labs-adf-20250701" `
 --name "pipeline1" `
 --resource-group "labs"
```
