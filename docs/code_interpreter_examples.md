# Code Interpreter - Exemplos Práticos

Este documento apresenta exemplos reais de como o Code Interpreter é usado no Contoso Sales Assistant, incluindo solicitações de usuários e o código Python gerado automaticamente.

## Exemplo 1: Gráfico de Pizza das Vendas por Região

### Solicitação do Usuário
```
"Crie um gráfico de pizza colorido das vendas por região"
```

### Código Python Gerado pelo LLM
```python
import matplotlib.pyplot as plt
import pandas as pd

# Dados obtidos da consulta SQL anterior
data = {
    'Region': ['North America', 'Europe', 'Asia', 'South America', 'Africa'],
    'Sales': [2500000, 1800000, 2200000, 900000, 600000]
}

df = pd.DataFrame(data)

# Criando o gráfico de pizza
plt.figure(figsize=(10, 8))
colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4', '#FFEAA7']
plt.pie(df['Sales'], labels=df['Region'], autopct='%1.1f%%', colors=colors, startangle=90)
plt.title('Vendas por Região - Distribuição Percentual', fontsize=16, fontweight='bold')
plt.axis('equal')
plt.tight_layout()
plt.show()
```

### Resultado
- **Saída**: Gráfico de pizza colorido exibido inline no chat
- **Processamento**: EventHandler captura imagem e exibe via `cl.Image`

## Exemplo 2: Gráfico de Barras Mensais

### Solicitação do Usuário
```
"Mostre um gráfico de barras das vendas mensais de produtos de inverno em 2023"
```

### Código Python Gerado
```python
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

# Dados das vendas mensais (obtidos via ask_database)
months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 
          'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
sales = [150000, 180000, 120000, 80000, 50000, 30000,
         25000, 35000, 75000, 120000, 200000, 250000]

# Criando o gráfico
plt.figure(figsize=(12, 6))
bars = plt.bar(months, sales, color='steelblue', alpha=0.7, edgecolor='navy')

# Adicionando valores no topo de cada barra
for bar, value in zip(bars, sales):
    plt.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 2000,
             f'${value:,}', ha='center', va='bottom', fontweight='bold')

plt.title('Vendas Mensais de Produtos de Inverno - 2023', fontsize=16, fontweight='bold')
plt.xlabel('Mês', fontsize=12)
plt.ylabel('Vendas ($)', fontsize=12)
plt.xticks(rotation=45)
plt.grid(axis='y', alpha=0.3)
plt.tight_layout()
plt.show()
```

## Exemplo 3: Arquivo Excel de Relatório

### Solicitação do Usuário
```
"Crie um arquivo Excel com as vendas por categoria e região"
```

### Código Python Gerado
```python
import pandas as pd
from datetime import datetime

# Dados obtidos do banco de dados
sales_data = [
    {'Category': 'Camping', 'Region': 'North America', 'Sales': 450000, 'Units': 1200},
    {'Category': 'Camping', 'Region': 'Europe', 'Sales': 380000, 'Units': 1050},
    {'Category': 'Winter Sports', 'Region': 'North America', 'Sales': 320000, 'Units': 800},
    {'Category': 'Winter Sports', 'Region': 'Europe', 'Sales': 420000, 'Units': 950},
    {'Category': 'Water Sports', 'Region': 'North America', 'Sales': 280000, 'Units': 600},
    {'Category': 'Water Sports', 'Region': 'Asia', 'Sales': 350000, 'Units': 750}
]

# Criando DataFrame
df = pd.DataFrame(sales_data)

# Criando resumo por categoria
category_summary = df.groupby('Category').agg({
    'Sales': 'sum',
    'Units': 'sum'
}).reset_index()

# Criando resumo por região
region_summary = df.groupby('Region').agg({
    'Sales': 'sum',
    'Units': 'sum'
}).reset_index()

# Criando arquivo Excel com múltiplas abas
with pd.ExcelWriter('contoso_sales_report.xlsx', engine='openpyxl') as writer:
    # Aba 1: Dados detalhados
    df.to_excel(writer, sheet_name='Vendas Detalhadas', index=False)
    
    # Aba 2: Resumo por categoria
    category_summary.to_excel(writer, sheet_name='Por Categoria', index=False)
    
    # Aba 3: Resumo por região
    region_summary.to_excel(writer, sheet_name='Por Região', index=False)
    
    # Aba 4: Análise combinada
    pivot_table = df.pivot_table(values='Sales', index='Category', 
                                columns='Region', aggfunc='sum', fill_value=0)
    pivot_table.to_excel(writer, sheet_name='Análise Cruzada')

print("Arquivo Excel 'contoso_sales_report.xlsx' criado com sucesso!")
print(f"Data de criação: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
```

