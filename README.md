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

## **CATEGORIA 3: SEGURANÇA E ANÁLISE (Presumível do contexto)**

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
