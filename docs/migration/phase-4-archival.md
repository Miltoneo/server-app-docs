# FASE 4: Arquivamento do Monorepo

**Período Estimado:** 1 semana  
**Data de Início:** A definir  
**Responsável:** DevOps Team  
**Status:** 📝 Planejamento

---

## 🎯 Objetivos

Arquivar o repositório monorepo original (`server-app`) de forma segura e organizada após a conclusão bem-sucedida da migração multi-repo, garantindo:

- ✅ Todos os projetos operacionais nos novos repositórios
- ✅ Zero dependências do monorepo original
- ✅ Histórico preservado e acessível
- ✅ Documentação clara sobre a migração
- ✅ Equipe informada e treinada

---

## 📋 Pré-Requisitos Obrigatórios

### ✅ Checklist de Validação (CRÍTICO - NÃO ARQUIVAR SEM CONCLUIR)

#### 1. Repositórios Novos Operacionais
- [ ] **server-app-construtora**: Deploy em produção e funcionando
- [ ] **server-app-medicos**: Deploy em produção e funcionando
- [ ] **server-app-emprestimos**: Deploy em produção e funcionando
- [ ] **server-app-sisu**: Deploy em produção e funcionando
- [ ] **server-app-maismedicos**: Deploy em produção e funcionando
- [ ] **server-app-rotinas**: Deploy em produção e funcionando
- [ ] **server-app-eleicao**: Deploy em produção e funcionando
- [ ] **server-app-tds**: Deploy em produção e funcionando
- [ ] **Resolução do prj_milenio**: Decisão tomada (manter/remover/renomear)

#### 2. Dependências Compartilhadas
- [ ] **server-app-shared**: Publicado e instalável
- [ ] **server-app-infrastructure**: Documentação completa
- [ ] Todos os projetos usando `server-app-shared` (não o monorepo)

#### 3. CI/CD e Automação
- [ ] GitHub Actions configurado em todos os projetos
- [ ] Testes automatizados rodando
- [ ] Deploys independentes funcionando (staging + production)
- [ ] Rollback testado em pelo menos 2 projetos

#### 4. Dados e Segurança
- [ ] Backup completo do monorepo original (código + issues + PRs + wikis)
- [ ] Secrets migrados para novos repositórios
- [ ] Permissões de acesso configuradas nos novos repos
- [ ] Sem dados sensíveis expostos (senhas, tokens, etc.)

#### 5. Documentação
- [ ] README.md atualizado em cada novo repositório
- [ ] Guias de setup local testados por outro membro da equipe
- [ ] Documentação de APIs (se aplicável)
- [ ] CHANGELOG.md com histórico de versões

#### 6. Equipe e Comunicação
- [ ] Equipe informada sobre o arquivamento (com 2 semanas de antecedência)
- [ ] Treinamento em novos workflows concluído
- [ ] Issues e PRs abertos migrados/fechados
- [ ] Links antigos documentados → novos repositórios

---

## 📝 Plano de Execução

### Semana 1: Preparação e Validação

#### Dia 1: Auditoria Completa
```powershell
# Script: audit_migration.ps1
# Valida estado de todos os repositórios novos

$repos = @(
    "server-app-construtora",
    "server-app-medicos",
    "server-app-emprestimos",
    "server-app-sisu",
    "server-app-maismedicos",
    "server-app-rotinas",
    "server-app-eleicao",
    "server-app-tds"
)

foreach ($repo in $repos) {
    Write-Host "=== Validando $repo ===" -ForegroundColor Cyan
    
    cd "f:\projects\$repo"
    
    # Check se branch main/master existe
    git branch -a | Select-String -Pattern "(main|master)"
    
    # Check último commit
    Write-Host "Último commit:" -ForegroundColor Yellow
    git log -1 --oneline
    
    # Check se tem CI/CD configurado
    if (Test-Path ".github\workflows") {
        Write-Host "✅ CI/CD configurado" -ForegroundColor Green
    } else {
        Write-Host "❌ CI/CD NÃO configurado" -ForegroundColor Red
    }
    
    # Check se tem README
    if (Test-Path "README.md") {
        Write-Host "✅ README.md existe" -ForegroundColor Green
    } else {
        Write-Host "❌ README.md NÃO existe" -ForegroundColor Red
    }
    
    Write-Host ""
}
```