### Resultado
- **Arquivo**: contoso_sales_report.xlsx disponível para download
- **Conteúdo**: 4 abas com análises diferentes
- **Download**: Link de download disponibilizado via Chainlit

## Exemplo 4: Análise Estatística

### Solicitação do Usuário
```
"Calcule estatísticas descritivas das vendas e mostre um histograma"
```

### Código Python Gerado
```python
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

# Dados das vendas individuais (sample data)
np.random.seed(42)
sales_data = np.random.normal(50000, 15000, 1000)  # Simulating 1000 sales
sales_data = sales_data[sales_data > 0]  # Remove negative values

# Estatísticas descritivas
stats = {
    'Média': np.mean(sales_data),
    'Mediana': np.median(sales_data),
    'Desvio Padrão': np.std(sales_data),
    'Mínimo': np.min(sales_data),
    'Máximo': np.max(sales_data),
    'Q1 (25%)': np.percentile(sales_data, 25),
    'Q3 (75%)': np.percentile(sales_data, 75)
}

# Exibindo estatísticas
print("=== ESTATÍSTICAS DESCRITIVAS DAS VENDAS ===")
for key, value in stats.items():
    print(f"{key}: ${value:,.2f}")

# Criando histograma
plt.figure(figsize=(12, 8))
plt.subplot(2, 1, 1)
plt.hist(sales_data, bins=30, alpha=0.7, color='skyblue', edgecolor='black')
plt.axvline(stats['Média'], color='red', linestyle='--', linewidth=2, label=f"Média: ${stats['Média']:,.0f}")
plt.axvline(stats['Mediana'], color='green', linestyle='--', linewidth=2, label=f"Mediana: ${stats['Mediana']:,.0f}")
plt.title('Distribuição das Vendas - Histograma', fontsize=14, fontweight='bold')
plt.xlabel('Valor da Venda ($)')
plt.ylabel('Frequência')
plt.legend()
plt.grid(alpha=0.3)

# Box plot
plt.subplot(2, 1, 2)
plt.boxplot(sales_data, vert=False, patch_artist=True, 
            boxprops=dict(facecolor='lightblue', alpha=0.7))
plt.title('Distribuição das Vendas - Box Plot', fontsize=14, fontweight='bold')
plt.xlabel('Valor da Venda ($)')
plt.grid(alpha=0.3)

plt.tight_layout()
plt.show()
```

## Exemplo 5: Análise de Tendências

### Solicitação do Usuário
```
"Crie um gráfico de linha mostrando a tendência de vendas trimestrais"
```

