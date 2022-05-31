## Домашнее задание к занятию "7.6. Написание собственных провайдеров для Terraform."

Бывает, что:

-   общедоступная документация по терраформ ресурсам не всегда достоверна,
-   в документации не хватает каких-нибудь правил валидации или неточно описаны параметры,
-   понадобиться использовать провайдер без официальной документации,
-   может возникнуть необходимость написать свой провайдер для системы используемой в ваших проектах.

### Задача 1.

Давайте потренируемся читать исходный код AWS провайдера, который можно склонировать от сюда:  [https://github.com/hashicorp/terraform-provider-aws.git](https://github.com/hashicorp/terraform-provider-aws.git). Просто найдите нужные ресурсы в исходном коде и ответы на вопросы станут понятны.

1.  Найдите, где перечислены все доступные  `resource`  и  `data_source`, приложите ссылку на эти строки в коде на гитхабе.
2.  Для создания очереди сообщений SQS используется ресурс  `aws_sqs_queue`  у которого есть параметр  `name`.
    -   С каким другим параметром конфликтует  `name`? Приложите строчку кода, в которой это указано.
    -   Какая максимальная длина имени?
    -   Какому регулярному выражению должно подчиняться имя?

**Ответ:**

1.  resources:  [https://github.com/hashicorp/terraform-provider-aws/blob/3f454457baf318faae5cc61dcf948c6b7a25575a/internal/provider/provider.go#L871](https://github.com/hashicorp/terraform-provider-aws/blob/3f454457baf318faae5cc61dcf948c6b7a25575a/internal/provider/provider.go#L871)  
    data_sources:  [https://github.com/hashicorp/terraform-provider-aws/blob/3f454457baf318faae5cc61dcf948c6b7a25575a/internal/provider/provider.go#L412](https://github.com/hashicorp/terraform-provider-aws/blob/3f454457baf318faae5cc61dcf948c6b7a25575a/internal/provider/provider.go#L412)
    
2.  Параметр name конфликтует с name_prefix  [https://github.com/hashicorp/terraform-provider-aws/blob/3f454457baf318faae5cc61dcf948c6b7a25575a/internal/service/sqs/queue.go#L82](https://github.com/hashicorp/terraform-provider-aws/blob/3f454457baf318faae5cc61dcf948c6b7a25575a/internal/service/sqs/queue.go#L82)
    
3.  Максимальная длина имени указана в условии  [https://github.com/hashicorp/terraform-provider-aws/blob/3f454457baf318faae5cc61dcf948c6b7a25575a/internal/service/sqs/queue.go#L424](https://github.com/hashicorp/terraform-provider-aws/blob/3f454457baf318faae5cc61dcf948c6b7a25575a/internal/service/sqs/queue.go#L424)  и равна 80 символам
    
4.  Регулярное выражение для имени  `^[a-zA-Z0-9_-]{1,75}\.fifo$`  или  `^[a-zA-Z0-9_-]{1,80}$`
