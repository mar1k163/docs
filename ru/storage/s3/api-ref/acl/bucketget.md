# Метод bucketGetAcl

Возвращает список управления доступом для бакета.


## Запрос {#request}

```
GET /{bucket}?acl HTTP/1.1
```

### Path параметры {#path-parameters}

Параметр | Описание
----- | -----
`bucket` | Имя бакета.

### Query параметры {#request-params}

Параметр | Описание
----- | -----
`acl` | Обязательный параметр для обозначения типа операции.


### Заголовки {#request-headers}

Используйте в запросе необходимые [общие заголовки](../common-request-headers.md).

## Ответ {#response}

### Заголовки {#response-headers}

Ответ может содержать только [общие заголовки](../common-response-headers.md).

### Схема данных {#response-scheme}

Возвращаемые данные имеют структуру, описанную в разделе [XML-cхема ACL](xmlscheme.md).

### Коды ответов {#response-codes}

Перечень возможных ответов смотрите в разделе [{#T}](../response-codes.md).
