# PetLovers
🚀 Apresento um projeto simulando uma colaboração com a PetLovers, um e-commerce (fictício) especializado em produtos para cães.
A PetLovers enfrentava um desafio comum em empresas digitais: entender, de forma estruturada, como seus clientes percebem os produtos. Embora algumas categorias registrassem boas vendas, a equipe ainda não compreendia claramente os fatores que impulsionavam esse desempenho: preço, marca, qualidade percebida ou outros elementos?
🧩 O principal problema identificado foi a falta de visibilidade consolidada sobre a opinião dos consumidores, o que limitava a atuação estratégica da equipe de marketing.
💡 Neste projeto, construí um pipeline de dados completo — da extração (via web scraping) até o tratamento, enriquecimento e modelagem — com foco em transformar dados dispersos (como comentários, avaliações e características dos produtos) em insights acionáveis.
A entrega final será um dashboard interativo, voltado para a equipe de marketing. Esse dashboard será uma ferramenta estratégica essencial para a equipe de marketing, transformando dados dispersos em decisões mais inteligentes e eficazes permitindo:
Identificar produtos com alto potencial de destaque;
Detectar padrões de satisfação e insatisfação por marca e categoria;
Relacionar percepção do cliente com preço e volume de avaliações.
Entre outros.
🔍 Como transformar dados soltos em inteligência para decisões de marketing?

📌 1. Coleta de Dados
 Utilizei técnicas de web scraping com Selenium, Requests e BeautifulSoup para extrair informações de produtos voltados para cães no site da PetLovers — incluindo preços, avaliações, comentários e descrições.
🧹 2. Processamento e Limpeza
 Com PySpark, realizei a padronização dos dados e tratamento de inconsistências. Essa etapa foi crucial para garantir a integridade da análise.
🔬 3. Enriquecimento de Dados
 Criei novas variáveis derivadas e features úteis para análise — como média ponderada de avaliações e classificação de produtos por desempenho — documentadas em um book de variáveis.
📦 4. Armazenamento Final
 Adotei uma estrutura em camadas para organização dos dados:
Raw (dados brutos),
Processed (dados limpos e organizados),
Curated (dados prontos para análise).
📊 5. Visualização e Análise
 O pipeline termina em um dashboard interativo no Power BI, desenhado para oferecer à equipe de marketing uma visão estratégica com insights sobre:
Produtos com alto potencial de destaque;
Marcas e categorias com maior (ou menor) satisfação;
A relação entre preço, avaliações e percepção dos clientes.
Essa estrutura permite à PetLovers transformar dados desconectados em ações orientadas por evidências.
🔎 Como coletei os dados para transformar opiniões soltas em inteligência de marketing?
O desafio era obter dados detalhados de diversos produtos – incluindo preços, marcas, avaliações, comentários e percepções específicas dos consumidores – de forma automatizada, confiável e escalável.
💻 Para isso, construí um processo robusto de web scraping, utilizando:
📌 Selenium + BeautifulSoup + Requests
Combinando essas três ferramentas, consegui navegar por múltiplas páginas do site, simular interações (como rolagem e cliques), capturar dados estruturados e não estruturados e tratar variações na resposta das páginas.
📂 O scraping resultou em quatro conjuntos de dados principais:
Informações gerais dos produtos (nome, categoria, marca, preço, nota média, etc.).
Comentários dos clientes, capturados até mesmo dentro de modais e iframes dinâmicos.
Avaliação por características (ex: durabilidade, custo-benefício).
URLs que não responderam — registradas para controle de qualidade e tentativa futura.
⚠️ Além disso, adicionei controle de fluxo para evitar bloqueios por excesso de requisições, com pausas estratégicas a cada 100 produtos extraídos.
Essa etapa é fundamental: sem dados confiáveis, não há análise relevante. Todo o pipeline posterior (limpeza, enriquecimento, visualização) depende de uma coleta bem feita — e, nesse caso, o foco foi garantir profundidade, diversidade e precisão dos dados extraídos.
