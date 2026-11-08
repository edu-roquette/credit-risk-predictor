# Projeto Final — Análise Exploratória de Dados Olist

Projeto desenvolvido na disciplina de Técnicas de Programação I, com o objetivo de analisar dados de pedidos, clientes, produtos, pagamentos e entregas do e-commerce brasileiro Olist.

## Integrantes

- Gustavo Pires Bogéa
- Haroé Silva de Jesus
- Hélio Serrano Brandão Junior

## Objetivo

Investigar padrões comerciais e logísticos da operação, respondendo às seguintes perguntas de negócio:

1. Qual a relação entre o custo do frete e o tempo real de entrega entre os estados?
2. Como o valor médio do pedido varia conforme o tipo de pagamento e o parcelamento?
3. Qual o comportamento dos status dos pedidos e do tempo de entrega ao longo do tempo?
4. Clientes recorrentes apresentam comportamento de compra diferente dos clientes pontuais?

## Dados utilizados

Os dados foram obtidos no dataset público da Olist disponibilizado no Kaggle.

Foram utilizadas as seguintes bases:

- `olist_customers_dataset.csv`
- `olist_orders_dataset.csv`
- `olist_order_items_dataset.csv`
- `olist_order_payments_dataset.csv`
- `olist_order_reviews_dataset.csv`
- `olist_products_dataset.csv`
- `product_category_name_translation.csv`

As bases estão localizadas no diretório `dados/`.

## Estrutura do projeto

```text
projeto-tecnicas-programacao-i-py-ada/
├── dados/
│   ├── olist_customers_dataset.csv
│   ├── olist_orders_dataset.csv
│   ├── olist_order_items_dataset.csv
│   ├── olist_order_payments_dataset.csv
│   ├── olist_order_reviews_dataset.csv
│   ├── olist_products_dataset.csv
│   └── product_category_name_translation.csv
├── notebooks/
│   └── analise.ipynb
└── README.md
```

## Tecnologias utilizadas

- Python 3
- Jupyter Notebook
- Pandas
- NumPy
- Matplotlib
- Seaborn

## Instalação

Recomenda-se criar um ambiente virtual:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Instale as dependências:

```bash
pip install pandas numpy matplotlib seaborn jupyter
```

## Execução

Na raiz do projeto, execute:

```bash
jupyter notebook
```

Em seguida, abra o arquivo:

```text
notebooks/analise.ipynb
```

Execute as células na ordem apresentada. Os caminhos utilizados no notebook esperam que os arquivos CSV estejam no diretório `dados/`.

## Principais etapas da análise

O notebook realiza:

1. Importação e inspeção das bases;
2. Verificação de dimensões, tipos, valores ausentes e duplicidades;
3. Validação da granularidade das tabelas;
4. Integração entre pedidos, clientes, itens e produtos;
5. Conversão e tratamento de datas;
6. Criação de variáveis derivadas, como:
   - prazo real de entrega;
   - diferença em relação à estimativa;
   - razão entre frete e preço;
   - status de pontualidade;
   - categorias de custo de frete;
7. Identificação de outliers pelo método do intervalo interquartil;
8. Análise exploratória com tabelas, gráficos, boxplots, heatmaps e dispersões;
9. Elaboração de conclusões e recomendações de negócio.

## Principais resultados

### Frete e prazo de entrega

Foi identificada uma correlação positiva de aproximadamente **0,167** entre o frete total e o prazo real de entrega. Essa relação é fraca: pedidos com fretes maiores tendem a apresentar prazos maiores, mas o frete isoladamente não permite prever o tempo de entrega.

Também foram observadas diferenças entre os estados. Regiões mais distantes dos principais centros de distribuição tendem a apresentar fretes e prazos médios mais elevados, além de maior variabilidade.

### Pagamentos e ticket médio

O valor médio dos pedidos varia conforme o tipo de pagamento e a quantidade de parcelas. Pedidos parcelados no cartão de crédito tendem a apresentar valores mais elevados, mas a análise é descritiva e não permite afirmar que o meio de pagamento cause o aumento do ticket.

### Comportamento ao longo do tempo

A análise temporal indicou melhora no prazo médio de entrega durante parte de 2017. Após o período da Black Friday, houve aumento nos atrasos, especialmente nas regiões Norte e Nordeste.

### Clientes recorrentes

A maior parte dos clientes realizou apenas uma compra. Foram identificados:

- 93.099 clientes pontuais;
- 2.997 clientes recorrentes.

O ticket médio foi de aproximadamente:

- **R$ 138,62** para clientes pontuais;
- **R$ 124,91** para clientes recorrentes.

Essa diferença descreve o comportamento observado na base, mas não permite concluir que a recorrência cause um ticket maior ou menor.

## Limitações

- A correlação não controla simultaneamente fatores como distância, vendedor, produto e transportadora.
- Os dados não permitem afirmar relações de causalidade.
- Não há informações detalhadas sobre rotas, centros de distribuição ou modais de transporte.
- Não é possível separar completamente o tempo de preparação do pedido do tempo de transporte.
- A análise de pagamentos pode simplificar pedidos com múltiplos registros de pagamento.
- A distribuição dos valores possui assimetria e pedidos de alto valor podem influenciar as médias.
- A base possui dados até agosto de 2018, limitando análises posteriores.

## Recomendações

- Planejar capacidade adicional para períodos sazonais, especialmente a Black Friday.
- Avaliar a criação de hubs ou centros de distribuição nas regiões Norte e Nordeste.
- Utilizar modelos de previsão de prazo que considerem distância, estado, vendedor, produto e período da compra.
- Investigar perfis de clientes associados a cada modalidade de pagamento.
- Avaliar campanhas de incentivo à recorrência e ao parcelamento considerando conversão, ticket e margem.
- Incorporar dados de descontos, juros, custos de pagamento e rentabilidade em análises futuras.