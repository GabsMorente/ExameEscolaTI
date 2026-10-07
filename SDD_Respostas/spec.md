# Specification — Zona Azul Digital

## 1. Objetivo

Implementar uma API REST para gerenciamento de bilhetes de estacionamento rotativo.

A API deve permitir abrir, encerrar e cancelar bilhetes, listar bilhetes ativos, consultar histórico por placa e emitir relatório diário.

Não fazem parte do escopo telas, autenticação, usuários ou back-office.

## 2. Configuração da variante

A implementação deve utilizar:

| Parâmetro | Valor |
|---|---:|
| TARIFA_HORA_CENTAVOS | 450 |
| FRACAO_MINUTOS | 30 |
| TETO_DIARIO_CENTAVOS | 5000 |
| TOLERANCIA_MINUTOS | 15 |
| PORTA_SERVICO | 8002 |
| Fuso | -03:00 |

A URL utilizada pela suíte será:

`http://localhost:8002`

---

# UC1 — Abrir bilhete

## Endpoint

`POST /bilhetes`

## Entrada

Body JSON:

- `placa`: obrigatório, exatamente 7 caracteres alfanuméricos maiúsculos.
- `entrada`: opcional, ISO-8601 com fuso.

Quando `entrada` não for informada, utilizar o instante atual.

Quando `entrada` for informada, utilizar exatamente o instante informado.

## Critérios de aceite

1. Uma placa válida sem bilhete aberto deve gerar HTTP `201`.
2. A resposta deve conter `id`, `placa`, `entrada` e `status`.
3. O status inicial deve ser `"aberto"`.
4. A entrada deve ser retornada em ISO-8601 com fuso `-03:00`.
5. Placa ausente ou inválida deve gerar HTTP `422` com `{"erro":"placa_invalida"}`.
6. Entrada inválida deve gerar HTTP `422` com `{"erro":"entrada_invalida"}`.
7. Se já existir bilhete aberto para a placa, deve retornar HTTP `409` com `{"erro":"bilhete_em_aberto"}`.
8. A validação de formato deve ocorrer antes da verificação de conflito.

---

# UC2 — Encerrar bilhete

## Endpoint

`POST /bilhetes/{id}/encerramento`

## Critérios de aceite

1. Bilhete existente e aberto deve ser encerrado com HTTP `200`.
2. A resposta deve conter `id`, `placa`, `entrada`, `saida`, `minutos` e `valor_centavos`.
3. `saida` deve representar o instante do encerramento.
4. `minutos` deve representar a duração entre entrada e saída em minutos.
5. Bilhete inexistente deve retornar `404` com `{"erro":"bilhete_nao_encontrado"}`.
6. Bilhete já encerrado deve retornar `409` com `{"erro":"bilhete_ja_encerrado"}`.

## Regra de cobrança

A duração deve ser convertida em minutos e a cobrança deve usar frações de 30 minutos, sempre arredondando para cima.

Com tarifa de R$ 4,50 por hora e fração de 30 minutos:

- 1 fração = 225 centavos;
- 30 minutos = 225 centavos;
- 31 minutos = 450 centavos;
- 60 minutos = 450 centavos;
- 61 minutos = 675 centavos.

O valor final nunca pode ultrapassar 5000 centavos.

O valor retornado deve ser sempre um inteiro.

---

# UC3 — Listar bilhetes ativos

## Endpoint

`GET /bilhetes/ativos`

## Critério de aceite

Deve retornar HTTP `200` contendo somente bilhetes com status `"aberto"`, ordenados dos mais recentes para os mais antigos.

Se não houver bilhetes abertos, deve retornar um array vazio.

---

# UC4 — Relatório diário

## Endpoint

`GET /relatorios/diario?data=AAAA-MM-DD`

## Critérios de aceite

