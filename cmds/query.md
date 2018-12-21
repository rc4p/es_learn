1.查询
```bash
    GET imfs/_search
    {
      "query": {
        "match": {
          "sms": "diajukan"
        }
      }
    }
```
2.查看分词效果
```bash
    GET imfs/_analyze
    {
      "analyzer": "ik_max_word",
      "text":"[Super-Mudah] Kredit anda telah diajukan, mohon untuk menunggu hasil analisa. Terima kasih atas partisipasinya."
    }
```

