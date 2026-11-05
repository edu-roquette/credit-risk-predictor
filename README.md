# 📊 Projeto Final: Modelo de Classificação para Risco de Crédito

# 

# Autores: Eduardo Roquette / Fernando Araújo / Gustavo Bogéa

# 

# Data: Novembro/2026

# 

# 🎯 Objetivo

# 

# Desenvolver um algoritmo de machine learning capaz de classificar clientes em bons ou maus pagadores com base em dados financeiros e demográficos (como renda anual, faixa etária, escolaridade e ocupação). O modelo foca em prever a probabilidade de inadimplência a partir do histórico de crédito, trazendo maior precisão analítica e automação para apoiar a tomada de decisão na concessão de crédito, equilibrando a mitigação de risco com a geração de negócios.

# 

# 💼 Contexto de Negócio e Desafio

# 

# No setor financeiro, a concessão de crédito lida com um desafio clássico: o desbalanceamento severo de dados. Nesta base, identificamos:

# 

# 88,31% da base formada por Bons Pagadores (Classe 0)

# 

# 11,69% da base formada por Inadimplentes (Classe 1)

# 

# Modelos tradicionais tendem a classificar todos os clientes como bons pagadores apenas por probabilidade, gerando perdas financeiras (falsos negativos), ou tornam-se excessivamente conservadores, barrando bons clientes (falsos positivos). O projeto foca em otimizar as métricas de Recall e Precision para garantir a saúde financeira e comercial da instituição.

# 

# 🛠️ Metodologia e Pipeline de Dados

# 

# O processamento dos dados envolveu as seguintes etapas:

# 

# Limpeza e Tratamento: Imputação de valores nulos, remoção de registros duplicados e padronização de nomenclatura de colunas.

# 

# Feature Engineering: Criação de variáveis estratégicas como idade e anos\_emprego a partir de contagens de dias, além de um tratamento específico para diferenciar desempregados/aposentados.

# 

# Tratamento de Outliers: Aplicação do método IQR (Intervalo Interquartil) na variável de renda anual para remover distorções.

# 

# Target (Variável Alvo): Mapeamento do histórico de pagamentos, considerando como inadimplente (1) qualquer cliente com atraso superior a 30 dias em qualquer momento da vida de crédito.

# 

# Pré-processamento para ML:

# 

# Separação Estratificada (stratify=y) para garantir a proporção correta de classes no treino e teste.

# 

# One-Hot Encoding (pd.get\_dummies com drop\_first=True) para variáveis categóricas.

# 

# Padronização de escala (StandardScaler) para a Regressão Logística.

# 

# 🤖 Modelagem e Comparação

# 

# Foram testados dois algoritmos, ambos configurados para lidar com o forte desbalanceamento da base de dados:

# 

# Regressão Logística: Modelo baseline (linear). Utilizou class\_weight='balanced' para penalizar erros na minoria.

# 

# XGBoost Classifier: Modelo baseado em árvores (não linear). Utilizou scale\_pos\_weight ajustado matematicamente pela proporção das classes para maximizar a captura da inadimplência.

# 

# 📈 Resultados e Impacto Comercial

# 

# O XGBoost demonstrou superioridade clara na detecção de padrões complexos, sendo o modelo recomendado para produção.

# 

# ROC AUC: Evolução de 0.54 (Logística) para 0.6838 (XGBoost), indicando um poder substancialmente maior de separar os bons dos maus perfis.

# 

# Geração de Negócios (Recall Classe 0): O XGBoost aprova corretamente 72% dos bons pagadores (contra apenas 58% do modelo anterior). Isso significa destravar milhares de concessões seguras e reduzir clientes barrados injustamente (Falsos Positivos).

# 

# Proteção do Capital (Recall Classe 1): O modelo barra 56% dos inadimplentes reais, evitando perdas financeiras na carteira de crédito.

# 

# Assertividade do Bloqueio (Precision Classe 1): Quando o modelo emite o alerta de risco, ele é mais preciso que a regressão (21% vs 14%), reduzindo alarmes falsos operacionais.


