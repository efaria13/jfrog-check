# Guia de Demonstração JFrog Platform - Baseado no Caderno de Testes

## Introdução
Este guia foi estruturado para demonstrar as funcionalidades da JFrog Platform seguindo os cenários específicos do **Caderno de Testes funcionais**. Cada seção aborda diretamente os requisitos testados no documento.

## Pré-requisitos
- Conta JFrog (teste gratuito disponível)
- JFrog CLI instalado
- Acesso administrativo à plataforma
- Artefatos de teste para diferentes tecnologias

---

## **CATEGORIA 1: REPOSITÓRIO - Testes 1.1 a 1.7**

### **Teste 1.1 - Suporte a Múltiplas Tecnologias de Artefatos**
**Requisito**: Hospedar e gerenciar artefatos usando múltiplas tecnologias

**Demonstração Passo a Passo:**

```bash
# 1. Verificar tecnologias suportadas via CLI
jfrog rt repo-template

# 2. Criar repositórios para cada tecnologia testada
# Maven
jfrog rt repo-create maven-local --type=local --package-type=maven

# npm  
jfrog rt repo-create npm-local --type=local --package-type=npm

# Docker
jfrog rt repo-create docker-local --type=local --package-type=docker

# Python (PyPI)
jfrog rt repo-create pypi-local --type=local --package-type=pypi

# NuGet
jfrog rt repo-create nuget-local --type=local --package-type=nuget

# APK (Alpine)
jfrog rt repo-create apk-local --type=local --package-type=alpine

# YUM
jfrog rt repo-create yum-local --type=local --package-type=yum

# APT (Debian/Ubuntu)
jfrog rt repo-create apt-local --type=local --package-type=debian

# Go
jfrog rt repo-create go-local --type=local --package-type=go

# Helm
jfrog rt repo-create helm-local --type=local --package-type=helm

# Conan (C++)
jfrog rt repo-create conan-local --type=local --package-type=conan

# Conda
jfrog rt repo-create conda-local --type=local --package-type=conda

# Bower
jfrog rt repo-create bower-local --type=local --package-type=bower

# RubyGems
jfrog rt repo-create gems-local --type=local --package-type=gems

# R (CRAN)
jfrog rt repo-create cran-local --type=local --package-type=cran

# GitLFS
jfrog rt repo-create gitlfs-local --type=local --package-type=gitlfs

# OCI
jfrog rt repo-create oci-local --type=local --package-type=oci

# Generic/RAW
jfrog rt repo-create raw-local --type=local --package-type=generic
```

**Validação**: Todos os repositórios devem ser criados com sucesso e visíveis na interface web.

**Evidência**: Upload de pelo menos um artefato de cada tipo para demonstrar funcionalidade completa.

---

### **Teste 1.2 - Proxying e Caching de Repositórios Externos**
**Requisito**: Fazer proxying e caching das tecnologias mencionadas

**Demonstração:**

```bash
# Criar repositórios remotos (proxy) para cada tecnologia
# Maven Central
jfrog rt repo-create maven-remote --type=remote --package-type=maven --url=https://repo1.maven.org/maven2/

# npm Registry
jfrog rt repo-create npm-remote --type=remote --package-type=npm --url=https://registry.npmjs.org/

# Docker Hub
jfrog rt repo-create docker-remote --type=remote --package-type=docker --url=https://registry-1.docker.io/

# PyPI
jfrog rt repo-create pypi-remote --type=remote --package-type=pypi --url=https://pypi.org/

# NuGet
jfrog rt repo-create nuget-remote --type=remote --package-type=nuget --url=https://api.nuget.org/v3/index.json

# Configurar cache settings
jfrog rt repo-update maven-remote --cache-enabled=true --cache-ttl=86400

# Testar download via proxy (exemplo Maven)
jfrog rt download "maven-remote:junit/junit/4.12/junit-4.12.jar" ./cache-test/
```

**Validação**: Artefatos baixados devem estar disponíveis no cache local e acessíveis via proxy.

**Tempo Estimado**: 45 minutos

---

### **Teste 1.3 - Políticas de Manutenção, Retenção e Limpeza**
**Requisito**: Configurar políticas por idade, espaço em disco ou outros critérios

**Demonstração:**

```bash
# 1. Configurar política de limpeza por idade (via interface web)
# Administration > Repositories > Cleanup Policies
```

**Configuração via REST API:**
```bash
# Criar política de limpeza
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/retention" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "cleanup-old-artifacts",
    "repository": "maven-local",
    "criteria": {
      "lastDownloaded": "30d",
      "createdBefore": "90d"
    },
    "action": "delete"
  }'

# Executar limpeza
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/retention/cleanup-old-artifacts/execute"
```

**Validação**: Execução bem-sucedida da política sem falha, com relatório de artefatos removidos.

**Tempo Estimado**: 60 minutos

---

### **Teste 1.4 - Movimentação de Artefatos Entre Repositórios**
**Requisito**: Mover artefatos de um repositório para outro

**Demonstração:**

```bash
# Upload inicial em repositório de desenvolvimento
jfrog rt upload "target/app-1.0.0.jar" dev-maven-local/com/example/app/1.0.0/

# Mover artefato para repositório de produção
jfrog rt move "dev-maven-local/com/example/app/1.0.0/app-1.0.0.jar" "prod-maven-local/com/example/app/1.0.0/"

# Verificar movimentação
jfrog rt search "prod-maven-local/com/example/app/1.0.0/*"

# Confirmar que não existe mais no repositório origem
jfrog rt search "dev-maven-local/com/example/app/1.0.0/*"
```

**Validação**: Artefato deve existir apenas no repositório de destino após a movimentação.

**Tempo Estimado**: 15 minutos

---

### **Teste 1.5 - Crescimento Ilimitado dos Repositórios**
**Requisito**: Demonstrar que não há restrições de crescimento

**Demonstração:**

```bash
# Script para testar limites de armazenamento
#!/bin/bash

# Verificar quotas configuradas
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/system/usage"

# Upload de múltiplos artefatos para testar limites
for i in {1..100}; do
  echo "Teste de conteúdo $i" > test-file-$i.txt
  jfrog rt upload test-file-$i.txt generic-local/stress-test/
done

# Verificar estatísticas de uso
jfrog rt search "generic-local/stress-test/*" --count
```

**Demonstração via Interface:**
- Mostrar configurações de quota (se aplicável)
- Demonstrar ausência de limites de repositórios
- Exibir métricas de crescimento

**Validação**: Upload de grande quantidade de dados sem restrições.

**Tempo Estimado**: 30 minutos

---

### **Teste 1.6 - Restrições de Acesso por Mês**
**Requisito**: Verificar se há limites de acesso aos repositórios

**Demonstração:**

```bash
# Ativar logs de acesso
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/system/logs/access" \
  -H "Content-Type: application/json" \
  -d '{"enable": true}'

# Script para gerar múltiplos acessos
#!/bin/bash
for i in {1..1000}; do
  jfrog rt download "maven-local/com/example/app/1.0.0/app-1.0.0.jar" "./test-downloads/download-$i.jar"
done

# Verificar logs de acesso
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/system/logs/access"

# Verificar se houve bloqueios
grep -i "limit\|restrict\|block" access.log
```

**Validação**: Logs devem mostrar todos os acessos sem restrições ou bloqueios.

**Tempo Estimado**: 45 minutos

---

### **Teste 1.7 - Múltiplas Instâncias de Repositórios**
**Requisito**: Suportar múltiplas instâncias da mesma tecnologia

**Demonstração:**

```bash
# Criar múltiplos repositórios Maven
jfrog rt repo-create maven-dev --type=local --package-type=maven
jfrog rt repo-create maven-staging --type=local --package-type=maven  
jfrog rt repo-create maven-prod --type=local --package-type=maven

# Criar múltiplos repositórios Docker
jfrog rt repo-create docker-dev --type=local --package-type=docker
jfrog rt repo-create docker-staging --type=local --package-type=docker
jfrog rt repo-create docker-prod --type=local --package-type=docker

# Testar upload independente em cada instância
jfrog rt upload "app-1.0.0.jar" maven-dev/com/example/app/1.0.0/
jfrog rt upload "app-1.0.0.jar" maven-staging/com/example/app/1.0.0/
jfrog rt upload "app-1.0.0.jar" maven-prod/com/example/app/1.0.0/

# Verificar independência das instâncias
jfrog rt search "maven-*/com/example/app/1.0.0/*"
```

**Validação**: Cada repositório deve funcionar independentemente com a mesma tecnologia.

**Tempo Estimado**: 20 minutos

---

## **CATEGORIA 2: API - Teste 2.1**

### **Teste 2.1 - API REST**
**Requisito**: Produto deve possuir API REST funcional

**Demonstração Completa:**

