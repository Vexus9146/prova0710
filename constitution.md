# Constitution — Zona Azul Digital

## Regras Operacionais Invariáveis

1. **Unidade Monetária**: Todos os valores monetários lidados no sistema (tarifas, tetos e totais) devem ser representados exclusivamente como números inteiros positivos em **centavos** (`int`). O uso de números de ponto flutuante (`float`) é estritamente proibido para operações financeiras.
2. **Porta de Execução**: A API deve escutar obrigatoriamente na porta configurada da variante: **`8002`**.
3. **Formato de Respostas de Erro**: Todo erro de validação ou de regra de negócio deve obrigatoriamente responder com o status HTTP correspondente e um corpo JSON no formato `{"erro": "codigo_do_erro"}`.
4. **Formato de Datas**: Todas as entradas e saídas de data/hora contendo horário devem estar estritamente no padrão ISO-8601 com fuso horário `-03:00`.
5. **Apenas Especificação**: Toda alteração neste projeto deve ser mantida na forma de especificações descritivas em Markdown (`.md`), sem arquivos de implementação em código.