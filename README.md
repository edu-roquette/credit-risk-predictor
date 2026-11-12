# Projeto Final: Modelo de Classificação para Risco de Crédito

Autores: Eduardo Roquette, Fernando Araújo e Gustavo Bogéa

Data: Novembro/2026

# Classificação de Risco de Crédito

## 1. Visão geral

O projeto compara modelos supervisionados de classificação binária para identificar **associações entre características cadastrais e financeiras e a ocorrência de atrasos no histórico de pagamentos** dos clientes.

O objetivo é estudar técnicas de preparação de dados, tratamento de desbalanceamento, treinamento e avaliação de classificadores. Os resultados **não demonstram previsão de inadimplência futura** e não justificam, isoladamente, decisões automatizadas de concessão de crédito.

## 2. Dados utilizados

O notebook lê dois arquivos CSV a partir da pasta `dados/`:

| Arquivo | Conteúdo |
|---|---|
| `dados/application_record.csv` | Informações cadastrais e socioeconômicas dos clientes. |
| `dados/credit_record.csv` | Histórico mensal de registros de pagamento. |

As bases são relacionadas por `id_cliente` (coluna `ID` nos arquivos de origem). O histórico mensal é agregado por cliente para formar a variável alvo e, depois, combinado com os dados cadastrais por *inner join*.

Na execução armazenada no notebook, a base final apresenta **36.457 linhas e 36.457 clientes únicos**. A verificação diagnóstica encontrou **47 ocorrências de IDs repetidos na base cadastral**, mas **nenhuma na base final**.

## 3. Definição da variável alvo

A coluna `inadimplente`, preservada no código por compatibilidade, representa neste experimento **a ocorrência de pelo menos um registro histórico de atraso**, e não necessariamente inadimplência grave:

- **Classe 1:** o cliente apresentou ao menos um status de pagamento `1`, `2`, `3`, `4` ou `5` no período observado.
- **Classe 0:** o cliente não apresentou nenhum desses status no período observado.

A classe positiva **inclui atrasos de 1 a 29 dias**, além das faixas superiores. O rótulo é obtido com uma agregação por cliente (`max`) da indicação binária de atraso ao longo dos registros mensais.

**Limitação central:** não há separação temporal entre o período usado para caracterizar os clientes e um período futuro de observação do desfecho. Portanto, o estudo avalia classificação associada ao histórico, não desempenho comprovado de previsão futura.

## 4. Preparação dos dados

O notebook realiza renomeação e padronização de colunas, mapeamento de categorias e engenharia de atributos. Entre as variáveis utilizadas estão:

- renda anual, idade, anos de emprego e quantidade de membros da família;
- posse de imóvel e carro;
- tipo de renda, escolaridade, estado civil, tipo de moradia e ocupação.

A escolaridade é codificada numericamente de forma ordinal. A variável `anos_emprego` mantém o tratamento definido anteriormente pelo grupo, inclusive a conversão em anos com limite inferior de zero; esse procedimento pode tratar valores especiais de forma simplificada e deve ser considerado uma limitação.

### Divisão e prevenção de vazamento

Os dados são separados de maneira estratificada, com `random_state=42`:

| Conjunto | Proporção | Registros |
|---|---:|---:|
| Treinamento | 60% | 21.873 |
| Validação | 20% | 7.292 |
| Teste | 20% | 7.292 |

O pré-processamento é **ajustado apenas no treinamento** usando `Pipeline` e `ColumnTransformer`:

- **Numéricas:** imputação pela mediana (`SimpleImputer`).
- **Categóricas:** imputação pela categoria mais frequente e `OneHotEncoder(handle_unknown='ignore', drop='first')`.
- **Regressão Logística:** `StandardScaler` ajustado somente no treinamento e aplicado aos demais conjuntos.
- **XGBoost:** utiliza as variáveis transformadas sem escalonamento adicional.

## 5. Modelos avaliados

| Modelo | Configuração principal | Papel no estudo |
|---|---|---|
| `DummyClassifier` | `strategy='most_frequent'` | Referência (*baseline*) que prevê a classe majoritária. |
| Regressão Logística | `max_iter=1000`, `class_weight='balanced'`, `random_state=42` | Classificador linear com ponderação de classes. |
| XGBoost | `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`, `scale_pos_weight` calculado no treino, `random_state=42` | Classificador baseado em árvores impulsionadas. |

A escolha dos limiares pelo **índice de Youden** é realizada sobre as probabilidades do **conjunto de validação**. Os limiares selecionados são então avaliados no conjunto de teste. Também são apresentados limiares fixos para ilustrar diferentes compromissos entre precisão e sensibilidade.

## 6. Resultados registrados na V2

As métricas abaixo são as **saídas salvas no notebook**, não resultados de uma nova execução independente.