```bash
# 1. Testar autenticação
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/system/ping" \
  -u "username:password"

# 2. Listar repositórios via API
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/repositories" \
  -u "username:password" \
  -H "Content-Type: application/json"

# 3. Upload via API
curl -X PUT "https://yourinstance.jfrog.io/artifactory/generic-local/test-api/file.txt" \
  -u "username:password" \
  -T "./test-file.txt"

# 4. Download via API
curl -X GET "https://yourinstance.jfrog.io/artifactory/generic-local/test-api/file.txt" \
  -u "username:password" \
  -o "./downloaded-file.txt"

# 5. Buscar artefatos via API
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/search/aql" \
  -u "username:password" \
  -H "Content-Type: text/plain" \
  -d 'items.find({"repo":"generic-local"})'

# 6. Criar repositório via API
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/repositories/api-test-repo" \
  -u "username:password" \
  -H "Content-Type: application/json" \
  -d '{
    "key": "api-test-repo",
    "rclass": "local",
    "packageType": "generic"
  }'

# 7. Configurar propriedades via API
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/storage/generic-local/test-api/file.txt?properties=environment=test;version=1.0" \
  -u "username:password"

# 8. Obter informações do sistema via API
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/system/version" \
  -u "username:password"
```

**Validação**: 
- API deve retornar JSON/XML válido
- Todas as operações devem executar sem erros
- Ações devem ser refletidas na interface web
- Não deve ser necessário acessar interface para executar operações

**Tempo Estimado**: 60 minutos

---

### **Teste 2.2 - Documentação da API**
**Requisito**: API deve ter documentação disponível e acessível

**Demonstração:**

```bash
# Acessar documentação da API via endpoint
curl -X GET "https://yourinstance.jfrog.io/artifactory/api" \
  -u "username:password"

# Verificar Swagger/OpenAPI documentation
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/swagger.json" \
  -u "username:password"

# Testar endpoint de documentação interativa
curl -X GET "https://yourinstance.jfrog.io/ui/api/" \
  -u "username:password"
```

**Interface Web:**
- Navegar para `Administration > User Management > Settings > API Key`
- Acessar `https://yourinstance.jfrog.io/ui/api/` para documentação interativa
- Verificar disponibilidade de exemplos de código

**Validação**: Documentação deve estar disponível e acessível para todos os usuários potenciais.

---

### **Teste 2.3 - APIs para Automação de Políticas**
**Requisito**: APIs para construção de automações das políticas do requisito F-1.3

**Demonstração:**

```bash
# Criar política de limpeza via API
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/retention" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "name": "automated-cleanup-policy",
    "repository": "maven-local", 
    "criteria": {
      "lastDownloaded": "30d",
      "createdBefore": "90d"
    },
    "action": "delete",
    "dryRun": false
  }'

# Executar política via API
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/retention/automated-cleanup-policy/execute" \
  -u "username:password"

# Verificar status da execução
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/retention/automated-cleanup-policy/status" \
  -u "username:password"

# Automatizar com script
#!/bin/bash
# automation-script.sh
POLICY_NAME="automated-cleanup-policy"
RESPONSE=$(curl -s -X POST "https://yourinstance.jfrog.io/artifactory/api/retention/$POLICY_NAME/execute" -u "username:password")
echo "Política executada: $RESPONSE"
```

**Validação**: Executar chamadas de API e obter respostas confirmáveis para automação de políticas.

---

### **Teste 2.4 - API para Resultados de Vulnerabilidades (F-7.1)**
**Requisito**: Apresentar resultados de funcionalidade F-7.1 através de APIs

**Demonstração:**

```bash
# Obter vulnerabilidades de artefato específico via API
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/artifacts/vulnerabilities" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "artifacts": [{
      "repository": "docker-local",
      "path": "my-app:latest"
    }]
  }'

# Scan de artefato com vulnerabilidade conhecida
curl -X POST "https://yourinstance.jfrog.io/xray/api/v1/scanArtifact" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "repository": "maven-local",
    "path": "com/example/vulnerable-lib/1.0.0/vulnerable-lib-1.0.0.jar"
  }'

# Obter relatório de vulnerabilidades
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/summary/build" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "build_name": "my-build",
    "build_number": "1"
  }'
```

**Validação**: Apresentação de vulnerabilidade de artefato via API.

---

### **Teste 2.5 - API para Marcar Artefatos com Vulnerabilidades**
**Requisito**: API para escrever que artefato possui vulnerabilidades

**Demonstração:**

```bash
# Adicionar propriedade de vulnerabilidade via API
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/app/1.0.0/app-1.0.0.jar?properties=vulnerability.status=BLOCKED;vulnerability.reason=CVE-2023-1234" \
  -u "username:password"

# Criar watch personalizada para marcar artefatos
curl -X POST "https://yourinstance.jfrog.io/xray/api/v2/watches" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "general_data": {
      "name": "vulnerability-marker",
      "description": "Mark artifacts with vulnerabilities"
    },
    "project_resources": {
      "repositories": [{"type": "local", "name": "maven-local"}]
    },
    "assigned_policies": [{
      "name": "vulnerability-policy",
      "type": "security"
    }]
  }'

# Verificar propriedades do artefato
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/app/1.0.0/app-1.0.0.jar?properties" \
  -u "username:password"
```

**Validação**: Executar chamadas de API e obter respostas confirmáveis para marcação de vulnerabilidades.

---

### **Teste 2.6 - API para Publicação de Artefatos**
**Requisito**: Capacidade de publicar artefatos via API

**Demonstração:**

```bash
# Upload de artefato via API
curl -X PUT "https://yourinstance.jfrog.io/artifactory/maven-local/com/example/app/1.0.0/app-1.0.0.jar" \
  -u "username:password" \
  -T "./app-1.0.0.jar" \
  -H "X-Checksum-Sha1: $(sha1sum app-1.0.0.jar | cut -d' ' -f1)" \
  -H "X-Checksum-Md5: $(md5sum app-1.0.0.jar | cut -d' ' -f1)"

# Upload com propriedades customizadas
curl -X PUT "https://yourinstance.jfrog.io/artifactory/maven-local/com/example/app/1.0.0/app-1.0.0.jar?properties=build.name=api-build;build.number=1" \
  -u "username:password" \
  -T "./app-1.0.0.jar"

# Upload múltiplo via spec file
jfrog rt upload --spec=upload-spec.json

# upload-spec.json
{
  "files": [{
    "pattern": "target/*.jar",
    "target": "maven-local/com/example/app/1.0.0/",
    "props": "uploaded.via=api;timestamp=$(date +%s)"
  }]
}

# Verificar upload
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/app/1.0.0/app-1.0.0.jar" \
  -u "username:password"
```

**Validação**: Executar chamadas de API e obter respostas confirmáveis para publicação de artefatos.

---

### **Teste 2.7 - Integração GitHub Actions com Restrições**
**Requisito**: Fornecer informações sobre restrições de artefatos ao GitHub Actions

**Demonstração GitHub Actions Workflow:**

```yaml
# .github/workflows/jfrog-integration.yml
name: JFrog Security Integration

on: [push, pull_request]

jobs:
  security-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup JFrog CLI
        uses: jfrog/setup-jfrog-cli@v3
        env:
          JF_URL: ${{ secrets.JF_URL }}
          JF_ACCESS_TOKEN: ${{ secrets.JF_ACCESS_TOKEN }}
      
      - name: Build Application
        run: |
          mvn clean package
      
      - name: Upload Artifacts
        run: |
          jfrog rt upload target/*.jar maven-local/com/example/app/${{ github.run_number }}/
      
      - name: Security Scan
        run: |
          jfrog xr scan --repo=maven-local --fail=true
          # Pipeline will fail if critical vulnerabilities found
      
      - name: Check Artifact Restrictions
        run: |
          # Verificar se artefato está bloqueado
          BLOCKED=$(jfrog rt search "maven-local/com/example/app/${{ github.run_number }}/*" --props="restriction.status=BLOCKED")
          if [ ! -z "$BLOCKED" ]; then
            echo "Artifact is restricted - stopping pipeline"
            exit 1
          fi
```

**Script de Verificação de Restrições:**
```bash
#!/bin/bash
# check-restrictions.sh

ARTIFACT_PATH="maven-local/com/example/app/1.0.0/app-1.0.0.jar"

# Verificar propriedades de restrição
PROPERTIES=$(curl -s -X GET "https://yourinstance.jfrog.io/artifactory/api/storage/$ARTIFACT_PATH?properties" -u "username:password")

# Verificar se há restrições
if echo "$PROPERTIES" | grep -q "restriction.status.*BLOCKED"; then
    echo "ERROR: Artifact is blocked due to security violations"
    exit 1
fi

echo "Artifact is approved for use"
```

**Validação**: Confirmação de que pipeline no GitHub Actions foi interrompido devido a restrição configurada.

---

### **Teste 2.8 - GitHub Actions Publicação de Artefatos**
**Requisito**: GitHub Actions deve conseguir publicar artefatos na solução

**Demonstração GitHub Actions:**

```yaml
# .github/workflows/publish-artifacts.yml
name: Publish Artifacts to JFrog

on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup JFrog CLI
        uses: jfrog/setup-jfrog-cli@v3
        env:
          JF_URL: ${{ secrets.JF_URL }}
          JF_ACCESS_TOKEN: ${{ secrets.JF_ACCESS_TOKEN }}
      
      - name: Build Artifacts
        run: |
          mvn clean package
          docker build -t myapp:${{ github.ref_name }} .
      
      - name: Publish Maven Artifacts
        run: |
          jfrog rt mvn-config --repo-resolve-releases=maven-virtual --repo-resolve-snapshots=maven-virtual --repo-deploy-releases=maven-releases --repo-deploy-snapshots=maven-snapshots
          jfrog rt mvn clean deploy --build-name=github-${{ github.repository }} --build-number=${{ github.run_number }}
      
      - name: Publish Docker Image
        run: |
          docker tag myapp:${{ github.ref_name }} yourinstance.jfrog.io/docker-local/myapp:${{ github.ref_name }}
          docker push yourinstance.jfrog.io/docker-local/myapp:${{ github.ref_name }}
      
      - name: Publish Build Info
        run: |
          jfrog rt build-publish github-${{ github.repository }} ${{ github.run_number }}
          
      - name: Verify Publication
        run: |
          jfrog rt search "maven-releases/com/example/app/${{ github.ref_name }}/*"
          jfrog rt search "docker-local/myapp:${{ github.ref_name }}"
```

