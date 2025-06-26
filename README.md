# Deploy a Python (Flask) web app to Azure App Service - Sample Application

This is the sample Flask application for the Azure Quickstart [Deploy a Python (Django or Flask) web app to Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/quickstart-python). For instructions on how to create the Azure resources and deploy the application to Azure, refer to the Quickstart article.

Sample applications are available for the other frameworks here:

* Django [https://github.com/Azure-Samples/msdocs-python-django-webapp-quickstart](https://github.com/Azure-Samples/msdocs-python-django-webapp-quickstart)
* FastAPI [https://github.com/Azure-Samples/msdocs-python-fastapi-webapp-quickstart](https://github.com/Azure-Samples/msdocs-python-fastapi-webapp-quickstart)

If you need an Azure account, you can [create one for free](https://azure.microsoft.com/en-us/free/).



### API DOC

def find_corrections(corrected_text):
    """
    GPT-4의 응답 결과(corrected_text)를 분석하여 틀린 부분을 찾습니다.
    """
    corrections = []
    # 정규 표현식을 사용하여 틀린 부분과 수정 이유 또는提示을 추출
    pattern = r'<span style="(?:background-color:#ace4e6;color:red;|color:red);">(.*?)</span> \((<span>(修正理由:|提示:)(.*?)</span>)\)'
    matches = re.findall(pattern, corrected_text)
    
    for match in matches:
        original_text = match[0]  # 틀린 부분
        reason_type = match[2]  # 수정 이유 또는提示
        comment = f"{reason_type} {match[4].strip()}"  # 수정 이유 또는提示

        # 'locations' 필드를 미리 빈 리스트로 추가해 둠
        corrections.append({
            "page": 0,  # 페이지 번호 (0부터 시작, 필요 시 수정)
            "original_text": original_text,
            "comment": comment,
            "locations": []  # 뒤에서 실제 PDF 위치(좌표)를 저장할 필드
        })
    
    return corrections


az resource update \
    --resource-group nripi-commentcheck-rg \
    --name nricosmosdb1 \
    --resource-type Microsoft.DocumentDB/databaseAccounts \
    --set properties.disableLocalAuth=true



hypercorn app:app --bind 0.0.0.0:8000 --workers 2



az resource update \
--resource-group nripi-commentcheck-rg \
--name scm \
--namespace Microsoft.Web \
--resource-type basicPublishingCredentialsPolicies \
--parent sites/NamCheckWeb \
--set properties.allow=true


### **Example Output:**
(1) If the meaning is inconsistent:

    (2) If the meaning is consistent:
   Do not print (Return nothing)


   You are a professional After understanding the professional rules, 
        you become a Japanese text proofreading assistant for the article.




####


```markdown
通过 Azure CLI 配置文件
Azure CLI 支持直接在配置文件中设置代理。
1. 打开 Azure CLI 配置文件
配置文件路径通常是：

  *   Windows: %USERPROFILE%\.azure\config
复制下面配置到config
――――――――――――――――――――――――――――――――――――――
[cloud]
name = AzureCloud
http_proxy = http://80023：nomura49@nriproxy.nri.co.jp:92
https_proxy = http://80023：nomura49@nriproxy.nri.co.jp:92
no_proxy = localhost,127.0.0.1,.example.com
――――――――――――――――――――――――――――――――――――――
```

```vscode setting
{
    "liveServer.settings.port": 5502,
    "appService.defaultWebAppToDeploy": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.Web/sites/NamCheckWeb",
    "appService.deploySubpath": "flaskQuick-master"
}
```


Cosmos DB에서 “(Forbidden) Request blocked by Auth” 오류가 계속 발생한다는 것은, RBAC(역할 기반 액세스 제어) 권한이 아직 완전히 충족되지 않았거나, 권한이 제대로 전파되지 않은 경우가 대부분입니다.
아래 항목들을 하나씩 점검해 보시길 권장드립니다.

1. 할당된 역할이 정말 “Cosmos DB Built-in Data Reader”인지 확인
Azure Portal 또는 CLI에서 다음을 꼭 확인해 주세요.

Azure Portal

nricosmosdb1 리소스의 왼쪽 메뉴에서 Access control (IAM) 클릭
‘Role assignments’ 탭에서 주체(principal)(eb4303ce-d5b0-417b-84cf-0acafc5dc606)에게
정확히 “Cosmos DB Built-in Data Reader” 역할이 할당되어 있는지 확인
단순히 Azure Resource Manager(ARM) 차원의 Reader가 아니라,
Cosmos DB 데이터 영역 전용 역할이 제대로 부여되었는지 주의
Azure CLI

bash
```
az cosmosdb sql role assignment list \
  --account-name nricosmosdb1 \
  --resource-group nripi-commentcheck-rg
```
출력 결과 중 "principalId": "eb4303ce-d5b0-417b-84cf-0acafc5dc606" 이며
"roleDefinitionId": "00000000-0000-0000-0000-000000000001" (Cosmos DB Built-in Data Reader)
인 항목이 존재하는지 확인
참고: 만약 az role assignment list (ARM 용)로 확인하면, Cosmos DB의 데이터 플레인 권한은 나오지 않을 수 있습니다. Cosmos DB Role Assignments는 az cosmosdb sql role assignment 명령어를 사용해야 제대로 확인됩니다.

2. Scope(할당 범위)가 올바른지 확인
Cosmos DB 데이터 플레인 권한은
/subscriptions/<subscriptionId>/resourceGroups/<rgName>/providers/Microsoft.DocumentDB/databaseAccounts/<accountName>
이 범위로 지정되어야 합니다.

CLI로 할당 시, --scope 값을 해당 Cosmos DB 리소스로 정확히 지정했는지 재확인
Azure Portal에서 역할 할당 시, 상단의 ‘Scope’(또는 ‘할당 범위’)가 제대로 해당 Cosmos DB 계정으로 잡혀 있는지 확인
예:

bash
复制代码
az cosmosdb sql role assignment create \
  --account-name nricosmosdb1 \
  --resource-group nripi-commentcheck-rg \
  --scope "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1" \
  --principal-id eb4303ce-d5b0-417b-84cf-0acafc5dc606 \
  --role-definition-id "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1/sqlRoleDefinitions/00000000-0000-0000-0000-000000000001"
3. 주체(Principal) ID가 정확한지 재확인
에러 메시지에 표시된 eb4303ce-d5b0-417b-84cf-0acafc5dc606가
정말 권한을 부여하려는 사용자/그룹/서비스 주체의 Object ID인지 확인합니다.

가령, 애플리케이션(Azure AD 등록 앱)일 경우, Application ID와 Object ID가 다를 수 있습니다.
사용자일 경우, 사용자 Principal ID가 올바른지, 그룹일 경우 그룹 Object ID가 맞는지 재확인이 필요합니다.
4. 권한 전파(프로비저닝) 대기
Azure RBAC나 Cosmos DB의 Role Assignment가 추가된 직후,
수 분(최대 15분 이상) 정도는 전파(Propagation) 시간이 걸릴 수 있습니다.
역할 할당 후 즉시 요청하면 “Forbidden” 오류가 계속 보일 수 있으니,
몇 분 후에 다시 시도해 보세요.

5. (참고) Azure RBAC vs. Cosmos DB "SQL Role" 혼동 주의
Cosmos DB에는 두 가지 레벨의 접근 제어 방식이 있습니다:

Azure Resource Manager(ARM) RBAC

Portal의 **Access control (IAM)**에서 부여하는 Reader, Contributor 등
이 권한은 리소스(계정) 자체의 생성·삭제·관리 권한을 정의
Cosmos DB Data Plane(데이터 레벨) RBAC

az cosmosdb sql role definition/assignment 명령어로 확인 가능한 권한
“Cosmos DB Built-in Data Reader” 등 데이터 접근(쿼리, 메타데이터, items 읽기)을 위한 역할
현재 에러 메시지에 등장하는 readMetadata 액션은 데이터 레벨 권한이 필요합니다.
ARM 레벨에서 Reader 역할을 줬더라도, Cosmos DB 내부 데이터 접근 권한이 자동으로 부여되지 않습니다.
반드시 Cosmos DB SQL Role Definition(Built-in Data Reader 등)을 az cosmosdb sql role assignment create 로 할당해야 합니다.
6. 점검 요약
주체의 Object ID가 맞는지 재확인
Cosmos DB Built-in Data Reader 역할을 해당 주체에게 정확한 Scope로 할당했는지 확인
포털이나 CLI를 통해 할당 내역이 실제로 보이는지(특히 az cosmosdb sql role assignment list) 확인
권한 전파 후 재시도
이 과정을 모두 점검했음에도 “Forbidden”이 계속 발생한다면:

(아주 드물게) Azure Portal 자체 캐싱 문제나 권한 전파 지연으로 수십 분 이상 걸릴 때도 있습니다.
서비스 주체를 사용하는 경우, 인증 토큰이 역할 할당 전에 발급된 토큰이라면, 토큰이 만료될 때까지 권한이 적용되지 않을 수 있습니다. 이 경우, 애플리케이션 측에서 토큰 재발급을 시도해야 합니다.
결론
위 단계를 모두 확인하고, 특히 az cosmosdb sql role assignment list 명령어에서 해당 principal ID가 “Cosmos DB Built-in Data Reader”에 정상적으로 매핑되어 있는지 먼저 확인해 주세요.
할당이 되어 있는데도 Forbidden이 발생한다면, 역할 전파 지연이 원인일 가능성이 높으므로, 조금 시간차를 두고 다시 시도하거나, 인증 토큰 재획득 등의 과정을 거쳐 재시도해 보시기 바랍니다.
이렇게 점검하시면 “Request is blocked because principal ... does not have required RBAC permissions” 오류를 해결하실 수 있을 것입니다.


####

###### ERROR  ######
```
{
    "error": "(Forbidden) Request blocked by Auth nricosmosdb1 : Request is blocked because principal [eb4303ce-d5b0-417b-84cf-0acafc5dc606] does not have required RBAC permissions to perform action [Microsoft.DocumentDB/databaseAccounts/readMetadata] on resource [/]. Learn more: https://aka.ms/cosmos-native-rbac.\r\nActivityId: 98c23aed-3e2c-464c-8eb9-7085ad298799, Microsoft.Azure.Documents.Common/2.14.0\nCode: Forbidden\nMessage: Request blocked by Auth nricosmosdb1 : Request is blocked because principal [eb4303ce-d5b0-417b-84cf-0acafc5dc606] does not have required RBAC permissions to perform action [Microsoft.DocumentDB/databaseAccounts/readMetadata] on resource [/]. Learn more: https://aka.ms/cosmos-native-rbac.\r\nActivityId: 98c23aed-3e2c-464c-8eb9-7085ad298799, Microsoft.Azure.Documents.Common/2.14.0",
    "success": false
}

```
####



1.  az cosmosdb sql role definition list --account-name nricosmosdb1 --resource-group nripi-commentcheck-rg

```
[
  {
    "assignableScopes": [
      "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1"
    ],
    "id": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1/sqlRoleDefinitions/00000000-0000-0000-0000-000000000001",
    "name": "00000000-0000-0000-0000-000000000001",
    "permissions": [
      {
        "dataActions": [
          "Microsoft.DocumentDB/databaseAccounts/readMetadata",
          "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/executeQuery",
          "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/readChangeFeed",
          "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/read"
        ],
        "notDataActions": []
      }
    ],
    "resourceGroup": "nripi-commentcheck-rg",
    "roleName": "Cosmos DB Built-in Data Reader",
    "type": "Microsoft.DocumentDB/databaseAccounts/sqlRoleDefinitions",
    "typePropertiesType": "BuiltInRole"
  },
  {
    "assignableScopes": [
      "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1"
    ],
    "id": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1/sqlRoleDefinitions/00000000-0000-0000-0000-000000000002",
    "name": "00000000-0000-0000-0000-000000000002",
    "permissions": [
      {
        "dataActions": [
          "Microsoft.DocumentDB/databaseAccounts/readMetadata",
          "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/*",
          "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/*"
        ],
        "notDataActions": []
      }
    ],
    "resourceGroup": "nripi-commentcheck-rg",
    "roleName": "Cosmos DB Built-in Data Contributor",
    "type": "Microsoft.DocumentDB/databaseAccounts/sqlRoleDefinitions",
    "typePropertiesType": "BuiltInRole"
  }
]
```

2. az cosmosdb sql role definition list --account-name nricosmosdb1 --resource-group nripi-commentcheck-rg

```
[
  {
    "id": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1/sqlRoleAssignments/daf22d25-03f8-4ef6-8512-f208a28d4159",
    "name": "daf22d25-03f8-4ef6-8512-f208a28d4159",
    "principalId": "7eb31883-a24f-422d-a99d-4250160b0070",
    "resourceGroup": "nripi-commentcheck-rg",
    "roleDefinitionId": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1/sqlRoleDefinitions/00000000-0000-0000-0000-000000000002",
    "scope": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1",
    "type": "Microsoft.DocumentDB/databaseAccounts/sqlRoleAssignments"
  },
  {
    "id": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1/sqlRoleAssignments/d12b434e-d91b-40c2-a488-8e0297161b61",
    "name": "d12b434e-d91b-40c2-a488-8e0297161b61",
    "principalId": "7eb31883-a24f-422d-a99d-4250160b0070",
    "resourceGroup": "nripi-commentcheck-rg",
    "roleDefinitionId": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1/sqlRoleDefinitions/00000000-0000-0000-0000-000000000001",
    "scope": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1",
    "type": "Microsoft.DocumentDB/databaseAccounts/sqlRoleAssignments"
  },
  {
    "id": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1/sqlRoleAssignments/9383c90f-217f-4377-9a5c-2a1d621e7cd4",
    "name": "9383c90f-217f-4377-9a5c-2a1d621e7cd4",
    "principalId": "eb4303ce-d5b0-417b-84cf-0acafc5dc606",
    "resourceGroup": "nripi-commentcheck-rg",
    "roleDefinitionId": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1/sqlRoleDefinitions/00000000-0000-0000-0000-000000000002",
    "scope": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1",
    "type": "Microsoft.DocumentDB/databaseAccounts/sqlRoleAssignments"
  },
  {
    "id": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1/sqlRoleAssignments/e93bd168-fbea-4211-8e60-1a31cc9ed5e8",
    "name": "e93bd168-fbea-4211-8e60-1a31cc9ed5e8",
    "principalId": "eb4303ce-d5b0-417b-84cf-0acafc5dc606",
    "resourceGroup": "nripi-commentcheck-rg",
    "roleDefinitionId": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1/sqlRoleDefinitions/00000000-0000-0000-0000-000000000001",
    "scope": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1",
    "type": "Microsoft.DocumentDB/databaseAccounts/sqlRoleAssignments"
  }
]
```

3. 권한 추가: 2는 Cosmos DB Built-in Data Contributor , 1는 Cosmos DB Built-in Data Reader , 
### cosmosdb에 접속할려면, Cosmos DB Built-in Data Contributor 권한이 필요
### --principal-id 는 Webapp에서, 설정->标识（Indentyfy）에서, 상태를  open해야 ,principal-id 자동으로 생성된다 

az cosmosdb sql role assignment create   --account-name nricosmosdb1   --resource-group nripi-commentcheck-rg   --scope "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1"   --principal-id eb4303ce-d5b0-417b-84cf-0acafc5dc606   --role-definition-id "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1/sqlRoleDefinitions/00000000-0000-0000-0000-000000000001"

az cosmosdb sql role assignment create   --account-name nricosmosdb1   --resource-group nripi-commentcheck-rg   --scope "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1"   --principal-id eb4303ce-d5b0-417b-84cf-0acafc5dc606   --role-definition-id "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1/sqlRoleDefinitions/00000000-0000-0000-0000-000000000002"


4. 다시 
```
az cosmosdb sql role definition list --account-name nricosmosdb1 --resource-group nripi-commentcheck-rg
```
으로 2번(Cosmos DB Built-in Data Contributor)권한이 제대로 부여되였는지 확인 


5. 
```error
azure.cosmos.exceptions.CosmosHttpRePrograms\Python\Python312\Lib\sitesponseError: (Unauthorized) Local Au_request.py", line 155, in _Requesthorization is disabled. Use an AAD 
token to authorize all requests.ActinseError(message=data, response=revityId: e2f31f0a-572b-41a3-b9ac-2fae9f233864, Microsoft.Azure.Documents.sponseError: (Unauthorized) Local Common/2.14.0                       D token to authorize all requests.
Code: Unauthorized                  2fae9f233864, Microsoft.Azure.Docu
Message: Local Authorization is disabled. Use an AAD token to authorize 
all requests.                       bled. Use an AAD token to authoriz
ActivityId: e2f31f0a-572b-41a3-b9ac-2fae9f233864, Microsoft.Azure.Docume2fae9f233864, Microsoft.Azure.Docunts.Common/2.14.0
```
- 
```json, "disableLocalAuth": true 설정을 해야 한다.
{
    "id": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.DocumentDB/databaseAccounts/nricosmosdb1",
    "name": "nricosmosdb1",
    "location": "Japan East",
    "type": "Microsoft.DocumentDB/databaseAccounts",
    "kind": "GlobalDocumentDB",
    "tags": {
        "defaultExperience": "Core (SQL)",
        "hidden-cosmos-mmspecial": ""
    },
    "systemData": {
        "createdAt": "2024-12-17T00:32:17.0815688+00:00"
    },
    "properties": {
        "provisioningState": "Succeeded",
        "documentEndpoint": "https://nricosmosdb1.documents.azure.com:443/",
        "sqlEndpoint": "https://nricosmosdb1.documents.azure.com:443/",
        "publicNetworkAccess": "Enabled",
        "enableAutomaticFailover": false,
        "enableMultipleWriteLocations": false,
        "enablePartitionKeyMonitor": false,
        "isVirtualNetworkFilterEnabled": true,
        "virtualNetworkRules": [
            {
                "id": "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.Network/virtualNetworks/kf-vnet/subnets/default",
                "ignoreMissingVNetServiceEndpoint": false
            }
        ],
        "EnabledApiTypes": "Sql",
        "disableKeyBasedMetadataWriteAccess": false,
        "enableFreeTier": false,
        "enableAnalyticalStorage": false,
        "analyticalStorageConfiguration": {
            "schemaType": "WellDefined"
        },
        "instanceId": "1bd55959-ddad-421b-ae8f-25d04020f902",
        "databaseAccountOfferType": "Standard",
        "enableMaterializedViews": false,
        "defaultIdentity": "FirstPartyIdentity",
        "networkAclBypass": "None",
        "disableLocalAuth": true,
        "enablePartitionMerge": false,
        "enableBurstCapacity": false,
        "minimalTlsVersion": "Tls12",
        "consistencyPolicy": {
            "defaultConsistencyLevel": "Session",
            "maxIntervalInSeconds": 5,
            "maxStalenessPrefix": 100
        },
        "configurationOverrides": {
            "EnablePerRegionPerPartitionAutoscaleOptIn": "True"
        },
        "writeLocations": [
            {
                "id": "nricosmosdb1-japaneast",
                "locationName": "Japan East",
                "documentEndpoint": "https://nricosmosdb1-japaneast.documents.azure.com:443/",
                "provisioningState": "Succeeded",
                "failoverPriority": 0,
                "isZoneRedundant": false
            }
        ],
        "readLocations": [
            {
                "id": "nricosmosdb1-japaneast",
                "locationName": "Japan East",
                "documentEndpoint": "https://nricosmosdb1-japaneast.documents.azure.com:443/",
                "provisioningState": "Succeeded",
                "failoverPriority": 0,
                "isZoneRedundant": false
            }
        ],
        "locations": [
            {
                "id": "nricosmosdb1-japaneast",
                "locationName": "Japan East",
                "documentEndpoint": "https://nricosmosdb1-japaneast.documents.azure.com:443/",
                "provisioningState": "Succeeded",
                "failoverPriority": 0,
                "isZoneRedundant": false
            }
        ],
        "failoverPolicies": [
            {
                "id": "nricosmosdb1-japaneast",
                "locationName": "Japan East",
                "failoverPriority": 0
            }
        ],
        "cors": [],
        "capabilities": [],
        "ipRules": [
            {
                "ipAddressOrRange": "4.190.202.251"
            },
            {
                "ipAddressOrRange": "133.250.166.4"
            },
            {
                "ipAddressOrRange": "133.250.166.5"
            },
            {
                "ipAddressOrRange": "133.250.166.117"
            },
            {
                "ipAddressOrRange": "133.250.166.118"
            },
            {
                "ipAddressOrRange": "4.190.207.63"
            },
            {
                "ipAddressOrRange": "4.190.204.45"
            },
            {
                "ipAddressOrRange": "4.190.206.149"
            },
            {
                "ipAddressOrRange": "4.190.206.129"
            },
            {
                "ipAddressOrRange": "4.190.206.126"
            },
            {
                "ipAddressOrRange": "4.190.201.43"
            },
            {
                "ipAddressOrRange": "20.27.245.202"
            },
            {
                "ipAddressOrRange": "20.27.244.132"
            },
            {
                "ipAddressOrRange": "20.27.244.7"
            },
            {
                "ipAddressOrRange": "20.27.240.14"
            },
            {
                "ipAddressOrRange": "4.190.207.180"
            },
            {
                "ipAddressOrRange": "4.190.207.181"
            },
            {
                "ipAddressOrRange": "4.190.206.140"
            },
            {
                "ipAddressOrRange": "4.190.206.76"
            },
            {
                "ipAddressOrRange": "4.190.206.115"
            },
            {
                "ipAddressOrRange": "4.190.206.2"
            },
            {
                "ipAddressOrRange": "4.190.205.124"
            },
            {
                "ipAddressOrRange": "4.190.205.18"
            },
            {
                "ipAddressOrRange": "4.190.201.52"
            },
            {
                "ipAddressOrRange": "4.190.204.57"
            },
            {
                "ipAddressOrRange": "4.190.203.88"
            },
            {
                "ipAddressOrRange": "4.190.202.27"
            },
            {
                "ipAddressOrRange": "4.190.202.18"
            },
            {
                "ipAddressOrRange": "4.190.202.251"
            },
            {
                "ipAddressOrRange": "4.190.206.191"
            },
            {
                "ipAddressOrRange": "20.27.243.141"
            },
            {
                "ipAddressOrRange": "40.74.100.143"
            },
            {
                "ipAddressOrRange": "4.190.206.167"
            },
            {
                "ipAddressOrRange": "20.27.241.135"
            },
            {
                "ipAddressOrRange": "20.27.245.217"
            },
            {
                "ipAddressOrRange": "20.27.246.33"
            },
            {
                "ipAddressOrRange": "20.24.122.172"
            },
            {
                "ipAddressOrRange": "133.250.166.4"
            },
            {
                "ipAddressOrRange": "133.250.166.5"
            },
            {
                "ipAddressOrRange": "133.250.166.117"
            },
            {
                "ipAddressOrRange": "133.250.166.118"
            },
            {
                "ipAddressOrRange": "24.239.132.222"
            },
            {
                "ipAddressOrRange": "24.239.132.223"
            },
            {
                "ipAddressOrRange": "24.239.141.227"
            },
            {
                "ipAddressOrRange": "24.239.141.228"
            },
            {
                "ipAddressOrRange": "24.239.147.179"
            },
            {
                "ipAddressOrRange": "24.239.147.180"
            },
            {
                "ipAddressOrRange": "24.239.154.8"
            },
            {
                "ipAddressOrRange": "24.239.154.9"
            },
            {
                "ipAddressOrRange": "4.210.172.107"
            },
            {
                "ipAddressOrRange": "13.88.56.148"
            },
            {
                "ipAddressOrRange": "13.91.105.215"
            },
            {
                "ipAddressOrRange": "40.91.218.243"
            }
        ],
        "backupPolicy": {
            "type": "Periodic",
            "periodicModeProperties": {
                "backupIntervalInMinutes": 240,
                "backupRetentionIntervalInHours": 8,
                "backupStorageRedundancy": "Geo"
            }
        },
        "networkAclBypassResourceIds": [],
        "diagnosticLogSettings": {
            "enableFullTextQuery": "None"
        },
        "capacity": {
            "totalThroughputLimit": 4000
        },
        "keysMetadata": {
            "primaryMasterKey": {
                "generationTime": "2025-02-04T23:57:58.3009824Z"
            },
            "secondaryMasterKey": {
                "generationTime": "2024-12-17T00:32:17.0815688+00:00"
            },
            "primaryReadonlyMasterKey": {
                "generationTime": "2024-12-17T00:32:17.0815688+00:00"
            },
            "secondaryReadonlyMasterKey": {
                "generationTime": "2024-12-17T00:32:17.0815688+00:00"
            }
        }
    },
    "identity": {
        "type": "systemassigned,userassigned",
        "principalId": "c5675513-efff-46c5-a862-cfa074ebb95a",
        "tenantId": "6700e121-fc69-448d-89fc-230dca6c2c25",
        "userAssignedIdentities": {
            "/subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourcegroups/nripi-commentcheck-rg/providers/Microsoft.ManagedIdentity/userAssignedIdentities/api-banned-pro-identity": {
                "principalId": "3eeddb04-d6c8-4741-97ff-7828baadfdae",
                "clientId": "0658eb0a-e3e1-42eb-91f2-aeff18afbf39"
            }
        }
    },
    "apiVersion": "2023-03-01-preview"
}
```

```Azure CLI ,
az cosmosdb show  --name nricosmosdb1 --resource-group nripi-commentcheck-rg --query "disableLocalAuth"
az cosmosdb update  --name nricosmosdb1 --resource-group nripi-commentcheck-rg --disable-local-auth false
```

##  local update to cosmosDB
```json
az resource update \
    --resource-group nripi-commentcheck-rg \
    --name nricosmosdb1 \
    --resource-type Microsoft.DocumentDB/databaseAccounts \
    --set properties.disableLocalAuth=false

```

```json, check show
az cosmosdb show \
    --name nricosmosdb1 \
    --resource-group nripi-commentcheck-rg \
    --query "disableLocalAuth"

```
### openai 를 추가시, 
azure_ad를 사용하고
버전명을 적어야 되고
---
iam访问控制里，需要添加 认知服务项目，标识访问，授权给web
---------------------------------



----------------------------------------------------------------------------
Account Key(계정 키) 사용
Connection String(계정 키 기반)
SAS 토큰(Shared Access Signature)
Azure AD(Azure Active Directory) RBAC
----------------------------------------------------------------------------



az storage account show -n nriazureaistudio --query networkRuleSet




# storage 

## error

Upload error: Error: Server responded with status 500: Blob 업로드 실패: This request is not authorized to perform this operation using this permission.
RequestId:5fa8fe0d-701e-000c-7e14-58d485000000
Time:2024-12-27T04:03:52.3695195Z
ErrorCode:AuthorizationPermissionMismatch
Content: <?xml version="1.0" encoding="utf-8"?><Error><Code>AuthorizationPermissionMismatch</Code><Message>This request is not authorized to perform this operation using this permission.
RequestId:5fa8fe0d-701e-000c-7e14-58d485000000
Time:2024-12-27T04:03:52.3695195Z</Message></Error>

## check list
az role assignment list   --scope /subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.Storage/storageAccounts/nriazureaistudio   --output table


# setting assignee ,,web application

az role assignment create \
  --assignee eb4303ce-d5b0-417b-84cf-0acafc5dc606 \
  --role "Storage Blob Data Contributor" \
  --scope /subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-commentcheck-rg/providers/Microsoft.Storage/storageAccounts/nriazureaistudio

```js

```js

az role assignment list   --scope /subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-guichatbot-rg/providers/Microsoft.Storage/storageAccounts/ailabdatanri  --output table

az role assignment create \
  --assignee 267eef03-df3b-410a-b809-84b7fbd2e2d8 \
  --role "Storage Blob Data Contributor" \
  --scope /subscriptions/3bd3d7d9-a57d-4fbe-86e5-eb04bbd6bbbe/resourceGroups/nripi-guichatbot-rg/providers/Microsoft.Storage/storageAccounts/ailabdatanri

```


az webapp vnet-integration update \
  --name ailab-webapp2 \
  --resource-group nripi-guichatbot-rg \
  --vnet-route-all-enabled false


az webapp config set \
  --name ailab-webapp2 \
  --resource-group nripi-guichatbot-rg \
  --always-on true




az webapp update \
  --name ailab-webapp2 \
  --resource-group nripi-guichatbot-rg \
  --set clientAffinityEnabled=true



az webapp config set \
  --name ailab-webapp2 \
  --resource-group nripi-guichatbot-rg \
  --generic-configurations "{\"clientCertEnabled\": false}"




az webapp update \
  --name ailab-webapp2 \
  --resource-group nripi-guichatbot-rg \
  --set hostNames=["yourcustomdomain.com"]



az webapp update \
  --name ailab-webapp2 \
  --resource-group nripi-guichatbot-rg \
  --set defaultHostName="yourapp.azurewebsites.net"



az webapp show \
  --name ailab-webapp2 \
  --resource-group nripi-guichatbot-rg \
  --query privateEndpointConnections
```


```prompt ---old version
你是一个日语文字校对助手。
请你根据一下几点对日语报告书的内容进行校正。
1.誤字脱字がないこと
2.ファンドマネージャコメント用語集に沿った記載となっていること：	
- 表記の統一	
- 禁止（NG）ワード及び文章の注意事項	
- 置き換えが必要な用語/表現、置き換えを推奨する用語/表現	
- ひらがなを表記するもの（常用漢字にない読み方を修正）	
- 一部かな書き等で表記するもの（常用漢字以外はかな書きが基本形）	
- 一般的な送り仮名など	
- 英略語、外来語、専門用語など
3.レポートのデータ部との整合性確認

上述内容改正的规则我已经上传了一份文件:
'''{{search_text_0}}'''

当你找到需要修改的文本时，请按照改正规则替换原文。
```


```
你是一个专业的日语文字校对助手，请根据以下规则对日语报告书的内容进行详细校正：

校正要求：
誤字脱字がないこと

确保报告书内容中不存在任何错别字或脱漏的文字。
ファンドマネージャコメント用語集に沿った記載となっていること：

表記の統一
确保术语的书写格式统一。
禁止（NG）ワード及び文章の注意事項
检查报告中是否使用了禁止用语，并根据规则改正。
置き換えが必要な用語/表現、置き換えを推奨する用語/表現
如果发现需要替换的用语或表述，请根据规则进行改写。
ひらがなを表記するもの（常用漢字にない読み方を修正）
确保报告中遵循假名表记的规则，将不符合常用汉字的内容用假名替代。
一部かな書き等で表記するもの（常用漢字以外はかな書きが基本形）
针对常用汉字以外的表述，确保使用假名作为标准。
一般的な送り仮名など
确保使用了正确的送假名。
英略語、外来語、専門用語など
针对英语缩写、外来语和专业术语，检查其是否正确表达。
レポートのデータ部との整合性確認

确保报告的文字描述与数据部分完全一致，不存在逻辑或内容上的矛盾。
注意事项：
我已上传一份改正规则的详细文件供你参考：
{search_text_0}

当你发现需要修改的文本时，请按照上述规则替换原文内容，并确保替换后的文本与上下文保持逻辑一致。
````

```日语提示词

補正要件：
誤字脱字がないこと

報告書の内容に誤字や脱落した文字がないことを確認します。
ファンド・マネージャ・コメット用語集に沿った記載となっていること：

表記の統一
用語の表記形式が統一されていることを確認します。
禁止（NG）ワード及び文書の注意事項
レポートに禁止用語が使用されているかどうかをチェックし、ルールに基づいて修正します。
置換が必要な用語/表現、置換を推賞する用語/表現
置き換えが必要な用語や表現が見つかった場合は、規則に従って書き換えてください。
ひらがなを表記するもの（常用漢字にない読み方を修正）
報告書の中で仮名表記の規則に従って、常用漢字に合わない内容を仮名で置き換えることを確保します。
一部かな書きなどで表記するもの（常用漢字以外はかな書きが基本形）
常用漢字以外の表記については、仮名を標準として使用することを確認します。
一般的な送り仮名など
正しい送り仮名が使用されていることを確認します。
英略語、外来語、専門用語など
英語の略語、外来語、専門用語について、正しく表現されているかどうかをチェックします。
レポートのデータ部との整合性確認

レポートのテキスト記述がデータ部分と完全に一致し、論理的または内容的な矛盾がないことを確認します。
```


```
prompt_result = f"""

        You are a professional Japanese text proofreading assistant. Please carefully proofread the content of a Japanese report following the rules below:

        Proofreading Requirements:
            1. Check for typos and missing characters (誤字脱字がないこと):
            - Ensure there are no spelling errors or missing characters in the content of the report.

            2. Follow the "Fund Manager Comment Terminology Guide" (ファンドマネージャコメント用語集に沿った記載となっていること):

                - Consistent Terminology (表記の統一):
                    Ensure the writing format of terms is consistent throughout the report.
                - Prohibited Words and Phrases (禁止（NG）ワード及び文章の注意事項):
                    Check if any prohibited words or phrases are used in the report and correct them as per the guidelines.
                - Replaceable and Recommended Terms/Expressions (置き換えが必要な用語/表現、置き換えを推奨する用語/表現):
                    If you find terms or expressions that need to be replaced, revise them according to the provided rules.
                - Use of Hiragana (ひらがなを表記するもの):
                    Ensure the report follows the rules for hiragana notation, replacing content that does not conform to commonly used kanji.
                - Kana Notation for Non-Standard Kanji (一部かな書き等で表記するもの):
                    Ensure non-standard kanji are replaced with kana as the standard writing format.
                - Correct Usage of Okurigana (一般的な送り仮名など):
                    Ensure the correct usage of okurigana is applied.
                - English Abbreviations, Loanwords, and Technical Terms (英略語、外来語、専門用語など):
                    Check if English abbreviations, loanwords, and technical terms are expressed correctly.
            3. Consistency with Report Data Section (レポートのデータ部との整合性確認):
                - Ensure the textual description in the report is completely consistent with the data section, without any logical or content-related discrepancies.

        Output Requirements:
        1. Output only valid HTML.

            - Do not include any additional explanations, markdown formatting, or surrounding text.
            - The result must start with <html> and end with </html>.
        2. Highlight corrections in red text.

            - For corrected parts, wrap the text in <span style="color:red;"> tags to indicate the modifications.
            - Example:
                Original: レポートデータが不一致です。
                Corrected: レポートデータが<span style="color:red;">一致</span>です。
        3. Preserve the original structure and formatting of the document.

            - Maintain paragraph breaks, headings, and any existing structure in the content.
        4. Use the uploaded correction rules for reference:
            {prompt}

        5. Do not provide any explanations or descriptions in the output. Only return the corrected HTML content.

        """

        ```


#### pdf 직접 수정


```

import fitz  # PyMuPDF
import os
from paddleocr import PaddleOCR
import tkinter as tk
from tkinter import filedialog

# PaddleOCR 초기화
ocr = PaddleOCR(use_angle_cls=True, lang="ch")


# 시스템에 설치된 폰트 목록 출력

# PDF에서 텍스트 추출 및 수정 함수
def extract_and_modify_pdf(pdf_path, output_pdf_path):
    doc = fitz.open(pdf_path)
    for i in range(len(doc)):
        page = doc.load_page(i)
        text_instances = page.search_for("2024年4月30日")  # 특정 텍스트 검색

        # 찾은 텍스트 인스턴스에 대해 수정
        for inst in text_instances:
            ## 텍스트 삭제 (redact)
            page.add_redact_annot(inst, fill=(1, 1, 1))  # 흰색으로 채우기 (배경색)
            page.apply_redactions()  # 삭제 적용

            # 새로운 텍스트 삽입
            page.insert_text(
                inst.bl,  # 텍스트 위치 (좌측 하단)
                "2024年4月30日",  # 텍스트 내용
                fontsize=8,  # 폰트 크기
                color=(1, 0, 0),  # 붉은색 (RGB)
                fontname="helv",  # 폰트 이름
            )

    # 수정된 PDF 저장
    doc.save(output_pdf_path)
    doc.close()

# OCR 결과를 텍스트 파일로 저장하는 함수
def save_text_to_txt(texts, output_file):
    with open(output_file, 'w', encoding='utf-8') as f:
        for idx, page_text in enumerate(texts):
            f.write(f"Page {idx + 1}:\n")
            f.write("\n".join(page_text))
            f.write("\n\n")

# 디렉토리에서 모든 PDF 파일을 처리하는 함수
def process_all_pdfs_in_directory(directory_path):
    # PDF 파일을 저장할 output 폴더를 선택된 디렉토리 하위에 생성
    output_directory = os.path.join(directory_path, "OCR_output")
    os.makedirs(output_directory, exist_ok=True)

    # 모든 PDF 파일을 검색 및 처리
    for filename in os.listdir(directory_path):
        if filename.endswith(".pdf"):
            pdf_path = os.path.join(directory_path, filename)
            output_pdf_path = os.path.join(output_directory, f"{filename[:-4]}_modified.pdf")
            output_txt_path = os.path.join(output_directory, f"{filename[:-4]}_output.txt")

            print(f"Processing {pdf_path}...")

            # PDF 텍스트 수정 및 저장
            extract_and_modify_pdf(pdf_path, output_pdf_path)

            # OCR을 통해 텍스트 추출 및 저장
            doc = fitz.open(pdf_path)
            all_texts = []
            for i in range(len(doc)):
                page = doc.load_page(i)
                image = page.get_pixmap()
                img_data = image.tobytes()

                # OCR 적용
                result = ocr.ocr(img_data, cls=True)
                page_text = [word[-1][0] for line in result for word in line]
                all_texts.append(page_text)

            doc.close()

            # OCR 결과 저장
            save_text_to_txt(all_texts, output_txt_path)
            print(f"Saved modified PDF to {output_pdf_path}")
            print(f"Saved OCR results to {output_txt_path}")

# 폴더 선택을 위한 filedialog 함수
def select_directory(prompt):
    root = tk.Tk()
    root.withdraw()  # GUI 창을 숨김
    folder_selected = filedialog.askdirectory(title=prompt)
    return folder_selected

# 실행 부분
if __name__ == "__main__":
    # PDF 파일이 있는 폴더 선택
    input_directory = select_directory("Select the folder containing PDF files")
    if not input_directory:
        print("No input folder selected. Exiting.")
        exit()

    # 디렉토리 내 모든 PDF 파일 처리 및 결과 저장
    process_all_pdfs_in_directory(input_directory)

    print("All PDF files processed successfully.")
    
```

# 2. excel to pdf

```
# import os
# import win32com.client

# # 엑셀 파일 로드
# excel_path = os.path.abspath("D:/CommentCheck/test/OCR_output/input.xlsx")
# pdf_path = os.path.abspath("D:/CommentCheck/test/140334_高利回り社債オープン_M_完成版.pdf")


# # Excel 애플리케이션 시작
# excel = win32com.client.Dispatch("Excel.Application")
# excel.Visible = False  # 백그라운드에서 실행

# try:
#     # 엑셀 파일 열기
#     workbook = excel.Workbooks.Open(excel_path)
    
#     # PDF로 저장
#     workbook.ExportAsFixedFormat(0, pdf_path)  # 0은 PDF 형식을 의미
#     print(f"PDF 파일이 생성되었습니다: {pdf_path}")

# finally:
#     # 파일 닫기 및 Excel 종료
#     workbook.Close(SaveChanges=False)
#     excel.Quit()

import win32com.client as win32
import os
from win32com.client import constants

def excel_to_pdf(excel_path, pdf_path,sheet_name="PAGE002", target_text="3.02%"):
    """
    Excel을 PDF로 변환하는 함수
    :param excel_path: 변환할 엑셀 파일의 전체 경로
    :param pdf_path: 내보낼 PDF 파일의 전체 경로
    """
    # 경로가 유효한지 확인(선택사항)
    excel_path = os.path.abspath(excel_path)
    pdf_path   = os.path.abspath(pdf_path)
    
    # Excel Application 실행
    excel = win32.Dispatch("Excel.Application")
    # excel.Visible = True  # 디버깅 시 엑셀창을 확인하고 싶다면 True로 설정

    try:
        # 엑셀 파일 열기
        workbook = excel.Workbooks.Open(excel_path)

        # 특정 시트 선택
        ws = workbook.Sheets(sheet_name)
        ws.Select()  # 해당 시트를 활성화(선택)

        # [1] "3.02%" 텍스트를 찾고 폰트 색상 빨간색으로 설정하기
        found = False  # 변경 여부 확인을 위한 플래그
        # [방법 1] Find 메서드 사용
        try:
            first_found = ws.Cells.Find(
                What=target_text,
                After=ws.Cells(ws.Rows.Count, ws.Columns.Count),
                LookIn=constants.xlValues,
                LookAt=constants.xlWhole,  # xlWhole 또는 xlPart
                SearchOrder=constants.xlByRows,
                SearchDirection=constants.xlNext,
                MatchCase=False
            )

            if first_found:
                print(f"Found '{target_text}' at {first_found.Address}")
                first_address = first_found.Address
                current_range = first_found
                while True:
                    # Font.Color을 사용하여 RGB 값으로 빨간색 설정 (255는 빨간색)
                    current_range.Font.Color = 255  # RGB 빨간색
                    print(f"Changed color of cell {current_range.Address} to red.")
                    found = True

                    # 다음 위치 찾기
                    current_range = ws.Cells.FindNext(current_range)
                    if not current_range or current_range.Address == first_address:
                        break
        except Exception as find_e:
            print(f"Find 메서드 중 오류 발생: {find_e}")

        # [방법 2] Find 메서드로 찾지 못한 경우 모든 셀을 반복하여 확인
        if not found:
            print("Find 메서드로 찾지 못했으므로, 모든 셀을 반복하여 확인합니다.")
            used_range = ws.UsedRange
            for row in range(1, used_range.Rows.Count + 1):
                for col in range(1, used_range.Columns.Count + 1):
                    cell = ws.Cells(row, col)
                    cell_value = cell.Value
                    cell_text = cell.Text

                    # 텍스트로 "3.02%"인 경우
                    if cell_text == target_text:
                        cell.Font.Color = 255  # 빨간색
                        print(f"Changed color of cell {cell.Address} to red (텍스트 매칭).")
                        found = True

                    # 값이 0.0302이고 퍼센트 형식인 경우
                    elif isinstance(cell_value, float) and cell_value == 0.0302:
                        # 셀의 NumberFormat이 퍼센트 형식인지 확인
                        if '%' in cell.NumberFormat:
                            cell.Font.Color = 255  # 빨간색
                            print(f"Changed color of cell {cell.Address} to red (값 매칭).")
                            found = True

        if not found:
            print(f"'{target_text}'를 포함하는 셀을 찾지 못했습니다.")

        # [2] 시트 → PDF로 내보내기
        # PDF로 내보내기 (0은 PDF 형식)
        ws.ExportAsFixedFormat(
            Type=0,            # 0=PDF, 1=XPS
            Filename=pdf_path,
            Quality=0,         # 기본값(표준) 또는 1=최적(고품질)
            IncludeDocProperties=True,
            IgnorePrintAreas=False,
            OpenAfterPublish=False
        )

        print(f"PDF 파일이 생성되었습니다: {pdf_path}")
    
    except Exception as e:
        print(f"PDF 변환 중 오류가 발생했습니다: {e}")
    
    finally:
        # Workbook 닫기 & Excel 종료
        workbook.Close(SaveChanges=False)
        excel.Quit()

# 예시 사용법
if __name__ == "__main__":

    
    excel_file_path = os.path.abspath("D:/CommentCheck/test/OCR_output/input.xlsx")
    output_pdf_path = os.path.abspath("D:/CommentCheck/test/140334_高利回り社債オープン_M_完成版.pdf")
    sheet_to_export = "PAGE002"                        # PDF로 내보낼 시트 이름

    excel_to_pdf(excel_file_path, output_pdf_path,sheet_to_export, target_text="3.02%")

```


4. 작동되는 API

```

@app.route('/ask_gpt', methods=['POST'])
def ask_gpt():
    try:
        data = request.json
        prompt = data.get("input", "")

        if not prompt:
            return jsonify({"success": False, "error": "No input provided"}), 400
        
        prompt_result = f"""
        You are a professional Japanese text proofreading assistant. Please carefully proofread the content of a Japanese report following the rules below:

        **Report Content to Proofread:**
        {prompt}

        **Proofreading Requirements:**
        1. **Check for typos and missing characters (誤字脱字がないこと):**
        - Ensure there are no spelling errors or missing characters in the content of the report.

        2. **Follow the "Fund Manager Comment Terminology Guide" (ファンドマネージャコメント用語集に沿った記載となっていること):**
        - **Consistent Terminology (表記の統一):**
            - Ensure the writing format of terms is consistent throughout the report.
        - **Prohibited Words and Phrases (禁止（NG）ワード及び文章の注意事項):**
            - Check if any prohibited words or phrases are used in the report and correct them as per the guidelines.
        - **Replaceable and Recommended Terms/Expressions (置き換えが必要な用語/表現、置き換えを推奨する用語/表現):**
            - If you find terms or expressions that need to be replaced, revise them according to the provided rules.
        - **Use of Hiragana (ひらがなを表記するもの):**
            - Ensure the report follows the rules for hiragana notation, replacing content that does not conform to commonly used kanji.
        - **Kana Notation for Non-Standard Kanji (一部かな書き等で表記するもの):**
            - Ensure non-standard kanji are replaced with kana as the standard writing format.
        - **Correct Usage of Okurigana (一般的な送り仮名など):**
            - Ensure the correct usage of okurigana is applied.
        - **English Abbreviations, Loanwords, and Technical Terms (英略語、外来語、専門用語など):**
            - Check if English abbreviations, loanwords, and technical terms are expressed correctly.

        3. **Consistency with Report Data Section (レポートのデータ部との整合性確認):**
        - Ensure the textual description in the report is completely consistent with the data section, without any logical or content-related discrepancies.

        **Special Rules:**
        1. **Do not modify proper nouns (e.g., names of people, places, or organizations) unless they are clearly misspelled.**
        2. **Remove unnecessary or redundant text instead of replacing it with other characters.**

        **Output Requirements:**
        1. **Highlight corrections in red and include additional details:**
        - For corrected parts:
            - Highlight the incorrected text in red using `<span style="color:red;">`.
            - Append the original incorrect text in parentheses, marked with a strikethrough using `<s>` tags.
            - Provide the reason for the correction and indicate the change using the format `123 → 456`.
            - Example:
            `<span style="color:red;">不一致</span> (<span style="background:yellow;">修正理由: 一致性不足 <s>123</s> → 456</span>)`

        2. **Preserve the original structure and formatting of the document:**
        - Maintain paragraph breaks, headings, and any existing structure in the content.

        3. **Use the uploaded correction rules for reference:**
        - {prompt}

        4. **Do not provide any explanations or descriptions in the output. Only return the corrected HTML content.**
        """     
        # ChatCompletion Call
        response = openai.ChatCompletion.create(
            deployment_id=deployment_id,  # Deploy Name
            messages=[
                {"role": "system", "content": "You are a professional Japanese text proofreading assistant."},
                {"role": "user", "content": prompt_result}
            ],
            max_tokens=2000,
            temperature=0.1,
            seed=42  # 재현 가능한 결과를 위해 seed 설정
        )
        answer = response['choices'][0]['message']['content'].strip()
        re_answer = remove_code_blocks(answer)
        return jsonify({"success": True, "answer": re_answer})
    except Exception as e:
        return jsonify({"success": False, "error": str(e)}), 500

```


# back

```
# PDF 파일에서 텍스트 추출
# def extract_text_from_pdf(pdf_path):
#     """
#     PDF 파일에서 텍스트를 추출합니다.
#     """
#     doc = fitz.open(pdf_path)
#     text = ""
#     for page in doc:
#         text += page.get_text()
#     return text

# # PDF 파일에 코멘트 추가
# def add_comments_to_pdf(pdf_bytes, corrections):
#     """
#     PDF 파일에서 틀린 부분을 찾아 코멘트를 추가합니다.
#     """
#     # PDF 파일 열기 (BytesIO에서 직접 열기)
#     doc = fitz.open(stream=pdf_bytes, filetype="pdf")
#     for correction in corrections:
#         page_num = correction["page"]
#         original_text = correction["original_text"]
#         comment = correction["comment"]

#         page = doc[page_num]
#         text_instances = page.search_for(original_text)

#         for inst in text_instances:
#             rect = fitz.Rect(inst)  # 틀린 부분의 위치
#             annot = page.add_text_annot(rect.top_left, comment)  # 코멘트 추가
#             annot.set_colors(stroke=(1, 0, 0))  # 코멘트 색상 설정 (빨간색)
#             annot.update()  # 주석 업데이트

#     # 수정된 PDF를 BytesIO에 저장
#     output = io.BytesIO()
#     doc.save(output)
#     output.seek(0)
#     doc.close()

# # 틀린 부분 찾기 (processed_text를 활용)
# def find_corrections(processed_text):
#     """
#     GPT-4의 응답 결과(processed_text)를 분석하여 틀린 부분을 찾습니다.
#     """
#     corrections = []
#     # 예시: processed_text에서 틀린 부분과 수정 이유를 추출
#     # 예를 들어, processed_text가 다음과 같은 형식이라면:
#     # <span style="color:red;">委</span> (<span style="background:yellow;">修正理由: 誤字 委 → の</span>)
#     # 이를 분석하여 corrections 리스트를 생성합니다.
#     import re
#     pattern = r'<span style="color:red;">(.*?)</span>.*?修正理由: (.*?)</span>'
#     matches = re.findall(pattern, processed_text)
#     for match in matches:
#         original_text = match[0]  # 틀린 부분
#         comment = f"修正理由: {match[1]}"  # 수정 이유
#         corrections.append({
#             "page": 0,  # 페이지 번호 (0부터 시작, 필요 시 수정)
#             "original_text": original_text,
#             "comment": comment,
#         })
#     return corrections

# @app.route('/api/check_upload', methods=['POST'])
# def check_upload_file():
#     logging.info("업로드 요청 수신")
#     logging.info(f"request.files keys: {request.files.keys()}")  # 전달된 키 로그 출력

#     file = request.files['file']

#     if 'file' not in request.files:
#         logging.warning("No file part in the request")
#         return jsonify({"message": "No file part"}), 400
    
#     # 파일을 메모리에서 읽기
#     pdf_bytes = file.read()

#     files = request.files.getlist('file')  # 여러 파일 동시에 업로드 가능
#     uploaded_files = []

#     for file in files:
#         try:
#             if file.filename == '':
#                 logging.warning("No selected file")
#                 return jsonify({"success": False, "error": "No selected file"}), 400
            
#             if file and allowed_file(file.filename):
#                 # PDF 파일 읽기
#                 reader = PdfReader(file)
#                 text = ""
#                 for page in reader.pages:
#                     text += page.extract_text()

#                 # GPT 모델을 사용하여 텍스트 처리
#                 prompt_result = f"""
#                 You are a professional Japanese text proofreading assistant. Please carefully proofread the content of a Japanese report following the rules below:

#                 **Report Content to Proofread:**
#                 {text}

#                 **Proofreading Requirements:**
#                 1. **Check for typos and missing characters (誤字脱字がないこと):**
#                 - Ensure there are no spelling errors or missing characters in the content of the report.

#                 2. **Follow the "Fund Manager Comment Terminology Guide" (ファンドマネージャコメント用語集に沿った記載となっていること):**
#                 - **Consistent Terminology (表記の統一):**
#                     - Ensure the writing format of terms is consistent throughout the report.
#                 - **Prohibited Words and Phrases (禁止（NG）ワード及び文章の注意事項):**
#                     - Check if any prohibited words or phrases are used in the report and correct them as per the guidelines.
#                 - **Replaceable and Recommended Terms/Expressions (置き換えが必要な用語/表現、置き換えを推奨する用語/表現):**
#                     - If you find terms or expressions that need to be replaced, revise them according to the provided rules.
#                 - **Use of Hiragana (ひらがなを表記するもの):**
#                     - Ensure the report follows the rules for hiragana notation, replacing content that does not conform to commonly used kanji.
#                 - **Kana Notation for Non-Standard Kanji (一部かな書き等で表記するもの):**
#                     - Ensure non-standard kanji are replaced with kana as the standard writing format.
#                 - **Correct Usage of Okurigana (一般的な送り仮名など):**
#                     - Ensure the correct usage of okurigana is applied.
#                 - **English Abbreviations, Loanwords, and Technical Terms (英略語、外来語、専門用語など):**
#                     - Check if English abbreviations, loanwords, and technical terms are expressed correctly.

#                 3. **Consistency with Report Data Section (レポートのデータ部との整合性確認):**
#                 - Ensure the textual description in the report is completely consistent with the data section, without any logical or content-related discrepancies.

#                 **Special Rules:**
#                 1. **Do not modify proper nouns (e.g., names of people, places, or organizations) unless they are clearly misspelled.**
#                 2. **Remove unnecessary or redundant text instead of replacing it with other characters.**

#                 **Output Requirements:**
#                 1. **Highlight corrections in red and include additional details:**
#                 - For corrected parts:
#                     - Highlight the incorrected text in red using `<span style="color:red;">`.
#                     - Append the original incorrect text in parentheses, marked with a strikethrough using `<s>` tags.
#                     - Provide the reason for the correction and indicate the change using the format `123 → 456`.
#                     - Example:
#                     `<span style="color:red;">不一致</span> (<span style="background:yellow;">修正理由: 一致性不足 <s>123</s> → 456</span>)`

#                 2. **Preserve the original structure and formatting of the document:**
#                 - Maintain paragraph breaks, headings, and any existing structure in the content.

#                 3. **Use the uploaded correction rules for reference:**
#                 - {text}

#                 4. **Do not provide any explanations or descriptions in the output. Only return the corrected HTML content.**
#                 """

#                 # OpenAI API 호출
#                 response = openai.ChatCompletion.create(
#                     deployment_id=deployment_id,  # Deploy Name
#                     messages=[
#                         {"role": "system", "content": "You are a professional Japanese text proofreading assistant."},
#                         {"role": "user", "content": prompt_result}
#                     ],
#                     max_tokens=2000,
#                     temperature=0,
#                     seed=42  # 재현 가능한 결과를 위해 seed 설정
#                 )
#                 corrected_text = response['choices'][0]['message']['content'].strip()

#                 # 틀린 부분 찾기 (processed_text를 활용)
#                 corrections = find_corrections(corrected_text)

#                 # PDF에 코멘트 추가
#                 output_pdf = add_comments_to_pdf(pdf_bytes, corrections)

#                 # PDF 파일 다운로드
#                 return send_file(
#                     output_pdf,
#                     mimetype='application/pdf',
#                     as_attachment=True,
#                     download_name='corrected_report.pdf'
#                 )

#         except Exception as e:
#             logging.error(f"Error processing file {file.filename}: {str(e)}")
#             return jsonify({"success": False, "error": str(e)}), 500

#     return jsonify({"success": True, "message": uploaded_files})

```



```

{
  "codingcopilot.enableAutoCompletions": true,
  "RainbowBrackets.depreciation-notice": false,
  "security.workspace.trust.untrustedFiles": "open",
  "liveServer.settings.donotShowInfoMsg": true,
  "explorer.confirmDelete": false,
  "python.createEnvironment.trigger": "off",
  "liveServer.settings.ChromeDebuggingAttachment": false,
  "cmake.showOptionsMovedNotification": false,
  "editor.unicodeHighlight.ambiguousCharacters": false,
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "http.proxy": "http://80023:nomura50@172.19.248.1:92",
  "git.ignoreMissingGitWarning": true,
  "http.proxyAuthorization": null,
  "http.proxyStrictSSL": false,
  "git.autofetch": true,
  "[html]": {
    "editor.suggest.insertMode": "replace"
  },
  "azureDatabases.useCosmosOAuth": true
}

```


```가운데 분할 세로선

/* 메시지 스타일 */
.message {
    margin-bottom: 1rem;
    padding: 0.75rem 1rem;
    border-radius: 10px;
    max-width: 80%; /* 메시지의 최대 너비를 제한 */
    word-wrap: break-word; /* 긴 텍스트가 넘치지 않도록 처리 */
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* 그림자 추가 */
}

/* 사용자 메시지 스타일 */
.message.user {
    background-color: #e3f2fd; /* 사용자 메시지 배경색 */
    margin-left: auto; /* 우측 정렬 */
    text-align: right;
    border-bottom-right-radius: 0; /* 우측 하단 모서리 둥글게 */
}

/* 봇 메시지 스타일 */
.message.bot {
    background-color: #f5f5f5; /* 봇 메시지 배경색 */
    margin-right: auto; /* 좌측 정렬 */
    text-align: left;
    border-bottom-left-radius: 0; /* 좌측 하단 모서리 둥글게 */
}

/* 입력 및 출력 컨테이너 스타일 */
.input-container,
.output-container {
    flex: 1;
    padding: 1rem;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    gap: 0.75rem; /* 메시지 간격 */
}

/* 입력 컨테이너 스타일 */
.input-container {
    border-right: 1px solid #ddd; /* 좌우 구분선 */
}

/* 푸터 스타일 */
#footer {
    background-color: #f8f9fa;
    padding: 1rem;
    display: flex;
    align-items: center;
    border-top: 1px solid #ddd;
}

#footer textarea {
    flex: 1;
    resize: none;
    border-radius: 10px;
    padding: 0.75rem;
    height: 100px; /* 높이 조정 */
    border: 1px solid #ddd; /* 테두리 추가 */
}

#sendButton {
    margin-left: 0.5rem;
    padding: 0.5rem 1rem;
    border-radius: 10px; /* 버튼 둥글게 */
    background-color: #17494d; /* 버튼 색상 */
    color: white; /* 버튼 텍스트 색상 */
    border: none; /* 테두리 제거 */
}

