# Specification — Zona Azul Digital

## Configurações da Variante

- **Porta do Serviço (`PORTA_SERVICO`)**: `8002`
- **Tarifa Hora (`TARIFA_HORA_CENTAVOS`)**: `450` centavos (R$ 4,50/h)
- **Fração Mínima (`FRACAO_MINUTOS`)**: `30` minutos
- **Valor da Fração**: `225` centavos por bloco de 30 minutos
- **Teto Diário (`TETO_DIARIO_CENTAVOS`)**: `5000` centavos (R$ 50,00)
- **Tolerância Gratuita (`TOLERANCIA_MINUTOS`)**: `0` minutos (sem tempo grátis)

---

## Base URL

`http://localhost:8002`

---

## Casos de Uso (UCs)

### UC1 — Abrir Bilhete
- **Endpoint**: `POST /bilhetes`
- **Descrição**: Abre um novo bilhete para a placa informada.
- **Requisição Body**:
  ```json
  {
    "placa": "ABC1D23",
    "entrada": "2026-10-07T10:00:00-03:00"
  }
(Nota: O campo entrada é opcional. Se omitido, utiliza o instante atual).Resposta Sucesso (201 Created):JSON{
  "id": 1,
  "placa": "ABC1D23",
  "entrada": "2026-10-07T10:00:00-03:00",
  "status": "aberto"
}
Critérios de Aceite:A placa deve ter exatamente 7 caracteres alfanuméricos maiúsculos.Se a placa já possuir um bilhete em aberto, deve retornar status 409 {"erro": "bilhete_em_aberto"}.Se a placa for inválida ou ausente, deve retornar status 422 {"erro": "placa_invalida"}.Se o campo entrada for fornecido e estiver fora do padrão ISO-8601 com fuso -03:00, retornar status 422 {"erro": "entrada_invalida"}.UC2 — Encerrar BilheteEndpoint: POST /bilhetes/{id}/encerramentoDescrição: Encerra um bilhete aberto e calcula o valor a pagar com base no tempo decorrido.Resposta Sucesso (200 OK):JSON{
  "id": 1,
  "placa": "ABC1D23",
  "entrada": "2026-10-07T10:00:00-03:00",
  "saida": "2026-10-07T11:35:00-03:00",
  "minutos": 95,
  "valor_centavos": 900
}
Regras de Cálculo de Valor:O tempo cobrado é arredondado sempre para cima na granularidade de FRACAO_MINUTOS (30 min).Cada fração de 30 minutos custa 225 centavos (450 / (60 / 30)).Exemplo: 95 minutos equivalem a 4 frações de 30 minutos (120 min de cobrança) = 4 * 225 = 900 centavos.O valor total retornado em valor_centavos nunca pode ultrapassar o teto diário de 5000 centavos.O campo valor_centavos deve ser estritamente um número inteiro (int), nunca ponto flutuante.Critérios de Aceite:Se o bilhete com {id} não existir, retornar 404 {"erro": "bilhete_nao_encontrado"}.Se o bilhete com {id} já estiver encerrado, retornar 409 {"erro": "bilhete_ja_encerrado"}.Se o bilhete estiver cancelado, retornar 409 {"erro": "bilhete_nao_aberto"}.UC3 — Listar AtivosEndpoint: GET /bilhetes/ativosDescrição: Retorna a lista de todos os bilhetes que estão atualmente com status: "aberto".Resposta Sucesso (200 OK):JSON[
  {
    "id": 2,
    "placa": "XYZ9K88",
    "entrada": "2026-10-07T14:00:00-03:00",
    "status": "aberto"
  }
]
Critérios de Aceite:A lista deve ser ordenada do bilhete mais recente para o mais antigo.Se não houver bilhetes abertos, deve retornar uma lista vazia [] com status 200.UC4 — Relatório DiárioEndpoint: GET /relatorios/diario?data=AAAA-MM-DDDescrição: Gera o resumo consolidado de faturamento e tempo médio do dia especificado.Resposta Sucesso (200 OK):JSON{
  "data": "2026-10-07",
  "total_bilhetes": 12,
  "faturamento_centavos": 8400,
  "tempo_medio_minutos": 47
}
Critérios de Aceite:tempo_medio_minutos considera apenas os bilhetes que foram encerrados no dia especificado.O arredondamento do tempo médio em minutos deve ser 0,5 para cima (half-up para o inteiro mais próximo).Bilhetes cancelados ou ainda abertos não entram no cálculo de faturamento_centavos nem de tempo_medio_minutos.Se o parâmetro data for inválido ou ausente do padrão AAAA-MM-DD, retornar status 422 {"erro": "data_invalida"}.UC5 — Cancelar BilheteEndpoint: POST /bilhetes/{id}/cancelamentoDescrição: Cancela um bilhete que esteja com status "aberto".Resposta Sucesso (200 OK):JSON{
  "id": 1,
  "placa": "ABC1D23",
  "entrada": "2026-10-07T10:00:00-03:00",
  "status": "cancelado"
}
Critérios de Aceite:Somente bilhetes com status "aberto" podem ser cancelados.Não há cobrança para cancelamento (não gera os campos saida nem valor_centavos).Tentativa de cancelar bilhete já encerrado ou já cancelado deve retornar 409 {"erro": "bilhete_nao_aberto"}.Se o bilhete não existir, retornar 404 {"erro": "bilhete_nao_encontrado"}.UC6 — Histórico por PlacaEndpoint: GET /bilhetes?placa=ABC1D23Descrição: Retorna o histórico de todos os bilhetes associados à placa informada.Resposta Sucesso (200 OK):JSON[
  {
    "id": 5,
    "placa": "ABC1D23",
    "entrada": "2026-10-07T12:00:00-03:00",
    "status": "aberto"
  },
  {
    "id": 1,
    "placa": "ABC1D23",
    "entrada": "2026-10-06T09:00:00-03:00",
    "saida": "2026-10-06T10:00:00-03:00",
    "minutos": 60,
    "valor_centavos": 450,
    "status": "encerrado"
  }
]
Critérios de Aceite:Deve incluir bilhetes de todos os status (aberto, encerrado, cancelado).Os bilhetes devem vir ordenados dos mais recentes para os mais antigos.Caso a placa nunca tenha estacionado, retornar uma lista vazia [] com status 200.Se o parâmetro placa for inválido ou ausente, retornar 422 {"erro": "placa_invalida"}.UC7 — Tolerância GratuitaDescrição da Regra:Para esta variante, TOLERANCIA_MINUTOS = 0.Qualquer tempo de permanência maior que 0 minutos sofrerá cobrança integral da primeira fração de 30 minutos (225 centavos).UC8 — Uma Vaga por PlacaDescrição da Regra:Cada placa pode possuir no máximo um bilhete com status "aberto" por vez.Tentar abrir um novo bilhete (POST /bilhetes) para uma placa com bilhete aberto ativo deve retornar status 409 com o corpo:JSON{
  "erro": "bilhete_em_aberto"
}
Após o encerramento ou cancelamento do bilhete atual, a placa fica liberada para abrir novos bilhetes.Tabela Consolidada de Códigos de Erro HTTPErroStatus HTTPBody retornadoPlaca ausente ou fora do formato (7 alfanuméricos maiúsculos)422{"erro": "placa_invalida"}Data entrada fora do padrão ISO-8601 com fuso422{"erro": "entrada_invalida"}Data do relatório fora do padrão AAAA-MM-DD422{"erro": "data_invalida"}Bilhete inexistente no sistema404{"erro": "bilhete_nao_encontrado"}Tentativa de encerrar bilhete já encerrado409{"erro": "bilhete_ja_encerrado"}Tentativa de cancelar bilhete que não esteja aberto409{"erro": "bilhete_nao_aberto"}Tentativa de abrir bilhete para placa já ocupada409{"erro": "bilhete_em_aberto"}