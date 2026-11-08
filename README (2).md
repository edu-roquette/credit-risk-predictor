Projeto Final: Modelo de Classificação para Risco de Crédito
Autores: Eduardo Roquette, Fernando Araújo e Gustavo Bogéa

Data: Novembro/2026

Objetivo
Desenvolver um algoritmo de machine learning capaz de prever a probabilidade de inadimplência de clientes com base em dados financeiros e demográficos. O foco do projeto é automatizar e apoiar a tomada de decisão na concessão de crédito, garantindo o equilíbrio entre a mitigação de risco e a geração de receita.

Contexto e Desafio de Negócio
O setor financeiro lida constantemente com o desbalanceamento severo de dados. Nesta base, identificamos um cenário real de mercado:

88,31% de Bons Pagadores (Classe 0)

11,69% de Inadimplentes (Classe 1)

Modelos tradicionais tendem a classificar todos os clientes como bons pagadores (gerando perdas financeiras por falsos negativos) ou tornam-se excessivamente conservadores (barrando bons clientes e limitando o lucro).

Pipeline de Dados
Limpeza e Feature Engineering: Imputação de nulos, transformação de dias corridos em idade e anos de emprego, e adequação de variáveis categóricas.

Variável Alvo (Target): Cliente classificado como inadimplente (1) caso possua histórico de atraso superior a 30 dias em qualquer momento da vida do crédito.

Pré-processamento: Separação estratificada (preservando a proporção de 88/11), One-Hot Encoding e padronização (StandardScaler) para o modelo linear.

Modelagem e Avaliação de Performance
Testamos dois algoritmos lidando nativamente com o desbalanceamento das classes. A avaliação de negócio baseou-se não apenas no padrão de 0.50, mas na análise de limiares de corte otimizados (J de Youden) e limiares comerciais.

1. Regressão Logística (Inviável)
Poder de separação fraco: ROC-AUC de apenas 0.550, atuando próximo a um classificador aleatório.

Inviabilidade comercial: Para tentar conter os calotes definindo um limiar de risco de 0.35 (alcançando Recall de 99.5%), a acurácia global do modelo despenca para 12.5%. Na prática, isso significaria negar crédito para quase toda a base de clientes para evitar a inadimplência.

2. XGBoost Classifier (Recomendado)
O modelo baseado em árvores capturou as complexidades e não-linearidades dos dados, elevando o ROC-AUC para 0.676.

O Custo do "Risco Zero" (Limiar 0.30): Focar excessivamente na proteção contra calotes (Recall de 92.6%) derruba a precisão para 12.9%. Isso faria a instituição rejeitar cerca de 7 a 8 bons clientes para cada mau pagador bloqueado, asfixiando a esteira de vendas.

O Cenário Ótimo (Limiar 0.50): Entrega o melhor cenário operacional. Consegue barrar mais da metade dos verdadeiros inadimplentes (Recall de 53.8%) ao mesmo tempo em que mantém uma acurácia global de 70.1%, aprovando um volume saudável de bons pagadores.

Decisão Final
O XGBoost operando no limiar ótimo estatístico de 0.50 (ou com um ajuste comercial sutil para 0.45) é a escolha final para produção.

Em concessão de crédito fundamentada por dados demográficos e de cadastro, forçar uma captura de risco acima de 90% resulta inevitavelmente no bloqueio massivo de clientes rentáveis. O limiar de 0.50 garante um ponto de equilíbrio sustentável: protege o capital ao reter a maior fatia do risco evidente, sem paralisar a geração de negócios da instituição.