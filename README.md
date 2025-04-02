# Integração contínua para Salesforce Commerce Cloud
Este documento têm como fim descrever como configurar a integração contínua através de Github Actions para a plataforma Salesforce Commerce Cloud.
# Como configurar

## 1. Criar uma chave de API
Certifique-se de que você tem uma chave de API válida do Commerce Cloud (clientId) configurada. Se você não tiver uma chave de API, pode criar uma usando o Account Manager. A gestão das chaves de API é feita em **Account Manager > API Client** e requer a função de **Account Administrator** ou **API Administrator**.

## 2. Fornecer acesso para a chave em seu ambiente
Para fornecer acesso, precisamos realizar as devidas configurações em nossa instância nas abas "Open Commerce API" e "WebDAV Client Permissions".

### 2.1 Open Commerce API
Em Administration > Site Development > Open Commerce API Settings, modo "Global (organization-Wide)" e "Data API", adicione sua chave com o seguinte escopo.
```json
{
	"_v": "19.5",
	"clients": [
		{
			"client_id": "<seu-client-id>",
			"resources": [
				{
					"resource_id": "/jobs/*/executions",
					"methods": [
						"post"
					],
					"read_attributes": "(**)",
					"write_attributes": "(**)"
				},
				{
					"resource_id": "/jobs/*/executions/*",
					"methods": [
						"get"
					],
					"read_attributes": "(**)",
					"write_attributes": "(**)"
				}
			]
		}
	]
}
```
### 2.2 WebDAV Client Permissions
Em Administration > Organization > WebDAV Client Permissions, adicione sua chave no JSON de configurações com o seguinte escopo:
```json
    {
      "clients":
      [
        {
			"client_id": "<seu-client-id>",
			"permissions": [
				{
					"path": "/impex",
					"operations": [
						"read_write"
					]
				}
			]
		}
      ]
    }
```
## 3. Criar ambiente e variáveis no repositório do Github
A esteira de deploy consome valores dinâmicos de configuração de ambientes vindos do repositório do Github, portanto, é necessário criar o espaço para armazenar os dados de deploy.

### 3.1 Criar ambiente
Para criar um ambiente, siga os passos abaixo:
- Em seu repositório vá para Settings > Environments
- Clique em "New Environment"
- Digite "main" e clique em "Configure Environment" (Aqui definimos como "main" pois é o nome padrão definido na esteira nos arquivos de workflow .yml, você pode definir qualquer outro nome, porém, leve em consideração que deve ser alterado no workflow .yml também)

### 3.2 Criar variáveis de ambiente
Após criar o ambiente, na mesma página do ambiente (Settings > Environments > <seu-ambiente>) você poderá criar secrets e variáveis. Crie a seguinte estrutura:
#### 3.2.1 Secrets
- CLIENT_SECRET: Senha definida no momento da criação de seu client_id no Account Manager
- PASS: Access Key definida no Business Manager, mesmo do dw.json
- PASSPHRASE: Senha para deploy

#### 3.2.2 Variáveis
- CARTRIDGES: Lista contendo os cartuchos para deploy - Formato: String[] - Ex: ["app_main", "app_storefront_base"]
- CLIENT_ID: O ID do client criado no Account Manager
- P12: Nome do arquivo p12 para deploy
- USER: O usuário da Access key definida no Business Manager, mesmo do dw.json
- CODE_VERSION: Nome da Code Version para deploy
- HOST: O Host para deploy

Com as configurações de API feitas e variáveis definidas, você já poderá executar a esteira de deploy.

# Ativação da esteira
Na esteira de deploy, existem dois trabalhos construídos que podem ser ativados em diferentes situações

## 1. Importação de metadados
A Importação de metadados (Objetos do sistema e personalizados) ocorrerá sempre que o sistema detectar alterações em arquivos dentro da pasta `📁 metadata/meta`, disparando assim um upload dos metadados e ativação.

## 2. Upload de código
O upload de código ocorrerá toda vez que o sistema detectar alteração em qualquer arquivo dentro da pasta `📁cartridges`.
