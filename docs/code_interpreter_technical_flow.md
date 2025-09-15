# Code Interpreter - Diagramas e Fluxos Técnicos

## Arquitetura do Code Interpreter

```mermaid
graph TB
    subgraph "Usuário"
        U[👤 Usuário]
    end
    
    subgraph "Chainlit Interface"
        CI[🖥️ Chainlit UI]
        CH[📨 Chat Handler]
    end
    
    subgraph "Azure OpenAI Assistants API"
        AI[🧠 Assistant LLM]
        TH[🔧 Tools Handler]
        subgraph "Ferramentas"
            DB[🗄️ ask_database]
            FS[📁 file_search]
            CODE[🐍 code_interpreter]
        end
    end
    
    subgraph "Event Handler"
        EH[⚡ EventHandler]
        TC[🔄 Tool Call Events]
        TD[📊 Tool Delta Events]
        TR[✅ Tool Done Events]
    end
    
    subgraph "Execução Code Interpreter"
        SANDBOX[🏖️ Python Sandbox]
        LIB[📚 Libraries<br/>pandas, matplotlib, seaborn]
        FILE[📄 File Generation]
        IMG[🖼️ Image Generation]
    end
    
    U --> CI
    CI --> CH
    CH --> AI
    AI --> TH
    TH --> CODE
    CODE --> SANDBOX
    SANDBOX --> LIB
    LIB --> FILE
    LIB --> IMG
    
    CODE --> EH
    EH --> TC
    EH --> TD
    EH --> TR
    
    TR --> CI
    FILE --> CI
    IMG --> CI
```

## Fluxo de Execução Detalhado

```mermaid
sequenceDiagram
    participant U as 👤 Usuário
    participant C as 🖥️ Chainlit
    participant A as 🧠 Assistant API
    participant E as ⚡ EventHandler
    participant CI as 🐍 Code Interpreter
    participant S as 🏖️ Sandbox
    
    U->>C: "Crie um gráfico de pizza das vendas"
    C->>A: Enviar mensagem + thread
    
    Note over A: LLM analisa solicitação
    A->>A: Determina necessidade de dados
    A->>A: Chama ask_database function
    A->>A: Recebe dados do banco
    A->>A: Decide usar code_interpreter
    
    A->>E: on_tool_call_created(code_interpreter)
    E->>C: Criar Step "python"
    
    A->>E: on_tool_call_delta
    E->>C: Stream código Python sendo gerado
    
    A->>CI: Executar código Python
    CI->>S: Rodar em ambiente isolado
    
    Note over S: Código executa:<br/>- Processa dados<br/>- Cria gráfico<br/>- Salva imagem
    
    S-->>CI: Imagem gerada
    CI-->>A: Resultado da execução
    A->>E: on_image_file_done
    E->>C: Exibir imagem no chat
    
    A->>E: on_tool_call_done
    E->>C: Finalizar Step
    
    C->>U: Mostrar gráfico + resposta
```

## Estados e Eventos do Code Interpreter

```mermaid
stateDiagram-v2
    [*] --> ToolCallCreated : Usuário solicita visualização
    
    ToolCallCreated --> StreamingCode : LLM gera código
    StreamingCode --> StreamingCode : delta.code_interpreter.input
    StreamingCode --> CodeExecuting : Código completo gerado
    
    CodeExecuting --> ProcessingOutput : Sandbox executa código
    ProcessingOutput --> ImageGenerated : Gráfico/visualização criada
    ProcessingOutput --> FileGenerated : Arquivo Excel/CSV criado
    ProcessingOutput --> ErrorOccurred : Erro na execução
    
    ImageGenerated --> DisplayingResult : on_image_file_done
    FileGenerated --> DisplayingResult : file_path annotation
    ErrorOccurred --> DisplayingError : Mostrar erro
    
    DisplayingResult --> ToolCallDone : Sucesso
    DisplayingError --> ToolCallDone : Erro tratado
    
    ToolCallDone --> [*] : Processo finalizado
```

## Tipos de Saída do Code Interpreter