1. Data válida deve retornar HTTP `200`.
2. O campo `data` deve reproduzir a data consultada.
3. `total_bilhetes` deve representar os bilhetes encerrados naquele dia.
4. `faturamento_centavos` deve somar os valores cobrados dos bilhetes encerrados naquele dia.
5. `tempo_medio_minutos` deve considerar somente bilhetes encerrados naquele dia.
6. Média com parte decimal de exatamente 0,5 deve ser arredondada para cima.
7. Data inválida deve retornar HTTP `422` com `{"erro":"data_invalida"}`.
8. Quando não houver bilhetes encerrados na data, retornar totais zerados.

---

# UC5 — Cancelar bilhete

## Endpoint

`POST /bilhetes/{id}/cancelamento`

## Critérios de aceite

1. Bilhete aberto deve ser cancelado com HTTP `200`.
2. A resposta deve possuir status `"cancelado"`.
3. Bilhete cancelado não deve possuir `saida`.
4. Bilhete cancelado não deve possuir `valor_centavos`.
5. Bilhete inexistente deve retornar `404`.
6. Bilhete encerrado ou cancelado deve retornar `409` com `{"erro":"bilhete_nao_aberto"}`.

---

# UC6 — Histórico por placa

## Endpoint

`GET /bilhetes?placa=ABC1D23`

## Critérios de aceite

1. Placa válida deve retornar HTTP `200`.
2. O resultado deve conter todos os bilhetes da placa, independentemente do status.
3. Os registros devem ser ordenados dos mais recentes para os mais antigos.
4. Placa sem histórico deve retornar array vazio.
5. Placa ausente ou inválida deve retornar `422` com `{"erro":"placa_invalida"}`.

---

# UC7 — Tolerância gratuita

A tolerância configurada é de 15 minutos.

## Critérios de aceite

1. Duração de até 15 minutos deve gerar `valor_centavos = 0`.
2. Duração de 16 minutos deve iniciar cobrança integral desde o primeiro minuto.
3. A tolerância não deve ser subtraída da duração quando a cobrança for iniciada.
4. Com tolerância igual a zero, toda duração deve ser cobrada normalmente.
5. A tolerância deve ser aplicada antes do cálculo das frações cobradas.

---

# UC8 — Uma vaga por placa

## Critérios de aceite

1. Uma placa não pode possuir dois bilhetes abertos simultaneamente.
2. A segunda abertura deve retornar HTTP `409` com `{"erro":"bilhete_em_aberto"}`.
3. Após encerramento, a placa pode abrir novo bilhete.
4. Após cancelamento, a placa pode abrir novo bilhete.
5. O conflito deve ser verificado somente depois das validações de formato.

---

# Regras gerais de erros

| Situação | HTTP | erro |
|---|---:|---|
| placa inválida | 422 | placa_invalida |
| entrada inválida | 422 | entrada_invalida |
| data inválida | 422 | data_invalida |
| bilhete inexistente | 404 | bilhete_nao_encontrado |
| encerramento repetido | 409 | bilhete_ja_encerrado |
| cancelamento de não aberto | 409 | bilhete_nao_aberto |
| placa já ocupada | 409 | bilhete_em_aberto |

A validação de formato sempre deve ocorrer antes das regras de conflito.

## Regras de cálculo

A tarifa é de 450 centavos por hora.

A fração é de 30 minutos.

Cada fração custa 225 centavos.

A quantidade de frações deve ser arredondada para cima.

A cobrança deve respeitar o teto de 5000 centavos.

Valores monetários devem permanecer inteiros durante todo o cálculo.

## Formato temporal

Todas as datas e horários devem utilizar ISO-8601 com fuso `-03:00`.

O instante utilizado para calcular a duração deve ser controlável nos testes.

## Persistência e estado

Os bilhetes devem manter seu estado durante a execução da aplicação.

Os estados permitidos são:

- `aberto`
- `encerrado`
- `cancelado`

As transições permitidas são:

`aberto -> encerrado`

`aberto -> cancelado`

Após encerramento ou cancelamento, não é permitido retornar ao estado aberto.