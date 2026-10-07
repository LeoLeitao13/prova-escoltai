# Casos de Teste — Zona Azul Digital

Variante: tarifa 450, fração 30 min (225 centavos), teto 5000,
tolerância 0, porta 8002.

> [!IMPORTANT]
> Cada linha das tabelas abaixo deve virar um teste automatizado (pytest)
> na pasta `tests/` do projeto gerado.

## Como os testes controlam o tempo

O encerramento usa o relógio real. Para simular durações:

1. Abrir o bilhete com `entrada` = agora menos N minutos.
2. Encerrar em seguida.
3. A duração real é N minutos + alguns milissegundos, e precisa resultar
   em `minutos = N` (truncamento).

## R1 — Fração arredonda para cima

| Caso | minutos | frações | valor_centavos |
| --- | --- | --- | --- |
| Fração exata | 30 | 1 | 225 |
| Exata + 1 min | 31 | 2 | 450 |
| Duas frações exatas | 60 | 2 | 450 |
| Duas + 1 min | 61 | 3 | 675 |
| Três exatas | 90 | 3 | 675 |
| Três + 1 min | 91 | 4 | 900 |

## R2 — Minutos são truncados (segundos descartados)

| Caso | Duração real | minutos | valor_centavos |
| --- | --- | --- | --- |
| Menos de 1 min | 59 s | 0 | 0 |
| 1 min exato | 60 s | 1 | 225 |
| Quase 31 min | 30 min 59 s | 30 | 225 |
| 90 min + milissegundos | 90 min 0,3 s | 90 | 675 |

## R3 — Teto diário (5000)

> [!WARNING]
> O teto é aplicado **depois** de multiplicar frações × 225.

| Caso | minutos | frações | bruto | valor_centavos |
| --- | --- | --- | --- | --- |
| Abaixo do teto | 660 | 22 | 4950 | 4950 |
| Primeira fração acima | 661 | 23 | 5175 | 5000 |
| Muito acima | 1440 | 48 | 10800 | 5000 |

## R4 — Tolerância (0 min nesta variante)

| Caso | minutos | valor_centavos |
| --- | --- | --- |
| Dentro da tolerância | 0 | 0 |
| 1 min além | 1 | 225 (cobra desde o primeiro minuto) |

## R5 — Entrada no futuro

| Caso | entrada | Esperado no encerramento |
| --- | --- | --- |
| Futuro | agora + 60 min | 200, `minutos: 0`, `valor_centavos: 0` |

## R6 — Tempo médio do relatório (0,5 para cima)

| Bilhetes encerrados (minutos) | Média exata | tempo_medio_minutos |
| --- | --- | --- |
| 47 | 47 | 47 |
| 30, 31 | 30,5 | 31 |
| 10, 11 | 10,5 | 11 |
| 30, 30, 31 | 30,33 | 30 |
| nenhum | — | 0 |

## R7 — Universo do relatório

| Situação | total_bilhetes | faturamento_centavos |
| --- | --- | --- |
| Encerrados de 90 e 30 min hoje | 2 | 900 |
| + 1 cancelado hoje | 2 | 900 (cancelado não conta) |
| + 1 ainda aberto | 2 | 900 (aberto não conta) |

## R8 — Uma vaga por placa

| Sequência | Último status | Corpo |
| --- | --- | --- |
| Abrir ABC1D23 → abrir ABC1D23 | 409 | `bilhete_em_aberto` |
| Abrir → encerrar → abrir | 201 | — |
| Abrir → cancelar → abrir | 201 | — |
| Abrir ABC1D23 → abrir XYZ9W87 | 201 | — (placas diferentes) |

## R9 — Transições de estado

| Estado atual | Ação | Status | Corpo |
| --- | --- | --- | --- |
| encerrado | encerrar | 409 | `bilhete_ja_encerrado` |
| cancelado | encerrar | 409 | `bilhete_ja_encerrado` |
| encerrado | cancelar | 409 | `bilhete_nao_aberto` |
| cancelado | cancelar | 409 | `bilhete_nao_aberto` |
| inexistente (id 999) | encerrar ou cancelar | 404 | `bilhete_nao_encontrado` |

## R10 — Validação de formato e precedência

| Requisição | Status | Corpo |
| --- | --- | --- |
| placa `ABC1D2` (6 caracteres) | 422 | `placa_invalida` |
| placa `ABC1D234` (8 caracteres) | 422 | `placa_invalida` |
| placa `abc1d23` (minúscula) | 422 | `placa_invalida` |
| placa `ABC-123` (símbolo) | 422 | `placa_invalida` |
| placa `1234567` (só números) | 201 | — (alfanumérico aceita só dígitos) |
| body sem `placa` | 422 | `placa_invalida` |
| `entrada: "2026-10-12T08:30:00"` (sem fuso) | 422 | `entrada_invalida` |
| `entrada: "ontem"` | 422 | `entrada_invalida` |
| placa inválida com outro bilhete aberto | 422 | `placa_invalida` (422 antes de 409) |
| `data=2026-02-30` | 422 | `data_invalida` |
| `data=07/10/2026` | 422 | `data_invalida` |

## R11 — Ordenação e tipos

| Caso | Esperado |
| --- | --- |
| Ativos com entrada 08:00 e 09:00 | O de 09:00 vem primeiro |
| `valor_centavos` na resposta | Tipo inteiro JSON (`675`, nunca `675.0`) |
| `entrada` e `saida` na resposta | Terminam em `-03:00` |