```mermaid
graph LR
    subgraph "Code Interpreter Outputs"
        CODE[🐍 Python Code]
    end
    
    CODE --> IMG[🖼️ Imagens]
    CODE --> FILE[📄 Arquivos]
    CODE --> LOG[📝 Logs]
    CODE --> ERR[❌ Erros]
    
    subgraph "Tipos de Imagem"
        IMG --> CHART[📊 Gráficos]
        IMG --> PLOT[📈 Plots]
        IMG --> DIAGRAM[📋 Diagramas]
    end
    
    subgraph "Tipos de Arquivo"
        FILE --> EXCEL[📊 Excel .xlsx]
        FILE --> CSV[📋 CSV]
        FILE --> JSON[📄 JSON]
        FILE --> PDF[📄 PDF]
    end
    
    subgraph "Processamento Chainlit"
        CHART --> DISPLAY[🖥️ Exibição Inline]
        EXCEL --> DOWNLOAD[⬇️ Link Download]
        LOG --> STREAM[📺 Stream Terminal]
        ERR --> ERROR[⚠️ Mensagem Erro]
    end
```

## Integração com Outras Ferramentas

```mermaid
graph TB
    subgraph "Fluxo Integrado"
        START[🎯 Solicitação do Usuário]
        
        subgraph "Fase 1: Obtenção de Dados"
            DB_CALL[🗄️ ask_database<br/>SQL Query]
            DB_RESULT[📊 Dados Retornados]
        end
        
        subgraph "Fase 2: Processamento"
            AI_DECIDE[🧠 LLM Decide<br/>Usar Code Interpreter]
            CODE_GEN[🐍 Gerar Código Python]
        end
        
        subgraph "Fase 3: Execução"
            CODE_EXEC[⚙️ Executar Código]
            CREATE_VIZ[🎨 Criar Visualização]
        end
        
        subgraph "Fase 4: Apresentação"
            SHOW_RESULT[📺 Exibir Resultado]
            USER_DOWNLOAD[⬇️ Disponibilizar Download]
        end
    end
    
    START --> DB_CALL
    DB_CALL --> DB_RESULT
    DB_RESULT --> AI_DECIDE
    AI_DECIDE --> CODE_GEN
    CODE_GEN --> CODE_EXEC
    CODE_EXEC --> CREATE_VIZ
    CREATE_VIZ --> SHOW_RESULT
    CREATE_VIZ --> USER_DOWNLOAD
```

## Event Handler - Métodos Específicos

```python
# Principais métodos para Code Interpreter no EventHandler

class EventHandler(AsyncAssistantEventHandler):
    
    async def on_tool_call_created(self, tool_call):
        """Quando code_interpreter é iniciado"""
        if tool_call.type == "code_interpreter":
            # Criar step visual no Chainlit
            self.current_step = cl.Step(name="code_interpreter", type="tool")
            self.current_step.language = "python"
    
    async def on_tool_call_delta(self, delta, snapshot):
        """Durante execução do código (streaming)"""
        if delta.type == "code_interpreter":
            if delta.code_interpreter.input:
                # Mostrar código sendo executado
                await self.current_step.stream_token(delta.code_interpreter.input)
    
    async def on_image_file_done(self, image_file):
        """Quando imagem é gerada"""
        # Fazer download da imagem
        # Criar elemento visual
        # Limpar arquivo temporário
    
    async def on_tool_call_done(self, tool_call):
        """Quando code_interpreter termina"""
        if tool_call.type == "code_interpreter":
            # Finalizar step visual
            self.current_step.end = utc_now()
            await self.current_step.update()
```

## Bibliotecas Python Disponíveis

O ambiente sandbox do Code Interpreter inclui:

- **Análise de Dados**: pandas, numpy
- **Visualização**: matplotlib, seaborn, plotly
- **Arquivos**: openpyxl, xlsxwriter
- **Utilidades**: datetime, json, os, sys
- **Matemática**: scipy, statsmodels (limitado)

## Configurações e Limitações

### Configurações no app.py
```python
MAX_COMPLETION_TOKENS = 4096    # Máximo tokens na resposta
MAX_PROMPT_TOKENS = 10240       # Máximo tokens no prompt
temperature=0.2                 # Controle de criatividade
```

### Limitações Importantes
1. **Tempo de execução**: ~60 segundos máximo
2. **Memória**: Limitada pelo ambiente sandbox
3. **Persistência**: Arquivos não persistem entre execuções
4. **Internet**: Sem acesso à internet no sandbox
5. **Bibliotecas**: Apenas bibliotecas pré-instaladas