### Código Python Gerado
```python
import matplotlib.pyplot as plt
import pandas as pd
from datetime import datetime

# Dados trimestrais de vendas
quarters = ['Q1 2022', 'Q2 2022', 'Q3 2022', 'Q4 2022',
           'Q1 2023', 'Q2 2023', 'Q3 2023', 'Q4 2023']
sales = [1200000, 1350000, 1180000, 1450000, 
         1300000, 1420000, 1250000, 1580000]

# Calculando crescimento percentual
growth_rates = []
for i in range(1, len(sales)):
    growth = ((sales[i] - sales[i-1]) / sales[i-1]) * 100
    growth_rates.append(growth)

# Criando gráfico de linha
plt.figure(figsize=(14, 8))

# Subplot 1: Tendência de vendas
plt.subplot(2, 1, 1)
plt.plot(quarters, sales, marker='o', linewidth=3, markersize=8, color='#2E86AB')
plt.fill_between(quarters, sales, alpha=0.3, color='#A23B72')

# Adicionando valores nos pontos
for i, (quarter, sale) in enumerate(zip(quarters, sales)):
    plt.annotate(f'${sale/1000000:.1f}M', 
                (i, sale), 
                textcoords="offset points", 
                xytext=(0,10), 
                ha='center', fontweight='bold')

plt.title('Tendência de Vendas Trimestrais - Contoso', fontsize=16, fontweight='bold')
plt.xlabel('Trimestre')
plt.ylabel('Vendas ($)')
plt.xticks(rotation=45)
plt.grid(True, alpha=0.3)

# Subplot 2: Taxa de crescimento
plt.subplot(2, 1, 2)
colors = ['green' if x >= 0 else 'red' for x in growth_rates]
bars = plt.bar(quarters[1:], growth_rates, color=colors, alpha=0.7)

# Adicionando valores nas barras
for bar, rate in zip(bars, growth_rates):
    height = bar.get_height()
    plt.text(bar.get_x() + bar.get_width()/2., height + (0.5 if height >= 0 else -1.5),
             f'{rate:.1f}%', ha='center', va='bottom' if height >= 0 else 'top',
             fontweight='bold')

plt.title('Taxa de Crescimento Trimestral (%)', fontsize=14, fontweight='bold')
plt.xlabel('Trimestre')
plt.ylabel('Crescimento (%)')
plt.xticks(rotation=45)
plt.axhline(y=0, color='black', linestyle='-', alpha=0.5)
plt.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()

# Resumo da análise
print("=== ANÁLISE DE TENDÊNCIAS ===")
print(f"Crescimento médio: {np.mean(growth_rates):.1f}%")
print(f"Maior crescimento: {max(growth_rates):.1f}% ({quarters[growth_rates.index(max(growth_rates))+1]})")
print(f"Maior queda: {min(growth_rates):.1f}% ({quarters[growth_rates.index(min(growth_rates))+1]})")
```

## Exemplo 6: Dashboard Multi-gráfico

### Solicitação do Usuário
```
"Crie um dashboard com múltiplos gráficos sobre as vendas"
```

