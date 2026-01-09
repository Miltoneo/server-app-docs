# Estratégia de Migração Multi-Repositório

## 📋 Visão Geral

Este documento descreve o processo de migração da estrutura monorepo atual para uma arquitetura multi-repo, onde cada projeto terá seu próprio repositório independente.

## 🎯 Objetivos

- **Isolamento completo** entre projetos
- **Controle granular** de permissões e acesso
- **Versionamento independente** por projeto
- **Eliminar commits indesejados** em projetos não relacionados
- **Simplificar CI/CD** para cada projeto

## 📊 Estrutura Atual

```
server-app/
├── www/
│   ├── prj_construtora/     → Projeto 1
│   ├── prj_eleicao/         → Projeto 2
│   ├── prj_emprestimos/     → Projeto 3
│   ├── prj_maismedicos/     → Projeto 4
│   ├── prj_medicos/         → Projeto 5
│   ├── prj_milenio/         → Projeto 6
│   ├── prj_rotinas/         → Projeto 7
│   ├── prj_sisu/            → Projeto 8
│   ├── prj_tds/             → Projeto 9
│   └── shared_apps/         → Dependências compartilhadas
├── shared/                  → Utilitários globais
├── infrastructure/          → Infraestrutura comum
└── config/                  → Configurações globais
```

## 🔄 Estrutura Proposta (Multi-Repo)

### Repositórios de Projetos

Cada projeto terá seu próprio repositório:

```
1. server-app-construtora      (prj_construtora)
2. server-app-eleicao          (prj_eleicao)
3. server-app-emprestimos      (prj_emprestimos)
4. server-app-maismedicos      (prj_maismedicos)
5. server-app-medicos          (prj_medicos)
6. server-app-milenio          (prj_milenio)
7. server-app-rotinas          (prj_rotinas)
8. server-app-sisu             (prj_sisu)
9. server-app-tds              (prj_tds)
```

### Repositórios Compartilhados

```
server-app-shared-core/
├── shared_apps/              # Apps Django compartilhados
│   ├── assinaturas/
│   ├── pagamentos/
│   └── business/
├── shared/                   # Utilitários globais
│   ├── middleware/
│   ├── utils/
│   └── templates/
└── setup.py                  # Para instalação como pacote
```

```
server-app-infrastructure/
├── infrastructure/           # Configs de infraestrutura
│   ├── nginx/
│   ├── gunicorn/
│   ├── databases/
│   ├── systemd/
│   └── scripts/
├── config/                   # Configurações ambiente
└── docs/                     # Documentação
```

## 📦 Dependências Identificadas

### Projetos que usam `shared_apps`:
- ✅ **prj_construtora**: usa `shared_apps.assinaturas`, `shared_apps.pagamentos`
- ⚠️ **Outros projetos**: verificar dependências individuais

### Projetos que usam `shared`:
- A verificar individualmente

## 🚀 Processo de Migração

### Fase 1: Preparação (Pré-Migração)

#### 1.1 Criar Repositório Shared-Core
```bash
# Extrair shared_apps e shared mantendo histórico
git filter-repo --path www/shared_apps/ --path shared/ --path-rename www/shared_apps/:shared_apps/

# Converter para pacote Python instalável
# Adicionar setup.py, pyproject.toml, README.md
```

#### 1.2 Criar Repositório Infrastructure
```bash
# Extrair infraestrutura mantendo histórico
git filter-repo --path infrastructure/ --path config/ --path docs/
```

### Fase 2: Extração de Projetos

Para cada projeto `prj_*`:

#### 2.1 Extrair Projeto com Histórico Git
```bash
# Script: extract_project.ps1 <nome_projeto>
# Exemplo: .\extract_project.ps1 prj_construtora

git clone f:/projects/server-app server-app-construtora
cd server-app-construtora
git filter-repo --path www/prj_construtora/ --path-rename www/prj_construtora/:app/
```

#### 2.2 Configurar Dependências
```bash
# requirements.txt adicionar:
server-app-shared-core @ git+https://github.com/org/server-app-shared-core.git@v1.0.0
```

#### 2.3 Ajustar Imports
```python
# Antes:
from shared_apps.assinaturas.models import AssinaturaConstrutora

# Depois:
from server_app_shared.assinaturas.models import AssinaturaConstrutora
```

#### 2.4 Configurar CI/CD Individual
```yaml
# .github/workflows/ci.yml
name: CI/CD - Construtora
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
```

### Fase 3: Validação e Testes

#### 3.1 Checklist por Projeto
- [ ] Histórico Git preservado
- [ ] Imports ajustados
- [ ] Dependências configuradas
- [ ] Testes passando
- [ ] CI/CD funcionando
- [ ] Deploy testado

#### 3.2 Testes de Integração
```bash
# Verificar que shared-core funciona em todos os projetos
pip install server-app-shared-core
python manage.py test
```

