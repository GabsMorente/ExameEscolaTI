# Test Plan — Zona Azul Digital

## Abertura

* Placa válida `ABC1D23` → `201`.
* Placa ausente → `422` `placa_invalida`.
* Placa com menos de 7 caracteres → `422` `placa_invalida`.
* Placa com mais de 7 caracteres → `422` `placa_invalida`.
* Placa em minúsculo → `422` `placa_invalida`.
* Entrada `2026-10-07T08:00:00-03:00` → `201`.
* Entrada no formato `07/10/2026 08:00` → `422` `entrada_invalida`.

## Frações e cobrança

Com tarifa de `450` centavos/hora e frações de 30 minutos:

| Duração | Frações | Valor |
| ------: | ------: | ----: |
|   1 min |       1 |   225 |
|  29 min |       1 |   225 |
|  30 min |       1 |   225 |
|  31 min |       2 |   450 |
|  59 min |       2 |   450 |
|  60 min |       2 |   450 |
|  61 min |       3 |   675 |

Testar principalmente cada minuto exato de mudança de fração e o minuto seguinte.

## Tolerância

Com tolerância de 15 minutos:

* `0`, `1`, `14` e `15` minutos → `0` centavos.
* `16` minutos → 1 fração, `225` centavos.
* `29` minutos → 1 fração, `225` centavos.
* `30` minutos → 1 fração, `225` centavos.
* `31` minutos → 2 frações, `450` centavos.

A tolerância não deve ser descontada da duração. Ela somente torna os primeiros 15 minutos gratuitos; após a tolerância, a cobrança considera a duração integral.

## Teto de cobrança

Testar:

* Duração abaixo do teto.
* Duração que resulte exatamente em `5000` centavos.
* Duração que ultrapasse o teto.

O valor nunca pode ultrapassar `5000` centavos.

## Encerramento

* Encerrar bilhete aberto → `200`.
* ID inexistente → `404` `bilhete_nao_encontrado`.
* Encerrar bilhete já encerrado → `409` `bilhete_ja_encerrado`.
* Repetir encerramento → `409` `bilhete_ja_encerrado`.

## Cancelamento

* Cancelar bilhete aberto → `200`.
* ID inexistente → `404` `bilhete_nao_encontrado`.
* Cancelar bilhete encerrado → `409` `bilhete_nao_aberto`.
* Cancelar bilhete já cancelado → `409` `bilhete_nao_aberto`.
* Após cancelamento, o status deve ser `CANCELADO`.
* Bilhete cancelado não deve possuir `saida` nem `valor_centavos`.
* Bilhete cancelado não deve entrar no faturamento.

## Conflito de placa

* Abrir `ABC1D23`.
* Tentar abrir novamente a mesma placa → `409` `bilhete_em_aberto`.
* Encerrar o primeiro bilhete.
* Abrir novamente a placa → deve funcionar.
* Repetir o cenário usando cancelamento em vez de encerramento → nova abertura também deve funcionar.

## Bilhetes ativos

* Criar bilhetes com diferentes estados.
* `GET /bilhetes/ativos` deve retornar somente bilhetes `ABERTO`.
* Verificar ordem do mais recente para o mais antigo.
* Sem bilhetes ativos → `[]`.

## Histórico por placa

* Criar vários bilhetes da mesma placa com estados `ABERTO`, `ENCERRADO` e `CANCELADO`.
* O histórico deve retornar todos, independentemente do estado.
* Verificar ordem do mais recente para o mais antigo.
* Placa sem histórico → `[]`.

## Relatório diário

* Criar bilhetes encerrados em dias diferentes.
* O relatório deve considerar somente os encerrados na data consultada.
* Validar `total_bilhetes`.
* Validar `faturamento_centavos`.
* Validar `tempo_medio_minutos`.
* Bilhetes abertos não devem entrar no relatório.
* Bilhetes cancelados não devem entrar no relatório.

### Média com `.5`

Com durações de `10` e `11` minutos:

* Média = `10,5`.
* Resultado esperado = `11` minutos.

O arredondamento deve ser para cima quando o valor for exatamente `.5`.

## Data inválida

Testar:

* Data ausente.
* Data no formato `DD/MM/AAAA`.
* Data contendo texto.
* Data incompleta.

Todos devem retornar `422` com `data_invalida`.

## Ordem das validações

A validação de formato deve ocorrer antes das regras de negócio.

Exemplo:

* Enviar uma placa inválida que também possui um bilhete aberto.
* Resultado esperado: `422` `placa_invalida`, e não `409` `bilhete_em_aberto`.

A mesma prioridade deve ser aplicada às demais validações de formato.

## Valores monetários

* `valor_centavos` deve ser sempre um número inteiro.
* Não podem existir valores monetários decimais ou em ponto flutuante.
* O valor nunca pode ultrapassar `5000` centavos.
* `faturamento_centavos` do relatório também deve ser retornado como inteiro.
