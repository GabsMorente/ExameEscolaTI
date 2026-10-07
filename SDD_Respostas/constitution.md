# Constitution — Zona Azul Digital

## 1. Regras operacionais

- A API deve implementar exatamente os endpoints, métodos HTTP, status codes, campos e formatos definidos em `contrato.json`.
- Valores monetários devem ser representados exclusivamente como inteiros em centavos. Nunca retornar valores monetários em ponto flutuante.
- Datas e horários devem ser tratados em ISO-8601 com fuso `-03:00`.
- A aplicação deve respeitar os parâmetros da variante: `TARIFA_HORA_CENTAVOS=450`, `FRACAO_MINUTOS=30`, `TETO_DIARIO_CENTAVOS=5000`, `TOLERANCIA_MINUTOS=15` e `PORTA_SERVICO=8002`.
- Validações de formato devem ocorrer antes das validações de regras de negócio. Payload inválido deve produzir `422`, mesmo quando existiria algum conflito de estado.
- Uma placa pode possuir no máximo um bilhete com status aberto simultaneamente.
- Bilhetes cancelados não geram cobrança, saída ou valor em centavos.
- Bilhetes encerrados não podem ser encerrados novamente.
- Bilhetes encerrados ou cancelados não podem ser cancelados.
- Listagens devem respeitar a ordenação definida no contrato: registros mais recentes primeiro.
- O código gerado deve possuir testes automatizados para as regras de negócio e casos de borda descritos nesta especificação.
- A aplicação deve ser executável em container e disponibilizar o serviço na porta `8002`.

## 2. Princípios de implementação

- Priorizar comportamento determinístico e testável.
- O relógio utilizado para obter o instante atual deve ser abstraído para permitir testes com horários controlados.
- A lógica de cálculo de cobrança deve ficar separada da camada HTTP.
- A API deve retornar respostas JSON consistentes com o contrato.
- Não implementar funcionalidades de back-office, autenticação ou telas, pois elas estão fora do escopo da prova.