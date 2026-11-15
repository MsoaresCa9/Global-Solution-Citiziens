# CitizenTech | Previsão da gravidade de sinistros de trânsito em São Paulo

**FIAP · Global Solution 2026 · Inteligência Artificial & Machine Learning · Turma 3ESPZ**

Este projeto usa Machine Learning e dados reais e públicos para responder a uma pergunta sobre a segurança no trânsito da capital paulista:

> Quando acontece um sinistro de trânsito com vítima na cidade de São Paulo, dá para prever, só com o que se sabe no momento do registro (onde, quando, tipo de sinistro e quem está envolvido), se ele vai ter **vítima fatal ou ferido grave**?

Um modelo assim ajuda a priorizar o envio de SAMU e resgate e mostra em quais cenários e vias o risco de morte é maior.

---

## Por que esse tema

Os dados do INFOSIGA mostram o tamanho do problema na capital entre jan/2022 e set/2026:

| Indicador | Valor |
|---|---|
| Mortes no trânsito no período | **4.665** |
| Sinistros com morte ou ferido grave | **13.879** |
| Mortes por 100 mil habitantes em 2024 | **9,18** (eram 7,56 em 2022) |
| Participação da capital nos sinistros do estado | **23,2%** |
| Percentual de sinistros graves | subiu de 9,3% (2023) para 11,6% (2026) |

---

## Dados reais utilizados

Nenhum dado simulado foi usado. O notebook baixa os dados direto da fonte oficial.

| Fonte | Conteúdo | Link | Período |
|---|---|---|---|
| INFOSIGA SP (DETRAN-SP) | Sinistros de trânsito do estado de SP: data, hora, local, tipo, veículos e gravidade (CC BY 4.0) | https://infosiga.detran.sp.gov.br/ | jan/2022 a set/2026 |
| IBGE, Censo 2022 | População do município de São Paulo: 11.451.999 habitantes | https://cidades.ibge.gov.br/brasil/sp/sao-paulo/panorama | 2022 |

Mais detalhes em [`ml/dados/README.md`](ml/dados/README.md).

---

## Como o problema foi resolvido

1. **Problema e dados:** 134.171 sinistros com vítima na capital. O alvo `grave_ou_fatal` vale 1 quando há morte ou ferido grave (10,34% dos casos).
2. **Preparação:** limpeza dos dados, criação de atributos (hora, madrugada, fim de semana, rodovia, usuário vulnerável, veículos envolvidos, tipo de sinistro, coordenadas) e uma **auditoria de vazamento de dados**, que removeu campos preenchidos de forma diferente nos registros fatais.
3. **Divisão treino/teste temporal:** treino de 2022 a 2025 (109.961 sinistros) e teste em 2026 (16.995 sinistros), um ano que o modelo nunca viu.
4. **Modelo:** XGBoost, escolhido por capturar interações entre fatores, tratar o desbalanceamento (`scale_pos_weight`) e permitir controlar a complexidade.
5. **Treino:** 12 combinações de 3 hiperparâmetros (`max_depth` 3/5/7, `learning_rate` 0,05/0,1, `n_estimators` 200/500), com validação cruzada estratificada de 5 folds feita só no treino.
6. **Avaliação:** métricas de treino e teste lado a lado, curvas ROC e Precisão-Recall, matriz de confusão e discussão de overfitting.

## Principais resultados

| Métrica | Treino | Teste (2026) |
|---|---|---|
| ROC-AUC | 0,750 | 0,710 |
| PR-AUC | 0,314 | 0,299 |
| Recall (graves detectados) | 0,656 | 0,639 |
| Acurácia balanceada | 0,679 | 0,657 |

- Melhor configuração: `max_depth=5`, `learning_rate=0.05`, `n_estimators=200`.
- Overfitting controlado: gap de AUC de 0,04 entre treino e teste. As configurações mais complexas chegaram a 0,28 de gap e foram descartadas pela validação.
- Os sinistros marcados como alto risco têm **3,1 vezes** mais casos graves que os de baixo risco.
- Atropelamentos (18,7% graves), a madrugada (20,5% às 2h), fins de semana (+34%) e rodovias (+31%) são os cenários mais letais.
- As 10 vias mais perigosas concentram 11,5% dos casos graves da cidade.

---

## Entregáveis deste repositório

```
.
├── README.md                         -> este arquivo
├── RELATORIO_ML.md                   -> relatório do projeto, do problema à conclusão
├── docs/
│   └── Insights_CitizenTech_ML.pdf   -> relatório visual com os insights, gráficos e parâmetros
└── ml/
    ├── GS_CitizenTech_ML.ipynb       -> notebook executado, com 34 insights, saídas e gráficos
    ├── requirements.txt              -> bibliotecas necessárias
    └── dados/
        └── README.md                 -> fontes, links, período e dicionário dos dados
```

| Entregável | Onde está |
|---|---|
| Notebook executado | [`ml/GS_CitizenTech_ML.ipynb`](ml/GS_CitizenTech_ML.ipynb) |
| Documentação dos dados | [`ml/dados/README.md`](ml/dados/README.md) |
| Dependências | [`ml/requirements.txt`](ml/requirements.txt) |
| Relatório | [`RELATORIO_ML.md`](RELATORIO_ML.md) |
| PDF de insights | [`docs/Insights_CitizenTech_ML.pdf`](docs/Insights_CitizenTech_ML.pdf) |
| Vídeo (até 5 min, YouTube não listado) | _inserir o link aqui_ |

## Como executar

**No Google Colab:** abra o notebook pelo GitHub (Arquivo > Abrir notebook > GitHub) e clique em *Ambiente de execução > Executar tudo*. O download dos dados é automático e a execução leva cerca de 5 minutos.

**Localmente:**

```bash
pip install -r ml/requirements.txt
jupyter notebook ml/GS_CitizenTech_ML.ipynb
```

---

# **Integrantes:**
Caio Soares Rossini RM555084

Carlos Eduardo RM556785

Gabriel Danius RM555747

Giulia Rocha RM558084