### Código Python Gerado
```python
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

# Configuração do estilo
plt.style.use('seaborn-v0_8')
fig, ((ax1, ax2), (ax3, ax4)) = plt.subplots(2, 2, figsize=(16, 12))
fig.suptitle('Dashboard de Vendas Contoso - 2023', fontsize=20, fontweight='bold')

# Dados simulados baseados no banco
regions = ['América do Norte', 'Europa', 'Ásia', 'América do Sul']
sales_by_region = [2500000, 1800000, 2200000, 900000]
categories = ['Camping', 'Esportes de Inverno', 'Esportes Aquáticos', 'Escalada']
sales_by_category = [1800000, 1600000, 1200000, 1000000]

# Gráfico 1: Pizza - Vendas por Região
colors1 = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4']
ax1.pie(sales_by_region, labels=regions, autopct='%1.1f%%', colors=colors1)
ax1.set_title('Vendas por Região', fontsize=14, fontweight='bold')

# Gráfico 2: Barras - Vendas por Categoria
bars = ax2.bar(categories, sales_by_category, color='steelblue', alpha=0.7)
ax2.set_title('Vendas por Categoria', fontsize=14, fontweight='bold')
ax2.set_ylabel('Vendas ($)')
ax2.tick_params(axis='x', rotation=45)

# Adicionando valores nas barras
for bar, value in zip(bars, sales_by_category):
    ax2.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 20000,
             f'${value/1000000:.1f}M', ha='center', va='bottom', fontweight='bold')

# Gráfico 3: Linha - Tendência Mensal
months = list(range(1, 13))
monthly_sales = [180000, 165000, 190000, 175000, 200000, 210000,
                185000, 195000, 220000, 240000, 260000, 280000]
ax3.plot(months, monthly_sales, marker='o', linewidth=2, markersize=6, color='#E74C3C')
ax3.fill_between(months, monthly_sales, alpha=0.3, color='#E74C3C')
ax3.set_title('Tendência de Vendas Mensais', fontsize=14, fontweight='bold')
ax3.set_xlabel('Mês')
ax3.set_ylabel('Vendas ($)')
ax3.grid(True, alpha=0.3)

# Gráfico 4: Barras Horizontais - Top Produtos
products = ['Barraca Alpine Pro', 'Kayak Explorer', 'Bota Montanha', 'Saco Dormir Pro', 'Mochila Trekking']
product_sales = [450000, 380000, 320000, 280000, 250000]
bars4 = ax4.barh(products, product_sales, color='#8E44AD', alpha=0.7)
ax4.set_title('Top 5 Produtos', fontsize=14, fontweight='bold')
ax4.set_xlabel('Vendas ($)')

# Adicionando valores
for bar, value in zip(bars4, product_sales):
    ax4.text(bar.get_width() + 5000, bar.get_y() + bar.get_height()/2,
             f'${value/1000:.0f}K', ha='left', va='center', fontweight='bold')

plt.tight_layout()
plt.show()

# Resumo executivo
print("=== RESUMO EXECUTIVO ===")
print(f"Total de Vendas: ${sum(sales_by_region)/1000000:.1f}M")
print(f"Região Líder: {regions[sales_by_region.index(max(sales_by_region))]}")
print(f"Categoria Líder: {categories[sales_by_category.index(max(sales_by_category))]}")
print(f"Crescimento Anual Estimado: +12.5%")
```

## Como o EventHandler Processa Estes Exemplos

### Fluxo no EventHandler
```python
# Quando o code_interpreter inicia
async def on_tool_call_created(self, tool_call):
    if tool_call.type == "code_interpreter":
        # Cria step visual no Chainlit
        self.current_step = cl.Step(name="code_interpreter", type="tool")
        self.current_step.language = "python"
        await self.current_step.send()

# Durante a execução (mostra código em tempo real)
async def on_tool_call_delta(self, delta, snapshot):
    if delta.type == "code_interpreter":
        if delta.code_interpreter.input:
            # Stream do código Python sendo executado
            await self.current_step.stream_token(delta.code_interpreter.input)

# Quando uma imagem é gerada
async def on_image_file_done(self, image_file):
    # Faz download da imagem
    image_id = image_file.file_id
    response = await self.async_openai_client.files.with_raw_response.content(image_id)
    
    # Cria elemento para exibição
    image_element = cl.Image(
        name=image_id, 
        content=response.content, 
        display="inline", 
        size="large"
    )
    
    # Adiciona à mensagem atual
    self.current_message.elements.append(image_element)
    await self.current_message.update()

# Quando termina a execução
async def on_tool_call_done(self, tool_call):
    if tool_call.type == "code_interpreter":
        self.current_step.end = utc_now()
        await self.current_step.update()
```

## Dicas para Melhor Uso

### Solicitações Eficazes
✅ **Bom**: "Crie um gráfico de barras das vendas por categoria com cores vibrantes"
❌ **Ruim**: "Faça um gráfico"

✅ **Bom**: "Gere um arquivo Excel com análise de vendas trimestrais por região"
❌ **Ruim**: "Crie uma planilha"

### Personalização de Idioma
```
"Crie um gráfico de pizza das vendas por região com legendas em português"
"Create a bar chart of monthly sales with English labels"
"Créer un graphique en barres des ventes mensuelles avec des étiquettes en français"
```

### Solicitações Complexas
```
"Crie um dashboard com 4 gráficos: pizza (regiões), barras (categorias), 
linha (tendência mensal) e tabela (top 10 produtos). Use cores profissionais 
e inclua valores nos gráficos."
```