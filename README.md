

```markdown
##  Sobre o Projeto

O EasyPark é uma solução de estacionamento inteligente utilizando sensores IoT para monitoramento de vagas em tempo real.
A aplicação foi desenvolvida utilizando as seguintes tecnologias:
* **Java Spring Boot** (Backend)
* **Oracle Database** (Banco de dados)
* **Azure Web App** (Hospedagem da API)
* **Azure DevOps** (Pipeline CI/CD)
* **GitHub** (Versionamento de código)

---

##  Objetivo

O objetivo do projeto foi implementar uma esteira DevOps completa utilizando conceitos de CI/CD para automatizar:

---

##  Tecnologias Utilizadas

| Tecnologia | Função |
| :--- | :--- |
| **Java Spring Boot** | Backend da aplicação |
| **Oracle Database** | Banco de dados relacional |
| **Azure Web App** | Hospedagem e execução da API |
| **Azure DevOps** | Gerenciamento e execução da pipeline CI/CD |
| **GitHub** | Controle de versão do código-fonte |
| **YAML** | Linguagem de configuração da pipeline |
| **Ubuntu Agent** | Sistema operacional do agente de execução da pipeline |

---

##  Estrutura do Projeto
```text
easypark-devops/
 ├── src/
 ├── pom.xml
 ├── azure-pipelines.yml
 ├── README.md
 └── application.properties

