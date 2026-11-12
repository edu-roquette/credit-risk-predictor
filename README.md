# Projeto Final: Modelo de Classificação para Risco de Crédito

Autores: Eduardo Roquette, Fernando Araújo e Gustavo Bogéa

Data: Novembro/2026

## 1. Objetivo

Desenvolver um modelo de machine learning para prever a probabilidade de inadimplência de clientes com base em informações cadastrais e históricas de crédito. O projeto foi estruturado para apoiar decisões de análise de risco em concessão de crédito, com foco em identificar corretamente perfis com histórico de atraso sem rejeitar de forma excessiva clientes financeiramente saudáveis.

## 2. Contexto e desafio de negócio

O setor financeiro lida com um desafio clássico de classificação binária em ambiente fortemente desbalanceado. Neste conjunto, a distribuição observada foi:

- 88,31% de bons pagadores (classe 0)
- 11,69% de inadimplentes (classe 1)

Esse cenário torna crítica a escolha de métricas e limiares na tomada de decisão. Modelos que priorizam apenas a acurácia global tendem a favorecer a classe majoritária e ignorar os casos de maior risco financeiro. Em crédito, isso resulta em perdas por falsos negativos ou, em uma abordagem excessivamente conservadora, rejeição de clientes rentáveis.

## 3. Fonte dos dados

A base utilizada foi extraída do dataset público do Kaggle:

https://www.kaggle.com/datasets/rikdifos/credit-card-approval-prediction

A estrutura do conjunto contempla:

- dados cadastrais dos clientes (renda, ocupação, educação, estado civil, tipo de moradia, etc.);
- histórico de pagamento mensal;
- indicadores de atraso e comportamento de crédito ao longo do tempo.

## 4. Definição da variável alvo

A variável alvo foi construída a partir do histórico de pagamentos mensais. Considerou-se como cliente inadimplente aquele que apresentou pelo menos um registro de atraso nos status de 1 a 5, que incluem atrasos de 1 a 29 dias e outras faixas de inadimplência observadas no histórico.

A lógica implementada foi:

- status de 1 a 5 = inadimplente
- status 0 = sem atraso relevante no período observado
- agregação por cliente para definir se houve pelo menos um atraso ao longo do histórico

Essa escolha reflete a realidade do setor financeiro, em que a existência de histórico de atraso é relevante para a análise de risco, mesmo que o comportamento mais recente esteja estável.

## 5. Pipeline de dados

### 5.1 Merge das bases

Foram combinadas as bases:

- application_record.csv
- credit_record.csv

A junção foi feita pelo identificador do cliente, permitindo reunir perfil cadastral e comportamento financeiro em um único conjunto.

### 5.2 Feature engineering

Durante o pré-processamento, foram aplicadas as seguintes transformações:

- conversão de variáveis temporais em indicadores mais interpretáveis, como idade e anos de emprego;
- mapeamento e padronização de categorias em português;
- imputação de valores ausentes em colunas numéricas e categóricas;
- criação de features representativas do perfil socioeconômico e do histórico financeiro;
- verificação de duplicações de clientes para garantir consistência da amostra.

### 5.3 Variáveis selecionadas

As features utilizadas no modelo incluíram:

- renda anual
- idade
- anos de emprego
- quantidade de membros na família
- possuir imóvel
- possuir carro
- tipo de renda
- escolaridade
- estado civil
- tipo de moradia
- ocupação

### 5.4 Preparação para modelagem

A base final foi dividida em:

- treino: 21.873 registros
- validação: 7.292 registros
- teste: 7.292 registros

A separação foi feita de forma estratificada pela variável alvo para preservar a proporção original das classes.

Além disso:

- o pré-processamento (imputação + One-Hot Encoding para variáveis categóricas) foi ajustado somente sobre o treino;
- o StandardScaler foi aplicado apenas para a Regressão Logística, mantendo os dados originais para o XGBoost.

## 6. Modelos testados

Foram avaliados três cenários principais:

1. Baseline (DummyClassifier)
2. Regressão Logística
3. XGBoost Classifier

### 6.1 Baseline

O baseline foi definido como um classificador que prevê sempre a classe majoritária.

Resultado no teste:

- acurácia: 0,882
- precisão: 0,000
- recall: 0,000
- ROC-AUC: 0,500

Esse cenário ilustra por que a acurácia isolada não é suficiente para uma análise confiável em dados desbalanceados.

### 6.2 Regressão Logística

A Regressão Logística foi treinada com class_weight='balanced' para mitigar o efeito do desbalanceamento.

Principais resultados no conjunto de teste:

- padrão (limiar 0,50):
  - acurácia: 0,555
  - precisão: 0,133
  - recall: 0,502
  - ROC-AUC: 0,545