#sendButton:hover {
    background-color: #0d2e31; /* 호버 시 색상 변경 */
}

/* 헤더 스타일 */
#header {
    background-color: #17494d;
    padding: 1rem;
    display: flex;
    align-items: center;
}

.header-title {
    font-size: 1.5rem;
    color: white;
    margin-left: 1rem;
}

```

```js
const API_URL = 'https://namcheckweb.azurewebsites.net/ask_gpt'; // GPT API URL
const UPLOAD_API_URL = 'https://namcheckweb.azurewebsites.net/api/check_upload'; // 파일 업로드 API URL

// 텍스트를 청크로 분할하는 함수
function split_text(text, max_tokens = 1000) {
    const words = text.split(/\s+/); // 텍스트를 단어 단위로 분할
    const chunks = []; // 분할된 청크를 저장할 배열
    let current_chunk = []; // 현재 청크
    let current_length = 0; // 현재 청크의 토큰 수

    for (const word of words) {
        // 현재 단어를 추가했을 때 max_tokens를 초과하는지 확인
        if (current_length + word.length + 1 <= max_tokens) {
            current_chunk.push(word);
            current_length += word.length + 1; // 단어 길이 + 공백
        } else {
            // 현재 청크를 완성하고 청크 리스트에 추가
            chunks.push(current_chunk.join(' '));
            current_chunk = [word]; // 새로운 청크 시작
            current_length = word.length; // 현재 청크의 토큰 수 초기화
        }
    }

    // 마지막 청크가 남아있는 경우 추가
    if (current_chunk.length > 0) {
        chunks.push(current_chunk.join(' '));
    }

    return chunks;
}