```

---

### 1. Clonar o Projeto

Abra o terminal e execute o comando para clonar o repositório:

```bash
git clone [https://github.com/Vi-debu/easypark-devops.git](https://github.com/Vi-debu/easypark-devops.git)
cd easypark-devops

```

### 2. Configurar o Banco Oracle

Abra o arquivo de propriedades em `src/main/resources/application.properties` e insira as credenciais do seu banco de dados:

```properties
spring.datasource.url=URL_DO_ORACLE
spring.datasource.username=USUARIO
spring.datasource.password=SENHA

```

### 3. Subir o Projeto para o GitHub

Após realizar as configurações locais, envie as alterações para a branch principal:

```bash
git add .
git commit -m "configurando projeto easypark"
git push origin main

```

### 4. Criar o Azure Web App

1. Acesse o Portal da Azure.
2. Vá em **App Services** e clique em **Create Web App**.
3. Utilize as seguintes configurações básicas:

| Campo | Valor |
| --- | --- |
| **Runtime Stack** | Java 21 |
| **Sistema Operacional** | Linux |
| **Região** | brazil south |
| **Nome do App** | api-easypark |

### 5. Criar Service Connection no Azure DevOps

1. No painel do seu projeto no Azure DevOps, acesse **Project Settings**.
2. Clique em **Service Connections** e depois em **New Service Connection**.
3. Selecione a opção **Azure Resource Manager** e conclua a autorização com a sua conta Azure.

### 6. Criar e Executar a Pipeline

1. No menu lateral do Azure DevOps, clique em **Pipelines** ➔ **Create Pipeline**.
2. Selecione o **GitHub** como origem e escolha o repositório `easypark-devops`.
3. Escolha a opção **Existing Azure Pipelines YAML file**.
4. Indique o caminho do arquivo `azure-pipelines.yml` presente na raiz do projeto e salve.

---

## 7. Código da Pipeline Utilizada

Arquivo: `azure-pipelines.yml`

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  webAppName: 'api-easypark'

steps:
- checkout: self

- task: ArchiveFiles@2
  displayName: 'Compactar arquivos do projeto'
  inputs:
    rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
    includeRootFolder: false
    archiveType: 'zip'
    archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip'
    replaceExistingArchive: true

- task: AzureRmWebAppDeployment@4
  displayName: 'Fazer Deploy para o Azure Web App'
  inputs:
    ConnectionType: 'AzureRM'
    azureSubscription: 'SUA-CONEXAO-AZURE'
    appType: 'webAppLinux'
    WebAppName: '$(webAppName)'
    package: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip'

- script: |
    mkdir artifact
    echo "EasyPark deploy realizado com sucesso" > artifact/deploy.txt
    echo "Seu app estará disponível em: https://$(webAppName).azurewebsites.net" >> artifact/deploy.txt
  displayName: 'Criar Informações do Deploy'

- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: 'artifact'
    ArtifactName: 'easypark-artifact'
  displayName: 'Publicar Artifact'

```

---

## 8. Explicação Detalhada da Pipeline

* **Trigger Automático (`trigger`):** Monitora a branch `main`. Qualquer `push` ou `merge` aceito nela inicia a execução da esteira imediatamente.
* **Ambiente da Pipeline (`pool`):** Aloca uma máquina virtual limpa com a imagem estável do Ubuntu administrada pela Microsoft.
* **Checkout do Código (`checkout`):** Realiza o download dos arquivos do repositório para o ambiente temporário de execução.
* **Compactação do Projeto (`ArchiveFiles@2`):** Compacta os arquivos de distribuição gerados no formato `.zip` para otimizar o envio.
* **Deploy Automático (`AzureRmWebAppDeployment@4`):** Conecta de forma segura com o Microsoft Azure e atualiza o serviço do Web App Linux com o novo pacote gerado.
* **Geração do Artifact (`script`):** Cria via linha de comando um arquivo informativo chamado `deploy.txt` contendo os dados de sucesso e a URL pública.
* **Publicação do Artifact (`PublishBuildArtifacts@1`):** Disponibiliza o arquivo gerado na aba de artefatos do Azure DevOps para auditoria e controle de releases.

---

## Fluxo do CI/CD

```text
Push no GitHub
      │
      ▼
Trigger da Pipeline
      │
      ▼
Checkout do Código
      │
      ▼
Compactação do Projeto (.zip)
      │
      ▼
Deploy Automático no Azure
      │
      ▼
Criação do Artifact (deploy.txt)
      │
      ▼
Publicação do Artifact no DevOps
      │
      ▼
Aplicação Online e Atualizada

```

---

## Artifact Gerado

* **Nome do Artifact publicado:** `easypark-artifact`
* **Arquivo contido:** `deploy.txt`

**Conteúdo interno do arquivo:**

```text
EasyPark deploy realizado com sucesso
[(https://api-easypark-agh6f3dugbemcfbh.brazilsouth-01.azurewebsites.net/web/login?logout)]

```

---

## URL da Aplicação e Testes da API

A API pode ser acessada publicamente através da URL base:

> **Link Principal:** (https://api-easypark-agh6f3dugbemcfbh.brazilsouth-01.azurewebsites.net/web/login?logout)

### Endpoints via Postman

* **GET** — Listar todas as vagas:
* (https://api-easypark-agh6f3dugbemcfbh.brazilsouth-01.azurewebsites.net/web/login?logout)
* **Exemplo de Retorno (200 OK):**
```json
{
  "content": [
    {
      "id": 1823,
      "codigo": "B20-3ANDAR",
      "ativa": false,
      "nivelId": 341,
      "tipoVagaId": 122
    }
  ]
}

```




* **POST** — Criar nova vaga:
* (https://api-easypark-agh6f3dugbemcfbh.brazilsouth-01.azurewebsites.net/web/login?logout)
* **Corpo da Requisição (JSON):**
```json
{
  "nivelId": 341,
  "tipoVagaId": 122,
  "codigo": "VAGA-TESTE-01",
  "ativa": true
}

```




* **PUT** — Atualizar dados de uma vaga existente:
* (https://api-easypark-agh6f3dugbemcfbh.brazilsouth-01.azurewebsites.net/web/login?logout)
* **Corpo da Requisição (JSON):**
```json
{
  "nivelId": 341,
  "tipoVagaId": 122,
  "codigo": "VAGA-TESTE-01-EDITADA",
  "ativa": false
}

```




* **DELETE** :
* Selecione o método `DELETE`, configure a rota correspondente ao ID desejado e envie a requisição.



### 11. Verificar Dados no Oracle


```sql
SELECT * FROM tb_vaga;
SELECT * FROM tb_usuario;

SELECT 
u.nome as usuario,
v.codigo as codigo,
tv.nome as tipo_vaga
FROM usuario u
INNER JOIN reserva r ON r.usuario_Id = u.Id
INNER JOIN vaga v ON v.id = r.vaga_Id
INNER JOIN tipo_vaga tv on tv.Id  = v.tipo_vaga_id

```

---

## Integrantes

* **Gabriel Cruz** — RM 559613
* **Kauã Ferreira** — RM 560992
* **Vinicius Bitú** — RM 560227

---

## Links do Projeto

* **Repositório GitHub:** [EasyPark DevOps GitHub]([https://github.com/Vi-debu/easypark-devops.git])
* **Vídeo Demonstrativo:** ([https://youtu.be/Cez0EF930uU])

```

```