**Validação**: Verificação de que artefato está disponível na ferramenta e foi gerado pelo pipeline GitHub Actions.

---

## **CATEGORIA 3: IDENTIDADE E INTEGRIDADE DO ARTEFATO - Testes 3.1 a 3.5**

### **Teste 3.1 - Verificação de Integridade com SHA-2/SHA-3**
**Requisito**: Verificar integridade através de checksums SHA-2 ou SHA-3

**Demonstração:**

```bash
# Upload com checksum SHA-256 (SHA-2 family)
CHECKSUM_SHA256=$(sha256sum app-1.0.0.jar | cut -d' ' -f1)
curl -X PUT "https://yourinstance.jfrog.io/artifactory/maven-local/com/example/app/1.0.0/app-1.0.0.jar" \
  -u "username:password" \
  -T "./app-1.0.0.jar" \
  -H "X-Checksum-Sha256: $CHECKSUM_SHA256"

# Verificar checksum armazenado
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/app/1.0.0/app-1.0.0.jar" \
  -u "username:password" | jq '.checksums'

# Download e verificação de integridade
jfrog rt download "maven-local/com/example/app/1.0.0/app-1.0.0.jar" ./downloaded/ --validate-checksums

# Verificação manual
DOWNLOADED_CHECKSUM=$(sha256sum ./downloaded/app-1.0.0.jar | cut -d' ' -f1)
STORED_CHECKSUM=$(curl -s -X GET "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/app/1.0.0/app-1.0.0.jar" -u "username:password" | jq -r '.checksums.sha256')

if [ "$DOWNLOADED_CHECKSUM" = "$STORED_CHECKSUM" ]; then
    echo "Integridade verificada com sucesso - SHA-256: $DOWNLOADED_CHECKSUM"
else
    echo "ERRO: Integridade comprometida!"
fi
```

**Validação**: Confirmação de que artefato foi armazenado integralmente conforme SHA.

---

### **Teste 3.2 - Identificador Único de Artefatos**
**Requisito**: Todo artefato deve ter identificador externo único no repositório

**Demonstração:**

```bash
# Upload de artefato com identificador único
jfrog rt upload "app-1.0.0.jar" "maven-local/com/example/app/1.0.0/" --build-name=unique-test --build-number=1

# Verificar identificador único (coordenadas Maven)
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/app/1.0.0/app-1.0.0.jar" \
  -u "username:password" | jq '.path'

# Tentar upload com mesmo identificador
jfrog rt upload "different-app.jar" "maven-local/com/example/app/1.0.0/app-1.0.0.jar"
# Deve falhar ou sobrescrever dependendo da configuração

# Verificar unicidade via AQL
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/search/aql" \
  -H "Content-Type: text/plain" \
  -u "username:password" \
  -d 'items.find({"repo":"maven-local","name":"app-1.0.0.jar"}).include("repo","path","name")'
```

**Validação**: Conferência de artefato único com seu respectivo identificador.

---

### **Teste 3.3 - Imutabilidade de Artefatos RELEASE**
**Requisito**: Artefato RELEASE não pode ser resubmetido com conteúdo diferente

**Demonstração:**

```bash
# Configurar repositório para bloquear redeploy de releases
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/repositories/maven-releases" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "key": "maven-releases",
    "rclass": "local",
    "packageType": "maven",
    "handleReleases": true,
    "handleSnapshots": false,
    "suppressPomConsistencyChecks": false,
    "blackedOut": false,
    "rejectInvalidJars": true
  }'

# Upload inicial de RELEASE
mvn deploy -DrepositoryId=jfrog-releases -Durl=https://yourinstance.jfrog.io/artifactory/maven-releases -Dversion=1.0.0

# Tentar reupload da mesma versão RELEASE com conteúdo diferente
echo "different content" > modified-app-1.0.0.jar
curl -X PUT "https://yourinstance.jfrog.io/artifactory/maven-releases/com/example/app/1.0.0/app-1.0.0.jar" \
  -u "username:password" \
  -T "./modified-app-1.0.0.jar"
# Deve retornar erro 409 Conflict

# Verificar política de redeploy
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/repositories/maven-releases" \
  -u "username:password" | jq '.rejectInvalidJars'
```

**Validação**: Confirmação de que tentativa de upload de artefato RELEASE na mesma versão é impedida, causando erro no processo.

---

### **Teste 3.4 - Flexibilidade de Artefatos SNAPSHOT**
**Requisito**: Artefato sem marcação RELEASE pode ser resubmetido

**Demonstração:**

```bash
# Configurar repositório para permitir redeploy de snapshots
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/repositories/maven-snapshots" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "key": "maven-snapshots",
    "rclass": "local",
    "packageType": "maven",
    "handleReleases": false,
    "handleSnapshots": true,
    "suppressPomConsistencyChecks": true
  }'

# Upload inicial de SNAPSHOT
mvn deploy -DrepositoryId=jfrog-snapshots -Durl=https://yourinstance.jfrog.io/artifactory/maven-snapshots -Dversion=1.0.0-SNAPSHOT

# Reupload da mesma versão SNAPSHOT com conteúdo diferente
echo "updated content $(date)" > updated-app-1.0.0-SNAPSHOT.jar
curl -X PUT "https://yourinstance.jfrog.io/artifactory/maven-snapshots/com/example/app/1.0.0-SNAPSHOT/app-1.0.0-SNAPSHOT.jar" \
  -u "username:password" \
  -T "./updated-app-1.0.0-SNAPSHOT.jar"
# Deve permitir o upload

# Verificar histórico de versões
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/storage/maven-snapshots/com/example/app/1.0.0-SNAPSHOT" \
  -u "username:password" | jq '.children'
```

**Validação**: Confirmação de que tentativa de upload de artefato sem marcação RELEASE na mesma versão é realizada com sucesso.

---

### **Teste 3.5 - Marcação de Status RELEASE**
**Requisito**: Artefato pode ou não ser marcado com status RELEASE

**Demonstração:**

```bash
# Marcar artefato como RELEASE via propriedades
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/app/1.0.0/app-1.0.0.jar?properties=release.status=RELEASE;release.date=$(date -I)" \
  -u "username:password"

# Marcar outro artefato como desenvolvimento
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/app/1.1.0-SNAPSHOT/app-1.1.0-SNAPSHOT.jar?properties=release.status=DEVELOPMENT;stage=testing" \
  -u "username:password"

# Listar artefatos por status de release
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/search/prop" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "repos": ["maven-local"],
    "properties": {
      "release.status": "RELEASE"
    }
  }'

# Via JFrog CLI com diferentes marcações
jfrog rt upload "*.jar" "maven-local/com/example/releases/" --props="release.status=RELEASE;stage=production"
jfrog rt upload "*.jar" "maven-local/com/example/snapshots/" --props="release.status=DEVELOPMENT;stage=testing"

# Verificar tipos de marcação disponíveis
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/app/1.0.0/app-1.0.0.jar?properties" \
  -u "username:password"
```

**Validação**: Confirmação dos tipos de marcação disponíveis.

---

## **CATEGORIA 4: METADADOS - Testes 4.1 a 4.2**

### **Teste 4.1 - Políticas de Controle de Acesso por Metadados**
**Requisito**: Criar políticas baseadas em metadados para controlar acesso de leitura

**Demonstração:**

```bash
# Configurar usuários e grupos
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/users/developer" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "developer",
    "email": "dev@company.com",
    "password": "dev123",
    "admin": false,
    "groups": ["developers"]
  }'

# Criar política de acesso baseada em metadados
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/permissions/metadata-access-policy" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "metadata-access-policy",
    "repositories": ["maven-local"],
    "principals": {
      "users": {
        "developer": ["r"]
      }
    },
    "includesPattern": "**",
    "excludesPattern": "",
    "actions": {
      "users": {
        "developer": ["read"]
      }
    }
  }'

# Adicionar metadados a artefatos
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/secure-app/1.0.0/secure-app-1.0.0.jar?properties=security.level=HIGH;access.team=backend" \
  -u "admin:password"

curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/public-app/1.0.0/public-app-1.0.0.jar?properties=security.level=LOW;access.team=all" \
  -u "admin:password"

# Testar acesso com usuário restrito
curl -X GET "https://yourinstance.jfrog.io/artifactory/maven-local/com/example/secure-app/1.0.0/secure-app-1.0.0.jar" \
  -u "developer:dev123"
# Deve ser bloqueado se política configurada

# Configurar filtro por metadados no repositório virtual
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/repositories/maven-filtered-virtual" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "key": "maven-filtered-virtual",
    "rclass": "virtual",
    "packageType": "maven",
    "repositories": ["maven-local"],
    "retrievalCachePeriodSecs": 43200,
    "pomRepositoryReferencesCleanupPolicy": "discard_active_reference"
  }'
```