document.getElementById('sendButton').addEventListener('click', sendMessage);
document.getElementById('userInput').addEventListener('keypress', function (e) {
    if (e.key === 'Enter') {
        e.preventDefault();
        sendMessage();
    }
});

function sendMessage() {
    const userInput = document.getElementById('userInput').value.trim();
    if (!userInput) return;

    // 좌측: 사용자 입력 추가
    addMessage(userInput, 'user', 'inputContainer');
    document.getElementById('userInput').value = '';

    // Fetch bot response from /api/ppt
    fetchBotResponse(userInput);
}

// 첨부파일 업로드 기능 추가
document.querySelector('.f02f0e25').addEventListener('click', function () {
    // 파일 선택기 생성
    const fileInput = document.createElement('input');
    fileInput.type = 'file';
    fileInput.accept = 'application/pdf'; // PDF 파일만 허용

    fileInput.onchange = async (event) => {
        const file = event.target.files[0];
        if (file) {
            const formData = new FormData();
            formData.append('file', file);

            try {
                // 1. 파일 업로드 및 원본 텍스트 가져오기
                const uploadResponse = await fetch(UPLOAD_API_URL, {
                    method: 'POST',
                    body: formData
                });

                if (!uploadResponse.ok) {
                    throw new Error(`サーバーエラー: ${uploadResponse.status} ${uploadResponse.statusText}`);
                }

                const uploadResult = await uploadResponse.json();
                if (!uploadResult.success) {
                    throw new Error(uploadResult.error || 'ファイルの処理中にエラーが発生しました。');
                }

                // 좌측: PDF에서 읽어온 원본 텍스트 표시
                if (uploadResult.original_text) {
                    addMessage(uploadResult.original_text, 'user', 'inputContainer');
                } else {
                    throw new Error('PDF 텍스트를 읽어오지 못했습니다.');
                }

                // 2. 원본 텍스트를 ask_gpt API로 전달 (토큰 수 제한 적용)
                const textChunks = split_text(uploadResult.original_text, 2000); // 텍스트를 2000 토큰 단위로 분할
                const gptResults = []; // GPT 모델의 처리 결과를 저장할 배열

                for (const chunk of textChunks) {
                    try {
                        const gptResponse = await fetch(API_URL, {
                            method: 'POST',
                            headers: {
                                'Content-Type': 'application/json'
                            },
                            body: JSON.stringify({ input: chunk }) // 청크 단위로 전달
                        });

                        if (!gptResponse.ok) {
                            throw new Error(`GPT APIエラー: ${gptResponse.status} ${gptResponse.statusText}`);
                        }

                        const gptResult = await gptResponse.json();
                        if (gptResult.answer) {
                            gptResults.push(gptResult.answer); // 결과 저장
                        } else {
                            throw new Error('GPT 모델の処理結果を取得できませんでした。');
                        }
                    } catch (error) {
                        console.error('GPT API 호출 중 에러가 발생했습니다:', error);
                        addMessage(`GPT API 호출 중 에러가 발생했습니다: ${error.message}`, 'bot', 'outputContainer');
                        return; // 에러 발생 시 중단
                    }
                }

                // 모든 청크의 결과를 하나로 합침
                const finalAnswer = gptResults.join('\n\n');

                // 우측: GPT 모델의 처리 결과 표시
                addMessage(finalAnswer, 'bot', 'outputContainer');
            } catch (error) {
                console.error('ファイルのアップロード中にエラーが発生しました:', error);
                addMessage(`ファイルのアップロード中にエラーが発生しました: ${error.message}`, 'bot', 'outputContainer');
            }
        }
    };

    fileInput.click(); // 파일 선택기 열기
});

function addMessage(content, sender, containerId) {
    const messageContainer = document.createElement('div');
    messageContainer.classList.add('message', sender);

    // content가 undefined 또는 null인 경우 빈 문자열로 대체
    const safeContent = content || '';

    // 줄바꿈을 <br> 태그로 변환
    const formattedContent = safeContent.replace(/\n/g, '<br>');

    messageContainer.innerHTML = sender === 'user' 
        ? `<strong>あなた:</strong><div>${formattedContent}</div>` 
        : `<strong>Nomura AI:</strong><div>${formattedContent}</div>`;

    document.getElementById(containerId).appendChild(messageContainer);

    // 스크롤을 최신 메시지로 이동
    const container = document.getElementById(containerId);
    container.scrollTop = container.scrollHeight;

    // 우측에도 동일한 높이의 빈 메시지 추가 (라인 맞추기)
    if (sender === 'user') {
        const emptyMessage = document.createElement('div');
        emptyMessage.classList.add('message', 'empty');
        emptyMessage.style.visibility = 'hidden'; // 보이지 않도록 처리
        document.getElementById('outputContainer').appendChild(emptyMessage);
    }
}

async function fetchBotResponse(userInput) {
    try {
        const response = await fetch(API_URL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ input: userInput })
        });
        const data = await response.json();

        // 우측: GPT 출력 추가
        addMessage(data.answer || '回答が見つかりませんでした。', 'bot', 'outputContainer');
    } catch (error) {
        console.error('Error fetching response:', error);
        addMessage('エラーが発生しました。もう一度お試しください。', 'bot', 'outputContainer');
    }
}


