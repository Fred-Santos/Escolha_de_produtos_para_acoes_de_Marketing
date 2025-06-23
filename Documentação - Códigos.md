
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

## ✅ Futuras ações

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

# 📄 Documentação  Geração de Índice de URLs `IDs_URLs.csv`

## 🎯 Objetivo

Este notebook tem como objetivo gerar um índice único para cada URL dos datasets extraídos via web scraping, possibilitando uma substituição consistente da coluna `URL` por `ID` em múltiplos datasets (df1, df2, df3).

---

## 📦 Instalação de Dependência

```python
get_ipython().system('pip install pyspark')
```

Instala o pacote PySpark, caso ainda não esteja presente no ambiente.

---

## 🚀 Inicialização da SparkSession

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("ID_URL").getOrCreate()
```

Criação da sessão Spark com nome `ID_URL`.

---

## 🧰 Importações Auxiliares

```python
from pyspark.sql.functions import countDistinct, col, count, when, monotonically_increasing_id
```

Funções para contagem de valores distintos, verificação de nulos e geração de IDs únicos.

---

## 📥 Leitura dos Dados

```python
df = spark.read.csv("dados\df1.csv", sep=";", header=True)
df.show()
```

Leitura do dataset `df1.csv` contendo as URLs.

---

## 🔍 Extração da Coluna `URL`

```python
df_indice_urls_1 = df.select("URL")
df_indice_urls_1.show()
df_indice_urls_1.count()
df_indice_urls_1.select(countDistinct(col("URL"))).show()
```

Isolamento da coluna `URL`, contagem total e contagem de valores distintos.

---

## 🧼 Verificação de Nulos

```python
df_indice_urls_1.select(count(when(col("URL").isNull(), True))).show()
```

Verifica se há valores nulos na coluna `URL`.

---

## 📥 Leitura dos Datasets Adicionais

```python
df2 = spark.read.csv("dados\df2.csv", sep=";", header=True)
df3 = spark.read.csv("dados\df3.csv", sep=";", header=True)
```

Leitura dos arquivos `df2.csv` e `df3.csv` para comparar URLs.

---

## 🔍 Coleta das URLs dos Datasets df2 e df3

```python
df_indice_urls_2 = df.select("URL")
df_indice_urls_3 = df.select("URL")
```

Extrai-se a coluna `URL` para análise.

---

## 📊 Análise de URLs Duplicadas e Nulas

```python
df_indice_urls_2.count()
df_indice_urls_2.select(countDistinct(col("URL"))).show()
df_indice_urls_2.select(count(when(col("URL").isNull(), True))).show()

df_indice_urls_3.count()
df_indice_urls_3.select(countDistinct(col("URL"))).show()
df_indice_urls_3.select(count(when(col("URL").isNull(), True))).show()
```

Validações semelhantes às realizadas para `df1`, aplicadas a `df2` e `df3`.

---

## 🧾 Conversão de URLs para Lista

```python
lista1 = [row["URL"] for row in urls_1.select("URL").collect()]
lista2 = [row["URL"] for row in urls_2.select("URL").collect()]
lista3 = [row["URL"] for row in urls_3.select("URL").collect()]
```

Converte as URLs distintas de cada dataset para listas Python.

---

## 🔎 Verificação de URLs Não Presentes

```python
urls = [i for i in lista1 if i not in lista2]
urls = [i for i in lista1 if i not in lista3]
```

Identifica URLs presentes em `df1`, mas ausentes em `df2` ou `df3`.

---

## 🆔 Geração de Identificadores Únicos

```python
id_url = urls_1.withColumn("ID", monotonically_increasing_id())
id_url.write.csv("dados\IDs_URLs.csv", sep=";", header=True)
```

Cria uma nova coluna `ID` com identificadores únicos para cada URL distinta e salva o resultado no arquivo `IDs_URLs.csv`.

---

## ✅ Resultado Final

O arquivo `IDs_URLs.csv` contém todas as URLs únicas de `df1` com um identificador exclusivo. Esse arquivo é utilizado para substituir a coluna `URL` por `ID` nos demais datasets.


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


# Documentação de Tratamento de Dados — Dataset `df3`

## 🎯 Objetivo

Realizar o tratamento dos dados contidos no arquivo `df3.csv`, padronizando valores, removendo inconsistências e preparando o dataset para integrações e análises posteriores.

---

## 🔧 Importações

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import count_distinct, col, when, sum, regexp_extract
```

