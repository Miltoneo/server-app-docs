# FASE 0: Preparação - Dias 3-4 Concluídos

## 📦 Estrutura de Pacote Python Criada

### ✅ Estrutura de Diretórios

```
server-app/
├── src/shared/                    # Pacote Python principal
│   ├── __init__.py                # Versão 0.1.0, exports públicos
│   ├── assinaturas/               # Módulo de assinaturas
│   │   └── __init__.py
│   ├── business/                  # Módulo de regras de negócio
│   │   ├── __init__.py
│   │   └── base.py               # ServicoNegocio (migrado)
│   └── pagamentos/                # Módulo de pagamentos
│       └── __init__.py
├── setup.py                       # Configuração setuptools
├── pyproject.toml                 # Configuração moderna (PEP 517/518)
├── MANIFEST.in                    # Arquivos a incluir no pacote
├── README.md                      # Documentação completa
├── .gitignore                     # Ignora __pycache__, dist/, etc.
├── migrate_code.py                # Script para copiar código restante
└── migrate_imports.py             # Script para converter imports
```

### ✅ Arquivos de Empacotamento

#### 1. **setup.py**
- Nome do pacote: `server-app-shared`
- Versão: `0.1.0`
- Python: `>=3.11`
- Dependências: Django 4.2, DRF, psycopg2, redis, celery, mercadopago
- Dev dependencies: pytest, black, flake8, mypy

#### 2. **pyproject.toml**
- Build system: setuptools >=68.0
- PEP 517/518 compliant
- Configurações: Black (line-length: 88), pytest, mypy
- Optional dependencies para ambiente dev

#### 3. **MANIFEST.in**
- Inclui README.md, LICENSE
- Inclui todos os .py do src/shared/
- Exclui __pycache__, *.pyc, .DS_Store

#### 4. **README.md**
- Guia completo de instalação (dev e produção)
- Exemplos de uso (antes/depois dos imports)
- Estrutura de diretórios
- Convenções de desenvolvimento
- Testes e versionamento

### ✅ Módulos Criados

#### `src/shared/__init__.py`
```python
__version__ = "0.1.0"
__author__ = "Onkoto Team"
__all__ = ["assinaturas", "business", "pagamentos"]
```

#### `src/shared/business/base.py`
- Classe `ServicoNegocio` (ABC)
- 184 linhas migradas de `shared_apps/business/base.py`
- Funcionalidades: multi-tenant, auditoria, transações atômicas

### 📜 Scripts de Migração

#### 1. **migrate_code.py**
Copia código de `www/shared_apps/` → `src/shared/`:
- Mantém estrutura de diretórios
- Ignora __pycache__, .pyc
- Exibe progresso detalhado
- Pula arquivos já migrados manualmente (base.py)

**Uso**:
```bash
python migrate_code.py
```

#### 2. **migrate_imports.py**
Converte imports em todos os 9 projetos:
- **Antes**: `from shared_apps.pagamentos import ...`
- **Depois**: `from shared.pagamentos import ...`

**Funcionalidades**:
- Modo simulação (dry run) primeiro
- Gera relatório detalhado: `RELATORIO_CONVERSAO_IMPORTS.md`
- Processa todos os .py em prj_* (exceto migrations, venv)
- Confirma antes de aplicar mudanças

**Uso**:
```bash
# 1. Simulação (não modifica arquivos)
python migrate_imports.py

# 2. Revisar relatório gerado
cat RELATORIO_CONVERSAO_IMPORTS.md

# 3. Aplicar (responder 's' quando perguntado)
python migrate_imports.py
```

## 🔄 Próximos Passos

### Dia 5: Conversão de Imports

1. **Copiar código restante** (se ainda não foi feito):
   ```bash
   python migrate_code.py
   ```

2. **Converter imports**:
   ```bash
   python migrate_imports.py
   ```

3. **Instalar pacote em modo dev**:
   ```bash
   pip install -e .
   ```

### Dias 6-7: Testes e Validação

1. **Testar cada projeto individualmente**:
   ```bash
   cd www/prj_construtora
   python manage.py check
   python manage.py test
   ```

2. **Verificar imports**:
   ```bash
   python -c "from shared.business import ServicoNegocio; print('OK')"
   python -c "from shared.pagamentos.services import PagamentoService; print('OK')"
   ```

3. **Executar aplicações em dev**:
   ```bash
   cd www/prj_construtora
   python manage.py runserver
   ```

## 📊 Status Atual

| Tarefa | Status | Observações |
|--------|--------|-------------|
| Estrutura src/shared/ | ✅ | 3 módulos criados |
| setup.py | ✅ | Dependências configuradas |
| pyproject.toml | ✅ | PEP 517/518 |
| README.md | ✅ | Documentação completa |
| migrate_code.py | ✅ | Script pronto |
| migrate_imports.py | ✅ | Script pronto |
| Copiar código restante | ⏳ | Executar script |
| Converter imports | ⏳ | Executar script |
| Instalar pacote | ⏳ | pip install -e . |
| Testes | ⏳ | Dias 6-7 |

## 🎯 Objetivos Atingidos (Dias 3-4)

✅ Estrutura de pacote Python instalável criada  
✅ Arquivos de configuração (setup.py, pyproject.toml, MANIFEST.in)  
✅ Documentação completa (README.md)  
✅ Módulo business.base migrado manualmente  
✅ Scripts automatizados para migração de código e imports  
✅ .gitignore configurado  

## 🚀 Comandos Resumidos

```bash
# 1. Copiar código
python migrate_code.py

# 2. Converter imports (simulação + aplicação)
python migrate_imports.py

# 3. Instalar pacote
pip install -e .

# 4. Testar
python -c "from shared.business import ServicoNegocio; print('OK')"
cd www/prj_construtora && python manage.py check
```

---

**Data**: 6 de Janeiro de 2026  
**Fase**: FASE 0 - Preparação  
**Dias**: 3-4/7 (Estrutura de Pacote)  
**Próximo**: Dia 5 (Conversão de Imports)
