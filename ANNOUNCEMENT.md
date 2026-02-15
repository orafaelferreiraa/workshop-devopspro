# 🚀 Workshop DevOps Pro - Platform Engineering

Workshop completo de **Engenharia de Plataforma** com **Azure**, **Terraform** e **GitHub Actions**!

---

## 🎯 O que você vai aprender?

### Transformar Infraestrutura em Código
Neste workshop, você aprenderá a construir uma **plataforma Azure profissional do zero**, combinando:

✅ **Infrastructure as Code (IaC)** com Terraform  
✅ **CI/CD Automatizado** com GitHub Actions  
✅ **Segurança em Primeiro Lugar** (RBAC, Managed Identity, sem senhas compartilhadas)  
✅ **Engenharia de Plataforma** (feature flags, composição de módulos, dependências determinísticas)  
✅ **Vibe Coding** com GitHub Copilot (sua IA codeando junto!)

---

## 🏗️ A Plataforma que você vai construir

```
┌─────────────────────────────────────────────────────────────┐
│         PLATFORM AS A SERVICE STACK                         │
│                                                             │
│  Dev → GitHub Actions → Terraform → Azure Resources         │
└─────────────────────────────────────────────────────────────┘
```

### Recursos Implementados ✨

| Camada | Recursos | Feature Flag |
|--------|----------|--------------|
| **🔐 Segurança** | Managed Identity, Key Vault (RBAC) | `enable_managed_identity`, `enable_key_vault` |
| **🌐 Rede** | VNet Spoke, Subnets, NSG | `enable_vnet` |
| **💾 Dados** | Storage Account (AAD only), SQL Server | `enable_storage`, `enable_sql` |
| **📨 Integração** | Service Bus, Event Grid | `enable_service_bus`, `enable_event_grid` |
| **📊 Observação** | Log Analytics, App Insights | `enable_observability` |
| **📦 Computação** | Container Registry, Container Apps | `enable_container_registry`, `enable_container_apps` |

---

## 🎓 Conteúdo Programático

#### Fundamentos & Setup
- ✅ O que é Engenharia de Plataforma?
- ✅ Arquitetura da solução (layers, dependências)
- ✅ Setup do ambiente (VM Windows 11, Git, VS Code, Copilot)
- ✅ Autenticação Azure (Service Principal, RBAC, Secrets GitHub)
- ✅ Criação do Terraform State Remoto (Azure Blob Storage)
- ✅ Estrutura modular Terraform (foundation, networking, security, workloads)

#### Hands-on + Deploy Real
- ✅ Naming convention determinístico (MD5 sufixos)
- ✅ Feature flags system (enable_*)
- ✅ Dependency injection entre módulos
- ✅ RBAC automático com `uuidv5()`
- ✅ **LIVE CODING**: Deploy completo com GitHub Actions
  - Validação (TFLint, tfsec, Checkov)
  - Terraform plan + apply
  - State protection
- ✅ RBAC-first strategy (RBAC-first strategy, sem shared keys)
- ✅ Segurança & Best Practices (Key Vault, Secrets, Diagnostic Settings)

#### 🏆 Entrega Final
- ✅ Plataforma Azure 100% provisionada
- ✅ CI/CD pronto para produção

---

## 👥 Para quem é este workshop?

### ✅ Você é um bom fit se:
- Trabalha com infraestrutura (DevOps, Cloud Engineer, SRE)
- Quer aprender Terraform profissionalmente
- Está migrando para CI/CD moderno (GitHub Actions)
- Busca implementar "platform engineering" na sua empresa
- Gosta de usar IA (GitHub Copilot) no dia a dia

### 📚 Pré-requisitos (Mínimos):
- Vontade de aprender! 🚀

---

## 📊 Stack Tecnológico

**35+ Tecnologias** cobertas, incluindo:

| Categoria | Tecnologias |
|-----------|------------|
| **IaC** | Terraform 1.9.0+, Providers (azurerm, random, time) |
| **Cloud** | Azure (11 serviços), Managed Identity, RBAC, AD Auth |
| **CI/CD** | GitHub Actions, Reusable Workflows, SARIF Reports |
| **Segurança** | TFLint, tfsec, Checkov, terraform fmt, terraform-docs |
| **IDE** | VS Code, GitHub Copilot, Docker, WSL2 |
| **Patterns** | Feature Flags, Composition, Dependency Injection, RBAC-First |

---

## 🎁 O que você leva?

Ao final do workshop, você receberá:

1. ✅ **Codebase Completo** - Todos os módulos Terraform prontos para produção
2. ✅ **Documentação** - README auto-gerado, Architecture diagrams
3. ✅ **CI/CD Workflows** - Pronto para usar em seus projetos
4. ✅ **Checklists** - Best practices + troubleshooting guide
---

