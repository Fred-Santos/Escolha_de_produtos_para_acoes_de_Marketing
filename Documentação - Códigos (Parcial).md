
# 🧾 Documentação Técnica - Web Scraping Mercado Livre

Este script realiza a **extração automatizada de dados de produtos para cães** no site Mercado Livre, utilizando `Selenium`, `BeautifulSoup` e `Requests`.

---

## 📦 Requisitos

Instale os pacotes necessários:

```bash
pip install requests
pip install beautifulsoup4
pip install selenium
```

---

## 🔗 URL de origem

```python
url = "https://lista.mercadolivre.com.br/_Container_pet-cpg-caes"
```

---

## 🚀 1. Inicialização do Selenium

```python
from selenium import webdriver
from selenium.webdriver.edge.service import Service

edge_service = Service(r"Caminho\para\msedgedriver.exe")
options = webdriver.EdgeOptions()
driver = webdriver.Edge(service=edge_service, options=options)
driver.get(url)
```

- Inicia o navegador Edge automatizado
- Acessa a URL de listagem de produtos

---

## 🔄 2. Extração de links de produtos

```python
produtos_links = []

title_wrappers = driver.find_elements(By.CLASS_NAME,"poly-component__title-wrapper")
for wrapper in title_wrappers:
    link_element = wrapper.find_element(By.TAG_NAME, "a")
    produtos_links.append(link_element.get_attribute("href"))
```

- Captura os links dos produtos em cada página
- Navega para a próxima página via botão "Próximo"
- Repetição até não haver mais páginas

---

## 💾 3. Armazenamento dos links

```python
df = pd.DataFrame(produtos_links)
df.to_csv('URLs_por_Produto.csv', index=False)
```

- Links são salvos em `URLs_por_Produto.csv`

---

## 🧮 4. Estrutura dos Datasets

São utilizados **4 dicionários principais**:

```python
dataset1 = {'Categoria': [], 'Produto': [], 'Marca': [], 'Preco': [], 'Avaliacao': [], 'Quant. Avaliacoes': [], 'Quant. Comentários': [], 'URL': []}
dataset2 = {'URL': [], 'Comentários': []}
dataset3 = {'Característica': [], 'Aval.Característica': [], 'URL': []}
nao_capturado = []
```

---

## 🔍 5. Loop de extração por URL

```python
for pag in product_links:
    try:
        req_books = requests.get(pag)
        conteudo_books = BeautifulSoup(req_books.text, 'html.parser')
        ...
    except Exception as e:
        nao_capturado.append(pag)
```

Cada URL passa pelas seguintes etapas:

### 🛒 Informações gerais

- `produto`: título da página
- `preco`: valor extraído de `<meta itemprop="price">`
- `categoria`: breadcrumb final
- `avaliacao`: média de notas
- `quant_avaliacao`: quantidade de avaliações
- `quant_comentario`: quantidade de comentários (se existir)
- `marca`: varre as tabelas para encontrar linha com título "Marca"

### 🗣️ Comentários

Dois fluxos possíveis:

1. **Com modal (iframe)**: usa Selenium para abrir comentários em página nova  
2. **Sem modal**: extrai diretamente com BeautifulSoup

### 📊 Características avaliadas

- Encontra tabelas com classe `ui-review-capability-categories__desktop--row`
- Extrai: `Característica`, `Avaliação`, e URL

---

## 📉 6. Controle de Requisições

```python
requisicoes += 1
if requisicoes == 100:
    time.sleep(1805)
```

Evita bloqueio do servidor fazendo pausa de 30 minutos a cada 100 requisições.

---

## 📤 7. Exportação dos resultados

```python
pd.DataFrame(dataset1).to_csv('df1.csv', sep=';', encoding='utf-8-sig', index=False)
pd.DataFrame(dataset2).to_csv('df2.csv', sep=';', encoding='utf-8-sig', index=False)
pd.DataFrame(dataset3).to_csv('df3.csv', sep=';', encoding='utf-8-sig', index=False)
pd.DataFrame(nao_capturado).to_csv('urls_faltantes.csv', sep=';')
```

---

## 🧠 Observações Técnicas

- A captura de algumas informações podem falhar devido a falha ou demora de carregamento do html. Isso é tratado com tentativas repetidas.
- A rolagem infinita é simulada via `PAGE_DOWN` e checagem de `scrollTop`.
- Botão “Ver mais” de comentários é clicado com `Selenium` e `ActionChains`.

---

## ✅ Sugestões de Refatoração

- Modularizar cada etapa (coleta de links, coleta de dados, coleta de comentários)
- Centralizar controle de exceções e log
- Criar arquivos `.py` separados para scraping e processamento
- Parametrizar caminho do driver e URL de origem
- Adicionar `try/except` com tipos de exceção específicas
- Otimizar checagem de “Marca” com regex

---

## 🧪 Testes e Validação

- Testado com dezenas de produtos da categoria PET
- URLs com falhas são salvas em `urls_faltantes.csv`
- Código resiste a pequenas variações de layout

---

# Documentação do Pré-processamento do Dataset `df1`

Este documento descreve o tratamento aplicado ao dataset `df1`, desenvolvido em Apache Spark com PySpark. O objetivo foi limpar, padronizar e preparar os dados para análises posteriores.

## Sumário

