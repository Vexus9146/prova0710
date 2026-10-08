# Tasks — Decomposição da Implementação

## Tarefa 1: Setup do Projeto e Estrutura de Dados
- Criar a estrutura base do projeto em Python/FastAPI.
- Carregar ou fixar as variáveis de configuração da variante (`PORTA_SERVICO = 8002`, `TARIFA_HORA_CENTAVOS = 450`, `FRACAO_MINUTOS = 30`, `TETO_DIARIO_CENTAVOS = 5000`, `TOLERANCIA_MINUTOS = 0`).
- Modelar as coleções de armazenamento em memória para os bilhetes.

## Tarefa 2: Endpoints do Ciclo de Vida do Bilhete (UC1, UC3, UC5, UC6, UC8)
- Implementar `POST /bilhetes` para abertura de bilhetes com validação de placa de 7 caracteres maiúsculos, parser de `entrada` em ISO-8601 fuso `-03:00` e bloqueio de placa já aberta (**409**).
- Implementar `GET /bilhetes/ativos` ordenados do mais recente para o mais antigo.
- Implementar `POST /bilhetes/{id}/cancelamento` (somente bilhetes abertos) e `GET /bilhetes?placa={placa}` com histórico completo.

## Tarefa 3: Motor de Cálculo Financeiro e Encerramento (UC2, UC7)
- Criar a função utilitária de cálculo financeiro:
  - Arredondamento do tempo em frações de 30 minutos para cima (`math.ceil`).
  - Cálculo de centavos por fração (`225`).
  - Aplicação do teto diário (`5000`).
- Implementar `POST /bilhetes/{id}/encerramento` registrando a `saida`, `minutos` e `valor_centavos`.

## Tarefa 4: Relatório Diário e Validações de Erros (UC4 e Middleware de Exceção)
- Implementar `GET /relatorios/diario?data=AAAA-MM-DD` com o cálculo consolidado de faturamento e arredondamento do `tempo_medio_minutos` (**0,5 para cima**).
- Mapear e padronizar os erros HTTP (422, 404, 409) para que todos retornem no formato exato `{"erro": "codigo_do_erro"}`.