#### Dia 2: Backup Completo
```powershell
# Script: backup_monorepo.ps1
# Cria backup completo do monorepo antes do arquivamento

$backupDir = "f:\backups\server-app-monorepo-$(Get-Date -Format 'yyyy-MM-dd')"
New-Item -ItemType Directory -Path $backupDir -Force

Write-Host "Criando backup em: $backupDir" -ForegroundColor Cyan

# 1. Clone completo com todo histórico
cd $backupDir
git clone --mirror f:\projects\server-app server-app.git
Write-Host "✅ Git mirror criado" -ForegroundColor Green

# 2. Exportar issues e PRs via GitHub CLI
cd $backupDir
gh issue list --repo Miltoneo/server-app --state all --limit 1000 --json number,title,state,createdAt,closedAt,author,labels > issues.json
gh pr list --repo Miltoneo/server-app --state all --limit 1000 --json number,title,state,createdAt,closedAt,author,labels > pull_requests.json
Write-Host "✅ Issues e PRs exportados" -ForegroundColor Green

# 3. Exportar wikis (se existir)
if (gh api repos/Miltoneo/server-app --jq '.has_wiki') {
    git clone https://github.com/Miltoneo/server-app.wiki.git
    Write-Host "✅ Wiki exportada" -ForegroundColor Green
}

# 4. Criar arquivo de metadados
@"
# Backup Monorepo server-app
Data: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')
Branch: master
Último Commit: $(git -C f:\projects\server-app log -1 --format='%H - %s')
Total de Commits: $(git -C f:\projects\server-app rev-list --count master)
"@ | Out-File "$backupDir\METADATA.txt"

Write-Host "✅ Backup completo criado em: $backupDir" -ForegroundColor Green
```

#### Dia 3: Migração de Issues/PRs
```powershell
# Script: migrate_issues.ps1
# Migra issues abertas do monorepo para repositórios específicos

$issueMapping = @{
    "construtora" = "Miltoneo/server-app-construtora"
    "medicos" = "Miltoneo/server-app-medicos"
    "emprestimos" = "Miltoneo/server-app-emprestimos"
    # ... adicionar outros
}

# Listar issues abertas no monorepo
$openIssues = gh issue list --repo Miltoneo/server-app --state open --json number,title,body,labels

foreach ($issue in $openIssues) {
    Write-Host "Processando Issue #$($issue.number): $($issue.title)"
    
    # Identificar projeto pela label ou título
    $targetRepo = $null
    foreach ($key in $issueMapping.Keys) {
        if ($issue.title -match $key -or $issue.labels -match $key) {
            $targetRepo = $issueMapping[$key]
            break
        }
    }
    
    if ($targetRepo) {
        # Criar issue no novo repositório
        $newBody = @"
**[Migrado do monorepo server-app#$($issue.number)]**

$($issue.body)

---
_Issue original: https://github.com/Miltoneo/server-app/issues/$($issue.number)_
"@
        
        gh issue create --repo $targetRepo --title $issue.title --body $newBody
        
        # Comentar na issue original
        gh issue comment $issue.number --repo Miltoneo/server-app --body "Issue migrada para $targetRepo"
        
        # Fechar issue original
        gh issue close $issue.number --repo Miltoneo/server-app
        
        Write-Host "✅ Migrada para $targetRepo" -ForegroundColor Green
    } else {
        Write-Host "⚠️ Projeto não identificado - requer ação manual" -ForegroundColor Yellow
    }
}
```