### Fase 4: Migração Gradual

#### Estratégia de Rollout

**Opção A: Big Bang** (Não recomendado)
- Migrar todos os projetos de uma vez
- Risco alto, rollback difícil

**Opção B: Incremental** ⭐ (Recomendado)
```
Semana 1: shared-core + infrastructure
Semana 2: prj_construtora (projeto piloto)
Semana 3: prj_eleicao, prj_emprestimos
Semana 4: prj_maismedicos, prj_medicos
Semana 5: prj_milenio, prj_rotinas
Semana 6: prj_sisu, prj_tds
```

#### Convivência Monorepo + Multi-Repo
Durante a transição:
1. Manter monorepo original como **read-only**
2. Novos desenvolvimentos nos novos repos
3. Bug fixes críticos no monorepo, migrar commits

## 🔧 Scripts de Automação

### Script 1: Extração de Projeto
Local: `infrastructure/scripts/migration/extract_project.ps1`

### Script 2: Conversão de Imports
Local: `infrastructure/scripts/migration/convert_imports.py`

### Script 3: Validação Pós-Migração
Local: `infrastructure/scripts/migration/validate_migration.ps1`

### Script 4: Setup Novo Repositório
Local: `infrastructure/scripts/migration/setup_new_repo.ps1`

## 📋 Estrutura de Cada Novo Repositório

```
server-app-<projeto>/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy-staging.yml
│       └── deploy-production.yml
├── app/                      # Código do projeto
│   ├── models/
│   ├── views/
│   ├── templates/
│   └── ...
├── tests/
├── docs/
├── .gitignore
├── .env.example
├── requirements.txt
├── requirements-dev.txt
├── manage.py
├── settings.py
├── README.md
└── docker-compose.yml
```

## ⚙️ Configurações Git

### Proteções de Branch
```yaml
# Settings > Branches > Branch protection rules
main:
  - Require pull request reviews (1 approver)
  - Require status checks to pass
  - Require conversation resolution
  - Include administrators

develop:
  - Require pull request reviews (1 approver)
  - Require status checks to pass
```

### Permissões
```
Repositório               | Equipe        | Permissão
--------------------------|---------------|----------
server-app-construtora    | Team-Const    | Admin
server-app-eleicao        | Team-Eleicao  | Admin
server-app-shared-core    | Tech-Leads    | Admin
server-app-infrastructure | DevOps        | Admin
```

## 🎯 Benefícios Esperados

### Técnicos
- ✅ Isolamento total de código
- ✅ CI/CD mais rápido (só testa o que mudou)
- ✅ Versionamento independente
- ✅ Deploys independentes
- ✅ Menor acoplamento

### Organizacionais
- ✅ Controle de acesso granular
- ✅ Equipes trabalham independentemente
- ✅ Elimina commits acidentais
- ✅ Code reviews mais focados
- ✅ Histórico Git mais limpo

### Desafios
- ⚠️ Gerenciamento de múltiplos repos
- ⚠️ Sincronização de shared-core
- ⚠️ Complexidade inicial maior
- ⚠️ Necessário tooling adicional

## 📚 Referências e Recursos

### Ferramentas
- **git-filter-repo**: https://github.com/newren/git-filter-repo
- **git-subtree**: Para manter histórico
- **GitHub Actions**: CI/CD por projeto

### Boas Práticas
- Semantic Versioning para shared-core
- Conventional Commits
- GitFlow ou GitHub Flow
- Documentação em cada repo

## 📅 Timeline Estimado

```
Preparação:        2 semanas
Shared-Core:       1 semana
Infraestrutura:    1 semana
Projeto Piloto:    2 semanas
Demais Projetos:   4 semanas
Testes Finais:     1 semana
-----------------------------------
Total:            11 semanas (~3 meses)
```

## ✅ Critérios de Sucesso

- [ ] Todos os 9 projetos em repositórios separados
- [ ] shared-core funcionando como dependência
- [ ] CI/CD operacional em todos os repos
- [ ] Deploys independentes funcionando
- [ ] Zero commits no monorepo antigo (arquivado)
- [ ] Documentação completa
- [ ] Equipe treinada

## 🔄 Próximos Passos

1. ✅ Revisar e aprovar esta documentação
2. ⏭️ Executar scripts de extração
3. ⏭️ Criar repositório shared-core
4. ⏭️ Migrar projeto piloto (prj_construtora)
5. ⏭️ Validar e ajustar processo
6. ⏭️ Migrar demais projetos

## 📞 Contatos e Suporte

- Tech Lead: [Nome]
- DevOps: [Nome]
- Repositórios: https://github.com/org/

---

**Última Atualização:** 05/01/2026  
**Status:** 📝 Em Planejamento  
**Responsável:** DevOps Team
