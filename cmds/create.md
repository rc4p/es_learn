
Mysql data to ES:
1. （字段名会自动将Mysql转为小写）
2. cmd:
```bash
    PUT imfs
    {
      "settings": {
        "number_of_replicas": 1, 
        "number_of_shards": 5,
        "analysis": {
          "analyzer": {
            "ik":{
              "type":"custom",
              "tokenizer":"standard"
            }
          }
        }
      },
      "mappings": {
        "tbSendRcd" :{
          "properties":{
            "keyid":{
              "type":"long",
              "index" : "false" 
            },
            "idxuserid":{
              "type":"integer"
            },
            "idxsupplierid":{
              "type":"integer" 
            },
            "srctype":{
              "type":"integer" 
            },
            "sendtype":{
              "type":"integer" 
            },
            "idxrouteid":{
              "type":"integer" 
            },
            "idxchannelid":{
              "type":"integer" 
            },
            "idxgoipid":{
              "type":"integer"
            },
            "iccid":{
              "type":"text",
              "index" : "false" 
            },
            "inmsgid":{
              "type":"text",
              "index" : "false" 
            },
            "outmsgid":{
              "type":"text",
              "index" : "false" 
            },
            "orireceiver":{
              "type":"text" 
            },
            "sender":{
              "type":"text"
            },
            "receiver":{
              "type":"text" 
            },
            "dstreceiver":{
              "type":"text" 
            },
            "sendtm":{
              "type":"date",
              "format": "strict_date_optional_time||epoch_millis"
            },
            "donetm":{
              "type":"date",
              "format": "strict_date_optional_time||epoch_millis"
            },
            "duration":{
              "type":"integer" 
            },
            "mcc":{
              "type":"integer" 
            },
            "mnc":{
              "type":"integer" 
            },
            "pay":{
              "type":"long" 
            },
            "cost":{
              "type":"long"
            },
            "chargecnt":{
              "type":"integer"
            },
            "sms":{
              "type":"text",
              "analyzer":"ik",
              "search_analyzer":"ik_smart"
            },
            "result":{
              "type":"integer",
              "index" : "false" 
            },
            "reason":{
              "type":"text",
              "index" : "false" 
            },
            "mdftm":{
              "type":"date",
              "format": "strict_date_optional_time||epoch_millis"
            },
            "useralias":{
              "type":"text"
            },
            "supplieralias":{
              "type":"text" 
            },
            "channelalias":{
              "type":"text" 
            },
            "goipalias":{
              "type":"text" 
            }
          }
        }
      }
    }
```

3.mysql2es.conf
```bash
input {
    stdin{
    }
    jdbc {
      # 数据库
      jdbc_connection_string => "jdbc:mysql://192.168.111.92:3306/IMFS?zeroDateTimeBehavior=convertToNull"
      # 用户名密码
      jdbc_user => "root"
      jdbc_password => ""
      # jar包的位置
      jdbc_driver_library => "/Users/c.p/app/mysql-connector-java-5.1.22-bin.jar"
      # mysql的Driver
      jdbc_driver_class => "com.mysql.jdbc.Driver"
      jdbc_paging_enabled => "true"
      jdbc_page_size => "50000"
      statement => "select keyId,idxUserId,idxSupplierId,srcType,sendType,idxRouteId,idxChannelId,idxGoipId,iccid,
                    inMsgId,outMsgId,oriReceiver,sender,receiver,dstReceiver,sendTm,doneTm,duration,mcc,mnc,pay,
                    cost,chargeCnt,sms,result,reason,mdfTm,userAlias,supplierAlias,ChannelAlias,GoipAlias
                    from tbSendRcd where keyId > :sql_last_value"
      schedule => "* * * * *"
      #索引的类型
      type => "tbSendRcd"
	record_last_run => true
	#使用递增列
	use_column_value => true
    	tracking_column => "keyid"
	#递增字段的类型， timestamp表示时间戳
	tracking_column_type => "numeric"
    	last_run_metadata_path => "/Users/c.p/.logstash_jdb_last_run"
	clean_run => "false"
	}
}

output {
    elasticsearch {
        hosts => "127.0.0.1:9200"
        # index名
        index => "imfs"
        # 需要关联的数据库中有有一个id字段，对应索引的id号
	document_id => "%{keyid}"
	document_type => "tbSendRcd"
    }
    stdout {
        codec => json_lines
    }
}
```

