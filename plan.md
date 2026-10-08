# Plan — Zona Azul Digital

## 1. Stack Tecnológica e Decisões de Arquitetura

- **Linguagem & Framework**: Python 3.10+ com **FastAPI** (ou Flask/built-in `http.server`). FastAPI é recomendado pela validação automática de dados com Pydantic, geração de schema OpenAPI e velocidade de execução em requisições REST.
- **Porta de Execução**: O servidor deve obrigatoriamente escutar na porta **`8002`**, conforme parâmetro `PORTA_SERVICO` da variante.
- **Persistência de Dados**: Armazenamento em memória (dicionários/listas em memória ou banco SQLite local em arquivo/memória) para garantir performance e resposta determinística nos testes automatizados sem dependências externas de banco de dados.

## 2. Justificativa da Unidade Monetária (Centavos Inteiros)

Trabalhar com números em ponto flutuante (`float`) para representação financeira introduz imprecisões numéricas inerentes ao padrão IEEE 754 (ex.: `0.1 + 0.2 = 0.30000000000000004`). 
Para eliminar completamente essa classe de bugs de arredondamento e garantir precisão exata nas operações financeiras da API, todos os valores (tarifas, custos e faturamentos) são calculados e armazenados estritamente como **números inteiros em centavos** (`int`).

## 3. Estratégia de Relógio e Fuso Horário

- **Fuso Horário Padrão**: Todas as datas e horas devem ser processadas e retornadas no fuso horário **`-03:00`** (Horário de Brasília), formatadas em ISO-8601 (ex.: `2026-10-07T10:00:00-03:00`).
- **Injeção de Tempo (`entrada` opcional)**: O endpoint `POST /bilhetes` aceita um campo opcional `entrada`. Caso fornecido, este valor sobrepõe a hora do sistema (`now()`), permitindo simular permanências sem a necessidade de espera em tempo real durante a execução da suíte de testes.

## 4. Regras de Negócio e Fórmulas de Cálculo

- **Valor da Fração**: `TARIFA_HORA_CENTAVOS / (60 / FRACAO_MINUTOS)` = `450 / (60 / 30)` = **`225` centavos** por fração de 30 minutos.
- **Arredondamento do Tempo**: `frações = math.ceil(minutos / 30)`.
- **Cálculo de Cobrança**: `valor = min(frações * 225, TETO_DIARIO_CENTAVOS)`.
- **Tolerância**: `TOLERANCIA_MINUTOS = 0`. Permanências $> 0$ minutos pagam o valor integral da primeira fração.
- **Tempo Médio do Relatório (UC4)**: Calculado apenas sobre bilhetes com `status == "encerrado"` no dia `AAAA-MM-DD`. Arredondamento do tempo médio segue a regra **0,5 para cima** (`math.floor(media + 0.5)`).