---

## 🚀 Inicialização da SparkSession

```python
spark = SparkSession.builder.appName('Prata2').getOrCreate()
```

Criação da sessão Spark com o nome `Prata2`.

---

## 📥 Leitura dos Dados

```python
df = spark.read.csv("dados\\df3.csv", sep=";", header=True)
df.show()
df.count()
df.printSchema()
```

Leitura do arquivo CSV contendo as avaliações de características dos produtos. Exibe os dados, conta as linhas e imprime o schema.

---

## ✍️ Renomeação de Colunas

```python
df = df.withColumnRenamed("Aval.Característica", "Aval_Caracteristica") \       .withColumnRenamed("Característica", "Caracteristica")
```

Padronização dos nomes de colunas para evitar problemas com caracteres especiais.

---

## 🔍 Análise de Distintos

```python
df.select("Caracteristica").distinct().withColumnRenamed("Caracteristica", "Caracteristiscas_Unicas").show(truncate=False)
df.select(count_distinct(col("Caracteristica")).alias("Caracteristicas_Distintas")).show()
```

Verifica a diversidade de características no dataset.

---

## 🔁 Padronização de Valores Categóricos

```python
df1 = df.withColumn("Caracteristica", when(col("Caracteristica") == "Relación precio-calidad", "Relação preço-qualidade")
                    .otherwise(col("Caracteristica"))) \        .withColumn("Caracteristica", when(col("Caracteristica") == "Calidad de los materiais", "Qualidade dos materiais")
                    .otherwise(col("Caracteristica")))
```

Padroniza valores escritos em espanhol para o português.

---

## ❓ Verificação de Valores Nulos

```python
nulos = df.select([
    sum(col("Aval_Caracteristica").isNull().cast("int")).alias("Quant_Nulos"),
    sum(col("Caracteristica").isNull().cast("int")).alias("Quant_Caracteristica"),
    sum(col("URL").isNull().cast("int")).alias("Quant_URL")
])
nulos.show()
```

Conta os valores nulos em cada coluna.

---

## 🔢 Extração de Números da Coluna de Avaliação

```python
df2 = df.withColumn("Aval_Caracteristica", regexp_extract("Aval_Caracteristica", r"(\d+(\.\d+)?)", 1))
df2.show()
```

Extrai os valores numéricos das strings na coluna `Aval_Caracteristica`.

---

## 🔄 Conversão de Tipos

```python
df3 = df2.withColumn("Aval_Caracteristica", col("Aval_Caracteristica").cast("double"))
df3.printSchema()
df3.show()
df3.count()
```

Converte a coluna de avaliação para tipo `double`.

---

## 🔗 Junção com IDs de URLs

```python
id_urls = spark.read.csv("dados\\IDs_URLs.csv", sep=";", header=True)
df3_tratado = id_urls.join(df3, on="URL", how="inner").drop("URL")
df3_tratado.show()
```

Relaciona os dados tratados com um identificador único por URL, removendo a URL do dataset final.

---

## 💾 Escrita do Dataset Tratado

```python
df3_tratado.write.parquet("dados\\df3_tratado.parquet")
df3_tratado.printSchema()
```

Escreve os dados tratados no formato Parquet para uso futuro.

---

## ✅ Considerações Finais

O dataset `df3` foi padronizado, limpo e enriquecido com identificadores únicos. Isso permite sua integração com outras tabelas e análise em modelos como o modelo estrela.

# 📄 Documentação Construção do Modelo Estrela

## 🎯 Objetivo

Este notebook realiza a transformação e modelagem dos dados coletados via web scraping, estruturando-os em um **modelo estrela** para análises de BI. As etapas envolvem o cálculo de métricas derivadas, categorização de preços, identificação de produtos destaque e a construção de tabelas fato e dimensões.

---

