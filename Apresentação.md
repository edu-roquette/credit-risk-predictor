Projeto Final: Modelo de Classificação para Risco de Crédito
Autores: Eduardo Roquette, Fernando Araújo e Gustavo Bogéa

## ⏱️ Roteiro Geral (15 Minutos)

| Parte | Tópico Principal | Tempo | Responsável Sugerido |
| :--- | :--- | :---: | :--- |
| **Parte 1** | Problema, Dataset e Por Que Importa | 3 min | Integrante 1 |
| **Parte 2** | Preparação dos Dados e Principais Achados da EDA | 3 min | Integrante 1 |
| **Parte 3** | Modelos Testados e Comparação com a Referência | 5 min | Integrante 2 |
| **Parte 4** | Boas Práticas, Limitações e Conclusão | 4 min | Integrante 3 |

---

## 📌 Parte 1: Problema, Dataset e Por Que Importa (3 min)

Objetivo: Desenvolver um algoritmo de machine learning capaz de classificar clientes em bons ou maus pagadores com base em dados financeiros e demográficos, como renda anual e faixa etária. O modelo foca em prever a probabilidade de inadimplência a partir do histórico de crédito, trazendo maior precisão analítica e automação para apoiar a tomada de decisão na concessão.

Relevância do Tema: No setor financeiro, a concessão de crédito sem uma análise de risco adequada acarreta altas taxas de inadimplência (prejuízo financeiro direto) ou a recusa desnecessária de clientes solventes (perda de receita e oportunidades de mercado).

Decisão que o Modelo apoia: O modelo apoia a decisão automatizada ou semi-automatizada de aprovação, ajuste de limites ou negação de crédito no momento da solicitação, além de permitir o monitoramento preventivo de clientes em carteira com maior propensão ao atraso.

*"A questão não é puramente estatística de acertar quem paga, mas sim uma decisão económica de gestão de capital."*

---

## 🧹 Parte 2: Preparação dos Dados e Principais Achados da EDA (3 min)

* **Criação do Target:** Definimos a inadimplência com base em atrasos superiores a 30 dias (status de pagamento entre `1` e `5`).

* **Engenharia de Recursos (*Feature Engineering*):** Limpeza e conversão de variáveis temporais (como transformar idade e tempo de emprego em anos), tratamento de nulos e codificação categórica.

* **Divisão dos Dados:** Uso de amostragem estratificada (`stratify=y`) para garantir que a proporção de maus pagadores fosse mantida tanto no treino quanto no teste.

*"Aplicamos o `StandardScaler` de forma isolada, apenas para a Regressão Logística, garantindo que o modelo linear convergisse corretamente sem prejudicar o XGBoost."*

---

## ⚙️ Parte 3: Modelos Testados e Comparação com a Referência (5 min)

* **Abordagem Comparativa:** Comparamos um modelo linear interpretável (**Regressão Logística**) com um modelo não-linear de alta performance (**XGBoost**).

* **Tratamento do Desbalanceamento:** Configuramos pesos adequados em ambos os modelos (`class_weight='balanced'` e `scale_pos_weight`) para evitar que o algoritmo previsse apenas a classe majoritária.

* **Métricas Utilizadas:** O foco principal foi a **ROC-AUC** e o **Recall**, deixando de lado a acurácia (que seria enganosa devido ao desbalanceamento).

* **Ajuste de Limiares (*Thresholds*):** Explicar porque um limiar (*threshold*) excessivamente agressivo (ex: buscar 90% de Recall usando apenas dados demográficos) destruiria a rentabilidade da carteira.

*"Utilizamos o limiar ótimo em torno de $0.50$ (ou ajuste comercial de $0.45$) como o ponto de equilíbrio de negócio que protege o capital sem asfixiar as aprovações."*

---

## 🚀 Parte 4: Boas Práticas, Limitações e Conclusão (4 min)

* **Boas Práticas:** Código limpo e modularizado em Jupyter Notebook, repositório sincronizado e limpo no GitHub, e reprodutibilidade garantida via `requirements.txt`.

* **Limitações:** O modelo baseia-se fortemente em dados demográficos e cadastrais iniciais, o que significa que choques macroeconómicos repentinos (como desemprego generalizado) podem exigir reajustes.

* **Recomendação Final:** Implementar o **XGBoost** em produção operando no limiar equilibrado.

## 🛡️ Questionamentos que tivemos durante o projeto

1. **Por que não usar a Acurácia como métrica principal?**
   * *Resposta:* Porque os dados são desbalanceados. Se o modelo previsse sempre "bom pagador", ele teria aproximadamente 88% de acurácia, mas falharia completamente em detetar os inadimplentes.

2. **Por que o XGBoost teve melhor desempenho que a Regressão Logística?**
   * *Resposta:* Porque o XGBoost consegue capturar relações não-lineares e interações complexas entre as variáveis socioeconómicas que o modelo linear, por ser restrito, acaba por ignorar.

3. **Como o modelo se comportaria na prática se o risco de crédito mudasse?**
   * *Resposta:* O modelo serve como um filtro inicial de triagem de risco automatizado, devendo ser reajustado periodicamente (*retreino*) com novos dados de comportamento de pagamento.