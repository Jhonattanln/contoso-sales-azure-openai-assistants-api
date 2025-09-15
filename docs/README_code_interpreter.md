# Code Interpreter no Contoso Sales Assistant - Resumo Completo

## O que é o Code Interpreter?

O **Code Interpreter** é uma ferramenta poderosa do Azure OpenAI Assistants API que permite executar código Python em tempo real dentro de um ambiente seguro (sandbox). No projeto Contoso Sales Assistant, ele é usado para:

- 🎨 **Criar visualizações** (gráficos, charts, dashboards)
- 📊 **Gerar arquivos** (Excel, CSV, PDF)
- 🔢 **Realizar análises estatísticas** avançadas
- 📈 **Processar dados** obtidos do banco de vendas

## Como Funciona no Projeto

### 1. Configuração Inicial

O Code Interpreter é configurado como uma das três ferramentas principais do assistente:

```python
# src/app.py - linha 101-125
tools_list = [
    {"type": "code_interpreter"},    # ← Ferramenta de execução Python
    {"type": "file_search"},         # ← Busca em arquivos
    {"type": "function", "function": {"name": "ask_database"}}  # ← Consulta BD
]
```

### 2. Fluxo de Execução

1. **Usuário solicita** → "Crie um gráfico de pizza das vendas por região"
2. **LLM analisa** → Determina que precisa de dados + visualização  
3. **Consulta dados** → Usa `ask_database` para obter informações
4. **Gera código** → LLM escreve código Python automaticamente
5. **Executa código** → Code Interpreter roda em ambiente sandbox
6. **Retorna resultado** → Imagem/arquivo é exibido no chat

### 3. Processamento de Eventos

O arquivo `src/event_handler.py` gerencia todo o ciclo de vida:

```python
# Quando code_interpreter inicia
async def on_tool_call_created(self, tool_call):
    if tool_call.type == "code_interpreter":
        self.current_step = cl.Step(name="code_interpreter", type="tool")
        self.current_step.language = "python"

# Durante execução (streaming do código)
async def on_tool_call_delta(self, delta, snapshot):
    if delta.code_interpreter.input:
        await self.current_step.stream_token(delta.code_interpreter.input)

# Quando gera imagem
async def on_image_file_done(self, image_file):
    # Download e exibição da imagem gerada
    
# Quando termina
async def on_tool_call_done(self, tool_call):
    # Finaliza o step visual
```

## Casos de Uso Principais

### 📊 Visualizações de Dados
- **Gráficos de Pizza**: Distribuição de vendas por região/categoria
- **Gráficos de Barras**: Comparações mensais/trimestrais
- **Gráficos de Linha**: Tendências temporais
- **Dashboards**: Múltiplos gráficos em uma visualização

### 📄 Geração de Arquivos
- **Excel**: Relatórios detalhados com múltiplas abas
- **CSV**: Dados para análise externa
- **Gráficos salvos**: PNG/JPG para apresentações

### 🔍 Análises Avançadas
- **Estatísticas descritivas**: Média, mediana, desvio padrão
- **Correlações**: Relacionamentos entre variáveis
- **Previsões**: Tendências futuras baseadas em dados históricos

## Vantagens Técnicas

### 🔒 Segurança
- **Ambiente Isolado**: Execução em sandbox sem acesso ao sistema
- **Sem Persistência**: Códigos não afetam a infraestrutura
- **Controle de Recursos**: Limitações de tempo e memória

### ⚡ Performance
- **Execução Assíncrona**: Não bloqueia outras operações
- **Streaming em Tempo Real**: Usuário vê código sendo executado
- **Cleanup Automático**: Arquivos temporários são removidos

### 🧠 Inteligência
- **Geração Automática**: LLM cria código otimizado
- **Correção de Erros**: Tentativas automáticas em caso de falha
- **Adaptação ao Contexto**: Código se adapta aos dados disponíveis

## Integração com Outras Ferramentas

O Code Interpreter trabalha em conjunto com:

### 🗃️ ask_database
```
Usuário: "Gráfico de vendas por região"
└── 1. ask_database → Obtém dados do SQLite
└── 2. code_interpreter → Cria visualização
└── 3. Exibe resultado no chat
```

### 📁 file_search  
```
Usuário: "Análise com dados do PDF carregado"
└── 1. file_search → Extrai informações do PDF
└── 2. ask_database → Complementa com dados do BD
└── 3. code_interpreter → Cria análise combinada
```