#### Dia 4: Atualização de Documentação
```markdown
# Criar README.md no monorepo avisando sobre arquivamento

# ⚠️ REPOSITÓRIO ARQUIVADO

Este repositório foi **arquivado** e não está mais em uso ativo.

## 🔄 Migração Multi-Repo Concluída

Em **[Data]**, concluímos a migração do monorepo para uma arquitetura multi-repositório.
Cada projeto agora possui seu próprio repositório independente.

## 🗂️ Novos Repositórios

Todos os projetos foram migrados para os seguintes repositórios:

### Projetos Principais
- **Construtora**: [server-app-construtora](https://github.com/Miltoneo/server-app-construtora)
- **Medicos**: [server-app-medicos](https://github.com/Miltoneo/server-app-medicos)
- **Emprestimos**: [server-app-emprestimos](https://github.com/Miltoneo/server-app-emprestimos)
- **SISU**: [server-app-sisu](https://github.com/Miltoneo/server-app-sisu)
- **Mais Médicos**: [server-app-maismedicos](https://github.com/Miltoneo/server-app-maismedicos)
- **Rotinas**: [server-app-rotinas](https://github.com/Miltoneo/server-app-rotinas)
- **Eleição**: [server-app-eleicao](https://github.com/Miltoneo/server-app-eleicao)
- **TDS (IoT)**: [server-app-tds](https://github.com/Miltoneo/server-app-tds)

### Repositórios Compartilhados
- **Shared**: [server-app-shared](https://github.com/Miltoneo/server-app-shared)
- **Infrastructure**: [server-app-infrastructure](https://github.com/Miltoneo/server-app-infrastructure)

## 📚 Histórico Preservado

Todo o histórico de commits, issues e pull requests foi preservado:
- **Backup completo**: Disponível em `f:\backups\server-app-monorepo-[data]`
- **Git mirror**: Clone bare com todo histórico
- **Issues/PRs**: Exportados em JSON

## ❓ Precisa de Acesso ao Código Antigo?

Se você precisa acessar código antigo ou histórico de commits:

1. **Clone o monorepo arquivado**:
   ```bash
   git clone https://github.com/Miltoneo/server-app
   cd server-app
   git checkout master
   ```

2. **Navegue pelo histórico**:
   ```bash
   git log --all --graph --oneline
   ```

3. **Ou consulte o projeto específico no novo repositório**

## 🔗 Links Úteis

- [Documentação da Migração](./MIGRACAO_MULTI_REPO.md)
- [Status da Migração](./STATUS_MIGRACAO_MULTIREPO.md)
- [Relatório da Fase 0](./FASE0_DIAS3-4_CONCLUSAO.md)

## 📞 Contato

Para dúvidas sobre a migração ou acesso aos novos repositórios, entre em contato com:
- Tech Lead: [Nome]
- DevOps: [Nome]

---

**Data de Arquivamento:** [Data]  
**Última Atualização do Monorepo:** $(Get-Date -Format 'dd/MM/yyyy')
```

#### Dia 5: Comunicação à Equipe
```markdown
# Template de Email/Slack para Equipe

Assunto: 🔔 IMPORTANTE: Arquivamento do Monorepo server-app

Olá time,

Após **[X meses]** de trabalho na migração multi-repo, estamos prontos para arquivar
o repositório monorepo original `server-app`.

**📅 Data Planejada de Arquivamento:** [Data] às [Hora]

**⏰ O que acontece no arquivamento:**
1. O repositório `server-app` será marcado como **read-only** no GitHub
2. Não será mais possível fazer commits, PRs ou abrir issues
3. Todo código permanecerá acessível para consulta
4. Backup completo foi criado e armazenado

**✅ Onde trabalhar a partir de agora:**
- Construtora → https://github.com/Miltoneo/server-app-construtora
- Medicos → https://github.com/Miltoneo/server-app-medicos
- Emprestimos → https://github.com/Miltoneo/server-app-emprestimos
- SISU → https://github.com/Miltoneo/server-app-sisu
- Mais Médicos → https://github.com/Miltoneo/server-app-maismedicos
- Rotinas → https://github.com/Miltoneo/server-app-rotinas
- Eleição → https://github.com/Miltoneo/server-app-eleicao
- TDS → https://github.com/Miltoneo/server-app-tds

**📚 Documentação:**
- Cada repositório tem um README.md completo com instruções de setup
- Guia de desenvolvimento disponível na wiki de cada projeto

**🆘 Precisa de Ajuda?**
- Slack: #dev-migration
- Email: devops@empresa.com

**⚠️ Ação Necessária:**
1. Atualize seus clones locais para os novos repositórios
2. Atualize bookmarks/favoritos para os novos URLs
3. Reporte qualquer problema antes do dia [Data]

Obrigado pela colaboração durante a migração! 🚀

---
DevOps Team
```

---

## 🚀 Execução do Arquivamento

### Dia 6: Arquivamento Técnico

#### Passo 1: Atualizar README.md no Monorepo
```powershell
cd f:\projects\server-app

# Substituir README.md pelo template de arquivamento (criado no Dia 4)
# Commit final
git add README.md
git commit -m "docs: Archive repository - Multi-repo migration completed"
git push origin master
```