```


## 2025-01-22
```prompt
prompt_result = f"""
        You are a professional Japanese text proofreading assistant. Please carefully proofread the content of a Japanese report following the rules below:

        **Report Content to Proofread:**
        {prompt}

        **Proofreading Requirements:**
        1. **Check for typos and missing characters (誤字脱字がないこと):**
        - Ensure there are no spelling errors or missing characters in the content of the report.

        2. **Follow the "Fund Manager Comment Terminology Guide" (ファンドマネージャコメント用語集に沿った記載となっていること):**
        - **Consistent Terminology (表記の統一):**
            - Ensure the writing format of terms is consistent throughout the report.
        - **Prohibited Words and Phrases (禁止（NG）ワード及び文章の注意事項):**
            - Check if any prohibited words or phrases are used in the report and correct them as per the guidelines.
        - **Replaceable and Recommended Terms/Expressions (置き換えが必要な用語/表現、置き換えを推奨する用語/表現):**
            - If you find terms or expressions that need to be replaced, revise them according to the provided rules.
        - **Use of Hiragana (ひらがなを表記するもの):**
            - Ensure the report follows the rules for hiragana notation, replacing content that does not conform to commonly used kanji.
        - **Kana Notation for Non-Standard Kanji (一部かな書き等で表記するもの):**
            - Ensure non-standard kanji are replaced with kana as the standard writing format.
        - **Correct Usage of Okurigana (一般的な送り仮名など):**
            - Ensure the correct usage of okurigana is applied.
        - **English Abbreviations, Loanwords, and Technical Terms (英略語、外来語、専門用語など):**
            - Check if English abbreviations, loanwords, and technical terms are expressed correctly.
        - **Layout and Formatting (レイアウトに関する統一):**
            - Font and Format (書式):
                Use MSP Mincho 10.5-point font for the main text.
            - Uniform Spacing for Bullet Points (文頭の「○」印と一文字目の間隔を統一):
                Ensure consistent spacing between the "○" symbol and the first character of the text. Choose either tight or loose spacing and apply it uniformly across the entire page.
            - Uniform Paragraph Spacing (文章の間隔の統一):
                Maintain consistent spacing for paragraphs starting with "○" within the same text box.
            - Date for "今後の運用方針" (「今後の運用方針」作成日付):
                Use the last business day of the previous month as the creation date. If the date falls on the first day of the next month, use the actual creation date. Do not include future dates (e.g., dates after sending to the Client Service Department).
            - Adjustment for "上位10銘柄" Comment Boxes (「上位10銘柄」コメント欄の枠調整):
                Ensure that comments for top 10 holdings fit within the text box. If the text is too long, adjust the box to accommodate it. Recheck and adjust the layout if the ranking changes in the following month, as this is often overlooked.
        - **Consistent Terminology (表記の統一):**
            - Fund NAV Fluctuation (基準価額の騰落率):
                - Avoid "0.00％となり": Do not use phrases like "0.00％となり." Instead, use "前月末から変わらず" or "前月末と同程度" (e.g., "基準価額（分配金再投資）は前月末から変わらず").
                - Rounding Rules: Round to two decimal places (e.g., round 0.125％ to 0.13％). Do not use more than two decimal places.
                - Comparison with Benchmark: When comparing the fund's NAV fluctuation with the benchmark or reference index, use the rounded figures from rule ②. If the fund and benchmark have the same fluctuation rate, state "騰落率は同程度となりました" (e.g., "ファンドとベンチマークの騰落率は同程度となりました").
            - Examples of Correct Usage:
                - Correct:
                    "基準価額（分配金再投資）は前月末から変わらず。"
                    "基準価額は前月末と同程度でした。"
                - Incorrect:
                    "基準価額の騰落率は0.00％となりました。"
                    "騰落率は変わらずでした。"
                - Benchmark Comparison:
                    "ファンドの騰落率は0.13％、ベンチマークの騰落率は0.12％でした。"
                    "ファンドとベンチマークの騰落率は同程度となりました。"               
            - Percentage Notation (％): Use 全角 for percentages (％).
                - Numbers, Alphabets, and Symbols (数字、アルファベット、「＋」・「－」): Use 半角 for numbers, alphabets, and symbols (＋, －).
                - Asterisk (※): Use superscript for asterisks (※) in notes, except for system-generated content.
                - Parentheses (カッコ書き): Use parentheses only for the first occurrence of a comment. For example, if a term is explained in the "投資環境" section on page 1, do not repeat the explanation in the "組入上位10銘柄の解説" section on page 2. Follow the disclosure page numbering rather than the order of sheets in the comment file.
                - Katakana Notation (カタカナ表記): Ensure consistency in katakana notation for loanwords (e.g., サステナブル/サスティナブル, エンターテイメント/エンターテインメント).
                - Economic Indicators (経済指標について):
                - Specify the Month: Include the month for economic indicators (e.g., "10月の製造業PMI（購買担当者景気指数）は～"). Add the country name if necessary.
                - Clarify Forecasts: Clearly state whether forecasts are based on PM's expectations or vendor predictions (e.g., "PMの予想" or "情報ベンダー等の予測").
                - Acceleration Description: When using phrases like "前月から加速しました," specify what is accelerating (e.g., "前月から上昇が加速しました"). Avoid ambiguous terms like "経済加速"; use "景気加速" instead.                   
            - Examples of Correct Usage
                Percentage Notation (％):
                    Correct: "利回りは5％上昇しました。"
                    Incorrect: "利回りは5%上昇しました。"
                Numbers and Symbols (数字、アルファベット、「＋」・「－」):
                    Correct: "前月比+3.0％"
                    Incorrect: "前月比＋3.0％"
                Asterisk (※):
                    Correct: "※注: データは四捨五入されています。"
                    Incorrect: "※ 注: データは四捨五入されています。"
                Parentheses (カッコ書き):
                    Correct (First Occurrence): "投資環境（米国債券利回りは上昇）は～"
                    Correct (Subsequent Mention): "米国債券利回りは上昇しました。"
                    Incorrect: "投資環境（米国債券利回りは上昇）は～。その後、米国債券利回り（上昇）は～"
                Katakana Notation (カタカナ表記):
                    Correct: "サステナブル投資が増加しています。"
                    Also Correct: "サスティナブル投資が増加しています。"
                    Incorrect: "サステナブル投資とサスティナブル投資が混在しています。"
                Economic Indicators (経済指標について):
                    Correct: "10月の製造業PMI（購買担当者景気指数）は50.8と市場予想を上回りました。"
                    Correct: "PMの予想によると、デフォルト率は低下する見込みです。"
                    Correct: "前月から上昇が加速しました。"
                    Incorrect: "前月から加速しました。"
                    Incorrect: "経済加速が進んでいます。"
            - Fiscal Periods (会計期間について):
                - Hyphen Usage: Use hyphens (-) instead of tildes (～) for date ranges (e.g., "6-8月期" instead of "6～8月期").
                - Calendar Year Notation: For countries using the calendar year, add parentheses (e.g., "ブラジルの2021年度（2021年1月-12月）予算").
                - Fiscal Year Notation: Use "yyyy年度" for fiscal years (e.g., "2021年度" instead of "今年度" or "来年度").
                - Quarterly Notation: Use "●-●月期" for quarterly periods, even for irregular cases (e.g., "6-8月期" instead of "第1四半期（5月21日～8月20日）").
            - Year Notation (年表記について):
                - 4-Digit Years: Use 4-digit years (e.g., "2022年" instead of "22年").
                - Year-over-Year Comparisons: Use 前年同月比 or 前年同期比 instead of 前年比, except for annual comparisons (e.g., "2023年のGDPは前年比+3.0％となり～").
                - Cross-Year Economic Indicators: Mention the year only for the first occurrence of cross-year economic indicators (e.g., "2023年12月のCPIは～。一方2024年1月のユーロ圏PMIは～、10-12月期のGDPは～").
            - Range Notation (レンジの表記について): Include ％ in range notations (e.g., "-1％～0.5％" instead of "-1～0.5％").
            - Investment Environment (投資環境について):
                - US References: When referring to the US in the context of other regions, use "米" (e.g., "トランプ米大統領" or "米トランプ新政権").
            - Previous Month References: Use "前月末" or "●月末" instead of "先月末" when referring to the previous month.
            - Redemption Notices (償還に関する記載): Include redemption notices one month before the final release (e.g., "当ファンドは、●●月●●日に信託の終了日（償還日or繰上償還日）を迎える予定です").
            - Country Names (国名の表記統一):
                - 米国 (アメリカ), 英国 (イギリス), 豪州 (オーストラリア): Use 漢字二文字 for the first occurrence (e.g., "米国株式市場"), then 漢字一文字 for subsequent mentions (e.g., "米株式市場"). Use 漢字一文字 when the country name is used as a modifier (e.g., "英CPI" instead of "イギリスのCPI").
            - フランス and ドイツ: Use katakana (e.g., "フランス株式市場"), except when used as a modifier (e.g., "独CPI" instead of "ドイツCPI").
            - Individual Company Names (個別企業名の表記): Avoid using individual company names in investment environment comments (e.g., use "スイスの大手金融グループ" instead of "クレディ・スイス").
            - Positive/Negative Impact (プラスに寄与/影響, マイナスに影響):
                - Use "プラスに寄与" or "プラスに影響" for positive impacts. Alternatively, use "～プラス要因となる".
                - Use "マイナスに影響" for negative impacts. Avoid using "マイナスに寄与". Alternatively, use "～マイナス要因となる".
            - Yield and Price Movements (利回りは上昇/低下, 低下と下落):
                - Use "利回りは上昇" (price falls) and "利回りは低下" (price rises).
                - Use "価格は下落" for price falls and "価格は上昇" for price rises.
            - Capital Flows (資金流出入): Use "外国人投資家の資金流出" or "外国人投資家からの資金流入".
            - Portfolio (ポートフォリオ): Use "●●の組み入れ" instead of "●●への組み入れ" (e.g., "米国債の組み入れ").
            - Geopolitical and Political Risks (地政学的リスク, 政治リスク):
                - Use "地政学的リスク" instead of "地政学リスク".
                - Use "政治リスク" instead of "政治的リスク".
            - Currency Notation (通貨表記の統一): Ensure consistent currency notation throughout the report.
            - Zero Percentage Notation (構成比の0％の表記): Use "0％程度" or "ゼロ％程度".
            - Central Banks (日銀, 中央銀行):
                - Use "日銀" instead of "日本銀行".
                - Use "中央銀行" as the standard term.
            - Yield Curve Terminology (利回り曲線関連):
                - Use "伸長" instead of "伸張".
                - Use "立ち遅れ" instead of "立ち後れ".
                - Use "沈静" for natural calming and "鎮静" for artificial calming.
            - Normalization (～正常化): Use "経済活動正常化" instead of "経済正常化".
            - US Treasury Bonds (米国国債と米国債の表記): Use "米国債" as the standard term.
            - COVID-19 (新型コロナウイルス): Avoid using "コロナウイルス" or "新型コロナ". Use "コロナ禍", "コロナショック", or "新型コロナ禍" as exceptions.
            - Russia-Ukraine Conflict (ロシアによるウクライナへの軍事侵攻): Avoid using terms like "ロシアとウクライナとの戦争".
            - Benchmark and Index Names (ベンチマーク・インデックス・参考指数の名称の表記): Ensure consistent notation for benchmarks and indices (e.g., "MSCIインド指数" → "MSCIインド・インデックス").
            - Dow and TOPIX (ダウ平均, TOPIX):
                - Use "ダウ平均株価" for the Dow Jones Industrial Average.
                - Use "東証株価指数（TOPIX）（配当込み）" for TOPIX.             

        - ** Prohibited Words and Phrases (禁止（NG）ワードおよび文章の注意事項):**
            - 魅力的な (魅力が入る言葉):
                - Avoid using terms like "魅力的な" unless justified with evidence.
                - Direct use for funds is prohibited (e.g., "このファンドは魅力的です" is not allowed).
            - 投資妙味（がある）:
                - Avoid using "投資妙味" unless justified with evidence.
                - Direct use for funds is prohibited (e.g., "このファンドには投資妙味がある" is not allowed).
            - 「割高（感）」「割安（感）」:
                - Avoid using "割高" or "割安" unless justified with evidence.
                - Direct use for funds is prohibited (e.g., "このファンドは割安です" is not allowed).
            - 「必ず～」「～になる。」「～である。」:
                - Avoid definitive expressions like "必ず～," "～になる," or "～である" without clear justification, especially when referring to future performance, economic indicators, or corporate earnings.  
            - 利益確定の売り and 利食い売り:
                - Avoid definitive expressions like "利益確定の売り" or "利食い売り." Instead, use phrases like "～が出たとの見方" (e.g., "利益確定の売りが出たとの見方" is acceptable).
            - 限定的:
                - Avoid using "限定的" as it is ambiguous regarding whether the effect or impact is positive or negative. Replace with clearer terms.

            - 「予想」、「心理」の使い方:
                - Clearly specify whose "予想" (forecast) or "心理" (sentiment) is being referred to. Use "市場予想" or "市場心理" to avoid ambiguity.

            - 企業の合併、業績悪化等に関する記述:
                - Avoid making statements about mergers, acquisitions, or performance declines without accurate information or factual confirmation.

            - 組入上位10銘柄以外の開示:
                - Do not disclose holdings beyond the top 10 holdings (including subsidiaries whose names could identify specific holdings), unless permitted by internal regulations.

            - 出所、出典:
                - When citing statistical data or reports, clearly indicate the source (e.g., report name, organization).
                - For publicly available statistics from government agencies or independent administrative institutions, include the source when using them in publications or advertisements.

            - 上昇要因・下落要因の記載:
                - Clearly state the reasons for increases or decreases in performance.
                - For funds with multiple courses, if only one course (rising or falling) is mentioned, focus on the course with the largest fluctuation. Alternatively, mention both causes.

            - リリース月に発生するイベントへの予測の記載:
                - When making predictions about events occurring in the release month, include the date the prediction was made (e.g., "当社は、日米の政治スケジュールや足もとの円安がインフレ率再上昇につながるリスクなどを勘案し、7月に追加利上げがあるとの見通しを維持しています。（2024年6月末時点）").   
            - Examples of Correct Usage:

            - 魅力的な:
                - Correct (with evidence): "この銘柄は魅力的な成長見通しを持っています。"
                - Incorrect: "このファンドは魅力的です。"
            - 投資妙味:
                - Correct (with evidence): "この銘柄には投資妙味があると考えられます。"
                - Incorrect: "このファンドには投資妙味がある。"
            - 割高/割安:
                - Correct (with evidence): "この銘柄は割安と評価されています。"
                - Incorrect: "このファンドは割安です。"
            - Definitive Expressions:
                - Correct: "今後の利上げが予想されます。"
                - Incorrect: "必ず利上げが行われます。"
            - 利益確定の売り/利食い売り:
                - Correct: "利益確定の売りが出たとの見方があります。"
                - Incorrect: "利益確定の売りが出ました。"
            - 限定的:
                - Correct: "影響は小さく、プラス要因となりました。"
                - Incorrect: "影響は限定的です。"
            - 予想/心理:
                - Correct: "市場予想によると、利上げが行われる見込みです。"
                - Incorrect: "予想では利上げが行われる見込みです。"
            - 企業の合併/業績悪化:
                - Correct: "業績悪化のリスクが指摘されています。"
                - Incorrect: "業績が悪化しました。"
            - 組入上位10銘柄以外の開示:
                - Correct: "組入上位10銘柄は以下の通りです。"
                - Incorrect: "組入上位15銘柄は以下の通りです。"
            - 出所、出典:
                - Correct: "データは日本銀行の統計によります。"
                - Incorrect: "データによります。"
            - 上昇要因・下落要因の記載:
                - Correct: "上昇の主な要因は米国債券利回りの低下でした。"
                - Incorrect: "上昇しました。"
            - リリース月に発生するイベントへの予測の記載:
                - Correct: "当社は、7月に追加利上げがあるとの見通しを維持しています。（2024年6月末時点）"
                - Incorrect: "7月に追加利上げがあると予想されます。"

        - **Replaceable Terms and Expressions (置き換えが必要な用語/表現):**

            - 先月/前月: Use "前月" when referring to the previous month (e.g., "前月の投資環境は～").
            - 前期比○％: Use "前期比年率○％" for year-over-year growth rates (e.g., "前期比年率+3.0％").
            - 第1四半期: Use "1-3月期" instead of "第1四半期" (e.g., "2018年1-3月期"). Avoid ambiguous notations like "18年第4四半期" → use "2018年10-12月期."
            - 蓋然性: Replace with "確率" or "可能性" (e.g., "蓋然性が高い" → "可能性が高い").
            - 商い: Replace with "出来高" or "取引" (e.g., "商いが活発" → "取引が活発").
            - 後倒し: Replace with "先送り", "延期", or "繰り下げ" (e.g., "後倒しになる" → "延期になる").
            - 薄商い: Replace with "取引量が少なく" or "活況でなく" (e.g., "薄商いが続く" → "取引量が少ない状況が続く").
            - ～に賭ける: Replace with "～を予想して" (e.g., "利上げに賭ける" → "利上げを予想して").
            - コア銘柄: Replace with "中核銘柄" or "コア（中核）銘柄" (e.g., "コア銘柄に投資" → "中核銘柄に投資").
            - 出遅れ感: Remove or rephrase as "～と考えます。" (e.g., "出遅れ感がある" → "遅れていると考えます。").
            - トリガー: Replace with "きっかけ" (e.g., "トリガーとなる" → "きっかけとなる").
            - ブルーチップ企業: Replace with "優良企業" (e.g., "ブルーチップ企業に投資" → "優良企業に投資").
            - 約○％程度: Avoid using "約" and "程度" together. Use either "約○％" or "○％程度" (e.g., "約5％程度" → "約5％" or "5％程度").
            - （割安に）放置: Replace with "割安感のある" (e.g., "割安に放置されている" → "割安感のある").
            - 横ばい: Use "横ばい" only when the fluctuation range is small. For larger fluctuations resulting in no net change, use "ほぼ変わらず" or "同程度となる" (e.g., "債券利回りは横ばい" → "債券利回りはほぼ変わらず").
            - 行って来い: Replace with "上昇（下落）したのち下落（上昇）" (e.g., "行って来いの動き" → "上昇したのち下落する動き").
            - まちまち: Replace with "異なる動き" (e.g., "まちまちでした" → "異なる動きとなりました").
            - 積極姿勢/消極姿勢: Replace with specific descriptions (e.g., "積極姿勢とした" → "長めとした").
            - 銘柄解説部分での注意事項:
                - For future predictions, clearly indicate the source if the information is from outside the company.
                - Avoid recommendation-like comments or definitive expressions without evidence.

            - Examples of Correct Usage:
            - 先月/前月:
                - Correct: "前月の投資環境は～"
                - Incorrect: "先月の投資環境は～"
            -前期比○％:
                - Correct: "前期比年率+3.0％"
                - Incorrect: "前期比+3.0％"
            - 第1四半期:
                - Correct: "2018年1-3月期"
                - Incorrect: "18年第1四半期"
            - 蓋然性:
                - Correct: "可能性が高い"
                - Incorrect: "蓋然性が高い"
            - 商い:
                - Correct: "取引が活発"
                - Incorrect: "商いが活発"
            - 後倒し:
                - Correct: "延期になる"
                - Incorrect: "後倒しになる"
            - 薄商い:
                - Correct: "取引量が少ない状況が続く"
                - Incorrect: "薄商いが続く"
            - ～に賭ける:
                - Correct: "利上げを予想して"
                - Incorrect: "利上げに賭ける"
            - コア銘柄:
                - Correct: "中核銘柄に投資"
                - Incorrect: "コア銘柄に投資"
            - 出遅れ感:
                - Correct: "遅れていると考えます。"
                - Incorrect: "出遅れ感がある"
            - トリガー:
                - Correct: "きっかけとなる"
                - Incorrect: "トリガーとなる"
            - ブルーチップ企業:
                - Correct: "優良企業に投資"
                - Incorrect: "ブルーチップ企業に投資"
            - 約○％程度:
                - Correct: "約5％" or "5％程度"
                - Incorrect: "約5％程度"
            - （割安に）放置:
                - Correct: "割安感のある"
                - Incorrect: "割安に放置されている"
            - 横ばい:- 
                - Correct: "債券利回りはほぼ変わらず"
                - Incorrect: "債券利回りは横ばい"
            - 行って来い:
                - Correct: "上昇したのち下落する動き"
                - Incorrect: "行って来いの動き"
            - まちまち:
                - Correct: "異なる動きとなりました"
                - Incorrect: "まちまちでした"
            - 積極姿勢/消極姿勢:
                - Correct: "長めとした"
                - Incorrect: "積極姿勢とした"
            - 銘柄解説部分での注意事項:
                - Correct: "市場予想によると、利上げが行われる見込みです。"
                - Incorrect: "利上げが行われる見込みです。"

        - **Recommended Replacements for Terms and Expressions (置き換えを推奨する用語/表現):**

            - ハト派／タカ派 (金融政策に関する): Replace with "金融緩和重視", "金融緩和に前向き", "金融引き締め重視", or "金融引き締めに積極的" (e.g., "ハト派" → "金融緩和重視").
            - 織り込む: Replace with "反映され" (e.g., "市場に織り込まれる" → "市場に反映される").
            - 相場: Replace with "市場" or "価格" (e.g., "相場が上昇" → "市場が上昇").
            - 連れ高: Replace with "影響を受けて上昇" (e.g., "連れ高となる" → "影響を受けて上昇する").
            - 伝播（でんぱ）: Replace with "広がる" (e.g., "影響が伝播する" → "影響が広がる").
            - トレンド: Replace with "傾向" (e.g., "上昇トレンド" → "上昇傾向").
            - レンジ: Replace with "範囲" (e.g., "レンジ相場" → "範囲内での価格変動").
            - ○○大手 (○○には業種（ex：通信）を想定): Replace with "大手○○メーカー", "大手○○会社", or "大手○○企業" (e.g., "通信大手" → "大手通信会社").
            - 〇％を上回る（下回る）マイナス: Replace with "〇％を超える/下回るマイナス幅" or "マイナス幅が拡大/縮小" (e.g., "5％を上回るマイナス" → "5％を超えるマイナス幅").
            - 回金: Replace with "円転" (e.g., "回金する" → "円転する").
            - ローン: Replace with "貸し付け" or "融資" (e.g., "ローンを組む" → "融資を受ける").
            - Exception: Use "住宅ローン" as is.

            - Examples of Correct Usage:
                - ハト派／タカ派:
                    - Correct: "金融緩和に前向きな政策"
                    - Incorrect: "ハト派の政策"
                - 織り込む:
                    - Correct: "市場に反映される"
                    - Incorrect: "市場に織り込まれる"
                - 相場:
                    - Correct: "市場が上昇"
                    - Incorrect: "相場が上昇"
                - 連れ高:
                    - Correct: "影響を受けて上昇する"
                    - Incorrect: "連れ高となる"
                - 伝播（でんぱ）:
                    - Correct: "影響が広がる"
                    - Incorrect: "影響が伝播する"
                - トレンド:
                    - Correct: "上昇傾向"
                    - Incorrect: "上昇トレンド"
                - レンジ:
                    - Correct: "範囲内での価格変動"
                    - Incorrect: "レンジ相場"
                - ○○大手:
                    - Correct: "大手通信会社"
                    - Incorrect: "通信大手"
                - 〇％を上回る（下回る）マイナス:
                    - Correct: "5％を超えるマイナス幅"
                    - Incorrect: "5％を上回るマイナス"
                - 回金:
                    - Correct: "円転する"
                    - Incorrect: "回金する"
                - ローン:
                    - Correct: "融資を受ける"
                    - Incorrect: "ローンを組む"
                    - Exception: "住宅ローン" (no change).

        - **Hiragana Notation (ひらがな表記するもの):**
            - Replace non-standard kanji with hiragana for readability and consistency. Below are the specific replacements:
                - 所謂: Replace with "いわゆる" (e.g., "いわゆる優良企業").
                - 暫く: Replace with "しばらく" (e.g., "しばらくお待ちください").
                - 留まる/止まる: Replace with "とどまる" (e.g., "価格がとどまる").
                - 尚: Replace with "なお" (e.g., "なお、詳細は以下をご覧ください").
                - 等: Replace with "など" when it clearly improves readability (e.g., "株式や債券など").
                - 筈: Replace with "はず" (e.g., "そうなるはずです").
                - 殆ど: Replace with "ほとんど" (e.g., "ほとんど影響がない").
                - 真似: Replace with "まね" (e.g., "まねをする").
                - 亘る: Replace with "わたる" (e.g., "長期間にわたる").
                - 但し: Replace with "ただし" (e.g., "ただし、例外があります").

            - Examples of Correct Usage
                - 所謂:
                    - Correct: "いわゆる優良企業"
                    - Incorrect: "所謂優良企業"
                - 暫く:
                    - Correct: "しばらくお待ちください"
                    - Incorrect: "暫くお待ちください"
                - 留まる/止まる:
                    - Correct: "価格がとどまる"
                    - Incorrect: "価格が留まる"
                - 尚:
                    - Correct: "なお、詳細は以下をご覧ください"
                    - Incorrect: "尚、詳細は以下をご覧ください"
                - 等:
                    - Correct: "株式や債券など"
                    - Incorrect: "株式や債券等"
                - 筈:
                    - Correct: "そうなるはずです"
                    - Incorrect: "そうなる筈です"
                - 殆ど:
                    - Correct: "ほとんど影響がない"
                    - Incorrect: "殆ど影響がない"
                - 真似:
                    - Correct: "まねをする"
                    - Incorrect: "真似をする"
                - 亘る:
                    - Correct: "長期間にわたる"
                    - Incorrect: "長期間に亘る"
                - 但し:
                    - Correct: "ただし、例外があります"
                    - Incorrect: "但し、例外があります"

        - ** Partial Kana Notation (一部かな書き等で表記するもの):**
            - For non-standard kanji or kanji with uncommon readings, use partial kana notation for clarity and consistency. Below are the specific replacements:

                牽制: Replace with "けん制" (e.g., "けん制する").
                牽引: Replace with "けん引" (e.g., "けん引役を果たす").
                終焉: Replace with "終えん" (e.g., "終えんを迎える").
                収斂: Replace with "収れん" (e.g., "収れんする").
                逼迫: Replace with "ひっ迫" (e.g., "資金繰りがひっ迫する").
                横這い: Replace with "横ばい" (e.g., "価格が横ばいになる").
            - Examples of Correct Usage
                - 牽制:
                    - Correct: "けん制する"
                    - Incorrect: "牽制する"
                - 牽引:
                    - Correct: "けん引役を果たす"
                    - Incorrect: "牽引役を果たす"
                - 終焉:
                    - Correct: "終えんを迎える"
                    - Incorrect: "終焉を迎える"
                - 収斂:
                    - Correct: "収れんする"
                    - Incorrect: "収斂する"
                - 逼迫:
                    - Correct: "資金繰りがひっ迫する"
                    - Incorrect: "資金繰りが逼迫する"
                - 横這い:
                    - Correct: "価格が横ばいになる"
                    - Incorrect: "価格が横這いになる"

        - **General Okurigana Rules (一般的な送り仮名など):**
            - Ensure proper usage of okurigana (送り仮名) for consistency and readability. Below are the specific rules and examples:
                - ○ヶ月: Replace with "○ヵ月" (e.g., "3ヵ月").
                - 入替え: Replace with "入れ替え" (avoid using "入れ換え").
                - 行う、行われる: Replace with "行なう", "行なわれる" to align with the prospectus notation.
                - 買付、買付け/売付、売付け: Replace with "買い付け", "売り付け".
                    - Correct: "買い付けしました", "売り付けしました", or "買い付け/売り付けを行ないました".
                    - Incorrect: "買い付けました", "売り付けました".
                - 格付:
                    - Use "格付け" in general (e.g., "格付けを上げる").
                    - For compound words, omit okurigana (e.g., "格付機関", "格付別").
                - 国債買い入れ: Use "国債買い入れ" or "国債買入れ".
                    - Exception: "国債買入オペ" does not require okurigana.
                - 買増し/売増し: Replace with "買い増し", "売り増し" (e.g., "買い増ししました").
                - 買建て/売建て: Replace with "買い建て", "売り建て".
                - 切上げ/切捨て: Replace with "切り上げ", "切り捨て".
                - 組入れ:
                    - Use "組み入れ" in general (e.g., "組み入れ比率").
                    - For compound words, omit okurigana (e.g., "組入比率").
                - 繰上げ償還: Replace with "繰上償還".
                - 先き行き: Replace with "先行き".
                - 下支える: Replace with "下支えする" (e.g., "下支えする要因").
                - 取り引き: Replace with "取引".
                - 引上げ/引下げ: Replace with "引き上げ", "引き下げ".
                - 引締め:
                    - Use "引き締め" in general (e.g., "金融引き締め").
                    - For compound words, omit okurigana (e.g., "引締策").
                - 引続き: Replace with "引き続き".
            - Examples of Correct Usage:
                - ○ヶ月:
                    - Correct: "3ヵ月"
                    - Incorrect: "3ヶ月"
                - 入替え:
                    - Correct: "入れ替え"
                    - Incorrect: "入れ換え"
                - 行う、行われる:
                    - Correct: "行なう", "行なわれる"
                    - Incorrect: "行う", "行われる"
                - 買付、買付け/売付、売付け:
                    - Correct: "買い付けしました", "売り付けしました", or "買い付け/売り付けを行ないました"
                    - Incorrect: "買い付けました", "売り付けました"
                - 格付:
                    - Correct: "格付けを上げる", "格付機関"
                    - Incorrect: "格付を上げる"
                - 国債買い入れ:
                    - Correct: "国債買い入れ", "国債買入れ", or "国債買入オペ"
                    - Incorrect: "国債買入れオペ"
                - 買増し/売増し:
                    - Correct: "買い増ししました"
                    - Incorrect: "買増ししました"
                - 買建て/売建て:
                    - Correct: "買い建て", "売り建て"
                    - Incorrect: "買建て", "売建て"
                - 切上げ/切捨て:
                    - Correct: "切り上げ", "切り捨て"
                    - Incorrect: "切上げ", "切捨て"
                - 組入れ:
                    - Correct: "組み入れ比率", "組入比率"
                    - Incorrect: "組入れ比率"
                - 繰上げ償還:
                    - Correct: "繰上償還"
                    - Incorrect: "繰上げ償還"
                - 先き行き:
                    - Correct: "先行き"
                    - Incorrect: "先き行き"
                    - 下支える:
                    - Correct: "下支えする"
                    - Incorrect: "下支える"
                - 取り引き:
                    - Correct: "取引"
                    - Incorrect: "取り引き"
                - 引上げ/引下げ:
                    - Correct: "引き上げ", "引き下げ"
                    - Incorrect: "引上げ", "引下げ"
                - 引締め:
                    - Correct: "金融引き締め", "引締策"
                    - Incorrect: "金融引締め"
                - 引続き:
                    - Correct: "引き続き"
                    - Incorrect: "引続き"

        - **English Abbreviations (英略語):**
            - Use half-width characters for English abbreviations and provide explanations in parentheses where necessary. Below are the specific rules and examples:

                - AAA: AAA（全米自動車協会）
                - ABS: ABS（資産担保証券、各種資産担保証券） ※ABSで使用可
                - ADB: ADB（アジア開発銀行）
                - ADR: ADR（米国預託証券）
                - AI: AI（人工知能） ※カッコ書きはなくても可
                - AIIB: AIIB（アジアインフラ投資銀行）
                - APEC: APEC（アジア太平洋経済協力会議）
                - API: API（全米石油協会）
                - BIS: BIS（国際決済銀行）
                - BOE: BOE（英中央銀行、イングランド銀行）
                - BRICS: BRICS（ブラジル、ロシア、インド、中国、南アフリカ） ※BRICSでも可
                - CDS市場: CDS（クレジット・デフォルト・スワップ）市場
                - CFROIC: CFROIC（投下資本キャッシュフロー率）または、（投下資本キャッシュフロー利益率）
                - Chat GPT: Chat GPT（AIを使った対話型サービス）
                - CMBS: CMBS（商業用不動産ローン担保証券） ※CMBSで使用可
                - COP26: COP26（国連気候変動枠組み条約第26回締約国会議）
                - CPI: CPI（消費者物価指数）
                - CSR: CSR（企業の社会的責任）
                - DR: DR（預託証書）
                - DRAM: DRAM（半導体素子を利用した記憶装置のひとつ）
                - DX: DX（デジタルトランスフォーメーション）
                - EC: EC（電子商取引）
                - ECB: ECB（欧州中央銀行）
                - EIA: EIA（米エネルギー省エネルギー情報局）
                - EMEA: EMEA（欧州・中東・アフリカ）
                - EPA: EPA（米環境保護局）
                - EPS: EPS（一株当たり利益）
                - ESM: ESM（欧州安定メカニズム） ※EFSFの後継機関
                - ESG: ESG（環境・社会・企業統治）
                - EU: EU（欧州連合）
                - EV: EV（電気自動車）
                - EVA: EVA（経済的付加価値）
                - FASB: FASB（米財務会計基準審議会）
                - FDA: FDA（米国食品医薬品局）
                - FFレート（米国の場合）: 政策金利（FFレート）
                - FOMC: FOMC（米連邦公開市場委員会）
                - FRB: FRB（米連邦準備制度理事会）
                - FTA: FTA（自由貿易協定）
                - G7: G7（主要7ヵ国会議） ※7ヵ国財務相・中央銀行総裁会議
                - G8: G8（主要8ヵ国首脳会議）
                - G20: G20（20ヵ国・地域）財務相・中央銀行総裁会議、首脳会議
                - GDP: GDP（国内総生産） ※例：実質GDP（国内総生産）成長率
                - Note: For the Eurozone, Hong Kong, and Taiwan, use "域内総生産" instead of "国内総生産."
                - GPIF: 年金積立金管理運用独立行政法人（GPIF） ※（）は後付け
                - GNP: GNP（国民総生産）
                - GST（インドの場合）: GST（物品・サービス税）
                - IEA: IEA（国際エネルギー機関）
                - 独Ifo企業景況感指数（または独Ifo景況感指数）: ※ドイツ6大研究所の最大手、IFO研究所が独企業の役員に対して行なうアンケート調査。日銀短観に相当する。ドイツ経済指標でもっとも注目度が高いとされ、鉱工業生産との関連性が強い。
                - IMF: IMF（国際通貨基金）
                - IoT: IoT（モノのインターネット）
                - IPEF: IPEF（インド太平洋経済枠組み）
                - IPO: IPO（新規株式公開）
                - ISM: ISM（全米供給管理協会）
                - ※ISM製造業景況指数で使う際は、括弧書きがなくても可。米国以外の地域も含むコメントの際は、（全米供給管理協会）をつけない場合、冒頭に「米」をつける。
                - IT: IT（情報技術） ※カッコ書きはなくても可
                - LBO: LBO（レバレッジド・バイアウト：対象企業の資産を担保に資金調達する買収）
                - LED: LED（発光ダイオード）
                - LME: LME（ロンドン金属取引所）
                - LNG: LNG（液化天然ガス）
                - M&A: M&A（企業の合併・買収）
                - MAS: MAS（シンガポール金融通貨庁）
                - MBA: MBA（全米抵当貸付銀行協会）
                - MBO: MBO（経営陣による買収）
                - MBS: MBS（住宅ローン担保証券） ※MBSで使用可
                - NAFTA: NAFTA（北米自由貿易協定）
                - NAHB: NAHB（全米住宅建設業者協会）
                - NAIC: NAIC（全米保険監督官協会）
                - NAR: NAR（全米不動産業者協会）
                - NDF: NDF（為替先渡取引のひとつ）
                - NISA: NISA（少額投資非課税制度）
                - OECD: OECD（経済協力開発機構）
                - OEM: OEM（相手先ブランドによる生産）
                - OPEC: OPEC（石油輸出国機構）
                - OPECプラス: OPECプラス（OPEC（石油輸出国機構）と非加盟産油国で構成するOPECプラス）
                - PBR: PBR（株価純資産倍率）
                - PCE: PCE（個人消費支出）
                - PCFR: PCFR（株価キャッシュフロー倍率）
                - PER: PER（株価収益率）
                - PMI: PMI（購買担当者景気指数）
                - ※コメントの使用例は、「5月の○○製造業PMI（購買担当者景気指数）は50.8と市場予想を上回りました。」と、○○に国名（域名）を入れて国別指数を表記する。
                - ※中国に関しては、国家統計局の公表する「確定値」（翌月1日公表）と、財新伝媒（CaixinMedia）が公表する「確報値」を区別して使い分けてください。
                - PO: PO（公募・売出し）
                - PPI: PPI（生産者物価指数） ※米国労働省発表。国内の製造業者の販売価格を調査。日本の卸売物価は輸送費や流通マージンを含むが、米国PPIは生産者の出荷時点の価格を対象とする。
                - QE: QE（量的金融緩和）
                - QT: QT（量的引き締め） ※類例）テーパリング（量的金融緩和の縮小）
                - Quad: Quad（日米豪印戦略対話）
                - RBA: RBA（豪州準備銀行）
                - RCEP: RCEP（地域的な包括的経済連携協定）
                - RBI: RBI（インド準備銀行）
                - ROA: ROA（総資産利益率）
                - ROE: ROE（自己資本利益率）
                - S&L: S&L（貯蓄貸付組合）
                - S&P: S&P（スタンダード・アンド・プアーズ）社
                - SDGs: SDGs（持続可能な開発目標）
                - SEC: SEC（米証券取引委員会）
                - SQ: SQ（特別清算指数）
                - SRI: SRI（社会的責任投資）
                - SUV: SUV（スポーツ用多目的車）
                - TALF: TALF（ターム物資産担保証券貸出制度）
                - TOB: TOB（株式公開買付け）
                - TTM: 仲値
                - TPP: TPP（環太平洋経済連携協定）
                - UAE: UAE（アラブ首長国連邦）
                - UAW: UAW（全米自動車労働組合）
                - USDA: USDA（米国農務省）
                - USMCA: USMCA（NAFTA新協定）または、（米国・メキシコ・カナダ協定）
                - USTR: USTR（米通商代表部）
                - VAT: VAT（付加価値税）
                - WTI: WTI（ウエスト・テキサス・インターミディエート） ※WTIで使用可
                - WTO: WTO（世界貿易機関）
                - 独ZEW景気期待指数（または独ZEW景況感指数）:
                    - ZEW単独使用の場合は括弧書き付き、または欧州経済研究センターのみとしZEWを使わない。
            
            - Examples of Correct Usage:
                - GDP:
                    - Correct: "実質GDP（国内総生産）成長率は～"
                    - Incorrect: "実質GDP成長率は～"
                - PMI:
                    - Correct: "5月の米製造業PMI（購買担当者景気指数）は50.8と市場予想を上回りました。"
                    - Incorrect: "5月のPMIは50.8と市場予想を上回りました。"
                - CPI:
                    - Correct: "CPI（消費者物価指数）は上昇しました。"
                    - Incorrect: "CPIは上昇しました。"
                - QE:
                    - Correct: "QE（量的金融緩和）が実施されました。"
                    - Incorrect: "量的金融緩和が実施されました。"
                - ESG:
                    - Correct: "ESG（環境・社会・企業統治）投資が増加しています。"
                    - Incorrect: "ESG投資が増加しています。"

        - **Loanwords and Technical Terms (外来語・専門用語など):**
            - Use loanwords and technical terms with their prescribed explanations in parentheses where necessary. Below are the specific rules and examples:
                - アセットアロケーション: アセットアロケーション（資産配分）
                - アンダーウェイト: アンダーウェイト（ベンチマーク※に比べ低めの投資比率） ※参考指数を用いているものはベンチマークの代わりに参考指数を使うこと
                - E-コマース／e-コマース／EC: Eコマース（電子商取引）/eコマース（電子商取引）/EC（電子商取引） ※カッコ書きなくても可
                - イールドカーブ: イールドカーブ（利回り曲線）
                - ※欄外に注記する場合: イールドカーブ（利回り曲線）とは、横軸に残存年数、縦軸に利回りをとった座標に、債券利回りを点描して結んだ曲線のことを指します。
                - イールドカーブ・コントロール: イールドカーブ・コントロール（長短金利操作）
                - イールドカーブのスティープ化: イールドカーブのスティープ化（長・短金利格差の拡大）
                - イールドカーブのフラット化: イールドカーブのフラット化（長・短金利格差の縮小）
                - インカムゲイン: インカムゲイン（利子収入）
                - インタラクティブ: インタラクティブ（双方向性）
                - インバウンド: インバウンド（訪日外国人） ※～消費として使用
                - エクイティ・ファイナンス: エクイティ・ファイナンス（新株発行等による資金調達）
                - エクスポージャー: ※積極的に使用しない。（価格変動リスク資産の配分比率、割合）
                - 円キャリートレード: 円キャリートレード（円などの低金利の通貨を売り、高金利の通貨を買う取引）
                - オバマケア: オバマケア（医療保険制度改革法）
                - オーバーウェイト: オーバーウェイト（ベンチマーク※に比べ高めの投資比率） ※参考指数を用いているものはベンチマークの代わりに参考指数を使うこと
                - オンデマンド: オンデマンド（注文生産）
                - カントリー･アロケーション: カントリー･アロケーション（国別資産配分）
                - 逆イールド: 逆イールド（短期債券の利回りが長期債券の利回りを上回っている状態）
                - キャッシュフロー: キャッシュフロー（現金収支）
                - キャピタルゲイン: キャピタルゲイン（値上がり益）
                - キャリートレード: キャリートレード（低金利の資金を調達して、高金利の資産で運用する取引）
                - クレジット（信用）市場: 信用リスク（資金の借り手の信用度が変化するリスク）を内包する商品（クレジット商品）を取引する市場の総称。企業の信用リスクを取引する市場。
                - クレジットスプレッド: クレジットスプレッド（企業の信用力の差による利回りの差）／（利回り格差）
                - グローバリゼーション: グローバリゼーション（地球規模化）
                - コジェネレーション: コジェネレーション（熱電供給システム）
                - コーポレート・ガバナンス: コーポレート・ガバナンス（企業統治）
                - コングロマリット: コングロマリット（複合企業）
                - コンソーシアム: コンソーシアム（共同事業）
                - サーベイランス: サーベイランス（調査監視）
                - サステナビリティ: サステナビリティ（持続可能性）
                - サブプライムローン: サブプライムローン（信用度の低い個人向け住宅融資）
                - サプライチェーン: サプライチェーン（供給網）
                - ジェネリック医薬品: ジェネリック医薬品（後発薬）
                - シャリーア/シャリア: 原則として「シャリーア」に表記を統一する。
                - シクリカル: シクリカル（景気敏感）
                - システミック・リスク: 個別の金融機関の支払不能等や、特定の市場または決済システム等の機能不全が、他の金融機関、他の市場、または金融システム全体に波及するリスク
                - シャドーバンキング: シャドーバンキング（影の銀行）
                - ショートポジション: ショートポジション（売り持ち）
                - 信用市場: 企業の信用リスクを取引する市場
                - スタグフレーション: （景気後退下のインフレ）、（物価上昇と景気悪化が同時に進むこと） ※スタグネーション（景気後退）とインフレーション（物価上昇）の合成語
                - スティープ化: スティープ化（長短金利格差の拡大）
                - ストレステスト: ストレステスト（健全性審査）
                - スプレッド: スプレッド（利回り格差）
                - スマートシティー: スマートシティー（ITを活用した次世代型の都市）
                - スマートモビリティ: スマートモビリティ（従来の交通・移動を変える新たなテクノロジー）
                - セーフティネット: セーフティネット（安全網）
                - 全人代: 全人代（全国人民代表大会）
                - センチメント: センチメント（雰囲気、市場に対する見方、市場心理） ※項目としての「消費者センチメント」の場合は修正しない
                - ソフトランディング: ソフトランディング（軟着陸） ※バブル状態になった経済状態を、大きなショックを与えることなく沈静化していくこと
                - ソブリンリスク: 国家のリスク、国家の信用リスク、政府債務の信認危機 ※言い換える方が良い
                - ダイバーシティ: ダイバーシティ（多様性）
                - ターミナルレート: ターミナルレート（政策金利の最終到達水準）
                - テーパリング: テーパリング（量的金融緩和の縮小）
                - ディストレス債券: ※欄外に注記: 信用事由などにより、価格が著しく下落した債券
                - ディフェンシブ: ディフェンシブ（景気に左右されにくい）
                - テクニカル: テクニカル（過去の株価の動きから判断すること） ※ただし、赤字部分は前後の文章に合わせ適宜修正
                - デフォルト: デフォルト（債務不履行）
                - デフォルト債: ※欄外に注記: デフォルトとは一般的には債券の利払いおよび元本返済の不履行、もしくは遅延などをいい、このような状態にある債券をデフォルト債といいます。
                - デュレーション: デュレーション（金利感応度）
                - ※欄外に注記: デュレーションとは、金利がある一定の割合で変動した場合、債券の価格がどの程度変化するかを示す指標です。すなわち、この値が大きいほど金利変動に対する債券価格の変動率が大きくなります。（目論見書の表記、他パターン有）
                - デリバティブ: デリバティブ（金融派生商品）
                - 投資適格債: ※欄外に注記: 格付機関によって格付けされた公社債のうち、債務を履行する能力が十分にあると評価された公社債
                - ドルペッグ制: ドルペッグ（連動）制
                - ニュートラル: ニュートラル（ベンチマーク※並みの投資比率） ※参考指数を用いているものはベンチマークの代わりに参考指数を使うこと
                - バイオシミラー: バイオシミラー（後続薬）
                - バイオマス: バイオマス（生物を利用して物質やエネルギーを得ること）
                - バーチャル: バーチャル（仮想）
                - バリュエーション: バリュエーション（投資価値評価）
                - バリュー: バリュー（割安）
                - 5G: 5G（第5世代移動通信システム） ※カッコ書きなくても可
                - ファンダメンタルズ: ファンダメンタルズ（基礎的条件）/（基礎的諸条件）/（経済の基礎的条件）
                - ※REITファンドで使用する場合: ファンダメンタルズ（賃料や空室率、需給関係などの基礎的条件）
                - フィンテック: フィンテック（金融と技術の融合） ※「Finance」+「Technology」を組み合わせた造語
                - フェアバリュー: フェアバリュー（適正価格）
                - フェーズ2: フェーズ2（臨床試験の中間段階）
                - フェーズ3: フェーズ3（臨床試験の最終段階）
                - フードデリバリー: フードデリバリー（料理等の宅配サービス）
                - フリーキャッシュフロー: 税引後営業利益に減価償却費を加え、設備投資額と運転資本の増加を差し引いたもの
                - フルインベストメント: フルインベストメント（高位組入）
                - ブロードバンド: ブロードバンド（大容量･高速通信）
                - ベージュブック: FRB（米連邦準備制度理事会）が発表したベージュブック（地区連銀経済報告） ※スペースがない場合は、ベージュブック（米地区連銀経済報告）
                - ポテンシャル: ポテンシャル（潜在力）
                - ポピュリズム: ポピュリズム（大衆迎合主義）
                - ボラティリティ（ー）: ボラティリティ（ー）（価格変動性/市場変動性） ※当社は、ボラティリティ
                - モーゲージ: モーゲージ（不動産担保ローン）
                - モーゲージ債: モーゲージ債（不動産ローン担保債券）
                - モメンタム: 相場の「勢い」とか「方向性」 ※使わず言い換える方がよい
                - モラルハザード: モラルハザード（倫理崩壊）
                - リエンジニアリング: リエンジニアリング（業務の抜本的革新）
                - リオープン: リオープン/リオープニング（経済活動再開） ※カッコ書きなくても可 ※「経済活動再開」のみでの表記も可
                - リスクプレミアム: 同じ投資期間内において、あるリスク資産の期待収益率が、無リスク資産（国債など）の収益率を上回る幅のこと。
                - リセッション: リセッション（景気後退）
                - リターンリバーサル: リターンリバーサル（過剰反応効果）
                - リバウンド: リバウンド（反発）
                - リバランス: リバランス（投資比率の再調整）
                - リフレーション: リフレーション ※デフレーションから抜けて、まだ、インフレーションにはなっていない状況のこと。
                - レパトリ減税: レパトリ（海外収益の本国還流）減税
                - レバレッジドローン: 低格付け等の借り手向け融資
                - レラティブ・バリュー: レラティブ・バリュー（相対価値）
                - ロックダウン: ロックダウン（都市封鎖）
                - ロングポジション: ロングポジション（買い持ち）

            - Examples of Correct Usage
                - アセットアロケーション:
                    - Correct: "アセットアロケーション（資産配分）を調整しました。"
                    - Incorrect: "資産配分を調整しました。"
                - イールドカーブ:
                    - Correct: "イールドカーブ（利回り曲線）がフラット化しました。"
                    - Incorrect: "利回り曲線がフラット化しました。"
                - キャッシュフロー:
                    - Correct: "キャッシュフロー（現金収支）が改善しました。"
                    - Incorrect: "現金収支が改善しました。"
                - デフォルト:
                    - Correct: "デフォルト（債務不履行）のリスクが高まっています。"
                    - Incorrect: "債務不履行のリスクが高まっています。"
                - 5G:
                    - Correct: "5G（第5世代移動通信システム）が普及しています。"
                    - Incorrect: "第5世代移動通信システムが普及しています。"

        3. **Consistency with Report Data Section (レポートのデータ部との整合性確認):**
        - Ensure the textual description in the report is completely consistent with the data section, without any logical or content-related discrepancies.

        **Special Rules:**
        1. **Do not modify proper nouns (e.g., names of people, places, or organizations) unless they are clearly misspelled.**
            -Exsample:
            ベッセント氏: Since this is correct and not a misspelling, it will not be modified.
        2. **Remove unnecessary or redundant text instead of replacing it with other characters.**
            -Exsample:
            ユーロ圏域内の景気委: Only the redundant character 委 will be removed, and no additional characters like の will be added. The corrected text will be: ユーロ圏域内の景気.
        3. **Preserve spaces between words in the original text unless they are at the beginning or end of the text.**
            -Example:
            input: 月の 前半は米国の 債券利回りの上昇 につれて
            Output: 月の 前半は米国の 債券利回りの上昇 につれて (spaces between words are preserved).

        **Output Requirements:**
        1. **Highlight the original incorrect text in red and include additional details:**
        - For corrected parts:
            - Highlight the original incorrect text in red using `<span style="color:red;">`.
            - Append the corrected text in parentheses, marked with a strikethrough using `<s>` tags.
            - Provide the reason for the correction and indicate the change using the format `123 → 456`.
            - Example:
            `<span style="color:red;">123</span> (<span style="background:yellow;">修正理由: 一致性不足 <s>123</s> → 456</span>)`
        
        2. **Preserve the original structure and formatting of the document:**
        - Maintain paragraph breaks, headings, and any existing structure in the content.

        3. **Use the uploaded correction rules for reference:**
        - {prompt}

        4. **Do not provide any explanations or descriptions in the output. Only return the corrected HTML content.**
    """
