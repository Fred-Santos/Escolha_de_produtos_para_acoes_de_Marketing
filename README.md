🐾 Projeto PetLovers – Data Pipeline e Análise Estratégica de E-commerce Pet
📌 Descrição Geral
Este projeto simula uma parceria com a PetLovers, um e-commerce fictício especializado em produtos para cães, com o objetivo de resolver um problema clássico de marketing digital:

Como entender o que realmente influencia a percepção dos clientes sobre os produtos — preço, marca, categoria ou qualidade percebida?

A solução foi construir um pipeline de dados completo, capaz de transformar dados soltos (comentários, avaliações e características dos produtos) em informações acionáveis que orientem a equipe de marketing na tomada de decisões.

🎯 Objetivos do Projeto
Identificar produtos com maior potencial de destaque;

Detectar pontos fortes e fracos em marcas e categorias;

Relacionar percepção dos clientes com preço, volume de comentários e avaliações;

Criar uma solução analítica escalável e reaproveitável.

⚙️ Arquitetura do Pipeline
1. Coleta de Dados (Web Scraping)
Tecnologias: Selenium, Requests, BeautifulSoup

Dados extraídos:

Nome e descrição do produto

Preço

Categoria

Marca

Avaliação média

Número de avaliações e comentários

URL

2. Processamento e Limpeza
Tecnologia: PySpark (processamento distribuído)

Etapas:

Padronização de textos

Conversão de tipos

Remoção de inconsistências

Tratamento de nulos e duplicados

3. Feature Engineering
Variáveis derivadas criadas para enriquecer a análise:

avaliacao_ponderada: pondera a média conforme o volume de avaliações

escore_engajamento: log1p(avaliações + comentários) → suaviza outliers

escore_custo_beneficio: avalia qualidade percebida em relação ao preço

faixa_preco: segmentação por tercis

produto_destaque: flag para produtos com avaliação + engajamento acima da média

4. Modelagem Dimensional
Modelo em estrela com:

Tabela fato: fato_avaliacoes_produto

Tabelas dimensão: dim_produto, dim_marca, dim_categoria, dim_caracteristicas, dim_url

5. Visualização Interativa
Ferramenta: Power BI

Dashboard estratégico para uso da equipe de marketing com:

Análises por faixa de preço

Marcas e categorias mais bem/mais mal avaliadas

Produtos com maior engajamento, avaliações e percepção de valor

Indicadores agregados por dimensão

📊 Principais Métricas Criadas
Métrica	Descrição
avaliacao_ponderada	Corrige distorções de produtos com poucas avaliações
escore_engajamento	Mede a interação real do público com cada produto
escore_custo_beneficio	Combina percepção (avaliação) com preço, destacando bom custo-benefício
faixa_preco	Agrupamento por tercis para análise comparativa
produto_destaque	Flag binária para destacar produtos acima da média em avaliação e engajamento

📈 Insights Estratégicos Gerados
🔹 Produtos em Destaque
Ração Premier: Nota próxima de 5,0, alto volume de feedbacks e excelente engajamento → Produto vitrine ideal.

Tapetes Higiênicos: Engajamento altíssimo e presença frequente entre os mais comentados.

🔹 Marcas em Evidência
Premier: Alta avaliação, alto volume e ticket médio premium → Parceria recomendada para campanhas.

Golden e Nestlé Purina: Também entre as mais bem avaliadas com forte aceitação de mercado.

🔹 Categorias com potencial e riscos
Ração, Sachês e Casinhas: Avaliações altas + engajamento → Investimento recomendado.

Antipulgas, Repelentes e Sabonetes: Avaliações abaixo da média e pouco engajamento → Reavaliação sugerida.

🧠 Soluções Entregues
Criação de um data mart analítico estruturado e escalável;

Desenvolvimento de um dashboard executivo com foco em marketing;

Extração de insights estratégicos por produto, marca e categoria;

Geração de valor real para decisões de campanha, posicionamento e curadoria de portfólio.

🔄 Próximos Passos
Aplicação de Análise de Sentimentos (NLP) nos comentários dos clientes para entender qualitativamente o que motiva avaliações positivas e negativas;

Evolução para um modelo preditivo (ex: prever percepção com base em atributos).

📂 Sobre o Projeto
Status: Concluído – Etapa 1 (Análise Quantitativa)

Fase atual: Preparação para Análise de Sentimentos

Tipo: Projeto autoral (portfólio acadêmico)

Tema: E-commerce, produtos pet, engenharia e análise de dados