- [1. Carregamento dos Dados](#1-carregamento-dos-dados)
- [2. Padronização de Variáveis e Valores](#2-padronização-de-variáveis-e-valores)
- [3. Tratamento de Campos Nulos](#3-tratamento-de-campos-nulos)
- [4. Remoção de Registros Duplicados](#4-remoção-de-registros-duplicados)
- [5. Tratamento da Variável `Categoria`](#5-tratamento-da-variável-categoria)
- [6. Tratamento da Variável `Marca`](#6-tratamento-da-variável-marca)
- [7. Substituição da Coluna `URL` por `ID`](#7-substituição-da-coluna-url-por-id)
- [8. Exportação do Dataset](#8-exportação-do-dataset)

---

## 1. Carregamento dos Dados

- Inicialização da SparkSession.
- Leitura do dataset `df1.csv` com `;` como delimitador e cabeçalho ativado.

## 2. Padronização de Variáveis e Valores

- Renomeadas colunas para evitar caracteres especiais.
- Substituídos caracteres não numéricos em `Quant_Comentarios`.
- Tipos de dados convertidos para `FloatType` e `IntegerType` quando aplicável.

## 3. Tratamento de Campos Nulos

- Identificação de valores nulos por coluna.
- Substituição de nulos nas colunas `Quant_Avaliacoes` e `Quant_Comentarios` por 0.
- Substituição de nulos na coluna `Marca` por `'Nao Informado'`.

## 4. Remoção de Registros Duplicados

- Eliminação de registros duplicados com `dropDuplicates()`.

## 5. Tratamento da Variável `Categoria`

- Normalização de valores de `Categoria` com base em palavras-chave presentes nas strings.
- Unificação de categorias similares para agrupamentos mais consistentes.
- Consideração também de valores presentes na coluna `Produto`.

## 6. Tratamento da Variável `Marca`

- Correções e padronizações nos valores da coluna `Marca`.
- Exclusão de termos genéricos ou inválidos substituindo-os por `'Nao Informado'`.

## 7. Substituição da Coluna `URL` por `ID`

- Leitura do dataset `IDs_URLs.csv`.
- Junção com o dataset tratado com base na coluna `URL`.
- Remoção da coluna `URL`.

## 8. Exportação do Dataset

- Escrita final dos dados tratados no formato `.parquet` com o nome `df1_tratado.parquet`.

---

## Considerações Finais

O dataset `df1` passou por um processo completo de ETL (Extract, Transform, Load), garantindo:

- Dados limpos e estruturados;
- Tipos de dados coerentes para análise;
- Categorização uniforme e padronizada;
- Base pronta para integração e uso em pipelines de análise de dados ou machine learning.

# Documentação de Tratamento de Dados — Dataset `df2`

## 📌 Objetivo
Este notebook realiza o tratamento do dataset `df2.csv`, contendo informações de comentários de produtos. O tratamento tem como finalidade a limpeza dos dados e a substituição da variável `URL` pela variável `ID`, utilizando uma tabela auxiliar.

---

## ⚙️ Etapas do Processamento

### 1. Criação da SparkSession
```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("Prata3").getOrCreate()
```
Inicialização do ambiente Spark para manipulação dos dados.

---

### 2. Importações
```python
from pyspark.sql.functions import col, sum, count, when, countDistinct
```
Funções auxiliares para operações de filtragem, contagem e renomeação de colunas.

---

### 3. Leitura dos Dados
```python
df = spark.read.csv("dados\df2.csv", sep=";", header=True)
```
Leitura do dataset `df2.csv` com separador `;` e cabeçalho.

---

### 4. Visualização e Contagem Inicial
```python
df.show()
df.count()
```
Visualização inicial dos dados e contagem do número total de registros.

---

### 5. Renomeação de Coluna
```python
df = df.withColumnRenamed("Comentários", "Comentarios")
```
Padronização do nome da coluna `Comentários` para `Comentarios`.

---

### 6. Verificação de Valores Nulos
```python
df.filter(col("Comentarios").isNull()).count()
df.select(count(when(df["Comentarios"].isNull(), True)).alias("Quant_Nulos")).show()
```
Contagem de registros com valores nulos na coluna `Comentarios`.

---

### 7. Remoção de Registros com Nulos
```python
df_sem_nulos = df.dropna()
df_sem_nulos.count()
df_sem_nulos.select(count(when(col("Comentarios").isNull(),True)).alias("Quant_Nulos")).show()
```
Remoção de registros com valores nulos e verificação pós-tratamento.

---

### 8. Verificação de Unicidade da Coluna `URL`
```python
df.select(countDistinct(col("URL"))).show()
df_sem_nulos.select(countDistinct(col("URL"))).show()
```
Verificação da quantidade de URLs distintas antes e depois do tratamento.

---

### 9. Substituição da Coluna `URL` por `ID`
```python
id_urls = spark.read.csv("dados\IDs_URLs.csv", sep=";", header=True)
df2_tratado = id_urls.join(df_sem_nulos, on="URL", how="inner").drop("URL")
```
Utilização do dataset `IDs_URLs.csv` para substituir a coluna `URL` pela `ID` correspondente.

---

### 10. Escrita do Dataset Final
```python
df2_tratado.write.parquet("dados\df2_tratado.parquet")
```
Salvamento do dataset tratado em formato `.parquet`.

---

### 11. Verificação Final do Schema
```python
df2_tratado.printSchema()
```
Visualização do schema final para conferência dos tipos e nomes das colunas.

---

## ✅ Resultado
O dataset `df2` foi tratado com sucesso, resultando em um arquivo `df2_tratado.parquet` sem valores nulos e com a substituição de `URL` por `ID`.