## 📦 Importações

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, count, sum, when, countDistinct, min, max, avg, round, lit, row_number, desc, asc, log1p, percent_rank, upper
from pyspark.sql.window import Window
```

Funções utilizadas para agregações, transformações e janelas de análise.

---

## 🚀 Inicialização da SparkSession

```python
spark = SparkSession.builder.appName('tabelas').getOrCreate()
```

Cria o ambiente Spark necessário para manipulação dos dados.

---

## 📥 Carregamento dos Datasets Tratados

```python
df1 = spark.read.parquet(".../df1_tratado.parquet")
df3 = spark.read.parquet(".../df3_tratado.parquet")
df4 = spark.read.csv(".../IDs_URLs.csv", sep=';', header=True)
```

Leitura dos datasets finais já tratados e prontos para modelagem.

---

## 📊 Avaliação Ponderada

```python
media_avaliacoes_produtos = df1.select(avg(col("Avaliacao"))).collect()[0][0]
m = 10  # valor mínimo de confiança
df_transicao = df1.withColumn(
    'avaliacao_ponderada',
    (col('Quant_Avaliacoes') / (col('Quant_Avaliacoes') + lit(m))) * col('Avaliacao') +
    ((lit(m) / (col('Quant_Avaliacoes') + lit(m))) * lit(media_avaliacoes_produtos)))
```

Calcula uma avaliação mais justa ponderando pela quantidade de avaliações.

---

## 📈 Escore de Engajamento

```python
df_transicao = df_transicao.withColumn(
    'Escore_Engajamento',
    log1p(col('Quant_Avaliacoes') + col('Quant_Comentarios')))
```

Cria um escore baseado na soma de avaliações e comentários com escala logarítmica.

---

## 💰 Escore Custo-Benefício

```python
df_transicao = df_transicao.withColumn(
    'Escore_Custo_Beneficio',
    when(col('Preco') > 0, col('avaliacao_ponderada') / col('Preco')).otherwise(lit(None)))
```

Mede a relação custo-benefício do produto.

---

## 🏷️ Faixa de Preço

```python
quantis = df_transicao.approxQuantile("Preco", [0.33, 0.66], 0.01)
df_transicao = df_transicao.withColumn(
    "Faixa_Preco",
    when(col("Preco") <= quantis[0], "Baixo")
    .when((col("Preco") > quantis[0]) & (col("Preco") <= quantis[1]), "Médio")
    .otherwise("Alto"))
```

Agrupamento de produtos por faixa de preço baseada em quantis.

---

## 🌟 Produto Destaque

```python
mediana_engajamento = df_transicao.approxQuantile("Escore_Engajamento", [0.5], 0.01)[0]
df_transicao = df_transicao.withColumn(
    "Produto_Destaque",
    when((col("avaliacao_ponderada") >= 4.5) & (col("Escore_Engajamento") >= mediana_engajamento), True).otherwise(False))
```

Marca produtos com boa avaliação e alto engajamento como destaque.

---

## 🏁 Criação da Tabela Fato

```python
fato_avaliacoes_produto = df_transicao
```

Tabela principal que consolida todas as métricas calculadas.

---

## 📦 Tabelas de Dimensão

### Produto

```python
dim_produto = df1.select('ID', 'Produto', 'Categoria', 'Marca')
```

### Características

```python
dim_caracteristicas = df3
```

### URLs

```python
dim_url = df4
```

### Marca

```python
dim_marca = fato_avaliacoes_produto.groupBy("Marca").agg(...).withColumn("Faixa_Preco_Marca", ...)
```

### Categoria

```python
dim_categoria = fato_avaliacoes_produto.groupBy("Categoria").agg(...).withColumn("Faixa_Preco_Categoria", ...)
```

Cada dimensão agrega dados por chave descritiva, com métricas e classificações por faixa.

---

## 💾 Escrita dos Dados

```python
fato_avaliacoes_produto.write.parquet(".../fato_avaliacoes_produto")
dim_produto.write.parquet(".../dim_produto")
dim_caracteristicas.write.parquet(".../dim_caracteristicas")
dim_url.write.parquet(".../dim_url")
dim_marca.write.parquet(".../dim_marca")
dim_categoria.write.parquet(".../dim_categoria")
```

Armazena todas as tabelas geradas no formato `.parquet`.

---

## ✅ Resultado Final

Estrutura final em modelo estrela, pronta para análise em ferramentas de BI ou exploração com SparkSQL.
