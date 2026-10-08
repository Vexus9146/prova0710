# Tests — Zona Azul Digital

## Matriz de Casos de Borda e Validações

### 1. Cálculo de Frações e Adjacências (UC2)

Como `FRACAO_MINUTOS = 30` e cada fração custa `225` centavos:

| Tempo Decorrido | Frações Cobradas | Valor Esperado (Centavos) | Motivo / Regra |
| :--- | :---: | :---: | :--- |
| **1 min** | 1 | `225` | Maior que 0 min (tolerância 0) → cobra 1ª fração completa |
| **30 min** | 1 | `225` | Limite exato da 1ª fração |
| **31 min** | 2 | `450` | 1 min a mais abre a 2ª fração integral |
| **60 min** | 2 | `450` | Limite exato de 1 hora cheia |
| **61 min** | 3 | `675` | Entra na 3ª fração |
| **95 min** | 4 | `900` | 3 frações cheias (90 min) + 5 min na 4ª fração |

> [!WARNING]
> Como `TOLERANCIA_MINUTOS = 0`, não há tempo gratuito nesta variante.

---

### 2. Aplicação do Teto Diário (UC2)

O teto diário da variante é `TETO_DIARIO_CENTAVOS = 5000` (R$ 50,00). Sem teto, cada hora cheia custaria 450 centavos (2 frações x 225).

| Tempo Decorrido | Frações Teóricas | Valor Sem Teto | Valor Cobrado | Comportamento |
| :--- | :---: | :---: | :---: | :--- |
| **600 min (10h)** | 20 | 4.500 centavos | **4.500 centavos** | Abaixo do teto |
| **660 min (11h)** | 22 | 4.950 centavos | **4.950 centavos** | Abaixo do teto |
| **690 min (11h30)** | 23 | 5.175 centavos | **5.000 centavos** | **Teto diário atingido e limitado** |
| **1.440 min (24h)** | 48 | 10.800 centavos | **5.000 centavos** | Permanência longa mantida no teto |

---

### 3. Arredondamento do Tempo Médio no Relatório Diário (UC4)

Regra: Arredondamento **0,5 para cima** (`half-up`) considerando apenas bilhetes encerrados no dia.

| Duração dos Bilhetes Encerrados no Dia | Soma / Qtd | Média Real | `tempo_medio_minutos` Esperado |
| :--- | :---: | :---: | :---: |
| 10 min, 20 min | 30 / 2 | 15.0 min | **15** |
| 10 min, 15 min | 25 / 2 | 12.5 min | **13** *(0,5 arredonda para cima)* |
| 10 min, 11 min, 12 min | 33 / 3 | 11.0 min | **11** |
| 10 min, 11 min | 21 / 2 | 10.5 min | **11** *(0,5 arredonda para cima)* |

---

### 4. Validações de Conflito e Erros HTTP

| Cenário de Teste | Endpoint | Condição | Status Esperado | Body Esperado |
| :--- | :--- | :--- | :---: | :--- |
| Placa minúscula ou fora do padrão | `POST /bilhetes` | `{"placa": "abc1d23"}` | **422** | `{"erro": "placa_invalida"}` |
| Placa com tamanho incorreto | `POST /bilhetes` | `{"placa": "ABC1D2"}` | **422** | `{"erro": "placa_invalida"}` |
| Formato de entrada inválido | `POST /bilhetes` | `{"placa": "ABC1D23", "entrada": "10/10/2026"}` | **422** | `{"erro": "entrada_invalida"}` |
| Duplicidade de vaga | `POST /bilhetes` | Abrir bilhete para placa que já está em aberto | **409** | `{"erro": "bilhete_em_aberto"}` |
| Encerramento duplo | `POST /bilhetes/1/encerramento` | Encerrar bilhete que já foi encerrado | **409** | `{"erro": "bilhete_ja_encerrado"}` |
| Cancelamento inválido | `POST /bilhetes/1/cancelamento` | Cancelar bilhete que já está encerrado | **409** | `{"erro": "bilhete_nao_aberto"}` |
| ID inexistente | `POST /bilhetes/999/encerramento` | ID não cadastrado no sistema | **404** | `{"erro": "bilhete_nao_encontrado"}` |