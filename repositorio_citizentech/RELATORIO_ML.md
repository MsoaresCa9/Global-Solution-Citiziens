# Relatório de Machine Learning | CitizenTech

**Tema:** previsão da gravidade de sinistros de trânsito na cidade de São Paulo
**Integrantes:** Caio Soares Rossini (RM555084), Carlos Eduardo (RM556785), Gabriel Danius (RM555747), Giulia Rocha (RM558084)

## 1. Problema

Entre jan/2022 e set/2026, 4.665 pessoas morreram no trânsito da capital paulista e 13.879 sinistros tiveram morte ou ferido grave. A taxa de mortes chegou a 9,18 por 100 mil habitantes em 2024, e a proporção de sinistros graves subiu de 9,3% em 2023 para 11,6% em 2026.

**Pergunta de ML:** com o que se sabe no momento do registro (onde, quando, tipo de sinistro e quem está envolvido), dá para prever se um sinistro com vítima terá morte ou ferido grave? É uma classificação binária: `grave_ou_fatal` = 1 ou 0.

## 2. Dados reais

| Fonte | Link | Período |
|---|---|---|
| INFOSIGA SP (DETRAN-SP), arquivos `sinistros_2022-2024.csv` e `sinistros_2025-2026.csv` | https://infosiga.detran.sp.gov.br/ | jan/2022 a set/2026 |
| IBGE, Censo 2022 (população de 11.451.999 hab.) | https://cidades.ibge.gov.br/brasil/sp/sao-paulo/panorama | 2022 |

Dos 212.311 registros da capital, usamos os 134.171 sinistros com vítima. Registros do tipo notificação ficaram de fora porque não têm a gravidade preenchida.

## 3. Preparação

- Conversão de data e hora, criação de mês, dia da semana, fim de semana e madrugada.
- Quantidades de veículos vazias preenchidas com zero; 394 coordenadas fora do município anuladas.
- **Auditoria de vazamento:** os registros fatais vêm de outra rotina de coleta. Por exemplo, `tipo_via = NAO DISPONIVEL` aparece em 6,8% dos fatais e em 0% dos não fatais. Esses campos foram removidos para o modelo aprender o risco real, e não o formato do registro.
- Modelo final com 21 atributos e 126.956 sinistros georreferenciados (taxa de graves de 10,23%).
- **Divisão temporal:** treino de 2022 a 2025 (109.961 registros, 87%) e teste em 2026 (16.995 registros, 13%).

## 4. Modelo

**XGBoost**, porque os dados são tabulares, a gravidade depende de combinações de fatores, o desbalanceamento (1 grave para 9 leves) é tratado com `scale_pos_weight = 9,04` e a profundidade das árvores permite controlar o overfitting.

## 5. Treino com diferentes parâmetros

Foram testadas 12 combinações de 3 hiperparâmetros, com validação cruzada estratificada de 5 folds feita só no treino (60 treinos), escolhendo pela ROC-AUC de validação:

- `max_depth`: 3, 5, 7
- `learning_rate`: 0,05 e 0,1
- `n_estimators`: 200 e 500

A melhor configuração foi `max_depth=5`, `learning_rate=0.05` e `n_estimators=200`, com AUC de validação de 0,7167 ± 0,0026. A pior foi `max_depth=7`, `learning_rate=0.1` e `n_estimators=500`: 0,96 de AUC no treino, mas só 0,67 na validação.

## 6. Resultados em treino e teste

| Métrica | Treino | Teste (2026) |
|---|---|---|
| ROC-AUC | 0,7498 | 0,7103 |
| PR-AUC | 0,3138 | 0,2986 |
| Recall | 0,6563 | 0,6393 |
| Precisão | 0,1956 | 0,2108 |
| F1 | 0,3014 | 0,3170 |
| Acurácia balanceada | 0,6788 | 0,6569 |

**Overfitting:** o gap de AUC entre treino e teste é de 0,040, parecido com o da validação cruzada (0,039). Isso indica um ajuste moderado e um modelo que generaliza para um ano novo. Nas configurações mais profundas o gap chegou a 0,28, e a validação cruzada evitou escolhê-las. A PR-AUC de 0,30 é 2,5 vezes a referência aleatória (0,12). No teste, o modelo identificou 1.301 dos 2.035 sinistros graves.

## 7. Conclusão para o cidadão

- O modelo separa sinistros graves e leves em um ano que ele nunca viu. Os casos que ele marca como alto risco têm 3,1 vezes mais chance de ser graves (21,1% contra 6,8%).
- Ele encontra 64% dos casos graves logo no registro, o que ajuda a priorizar SAMU e resgate.
- Os cenários mais letais são atropelamentos (18,7% graves), a madrugada (20,5% às 2h), os fins de semana (+34% de risco) e as rodovias dentro da cidade (+31%).
- As 10 vias mais perigosas, entre elas Raposo Tavares, Rodoanel e as Marginais, concentram 11,5% dos casos graves e são prioridade para fiscalização e redução de velocidade.
- **Limitação:** o modelo mostra risco, não causa. Variáveis como velocidade, clima e uso de capacete podem melhorar versões futuras.