#### Passo 2: Arquivar no GitHub
```powershell
# Via GitHub CLI
gh repo archive Miltoneo/server-app --confirm

# Ou via web:
# 1. Acessar https://github.com/Miltoneo/server-app/settings
# 2. Scroll até "Danger Zone"
# 3. Clicar em "Archive this repository"
# 4. Confirmar digitando o nome do repositório
```

#### Passo 3: Adicionar Topics/Tags
```powershell
# Adicionar tags para facilitar localização
gh repo edit Miltoneo/server-app --add-topic "archived"
gh repo edit Miltoneo/server-app --add-topic "monorepo"
gh repo edit Miltoneo/server-app --add-topic "django"
gh repo edit Miltoneo/server-app --add-topic "legacy"

# Atualizar descrição
gh repo edit Miltoneo/server-app --description "⚠️ ARCHIVED - Migrated to multi-repo architecture. See individual repos."
```

#### Passo 4: Criar Release Final
```powershell
# Criar tag final
cd f:\projects\server-app
git tag -a v-final -m "Final version before archiving - Multi-repo migration completed"
git push origin v-final

# Criar release no GitHub
gh release create v-final `
    --title "🏁 Final Release - Repository Archived" `
    --notes @"
# Final Release - Multi-Repo Migration Completed

This is the final release of the monorepo before archiving.

## 📦 Projects Migrated

All projects have been successfully migrated to individual repositories:
- server-app-construtora
- server-app-medicos
- server-app-emprestimos
- server-app-sisu
- server-app-maismedicos
- server-app-rotinas
- server-app-eleicao
- server-app-tds

## 📚 Documentation

- [Migration Plan](./MIGRACAO_MULTI_REPO.md)
- [Migration Status](./STATUS_MIGRACAO_MULTIREPO.md)
- [New Repositories](./README.md#novos-repositórios)

## 🔗 Links

- Shared Package: https://github.com/Miltoneo/server-app-shared
- Infrastructure: https://github.com/Miltoneo/server-app-infrastructure

Thank you for being part of this journey! 🚀
"@
```

---

## 🔄 Pós-Arquivamento

### Dia 7: Validação e Monitoramento

#### Checklist Final
- [ ] Repositório marcado como archived no GitHub
- [ ] README.md do monorepo atualizado com links para novos repos
- [ ] Release final criada e tagueada
- [ ] Backup validado e acessível
- [ ] Equipe notificada
- [ ] Issues/PRs migrados ou fechados
- [ ] Documentação atualizada em todos os novos repos
- [ ] Clones locais da equipe atualizados

#### Monitoramento (Primeiras 2 Semanas)
```markdown
## Métricas a Observar

### Semana 1 Pós-Arquivamento
- [ ] Quantidade de tentativas de commit no monorepo arquivado
- [ ] Issues abertas nos novos repositórios
- [ ] Dúvidas da equipe (Slack/Email)
- [ ] Tempo de deploy nos novos repos
- [ ] Incidentes em produção

### Semana 2 Pós-Arquivamento
- [ ] CI/CD rodando sem problemas
- [ ] Todos os membros da equipe usando novos repos
- [ ] Zero referências ao monorepo em PRs novos
- [ ] Feedback da equipe coletado
```

