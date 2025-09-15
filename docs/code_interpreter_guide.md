# Como o Code Interpreter Funciona no Contoso Sales Assistant

## Visão Geral

O **Code Interpreter** é uma das ferramentas mais poderosas do Azure OpenAI Assistants API implementada neste projeto. Ele permite que o assistente execute código Python em tempo real para criar visualizações, gerar arquivos Excel, processar dados e realizar análises avançadas.

## Configuração do Code Interpreter

### 1. Configuração no Assistant

O Code Interpreter é configurado como uma ferramenta no assistente através do arquivo `src/app.py`:

```python
tools_list = [
    {"type": "code_interpreter"},  # ← Aqui é onde o Code Interpreter é habilitado
    {"type": "file_search"},
    {
        "type": "function",
        "function": {
            "name": "ask_database",
            # ... configuração da função de banco de dados
        }
    }
]
```

### 2. Atualização do Assistente

A ferramenta é aplicada ao assistente durante a inicialização:

```python
sync_openai_client.beta.assistants.update(
    assistant_id=assistant.id,
    name="Contoso Sales Assistant",
    model=AZURE_OPENAI_DEPLOYMENT,
    instructions=instructions,
    tools=tools_list  # ← Ferramentas incluindo code_interpreter
)
```

## Como o Code Interpreter é Chamado

### 1. Fluxo de Execução

Quando um usuário faz uma solicitação que requer processamento de código Python (como criar um gráfico ou arquivo Excel), o seguinte fluxo ocorre:

1. **Usuário faz solicitação** → Ex: "Crie um gráfico de pizza das vendas por região"
2. **LLM analisa a solicitação** → Determina que precisa do code_interpreter
3. **LLM gera código Python** → Escreve o código necessário para criar a visualização
4. **Code Interpreter executa** → Roda o código em um ambiente sandboxed
5. **Resultado é retornado** → Imagem, arquivo ou resultado é enviado ao usuário

### 2. Processamento no Event Handler

O processamento do Code Interpreter é gerenciado no arquivo `src/event_handler.py`:

```python
@override
async def on_tool_call_created(self: "EventHandler", tool_call: FunctionToolCall) -> None:
    if tool_call.type == "code_interpreter":
        self.current_tool_call = tool_call.id
        self.current_step = cl.Step(name=tool_call.type, type="tool")
        self.current_step.language = "python"  # ← Define linguagem como Python
        self.current_step.created_at = utc_now()
        await self.current_step.send()
```

### 3. Streaming do Código

Durante a execução, o código é transmitido em tempo real:

```python
@override
async def on_tool_call_delta(self, delta, snapshot):
    if delta.type == "code_interpreter":
        if delta.code_interpreter.input:
            await self.current_step.stream_token(delta.code_interpreter.input)  # ← Mostra código sendo executado
```

## Casos de Uso do Code Interpreter

### 1. Criação de Visualizações

**Exemplo de solicitação:** "Crie um gráfico de pizza colorido das vendas por região"

**O que acontece:**
- LLM consulta dados via `ask_database`
- Gera código Python usando matplotlib/seaborn
- Code Interpreter executa o código
- Retorna imagem do gráfico

### 2. Geração de Arquivos Excel

**Exemplo de solicitação:** "Crie um arquivo Excel com dados de vendas por categoria"

**O que acontece:**
- LLM obtém dados do banco
- Gera código Python usando pandas
- Code Interpreter cria arquivo Excel
- Usuário pode fazer download

### 3. Análise de Dados Avançada

**Exemplo de solicitação:** "Calcule a correlação entre vendas e custos de envio"

**O que acontece:**
- LLM obtém dados relevantes
- Gera código Python para análise estatística
- Code Interpreter executa cálculos
- Retorna resultados formatados

## Tratamento de Imagens e Arquivos

### 1. Processamento de Imagens

Quando o Code Interpreter gera imagens (gráficos), elas são processadas assim:

