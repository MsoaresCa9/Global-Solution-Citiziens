# Dados utilizados

Os dados não ficam salvos no repositório porque o pacote oficial tem 953 MB. O notebook baixa tudo automaticamente na primeira execução.

## 1. INFOSIGA SP (DETRAN-SP)

| Item | Detalhe |
|---|---|
| O que é | Sistema de Informações Gerenciais de Acidentes de Trânsito do Estado de São Paulo |
| Link | https://infosiga.detran.sp.gov.br/ |
| Download usado | https://infosiga.detran.sp.gov.br/rest/painel/download/file/dados_infosiga.zip |
| Espelho (se o portal estiver fora do ar) | https://github.com/viniciusoike/infosigasp/releases/download/data-current/dados_infosiga.zip |
| Arquivos usados | `sinistros_2022-2024.csv` e `sinistros_2025-2026.csv` |
| Publicação do pacote | 17/10/2026 |
| Período analisado | 01/01/2022 a 30/09/2026 |
| Recorte | Município de São Paulo (código IBGE 3550308), só sinistros com vítima (`SINISTRO FATAL` e `SINISTRO NAO FATAL`) |
| Formato | CSV, separador `;`, codificação latin-1, decimal `,` |
| Licença | Creative Commons Atribuição 4.0 (CC BY 4.0) |

### Colunas usadas

| Coluna | Uso no projeto |
|---|---|
| `data_sinistro`, `hora_sinistro` | Ano, mês, dia da semana, fim de semana, hora e madrugada |
| `latitude`, `longitude` | Localização do sinistro |
| `tipo_via` | Indicador de rodovia |
| `tp_sinistro_primario` | Tipo do sinistro (atropelamento, colisão, choque, outros) |
| `qtd_pedestre`, `qtd_bicicleta`, `qtd_motocicleta`, `qtd_automovel`, `qtd_onibus`, `qtd_caminhao`, `qtd_veic_outros` | Quem estava envolvido |
| `qtd_gravidade_fatal`, `qtd_gravidade_grave` | Só para criar o alvo `grave_ou_fatal` (não entram no modelo) |
| `logradouro` | Ranking das vias mais perigosas (não entra no modelo) |

Ficaram de fora do modelo, por terem padrão de preenchimento diferente nos registros fatais: as flags secundárias `tp_sinistro_*`, `tipo_local`, `administracao` e `circunscricao`.

## 2. IBGE, Censo Demográfico 2022

| Item | Detalhe |
|---|---|
| Link | https://cidades.ibge.gov.br/brasil/sp/sao-paulo/panorama |
| Dado | População residente do município de São Paulo: 11.451.999 habitantes |
| Uso | Taxa de mortes no trânsito por 100 mil habitantes |