## Bibliotecas Python Disponíveis

O ambiente sandbox inclui bibliotecas essenciais:

- **📊 Análise**: `pandas`, `numpy`
- **🎨 Visualização**: `matplotlib`, `seaborn`, `plotly`
- **📄 Arquivos**: `openpyxl`, `xlsxwriter`
- **🔢 Matemática**: `scipy`, `statsmodels`
- **🛠️ Utilidades**: `datetime`, `json`, `os`

## Configurações e Limitações

### ⚙️ Configurações (src/app.py)
```python
MAX_COMPLETION_TOKENS = 4096    # Máximo tokens na resposta
MAX_PROMPT_TOKENS = 10240       # Máximo tokens no prompt  
temperature = 0.2               # Controle de criatividade
```

### ⚠️ Limitações Importantes
- **Tempo**: ~60 segundos máximo de execução
- **Memória**: Limitada pelo ambiente sandbox  
- **Internet**: Sem acesso à rede externa
- **Persistência**: Arquivos não persistem entre sessões
- **Bibliotecas**: Apenas pré-instaladas disponíveis

## Exemplo Prático Completo

### Solicitação
```
"Crie um dashboard com gráfico de pizza das vendas por região e gráfico de barras por categoria"
```

### Processo Interno
1. **LLM entende** → Precisa de dados de vendas + duas visualizações
2. **Consulta dados** → `ask_database("SELECT region, SUM(sales) FROM...")`
3. **Gera código** → Script Python com matplotlib para 2 subplots
4. **Executa** → Code Interpreter roda o código
5. **EventHandler** → Captura imagem e exibe no Chainlit
6. **Resultado** → Dashboard aparece no chat do usuário

### Código Python Gerado (exemplo)
```python
import matplotlib.pyplot as plt
import pandas as pd

# Dados obtidos via ask_database
regions = ['North America', 'Europe', 'Asia', 'South America']  
sales_region = [2500000, 1800000, 2200000, 900000]
categories = ['Camping', 'Winter Sports', 'Water Sports', 'Climbing']
sales_category = [1800000, 1600000, 1200000, 1000000]

# Dashboard com 2 gráficos
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(15, 6))

# Gráfico de pizza
ax1.pie(sales_region, labels=regions, autopct='%1.1f%%', 
        colors=['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4'])
ax1.set_title('Vendas por Região')

# Gráfico de barras  
ax2.bar(categories, sales_category, color='steelblue', alpha=0.7)
ax2.set_title('Vendas por Categoria')
ax2.set_ylabel('Vendas ($)')
ax2.tick_params(axis='x', rotation=45)

plt.tight_layout()
plt.show()
```

## Interface do Usuário

### 👀 O que o Usuário Vê
1. **Step "python"** → Caixa expandível com código sendo executado
2. **Resultado visual** → Gráfico/imagem exibida inline no chat
3. **Link de download** → Para arquivos gerados (Excel, CSV)
4. **Resposta textual** → Explicação do LLM sobre os resultados

### 🎛️ Controles Disponíveis
- **Expandir/Colapsar** → Ver/ocultar código Python executado
- **Download** → Baixar arquivos gerados
- **Zoom** → Ampliar imagens/gráficos
- **Histórico** → Navegar entre visualizações anteriores

## Arquivos de Documentação Criados

1. **📚 `code_interpreter_guide.md`** → Guia completo e detalhado
2. **🔧 `code_interpreter_technical_flow.md`** → Diagramas e fluxos técnicos  
3. **💡 `code_interpreter_examples.md`** → Exemplos práticos com código
4. **📋 `README_code_interpreter.md`** → Este resumo executivo

## Conclusão

O Code Interpreter no Contoso Sales Assistant representa uma implementação sofisticada que combina:

- **🤖 IA Generativa** para criação automática de código
- **🔒 Execução Segura** em ambiente sandbox
- **🎨 Interface Intuitiva** com streaming em tempo real  
- **🔗 Integração Seamless** com outras ferramentas do assistente

Esta implementação permite que usuários de negócio obtenham insights visuais e análises avançadas através de conversas naturais, sem necessidade de conhecimento técnico em programação ou análise de dados.

---

**Arquivos para referência completa:**
- `src/app.py` - Configuração principal
- `src/event_handler.py` - Processamento de eventos
- `src/instructions.txt` - Diretrizes para o LLM
- `docs/docs/conversation.md` - Exemplos de uso