**Validação**: Confirmação de que política foi criada restringindo acesso a artefatos baseado em metadados.

---

### **Teste 4.2 - Políticas de Limpeza por Metadados**
**Requisito**: Criar políticas de limpeza usando metadados como critério

**Demonstração:**

```bash
# Adicionar metadados de tempo de vida aos artefatos
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/temp-app/1.0.0/temp-app-1.0.0.jar?properties=ttl.days=30;cleanup.category=temporary;last.accessed=$(date -d '40 days ago' -I)" \
  -u "username:password"

curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/permanent-app/1.0.0/permanent-app-1.0.0.jar?properties=ttl.days=永久;cleanup.category=permanent" \
  -u "username:password"

# Criar política de limpeza baseada em metadados
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/retention" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "name": "metadata-cleanup-policy",
    "repository": "maven-local",
    "criteria": {
      "properties": {
        "cleanup.category": "temporary"
      },
      "lastDownloaded": "30d"
    },
    "action": "delete",
    "dryRun": false
  }'

# Executar política de limpeza
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/retention/metadata-cleanup-policy/execute" \
  -u "username:password"

# Verificar resultados da limpeza
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/retention/metadata-cleanup-policy/results" \
  -u "username:password"

# Confirmar que artefatos permanentes não foram removidos
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/permanent-app/1.0.0/permanent-app-1.0.0.jar" \
  -u "username:password"

# Verificar que artefatos temporários antigos foram removidos
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/temp-app/1.0.0/temp-app-1.0.0.jar" \
  -u "username:password"
# Deve retornar 404 se removido
```

**Validação**: Confirmação de saneamento baseado em metadados configurados na política.

---

## **CATEGORIA 5: CONTROLE DE ACESSO - Testes 5.1 a 5.8**

### **Teste 5.1 - Autenticação Obrigatória**
**Requisito**: Produto deve exigir autenticação para acesso aos repositórios

**Demonstração:**

```bash
# Tentar acesso sem autenticação
curl -X GET "https://yourinstance.jfrog.io/artifactory/maven-local/" 
# Deve retornar 401 Unauthorized

# Configurar autenticação LDAP
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/system/configuration" \
  -H "Content-Type: application/xml" \
  -u "admin:password" \
  -d '<config>
    <security>
      <ldapSettings>
        <ldapSetting>
          <key>ldap1</key>
          <enabled>true</enabled>
          <ldapUrl>ldap://your-ldap-server:389</ldapUrl>
          <userDnPattern>uid={0},ou=users,dc=company,dc=com</userDnPattern>
        </ldapSetting>
      </ldapSettings>
    </security>
  </config>'

# Configurar autenticação SAML (exemplo)
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/saml/config" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "enableIntegration": true,
    "loginUrl": "https://your-saml-provider/login",
    "logoutUrl": "https://your-saml-provider/logout",
    "serviceProviderName": "Artifactory",
    "certificate": "-----BEGIN CERTIFICATE-----..."
  }'

# Testar acesso com autenticação válida
curl -X GET "https://yourinstance.jfrog.io/artifactory/maven-local/" \
  -u "validuser:validpassword"
# Deve permitir acesso

# Tentar download de artefato privado sem autenticação
curl -X GET "https://yourinstance.jfrog.io/artifactory/maven-private/com/example/app/1.0.0/app-1.0.0.jar"
# Deve ser negado
```

**Validação**: Tentativa de download de artefatos privados sendo negada se usuário não autenticado e não autorizado.

---

### **Teste 5.2 - Controle de Acesso por Papéis**
**Requisito**: Controlar acesso através de papéis de usuário conforme especificações

**Demonstração:**

```bash
# Criar papéis
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/groups/developers" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "developers",
    "description": "Development team",
    "autoJoin": false,
    "adminPrivileges": false
  }'

curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/groups/testers" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "testers", 
    "description": "QA team",
    "autoJoin": false,
    "adminPrivileges": false
  }'

# Criar usuários e atribuir a múltiplos papéis
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/users/john" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "john",
    "email": "john@company.com",
    "password": "john123",
    "admin": false,
    "groups": ["developers", "testers"]
  }'

# Configurar permissões por papel
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/permissions/dev-permissions" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "dev-permissions",
    "repositories": ["maven-dev", "maven-snapshots"],
    "principals": {
      "groups": {
        "developers": ["r", "w", "d"]
      }
    }
  }'

curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/permissions/test-permissions" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "test-permissions", 
    "repositories": ["maven-test"],
    "principals": {
      "groups": {
        "testers": ["r"]
      }
    }
  }'

# Testar acesso com usuário multi-papel
curl -X GET "https://yourinstance.jfrog.io/artifactory/maven-dev/" \
  -u "john:john123"
# Deve permitir (papel developers)

curl -X GET "https://yourinstance.jfrog.io/artifactory/maven-test/" \
  -u "john:john123"
# Deve permitir (papel testers)
```

**Validação**: Tentativas de acesso a repositórios sendo permitidas ou negadas conforme papéis definidos.

---

### **Teste 5.3 - Papel Administrativo de Usuários**
**Requisito**: Pelo menos 1 papel com direitos CRUD para usuários, grupos e papéis

**Demonstração:**

```bash
# Criar papel de administrador de usuários
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/groups/user-admins" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "user-admins",
    "description": "User management administrators", 
    "autoJoin": false,
    "adminPrivileges": false
  }'

# Criar usuário administrador
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/users/useradmin" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "useradmin",
    "email": "useradmin@company.com",
    "password": "useradmin123",
    "admin": false,
    "groups": ["user-admins"]
  }'

# Conceder permissões administrativas ao papel
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/permissions/user-management" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "user-management",
    "repositories": [],
    "principals": {
      "groups": {
        "user-admins": ["m"]
      }
    },
    "actions": {
      "groups": {
        "user-admins": ["manage"]
      }
    }
  }'

# Testar capacidades CRUD com usuário admin
# Criar usuário
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/users/newuser" \
  -H "Content-Type: application/json" \
  -u "useradmin:useradmin123" \
  -d '{
    "name": "newuser",
    "email": "newuser@company.com",
    "password": "new123",
    "admin": false
  }'

# Ler usuário
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/security/users/newuser" \
  -u "useradmin:useradmin123"

# Alterar usuário  
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/security/users/newuser" \
  -H "Content-Type: application/json" \
  -u "useradmin:useradmin123" \
  -d '{
    "email": "updated@company.com"
  }'

# Excluir usuário
curl -X DELETE "https://yourinstance.jfrog.io/artifactory/api/security/users/newuser" \
  -u "useradmin:useradmin123"
```

**Validação**: Validação de criação de perfil para criar, ler, alterar ou excluir papéis.

---

### **Teste 5.4 - Papel para Políticas de Controle de Acesso**
**Requisito**: Papel com direitos CRUD para políticas de controle de acesso

**Demonstração:**

```bash
# Criar papel para administrador de políticas
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/groups/policy-admins" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "policy-admins",
    "description": "Access policy administrators"
  }'

# Usuário administrador de políticas
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/users/policyadmin" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "policyadmin",
    "email": "policyadmin@company.com", 
    "password": "policy123",
    "groups": ["policy-admins"]
  }'

# Testar CRUD de políticas de acesso
# Criar política
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/permissions/test-policy" \
  -H "Content-Type: application/json" \
  -u "policyadmin:policy123" \
  -d '{
    "name": "test-policy",
    "repositories": ["maven-test"],
    "principals": {
      "users": {
        "testuser": ["r"]
      }
    }
  }'

# Ler política
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/security/permissions/test-policy" \
  -u "policyadmin:policy123"

# Alterar política
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/permissions/test-policy" \
  -H "Content-Type: application/json" \
  -u "policyadmin:policy123" \
  -d '{
    "name": "test-policy",
    "repositories": ["maven-test", "maven-dev"],
    "principals": {
      "users": {
        "testuser": ["r", "w"]
      }
    }
  }'

# Excluir política
curl -X DELETE "https://yourinstance.jfrog.io/artifactory/api/security/permissions/test-policy" \
  -u "policyadmin:policy123"
```

**Validação**: Validação de criação de perfil para criar, ler, alterar ou excluir políticas de controle de acesso.

---

### **Teste 5.5 - Papel para Relatórios**
**Requisito**: Papel com permissões CRUD para relatórios sobre repositórios e artefatos

**Demonstração:**

```bash
# Criar papel para relatórios
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/groups/report-users" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "report-users",
    "description": "Users who can generate reports"
  }'

# Usuário para relatórios
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/users/reporter" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "reporter",
    "email": "reporter@company.com",
    "password": "report123", 
    "groups": ["report-users"]
  }'

# Configurar permissões de relatório
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/permissions/report-permissions" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "report-permissions",
    "repositories": ["maven-local", "docker-local"],
    "principals": {
      "groups": {
        "report-users": ["r"]
      }
    }
  }'

# Testar geração de relatórios
# Relatório de uso de artefatos
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/search/usage?from=2024-01-01&to=2024-12-31" \
  -u "reporter:report123"

# Relatório de estatísticas de repositório
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/repositories/maven-local/statistics" \
  -u "reporter:report123"

# Relatório via AQL
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/search/aql" \
  -H "Content-Type: text/plain" \
  -u "reporter:report123" \
  -d 'items.find({"repo":"maven-local"}).include("name","repo","path","created","size")'

# Executar relatório customizado
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local?list&deep=1&listFolders=1&mdTimestamps=1&statsTimestamps=1" \
  -u "reporter:report123"
```

