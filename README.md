# Integração Azure DevOps + SonarQube

Este projeto demonstra a configuração de um **pipeline CI/CD no Azure DevOps** que integra com o **SonarQube Server** para análise estática de código, geração de métricas de qualidade e verificação de **Quality Gate**.  

---

## 📌 Fluxo do Pipeline

1. **Preparação SonarQube**  
   A task `SonarQubePrepare@7` inicializa a configuração do scanner, define variáveis de projeto e aponta para o servidor SonarQube.

2. **Build e Testes**  
   A task `Maven@4` compila o projeto, executa os testes e gera relatórios de cobertura (JaCoCo).

3. **Análise SonarQube**  
   A task `SonarQubeAnalyze@7` envia os resultados da análise para o SonarQube Server.

4. **Publicação do Quality Gate**  
   A task `SonarQubePublish@7` consulta o Quality Gate no SonarQube e publica o resultado no sumário do Azure DevOps.  
   > Caso seja necessário interromper o pipeline automaticamente em falhas de Quality Gate, um passo adicional em Bash pode ser usado para validar o status via API do SonarQube.

5. **Publicação de Artefatos**  
   As tasks `CopyFiles@2` e `PublishBuildArtifacts@1` preparam e publicam o artefato `.war` gerado.

---

## ⚙️ Pré-requisitos

Antes de rodar o pipeline, configure os seguintes pontos:

1. **Extensão do SonarQube no Azure DevOps**  
   - Instalar a extensão **SonarQube** na organização do Azure DevOps:  
     [SonarQube Extension - Marketplace](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarqube)

2. **Service Connection**  
   - Criar uma **Service Connection** do tipo **SonarQube** no Azure DevOps.  
   - Essa conexão será usada no `SonarQubePrepare@7` para autenticar no servidor.

3. **Token no SonarQube Server**  
   - Acessar o SonarQube com um usuário administrador.  
   - Ir em **My Account > Security > Generate Token**.  
   - Usar esse token no momento da configuração da Service Connection no Azure DevOps.

4. **Pipeline Variables**  
   Configurar as variáveis no pipeline ou no grupo de variáveis:
   - `SONARQUBE_PROJECT_KEY` → chave do projeto no SonarQube  
   - `SONARQUBE_PROJECT_NAME` → nome do projeto no SonarQube  
   - `SONAR_TOKEN` → token de autenticação (caso use a task Bash opcional)  
   - `SONAR_HOST_URL` → URL do servidor SonarQube (ex: `https://sonarqube.local`)  

5. **Ambiente do Agente**  
   - O agente de build deve ter **Java** e **Maven** instalados.  
   - Certificados internos (se aplicável) devem estar importados para que o agente consiga se conectar ao SonarQube Server com HTTPS.

---

## 🔗 Documentação Oficial

- [SonarQube Extension for Azure DevOps](https://docs.sonarsource.com/sonarqube/latest/devops-platform-integration/azure-devops/)  
- [Azure DevOps Marketplace - SonarQube](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarqube)  
- [Quality Gates - SonarQube](https://docs.sonarqube.org/latest/user-guide/quality-gates/)

---

![Quality Gate Status](https://github.com/geovanegrippa/myshuttle-java-ci-sonarqubeserver/blob/master/Quality%20Gate%20Status.jpg)
![SonarQube Server Analysis Report](https://github.com/geovanegrippa/myshuttle-java-ci-sonarqubeserver/blob/master/SonarQube%20Server%20Analysis%20Report.jpg)

