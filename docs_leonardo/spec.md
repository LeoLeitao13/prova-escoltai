# Especificação — Zona Azul Digital

## Convenções gerais

- Base URL: `http://localhost:8002`
- Datas e horas: ISO-8601 com fuso `-03:00` (ex.: `2026-10-07T20:15:00-03:00`)
- Valores em dinheiro: sempre **centavos inteiros**
- Todo erro: corpo `{"erro": "<codigo>"}`
- Ordem das verificações: 422 (formato) → 404 (inexistente) → 409 (conflito)

### Formato do bilhete

| Campo | aberto | encerrado | cancelado |
| --- | --- | --- | --- |
| id, placa, entrada, status | sim | sim | sim |
| saida, minutos, valor_centavos | não | sim | não |

## UC1 — Abrir bilhete

`POST /bilhetes` com body `{"placa": "ABC1D23"}` e `entrada` opcional.

| # | Entrada | Esperado |
| --- | --- | --- |
| CA1.1 | `{"placa": "ABC1D23"}` | 201, `status: "aberto"`, `id` inteiro, `entrada` termina em `-03:00` |
| CA1.2 | `entrada: "2026-10-12T08:30:00-03:00"` | 201, `entrada` igual à enviada |
| CA1.3 | Dois bilhetes de placas diferentes | Segundo `id` = primeiro + 1 |
| CA1.4 | `{"placa": "abc1d23"}` | 422 `placa_invalida` |
| CA1.5 | `{"placa": "ABC12"}` ou body vazio | 422 `placa_invalida` |
| CA1.6 | `entrada: "12/10/2026"` ou sem fuso | 422 `entrada_invalida` |

## UC2 — Encerrar bilhete

`POST /bilhetes/{id}/encerramento`

| # | Entrada | Esperado |
| --- | --- | --- |
| CA2.1 | Bilhete aberto há 90 min | 200, `minutos: 90`, `valor_centavos: 675`, `status: "encerrado"` |
| CA2.2 | Resposta | Contém `id, placa, entrada, saida, minutos, valor_centavos` |
| CA2.3 | `valor_centavos` | Sempre inteiro|
| CA2.4 | Id inexistente | 404 `bilhete_nao_encontrado` |
| CA2.5 | Encerrar duas vezes | 409 `bilhete_ja_encerrado` |
| CA2.6 | Encerrar bilhete cancelado | 409 `bilhete_ja_encerrado` |
| CA2.7 | `entrada` no futuro | 200, `minutos: 0`, `valor_centavos: 0` |

## UC3 — Listar ativos

`GET /bilhetes/ativos`

| # | Situação | Esperado |
| --- | --- | --- |
| CA3.1 | Nenhum bilhete aberto | 200, `[]` |
| CA3.2 | Abertos A (08:00) e B (09:00) | 200, `[B, A]` (mais recente primeiro) |
| CA3.3 | Bilhete encerrado ou cancelado | Não aparece na lista |

## UC4 — Relatório diário

`GET /relatorios/diario?data=AAAA-MM-DD`

Conta só bilhetes **encerrados** cuja `saida` caiu na data consultada.

| # | Situação | Esperado |
| --- | --- | --- |
| CA4.1 | Dia sem encerramentos | 200, `total_bilhetes: 0`, `faturamento_centavos: 0`, `tempo_medio_minutos: 0` |
| CA4.2 | Encerrados com 90 e 30 min | `total_bilhetes: 2`, `faturamento_centavos: 900`, `tempo_medio_minutos: 60` |
| CA4.3 | Encerrados com 30 e 31 min | `tempo_medio_minutos: 31` (30,5 arredonda para cima) |
| CA4.4 | Bilhete cancelado no dia | Não entra em nenhum total |
| CA4.5 | `data=05/10/2026`, `data=2026-13-40` ou ausente | 422 `data_invalida` |

## UC5 — Cancelar bilhete

`POST /bilhetes/{id}/cancelamento`

| # | Situação | Esperado |
| --- | --- | --- |
| CA5.1 | Bilhete aberto | 200, `status: "cancelado"`, sem `saida` nem `valor_centavos` |
| CA5.2 | Id inexistente | 404 `bilhete_nao_encontrado` |
| CA5.3 | Bilhete já cancelado ou encerrado | 409 `bilhete_nao_aberto` |

## UC6 — Histórico por placa

`GET /bilhetes?placa=ABC1D23`

| # | Situação | Esperado |
| --- | --- | --- |
| CA6.1 | Placa com 1 encerrado e 1 aberto | 200, 2 itens, mais recente primeiro |
| CA6.2 | Placa que nunca estacionou | 200, `[]` |
| CA6.3 | `placa=abc` ou ausente | 422 `placa_invalida` |

## UC7 — Tolerância gratuita

Tolerância desta variante: **0 minutos**.

| # | Duração | Esperado |
| --- | --- | --- |
| CA7.1 | 0 min | `valor_centavos: 0` |
| CA7.2 | 1 min | `valor_centavos: 225` (já cobra a 1ª fração) |

> [!NOTE]
> Se a tolerância fosse maior que 0, passar dela cobraria **desde o
> primeiro minuto**; a tolerância nunca é descontada.

## UC8 — Uma vaga por placa

| # | Situação | Esperado |
| --- | --- | --- |
| CA8.1 | Abrir placa que já tem bilhete aberto | 409 `bilhete_em_aberto` |
| CA8.2 | Abrir após encerrar o anterior | 201 |
| CA8.3 | Abrir após cancelar o anterior | 201 |
| CA8.4 | Placa inválida, mesmo com outro aberto | 422 `placa_invalida` (formato vem antes) |