**Validação**: Validação de criação de perfil para criar, ler, alterar ou excluir relatórios sobre repositórios e artefatos.

---

### **Teste 5.6 - Papel para Publicação de Artefatos**
**Requisito**: Papel com permissões para publicar artefatos em repositórios

**Demonstração:**

```bash
# Criar papel de publicador
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/groups/publishers" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "publishers",
    "description": "Users who can publish artifacts"
  }'

# Usuário publicador
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/users/publisher" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "publisher",
    "email": "publisher@company.com",
    "password": "publish123",
    "groups": ["publishers"]
  }'

# Configurar permissões de publicação
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/permissions/publish-permissions" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "publish-permissions",
    "repositories": ["maven-releases", "maven-snapshots"],
    "principals": {
      "groups": {
        "publishers": ["w", "d"]
      }
    }
  }'

# Testar publicação com usuário publisher
curl -X PUT "https://yourinstance.jfrog.io/artifactory/maven-releases/com/example/published-app/1.0.0/published-app-1.0.0.jar" \
  -u "publisher:publish123" \
  -T "./published-app-1.0.0.jar"

# Verificar que publicação foi bem-sucedida
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/storage/maven-releases/com/example/published-app/1.0.0/published-app-1.0.0.jar" \
  -u "publisher:publish123"

# Testar que usuário sem permissão não consegue publicar
curl -X PUT "https://yourinstance.jfrog.io/artifactory/maven-releases/com/example/unauthorized/1.0.0/app.jar" \
  -u "regularuser:user123" \
  -T "./app.jar"
# Deve retornar 403 Forbidden
```

**Validação**: Validação de atribuição de papel que permite publicar documentos em repositórios controlados pelo produto.

---

### **Teste 5.7 - Papel para Políticas de Limpeza**  
**Requisito**: Papel com permissões CRUD para políticas de limpeza e remoção

**Demonstração:**

```bash
# Criar papel para administração de limpeza
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/groups/cleanup-admins" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "cleanup-admins",
    "description": "Cleanup policy administrators"
  }'

# Usuário administrador de limpeza
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/users/cleanupadmin" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "cleanupadmin",
    "email": "cleanup@company.com",
    "password": "cleanup123",
    "groups": ["cleanup-admins"]
  }'

# Testar CRUD de políticas de limpeza
# Criar política
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/retention" \
  -H "Content-Type: application/json" \
  -u "cleanupadmin:cleanup123" \
  -d '{
    "name": "test-cleanup-policy",
    "repository": "maven-local",
    "criteria": {
      "lastDownloaded": "60d"
    },
    "action": "delete"
  }'

# Ler política
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/retention/test-cleanup-policy" \
  -u "cleanupadmin:cleanup123"

# Alterar política  
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/retention/test-cleanup-policy" \
  -H "Content-Type: application/json" \
  -u "cleanupadmin:cleanup123" \
  -d '{
    "name": "test-cleanup-policy",
    "repository": "maven-local",
    "criteria": {
      "lastDownloaded": "90d"
    },
    "action": "delete"
  }'

# Executar política
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/retention/test-cleanup-policy/execute" \
  -u "cleanupadmin:cleanup123"

# Excluir política
curl -X DELETE "https://yourinstance.jfrog.io/artifactory/api/retention/test-cleanup-policy" \
  -u "cleanupadmin:cleanup123"
```

**Validação**: Validação de criação de perfil para criar, ler, alterar ou excluir políticas de limpeza e remoção de artefatos.

---

### **Teste 5.8 - Integração com Entidades Externas**
**Requisito**: Configurar papéis de LDAP, Entra ID (Azure AD) ou SAML

**Demonstração:**

```bash
# Configurar integração LDAP
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/system/configuration" \
  -H "Content-Type: application/xml" \
  -u "admin:password" \
  -d '<config>
    <security>
      <ldapSettings>
        <ldapSetting>
          <key>company-ldap</key>
          <enabled>true</enabled>
          <ldapUrl>ldap://ldap.company.com:389</ldapUrl>
          <userDnPattern>uid={0},ou=users,dc=company,dc=com</userDnPattern>
          <groupBaseDn>ou=groups,dc=company,dc=com</groupBaseDn>
          <groupNameAttribute>cn</groupNameAttribute>
          <groupMemberAttribute>member</groupMemberAttribute>
          <descriptionAttribute>description</descriptionAttribute>
          <emailAttribute>mail</emailAttribute>
          <autoCreateUser>true</autoCreateUser>
        </ldapSetting>
      </ldapSettings>
      <ldapGroupSettings>
        <ldapGroupSetting>
          <name>developers-ldap</name>
          <groupBaseDn>ou=groups,dc=company,dc=com</groupBaseDn>
          <groupNameAttribute>cn</groupNameAttribute>
          <descriptionAttribute>description</descriptionAttribute>
          <filter>(cn=developers)</filter>
          <strategy>STATIC</strategy>
        </ldapGroupSetting>
      </ldapGroupSettings>
    </security>
  </config>'

# Configurar mapeamento de grupos LDAP para papéis locais
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/security/groups/developers" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "name": "developers",
    "description": "Development team from LDAP",
    "autoJoin": false,
    "realm": "ldap",
    "realmAttributes": "developers-ldap"
  }'

# Configurar Azure AD/Entra ID (SAML)
curl -X PUT "https://yourinstance.jfrog.io/artifactory/api/saml/config" \
  -H "Content-Type: application/json" \
  -u "admin:password" \
  -d '{
    "enableIntegration": true,
    "loginUrl": "https://login.microsoftonline.com/your-tenant-id/saml2",
    "logoutUrl": "https://login.microsoftonline.com/your-tenant-id/saml2",
    "serviceProviderName": "Artifactory",
    "certificate": "-----BEGIN CERTIFICATE-----...",
    "groupAttribute": "http://schemas.microsoft.com/ws/2008/06/identity/claims/groups",
    "autoCreateUsers": true,
    "autoRedirect": true
  }'

# Testar login com usuário LDAP
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/system/ping" \
  -u "ldapuser:ldappassword"

# Verificar mapeamento de grupos
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/security/users/ldapuser" \
  -u "admin:password"

# Testar que grupos externos foram mapeados corretamente
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/security/groups/developers" \
  -u "admin:password"
```

**Validação**: Verificação do mapeamento na ferramenta de perfil conforme papéis configurados nos grupos externos.

---

## **CATEGORIA 6: SBOM - Testes 6.1 a 6.10**

### **Teste 6.1 - SBOM via API REST**
**Requisito**: API deve fornecer SBOM dos artefatos

**Demonstração:**

```bash
# Obter SBOM via API para artefato específico
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/sbom/build/my-build/1" \
  -H "Content-Type: application/json" \
  -u "username:password"

# SBOM de artefato específico
curl -X POST "https://yourinstance.jfrog.io/xray/api/v1/sbom/artifacts" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "artifacts": [{
      "repository": "docker-local",
      "path": "my-app:latest"
    }]
  }'

# Exportar SBOM em formato CycloneDX
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/export/sbom?repository=maven-local&path=com/example/app/1.0.0&format=cyclonedx" \
  -u "username:password" \
  -H "Accept: application/json"

# Exportar SBOM em formato SPDX  
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/export/sbom?repository=maven-local&path=com/example/app/1.0.0&format=spdx" \
  -u "username:password" \
  -H "Accept: application/json"
```

**Validação**: Validar consumo de SBOM via API.

---

### **Teste 6.2 - SBOM via Interface Gráfica**
**Requisito**: Produto deve fornecer SBOM dos artefatos via interface

**Demonstração via Interface Web:**
1. Navegar para `Application > Builds`
2. Selecionar build específico
3. Aba `Dependencies` para ver SBOM
4. Exportar SBOM em formato desejado
5. Verificar componentes, dependências e vulnerabilidades

**Via CLI:**
```bash
# Gerar build info com dependências
jfrog rt build-collect-env my-build 1
jfrog rt mvn clean install --build-name=my-build --build-number=1
jfrog rt build-publish my-build 1

# Visualizar SBOM via interface após publicação
# Acessar: https://yourinstance.jfrog.io/ui/builds/my-build/1
```

**Validação**: Validar consumo de SBOM via interface da aplicação.

---

### **Teste 6.3 - Formatos SBOM (CycloneDX/SPDX)**
**Requisito**: SBOMs devem estar em formato CycloneDX ou SPDX

**Demonstração:**

