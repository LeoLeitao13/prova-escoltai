# Constituição — Zona Azul Digital

Regras fixas que valem para todo o código gerado. Nenhuma tarefa pode
violá-las.

## 1. Dinheiro

- Todo valor monetário é **centavo inteiro** (`valor_centavos`,
  `faturamento_centavos`).
- **Nunca** usar float, nem em cálculos intermediários: só aritmética inteira.
- Na resposta JSON, o valor é inteiro: `675`, nunca `675.0`.

## 2. Datas e horas

- Fuso fixo **-03:00** (horário de Brasília).
- Formato de saída: `AAAA-MM-DDTHH:MM:SS-03:00`, sem microssegundos.
  Exemplo: `2026-10-07T20:15:00-03:00`.
- `entrada` com outro fuso é aceita e convertida para -03:00 na resposta.
- `entrada` sem fuso ou fora de ISO-8601 → 422 `entrada_invalida`.

## 3. Cobrança (sempre nesta ordem)

0. Minutos = (saída − entrada) em segundos ÷ 60, **truncado**. Se negativo,
   minutos = 0.
1. **Tolerância:** se minutos ≤ `TOLERANCIA_MINUTOS`, valor = 0. Senão cobra
   integral, **sem descontar** a tolerância.
2. **Fração:** frações = minutos ÷ `FRACAO_MINUTOS`, **arredondado para cima**.
3. **Valor:** frações × 225 centavos.
4. **Teto:** valor final nunca passa de `TETO_DIARIO_CENTAVOS` (5000).

## 4. Erros

- Todo erro tem o corpo **exatamente** `{"erro": "<codigo>"}`, sem outros
  campos.
- O erro padrão do framework (ex.: `{"detail": [...]}`) deve ser substituído.
- Só usar os códigos do contrato: `placa_invalida`, `entrada_invalida`,
  `data_invalida`, `bilhete_nao_encontrado`, `bilhete_ja_encerrado`,
  `bilhete_nao_aberto`, `bilhete_em_aberto`.

## 5. Contrato acima de exemplos

- Nomes de campos e códigos seguem o contrato **letra por letra**, sem
  renomear nem traduzir.

> [!CAUTION]
> O exemplo `{"id": 7, "valor": 12.50}` do enunciado está errado. Usar
> sempre `valor_centavos` inteiro.

## 6. Precedência de erros

Verificar nesta ordem: **422** (formato) → **404** (inexistente) →
**409** (conflito). Payload malformado nunca gera 409.

## 7. Configuração

- Parâmetros da variante existem **só** como constantes nomeadas em um
  único módulo (450, 30, 5000, 0, 8002).
- O serviço escuta na porta **8002** sem variável de ambiente obrigatória.

## 8. Segurança e LGPD

- Logs mostram a placa mascarada (`ABC****`); respostas da API mantêm a
  placa completa.
- Nenhum segredo (senha, token, chave) no repositório.
- Erros nunca expõem stack trace.