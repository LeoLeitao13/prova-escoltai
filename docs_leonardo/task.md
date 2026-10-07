# Tarefas — Zona Azul Digital

Cada tarefa cria os próprios testes (pytest, em `tests/`) e só termina
quando eles passam. Executar na ordem.

## T1 — Esqueleto, constantes e relógio
- **Fazer:** projeto FastAPI; módulo de constantes da variante; módulo de
  relógio que devolve "agora" em -03:00.
- **Depende de:** —
- **Referências:** constitution 2 e 7; plan seções 2 e 3
- **Pronto quando:** app sobe na porta 8002; nenhum número da variante fora
  do módulo de constantes.

## T2 — Cálculo de valor (função pura)
- **Fazer:** função minutos → centavos: tolerância → fração → teto.
- **Depende de:** T1
- **Referências:** constitution 3; spec UC2 e UC7; tests R1, R3, R4
- **Pronto quando:** todos os casos de R1, R3 e R4 passam.

## T3 — Modelo, armazenamento e auditoria
- **Fazer:** entidade Bilhete; armazenamento em memória com lock; ids
  sequenciais a partir de 1; log JSON a cada mudança de estado, com placa
  mascarada.
- **Depende de:** T1
- **Referências:** plan seções 3 e 7; constitution 8
- **Pronto quando:** ids nunca se repetem; log não contém placa completa.

## T4 — Validação e formato de erro
- **Fazer:** validar placa, `entrada` e `data`; handler global que devolve
  sempre `{"erro": "<codigo>"}`; precedência 422 → 404 → 409.
- **Depende de:** T1
- **Referências:** constitution 4 e 6; tests R10
- **Pronto quando:** todos os casos de R10 passam; nenhuma resposta contém
  `detail`.

## T5 — Abrir bilhete e conflito de placa (UC1, UC8)
- **Fazer:** `POST /bilhetes`, com `entrada` opcional; 409 se a placa já
  tiver bilhete aberto.
- **Depende de:** T3, T4
- **Referências:** spec UC1 e UC8; tests R8
- **Pronto quando:** CA1.x, CA8.x e R8 passam.

## T6 — Encerrar bilhete (UC2)
- **Fazer:** `POST /bilhetes/{id}/encerramento`; calcular minutos com
  **truncamento**; entrada no futuro → 0 minutos; usar a função da T2.
- **Depende de:** T2, T5
- **Referências:** spec UC2; tests R2, R5, R9
- **Pronto quando:** CA2.x, R2, R5 e R9 (encerrar) passam.

## T7 — Cancelar bilhete (UC5)
- **Fazer:** `POST /bilhetes/{id}/cancelamento`; só bilhete aberto; sem
  `saida` nem `valor_centavos`.
- **Depende de:** T5
- **Referências:** spec UC5; tests R9
- **Pronto quando:** CA5.x e R9 (cancelar) passam.

## T8 — Listar ativos e histórico (UC3, UC6)
- **Fazer:** `GET /bilhetes/ativos` e `GET /bilhetes?placa=`; mais recentes
  primeiro.
- **Depende de:** T6, T7
- **Referências:** spec UC3 e UC6; tests R11
- **Pronto quando:** CA3.x, CA6.x e R11 passam.

## T9 — Relatório diário (UC4)
- **Fazer:** `GET /relatorios/diario?data=`; só bilhetes **encerrados** pela
  data da `saida`; tempo médio com 0,5 para cima.
- **Depende de:** T6
- **Referências:** spec UC4; tests R6, R7
- **Pronto quando:** CA4.x, R6 e R7 passam.

## T10 — Entregáveis de SDLC
- **Fazer:** `Containerfile` (expõe 8002); `requirements.txt` com versões
  fixas; `README.md` (como rodar, como testar); `.gitignore`.
- **Depende de:** T1–T9
- **Referências:** plan seção 3; constitution 7 e 8
- **Pronto quando:** container sobe e responde na porta 8002; README tem os
  comandos de execução e de teste.

## T11 — Verificação final
- **Fazer:** rodar todos os testes e conferir as regras da constitution.
- **Depende de:** T1–T10
- **Pronto quando:** `pytest` todo verde; nenhuma resposta com float ou
  `detail`; todas as datas em -03:00.