```


## 课题 2025-01-23
-------------------------------------------------------------
### 全自动化实现，目前想到的几个方法

1. 个人用pc，需要执行程序，对文件夹（指定目录的文件夹，可以局域网内共享文件夹）进行监控及触发事件处理。
    - 当发生变动或者有文件时候，执行上传操作。 
    - 需要个人pc一直运行着exe，程序来触发上传事件

2. 市面上，看看有监控文件夹类的可用的程序。

3. GPT，上周发布了task 定时的任务执行的功能，实现了自动化助理。
    - 研究看看，此部分能否应用上

------------------------------------------------------------




```python
Invalid response object from API: '{ "statusCode": 401, "message": "Unauthorized. Access token is missing, invalid, audience is incorrect (https://cognitiveservices.azure.com), or have expired." }' (HTTP response code was 401)

```

``` # 使用MSAL库自动处理令牌刷新（Python示例）
from msal import ConfidentialClientApplication

app = ConfidentialClientApplication(
    "client_id",
    authority="https://login.microsoftonline.com/your_tenant_id",
    client_credential="client_secret"
)

# 首次获取令牌
result = app.acquire_token_for_client(scopes=["https://cognitiveservices.azure.com/.default"])

# 使用令牌时检查过期并自动刷新
if "access_token" in result:
    while True:
        # 每次调用API前检查令牌有效期
        if result["expires_in"] < 300:  # 剩余5分钟时刷新
            result = app.acquire_token_for_client(scopes=["https://cognitiveservices.azure.com/.default"])
        
        # 调用ChatGPT服务
        headers = {"Authorization": f"Bearer {result['access_token']}"}
        response = requests.post(endpoint, headers=headers, json=payload)
        
        if response.status_code == 401:
            # 强制刷新令牌后重试
            result = app.acquire_token_for_client(scopes=["https://cognitiveservices.azure.com/.default"])
            continue
