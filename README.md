
# Resumo do Projeto Serverlesspresso

## 1. Introdução
Serverlesspresso é um projeto desenvolvido para gerenciar pedidos em uma cafeteria usando uma arquitetura orientada a eventos na AWS. O sistema é capaz de processar pedidos e coletar métricas de negócios.

## 2. Arquitetura do Sistema
O sistema é construído com os seguintes componentes:
- **AWS Lambda**: Funções que processam eventos e gerenciam a lógica do fluxo de trabalho.
- **EventBridge**: Usado para roteamento de eventos entre os componentes do sistema.
- **SQS (Simple Queue Service)**: Usado como buffer para gerenciar picos de tráfego e garantir que as mensagens sejam processadas de forma ordenada.
- **DynamoDB**: Armazenamento de dados para pedidos e métricas.
- **CloudWatch**: Monitoramento e coleta de métricas do sistema.

## 3. Configuração de Regras no EventBridge
### Criando a Regra "WaitingCompletion"
1. Acesse o console do EventBridge e selecione "Criar regra".
2. Nomeie a regra como `OrderManagerWaitingCompletion` e selecione o barramento de eventos `Serverlesspresso`.
3. Na configuração de evento, use o seguinte padrão:
   ```json
   {
       "source": ["awsserverlessda.serverlesspresso"],
       "detail-type": ["OrderManager.WaitingCompletion"]
   }
   ```
4. Configure o destino para uma fila SQS chamada `MetricsQueue-Cloudwatch`.

## 4. Testando o Sistema
### Teste de Ordem Única
- Crie um novo pedido e verifique o painel `Serverlesspresso_Metrics` no CloudWatch.
- As métricas devem mostrar o total de pedidos e a quantidade de bebidas vendidas.

### Teste de Carga
- Utilize a função Lambda `EventsLoadTest` para simular um grande número de pedidos.
- Monitore o painel `Serverlesspresso_Metrics` para ver como as métricas são atualizadas durante o teste.

## 5. Fluxo de Trabalho do Processador de Pedidos
### Teste com Pedidos em Excesso
- A capacidade máxima da loja é de 20 pedidos simultâneos.
- Ao enviar 21 pedidos, o fluxo de trabalho rejeitará novos pedidos.

### Teste de Tempo Limite
- Se o cliente não concluir o pedido em 5 minutos, a execução falhará.
- No console do Step Functions, as execuções com tempo limite serão exibidas como falhas.

## 6. Limpeza de Recursos
Para remover os recursos criados durante o workshop:
1. **Buckets S3**: 
   ```bash
   aws s3 ls | grep serverlesspresso
   aws s3 rb --force s3://your-bucket-name
   ```
2. **Recursos no CloudFormation**:
   ```bash
   aws cloudformation list-stacks | grep serverlesspresso
   aws cloudformation delete-stack --stack-name your-stack-name
   ```
3. **Regras do EventBridge**:
   ```bash
   aws events list-rules --event-bus-name Serverlesspresso
   aws events delete-rule --name 'your-rule-name'
   ```
4. **Excluir a Instância do Cloud9**: No console do Cloud9, selecione sua instância e escolha excluir.

## 7. Conclusão
O projeto Serverlesspresso demonstra como criar um sistema escalável e eficiente para gerenciar pedidos em uma cafeteria utilizando serviços serverless da AWS. A arquitetura orientada a eventos permite fácil extensão e coleta de métricas para monitoramento de desempenho.