```bash
# Verificar suporte a CycloneDX
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/export/sbom?repository=maven-local&path=com/example/app/1.0.0&format=cyclonedx" \
  -u "username:password" \
  -H "Accept: application/json" > sbom-cyclonedx.json

# Verificar estrutura CycloneDX
cat sbom-cyclonedx.json | jq '.bomFormat'
cat sbom-cyclonedx.json | jq '.specVersion' 
cat sbom-cyclonedx.json | jq '.components[]'

# Verificar suporte a SPDX
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/export/sbom?repository=maven-local&path=com/example/app/1.0.0&format=spdx" \
  -u "username:password" \
  -H "Accept: application/json" > sbom-spdx.json

# Verificar estrutura SPDX
cat sbom-spdx.json | jq '.spdxVersion'
cat sbom-spdx.json | jq '.creationInfo'
cat sbom-spdx.json | jq '.packages[]'

# Validar conformidade com schemas
# CycloneDX validation
curl -X POST "https://cyclonedx.org/tool-center/validate" \
  -F "file=@sbom-cyclonedx.json"

# SPDX validation  
curl -X POST "https://tools.spdx.org/validate" \
  -F "file=@sbom-spdx.json"
```

**Validação**: Validar retorno nos formatos CycloneDX ou SPDX.

---

### **Teste 6.4 - Informações Mínimas NTIA**
**Requisito**: Fornecer informações mínimas sobre artefatos conforme NTIA

**Demonstração:**

```bash
# Obter informações completas do artefato
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/storage/maven-local/com/example/app/1.0.0/app-1.0.0.jar?properties" \
  -u "username:password" | jq '{
    name: .uri,
    supplier: .properties."build.created-by"[0],
    version: .properties."build.version"[0], 
    timestamp: .created,
    checksums: .checksums
  }'

# SBOM com informações NTIA
curl -X POST "https://yourinstance.jfrog.io/xray/api/v1/sbom/artifacts" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "artifacts": [{
      "repository": "maven-local",
      "path": "com/example/app/1.0.0/app-1.0.0.jar"
    }]
  }' | jq '{
    component_name: .components[0].name,
    supplier: .components[0].supplier,
    version: .components[0].version,
    dependencies: .components[0].dependencies,
    timestamp: .metadata.timestamp
  }'

# Verificar dependências transitivas
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/search/aql" \
  -H "Content-Type: text/plain" \
  -u "username:password" \
  -d 'builds.find({"name":"my-build","number":"1"}).include("name","number","dependencies")'

# Build info com metadados completos
curl -X GET "https://yourinstance.jfrog.io/artifactory/api/build/my-build/1" \
  -u "username:password" | jq '{
    name: .buildInfo.name,
    number: .buildInfo.number,
    started: .buildInfo.started,
    modules: [.buildInfo.modules[] | {
      id: .id,
      artifacts: [.artifacts[] | {name: .name, type: .type, sha1: .sha1}],
      dependencies: [.dependencies[] | {id: .id, type: .type, sha1: .sha1}]
    }]
  }'
```

**Validação**: Validar informações do artefato conforme publicação NTIA.

---

### **Teste 6.5 - Análise de Vulnerabilidades com VulnDB/NVD**
**Requisito**: Analisar vulnerabilidades usando bases de referência

**Demonstração:**

```bash
# Configurar fontes de dados de vulnerabilidade
curl -X PUT "https://yourinstance.jfrog.io/xray/api/v1/system/db/sync" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "nvd": {
      "enabled": true,
      "url": "https://nvd.nist.gov/feeds/json/cve/1.1/"
    },
    "vulndb": {
      "enabled": true
    }
  }'

# Scan de artefato open-source conhecido com vulnerabilidades
curl -X POST "https://yourinstance.jfrog.io/xray/api/v1/scanArtifact" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "repository": "maven-local",
    "path": "log4j/log4j/1.2.17/log4j-1.2.17.jar"
  }'

# Verificar vulnerabilidades detectadas
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/artifacts/vulnerabilities" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "artifacts": [{
      "repository": "maven-local", 
      "path": "log4j/log4j/1.2.17/log4j-1.2.17.jar"
    }]
  }' | jq '.artifacts[0].vulnerabilities[] | {
    cve: .cve,
    severity: .severity,
    summary: .summary,
    sources: [.sources[] | {source_id: .source_id}]
  }'

# Verificar fonte das vulnerabilidades (NVD/VulnDB)
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/vulnerabilities/CVE-2021-44228" \
  -u "username:password" | jq '{
    cve: .cve,
    sources: [.sources[] | .source_id]
  }'
```

**Validação**: Validar que artefato está na base de referência VulnDB ou NVD NIST.

---

### **Teste 6.6 - Análise de Violações de Licenças**
**Requisito**: Analisar violações de licenças usando bases de referência

**Demonstração:**

```bash
# Scan de licenças em artefatos
curl -X POST "https://yourinstance.jfrog.io/xray/api/v1/scanArtifact" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "repository": "maven-local",
    "path": "com/example/app/1.0.0/app-1.0.0.jar",
    "scan_types": ["license"]
  }'

# Verificar licenças detectadas
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/artifacts/licenses" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "artifacts": [{
      "repository": "maven-local",
      "path": "com/example/app/1.0.0/app-1.0.0.jar"
    }]
  }' | jq '.artifacts[0].licenses[] | {
    name: .name,
    full_name: .full_name,
    more_info_url: .more_info_url
  }'

# Configurar política de licenças
curl -X POST "https://yourinstance.jfrog.io/xray/api/v2/policies" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "name": "license-compliance-policy",
    "type": "license",
    "rules": [{
      "name": "prohibited-licenses",
      "criteria": {
        "banned_licenses": ["GPL-3.0", "AGPL-3.0"]
      },
      "actions": {
        "block_download": {
          "active": true
        }
      }
    }]
  }'

# Testar violação de licença
curl -X GET "https://yourinstance.jfrog.io/artifactory/maven-local/com/example/gpl-app/1.0.0/gpl-app-1.0.0.jar" \
  -u "username:password"
# Deve ser bloqueado se licença GPL detectada
```

**Validação**: Validar que artefato está em conformidade com licença de uso baseado na referência VulnDB ou NVD NIST.

---

### **Teste 6.7 - Bloqueio de Artefatos por Políticas**
**Requisito**: Impedir download de artefatos que violam políticas

**Demonstração:**

```bash
# Criar política restritiva
curl -X POST "https://yourinstance.jfrog.io/xray/api/v2/policies" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "name": "security-block-policy",
    "type": "security", 
    "rules": [{
      "name": "critical-vulnerabilities",
      "criteria": {
        "min_severity": "Critical"
      },
      "actions": {
        "block_download": {
          "active": true,
          "unscanned": false
        }
      }
    }]
  }'

# Criar watch para aplicar política
curl -X POST "https://yourinstance.jfrog.io/xray/api/v2/watches" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "general_data": {
      "name": "security-watch"
    },
    "project_resources": {
      "repositories": [{"type": "local", "name": "maven-local"}]
    },
    "assigned_policies": [{
      "name": "security-block-policy",
      "type": "security"
    }]
  }'

# Upload de artefato com vulnerabilidade conhecida
curl -X PUT "https://yourinstance.jfrog.io/artifactory/maven-local/log4j/log4j/1.2.17/log4j-1.2.17.jar" \
  -u "username:password" \
  -T "./log4j-1.2.17.jar"

# Aguardar scan automático, depois tentar download
sleep 30
curl -X GET "https://yourinstance.jfrog.io/artifactory/maven-local/log4j/log4j/1.2.17/log4j-1.2.17.jar" \
  -u "username:password"
# Deve retornar erro de bloqueio

# Verificar bloqueio via API
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/violations" \
  -u "username:password" | jq '.violations[] | select(.artifact_path | contains("log4j"))'
```

**Validação**: Validar que tentativa de upload de artefato sem licença é bloqueada conforme política definida.

---

### **Teste 6.8 - Políticas de Bloqueio por Vulnerabilidades**
**Requisito**: Implementar políticas que impeçam uso de artefatos vulneráveis

**Demonstração:**

```bash
# Política específica para vulnerabilidades (referência F-6.4)
curl -X POST "https://yourinstance.jfrog.io/xray/api/v2/policies" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "name": "vulnerability-prevention-policy",
    "type": "security",
    "rules": [{
      "name": "high-severity-block",
      "criteria": {
        "min_severity": "High",
        "cvss_range": {
          "from": 7.0,
          "to": 10.0
        }
      },
      "actions": {
        "block_download": {
          "active": true
        },
        "fail_build": {
          "active": true
        }
      }
    }]
  }'

# Aplicar política a repositórios específicos
curl -X POST "https://yourinstance.jfrog.io/xray/api/v2/watches" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "general_data": {
      "name": "vulnerability-watch"
    },
    "project_resources": {
      "repositories": [
        {"type": "local", "name": "maven-local"},
        {"type": "local", "name": "docker-local"}
      ]
    },
    "assigned_policies": [{
      "name": "vulnerability-prevention-policy", 
      "type": "security"
    }]
  }'

# Testar bloqueio com artefato vulnerável
# Upload de imagem Docker com vulnerabilidades conhecidas
docker pull vulnerables/web-dvwa:latest
docker tag vulnerables/web-dvwa:latest yourinstance.jfrog.io/docker-local/vulnerable-app:latest
docker push yourinstance.jfrog.io/docker-local/vulnerable-app:latest

# Aguardar scan e tentar pull
sleep 60
docker pull yourinstance.jfrog.io/docker-local/vulnerable-app:latest
# Deve falhar devido a política de bloqueio

# Verificar violações
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/violations?direction=asc&order_by=created&page_num=1&num_of_rows=25" \
  -u "username:password" | jq '.violations[] | select(.artifact_path | contains("vulnerable-app"))'
```

**Validação**: Validar que tentativa de upload de artefato com vulnerabilidade é bloqueada conforme política definida.

---

### **Teste 6.9 - Políticas de Bloqueio por Licenças**
**Requisito**: Implementar políticas que impeçam uso de artefatos com licenças incompatíveis

**Demonstração:**

```bash
# Política para bloqueio de licenças incompatíveis
curl -X POST "https://yourinstance.jfrog.io/xray/api/v2/policies" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "name": "license-compatibility-policy",
    "type": "license",
    "rules": [{
      "name": "incompatible-licenses",
      "criteria": {
        "allowed_licenses": ["Apache-2.0", "MIT", "BSD-3-Clause"],
        "banned_licenses": ["GPL-3.0", "AGPL-3.0", "LGPL-3.0"]
      },
      "actions": {
        "block_download": {
          "active": true
        },
        "notify_deployer": {
          "active": true
        }
      }
    }]
  }'

# Aplicar política
curl -X POST "https://yourinstance.jfrog.io/xray/api/v2/watches" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "general_data": {
      "name": "license-watch"
    },
    "project_resources": {
      "repositories": [{"type": "local", "name": "maven-local"}]
    },
    "assigned_policies": [{
      "name": "license-compatibility-policy",
      "type": "license"
    }]
  }'

# Upload de artefato com licença incompatível (simulação)
curl -X PUT "https://yourinstance.jfrog.io/artifactory/maven-local/com/example/gpl-lib/1.0.0/gpl-lib-1.0.0.jar?properties=license=GPL-3.0" \
  -u "username:password" \
  -T "./gpl-lib-1.0.0.jar"

# Aguardar scan e tentar download
sleep 30
curl -X GET "https://yourinstance.jfrog.io/artifactory/maven-local/com/example/gpl-lib/1.0.0/gpl-lib-1.0.0.jar" \
  -u "username:password"
# Deve ser bloqueado

# Verificar bloqueio por licença
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/violations" \
  -u "username:password" | jq '.violations[] | select(.type == "license" and (.artifact_path | contains("gpl-lib")))'
```

**Validação**: Validar que tentativas de uso de artefatos cujas licenças são incompatíveis com políticas definidas são impedidas de ser consumidas.

---

### **Teste 6.10 - Identificação de Licenças Open-Source**
**Requisito**: Analisar e identificar licenças usando bases de referência

**Demonstração:**

```bash
# Scan completo de licenças em repositório
curl -X POST "https://yourinstance.jfrog.io/xray/api/v1/scanRepository" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "repository": "maven-local",
    "scan_types": ["license", "security"]
  }'

# Relatório de licenças por repositório
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/summary/repository" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "repository": "maven-local"
  }' | jq '.licenses[] | {
    name: .name,
    count: .count,
    components: [.components[] | .component_id]
  }'

# Identificar licenças específicas de artefatos open-source populares
# Apache Commons
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/artifacts/licenses" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "artifacts": [{
      "repository": "maven-central-remote",
      "path": "org/apache/commons/commons-lang3/3.12.0/commons-lang3-3.12.0.jar"
    }]
  }'

# JUnit  
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/artifacts/licenses" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "artifacts": [{
      "repository": "maven-central-remote", 
      "path": "junit/junit/4.13.2/junit-4.13.2.jar"
    }]
  }'

# Verificar fonte das informações de licença
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/licenses/Apache-2.0" \
  -u "username:password" | jq '{
    name: .name,
    full_name: .full_name,
    references: [.references[] | .source]
  }'

# Gerar relatório consolidado de uso de componentes
curl -X POST "https://yourinstance.jfrog.io/artifactory/api/search/aql" \
  -H "Content-Type: text/plain" \
  -u "username:password" \
  -d 'items.find({"repo":"maven-local"}).include("name","path","property.key","property.value")' | \
  jq '.results[] | select(.properties[]?.key == "license") | {
    artifact: .name,
    path: .path,
    license: (.properties[] | select(.key == "license") | .value)
  }'
```

**Validação**: Validar que ferramenta identifica licenças dos artefatos baseado em VulnDB e NVD NIST.

---

## **CATEGORIA 7: SEGURANÇA - Testes 7.1 a 7.2**

### **Teste 7.1 - Varredura de Vulnerabilidades**
**Requisito**: Realizar varredura de vulnerabilidades em artefatos e imagens

**Demonstração:**

```bash
# Configurar Xray para varredura automática
curl -X PUT "https://yourinstance.jfrog.io/xray/api/v1/binMgr/default/repos" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "indexed_repos": [{
      "name": "maven-local",
      "type": "local"
    }, {
      "name": "docker-local", 
      "type": "local"
    }]
  }'

# Upload e scan de artefato JAR
curl -X PUT "https://yourinstance.jfrog.io/artifactory/maven-local/com/example/vulnerable-app/1.0.0/vulnerable-app-1.0.0.jar" \
  -u "username:password" \
  -T "./vulnerable-app-1.0.0.jar"

# Scan manual de artefato
curl -X POST "https://yourinstance.jfrog.io/xray/api/v1/scanArtifact" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "repository": "maven-local",
    "path": "com/example/vulnerable-app/1.0.0/vulnerable-app-1.0.0.jar"
  }'

# Upload e scan de imagem Docker
docker build -t vulnerable-image:latest .
docker tag vulnerable-image:latest yourinstance.jfrog.io/docker-local/vulnerable-image:latest
docker push yourinstance.jfrog.io/docker-local/vulnerable-image:latest

# Scan de imagem Docker
curl -X POST "https://yourinstance.jfrog.io/xray/api/v1/scanArtifact" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "repository": "docker-local",
    "path": "vulnerable-image/latest"
  }'

# Obter resultados da varredura - JAR
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/artifacts/vulnerabilities" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "artifacts": [{
      "repository": "maven-local",
      "path": "com/example/vulnerable-app/1.0.0/vulnerable-app-1.0.0.jar"
    }]
  }' | jq '.artifacts[0] | {
    path: .general.path,
    vulnerabilities: [.vulnerabilities[] | {
      cve: .cve,
      severity: .severity,
      summary: .summary,
      fixed_versions: .fixed_versions
    }]
  }'

# Obter resultados da varredura - Docker
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/artifacts/vulnerabilities" \
  -H "Content-Type: application/json" \
  -u "username:password" \
  -d '{
    "artifacts": [{
      "repository": "docker-local",
      "path": "vulnerable-image/latest"
    }]
  }' | jq '.artifacts[0] | {
    image: .general.path,
    total_vulnerabilities: (.vulnerabilities | length),
    critical: [.vulnerabilities[] | select(.severity == "Critical")],
    high: [.vulnerabilities[] | select(.severity == "High")]
  }'

# Relatório consolidado de vulnerabilidades
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/summary/vulnerabilities" \
  -u "username:password" | jq '{
    total: .total_vulnerabilities,
    by_severity: .vulnerabilities_by_severity,
    top_cves: [.top_vulnerabilities[] | {cve: .cve, count: .count}]
  }'
```

**Validação**: Validar relatório gerado sobre varredura de vulnerabilidades detectadas nos artefatos.

---

### **Teste 7.2 - Resultados via Interface Gráfica**
**Requisito**: Apresentar resultados de F-7.1 através da interface gráfica

**Demonstração via Interface Web:**

**Passos na Interface:**
1. **Acessar Xray Dashboard:**
   - Navegar para `Application > Security & Compliance`
   - Visualizar dashboard de vulnerabilidades

2. **Visualizar Artefatos Vulneráveis:**
   - Ir para `Violations` tab
   - Filtrar por repositório: `maven-local`, `docker-local`
   - Expandir violações para ver detalhes

3. **Análise Detalhada de Artefato:**
   - Clicar em artefato específico
   - Visualizar gráfico de dependências
   - Ver vulnerabilidades por componente
   - Examinar informações de CVE

4. **Relatórios de Segurança:**
   - Gerar relatório de vulnerabilidades
   - Exportar resultados em PDF/Excel
   - Configurar notificações por email

**Comandos para Preparar Dados na Interface:**
```bash
# Garantir que há dados para visualizar
# Upload de artefatos conhecidos com vulnerabilidades
docker pull vulnerables/web-dvwa:latest
docker tag vulnerables/web-dvwa:latest yourinstance.jfrog.io/docker-local/web-dvwa:latest
docker push yourinstance.jfrog.io/docker-local/web-dvwa:latest

# Upload JAR com dependências vulneráveis
curl -X PUT "https://yourinstance.jfrog.io/artifactory/maven-local/log4j/log4j/1.2.17/log4j-1.2.17.jar" \
  -u "username:password" \
  -T "./log4j-1.2.17.jar"

# Aguardar indexação automática
sleep 60

# Verificar se dados apareceram na interface
curl -X GET "https://yourinstance.jfrog.io/xray/api/v1/violations" \
  -u "username:password" | jq '.violations | length'
```

**Screenshots Necessários para Validação:**
1. Dashboard principal do Xray mostrando estatísticas
2. Lista de violações com artefatos vulneráveis  
3. Detalhes de uma vulnerabilidade específica
4. Gráfico de impacto de dependências
5. Relatório de varredura exportado

**Validação**: Exibir detalhes do artefato com vulnerabilidade através da interface gráfica.

---

## **RELATÓRIO DE EVIDÊNCIAS ATUALIZADO**

### **Checklist Completo de Validação dos Testes**

| Teste | Categoria | Descrição | Status | Evidência | Tempo Real |
|-------|-----------|-----------|---------|-----------|------------|
| 1.1 | Repositório | Múltiplas tecnologias | ☐ | Screenshots repositórios | 300 min |
| 1.2 | Repositório | Proxying e caching | ☐ | Logs download via proxy | ___ min |
| 1.3 | Repositório | Políticas de limpeza | ☐ | Relatório execução política | ___ min |
| 1.4 | Repositório | Movimentação artefatos | ☐ | Antes/depois movimentação | ___ min |
| 1.5 | Repositório | Crescimento ilimitado | ☐ | Métricas armazenamento | ___ min |
| 1.6 | Repositório | Sem restrição acessos | ☐ | Logs sem bloqueios | ___ min |
| 1.7 | Repositório | Múltiplas instâncias | ☐ | Lista repositórios | ___ min |
| 2.1 | API | API REST | ☐ | Respostas JSON/XML | ___ min |
| 2.2 | API | Documentação API | ☐ | Acesso documentação | ___ min |
| 2.3 | API | APIs automação políticas | ☐ | Execução API políticas | ___ min |
| 2.4 | API | API vulnerabilidades | ☐ | Vulnerabilidades via API | ___ min |
| 2.5 | API | API marcar vulneráveis | ☐ | Marcação via API | ___ min |
| 2.6 | API | API publicação | ☐ | Upload via API | ___ min |
| 2.7 | CI/CD | GitHub Actions restrições | ☐ | Pipeline interrompido | ___ min |
| 2.8 | CI/CD | GitHub Actions publicação | ☐ | Artefato publicado | ___ min |
| 3.1 | Integridade | Verificação SHA-2/SHA-3 | ☐ | Checksum validado | ___ min |
| 3.2 | Integridade | Identificador único | ☐ | Unicidade confirmada | ___ min |
| 3.3 | Integridade | RELEASE imutável | ☐ | Bloqueio redeploy | ___ min |
| 3.4 | Integridade | SNAPSHOT flexível | ☐ | Redeploy permitido | ___ min |
| 3.5 | Integridade | Marcação RELEASE | ☐ | Tipos marcação | ___ min |
| 4.1 | Metadados | Políticas acesso metadados | ☐ | Política aplicada | ___ min |
| 4.2 | Metadados | Limpeza por metadados | ☐ | Limpeza executada | ___ min |
| 5.1 | Acesso | Autenticação obrigatória | ☐ | Bloqueio não autenticado | ___ min |
| 5.2 | Acesso | Controle por papéis | ☐ | Acesso por papel | ___ min |
| 5.3 | Acesso | Papel admin usuários | ☐ | CRUD usuários | ___ min |
| 5.4 | Acesso | Papel admin políticas | ☐ | CRUD políticas | ___ min |
| 5.5 | Acesso | Papel relatórios | ☐ | Geração relatórios | ___ min |
| 5.6 | Acesso | Papel publicação | ☐ | Upload permitido | ___ min |
| 5.7 | Acesso | Papel limpeza | ☐ | CRUD políticas limpeza | ___ min |
| 5.8 | Acesso | Integração externa | ☐ | Mapeamento grupos | ___ min |
| 6.1 | SBOM | SBOM via API | ☐ | SBOM consumido API | ___ min |
| 6.2 | SBOM | SBOM via interface | ☐ | SBOM via aplicação | ___ min |
| 6.3 | SBOM | Formatos CycloneDX/SPDX | ☐ | Formato validado | ___ min |
| 6.4 | SBOM | Informações NTIA | ☐ | Dados conforme NTIA | ___ min |
| 6.5 | SBOM | Vulnerabilidades VulnDB/NVD | ☐ | Vulnerabilidade detectada | ___ min |
| 6.6 | SBOM | Licenças VulnDB/NVD | ☐ | Licença identificada | ___ min |
| 6.7 | SBOM | Bloqueio por políticas | ☐ | Download bloqueado | ___ min |
| 6.8 | SBOM | Bloqueio vulnerabilidades | ☐ | Upload bloqueado | ___ min |
| 6.9 | SBOM | Bloqueio licenças | ☐ | Uso impedido | ___ min |
| 6.10 | SBOM | Identificação licenças | ☐ | Licenças identificadas | ___ min |
| 7.1 | Segurança | Varredura vulnerabilidades | ☐ | Relatório varredura | ___ min |
| 7.2 | Segurança | Interface gráfica | ☐ | Detalhes na interface | ___ min |

### **Tempo Total Estimado Atualizado**
**Total**: Aproximadamente 10-12 horas de demonstração completa

**Avaliadores**: TIC/AID/ARQTIC/ARQNUV e TIC/OI/ADG/IDN

O guia agora está completo com todos os testes funcionais especificados no caderno, desde repositórios básicos até funcionalidades avançadas de segurança e SBOM.

### **Demonstração de Capacidades de Segurança**

```bash
# Xray - Análise de Vulnerabilidades
jfrog xr scan --repo=maven-local

# Configurar política de segurança
curl -X POST "https://yourinstance.jfrog.io/xray/api/v2/policies" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "critical-policy",
    "type": "security",
    "rules": [{
      "name": "critical-vulnerabilities",
      "priority": 1,
      "criteria": {
        "min_severity": "Critical"
      },
      "actions": {
        "block_download": {
          "unscanned": false,
          "active": true
        }
      }
    }]
  }'

# Watch para monitoramento
curl -X POST "https://yourinstance.jfrog.io/xray/api/v2/watches" \
  -H "Content-Type: application/json" \
  -d '{
    "general_data": {
      "name": "security-watch",
      "description": "Watch for security violations"
    },
    "project_resources": {
      "repositories": [{
        "type": "local",
        "name": "maven-local"
      }]
    },
    "assigned_policies": [{
      "name": "critical-policy",
      "type": "security"
    }]
  }'
```

---

## **CATEGORIA 4: INTEGRAÇÃO E AUTOMAÇÃO**

### **Demonstração de Integrações CI/CD**

```yaml
# Jenkins Pipeline
pipeline {
    agent any
    stages {
        stage('Upload Artifacts') {
            steps {
                script {
                    def server = Artifactory.server 'jfrog-instance'
                    def uploadSpec = """{
                        "files": [{
                            "pattern": "target/*.jar",
                            "target": "maven-local/com/example/app/${BUILD_NUMBER}/"
                        }]
                    }"""
                    server.upload(uploadSpec)
                }
            }
        }
        stage('Security Scan') {
            steps {
                xrayScan failBuild: true, buildName: env.JOB_NAME, buildNumber: env.BUILD_NUMBER
            }
        }
    }
}
```

```bash
# GitHub Actions equivalent
name: JFrog Integration Test
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup JFrog CLI
        uses: jfrog/setup-jfrog-cli@v2
      - name: Upload and Scan
        run: |
          jfrog rt upload target/*.jar maven-local/
          jfrog xr scan --repo=maven-local
```

---

## **RELATÓRIO DE EVIDÊNCIAS**

### **Checklist de Validação dos Testes**

| Teste | Descrição | Status | Evidência | Tempo Real |
|-------|-----------|---------|-----------|------------|
| 1.1 | Suporte múltiplas tecnologias | ☐ | Screenshots dos repositórios criados | ___ min |
| 1.2 | Proxying e caching | ☐ | Logs de download via proxy | ___ min |
| 1.3 | Políticas de limpeza | ☐ | Relatório de execução da política | ___ min |
| 1.4 | Movimentação de artefatos | ☐ | Antes/depois da movimentação | ___ min |
| 1.5 | Crescimento ilimitado | ☐ | Métricas de armazenamento | ___ min |
| 1.6 | Sem restrição de acessos | ☐ | Logs de acesso sem bloqueios | ___ min |
| 1.7 | Múltiplas instâncias | ☐ | Lista de repositórios da mesma tecnologia | ___ min |
| 2.1 | API REST | ☐ | Respostas JSON/XML das APIs | ___ min |

### **Documentação de Resultados**
- **Screenshots**: Capturar telas de cada funcionalidade demonstrada
- **Logs**: Salvar logs de execução de comandos críticos
- **Métricas**: Coletar dados de performance e utilização
- **Relatórios**: Gerar relatórios de segurança e compliance

### **Tempo Total Estimado**
**Baseado no caderno de testes**: ~300 minutos (5 horas) conforme especificado para o teste 1.1

---

## **PRÓXIMOS PASSOS**

1. **Execução Sequencial**: Seguir os testes na ordem especificada
2. **Documentação**: Registrar todos os resultados conforme template
3. **Validação**: Confirmar critérios de aceitação para cada teste
4. **Relatório Final**: Compilar evidências para avaliação final

## **Observações Importantes**

- Todos os comandos devem ser executados com usuário com privilégios administrativos
- Manter backup dos dados antes dos testes de limpeza
- Documentar qualquer limitação encontrada durante os testes
- Seguir exatamente os critérios de aceitação definidos no caderno de testes

---

**Avaliadores**: TIC/AID/ARQTIC/ARQNUV e TIC/OI/ADG/IDN conforme especificado no documento original.