```

用户可能没有充分利用缓存机制。
例如，如果多个用户请求相同的内容，可以缓存结果以减少对API的调用次数。
或者，在需要频繁调用的情况下，预先获取令牌并复用，而不是每次请求都获取新令牌

```json
{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3Mzg1NTA1OTUsIm5iZiI6MTczODU1MDU5NSwiZXhwIjoxNzM4NjM3Mjk1LCJhaW8iOiJrMlJnWVBCWmNsa2tLMGZrelU2RGc1eHZEckVXQXdBPSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiJ0MFNfUS13VEcwV0dJT2FPRHRNZUFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiNCA3IiwieG1zX21pcmlkIjoiL3N1YnNjcmlwdGlvbnMvM2JkM2Q3ZDktYTU3ZC00ZmJlLTg2ZTUtZWIwNGJiZDZiYmJlL3Jlc291cmNlZ3JvdXBzL25yaXBpLWNvbW1lbnRjaGVjay1yZy9wcm92aWRlcnMvTWljcm9zb2Z0LldlYi9zaXRlcy9OYW1DaGVja1dlYiJ9.Wzt8UYuWLkVlaKAsU6jzaEeKyKkvh8SFCompH9A-ZenspUg9XwFcljSDepCB62RrwjlsqQLifXKMOK-hgWQRBaUwUKnZeepQyARgvalh7hutFjpWl66JHLVjrHZPOxidnKzfiOkrqy6cfADm8sfOmUQWz_CveObfjbDazPeiWp-NR2zGyFcsbMPc9H_gV1P8Pr3w3goTzrYLHoA9BQJjVYDHctx_tJFPJ_i6yAopjocmd5YZK_OO1vWmEgBdjpIPYcNY4Tku834WU9mhw-FEBDtbH-8fWza3i4XPHwsfjWnfor1IKYrDsoQSgb2Kh5J5Pa2y0dACBhb7Wv_MHZ70UQ"}

{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3Mzg1NTExNzEsIm5iZiI6MTczODU1MTE3MSwiZXhwIjoxNzM4NjM3ODcxLCJhaW8iOiJrMlJnWUZoa015Ty9lT1hhbEJyMmdEV3ZPZis5QUFBPSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiJpOHNXUF9sR2IwbU5JbUlwakJBWUFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiMjggNyIsInhtc19taXJpZCI6Ii9zdWJzY3JpcHRpb25zLzNiZDNkN2Q5LWE1N2QtNGZiZS04NmU1LWViMDRiYmQ2YmJiZS9yZXNvdXJjZWdyb3Vwcy9ucmlwaS1jb21tZW50Y2hlY2stcmcvcHJvdmlkZXJzL01pY3Jvc29mdC5XZWIvc2l0ZXMvTmFtQ2hlY2tXZWIifQ.RfMK8LAhapcEsQ2wrsdRcCXTqfa-BoH4t39zHyWxKPVHSyTbHzRuAcclyYEuLWNbFZeZlUgwOQnWHRecZdxCieCxBUhWebo9HhNz9epRPvPjQxaqGiJTBZeHlNXD_4KzVn-wlvrOKfDBu7nO6Pm_-fSItEVO9QlF_nKO-vkE0nv2Dp9extOif5chuN2AO_nCZhjzO0Ps6GccxThAHBlr2z4pTdQlXACsRe2vYVGRRBkxr4LweYMSH6_5-CMQRAkawp33DaJgu-pdtw-5dBCIVTGCzHSz4V1VFieyVMPKHeAlaHgnnHD8a_n-ZqxjL-XA_jCRHcPJoSKs_aglcosBZA"}


{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3Mzg1NTQyMTYsIm5iZiI6MTczODU1NDIxNiwiZXhwIjoxNzM4NjQwOTE2LCJhaW8iOiJrMlJnWUhCK2ZUdmp6c2tHUmQyZTRpTU1lbnRsQVE9PSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiJXUFU4cVJHTncwT0VJQlc0SHhSaUFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiNyA4IiwieG1zX21pcmlkIjoiL3N1YnNjcmlwdGlvbnMvM2JkM2Q3ZDktYTU3ZC00ZmJlLTg2ZTUtZWIwNGJiZDZiYmJlL3Jlc291cmNlZ3JvdXBzL25yaXBpLWNvbW1lbnRjaGVjay1yZy9wcm92aWRlcnMvTWljcm9zb2Z0LldlYi9zaXRlcy9OYW1DaGVja1dlYiJ9.ODUcc-WrpezlTufJL2h7tvPfl1dY9lB_bitR4MHiHAzvVf69CSkYb0PUaxQF299pYZLf7Ty0BJjqfASZ8XYM9mw00lI1E9HlHXSveJDVhfHp69ZEWrklGqYu_DdmnYxt5Wql1JdUq8AeUqftS35vLPLA_BUIQsAgFSdV-8N6_L87aAzbVh5q9qEGvmaVmE1q-aVCx-qNZyW79tsrkpOFGLZpj5b7BVYgDnSA2-6h3bErSwLTg8BcJS-YLrEeriadR_n1cd8JA6bVurtqHjn8ytTdkJUKTwEkYVbwNAXno9ULl4k_cmgsUcH-9WHcM_IBrBjrgDLfLYkPrpDVdIzx9Q"}
```


```js
async function fetchBotResponse(userInputs) {
    try {
        // 단일 입력인 경우 배열로 변환
        if (!Array.isArray(userInputs)) {
            userInputs = [userInputs];
        }

        // 여러 입력에 대해 동시에 요청을 보냄
        const promises = userInputs.map(async (userInput) => {
            const response = await fetch(API_URL, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ input: userInput })
            });
            return response.json();
        });

        // 모든 요청이 완료될 때까지 기다림
        const results = await Promise.all(promises);

        // 각 결과를 처리
        results.forEach((data) => {
            addMessage(data.answer || '回答が見つかりませんでした。', 'bot', 'outputContainer');
        });
    } catch (error) {
        console.error('Error fetching response:', error);
        addMessage('エラーが発生しました。もう一度お試しください。', 'bot', 'outputContainer');
    }
}