| Modelo / limiar | Acurácia | Precisão | Recall | F1 | ROC-AUC |
|---|---:|---:|---:|---:|---:|
| Baseline (`DummyClassifier`) | 0,8823 | 0,0000 | 0,0000 | 0,0000 | 0,5000 |
| Regressão Logística — 0,50 | 0,5546 | 0,1325 | 0,5023 | 0,2097 | 0,5448 |
| Regressão Logística — Youden 0,51 | 0,6163 | 0,1370 | 0,4266 | 0,2074 | 0,5448 |
| Regressão Logística — negócio 0,35 | 0,1267 | 0,1183 | 0,9953 | 0,2115 | 0,5448 |
| XGBoost — 0,50 | 0,6983 | 0,1991 | 0,5175 | **0,2876** | **0,6706** |
| XGBoost — Youden 0,48 | 0,6402 | 0,1817 | 0,5874 | 0,2775 | **0,6706** |
| XGBoost — comercial 0,45 | 0,5720 | 0,1714 | 0,6876 | 0,2744 | **0,6706** |
| XGBoost — negócio 0,30 | 0,2392 | 0,1266 | 0,9266 | 0,2228 | **0,6706** |

**Interpretação:**

- O baseline alcança acurácia elevada por privilegiar a classe majoritária, mas apresenta **recall zero** para a classe com registros históricos de atraso.
- O **XGBoost apresenta o maior ROC-AUC** entre os modelos testados (aproximadamente **0,671**), contra **0,545** da Regressão Logística e **0,500** do baseline.
- No teste, o XGBoost com limiar **0,50** apresentou o maior F1 entre os cenários registrados (**0,288**). Isso **não prova** que 0,50 seja o limiar ótimo para novos dados.
- Ao reduzir o limiar do XGBoost para **0,45**, o recall sobe para aproximadamente **68,8%**, com redução da precisão para aproximadamente **17,1%**. No limiar **0,30**, o recall alcança **92,7%**, mas a precisão cai para aproximadamente **12,7%**.
- Alterar o limiar modifica as classificações e métricas dependentes do limiar, **não o ROC-AUC** calculado a partir das mesmas probabilidades.

O notebook inclui tabela comparativa, matrizes de confusão para três cenários da Regressão Logística e quatro do XGBoost, além de análises de coeficientes e importância de atributos.

## 7. Conclusão e limitações

Entre os modelos testados, o **XGBoost é o mais promissor para aprofundamento**, devido à maior capacidade de discriminação observada no conjunto de teste. Entretanto, sua precisão ainda é baixa em todos os cenários apresentados; não se recomenda implantação imediata.

Antes de qualquer aplicação real, seriam necessários, entre outros pontos:

1. **Validação temporal** com dados de períodos posteriores e definição de um desfecho futuro.
2. **Análise dos custos de falsos positivos e falsos negativos**, para fundamentar a escolha do limiar.
3. **Avaliação de calibração**, robustez e generalização em novas amostras.
4. Revisão das hipóteses de tratamento de atributos, incluindo a **codificação ordinal da escolaridade** e o tratamento de **anos de emprego**.

O projeto demonstra a importância de utilizar um baseline, evitar vazamento no ajuste do pré-processamento e separar a escolha dos limiares da avaliação final.

## 8. Como executar

### Requisitos

- Python 3 e ambiente Jupyter (por exemplo, VS Code com extensão Jupyter);
- Bibliotecas Python: `pandas`, `numpy`, `matplotlib`, `scikit-learn` e `xgboost`.

Exemplo de instalação:

```bash
python -m pip install pandas numpy matplotlib scikit-learn xgboost jupyter
```

> O código usa `OneHotEncoder(sparse_output=False)`, parâmetro disponível em versões recentes do `scikit-learn` (a partir da versão 1.2).

### Estrutura esperada

```text
repositorio/
├── credit_risk_model.ipynb       # versão original, se mantida
├── credit_risk_model_V2.ipynb    # versão revisada
├── README.md                     # documentação original, se mantida
├── README_V2.md                  # esta documentação
└── dados/
    ├── application_record.csv
    └── credit_record.csv
```

A estrutura acima mostra os caminhos **esperados pelo notebook**; confirme que os CSVs estão disponíveis nesses locais. Se os dados não estiverem versionados no repositório, será necessário obtê-los separadamente e colocá-los na pasta `dados/`.

1. Abra o repositório como pasta de trabalho no VS Code ou inicie o Jupyter a partir da raiz do projeto.
2. Selecione o ambiente Python com as dependências instaladas.
3. Abra `credit_risk_model_V2.ipynb`.
4. Execute as células em ordem (`Run All`).
5. Confira a tabela de métricas e as visualizações geradas.

---

*Documentação elaborada a partir do conteúdo e das saídas armazenadas em `credit_risk_model.ipynb`.*

---

A utilização do XGBoost combinada com o ajuste de limiar mostrou ser a alternativa mais robusta para capturar clientes com risco real de inadimplência enquanto preserva uma base de decisões racional e sustentável para crédito.