- limiar ótimo (0,51):
  - acurácia: 0,616
  - precisão: 0,137
  - recall: 0,427
  - ROC-AUC: 0,545

- limiar de negócio (0,35):
  - acurácia: 0,127
  - precisão: 0,118
  - recall: 0,995
  - ROC-AUC: 0,545

Conclusão: a Regressão Logística apresentou baixo poder discriminativo para o problema e não foi considerada a melhor opção para produção.

### 6.3 XGBoost

O modelo XGBoost foi treinado com ajuste para o desbalanceamento da classe positiva utilizando scale_pos_weight e parâmetros adaptados para classificação de risco.

Resultados no conjunto de teste:

- padrão (limiar 0,50):
  - acurácia: 0,698
  - precisão: 0,199
  - recall: 0,517
  - ROC-AUC: 0,671

- limiar ótimo (0,48):
  - acurácia: 0,640
  - precisão: 0,182
  - recall: 0,587
  - ROC-AUC: 0,671

- limiar comercial (0,45):
  - acurácia: 0,572
  - precisão: 0,171
  - recall: 0,688
  - ROC-AUC: 0,671

- limiar de negócio (0,30):
  - acurácia: 0,239
  - precisão: 0,127
  - recall: 0,927
  - ROC-AUC: 0,671

## 7. Avaliação de desempenho

Para esse problema, as métricas mais relevantes não são apenas acurácia e precisão isoladas, mas também:

- ROC-AUC: mede a capacidade do modelo de distinguir clientes inadimplentes de bons pagadores;
- Recall: mede a proporção de inadimplentes corretamente identificados;
- Precisão: mede a confiabilidade dos clientes sinalizados como risco;
- F1-Score: equilibra recall e precisão em uma única métrica.

## 8. Interpretação do negócio

A análise mostra que o objetivo não é maximizar indiscriminadamente a redução de falsos negativos, porque isso leva à rejeição massiva de clientes rentáveis. Em crédito, a decisão ótima depende do trade-off entre:

- risco de concessão excessiva;
- perda por inadimplência;
- rejeição de bons clientes.

Nesse contexto, o modelo XGBoost oferece o melhor equilíbrio operacional, especialmente quando o limiar de decisão é ajustado para um cenário mais comercial, como 0,45. Esse ajuste consegue aumentar o recall para 68,8%, capturando uma parcela muito maior dos potenciais inadimplentes sem exigir um corte excessivamente agressivo.

## 9. Decisão final

O modelo final recomendado para aplicação operacional é o XGBoost com limiar de decisão em 0,45 (ou ajuste fino em torno de 0,50, conforme a tolerância de risco da instituição).

### Recomendações práticas

- utilizar XGBoost como modelo principal;
- calibrar o limiar conforme a política de risco da instituição;
- manter monitoramento contínuo do modelo em produção;
- revisar a definição de inadimplência e a qualidade dos dados periodicamente;
- considerar ajustes adicionais de custo de decisão se o negócio exigir priorização de recall ou precisão.

## 10. Conclusão

O projeto demonstrou que, para um problema de risco de crédito fortemente desbalanceado, modelos lineares tradicionais são limitados, enquanto abordagens baseadas em árvores, como XGBoost, entregam desempenho superior em discriminação e utilidade operacional.

A combinação de engenharia de features, tratamento adequado do desbalanceamento e escolha de limiar comercial apropriado foi essencial para transformar o modelo em uma ferramenta útil para apoio à decisão de concessão de crédito.

## 11. Estrutura do repositório

- README.md
- credit_risk_model_V2.ipynb
- dados/
  - application_record.csv
  - credit_record.csv

## 12. Como reproduzir

1. Clone o repositório.
2. Instale as dependências do projeto.
3. Acesse a pasta `dados` e confirme que os arquivos de entrada estejam presentes.
4. Abra o notebook `credit_risk_model_V2.ipynb`.
5. Execute as células em sequência para reproduzir o pipeline completo.

Dependências principais:

- Python 3.x
- pandas
- numpy
- scikit-learn
- xgboost
- matplotlib

## 13. Observações finais

Este projeto reforça a importância de analisar classificação de risco de crédito com uma visão de negócio, e não apenas com foco em métricas acadêmicas. A escolha final do modelo e do limiar deve ser alinhada ao perfil de risco da instituição, ao custo de falsos positivos e falsos negativos e à estratégia operacional de aprovação de crédito.

---

A utilização do XGBoost combinada com o ajuste de limiar mostrou ser a alternativa mais robusta para capturar clientes com risco real de inadimplência enquanto preserva uma base de decisões racional e sustentável para crédito.
