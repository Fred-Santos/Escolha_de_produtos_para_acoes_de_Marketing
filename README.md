# 🐾 Projeto PetLovers – Data Pipeline e Análise Estratégica de E-commerce Pet  

## 📌 Descrição Geral  
Este projeto simula uma parceria com a **PetLovers**, um e-commerce fictício especializado em produtos para cães, com o objetivo de resolver um problema clássico de marketing digital:

> **"Como entender o que realmente influencia a percepção dos clientes sobre os produtos — preço, marca, categoria ou qualidade percebida?"**

A solução foi construir um **pipeline de dados completo**, capaz de transformar dados soltos (comentários, avaliações e características dos produtos) em **informações acionáveis** que orientem a equipe de marketing na tomada de decisões.

---

## 🎯 Objetivos do Projeto
- Identificar **produtos com maior potencial de destaque**;
- Detectar **pontos fortes e fracos** em marcas e categorias;
- Relacionar percepção dos clientes com **preço, volume de comentários e avaliações**;
- Criar uma solução analítica escalável e reaproveitável.

---

## ⚙️ Arquitetura do Pipeline

### 1. Coleta de Dados (Web Scraping)
- **Tecnologias**: `Selenium`, `Requests`, `BeautifulSoup`
- **Dados extraídos**:
  - Nome e descrição do produto
  - Preço
  - Categoria
  - Marca
  - Avaliação média
  - Número de avaliações e comentários
  - URL
  - E outros para análises qualitativas posteriores

### 2. Processamento e Limpeza
- **Tecnologia**: `PySpark`
- **Tratamentos aplicados**:
  - Padronização de textos
  - Conversão de tipos
  - Remoção de inconsistências
  - Tratamento de nulos e duplicados

### 3. Feature Engineering
Variáveis derivadas criadas para enriquecer a análise:
- `avaliacao_ponderada`
- `escore_engajamento = log1p(avaliações + comentários)`
- `escore_custo_beneficio`
- `faixa_preco` (agrupamento por tercis)
- `produto_destaque` (flag binária)

### 4. Modelagem Dimensional
- **Modelo em estrela** com:
  - Fato: `fato_avaliacoes_produto`
  - Dimensões: `dim_produto`, `dim_marca`, `dim_categoria`, `dim_caracteristicas`, `dim_url`

### 5. Visualização Interativa
- **Ferramenta**: `Power BI`
- **Principais visões**:
  - Análise por faixa de preço
  - Marcas e categorias melhor/pior avaliadas
  - Produtos mais engajados, comentados e avaliados
  - Indicadores agregados por dimensão

---

## 📊 Principais Métricas Criadas

| Métrica                   | Descrição                                                                 |
|---------------------------|---------------------------------------------------------------------------|
| `avaliacao_ponderada`     | Corrige distorções de produtos com poucas avaliações                     |
| `escore_engajamento`      | Mede a interação real do público com cada produto                        |
| `escore_custo_beneficio`  | Avalia qualidade percebida em relação ao preço                           |
| `faixa_preco`             | Agrupamento por tercis para análise comparativa                          |
| `produto_destaque`        | Flag binária para destacar produtos com boa percepção e interação        |

---

## 📈 Insights Estratégicos

### 🐶 Produto com Maior Destaque: **Ração Premier**
- Nota próxima de **5,0**
- Alto volume de avaliações e comentários
- Presente nos rankings de melhor avaliação, maior engajamento e mais comentado

**💡 Recomendação**: Usar como produto vitrine, em kits ou campanhas promocionais.

---

### 🏷️ Marca com Maior Destaque: **Premier**
- Top 3 marcas mais bem avaliadas (~4,98)
- Alta presença no portfólio e forte percepção de valor
- Ticket médio intermediário-alto

**💡 Recomendação**: Parcerias comerciais, destaque em campanhas e kits premium.

---

### 📦 Categorias com Potencial e Risco

| Categoria        | Avaliação Alta | Engajamento | Ação Recomendada                                     |
|------------------|----------------|-------------|------------------------------------------------------|
| Ração, Sachês    | ✅             | ✅          | Investir em campanhas, usar como referência positiva |
| Antipulgas       | ❌             | ❌          | Reavaliar mix, mensagens e portfólio                |
| Escada Pet       | ✅             | ❌          | Justificar valor com comunicação de benefícios       |

---

## 🧠 Soluções Entregues

- Pipeline completo e escalável: **Coleta → Processamento → Enriquecimento → Visualização**
- Modelo dimensional com métricas estratégicas
- Dashboard com foco em **ações de marketing orientadas por dados**
- Insights claros sobre percepção de **categorias, marcas e produtos**

---

## 🔄 Próximos Passos

- Aplicar **Análise de Sentimentos (NLP)** nos comentários para entender atributos qualitativos;
- Explorar relações entre métricas compostas e comportamento do consumidor;
- Evoluir para análises preditivas com base nas variáveis já criadas.

---

## 📂 Sobre o Projeto

- **Tipo**: Projeto autoral / portfólio de Engenharia e Análise de Dados
- **Foco**: Estratégia de marketing orientada por dados em e-commerce
- **Ferramentas**: Python, PySpark, Power BI
- **Status**: Etapa 1 (quantitativa) concluída ✅ | Etapa 2 (qualitativa) em planejamento

---


