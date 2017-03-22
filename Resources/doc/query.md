# Query callbacks

You can apply additional `where` conditions to your query.

1. [WhereAll callback](#1-whereall-callback)
2. [Get query](#2-get-query)
3. [Response callback](#3-response-callback)
4. [Query callback](#4-query-callback)

## 1. WhereAll callback

This is applied to all queries that are made and reduces the number of records that the user can access. 

```php
public function indexResultsAction()
{
    $datatable = $this->get('app.datatable.comment');
    $datatable->buildDatatable();

    $query = $this->get('sg_datatables.query')->getQueryFrom($datatable);

    $function = function($qb)
    {
        // $qb->andWhere("post.title = :p");
        $qb->andWhere("post_categories.title = :p");
        $qb->setParameter('p', 'Test');
    };

    $query->addWhereAll($function);

    return $query->getResponse();
}
```

## 2. Get query

```php
public function indexResultsAction()
{
    $datatable = $this->get('app.datatable.comment');
    $datatable->buildDatatable();

    $query = $this->get('sg_datatables.query')->getQueryFrom($datatable);
    $query->buildQuery();

    $qb = $query->getQuery();
    $qb->andWhere("post.visible = true");

    $query->setQuery($qb);

    return $query->getResponse(false); // important: $query->getResponse(false) instead $query->getResponse()
}
```

## 3. Response callback

This can be used to change the response data of a datatable query after the datatable specific information are added and before it is rendered.
The first parameter passed to the callbacks are the response data and the second parameter is the DatatableQuery object.

```php
public function indexResultsAction()
{
    $datatable = $this->get('app.datatable.comment');
    $datatable->buildDatatable();

    $query = $this->get('sg_datatables.query')->getQueryFrom($datatable);

    $function = function($data, DatatableQuery $query)
    {
        $data['custom_value'] = 123;
        return $data;
    };

    $query->addResponseCallback($function);

    return $query->getResponse();
}
```

After the example above the JSON output will look like this:

```json
{
  "draw": 1,
  "data": [
    ...
  ],
  "recordsTotal": 100,
  "recordsFiltered": 30,
  "custom_value": 123
}
```

## 4. Query callback

This can be used to configure generated query, which will be used to retrieve data for datatable. Using this callback it
is possible to change almost anything on query, so it is strongly advised not to try to change DQL, parameters or any
property related to those.

Below example shows how it might be used to enable query and result cache on query:

```php
public function indexResultsAction()
{
    $datatable = $this->get('app.datatable.comment');
    $datatable->buildDatatable();
    $query = $this->get('sg_datatables.query')->getQueryFrom($datatable);
    $function = function (DatatableQuery $query)
    {
        return $query->useQueryCache(true)
            ->useResultCache(true, 60, 'cache_datatable_comment_' . md5(serialize($query->getParameters());
    }
}
```

Above code will enable query cache and configure result cache as valid for 60s and stored under ID based on query parameters. 