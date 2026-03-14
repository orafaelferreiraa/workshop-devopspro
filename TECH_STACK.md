# 🚀 Stack Tecnológico Completo - Platform as a Service Stack

## 📋 Índice

1. [Infrastructure as Code](#infrastructure-as-code)
2. [Cloud & Azure Services](#cloud--azure-services)
3. [CI/CD & GitHub Actions](#cicd--github-actions)
4. [Validation & Security Tools](#validation--security-tools)
5. [Development Tools](#development-tools)
6. [Architecture Concepts](#architecture-concepts)
7. [Design Patterns](#design-patterns)

---

## 🏗️ Infrastructure as Code

### Terraform
| Componente | Descrição | Uso |
|-----------|-----------|-----|
| **Terraform Core** | `~> 1.14` | Orquestração e provisionamento de infraestrutura |
| **Providers** | `azurerm ~> 4.64.0`  | Azure Resource Manager - gerenciamento de recursos Azure |
| | `random ~> 3.8.1` | Geração de sufixos aleatórios para naming |
| | `time ~> 0.13.1` | Delay de propagação RBAC (180s) |
| **Backend** | Azure Blob Storage | State remoto com autenticação Azure AD (`use_azuread_auth = true`) |
| **Versioning** | Semantic Versioning | 3.0.0 (completo com feature flags + RBAC) |

### Módulos Terraform (Platform Stack)
| Módulo | Responsabilidade | Tipo |
|--------|-----------------|------|
| **foundation/naming** | Convenção de nomenclatura | Base |
| **foundation/resource-group** | Lifecycle management (prevent_destroy) | Base |
| **security/managed-identity** | User-Assigned Identity para RBAC | Segurança |
| **security/key-vault** | Secrets + RBAC habilitado | Segurança |
| **networking/vnet-spoke** | Virtual Network com subnets delegadas | Rede |
| **workloads/storage-account** | Blob Storage (Azure AD only) | Dados |
| **workloads/service-bus** | Message Queue + Topics (Standard tier) | Mensageria |
| **workloads/event-grid** | Domain para roteamento de eventos | Eventos |
| **workloads/sql** | SQL Server + Database + AAD admin | Dados |
| **workloads/observability** | Log Analytics + Application Insights | Monitoramento |
| **workloads/container-apps** | Serverless containers com Log Analytics | Computação |
| **workloads/container-registry** | ACR com RBAC automático | Registry |

### Módulos Externos (tfmodules-as-a-service-stack)
| Módulo | Responsabilidade |
|--------|-----------------|
| **azurerm_container_registry** | Container Registry com Managed Identity + Diagnóstico |

---

## ☁️ Cloud & Azure Services

### Computação
| Serviço | Função | Configuração |
|---------|--------|--------------|
| **Azure Container Apps** | Serverless container platform | REQUER Observability (Log Analytics) |
| | | Integração com ACR (AcrPull) |
| | | Anexação automática de Managed Identity |
| | | Infrastructure subnet /27 (delegada) |

### Armazenamento
| Serviço | Função | Configuração |
|---------|--------|--------------|
| **Storage Account** | Blob storage | `shared_access_key_enabled = false` |
| | | Autenticação somente Azure AD |
| | | Diagnostic settings via Log Analytics |
| **SQL Server + Database** | Relational database | Admin: Azure AD |
| | | TLS 1.2+ obrigatório |
| | | Firewall rules via VNet |
| | | Password armazenada em Key Vault |

### Mensageria & Eventos
| Serviço | Função | Configuração |
|---------|--------|--------------|
| **Service Bus** | Message Queue + Topics | Tier: Standard |
| | | RBAC automático via Managed Identity |
| **Event Grid Domain** | Event routing | Integração com Service Bus |
| | | Subscriptions por Domain Topic |

### Segurança & Identidade
| Serviço | Função | Configuração |
|---------|--------|--------------|
| **Azure Key Vault** | Secrets & Keys | RBAC enabled (sem Access Policies) |
| | | 180s delay antes de criar secrets (propagação RBAC) |
| **Managed Identity (User-Assigned)** | RBAC sem senha | Compartilhada entre múltiplos recursos |
| | | Roles: Storage Blob Data Contributor, Key Vault Secrets Officer, etc. |
| **Azure AD (Entra ID)** | Autenticação centralizada | SQL admin, Storage auth, Key Vault auth |

### Rede
| Serviço | Função | Configuração |
|---------|--------|--------------|
| **Virtual Network Spoke** | Rede isolada | Integração com Hub (futura) |
| | | Subnet default + delegada para Container Apps |
| | | Container Apps subnet: /27 mínimo |
| | | Network rules em Storage Account |
| **Network Security Group** | Firewall de subnet | Criado automaticamente pela VNet |

### Monitoramento & Observabilidade
| Serviço | Função | Configuração |
|---------|--------|--------------|
| **Log Analytics Workspace** | Centralização de logs | Retenção: 30 dias |
| **Application Insights** | APM (Application Performance Monitoring) | Tipo: web |
| | | Linked ao Log Analytics |
| **Diagnostic Settings** | Stream de logs para Log Analytics | Habilitado para Storage, SQL, ACR, Service Bus |

### Registry
| Serviço | Função | Configuração |
|---------|--------|--------------|
| **Container Registry (ACR)** | Docker image storage | SKU: Basic (configurável) |
| | | RBAC: AcrPush + AcrPull via Managed Identity |
| | | Autenticação: Managed Identity (sem credenciais) |

---

## 🔄 CI/CD & GitHub Actions

### Workflow Framework
| Componente | Descrição | Propósito |
|-----------|-----------|----------|
| **pipeline-core.yaml** | Reusable workflow (workflow_call) | Centraliza toda validação Terraform |
| **deploy-plan.yaml** | Chamador 1 | Disparado em PR + manual > validação + plan |
| **deploy-apply.yaml** | Chamador 2 | Disparado em push main + manual > apply |

### Stages da Pipeline Core

#### 1️⃣ Terraform Format
- **Tool**: `terraform fmt`
- **Scope**: Recursivo em diretório
- **Output**: Reformatação automática

#### 2️⃣ TFLint (Linting)
- **Tool**: TFLint `latest`
- **Scope**: Best practices + style guide
- **Flag**: `enable_tflint` (default: true)

#### 3️⃣ Trivy (Security Scanning)
- **Tool**: Trivy `aquasecurity/trivy-action@0.35.0`
- **Output**: SARIF → GitHub Security tab
- **Flag**: `enable_trivy` (default: true)

#### 4️⃣ Checkov (Policy Compliance)
- **Tool**: `checkov` (pip) com Python 3.12
- **Output**: SARIF → GitHub Security tab
- **Configuração**: `.checkov.yaml` na raiz
- **Flag**: `enable_checkov` (default: true)

#### 5️⃣ Terraform Docs (Documentation)
- **Tool**: `terraform-docs/gh-actions` v1.3.0
- **Modo**: `inject` direto no README.md
- **Drift Detection**: `git diff` para alertar mudanças
- **PR Comment**: Notifica se docs desatualizado
- **Flag**: `generate_tfdocs` (default: true)

### GitHub Actions Features
| Recurso | Descrição |
|---------|-----------|
| **Caching** | Plugin TFLint (chave: `.tflint.hcl`) |
| **Soft Fail** | Continue pipeline mesmo com erros (flag: `soft_fail`) |
| **Step Summary** | Tabela consolidada de resultados |
| **PR Comments** | Documentação drift + validação feedback |
| **Workflow State Protection** | Previne destroy acidental (validação tfstate) |

### Actions Utilizadas
| Action | Função |
|--------|--------|
| `actions/checkout@v4` | Clone do repositório |
| `actions/cache@v4` | Caching de dependências |
| `hashicorp/setup-terraform@v3` | Setup Terraform |
| `terraform-linters/setup-tflint@v4` | Setup TFLint |
| `aquasecurity/trivy-action@0.35.0` | Trivy security scanning |
| `actions/setup-python@v5` | Setup Python 3.12 (Checkov) |
| `github/codeql-action/upload-sarif@v4` | Upload SARIF reports |
| `terraform-docs/gh-actions@v1.3.0` | Docs generation |
| `actions/github-script@v7` | Custom scripts para comentários |

---

## ✅ Validation & Security Tools

### Linting & Code Quality
| Tool | Função | Output |
|------|--------|--------|
| **TFLint** | Linting de código Terraform | JSON + console |
| **terraform fmt** | Formatação automática | -recursive flag |

### Security Scanning
| Tool | Função | Output |
|------|--------|--------|
| **Trivy** | SAST para Terraform | SARIF (GitHub Security) |
| **Checkov** | Policy compliance IaC | SARIF (GitHub Security) |

### Documentation
| Tool | Função | Output |
|------|--------|--------|
| **terraform-docs** | Auto-generate docs | Markdown inject em README |
| **Drift Detection** | Detecta desatualização | PR comment + CI failure |

### Diagnostic Tools
| Tool | Função | Uso |
|------|--------|-----|
| **terraform validate** | Validação de sintaxe | Implícito na pipeline |
| **terraform plan** | Plano de mudanças | PR review antes de apply |

---

## 🛠️ Development Tools

### Version Control
| Tool  | Uso |
|------|-----|
| **Git** | SSH authentication |
| **GitHub** | Repository hosting + Actions |

### IDE & Editors
| Tool | Função | MCPs |
|------|--------|------|
| **VS Code** | Code editor | ✅ Terraform |
| | | ✅ GitHub Actions |
| | | ✅ Microsoft Learn |
| | | ✅ context7 |
| **GitHub Copilot** | AI-powered coding | Chat + inline suggestions |
| **GitHub Copilot Chat** | Extended AI | Code explanation + debugging |

### Containerization
| Tool | Versão | Uso |
|------|--------|-----|
| **Docker Desktop** | Latest | Local container development |

### Local Development Environment
| Componente | Descrição |
|-----------|-----------|
| **Windows 11 Enterprise** | Host OS (VM ou local) |
| **WSL2** | Windows Subsystem for Linux |
| **Ubuntu** | Container Linux para ferramentas |
| **PowerShell 5.1** | Shell nativo Windows |
| **Bash/Shell** | WSL environment |

### CLI Tools
| Tool | Propósito | Instalação |
|------|-----------|-----------|
| **Terraform CLI** | IaC provisioning | choco/apt/direct |
| **Azure CLI (az)** | Azure resource management | WSL/Windows |
| **Git CLI** | Version control | choco/apt |
---

## 🏛️ Architecture Concepts

### Naming Convention


### Suffix Determinístico
| Método | Fórmula | Propósito |
|--------|---------|----------|
| **MD5 Hash** | `substr(md5(var.name), 0, 4)` | Unicidade global (determinístico) |
| **UUID v5** | `uuidv5("dns", "${scope}-${principal}-${role}")` | Role assignment IDs estáveis |

### Feature Flags System

### RBAC Strategy

---

## 🎨 Design Patterns

### Padrões Utilizados

#### 1. **Feature Flags / Feature Toggle**
```hcl
# Exemplo: criar recurso apenas se flag habilitada
module "storage" {
  count = var.enable_storage ? 1 : 0
  # ...
}
```

#### 2. **Composition Pattern**
```hcl
# Exemplo: Stack Principal orquestra módulos
module "naming" { source = "./modules/foundation/naming" }
module "resource_group" { source = "./modules/foundation/resource-group" }
module "managed_identity" { source = "./modules/security/managed-identity" }

```

#### 3. **Dependency Injection**
```hcl
# Storage recebe IDs de dependências
module "storage" {
  managed_identity_id = module.managed_identity.principal_id
  vnet_subnet_ids = module.vnet.subnet_ids
}
```

#### 4. **Convention over Configuration**
```hcl
# Naming segue padrão CAF Microsoft
# Sem customização necessária
naming = module.naming # utiliza automaticamente
```

#### 5. **Reusable Workflows**
```yaml
# deploy-plan.yaml chama pipeline-core.yaml
jobs:
  validate:
    uses: org/pipeline-as-a-service/.github/workflows/pipeline-core.yaml@main
```
#### 6. **Security by Default (RBAC-First)**
```hcl
# Toda secret acesso via RBAC, nunca shared keys
shared_access_key_enabled = false # Storage
# Todos role assignments = uuidv5() determinístico
name = uuidv5("dns", "${scope}-${principal}-${role}")
```

#### 7. **Time-Based Synchronization**
```hcl
# Aguarda propagação Azure AD antes de criar secrets
resource "time_sleep" "wait_for_rbac_propagation" {
  create_duration = "180s"
  depends_on = [azurerm_role_assignment.main]
}
```

---

## 📊 Stack Summary

### Por Categoria (Total de Tecnologias: 35+)

| Categoria | Count | Exemplos |
|-----------|-------|----------|
| **IaC & Orquestração** | 3 | Terraform, Providers (azurerm, random, time) |
| **Azure Cloud Services** | 11 | Container Apps, Storage, SQL, Service Bus, Event Grid, Key Vault, Log Analytics, ACR, VNet, Managed Identity, App Insights |
| **CI/CD & Automation** | 8 | GitHub Actions, pipeline-core, deploy-plan, deploy-apply, Caching, SARIF upload, State protection |
| **Validation & Security** | 5 | TFLint, Trivy, Checkov, terraform-docs, terraform fmt |
| **Development & IDE** | 6 | VS Code, GitHub Copilot, Docker, Git, PowerShell, WSL2 |
| **Architecture Patterns** | 7 | Feature Flags, Composition, Dependency Injection, Convention over Config, Reusable Workflows, RBAC-First, Time Sync |

---

## 🎓 Conceitos Principais Aprendidos

### Azure
- [ ] Managed Identity (User-Assigned)
- [ ] Virtual Networks + Subnets delegadas
- [ ] Role-Based Access Control (RBAC)
- [ ] Azure AD / Entra ID authentication
- [ ] Service Bus (Standard tier)
- [ ] Event Grid domains
- [ ] SQL Server com AAD admin
- [ ] Container Apps (serverless)
- [ ] Container Registry
- [ ] Key Vault (RBAC enabled)
- [ ] Log Analytics + App Insights
- [ ] Diagnostic Settings

### Terraform
- [ ] Module composition
- [ ] Dynamic blocks
- [ ] Feature flags via booleans
- [ ] Outputs (apenas nomes/FQDNs/URIs, sem resource IDs)
- [ ] Remote state (Azure Blob)
- [ ] Azure AD auth para state
- [ ] uuidv5 para determinismo
- [ ] Provider versioning
- [ ] count vs for_each
- [ ] Depends_on (explícito vs implícito)
- [ ] Local values + base_tags

### GitHub Actions
- [ ] Workflow composition (workflow_call)
- [ ] Job dependencies (needs)
- [ ] Conditional steps (if: always())
- [ ] Outputs entre jobs
- [ ] Secrets + permissions matrix
- [ ] Caching strategies
- [ ] SARIF report upload
- [ ] PR comments via script
- [ ] Step Summary markdown

### Security & Best Practices
- [ ] No shared access keys
- [ ] RBAC + Managed Identity
- [ ] Deterministic naming (MD5)
- [ ] Secrets in Key Vault
- [ ] Time-based synchronization
- [ ] IaC scanning (Trivy, Checkov)
- [ ] Drift detection (terraform-docs)
- [ ] State locking + encryption

---

## 🔗 Repositórios Relacionados

| Repo | Propósito |
|------|----------|
| `platform-as-a-service-stack` | Infraestrutura principal (Terraform) |
| `pipeline-as-a-service-stack` | CI/CD reutilizável (GitHub Actions) |
| `tfmodules-as-a-service-stack` | Módulos Terraform reutilizáveis |
| `workshop-devopspro` | Guia prático + ementa |

---