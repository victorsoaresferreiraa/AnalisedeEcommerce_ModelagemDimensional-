Análise de E-commerce — Modelagem Dimensional em Power BI
Projeto desenvolvido como desafio técnico para vaga de Analista de Dados (RZK Digital), usando o Brazilian E-Commerce Public Dataset (Olist), disponível no Kaggle.

Objetivo
Modelar os dados de pedidos, produtos e vendedores de um marketplace em um esquema dimensional, e responder a perguntas de negócio sobre as melhores categorias e vendedores por ano, sob três perspectivas: faturamento, volume de itens vendidos e nota média de avaliação.

Status do projeto
✅ Modelagem dimensional completa (esquema estrela)
✅ Transformações no Power Query
✅ Medidas DAX para as métricas de negócio pedidas
🔧 Visualização em progresso — a estrutura de dados e os cálculos estão prontos e validados; a camada de gráficos/dashboard ainda está em refinamento
Optei por documentar e entregar o projeto no estágio atual em vez de atrasar a entrega — a modelagem e a lógica de negócio (a parte que considero mais crítica de acertar) estão completas e testadas.

Fonte de dados
6 tabelas do dataset Olist foram usadas: orders, order_items, order_reviews, products, sellers e product_category_name_translation.

Modelagem — Esquema Estrela
Tabela Fato: ft_vendas
Grão: uma linha por item vendido (nível de order_item).

Coluna	Origem	Descrição
order_id	orders	Identificador do pedido
order_item_id	order_items	Identificador do item dentro do pedido
product_id	order_items	FK → dm_produtos
seller_id	order_items	FK → dm_sellers
date_key	orders (order_purchase_timestamp)	FK → dm_calendario
price	order_items	Valor do item
freight_value	order_items	Valor do frete
review_score	order_reviews (agregado)	Nota média da avaliação do pedido
Dimensões
dm_produtos: product_id, product_category_name, product_category_name_english (via merge com a tabela de tradução)

dm_sellers: seller_id, seller_city, seller_state

dm_calendario: date_key, year, month, month_name, quarter — tabela de calendário gerada via M (Power Query), cobrindo o intervalo de datas do dataset, para permitir hierarquias de tempo (ano/trimestre/mês) sem depender diretamente da granularidade bruta da fato.

Decisões de modelagem que valem destacar
Review score por pedido, não por item: a tabela original de reviews é por order_id. Antes de juntar na fato, agrupei por order_id tirando a média (Group By), evitando duplicar linhas de itens quando um pedido tem mais de uma avaliação.
Junção "Esquerda Externa" em todos os merges: garante que nenhum item de venda seja perdido caso não exista review ou dado de data correspondente.
Medidas DAX
Faturamento = SUM(ft_vendas[price])

Itens Vendidos = COUNTROWS(ft_vendas)

Pedidos Distintos = DISTINCTCOUNT(ft_vendas[order_id])

Nota Media = AVERAGE(ft_vendas[review_score])

Ticket Medio por Seller = DIVIDE([Faturamento], [Pedidos Distintos])
Por que Ticket Médio usa Pedidos Distintos e não Itens Vendidos: ticket médio mede o valor de uma "cesta de compra" — um pedido com 3 itens do mesmo cliente é uma venda, não três. Usar COUNTROWS (contagem de itens) no denominador infla artificialmente o número de "vendas" e sub-representa o ticket médio real. Esse foi o ponto de modelagem que mais me exigiu atenção no projeto.

Principais aprendizados
A diferença entre coluna calculada (fixa, calculada linha a linha) e medida (recalculada dinamicamente pelo contexto de filtro) foi o conceito que mais me custou entender no início, e o que mais uso agora.
Modelar o grão correto da tabela fato antes de escrever qualquer medida evita retrabalho — comecei a modelagem duas vezes até fixar o grão certo (item vendido, não pedido).
A camada de Power Query/modelagem se aproxima muito de um processo de ETL que já conhecia de outros projetos (Airflow/dbt); a camada de visualização é uma disciplina separada, mais próxima de design de informação, na qual ainda estou evoluindo.
Próximos passos
Finalizar os visuais das 3 páginas do relatório (visão geral, categorias por ano, sellers por ano)
Aplicar a paleta de cores institucional da RZK Digital
