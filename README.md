# Задание

## Часть 1: Создание робота для сбора публикаций с сайта (crawler)

- Выберите произвольный сайт или используйте RSS ленты.
- Скачивайте HTML/XML страницы по ссылкам из очереди задач.
- Обрабатывайте коды ошибок при скачивании.
- Парсьте текст публикации (новости) с заголовком, временем публикации, автором и ссылкой.
- Обеспечьте обработку в несколько потоков.
- Сохраняйте результаты в очередь результатов.

## Часть 2: Планировщик - использование очередей

- Парсьте стартовую страницу и получайте ссылки и заголовки новостей (включая вычисление hash).
- Используйте брокер сообщений RabbitMQ.
- Создайте очередь с задачами (ссылками со стартовой страницы) в программе.
- Создайте очередь с результатами (текст публикации, заголовок, дата и т.д.).
- Обеспечьте защиту от потери необработанных сообщений (используйте флаг Ack).
- Умейте использовать Basic.Consume и Basic.Get (понимайте разницу).
- Выполните проверку на дубли по hash (вычисленному из url и title новости).

## Часть 3: Сохранение данных в базу

1. Используйте базу данных Elasticsearch.
2. Создайте индекс для хранения документов с соответствующими полями (заголовок, время публикации, текст, ссылка, автор).
3. В планировщике (при парсинге стартовой страницы) сохраняйте документы (только их url и title) с идентификаторами, вычисленными как хэш от заголовка и даты документа (или хэш от ссылки на страницу с новостью).
4. Проверяйте наличие в базе документа по идентификатору (хешу) - не пускайте в обработку, если уже существует.
5. Считывайте документы из очереди с результатами и обновляйте их (записывайте полный текст публикации) в базе по идентификатору (хэшу от заголовка и даты документа).
6. Выполняйте поиск по нескольким полям документа в разных комбинациях (с различными логическими операторами) + используйте скрипты и MultiGet.
7. Умейте составлять сложные полнотекстовые поисковые запросы (понимайте синтаксис полнотекстового поиска Elasticsearch).
8. Выполняйте различные типы агрегаций (например, вывод гистограммы по количеству публикаций на различные даты и для различных авторов) + агрегации по логам, сохраненным через logstash.

## Запросы
### Запрос с использованием скрипта
```
GET /article/_search
{
  "query": {
    "script_score": {
      "query": {"match_all": {}},
      "script": {
        "source": "doc['author'].value.length()"
      }
    }
  }
}
```
![](/image/qSCRIPT.png "Запрос с использованием скрипта")

### Запрос с использованием операторов логического поиска
```
GET /article/_search
{
  "query": {
    "query_string": {
      "query": "(Wildberries AND российского) NOT \"до свиданья\""
    }
  }
}
```
![](/image/qANDNOT.png "Запрос с использованием операторов логического поиска")

### MultiGet запрос
```
GET /article/_mget
{
  "docs": [
    { "_id": "tXYiu48Bpz-yHQp52ORc" },
    { "_id": "x3Zyu48Bpz-yHQp59-R_" }
  ]
}
```
![](/image/qMGET.png "MultiGet запрос")

### multi_match запрос
```
GET /article/_search
{
  "query": {
    "multi_match": {
      "query": "результаты",
      "fields": [ "title", "text", "author" ]
    }
  }
}
```
![](/image/qFIELDS.png "multi_match запрос")

### Сложный поисковый запрос с использованием оператора "must" и оператора "match"
```
GET /article/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "Positive" }},
        { "match": { "text": "угроз" }}
      ]
    }
  }
}
```
![](/image/qCOMPLEX.png "Сложный поисковый запрос с использованием оператора must и оператора match")