#### Script de Validação Pós-Arquivamento
```powershell
# Script: validate_post_archive.ps1

Write-Host "=== Validação Pós-Arquivamento ===" -ForegroundColor Cyan

# 1. Verificar que monorepo está read-only
$repoInfo = gh repo view Miltoneo/server-app --json isArchived,description
if ($repoInfo.isArchived) {
    Write-Host "✅ Repositório está arquivado" -ForegroundColor Green
} else {
    Write-Host "❌ ERRO: Repositório NÃO está arquivado" -ForegroundColor Red
}

# 2. Verificar que novos repos estão ativos
$newRepos = @(
    "server-app-construtora",
    "server-app-medicos",
    "server-app-emprestimos",
    "server-app-sisu",
    "server-app-maismedicos",
    "server-app-rotinas",
    "server-app-eleicao",
    "server-app-tds"
)

foreach ($repo in $newRepos) {
    $info = gh repo view "Miltoneo/$repo" --json isArchived
    if (-not $info.isArchived) {
        Write-Host "✅ $repo está ativo" -ForegroundColor Green
    } else {
        Write-Host "❌ $repo está ARQUIVADO (ERRO)" -ForegroundColor Red
    }
}

# 3. Verificar CI/CD nos novos repos
foreach ($repo in $newRepos) {
    $workflows = gh workflow list --repo "Miltoneo/$repo" --json name,state
    if ($workflows.Count -gt 0) {
        Write-Host "✅ $repo tem CI/CD configurado" -ForegroundColor Green
    } else {
        Write-Host "⚠️ $repo NÃO tem workflows" -ForegroundColor Yellow
    }
}

# 4. Verificar backup existe
$backupPath = "f:\backups\server-app-monorepo-*"
if (Test-Path $backupPath) {
    Write-Host "✅ Backup do monorepo encontrado" -ForegroundColor Green
} else {
    Write-Host "❌ CRÍTICO: Backup NÃO encontrado" -ForegroundColor Red
}

Write-Host "`n=== Validação Concluída ===" -ForegroundColor Cyan
```

---

## 📊 Métricas de Sucesso

### KPIs da Fase 4

| Métrica | Meta | Verificação |
|---------|------|-------------|
| **Projetos Migrados** | 8/8 (100%) | ✅ Todos operacionais |
| **Monorepo Arquivado** | Sim | ✅ Read-only no GitHub |
| **Backup Criado** | Sim | ✅ Mirror + Issues/PRs |
| **CI/CD Configurado** | 8/8 repos | Verificar por repo |
| **Equipe Treinada** | 100% | Survey pós-migração |
| **Issues Migradas** | 100% abertas | Script automatizado |
| **Zero Downtime** | 0 min | Monitorar deploys |
| **Rollback Testado** | ≥2 projetos | Testar staging |

---

## 🚨 Plano de Contingência

### Cenário 1: Projeto Crítico com Bug Pós-Arquivamento

**Sintoma:** Bug crítico descoberto que requer código do monorepo

**Solução:**
1. **NÃO desarquivar** o monorepo
2. Acessar backup local: `f:\backups\server-app-monorepo-[data]`
3. Identificar commit/código relevante
4. Aplicar fix no novo repositório
5. Documentar no CHANGELOG.md

### Cenário 2: Descoberta de Dependência Não Migrada

**Sintoma:** Projeto novo quebrando por falta de arquivo/módulo do monorepo

**Solução:**
1. Identificar arquivo/módulo faltante no backup
2. Avaliar se deve ir para `server-app-shared` ou no projeto específico
3. Copiar e adaptar no destino correto
4. Atualizar requirements.txt se necessário
5. Commit com mensagem: `fix: add missing [module] from monorepo`

### Cenário 3: Necessidade de Desarquivar Temporariamente

**Procedimento de Emergência:**
```powershell
# 1. Desarquivar no GitHub (via web)
# Settings > Danger Zone > Unarchive

# 2. Fazer correção necessária
cd f:\projects\server-app
git checkout -b hotfix/emergency-fix
# ... fazer alterações ...
git commit -m "hotfix: [descrição]"
git push origin hotfix/emergency-fix

# 3. Copiar correção para repo(s) novo(s)
# ... copiar código ...

# 4. Arquivar novamente
gh repo archive Miltoneo/server-app --confirm
```

**⚠️ IMPORTANTE:** Desarquivar deve ser última opção e requer aprovação do Tech Lead.

---

## 📚 Documentação Gerada

### Arquivos Criados Durante Fase 4

```
server-app/
├── FASE4_ARQUIVAMENTO_MONOREPO.md       ← Este documento
├── README.md (atualizado)                ← Com aviso de arquivamento
├── MIGRACAO_CONCLUSAO.md                 ← Relatório final da migração
└── scripts/
    └── phase4/
        ├── audit_migration.ps1
        ├── backup_monorepo.ps1
        ├── migrate_issues.ps1
        └── validate_post_archive.ps1

backups/
└── server-app-monorepo-YYYY-MM-DD/
    ├── server-app.git/                   ← Git mirror completo
    ├── issues.json                       ← Todas issues exportadas
    ├── pull_requests.json                ← Todos PRs exportados
    ├── server-app.wiki/                  ← Wiki (se houver)
    └── METADATA.txt                      ← Info do backup
