# Tarefas de Implementação

## Tarefa 1 — Estrutura e configuração

- Criar projeto Java 21 com Spring Boot e Maven.
- Configurar execução da API.
- Configurar porta 8002.
- Criar Containerfile/Dockerfile.
- Criar README de execução.
- Configurar testes automatizados.

## Tarefa 2 — Modelo e persistência

- Criar representação do bilhete.
- Implementar estados aberto, encerrado e cancelado.
- Implementar identificador único.
- Criar Repository para armazenar e consultar bilhetes.
- Implementar consultas por ID, placa, status e data.

## Tarefa 3 — Abertura de bilhetes

- Implementar POST /bilhetes.
- Validar placa.
- Validar entrada ISO-8601.
- Aplicar regra de uma placa aberta por vez.
- Implementar geração do ID e estado inicial aberto.

## Tarefa 4 — Encerramento e cobrança

- Implementar POST /bilhetes/{id}/encerramento.
- Calcular duração.
- Aplicar tolerância.
- Calcular frações arredondando para cima.
- Aplicar tarifa.
- Aplicar teto.
- Retornar valores em centavos inteiros.

## Tarefa 5 — Cancelamento

- Implementar POST /bilhetes/{id}/cancelamento.
- Permitir cancelamento somente de bilhetes abertos.
- Garantir ausência de cobrança e saída.

## Tarefa 6 — Consultas

- Implementar GET /bilhetes/ativos.
- Implementar GET /bilhetes?placa=...
- Garantir ordenação dos mais recentes primeiro.

## Tarefa 7 — Relatório diário

- Implementar GET /relatorios/diario.
- Validar data.
- Filtrar encerramentos pela data local.
- Calcular quantidade.
- Calcular faturamento.
- Calcular média de duração.
- Aplicar arredondamento 0,5 para cima.

## Tarefa 8 — Testes e validação final

- Criar testes unitários para regras de cobrança.
- Criar testes de integração dos endpoints.
- Testar casos de borda.
- Testar conflitos e transições de estado.
- Testar respostas de erro.
- Executar a suíte de testes.
- Verificar execução em container.
- Verificar que nenhum segredo foi incluído no repositório.