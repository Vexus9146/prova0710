### UC1 — Abrir Bilhete
- **Endpoint**: `POST /bilhetes`
- **Descrição**: Abre um novo bilhete para a placa informada.
- **Requisição Body**:
{ "placa": "ABC1D23", "entrada": "2026-10-07T10:00:00-03:00" }


### UC2 — Encerrar Bilhete
- **Endpoint**: `POST /bilhetes/{id}/encerramento`
- **Descrição**: Encerra um bilhete aberto e calcula o valor a pagar com base no tempo decorrido.
- **Resposta Sucesso (200 OK)**:
{ "id": 1, "placa": "ABC1D23", "entrada": "2026-10-07T10:00:00-03:00", "saida": "2026-10-07T11:35:00-03:00", "minutos": 95, "valor_centavos": 900 }

- **Regras de Cálculo**:
  - Arredonda tempo cobrado sempre para cima na granularidade de 30 minutos.
  - Cada fração de 30 minutos custa 225 centavos (450 / 2).
  - Exemplo: 95 minutos equivalem a 4 frações (120 min) = 900 centavos.
  - Valor máximo cobrado não pode ultrapassar o teto diário de 5000 centavos.
  - O campo `valor_centavos` deve ser estritamente um número inteiro (`int`).
- **Critérios de Aceite**:
  - Bilhete `{id}` inexistente retorna status `404` `{"erro": "bilhete_nao_encontrado"}`.
  - Bilhete `{id}` já encerrado retorna status `409` `{"erro": "bilhete_ja_encerrado"}`.
  - Bilhete cancelado retorna status `409` `{"erro": "bilhete_nao_aberto"}`.

### UC3 — Listar Ativos
- **Endpoint**: `GET /bilhetes/ativos`
- **Descrição**: Retorna a lista de todos os bilhetes atualmente com status `"aberto"`.
- **Resposta Sucesso (200 OK)**:
[ { "id": 2, "placa": "XYZ9K88", "entrada": "2026-10-07T14:00:00-03:00", "status": "aberto" } ]

- **Critérios de Aceite**:
  - A lista deve vir ordenada do bilhete mais recente para o mais antigo.
  - Sem bilhetes abertos, deve retornar uma lista vazia `[]` com status `200`.

### UC4 — Relatório Diário
- **Endpoint**: `GET /relatorios/diario?data=AAAA-MM-DD`
- **Descrição**: Gera o resumo consolidado de faturamento e tempo médio do dia especificado.
- **Resposta Sucesso (200 OK)**:
{ "data": "2026-10-07", "totalbilhetes": 12, "faturamentocentavos": 8400, "tempomediominutos": 47 }

- **Critérios de Aceite**:
  - `tempo_medio_minutos` considera apenas bilhetes encerrados no dia.
  - Arredondamento do tempo médio é 0,5 para cima (half-up).
  - Bilhetes cancelados ou abertos não entram no faturamento nem no tempo médio.
  - Parâmetro `data` inválido ou ausente retorna status `422` `{"erro": "data_invalida"}`.

### UC5 — Cancelar Bilhete
- **Endpoint**: `POST /bilhetes/{id}/cancelamento`
- **Descrição**: Cancela um bilhete que esteja com status `"aberto"`.
- **Resposta Sucesso (200 OK)**:
{ "id": 1, "placa": "ABC1D23", "entrada": "2026-10-07T10:00:00-03:00", "status": "cancelado" }

- **Critérios de Aceite**:
  - Somente bilhetes com status `"aberto"` podem ser cancelados.
  - Não gera cobrança (sem campos `saida` ou `valor_centavos`).
  - Tentar cancelar bilhete encerrado ou cancelado retorna status `409` `{"erro": "bilhete_nao_aberto"}`.
  - Bilhete inexistente retorna status `404` `{"erro": "bilhete_nao_encontrado"}`.

### UC6 — Histórico por Placa
- **Endpoint**: `GET /bilhetes?placa=ABC1D23`
- **Descrição**: Retorna o histórico de todos os bilhetes da placa informada.
- **Resposta Sucesso (200 OK)**:
[ { "id": 1, "placa": "ABC1D23", "entrada": "2026-10-06T09:00:00-03:00", "saida": "2026-10-06T10:00:00-03:00", "minutos": 60, "valor_centavos": 450, "status": "encerrado" } ]

- **Critérios de Aceite**:
  - Inclui bilhetes de todos os status (`aberto`, `encerrado`, `cancelado`).
  - Ordenados do mais recente para o mais antigo.
  - Placa sem registros retorna lista vazia `[]` com status `200`.
  - Parâmetro `placa` inválido ou ausente retorna status `422` `{"erro": "placa_invalida"}`.

### UC7 — Tolerância Gratuita
- **Descrição da Regra**:
  - Para esta variante, `TOLERANCIA_MINUTOS = 0`.
  - Qualquer tempo de permanência maior que 0 minutos sofre cobrança integral da primeira fração de 30 minutos (225 centavos).

### UC8 — Uma Vaga por Placa
- **Descrição da Regra**:
  - Cada placa pode ter no máximo 1 bilhete com status `"aberto"` por vez.
  - Tentar abrir novo bilhete (`POST /bilhetes`) para placa com bilhete ativo retorna status `409` com o corpo: