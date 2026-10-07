# Plano Técnico — Zona Azul Digital

## 1. Visão geral

API REST para abrir e encerrar bilhetes de estacionamento por placa, listar
bilhetes ativos, consultar histórico e emitir relatório diário.
O contrato exato de cada endpoint está em `spec.md`.

## 2. Parâmetros da variante

| Parâmetro | Valor |
| --- | --- |
| TARIFA_HORA_CENTAVOS | 450 |
| FRACAO_MINUTOS | 30 |
| TETO_DIARIO_CENTAVOS | 5000 |
| TOLERANCIA_MINUTOS | 0 |
| PORTA_SERVICO | 8002 |

Valor de cada fração: 450 / 2 = 225 centavos.

## 3. Decisões técnicas

| Decisão | Escolha | Por quê |
| --- | --- | --- |
| Linguagem | Python 3.12 com FastAPI | Simples, com pouco código para gerar |
| Armazenamento | Em memória, com lock | O contrato não pede persistência em disco |
| Dinheiro | Centavos inteiros, nunca float | Float acumula erro de arredondamento |
| Horário | Fuso -03:00, relógio isolado em um módulo | Facilita trocar o "agora" nos testes |
| Porta | Escuta em 8002 por padrão, sem variável obrigatória | O contrato dispensa configuração externa |
| Empacotamento | Containerfile e requirements.txt com versões fixas | Roda igual na correção |

## 4. Organização do código

| Camada | Faz | Não faz |
| --- | --- | --- |
| Rotas | Recebe a requisição e devolve status e JSON | Regras de negócio |
| Serviço | Aplica as regras (404, 409) | Calcula valor |
| Cálculo de valor | Função pura de minutos para centavos | Acessa relógio ou armazenamento |
| Armazenamento | Guarda e busca bilhetes | Regras de negócio |

> [!WARNING]
> Ordem das verificações: primeiro formato inválido (422), depois bilhete
> inexistente (404), por último conflito de estado (409).

## 5. Como calcular minutos e valor

1. `saida` é o horário do servidor no momento do encerramento.
2. `minutos` = diferença em segundos dividida por 60, **truncada**.
3. Se `entrada` estiver no futuro: `minutos = 0` e `valor_centavos = 0`.
4. `frações` = minutos ÷ 30, **arredondado para cima**.
5. `valor_centavos` = frações × 225, limitado a 5000.

> [!WARNING]
> Truncar os minutos é obrigatório: 90 min + milissegundos deve dar 90
> minutos (3 frações), nunca 91.

| minutos | valor_centavos |
| --- | --- |
| 0 | 0 |
| 1 | 225 |
| 30 | 225 |
| 31 | 450 |
| 90 | 675 |
| 91 | 900 |
| 661 | 5000 (teto) |

## 6. Regras de formato e erro

- Todo erro tem o formato `{"erro": "<codigo>"}`; o erro padrão do
  framework deve ser substituído por esse.
- Placa válida: 7 caracteres, letras maiúsculas e números.
- `entrada` precisa de fuso; sem fuso é `entrada_invalida`.
- Id inexistente ou que não seja número: 404 `bilhete_nao_encontrado`.

## 7. Segurança, LGPD e auditoria

| Requisito | Regra |
| --- | --- |
| Dado pessoal | Placa é dado pessoal indireto (identifica o dono via cadastro do veículo) |
| Logs mascarados | Logs exibem só os 3 primeiros caracteres da placa: `ABC****` |
| Respostas da API | Mantêm a placa completa (exigido pelo contrato) |
| Retenção | Dados só em memória; nada é gravado em disco nem em arquivo de log |
| Segredos | Nenhuma senha, token ou chave no repositório |
| Erros | Nunca expõem stack trace nem detalhes internos; só `{"erro": "<codigo>"}` |

### Log de auditoria

Cada mudança de estado gera uma linha JSON na saída padrão (stdout):

| Campo | Exemplo |
| --- | --- |
| evento | `bilhete_aberto`, `bilhete_encerrado`, `bilhete_cancelado` |
| id | 1 |
| placa | `ABC****` |
| momento | `2026-10-07T20:15:00-03:00` |

> [!IMPORTANT]
> Logs nunca alteram o corpo nem o status das respostas da API.