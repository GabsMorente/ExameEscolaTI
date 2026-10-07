# Plano Técnico — Zona Azul Digital

## 1. Stack

Utilizar Java 21 com Spring Boot e Maven.

Justificativa: a stack fornece suporte adequado para criação de API REST, validação, testes automatizados e execução em container.

## 2. Arquitetura

Utilizar separação em camadas:

- Controller: HTTP, rotas, parâmetros e respostas.
- Service: regras de negócio e cálculos.
- Repository: armazenamento e consulta dos bilhetes.
- Model/Entity: representação dos bilhetes e seus estados.
- DTO: contratos de entrada e saída da API.

A lógica de cobrança não deve ficar no Controller.

## 3. Persistência

Para a aplicação gerada, utilizar uma implementação de persistência simples e determinística adequada à suíte de testes.

A camada Repository deve ser abstraída da camada Service para permitir substituição da implementação sem alterar as regras de negócio.

## 4. Identificadores

Os bilhetes devem possuir identificadores numéricos únicos e crescentes durante a execução da aplicação.

## 5. Relógio

Abstrair o acesso ao horário atual por meio de uma fonte de tempo injetável.

Isso permite testar duração, tolerância, frações e teto sem depender do relógio real.

## 6. Datas

Utilizar tipos de data/hora que preservem o offset.

Os horários da API devem ser tratados no fuso `-03:00`.

O relatório diário deve considerar a data local no fuso da aplicação.

## 7. Cálculo monetário

Não utilizar ponto flutuante para valores monetários.

Com os parâmetros desta variante:

- tarifa horária: 450 centavos;
- fração: 30 minutos;
- valor por fração: 225 centavos;
- teto: 5000 centavos.

O cálculo deve utilizar somente inteiros.

## 8. Tolerância

A tolerância deve ser avaliada antes da cobrança.

Com tolerância de 15 minutos:

- duração <= 15 minutos: valor 0;
- duração >= 16 minutos: cobrar a duração integral desde o primeiro minuto.

## 9. Arredondamentos

A duração cobrada deve ser arredondada sempre para a próxima fração quando não for múltipla exata.

O tempo médio do relatório deve arredondar 0,5 para cima.

Não utilizar arredondamento bancário/half-even para o tempo médio.

## 10. Container

A aplicação deve possuir Containerfile/Dockerfile e manifesto de dependências necessário para construção e execução.

O serviço deve ficar acessível na porta `8002`, conforme a variante.

## 11. Testes

Criar testes automatizados para:

- endpoints principais;
- validações;
- estados dos bilhetes;
- tolerância;
- arredondamento de frações;
- teto;
- relatório;
- conflitos de placa;
- cancelamento;
- histórico;
- casos de borda.

## 12. README

O projeto gerado deve possuir README contendo:

- objetivo;
- stack;
- instruções para executar;
- instruções para executar testes;
- porta utilizada;
- exemplos básicos de uso da API.

## 13. Segurança e higiene

Não incluir senhas, tokens, chaves ou outros segredos no repositório.

Manter dependências e configuração mínimas necessárias para a execução.