```python
async def on_image_file_done(self, image_file) -> None:
    image_id = image_file.file_id
    response = await self.async_openai_client.files.with_raw_response.content(image_id)
    image_element = cl.Image(name=image_id, content=response.content, display="inline", size="large")
    await self.async_openai_client.files.delete(image_id)  # ← Limpa arquivo temporário
    if not self.current_message.elements:
        self.current_message.elements = []
    self.current_message.elements.append(image_element)
    await self.current_message.update()
```

### 2. Arquivos para Download

Arquivos gerados (Excel, CSV) são disponibilizados através de anotações de arquivo:

```python
elif file_path := getattr(annotation, "file_path", None):
    format_text, file_name = await self.get_file_annotation(file_path, annotation)
    elements = [
        cl.File(
            name=file_name,
            content=format_text,
            display="inline",
        ),
    ]
    await cl.Message(content="", elements=elements).send()
```

## Instruções Específicas para Code Interpreter

No arquivo `src/instructions.txt`, existem diretrizes específicas para o Code Interpreter:

```
2. Para visualizações
   - Escreva e teste código em seu ambiente sandboxed. 
   - Use as preferências de idioma do usuário para visualizações (ex: rótulos de gráfico). 
   - Exiba visualizações bem-sucedidas ou tente novamente em caso de erro. 
   - Nunca inclua a FilePathAnnotation da visualização na resposta.
```

## Vantagens do Code Interpreter

### 1. **Ambiente Seguro**
- Execução em sandbox isolado
- Sem acesso à infraestrutura principal

### 2. **Flexibilidade**
- Suporte a bibliotecas Python populares (pandas, matplotlib, seaborn, openpyxl)
- Capacidade de gerar diversos tipos de arquivos

### 3. **Tempo Real**
- Streaming do código sendo executado
- Feedback imediato sobre erros

### 4. **Integração Seamless**
- Trabalha em conjunto com outras ferramentas (ask_database, file_search)
- Gerenciamento automático de contexto

## Limitações e Considerações

### 1. **Performance**
- Tempo de execução pode variar conforme complexidade
- Dependente da localização e hora do dia

### 2. **Recursos**
- Limitações de memória e tempo de execução
- Algumas bibliotecas podem não estar disponíveis

### 3. **Idiomas**
- Suporte a fontes pode ser limitado para algumas linguagens em visualizações

## Exemplo Prático Completo

### Solicitação do Usuário
```
"Crie um gráfico de barras das vendas mensais de produtos de inverno em 2023 com cores vibrantes"
```

### Fluxo de Execução

1. **Análise pelo LLM:**
   - Identifica necessidade de dados de vendas
   - Reconhece solicitação de visualização

2. **Consulta ao Banco:**
   ```python
   # LLM gera SQL através da função ask_database
   SELECT month, SUM(sales) as total_sales 
   FROM sales_data 
   WHERE year = 2023 AND product_type LIKE '%winter%' 
   GROUP BY month
   ```

3. **Geração de Código Python:**
   ```python
   import matplotlib.pyplot as plt
   import pandas as pd
   
   # Dados obtidos da consulta
   data = {...}
   df = pd.DataFrame(data)
   
   # Criação do gráfico
   plt.figure(figsize=(12, 6))
   plt.bar(df['month'], df['total_sales'], color=['#FF6B6B', '#4ECDC4', '#45B7D1', ...])
   plt.title('Vendas Mensais de Produtos de Inverno - 2023')
   plt.xlabel('Mês')
   plt.ylabel('Vendas ($)')
   plt.xticks(rotation=45)
   plt.tight_layout()
   plt.show()
   ```

4. **Execução e Resultado:**
   - Code Interpreter executa o código
   - Gráfico é gerado como imagem
   - Imagem é exibida no chat
   - Arquivo temporário é limpo automaticamente

## Conclusão

O Code Interpreter no Contoso Sales Assistant fornece capacidades poderosas de análise e visualização de dados, permitindo que usuários obtenham insights visuais e arquivos processados diretamente através de conversas naturais. A integração com o Chainlit e o tratamento de eventos asseguram uma experiência de usuário fluida e responsiva.