```


```js
{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3Mzg1OTk3NjcsIm5iZiI6MTczODU5OTc2NywiZXhwIjoxNzM4Njg2NDY3LCJhaW8iOiJrMlJnWU9pT08vNWZzdjV1ZkZISmhxQ2pzd1BzQVE9PSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiJXRy01ajV6eWIwS3BBMmd0emUwZEFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiNyAyNiIsInhtc19taXJpZCI6Ii9zdWJzY3JpcHRpb25zLzNiZDNkN2Q5LWE1N2QtNGZiZS04NmU1LWViMDRiYmQ2YmJiZS9yZXNvdXJjZWdyb3Vwcy9ucmlwaS1jb21tZW50Y2hlY2stcmcvcHJvdmlkZXJzL01pY3Jvc29mdC5XZWIvc2l0ZXMvTmFtQ2hlY2tXZWIifQ.Ixl6t8lkwHYCLagTWldHHfDUDRv_CemKPl_uQes-O_N_iVe2WRpziKVFeA_ubbEmSIR-nX0mpbPIJJMM_x1EiLNG3YrMeKw80Go0977XxfgEBxEFrYEETqaU4sY96T8BGRBuqy81I-gxThqO4ZEuQgOu9v7fXWidu-IQX2uXKhGJIV5jsuuWFW6Fj_Wo-Qa8oDaQIjh4srI76y_gzPNxpjSoto_AdfjrfOYky9nD5Uxgxk7fwsTH400xfiK0exjU8t50Pm6UoohpWisi_7mMBdjw-R5hdPAL9jPyEFjx84QCe1YGX8KJbYEsQICcp22HUkKZiZa11wa-VosjPsMc0A"}
```

"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3Mzg2MjY3MTcsIm5iZiI6MTczODYyNjcxNywiZXhwIjoxNzM4NzEzNDE3LCJhaW8iOiJrMlJnWU9pT08vNWZzdjV1ZkZISmhxQ2pzd1BzQVE9PSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiJ0MFNfUS13VEcwV0dJT2FPTXlrdEFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiMTggNyIsInhtc19taXJpZCI6Ii9zdWJzY3JpcHRpb25zLzNiZDNkN2Q5LWE1N2QtNGZiZS04NmU1LWViMDRiYmQ2YmJiZS9yZXNvdXJjZWdyb3Vwcy9ucmlwaS1jb21tZW50Y2hlY2stcmcvcHJvdmlkZXJzL01pY3Jvc29mdC5XZWIvc2l0ZXMvTmFtQ2hlY2tXZWIifQ.SnKLWNg7CRrQ4DNFuikE7IzcF2NaAHsbnqy3Kxip1XdQCCLmIw8wsYnYKEjSayY-jr3HueYECBDQkHlr0Y9TQxoT1xi0wLLTv0OnchC58sFje9Ro2mjSGZc8iUPV2U59YXXx_wXYCi5TArF-zyre5iurlpyBq4CwcDH7knkgQpKJmYbgQpvzMWJJBwh2I7w1giL8OjXHVTGBdVlWdfg26xNQJih10PTY_IPCH66BMpCkngDfM8vEBZWKt3seKoXz2l_2biruOYaJpmCjDGR6KIBwXFlj3VQsrDLu1eCK_PcWYNVV4qa3dfu-LVSwav8iRVdv-oW7NdYT2JqyF0fo0A"}

{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3Mzg2Mjg0ODMsIm5iZiI6MTczODYyODQ4MywiZXhwIjoxNzM4NzE1MTgzLCJhaW8iOiJrMlJnWU5oOUxQV25TYlhxdFMwdkduakZ2ay81QUFBPSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiI2RWZUT0ZwOWYwcWZYd21mem9jZEFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiMTAgNyIsInhtc19taXJpZCI6Ii9zdWJzY3JpcHRpb25zLzNiZDNkN2Q5LWE1N2QtNGZiZS04NmU1LWViMDRiYmQ2YmJiZS9yZXNvdXJjZWdyb3Vwcy9ucmlwaS1jb21tZW50Y2hlY2stcmcvcHJvdmlkZXJzL01pY3Jvc29mdC5XZWIvc2l0ZXMvTmFtQ2hlY2tXZWIifQ.ArPmYWtFSACYTgS9yrUEnpC3z5LrNSnvo8YzZVI5RdXPuJ1jrjIrl_HAUxWRukPOHPRoCnJqsFdc5r3eZKE2oA6-VXoe8xKsRInAsHOAGG4IEtj-xNGWKsxrEbP1PfJfKJaAHQvlvvVlYiPegTRpMbWmLrAYPaJ6e0ps8pUr8r3HG6P1VOXfF6o5gV25_kMzzVssmgWzSQC4ED8seJTpg3MgmbAB45VSSjpo6EHpd-QedlulLh48GoaVcm2oTQzuVtty19JVhjyk0QjCTxuGckkbbZEdM0hRbn0DePsadNC_niCni6VfL9qzrTwtq5fCk56WccDJHJbZvSfqtyigfg"}


```markdown
- **Task**: Header Date Format Validation & Correction  
        - **Target Area**: Date notation in parentheses following "今後運用方針 (Future Policy Decision Basis)"  
        ---
        ### Validation Requirements  
        1. **Full Format Compliance Check**:  
        - Must follow "YYYY年MM月DD日現在" (Year-Month-Day as of)  
        - **Year**: 4-digit number (e.g., 2024)  
        - **Month**: 2-digit (01-12, e.g., April → 04)  
        - **Day**: 2-digit (01-31, e.g., 5th → 05)  
        - **Suffix**: Must end with "現在" (as of)  

        2. **Common Error Pattern Detection**:  
        ❌ "1月0日" → Missing month leading zero + invalid day 0  
        ❌ "2024年4月1日" → Missing month leading zero (should be 04)  
        ❌ "2024年12月" → Missing day value  
        ❌ "2024-04-05現在" → Incorrect separator usage (hyphen/slash)  
        ---
        ### Correction Protocol  
        1. **Leading Zero Enforcement**  
        - Add leading zeros to single-digit months/days (4月 → 04月, 5日 → 05日)  

        2. **Day 0 Handling**  
        - Replace day 0 with last day of previous month  
        - Example: 2024年4月0日 → 2024年03月31日  

        3. **Separator Standardization**  
        - Convert hyphens/slashes to CJK characters:  
            `2024/04/05` → `2024年04月05日`  

        ---
        ### Output Format Specification  
        ```html
        <Correction Example>
        <span style="color:red;">（2024年4月0日現在）</span> 
        → 
        <span style="color:green;">（2024年04月01日現在）</span>
        修正理由：
        ①先頭ゼロを欠いた月（1→01）
        ②無効な0日目→前月の最終日に調整（31）
        ---
```


```js
// // const API_URL = 'https://namcheckweb.azurewebsites.net/api/ask_gpt_api'; // GPT API URL
// const UPLOAD_API_URL = 'https://namcheckweb.azurewebsites.net/api/check_upload'; // 파일 업로드 API URL
// const WRITE_API_URL = 'https://namcheckweb.azurewebsites.net/api/write_upload'; // 파일 업로드 API URL


// uploadedText = null
// uploadedTextprompt = null
// userInput = null

// // 텍스트를 청크로 분할하는 함수
// function split_text(text, max_tokens = 1000) {
//     const words = text.split(/\s+/); // 텍스트를 단어 단위로 분할
//     const chunks = []; // 분할된 청크를 저장할 배열
//     let current_chunk = []; // 현재 청크
//     let current_length = 0; // 현재 청크의 토큰 수

//     for (const word of words) {
//         // 현재 단어를 추가했을 때 max_tokens를 초과하는지 확인
//         if (current_length + word.length + 1 <= max_tokens) {
//             current_chunk.push(word);
//             current_length += word.length + 1; // 단어 길이 + 공백
//         } else {
//             // 현재 청크를 완성하고 청크 리스트에 추가
//             chunks.push(current_chunk.join(' '));
//             current_chunk = [word]; // 새로운 청크 시작
//             current_length = word.length; // 현재 청크의 토큰 수 초기화
//         }
//     }

//     // 마지막 청크가 남아있는 경우 추가
//     if (current_chunk.length > 0) {
//         chunks.push(current_chunk.join(' '));
//     }

//     return chunks;
// }

// document.getElementById('sendButton').addEventListener('click', sendMessage);
// document.getElementById('userInput').addEventListener('keypress', function (e) {
//     if (e.key === 'Enter') {
//         e.preventDefault();
//         sendMessage();
//     }
// });

// function sendMessage() {
//     const userInput = document.getElementById('userInput').value.trim();
//     // if (!userInput) return;
    
//     // 좌측: 사용자 입력 추가
//     addMessage(userInput, 'user', 'inputContainer');
//     document.getElementById('userInput').value = '';

//     // write_upload API 호출
//     if (uploadedText && uploadedTextprompt) {
//         fetchBotResponse(userInput, uploadedText, uploadedTextprompt);
//     } else {
//         addMessage('ファイルがアップロードされていません。', 'bot', 'outputContainer');
//     }
// }

// // 첨부파일 업로드 기능 추가
// document.querySelector('.f02f0e25').addEventListener('click', function () {
//     // 파일 선택기 생성
//     const fileInput = document.createElement('input');
//     fileInput.type = 'file';
//     fileInput.accept = 'application/pdf'; // PDF 파일만 허용

//     fileInput.onchange = async (event) => {
//         const file = event.target.files[0];
//         if (file) {
//             const formData = new FormData();
//             formData.append('file', file);

//             try {
//                 // 1. 파일 업로드 및 원본 텍스트 가져오기
//                 const uploadResponse = await fetch(UPLOAD_API_URL, {
//                     method: 'POST',
//                     body: formData
//                 });

//                 if (!uploadResponse.ok) {
//                     throw new Error(`サーバーエラー: ${uploadResponse.status} ${uploadResponse.statusText}`);
//                 }

//                 const uploadResult = await uploadResponse.json();
//                 if (!uploadResult.success) {
//                     throw new Error(uploadResult.error || 'ファイルの処理中にエラーが発生しました。');
//                 }

//                 // 좌측: PDF에서 읽어온 원본 텍스트 표시
//                 if (uploadResult.original_text) {
//                     uploadedText = uploadResult.original_text
//                     addMessage(uploadResult.original_text, 'user', 'inputContainer');
//                 } else {
//                     throw new Error('PDF 读取发生错误.');
//                 }
// ;
//             } catch (error) {
//                 console.error('ファイルのアップロード中にエラーが発生しました:', error);
//                 addMessage(`ファイルのアップロード中にエラーが発生しました: ${error.message}`, 'bot', 'outputContainer');
//             }
//         }
        
//     };

//     fileInput.click(); // 파일 선택기 열기
// });

// // 첨부파일 업로드 기능 추가
// document.querySelector('.prompt').addEventListener('click', function () {
//     // 파일 선택기 생성
//     const fileInput = document.createElement('input');
//     fileInput.type = 'file';
//     fileInput.accept = 'text/plain'; // PDF 및 TXT 파일 허용

//     fileInput.onchange = async (event) => {
//         const file = event.target.files[0];
//         if (file) {
//             const formData = new FormData();
//             formData.append('file', file);

//             try {
//                 // 파일 확장자 확인
//                 const fileExtension = file.name.split('.').pop().toLowerCase();
//                 if (fileExtension !== 'txt') {
//                     throw new Error('TXTまたはTXTファイルのみアップロードできます。');
//                 }

//                 // 1. 파일 업로드 및 원본 텍스트 가져오기
//                 const uploadResponse = await fetch(UPLOAD_API_URL, {
//                     method: 'POST',
//                     body: formData
//                 });

//                 if (!uploadResponse.ok) {
//                     throw new Error(`サーバーエラー: ${uploadResponse.status} ${uploadResponse.statusText}`);
//                 }

//                 const uploadResult = await uploadResponse.json();
//                 if (!uploadResult.success) {
//                     throw new Error(uploadResult.error || 'ファイルの処理中にエラーが発生しました。');
//                 }

//                 // 좌측: 파일에서 읽어온 원본 텍스트 표시
//                 if (uploadResult.prompt_text) {
//                     uploadedTextprompt = uploadResult.prompt_text; // 업로드된 prompt텍스트 저장
//                     addMessage(uploadResult.prompt_text, 'user', 'inputContainer');
//                 } else {
//                     throw new Error('ファイルの読み取り中にエラーが発生しました。');
//                 }
//             } catch (error) {
//                 console.error('ファイルのアップロード中にエラーが発生しました:', error);
//                 addMessage(`ファイルのアップロード中にエラーが発生しました: ${error.message}`, 'bot', 'outputContainer');
//             }
//         }
//     };

//     fileInput.click(); // 파일 선택기 열기
// });

// function addMessage(content, sender, containerId) {
//     const messageContainer = document.createElement('div');
//     messageContainer.classList.add('message', sender);

//     // content가 undefined 또는 null인 경우 빈 문자열로 대체
//     const safeContent = content || '';

//     // 줄바꿈을 <br> 태그로 변환
//     const formattedContent = safeContent.replace(/\n/g, '<br>');

//     messageContainer.innerHTML = sender === 'user' 
//         ? `<strong>あなた:</strong><div>${formattedContent}</div>` 
//         : `<strong>T-STARヘルプ AI:</strong><div>${formattedContent}</div>`;

//     document.getElementById(containerId).appendChild(messageContainer);

//     // 스크롤을 최신 메시지로 이동
//     const container = document.getElementById(containerId);
//     container.scrollTop = container.scrollHeight;

//     // 우측에도 동일한 높이의 빈 메시지 추가 (라인 맞추기)
//     if (sender === 'user') {
//         const emptyMessage = document.createElement('div');
//         emptyMessage.classList.add('message', 'empty');
//         emptyMessage.style.visibility = 'hidden'; // 보이지 않도록 처리
//         document.getElementById('outputContainer').appendChild(emptyMessage);
//     }
// }

// async function fetchBotResponse(userInput,originalText,uploadedTextprompt) {
//     try {
//         const response = await fetch(WRITE_API_URL, {
//             method: 'POST',
//             headers: {
//                 'Content-Type': 'application/json'
//             },
//             body: JSON.stringify({ input: userInput ? userInput + uploadedTextprompt : uploadedTextprompt, original_text: originalText})
//         });
//         const data = await response.json();

//         // 파일 다운로드 처리
//         const blob = new Blob([data.corrected_text], { type: 'text/plain' }); // 텍스트 파일로 Blob 생성
//         const url = window.URL.createObjectURL(blob); // Blob URL 생성
//         const a = document.createElement('a'); // 다운로드 링크 생성
//         a.href = url;

//         // 파일 타입에 따라 다운로드 파일명 설정
//         a.download = 'annotated.txt'; // 텍스트 파일명 설정

//         a.click(); // 다운로드 실행
//         window.URL.revokeObjectURL(url); // Blob URL 해제

//         // 우측: GPT 출력 추가
//         addMessage(data.corrected_text || '回答が見つかりませんでした。', 'bot', 'outputContainer');
//     } catch (error) {
//         console.error('Error fetching response:', error);
//         addMessage('エラーが発生しました。もう一度お試しください。', 'bot', 'outputContainer');
//     }
// }

const API_URL = 'https://namcheckweb.azurewebsites.net/ask_gpt'; // GPT API URL
const UPLOAD_API_URL = 'https://namcheckweb.azurewebsites.net/api/check_upload'; // 파일 업로드 API URL
const WRITE_API_URL = 'https://namcheckweb.azurewebsites.net/api/write_upload'; // 파일 업로드 API URL

// 텍스트를 청크로 분할하는 함수
// function split_text(text, max_tokens = 4000) {
//     const words = text.split(/\s+/); // 텍스트를 단어 단위로 분할
//     const chunks = []; // 분할된 청크를 저장할 배열
//     let current_chunk = []; // 현재 청크
//     let current_length = 0; // 현재 청크의 토큰 수

//     for (const word of words) {
//         // 현재 단어를 추가했을 때 max_tokens를 초과하는지 확인
//         if (current_length + word.length + 1 <= max_tokens) {
//             current_chunk.push(word);
//             current_length += word.length + 1; // 단어 길이 + 공백
//         } else {
//             // 현재 청크를 완성하고 청크 리스트에 추가
//             chunks.push(current_chunk.join(' '));
//             current_chunk = [word]; // 새로운 청크 시작
//             current_length = word.length; // 현재 청크의 토큰 수 초기화
//         }
//     }

//     // 마지막 청크가 남아있는 경우 추가
//     if (current_chunk.length > 0) {
//         chunks.push(current_chunk.join(' '));
//     }

//     return chunks;
// }

function split_text(text, maxLength) {
    const chunks = [];
    let startIndex = 0;
    while (startIndex < text.length) {
      const endIndex = startIndex + maxLength;
      chunks.push(text.slice(startIndex, endIndex));
      startIndex = endIndex;
    }
    return chunks;
  }
  

  
document.getElementById('sendButton').addEventListener('click', sendMessage);
document.getElementById('userInput').addEventListener('keypress', function (e) {
    if (e.key === 'Enter') {
        e.preventDefault();
        sendMessage();
    }
});

function sendMessage() {
    const userInput = document.getElementById('userInput').value.trim();
    if (!userInput) return;

    // 좌측: 사용자 입력 추가
    addMessage(userInput, 'user', 'inputContainer');
    document.getElementById('userInput').value = '';

    // Fetch bot response from /api/ppt
    fetchBotResponse(userInput);
}

// 첨부파일 업로드 기능 추가 (PDF 및 Excel)
// document.querySelector('.f02f0e25').addEventListener('click', function () {
//     // 파일 선택기 생성
//     const fileInput = document.createElement('input');
//     fileInput.type = 'file';
//     fileInput.accept = 'application/pdf, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'; // PDF 및 Excel 파일 허용

//     fileInput.onchange = async (event) => {
//         const file = event.target.files[0];
//         if (file) {
//             const formData = new FormData();
//             formData.append('file', file);

//             try {
//                 // 1. 파일 업로드 및 원본 텍스트 가져오기
//                 const uploadResponse = await fetch(UPLOAD_API_URL, {
//                     method: 'POST',
//                     body: formData
//                 });

//                 if (!uploadResponse.ok) {
//                     throw new Error(`サーバーエラー: ${uploadResponse.status} ${uploadResponse.statusText}`);
//                 }

//                 const uploadResult = await uploadResponse.json();
//                 if (!uploadResult.success) {
//                     throw new Error(uploadResult.error || 'ファイルの処理中にエラーが発生しました。');
//                 }

//                 // 좌측: 파일에서 읽어온 원본 텍스트 표시
//                 if (uploadResult.original_text) {
//                     addMessage(uploadResult.original_text, 'user', 'inputContainer');
//                 } else {
//                     throw new Error('ファイルの読み取り中にエラーが発生しました。');
//                 }

//                 // 2. 원본 텍스트를 ask_gpt API로 전달 (토큰 수 제한 적용)
//                 const textChunks = split_text(uploadResult.original_text, 4000); // 텍스트를 4000 토큰 단위로 분할
//                 const gptResults = []; // GPT 모델의 처리 결과를 저장할 배열

//                 for (const chunk of textChunks) {
//                     try {
//                         const gptResponse = await fetch(WRITE_API_URL, {
//                             method: 'POST',
//                             headers: {
//                                 'Content-Type': 'application/json'
//                             },
//                             body: JSON.stringify({
//                                 input: chunk,
//                                 pdf_bytes: uploadResult.pdf_bytes,
//                                 excel_bytes: uploadResult.excel_bytes // 엑셀 파일 데이터 추가
//                             })
//                         });

//                         if (!gptResponse.ok) {
//                             throw new Error(`GPT APIエラー: ${gptResponse.status} ${gptResponse.statusText}`);
//                         }

//                         // 파일 다운로드 처리
//                         // const blob = await gptResponse.blob();
//                         // const url = window.URL.createObjectURL(blob);
//                         // const a = document.createElement('a');
//                         // a.href = url;

//                         // // 파일 타입에 따라 다운로드 파일명 설정
//                         // if (file.type === 'application/pdf') {
//                         //     a.download = 'annotated.pdf';
//                         // } else if (file.type === 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet') {
//                         //     a.download = 'annotated.xlsx';
//                         // }

//                         // a.click();
//                         // window.URL.revokeObjectURL(url);

//                         // addMessage(`ファイルのダウンロードが完了しました！`, 'bot', 'outputContainer');
//                         const data = await gptResponse.json();
//                         // 우측: GPT 출력 추가
//                         addMessage(data.corrected_text || '回答が見つかりませんでした。', 'bot', 'outputContainer');
//                     } catch (error) {
//                         console.error('GPT API エラー:', error);
//                         addMessage(`GPT API エラー: ${error.message}`, 'bot', 'outputContainer');
//                         return; // 에러 발생 시 다음 청크로 넘어감
//                     }
//                 }
//             } catch (error) {
//                 console.error('ファイルのアップロード中にエラーが発生しました:', error);
//                 addMessage(`ファイルのアップロード中にエラーが発生しました: ${error.message}`, 'bot', 'outputContainer');
//             }
            
//         }
//     };

//     fileInput.click(); // 파일 선택기 열기
// });

let loadingInterval = null;
let loadingMessageElement = null;

// /**
//  * 로딩 메시지 추가 및 애니메이션 시작
//  */
// function showLoadingMessage() {
//     // 봇 메시지 형태로 “...” 추가
//     loadingMessageElement = document.createElement('div');
//     loadingMessageElement.classList.add('message', 'bot');
//     loadingMessageElement.innerHTML = '<strong>NOMURA AI:</strong> <div>解析中...</div>';
    
//     // outputContainer에 추가
//     const outputContainer = document.getElementById('outputContainer');
//     outputContainer.appendChild(loadingMessageElement);
//     outputContainer.scrollTop = outputContainer.scrollHeight;

//     // 0.5초 간격으로 “.” 갯수를 순환
//     let dotCount = 1;
//     loadingInterval = setInterval(() => {
//         dotCount = (dotCount % 6) + 1; // 1~3 순환
//         loadingMessageElement.innerHTML = `<strong>NOMURA AI:</strong> <div>${'.'.repeat(dotCount)}</div>`;
//     }, 500);
// }

// /**
//  * 로딩 메시지를 제거(또는 갱신)
//  */
// function hideLoadingMessage() {
//     if (loadingInterval) {
//         clearInterval(loadingInterval);
//         loadingInterval = null;
//     }
//     if (loadingMessageElement) {
//         // 메시지를 아예 지우려면 remove()
//         // loadingMessageElement.remove();

//         // 혹은 메시지를 “응답 완료” 등으로 교체
//         // loadingMessageElement.innerHTML = '<strong>NOMURA AI:</strong> <div>応答を受信しました。</div>';
//         // 필요하면 여기서도 제거 가능

//         loadingMessageElement.remove();
//         loadingMessageElement = null;
//     }
// }
/**
 * 로딩 메시지 표시 함수
 */
function showLoadingMessage() {
    // 봇 메시지 형태로 “思考中...” 추가
    loadingMessageElement = document.createElement('div');
    loadingMessageElement.classList.add('message', 'bot');

    // 처음에는 점 하나만 표시
    loadingMessageElement.innerHTML = '<strong>NOMURA AI:</strong><div>考えている.</div>';

    // outputContainer에 추가
    const outputContainer = document.getElementById('outputContainer');
    outputContainer.appendChild(loadingMessageElement);
    outputContainer.scrollTop = outputContainer.scrollHeight;

    // 0.5초 간격으로 “思考中.” → “思考中..” → “思考中...” → 반복
    let dotCount = 1;
    loadingInterval = setInterval(() => {
        dotCount = (dotCount % 3) + 1; // 1~3 순환
        loadingMessageElement.innerHTML =
            `<strong>NOMURA AI:</strong><div>考えている${'.'.repeat(dotCount)}</div>`;
    }, 500);
}

/**
 * 로딩 메시지 제거 함수
 */
function hideLoadingMessage() {
    if (loadingInterval) {
        clearInterval(loadingInterval);
        loadingInterval = null;
    }
    if (loadingMessageElement) {
        // 메시지를 완전히 제거
        loadingMessageElement.remove();
        loadingMessageElement = null;
    }
}

// 첨부파일 업로드 기능 추가 (PDF 및 Excel)
document.querySelector('.f02f0e25').addEventListener('click', function () {
    // 파일 선택기 생성
    const fileInput = document.createElement('input');
    fileInput.type = 'file';
    fileInput.accept = 'application/pdf, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'; // PDF 및 Excel 파일 허용

    fileInput.onchange = async (event) => {
        const file = event.target.files[0];
        if (file) {
            const formData = new FormData();
            formData.append('file', file);

            try {
                // 1. 파일 업로드 및 원본 텍스트 가져오기
                const uploadResponse = await fetch(UPLOAD_API_URL, {
                    method: 'POST',
                    body: formData
                });

                if (!uploadResponse.ok) {
                    throw new Error(`サーバーエラー: ${uploadResponse.status} ${uploadResponse.statusText}`);
                }

                const uploadResult = await uploadResponse.json();
                if (!uploadResult.success) {
                    throw new Error(uploadResult.error || 'ファイルの処理中にエラーが発生しました。');
                }

                // 좌측: 파일에서 읽어온 원본 텍스트 표시
                if (uploadResult.original_text) {
                    addMessage(uploadResult.original_text, 'user', 'inputContainer');
                } else {
                    throw new Error('ファイルの読み取り中にエラーが発生しました。');
                }

                // 2. 원본 텍스트를 여러 청크로 분할 (문자 수 기준 2000)
                const textChunks = split_text(uploadResult.original_text, 2000); 
                
                // GPT 모델의 결과들을 저장할 배열
                const gptResults = [];

                // 3. 각 청크마다 write_upload API 호출
                for (const chunk of textChunks) {
                    try {
                        // 로딩 시작
                        showLoadingMessage()
                        const gptResponse = await fetch(WRITE_API_URL, {
                            method: 'POST',
                            headers: {
                                'Content-Type': 'application/json'
                            },
                            body: JSON.stringify({
                                input: chunk,
                                pdf_bytes: uploadResult.pdf_bytes,
                                excel_bytes: uploadResult.excel_bytes // 엑셀 파일 데이터 추가
                            })
                        });

                        // 로딩 종료
                        hideLoadingMessage();

                        if (!gptResponse.ok) {
                            throw new Error(`GPT APIエラー: ${gptResponse.status} ${gptResponse.statusText}`);
                        }

                        // 만약 write_upload가 파일을 직접 반환하지 않고,
                        // JSON 형태라면 다음처럼 JSON 파싱
                        const data = await gptResponse.json();

                        // 청크별 결과 누적
                        gptResults.push(data.corrected_text || '');

                    } catch (error) {
                        console.error('GPT API エラー:', error);
                        addMessage(`GPT API エラー: ${error.message}`, 'bot', 'outputContainer');
                        // 특정 청크에서 에러가 발생하면, 다음 청크는 계속 처리할 지,
                        // 중단할 지는 선택 사항. 여기서는 그냥 return해서 중단 예시
                        return;
                    }
                }

                // 4. 모든 청크 처리 후 결과를 하나로 합침
                //    (줄바꿈이나 구분자를 원하시면 '\n\n' 등으로 변경)
                const finalAnswer = gptResults.join('\n\n');
                
                // 5. 우측: 최종 GPT 출력 추가
                addMessage(finalAnswer, 'bot', 'outputContainer');

            } catch (error) {
                console.error('ファイルのアップロード中にエラーが発生しました:', error);
                addMessage(`ファイルのアップロード中にエラーが発生しました: ${error.message}`, 'bot', 'outputContainer');
            }
        }
    };

    fileInput.click(); // 파일 선택기 열기
});
// split 
// document.querySelector('.f02f0e25').addEventListener('click', function () {
//     // 파일 선택기 생성
//     const fileInput = document.createElement('input');
//     fileInput.type = 'file';
//     fileInput.accept = 'application/pdf, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'; // PDF 및 Excel 파일 허용

//     fileInput.onchange = async (event) => {
//         const file = event.target.files[0];
//         if (file) {
//             const formData = new FormData();
//             formData.append('file', file);

//             try {
//                 // 1. 파일 업로드
//                 const uploadResponse = await fetch(UPLOAD_API_URL, {
//                     method: 'POST',
//                     body: formData
//                 });
//                 if (!uploadResponse.ok) {
//                     throw new Error(`サーバーエラー: ${uploadResponse.status} ${uploadResponse.statusText}`);
//                 }
//                 const uploadResult = await uploadResponse.json();
//                 if (!uploadResult.success) {
//                     throw new Error(uploadResult.error || 'ファイルの処理中にエラーが発生しました。');
//                 }

//                 // 좌측에 원본 텍스트 출력
//                 if (uploadResult.original_text) {
//                     addMessage(uploadResult.original_text, 'user', 'inputContainer');
//                 } else {
//                     throw new Error('ファイルの読み取り中にエラーが発生しました。');
//                 }

//                 // 2. 원본 텍스트를 2,000자 단위로 분할
//                 const textChunks = split_text(uploadResult.original_text, 2000);

//                 // 3. 각 청크를 순차적으로 전송 → 응답이 도착하면 즉시 화면에 표시
//                 for (const chunk of textChunks) {
//                     try {
//                         const gptResponse = await fetch(WRITE_API_URL, {
//                             method: 'POST',
//                             headers: {
//                                 'Content-Type': 'application/json'
//                             },
//                             body: JSON.stringify({
//                                 input: chunk,
//                                 pdf_bytes: uploadResult.pdf_bytes,
//                                 excel_bytes: uploadResult.excel_bytes
//                             })
//                         });

//                         if (!gptResponse.ok) {
//                             throw new Error(`GPT APIエラー: ${gptResponse.status} ${gptResponse.statusText}`);
//                         }

//                         const data = await gptResponse.json();

//                         // 응답이 도착하자마자 화면에 표시
//                         addMessage(data.corrected_text || '回答が見つかりませんでした。', 'bot', 'outputContainer');

//                     } catch (error) {
//                         console.error('GPT API エラー:', error);
//                         addMessage(`GPT API エラー: ${error.message}`, 'bot', 'outputContainer');
//                         // 에러 발생 시 남은 청크 처리는 생략(또는 continue로 계속 진행)
//                         break;
//                     }
//                 }

//             } catch (error) {
//                 console.error('ファイルのアップロード中にエラーが発生しました:', error);
//                 addMessage(`ファイルのアップロード中にエラーが発生しました: ${error.message}`, 'bot', 'outputContainer');
//             }
//         }
//     };

//     fileInput.click(); // 파일 선택기 열기
// });


function addMessage(content, sender, containerId) {
    const messageContainer = document.createElement('div');
    messageContainer.classList.add('message', sender);

    // content가 undefined 또는 null인 경우 빈 문자열로 대체
    const safeContent = content || '';

    // 줄바꿈을 <br> 태그로 변환
    const formattedContent = safeContent.replace(/\n/g, '<br>');

    messageContainer.innerHTML = sender === 'user' 
        ? `<strong>あなた:</strong><div>${formattedContent}</div>` 
        : `<strong>Nomura AI:</strong><div>${formattedContent}</div>`;

    document.getElementById(containerId).appendChild(messageContainer);

    // 스크롤을 최신 메시지로 이동
    const container = document.getElementById(containerId);
    container.scrollTop = container.scrollHeight;

    // 우측에도 동일한 높이의 빈 메시지 추가 (라인 맞추기)
    if (sender === 'user') {
        const emptyMessage = document.createElement('div');
        emptyMessage.classList.add('message', 'empty');
        emptyMessage.style.visibility = 'hidden'; // 보이지 않도록 처리
        document.getElementById('outputContainer').appendChild(emptyMessage);
    }
}

// async function fetchBotResponse(userInput) {
//     const textChunks = split_text(userInput, 4000); // 텍스트를 4000 토큰 단위로 분할
//     const gptResults = []; // GPT 모델의 처리 결과를 저장할 배열

//     for (const chunk of textChunks) {

//         try {
//             const response = await fetch(API_URL, {
//                 method: 'POST',
//                 headers: {
//                     'Content-Type': 'application/json'
//                 },
//                 body: JSON.stringify({ input: chunk })
//             });
//             const data = await response.json();
//             // const data = await response.json('\n\n');


//             // 우측: GPT 출력 추가
//             addMessage(data.answer || '回答が見つかりませんでした。', 'bot', 'outputContainer');
//         } catch (error) {
//             console.error('Error fetching response:', error);
//             addMessage('エラーが発生しました。もう一度お試しください。', 'bot', 'outputContainer');
//         }
//     }
// }



async function fetchBotResponse(userInput) {
    // 1) 긴 텍스트를 2000자 단위로 나누기
    const textChunks = split_text(userInput, 1000);
    // GPT 모델의 결과들을 저장할 배열
    const gptResults = [];

    // 2) 각 청크를 순차적으로 API에 보내기
    for (const chunk of textChunks) {
        try {
            showLoadingMessage();
            const response = await fetch(API_URL, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ input: chunk })
            });

            // 로딩 종료
            hideLoadingMessage();

            if (!response.ok) {
                throw new Error(`サーバーエラー: ${response.status} ${response.statusText}`);
            }

            // 3) API 응답 받기
            const data = await response.json();

            // 청크별 결과 누적
            gptResults.push(data.answer || '');
            

        } catch (error) {
            console.error('GPT API エラー:', error);
            addMessage(`GPT API エラー: ${error.message}`, 'bot', 'outputContainer');
            // 특정 청크에서 에러가 발생하면, 다음 청크는 계속 처리할 지,
            // 중단할 지는 선택 사항. 여기서는 그냥 return해서 중단 예시
            return;
        }
    }
    // 4. 모든 청크 처리 후 결과를 하나로 합침
    //    (줄바꿈이나 구분자를 원하시면 '\n\n' 등으로 변경)
    const finalAnswer = gptResults.join('\n\n');
    
    // 5. 우측: 최종 GPT 출력 추가
    addMessage(finalAnswer, 'bot', 'outputContainer');

}

// async function fetchBotResponse(userInput) {
//         try {
//             const response = await fetch(API_URL, {
//                 method: 'POST',
//                 headers: {
//                     'Content-Type': 'application/json'
//                 },
//                 body: JSON.stringify({ input: userInput })
//             });
//             const data = await response.json();
//             // const data = await response.json('\n\n');


//             // 우측: GPT 출력 추가
//             addMessage(data.answer || '回答が見つかりませんでした。', 'bot', 'outputContainer');
//         } catch (error) {
//             console.error('Error fetching response:', error);
//             addMessage('エラーが発生しました。もう一度お試しください。', 'bot', 'outputContainer');
//         }
// }

// async function fetchBotResponse(userInputs) {
//     try {
//         // 단일 입력인 경우 배열로 변환
//         if (!Array.isArray(userInputs)) {
//             userInputs = [userInputs];
//         }

//         // 여러 입력에 대해 동시에 요청을 보냄
//         const promises = userInputs.map(async (userInput) => {
//             const response = await fetch(API_URL, {
//                 method: 'POST',
//                 headers: {
//                     'Content-Type': 'application/json'
//                 },
//                 body: JSON.stringify({ input: userInput })
//             });
//             return response.json();
//         });

//         // 모든 요청이 완료될 때까지 기다림
//         const results = await Promise.all(promises);

//         // 각 결과를 처리
//         results.forEach((data) => {
//             addMessage(data.answer || '回答が見つかりませんでした。', 'bot', 'outputContainer');
//         });
//     } catch (error) {
//         console.error('Error fetching response:', error);
//         addMessage('エラーが発生しました。もう一度お試しください。', 'bot', 'outputContainer');
//     }
// }
```

```json, 2025-16:45
{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3Mzg2NTg0MTksIm5iZiI6MTczODY1ODQxOSwiZXhwIjoxNzM4NzQ1MTE5LCJhaW8iOiJrMlJnWUxDWGptbXIxbUYvL3YvY3BiNVRkeDR3QWdBPSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiJPWGVQaG1kNUdFQ1E2OEZnVmRVaEFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiMzAgNyIsInhtc19taXJpZCI6Ii9zdWJzY3JpcHRpb25zLzNiZDNkN2Q5LWE1N2QtNGZiZS04NmU1LWViMDRiYmQ2YmJiZS9yZXNvdXJjZWdyb3Vwcy9ucmlwaS1jb21tZW50Y2hlY2stcmcvcHJvdmlkZXJzL01pY3Jvc29mdC5XZWIvc2l0ZXMvTmFtQ2hlY2tXZWIifQ.azDmL6ndAqCXfUjGnK3c23GZawWfWvLj02oETnDEt673LN6lXIHAEogg1WvcMY1rsTuSk7SiyGqY-gS4iSHPqp8oWAUmMW-2MvsCGnZQsi6wtn9dEgf7gU0_dCTMbpnbPAYhCusSyDc3aMBK-ign4zdChFa91QmXFBye_gZi4ycSa5OqaC6HpaTocm7M6JU_TBwwCoKMG3PvtxkTCcFXZDqym9uaDUvsaz2QFbhAppBMMJQmjXsOTPcKnY_c-EbOt1KTtfHKoYhwrH5igmaN18qCYHmsxk1-9f-8gb6yzpwid9VKr5KHN5d94S-6D8K8S44F7Vnq2ZErHVA3QIyL0Q"}
```
```json, 25-17:07
{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3Mzg2NTk3MDgsIm5iZiI6MTczODY1OTcwOCwiZXhwIjoxNzM4NzQ2NDA4LCJhaW8iOiJrMlJnWUxpN1dTNU9XMi9Xbko4bkxjc1RQalM5QXdBPSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiIzWThhX3o1T3lFR2dkRnBNNHZzUUFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiMiA3IiwieG1zX21pcmlkIjoiL3N1YnNjcmlwdGlvbnMvM2JkM2Q3ZDktYTU3ZC00ZmJlLTg2ZTUtZWIwNGJiZDZiYmJlL3Jlc291cmNlZ3JvdXBzL25yaXBpLWNvbW1lbnRjaGVjay1yZy9wcm92aWRlcnMvTWljcm9zb2Z0LldlYi9zaXRlcy9OYW1DaGVja1dlYiJ9.TXiiD1cotRn9kkG0bPuqVZ_blAyqO0nVbjsCYOhKrtcUMgdubSEdV5FPwqEcCKIv9a3axa4bIuGi846zFFiTqFBm3cJvBuL5gWddYleFxEGIWLRbuiUzaghzkmrs5Dl5e2FCduM0jZX4pfxfxDxAQu2iKwPPpM9lzCQz66__xB20KGUWiu0RJwNkuL9tShUHtilH2zOiYwZi99oquCMEBzzBS-_pe1IZDq0JNoLVE8f_Dox_h41Jb-MOzQQ-GIIAR52rrGCn-7NIGCon5qNpTUPoTkAaBvgVfb0ogLarem3XVCnEYHCUydYO8oZ-jOxzfrXrt-uHDAp6u7hDzcgPgQ"}


{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3Mzg2NjgyMzUsIm5iZiI6MTczODY2ODIzNSwiZXhwIjoxNzM4NzU0OTM1LCJhaW8iOiJrMlJnWUxDWGptbXIxbUYvL3YvY3BiNVRkeDR3QWdBPSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiJsdnk2Zm9JYnVVMkg2SzZQV0ZrWEFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiNyAxMiIsInhtc19taXJpZCI6Ii9zdWJzY3JpcHRpb25zLzNiZDNkN2Q5LWE1N2QtNGZiZS04NmU1LWViMDRiYmQ2YmJiZS9yZXNvdXJjZWdyb3Vwcy9ucmlwaS1jb21tZW50Y2hlY2stcmcvcHJvdmlkZXJzL01pY3Jvc29mdC5XZWIvc2l0ZXMvTmFtQ2hlY2tXZWIifQ.ItuWI6lslss2o-O6ZuwoAac_hu_Mi72-PT5yPrsCV-fxk6lmm00VslKViiSlJ_KUQjEgecgmcrL5DsvN9zPORdDW8AxTvsJxIc845y8M0hfEioctPtFK7-aGHA3IHUP-XK567fBBqLzsaIFn8kzjklDxocu3fpzmp6k_WfIbasXKP7mkb3545E53gOV7W0cR9eapTqSW5_A1cTAPAXyqMVTPM3Ts0vytfSqDMNEKzKcKceogqdRjOSCZ_q37Bd-4oxJ2SjkNvVl8JwFSHU2Na-gW2Vtu5OBO82ooPDisBMZgwAqTMkIFq-i5mNbjgtMpSFBiKzTUPZIin9zo5TmCPQ"}
```


```json
 corrected_map = {
                "地政学リスク": "地政学的リスク",
                "マイナスに寄与": "マイナスに影響",
                "債券利回りは下落": "債券利回りは低下",
                "価格は低下": "価格は下落",
                "金利の下落": "金利の低下",
                "への組み入れ": "の組み入れ",
                "政治的リスク": "政治リスク",
                "日本銀行": "日銀",
                "中央銀行": "中央銀行",
                "伸張": "伸長",
                "立ち後れ": "立ち遅れ",
                "経済正常化": "経済活動正常化",
                "金融正常化": "金融政策正常化",
                "米国国債": "米国債",
                "コロナウイルス": "新型コロナウイルス",
                "新型コロナ": "新型コロナウイルス",
                "ロシアとウクライナとの戦争": "ロシアによるウクライナへの軍事侵攻",
                "ダウ平均": "ダウ平均株価",
                "NYダウ": "ダウ平均株価",
                "1ヵ月前": "前月",
                "先月": "前月",
                "前期比": "前期比年率",
                "蓋然性": "可能性",
                "商い": "取引",
                "後倒し": "延期",
                "薄商い": "活況でなく",
                "に賭ける": "を予想して",
                "コア銘柄": "中核銘柄",
                "トリガー": "きっかけ",
                "ブルーチップ企業": "優良企業",
                "トレンド": "傾向",
                "レンジ": "範囲",
                "回金": "円転",
                "所謂": "いわゆる",
                "暫く": "しばらく",
                "留まる": "とどまる",
                "止まる": "とどまる",
                "尚": "なお",
                "筈": "はず",
                "殆ど": "ほとんど",
                "真似": "まね",
                "亘る": "わたる",
                "但し": "ただし",
                "牽制": "けん制",
                "牽引": "けん引",
                "終焉": "終えん",
                "収斂": "収れん",
                "逼迫": "ひっ迫",
                "横這い": "横ばい",
                "ヶ月": "ヵ月",
                "入替え": "入れ替え",
                "行う": "行なう",
                "行われる": "行なわれる",
                "買付": "買い付け",
                "買付け": "買い付け",
                "売付": "売り付け",
                "売付け": "売り付け",
                "格付": "格付け",
                "国債買い入れ": "国債買入れ",
                "国債買入オペ": "国債買入",
                "買増し": "買い増し",
                "売増し": "売り増し",
                "買建て": "買い建て",
                "売建て": "売り建て",
                "切上げ": "切り上げ",
                "切捨て": "切り捨て",
                "組入れ": "組み入れ",
                "繰上げ償還": "繰上償還",
                "先き行き": "先行き",
                "下支える": "下支えする",
                "取り引き": "取引",
                "引上げ": "引き上げ",
                "引下げ": "引き下げ",
                "引締め": "引き締め",
                "引続き": "引き続き",
            }


```


```json, 25=0205,17:24
{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3Mzg3NDcxMjcsIm5iZiI6MTczODc0NzEyNywiZXhwIjoxNzM4ODMzODI3LCJhaW8iOiJrMlJnWU5oOWJmTWE5UTJhTjV4ODUzdHBuQ3VJQkFBPSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiJzT3pIZENHakFFYTBQSkdydmJjYkFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiNyAyOCIsInhtc19taXJpZCI6Ii9zdWJzY3JpcHRpb25zLzNiZDNkN2Q5LWE1N2QtNGZiZS04NmU1LWViMDRiYmQ2YmJiZS9yZXNvdXJjZWdyb3Vwcy9ucmlwaS1jb21tZW50Y2hlY2stcmcvcHJvdmlkZXJzL01pY3Jvc29mdC5XZWIvc2l0ZXMvTmFtQ2hlY2tXZWIifQ.RGyx1f7wheRw51R6euzm-SvZ9_7N2KjfWUJZsVzK8iuoU40v_KmRePgrX8XWpzx6909vpAxa8ZIZQj_nYYZEqEwDi36BuWiOqusLuYLRH0L7_Fjl5EPZhPLavaX15qWcc8mkEiSBSn2JrBwXKKD3-mSZrqHd7m4dBb8q_Nj4Wx4W8Oo_9hDuo0RnRkd2SIKmfJzdiMYtSfmsaiNgcq1G0JgmsqB6WDhIx82L8CD7cAUJAO21es6XKNSgiAvLBfQQgDSm2tf0i47TP6s8ufraf1K3dTb3brtgndbmGl1q_ud307cpjP5e4WxweCYXErtaSE1av-p2INfyCxRZutUKmA"}


25-0206-16:58
{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3Mzg4MzE4NzYsIm5iZiI6MTczODgzMTg3NiwiZXhwIjoxNzM4OTE4NTc2LCJhaW8iOiJrMlJnWU9oeU5HNzhQU2tqL0FaM3pCT0RHbTFiQUE9PSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiJFWENEVExTenJVNktuVnBoV0FJSUFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiNyAxMiIsInhtc19taXJpZCI6Ii9zdWJzY3JpcHRpb25zLzNiZDNkN2Q5LWE1N2QtNGZiZS04NmU1LWViMDRiYmQ2YmJiZS9yZXNvdXJjZWdyb3Vwcy9ucmlwaS1jb21tZW50Y2hlY2stcmcvcHJvdmlkZXJzL01pY3Jvc29mdC5XZWIvc2l0ZXMvTmFtQ2hlY2tXZWIifQ.A5Y0siFLFZeoRZSixhTbEIz7DyXvwgvsz4jxS1vYTWVR9LU_bIspEUBVyUSeAx7rCXKgTtjI1jnsTXUUQ7n0aD0T0yqfxHNZGYGD7wTDa0JfpfVRE8XFHc407gfD9A86aS6B0A0jt_d2_X8jDSmfdhpmyXuxHBzSUmZtBH6-TTR0GJ20JGSYGYj0nZeD1m4E-554I9oy572jN51dyfESJ1Spq4qEnOYs6sjndahXSbPDZ2ns_TErpzRG2z_bwrZ_WhKLrOUesfuLu_HWCkKt7ELxUPVrxW1EOuBt3ryKsBTDUTDJ6KzKUVnIFe2YXbLICXI_COGQWKCvBru-Azdq8w"}

```

``` json, 17:09
{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3MzkxNzYyODksIm5iZiI6MTczOTE3NjI4OSwiZXhwIjoxNzM5MjYyOTg5LCJhaW8iOiJrMlJnWUNpSjgxMWUvNUZ4MndYbTEzZU85NGFmQVFBPSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiJhRnZMTjBBNWtVS0UyUi1NUVJnLUFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiNyAxMCIsInhtc19taXJpZCI6Ii9zdWJzY3JpcHRpb25zLzNiZDNkN2Q5LWE1N2QtNGZiZS04NmU1LWViMDRiYmQ2YmJiZS9yZXNvdXJjZWdyb3Vwcy9ucmlwaS1jb21tZW50Y2hlY2stcmcvcHJvdmlkZXJzL01pY3Jvc29mdC5XZWIvc2l0ZXMvTmFtQ2hlY2tXZWIifQ.g_84lxa-oug46Xess9JXKXDM1KzPlXWkfi7-Fqy9IbgYcmLgcsA0kyQ7x0jSG9frEMJtsczo_uyWSFtkVwnZ4UwYVYxObfgj8ER5tATxwdohXsd3p3WxKj6W06S6pkl9-Ne0Ov3IXm7q6R5uPic-_CccSeXQzLiNxKDDjGzcLaED6S-q1MTt5wnXFj4YhmTdCphbF4FN5NHXsDw7QHvvyuuHrNpiDIoWRzf7__J_FssJFoJYoNtJDaoBJ-GsrlvFNvbuRlvnlEPBtqoOFNuKzEpWTBMqeuzwGYRcJHveLqn16gYdxi9hwrC7xVnHjpj4WfHKEn-YvR59XHLIy6bpQw"}


17:23
{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3MzkxNzkwODUsIm5iZiI6MTczOTE3OTA4NSwiZXhwIjoxNzM5MjY1Nzg1LCJhaW8iOiJrMlJnWUpBNE5hZGg4MGJQdXlyRmxlV1o3dHZiQUE9PSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiJpcG9LS1JzVVJreTZmOWxwNTJJLUFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiMTQgNyIsInhtc19taXJpZCI6Ii9zdWJzY3JpcHRpb25zLzNiZDNkN2Q5LWE1N2QtNGZiZS04NmU1LWViMDRiYmQ2YmJiZS9yZXNvdXJjZWdyb3Vwcy9ucmlwaS1jb21tZW50Y2hlY2stcmcvcHJvdmlkZXJzL01pY3Jvc29mdC5XZWIvc2l0ZXMvTmFtQ2hlY2tXZWIifQ.RMG4OVVhlA5bsa2o59ZGLVWoTMYhcYO6Owj50Wz_ZZ0JK33UHrWP0kNDX71VgJ7s2dg6BGg9qQHIEK4o0CmWhXzhVDCrx_-h6NTd7Hg5ljQEfCdNEj9qTE9WleAAFzPXDjnfnUpg30xQiZZH2f6oc7Ih6MKLPcuOOkmKXE5MkIbPsKTpaK4S97K_LaRyjAGjzUyiV8l3Q6EWjK3oBuUC1ikmNeA2Mzitrx1BpE6G24duvol7xN9C5Nxg-1ByvVWoSuAcx2Pemug14Y7KumJwgjKWw88TEvlhYWf_T4YocsztxAnhSKWCfOsJDA-Q0xcTcIEnHxxbqhkREUdT1qLr3Q"}


17:34
{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3MzkxNzk2MjAsIm5iZiI6MTczOTE3OTYyMCwiZXhwIjoxNzM5MjY2MzIwLCJhaW8iOiJrMlJnWUpCZWNWYjhhRXNJMDVMenp0TWZoK2FzQmdBPSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiIteUpkaS1WTGhrS0lkT3NpeWhwUEFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiNyAxNiIsInhtc19taXJpZCI6Ii9zdWJzY3JpcHRpb25zLzNiZDNkN2Q5LWE1N2QtNGZiZS04NmU1LWViMDRiYmQ2YmJiZS9yZXNvdXJjZWdyb3Vwcy9ucmlwaS1jb21tZW50Y2hlY2stcmcvcHJvdmlkZXJzL01pY3Jvc29mdC5XZWIvc2l0ZXMvTmFtQ2hlY2tXZWIifQ.LuiOsNyPVn_3wUb4G1Q7vrSG97i-YtK6JcWFSgbTAaYudPLY8-GvVEIQqwJYtnMFT1q2OHLHkRdxI8iBoytuUYHUz2b0YpPhcXYaOZCoo541VOPMJ1KohBJOF8_LDklgquur_8eGPTZ7s2PkW2jTk_GcD2Jn_ILzwwksYGL-_-omgARgVeva3yX2IS1GB6VJ2KZiFGFj5yWv-oKi3Q-skVsmoszrLSxaGf5snLXRs75cj9xyos0VP48EFEb-DZWPsAkK2AOnipVhzZx5h3z6qZZHUSnE2DCit5uzxmReCEq6W_M8fPqMmLmnqJIl2YBDPn82eImN_wdxUqetufzRuQ"}


17:46

{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyIsImtpZCI6IllUY2VPNUlKeXlxUjZqekRTNWlBYnBlNDJKdyJ9.eyJhdWQiOiJodHRwczovL2NvZ25pdGl2ZXNlcnZpY2VzLmF6dXJlLmNvbSIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzY3MDBlMTIxLWZjNjktNDQ4ZC04OWZjLTIzMGRjYTZjMmMyNS8iLCJpYXQiOjE3MzkxODA1MzEsIm5iZiI6MTczOTE4MDUzMSwiZXhwIjoxNzM5MjY3MjMxLCJhaW8iOiJrMlJnWVBEZEZOUnB0eXU2YVdXbzdVbmI4OS8wQUE9PSIsImFwcGlkIjoiMzA5ZDYyMTgtNDEyMi00NjhjLTliODEtZmJkM2E4OGNjNjJkIiwiYXBwaWRhY3IiOiIyIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNjcwMGUxMjEtZmM2OS00NDhkLTg5ZmMtMjMwZGNhNmMyYzI1LyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZWI0MzAzY2UtZDViMC00MTdiLTg0Y2YtMGFjYWZjNWRjNjA2IiwicmgiOiIxLkFHc0FJZUVBWjJuOGpVU0pfQ01OeW13c0paQWlNWDNJS0R4SG9PMk9VM1NiYlcxckFBQnJBQS4iLCJzdWIiOiJlYjQzMDNjZS1kNWIwLTQxN2ItODRjZi0wYWNhZmM1ZGM2MDYiLCJ0aWQiOiI2NzAwZTEyMS1mYzY5LTQ0OGQtODlmYy0yMzBkY2E2YzJjMjUiLCJ1dGkiOiIyVzBLcjAtWGFrT3BlVWtyVzdGRUFBIiwidmVyIjoiMS4wIiwieG1zX2lkcmVsIjoiMTYgNyIsInhtc19taXJpZCI6Ii9zdWJzY3JpcHRpb25zLzNiZDNkN2Q5LWE1N2QtNGZiZS04NmU1LWViMDRiYmQ2YmJiZS9yZXNvdXJjZWdyb3Vwcy9ucmlwaS1jb21tZW50Y2hlY2stcmcvcHJvdmlkZXJzL01pY3Jvc29mdC5XZWIvc2l0ZXMvTmFtQ2hlY2tXZWIifQ.KxGjEJGyO5xovTN_ECXq1SALyjx9IYe-zj3SLQcLopl2_XhHHU9FrsnaMi-LSmduqz6tHdjrUXogvLjCdWWk9YNgqGZ5-fIFzP_EGO8Gp4aaufVFJxeidT3QmLYcgjPXt0Z7fHv2MxwAXggnR_Wo4T0edZ4n5dDvNZHz-hTbMPEq9cUJmd4WrsIFNtM0VyVBTS_PnjN5pITmF8ihig3xYKme15n8FAppjwi_nCkCIxOPDJb8FTmygOkMLSimA4Ttjw_sei--N21YAY7zEYcu2OEgYETZqo53lvhnN206MP-5raDsq5V215PU23ToslrxV1LJbQabSmkdJj1YTHdmkw"}

```




gpt4.5 

```js
gpt-4.5-preview
2025-02-27

https://d-liu-m8y372uz-eastus2.cognitiveservices.azure.com/openai/deployments/gpt-4.5-preview/chat/completions?api-version=2025-01-01-preview

BYyBGpw5tfiNvToOPjPbtzIQFPwNXLZ19LeYR6XbV6cPgVdp0GrqJQQJ99BDACHYHv6XJ3w3AAAAACOGwLzO
```

```
https://namcheckopenai.openai.azure.com/openai/deployments/gpt-4o/chat/completions?api-version=2025-01-01-preview


```js
{
name: gpt-4o
ver:2024-08-01-preview
https://namcheckopenai.openai.azure.com/openai/deployments/gpt-4o/chat/completions?api-version=2025-01-01-preview
}
```


```js

2024-08-01-preview

https://namcheckopenai.openai.azure.com/openai/deployments/gpt-4o/chat/completions?api-version=2025-01-01-preview
```

```js
name: gpt-4.5-preview
ver: 2025-02-27
2025-02-27
https://namcheckopenai-uat.openai.azure.com/openai/deployments/gpt-4.5-preview-2/chat/completions?api-version=2025-01-01-preview









az resource update \
    --resource-group nripi-commentcheck-rg \
    --name nricosmosdb1 \
    --resource-type Microsoft.DocumentDB/databaseAccounts \
    --set properties.disableLocalAuth=false




Disable for Web Deploy and Git: https://\<app-name>.scm.azurewebsites.net,

az resource update \
--resource-group nripi-commentcheck-rg \
--name scm \
--namespace Microsoft.Web \
--resource-type basicPublishingCredentialsPolicies \
--parent sites/NamCheckWeb \
--set properties.allow=true



###
