# Desafio PB

Obrigado pelo interesse demonstrado no meu perfil. Bom desafio, me diverti desenvolvendo ele, a ideia é simples mas deu para explorar muita coisa interessante.

## Arquitetura
### Versão 1 – Primeira ideia (mínima viável)
- **PB_Clientes (API REST):** cadastra o usuário e envia um evento para a fila analise-credito-queue.
- **PB_AnaliseCredito (Worker):** consome eventos da fila analise-credito-queue, realiza a análise de crédito e publica um evento em cartao-credito-queue.
- **PB_Cartao (Worker):** consome eventos da fila cartao-credito-queue, gera o cartão, salva no banco e publica em envio-cartao-queue.
>Observação: arquitetura enxuta, mas sem mecanismos de resiliência caso filas falhem ou workers estejam offline.

<img width="1362" height="271" alt="PB_Desafio_Versao_1" src="https://github.com/user-attachments/assets/43e13766-b679-4aaf-ad65-36ffe98ef8ea" />


### Versão 2 – Tratamento de erros nas filas
**Problema identificado:** se a publicação em uma fila falhar, a informação se perde.

**Solução:** criar filas de erro (dead-letter queues) para cada fila principal.

Introduzi um orquestrador para monitorar essas filas de erro e tentar reprocessar mensagens.
Caso exceda o número máximo de tentativas, a mensagem fica disponível em uma API REST para consulta e processamento manual.

<img width="1421" height="582" alt="PB_Desafio_Versao_2" src="https://github.com/user-attachments/assets/70156913-c279-412e-a039-d6a17fdd6097" />

### Versão 3 – Resiliência contra indisponibilidade do RabbitMQ
**Problema identificado:** se o RabbitMQ estiver fora do ar, todo o fluxo de mensagens falha.

**Solução:** implementar um outbox pattern:

Ao criar um usuário, as mensagens que seriam enviadas para o RabbitMQ são gravadas em uma tabela auxiliar no banco de dados (Outbox).
Um worker lê essa tabela e tenta enviar as mensagens para as filas corretas (que no caso é para a fila de analise de crédito).
Isso desacopla o cadastro de clientes da disponibilidade do RabbitMQ e garante que o usuário receba sucesso imediato, mesmo se o broker estiver offline.

**Principais Benefícios:**
Maior resiliência e confiabilidade;
Mensagens não se perdem;
Facilita monitoramento e troubleshooting.

<img width="1461" height="602" alt="PB_Desafio_Versao_3" src="https://github.com/user-attachments/assets/550e9f69-a1ae-4629-bbe9-558dd25ddeae" />


### versao 4 - Melhorias que poderiam ser implementadas futuramente
**Job scheduler + alertas**
- Monitorar a tabela Outbox e alertar se mensagens estiverem há muito tempo sem serem enviadas.
- Possibilita intervenção manual antes que o volume cresça demais.

**Idempotência e deduplicação**
- Adicionar controle de idempotência: cada mensagem tem ID único e o sistema verifica se já foi processada.
- Evita problemas de duplicação em reprocessamentos.

## Lista dos projetos
PB_Clientes (API REST)
PB_AnaliseCredito (Worker)
PB_Cartao (Worker)
PB_Orquestrador
- [PB_Clientes](https://github.com/DinisSimoes/PB_Clientes)
- [PB_AnaliseCredito](https://github.com/DinisSimoes/PB_AnaliseCredito)
- [PB_Cartao](https://github.com/DinisSimoes/PB_Cartao)
- [PB_Orquestrador](https://github.com/DinisSimoes/PB_Orquestrador)
- [PB_Commom](https://github.com/DinisSimoes/PB_Common)
