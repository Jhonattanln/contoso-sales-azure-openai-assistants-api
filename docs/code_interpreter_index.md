# Documentação Completa do Code Interpreter

Esta é a documentação completa sobre como o **Code Interpreter** funciona no projeto Contoso Sales Assistant.

## 📚 Guias Disponíveis

### 1. 📋 [Resumo Executivo](README_code_interpreter.md)
**Para:** Gestores, Product Owners, usuários finais  
**Conteúdo:** Visão geral simplificada, casos de uso, benefícios  
**Tempo de leitura:** 5-10 minutos

### 2. 📖 [Guia Completo](code_interpreter_guide.md)  
**Para:** Desenvolvedores, arquitetos, analistas técnicos  
**Conteúdo:** Explicação detalhada da implementação, configuração, fluxos  
**Tempo de leitura:** 15-20 minutos

### 3. 🔧 [Fluxos Técnicos](code_interpreter_technical_flow.md)
**Para:** Desenvolvedores, DevOps, arquitetos de sistema  
**Conteúdo:** Diagramas técnicos, arquitetura, estados e eventos  
**Tempo de leitura:** 10-15 minutos

### 4. 💡 [Exemplos Práticos](code_interpreter_examples.md)
**Para:** Desenvolvedores, QA, usuários avançados  
**Conteúdo:** Código Python real, casos de uso detalhados, exemplos completos  
**Tempo de leitura:** 20-30 minutos

## 🎯 Guia de Leitura Recomendado

### Para Gestores e Business Users
```
1. Resumo Executivo (README_code_interpreter.md)
2. Exemplos Práticos - apenas casos de uso (code_interpreter_examples.md)
```

### Para Product Owners e Analistas
```
1. Resumo Executivo (README_code_interpreter.md)
2. Guia Completo - seções de casos de uso (code_interpreter_guide.md)
3. Exemplos Práticos (code_interpreter_examples.md)
```

### Para Desenvolvedores
```
1. Guia Completo (code_interpreter_guide.md)
2. Fluxos Técnicos (code_interpreter_technical_flow.md)
3. Exemplos Práticos (code_interpreter_examples.md)
4. Código fonte: src/app.py, src/event_handler.py
```

### Para Arquitetos e DevOps
```
1. Fluxos Técnicos (code_interpreter_technical_flow.md)
2. Guia Completo - seções técnicas (code_interpreter_guide.md)
3. Código fonte completo
```

## 🔍 Índice Detalhado por Tópico

### 🎨 Visualizações e Gráficos
- **Configuração:** [Guia Completo - Configuração](code_interpreter_guide.md#configuração-do-code-interpreter)
- **Tipos de gráficos:** [Exemplos - Visualizações](code_interpreter_examples.md#exemplo-1-gráfico-de-pizza-das-vendas-por-região)
- **Processamento:** [Fluxos Técnicos - Imagens](code_interpreter_technical_flow.md#tipos-de-saída-do-code-interpreter)

### 📄 Geração de Arquivos
- **Excel/CSV:** [Exemplos - Relatórios](code_interpreter_examples.md#exemplo-3-arquivo-excel-de-relatório)
- **Download:** [Guia Completo - Arquivos](code_interpreter_guide.md#tratamento-de-imagens-e-arquivos)
- **Processamento:** [Fluxos Técnicos - Arquivos](code_interpreter_technical_flow.md#tipos-de-saída-do-code-interpreter)

### 🔧 Implementação Técnica
- **EventHandler:** [Guia Completo - Eventos](code_interpreter_guide.md#processamento-no-event-handler)
- **Configuração:** [Fluxos Técnicos - Configurações](code_interpreter_technical_flow.md#configurações-e-limitações)
- **Integração:** [Fluxos Técnicos - Integração](code_interpreter_technical_flow.md#integração-com-outras-ferramentas)

### 🧠 Inteligência Artificial
- **LLM Integration:** [Guia Completo - Como é Chamado](code_interpreter_guide.md#como-o-code-interpreter-é-chamado)
- **Geração de Código:** [Exemplos - Código Gerado](code_interpreter_examples.md#código-python-gerado-pelo-llm)
- **Fluxo Completo:** [Fluxos Técnicos - Fluxo de Execução](code_interpreter_technical_flow.md#fluxo-de-execução-detalhado)

## 🚀 Início Rápido

### Para entender rapidamente o Code Interpreter:

1. **O que é?** → Ferramenta que executa código Python automaticamente  
2. **Onde está configurado?** → `src/app.py` linha 102  
3. **Como funciona?** → LLM gera código → Sandbox executa → Resultado no chat  
4. **Principais usos?** → Gráficos, arquivos Excel, análises estatísticas  

### Para implementar ou modificar:

1. **Leia:** [Guia Completo](code_interpreter_guide.md)  
2. **Veja diagramas:** [Fluxos Técnicos](code_interpreter_technical_flow.md)  
3. **Estude código:** `src/event_handler.py` métodos `on_tool_call_*`  
4. **Teste com:** [Exemplos Práticos](code_interpreter_examples.md)  

## 📋 Arquivos do Projeto Relacionados

### Código Principal
- `src/app.py` - Configuração do assistente e ferramentas
- `src/event_handler.py` - Processamento de eventos do Code Interpreter  
- `src/instructions.txt` - Diretrizes para o LLM sobre Code Interpreter

### Documentação Oficial do Projeto
- `docs/docs/conversation.md` - Exemplos de uso no chat
- `docs/docs/index.md` - Visão geral do projeto
- `README.md` - Documentação principal do projeto

### Configuração
- `pyproject.toml` - Configurações de linting (black, ruff)
- `requirements.txt` - Dependências do projeto

## ❓ Perguntas Frequentes

### Como o Code Interpreter é diferente de outras ferramentas?
- **ask_database:** Consulta dados estruturados (SQL)
- **file_search:** Busca informações em documentos  
- **code_interpreter:** Executa código Python para análises e visualizações

### Posso modificar as bibliotecas disponíveis?
Não. O ambiente sandbox do Azure OpenAI tem bibliotecas pré-definidas. Veja a lista completa em [Fluxos Técnicos - Bibliotecas](code_interpreter_technical_flow.md#bibliotecas-python-disponíveis).

### Como debugar problemas no Code Interpreter?
1. Verifique logs no EventHandler  
2. Analise o código Python gerado (visível no chat)
3. Teste bibliotecas em ambiente local
4. Consulte [Exemplos - Tratamento de Erros](code_interpreter_examples.md#como-o-eventhandler-processa-estes-exemplos)

### Posso usar o Code Interpreter para outros tipos de análise?
Sim! Os exemplos mostram casos básicos, mas é possível análises estatísticas avançadas, machine learning simples, processamento de texto, etc. Limitado apenas pelas bibliotecas disponíveis.

---

**Contribuições:** Para melhorar esta documentação, edite os arquivos markdown e mantenha a consistência de formato e estrutura.