```

---

## ✅ Critérios de Conclusão da Fase 4

- [ ] ✅ Todos os 8 projetos operacionais em produção (sem monorepo)
- [ ] ✅ Backup completo do monorepo criado e validado
- [ ] ✅ Issues abertas migradas para repositórios específicos
- [ ] ✅ README.md do monorepo atualizado com links para novos repos
- [ ] ✅ Repositório arquivado no GitHub (read-only)
- [ ] ✅ Release final criada e tagueada (v-final)
- [ ] ✅ Equipe notificada com 2 semanas de antecedência
- [ ] ✅ CI/CD rodando em todos os novos repositórios
- [ ] ✅ Documentação completa em cada novo repo
- [ ] ✅ Monitoramento pós-arquivamento por 2 semanas
- [ ] ✅ Zero referências ao monorepo em código novo
- [ ] ✅ Plano de contingência testado (rollback)

---

## 🎉 Marcos da Migração Multi-Repo

```
┌─────────────────────────────────────────────────────────────┐
│  FASE 0: Preparação                         [Concluída ✅]  │
│  ├─ server-app-shared criado                                │
│  └─ server-app-infrastructure criado                        │
├─────────────────────────────────────────────────────────────┤
│  FASE 1: Extração de Projetos              [Concluída ✅]  │
│  ├─ 8 projetos extraídos e migrados                        │
│  └─ Histórico Git preservado                               │
├─────────────────────────────────────────────────────────────┤
│  FASE 2: CI/CD e Automação                 [Em Progresso]  │
│  ├─ GitHub Actions configurado                             │
│  └─ Deploys independentes                                  │
├─────────────────────────────────────────────────────────────┤
│  FASE 3: Otimização                       [Planejada]       │
│  ├─ Padronização de código                                 │
│  └─ Performance e segurança                                │
├─────────────────────────────────────────────────────────────┤
│  FASE 4: Arquivamento (VOCÊ ESTÁ AQUI)    [Planejada]      │
│  ├─ Validação final                                        │
│  ├─ Backup completo                                        │
│  ├─ Migração de issues                                     │
│  └─ Arquivamento no GitHub                                │
└─────────────────────────────────────────────────────────────┘
```

---

## 📞 Suporte e Contatos

### Responsáveis pela Fase 4
- **Tech Lead:** [Nome] - [email]
- **DevOps:** [Nome] - [email]
- **QA Lead:** [Nome] - [email]

### Canais de Comunicação
- **Slack:** #dev-migration
- **Email:** devops@empresa.com
- **Meeting:** Toda segunda-feira às 10h (status update)

### Escalação de Problemas
1. Nível 1: Slack #dev-migration
2. Nível 2: Email devops@empresa.com
3. Nível 3: Tech Lead direto
4. Nível 4: CTO (apenas emergências)

---

## 📅 Timeline Detalhado

```
Semana de Arquivamento:

Segunda    | Terça     | Quarta    | Quinta    | Sexta     | Sábado   | Domingo
-----------|-----------|-----------|-----------|-----------|----------|--------
Dia 1      | Dia 2     | Dia 3     | Dia 4     | Dia 5     | Dia 6    | Dia 7
Auditoria  | Backup    | Migração  | Docs      | Comunicar | Arquivar | Validar
Completa   | Completo  | Issues    | Atualizar | Equipe    | GitHub   | Final
-----------|-----------|-----------|-----------|-----------|----------|--------
  ✓ Audit  |  ✓ Mirror |  ✓ Issues |  ✓ README |  ✓ Email  |  ✓ Tag   |  ✓ Check
  ✓ Report |  ✓ Export |  ✓ PRs    |  ✓ Release|  ✓ Slack  |  ✓ Archive| ✓ Monitor
```

---

**Última Atualização:** 09/01/2026  
**Próxima Revisão:** Antes de iniciar execução  
**Aprovação Necessária:** Tech Lead + CTO

---

## 🔗 Links Relacionados

- [Plano de Migração Completo](./MIGRACAO_MULTI_REPO.md)
- [Status Atual da Migração](./STATUS_MIGRACAO_MULTIREPO.md)
- [Fase 0: Preparação](./FASE0_DIAS3-4_CONCLUSAO.md)
- [Server App Shared](https://github.com/Miltoneo/server-app-shared)
- [Server App Infrastructure](https://github.com/Miltoneo/server-app-infrastructure)
