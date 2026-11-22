# 🍽️ Análise do Novo Cardápio de um Restaurante com MySQL

Projeto de análise de dados em **MySQL** para avaliar o desempenho do **novo cardápio** de um restaurante. A partir da tabela de itens do menu (`menu_items`) e dos detalhes dos pedidos (`order_details`), o objetivo é descobrir **quais itens estão indo bem e quais não**, e **o que os clientes mais assíduos parecem preferir**, apoiando decisões sobre o que manter, ajustar ou retirar do cardápio.

---

## 📑 Sumário

- [Contexto e objetivos](#-contexto-e-objetivos)
- [Perguntas de negócio](#-perguntas-de-negócio)
- [Tecnologias](#-tecnologias)
- [Modelo de dados](#-modelo-de-dados)
- [Estrutura do projeto](#-estrutura-do-projeto)
- [Como executar](#-como-executar)
- [Etapas da análise](#-etapas-da-análise)
- [Exemplos de consultas](#-exemplos-de-consultas)
- [Principais resultados](#-principais-resultados)
- [Limitações](#-limitações)
- [Próximos passos](#-próximos-passos)
- [Autora](#-autora)

---

## 🎯 Contexto e objetivos

O restaurante reformulou o cardápio e quer entender como os clientes reagiram. Este projeto usa SQL para:

- Medir a **popularidade e a receita** de cada item e de cada categoria.
- Identificar os itens de **melhor e pior desempenho** no novo menu.
- Entender o **comportamento dos clientes mais assíduos** e o que eles mais pedem.
- Gerar recomendações práticas sobre o que manter, destacar, reprecificar ou retirar.

## ❓ Perguntas de negócio

**Desempenho do cardápio**
- Quantos itens e categorias existem no menu? Qual a faixa de preços de cada categoria?
- Quais são os itens **mais e menos pedidos**?
- Quais itens e categorias geram **mais receita**?
- Existem itens que **vendem muito mesmo sendo caros** (ou vendem pouco mesmo sendo baratos)?
- Há itens do menu que **nunca foram pedidos**?

**Comportamento dos clientes**
- Quais são os **pedidos de maior valor** e o que eles contêm?
- Quais categorias e itens predominam nos pedidos **maiores** (perfil dos clientes mais assíduos / que mais consomem)?
- Quais itens costumam aparecer **juntos** no mesmo pedido?

**Tempo**
- Como os pedidos se distribuem por **dia da semana**, **horário** e **mês**?

## 🛠️ Tecnologias

- **MySQL 8.0+** — banco de dados e consultas
- **SQL** — `JOIN`, agregações, subconsultas, CTEs e funções de janela
- **MySQL Workbench** (ou DBeaver) — execução e exploração das queries

## 🗂️ Modelo de dados

```
menu_items ──< order_details
 (item)         (cada linha = 1 item pedido em um pedido)
```

### `menu_items` — itens do cardápio

| Coluna | Descrição |
| --- | --- |
| `menu_item_id` | Identificador único do item (chave primária) |
| `item_name` | Nome do prato/bebida |
| `category` | Categoria do item (ex.: tipo de culinária) |
| `price` | Preço do item |

### `order_details` — itens de cada pedido

| Coluna | Descrição |
| --- | --- |
| `order_details_id` | Identificador único da linha do pedido (chave primária) |
| `order_id` | Identificador do pedido (um pedido pode ter vários itens) |
| `order_date` | Data do pedido |
| `order_time` | Hora do pedido |
| `item_id` | Item pedido (chave estrangeira → `menu_items.menu_item_id`) |

> Cada linha de `order_details` representa **um item** em um pedido. O valor de um pedido é a soma dos preços dos itens que compartilham o mesmo `order_id`. Itens do menu sem correspondência em `order_details` nunca foram vendidos, e linhas com `item_id` nulo indicam pedidos sem item registrado.

## 📁 Estrutura do projeto

```
analise-cardapio-restaurante/
├── data/
│   ├── menu_items.csv
│   └── order_details.csv
├── sql/
│   ├── 01_criar_tabelas.sql       # Criação do banco e das tabelas
│   ├── 02_importar_dados.sql      # Carga dos CSVs
│   ├── 03_explorar_menu.sql       # Exploração da tabela menu_items
│   ├── 04_explorar_pedidos.sql    # Exploração da tabela order_details
│   └── 05_analise_clientes.sql    # Pedidos maiores e perfil dos clientes assíduos
├── docs/
│   └── resultados.md              # Resumo dos achados e recomendações
└── README.md
```

## 🚀 Como executar

### Pré-requisitos

- MySQL Server 8.0 ou superior
- MySQL Workbench, DBeaver ou o cliente `mysql`

### Passo a passo

```bash
# 1. Clone o repositório
git clone https://github.com/<seu-usuario>/analise-cardapio-restaurante.git
cd analise-cardapio-restaurante

# 2. Crie o banco e as tabelas
mysql -u root -p < sql/01_criar_tabelas.sql

# 3. Importe os dados
mysql -u root -p --local-infile=1 restaurant_db < sql/02_importar_dados.sql

# 4. Rode as análises na ordem
mysql -u root -p restaurant_db < sql/03_explorar_menu.sql
mysql -u root -p restaurant_db < sql/04_explorar_pedidos.sql
mysql -u root -p restaurant_db < sql/05_analise_clientes.sql
```

> 💡 Se a importação de CSV falhar, habilite `local_infile` (`SET GLOBAL local_infile = 1;`) ou use o **Table Data Import Wizard** do MySQL Workbench.

## 🧭 Etapas da análise

1. **Exploração do menu** — quantidade de itens, categorias, preço mínimo, máximo e médio por categoria.
2. **Exploração dos pedidos** — período coberto, total de pedidos, total de itens vendidos, média de itens por pedido.
3. **Junção das tabelas** — `LEFT JOIN` entre `order_details` e `menu_items` para tratar itens sem registro e itens nunca vendidos.
4. **Desempenho dos itens** — ranking por quantidade e por receita, itens mais e menos vendidos.
5. **Análise de clientes assíduos** — pedidos de maior valor, categorias e itens mais frequentes nesses pedidos.
6. **Recomendações** — o que manter, destacar, reprecificar ou remover.

## 🔍 Exemplos de consultas

**Itens e preço por categoria**

```sql
SELECT
    category,
    COUNT(*)            AS qtd_itens,
    MIN(price)          AS preco_min,
    ROUND(AVG(price),2) AS preco_medio,
    MAX(price)          AS preco_max
FROM menu_items
GROUP BY category
ORDER BY preco_medio DESC;
```

**Itens mais e menos pedidos (incluindo os nunca vendidos)**

```sql
SELECT
    m.item_name,
    m.category,
    m.price,
    COUNT(o.order_details_id) AS qtd_pedidos
FROM menu_items m
LEFT JOIN order_details o ON o.item_id = m.menu_item_id
GROUP BY m.menu_item_id, m.item_name, m.category, m.price
ORDER BY qtd_pedidos DESC;
```

**Receita por item e por categoria**

```sql
SELECT
    m.category,
    m.item_name,
    COUNT(*)     AS qtd_vendida,
    SUM(m.price) AS receita
FROM order_details o
JOIN menu_items m ON m.menu_item_id = o.item_id
GROUP BY m.category, m.item_name
ORDER BY receita DESC;
```

**Pedidos de maior valor**

```sql
SELECT
    o.order_id,
    COUNT(*)     AS qtd_itens,
    SUM(m.price) AS valor_total
FROM order_details o
JOIN menu_items m ON m.menu_item_id = o.item_id
GROUP BY o.order_id
ORDER BY valor_total DESC
LIMIT 10;
```

**O que aparece nos pedidos de maior valor (perfil dos clientes que mais consomem)**

```sql
WITH pedidos_top AS (
    SELECT o.order_id
    FROM order_details o
    JOIN menu_items m ON m.menu_item_id = o.item_id
    GROUP BY o.order_id
    ORDER BY SUM(m.price) DESC
    LIMIT 100
)
SELECT
    m.category,
    m.item_name,
    COUNT(*) AS vezes_pedido
FROM order_details o
JOIN pedidos_top t ON t.order_id = o.order_id
JOIN menu_items m  ON m.menu_item_id = o.item_id
GROUP BY m.category, m.item_name
ORDER BY vezes_pedido DESC;
```

**Itens pedidos juntos com frequência**

```sql
SELECT
    a.item_id AS item_a,
    b.item_id AS item_b,
    COUNT(*)  AS vezes_juntos
FROM order_details a
JOIN order_details b
  ON a.order_id = b.order_id
 AND a.item_id  < b.item_id
GROUP BY a.item_id, b.item_id
ORDER BY vezes_juntos DESC
LIMIT 10;
```

## 📊 Principais resultados

> Preencha com os achados reais da sua análise. Sugestão de formato:

- 🏆 **Itens campeões:** __, __ e __ lideram em volume de pedidos.
- 📉 **Itens com baixo desempenho:** __ e __ vendem pouco e são candidatos a revisão ou retirada.
- 💰 **Categoria com maior receita:** __ (__% do total).
- 👥 **Clientes de maior consumo:** costumam pedir __ e __, com preferência pela categoria __.
- 🔗 **Combinações frequentes:** __ + __.
- ✅ **Recomendações:** manter __, destacar __, reprecificar __, retirar __.

## ⚠️ Limitações

- As tabelas **não trazem identificação de cliente**. Por isso, os "clientes mais assíduos" são analisados por aproximação, usando os **pedidos de maior valor e maior volume de itens** (`order_id`). Se houver uma coluna de cliente em outra tabela, a análise pode ser refinada por frequência de visitas.
- Itens com `item_id` nulo em `order_details` não puderam ser associados ao cardápio e foram tratados à parte.
- A análise cobre apenas o período presente nos dados e não inclui custos, então olha para **receita**, não para **lucro**.

## 🗺️ Próximos passos

- [ ] Comparar o desempenho do novo cardápio com o cardápio anterior
- [ ] Analisar sazonalidade (dia da semana e horário) por categoria
- [ ] Criar **views** com os principais indicadores
- [ ] Construir um **dashboard** (Power BI, Metabase ou similar)
- [ ] Incluir dados de custo para analisar **margem** por item

## 👩‍💻 Autora

Desenvolvido por **Bruna Luísa** — [GitHub](https://github.com/BrunaLuisaCRS)

---

> *Os dados utilizados têm finalidade educacional.*
