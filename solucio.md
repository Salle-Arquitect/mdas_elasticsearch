# 1) ¿Cuántos restaurantes hay en el index de `nyc_restaurants_inspection` y el de `amazon_reviews`?
```elasticsearch
GET /nyc_restaurants_inspection/_count
GET /amazon_reviews/_count
```
- `nyc_restaurants_inspection` = 383726
- `amazon_reviews` = 1248


# 2) Busca los restaurantes donde sirvan Seafood con match.
```elasticsearch
GET /_search
{
  "query": {
    "match": {
      "CUISINE DESCRIPTION": "Seafood"
    }
  }
}
```

# 3) Busca los restaurantes donde sirvan Seafood con term y justifica porque no devuelve resultados.
No encuentra nada:
```elasticsearch
GET /_search
{
  "query": {
    "term": {
      "CUISINE DESCRIPTION": "Seafood"
    }
  }
}
```

Da el mismo resultado que en `match`:
```elasticsearch
GET /_search
{
  "query": {
    "term": {
      "CUISINE DESCRIPTION": "seafood"
    }
  }
}
```

El motivo es debido a que ElasticSearch tiene un ETL que tokeniza el input,
poniéndolo todo en minúscula por ejemplo.
Entonces el `term` que busca literalmente el input,
con lo que no encuentra porque los datos han sido modificados.

# 4) Busca los restaurantes donde sirvan Seafood y no sean de MANHATTAN.
Sabemos que el total de las anteriores son 2954 restaurantes con `Seafood`.
Si queremos saber también cuantos tienen los dos:
```elasticsearch
GET /_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"CUISINE DESCRIPTION":"Seafood"}},
        {"match": {"BORO": "MANHATTAN"}}
      ]
    }
  }
}
```
Tenemos que Seafood y MANHATTAN son un total de 1306,
entonces Seafood y que no sea de MANHATTAN tiene que ser un total de 1648.

## Solución
Tenemos el resultado deseado de 1648:
```elasticsearch
GET /_search
{
  "query": {
    "bool": {
      "must": [{"match": {"CUISINE DESCRIPTION":"Seafood"}}],
      "must_not": [{"match": {"BORO": "MANHATTAN"}}]
    }
  }
}
```

# 5) Lista los barrios (BORO) con la suma de las inspecciones (VIOLATION DESCRIPTION) donde se han econtrado ratas(mice,rats)
```elasticsearch
GET /_search
{
  "size": 0,
  "aggs": {
    "boros":{
      "terms": {"field": "BORO"}
    }
  },
  "query": {
    "match": {
      "VIOLATION DESCRIPTION": "mice rats"
    }
  }
}
```

# 6) Mostrar cuantas inspecciones hay por año (INSPECTION DATE), solo mostrar los años que tengan como mínimo una inspección por año.
```elasticsearch
GET /_search
{
  "size": 1,
  "aggs": {
    "anyos": {
      "date_histogram": {
        "field": "INSPECTION DATE",
        "interval": "year",
        "min_doc_count": 1
      }
    }
  }
}
```

# 7) Contar cuantos restaurantes hay por los diferentes tipos de flags (CRITICAL FLAG)
```elasticsearch
GET /_search
{
  "size": 0,
  "aggs": {
    "flags":{
      "terms": {"field": "CRITICAL FLAG"}
    }
  }
}
```

# 8) Elimina los restaurantes del BORO MANHATTAN
```elasticsearch
GET /_search
{
  "query": {
    "match": {
      "BORO": "MANHATTAN"
    }
  }
}
```
Hay 10 de manhattan


```elasticsearch
POST /nyc_restaurants_inspection/_delete_by_query
{
  "query": {
    "match": {
      "BORO": "MANHATTAN"
    }
  }
}
```

```elasticsearch
GET /_search
{
  "query": {
    "match": {
      "BORO": "MANHATTAN"
    }
  }
}
```
Ahora dice que no hay nada.

# 9) Dado el index `amazon_reviews` realiza una query que cumpla que exista un producto con una review que tenga rating=5 y doRecomend=FALSE. Fijarse que devuelva los valores esperados y fijarse en el mapping actual que tiene el índice, es posible que sea necesario reindexar.
