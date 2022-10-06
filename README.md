# elasticsearch-snapshot-restore

Below commands are for kubernetes

```
k edit sts elastic-search
```

# Add below env to old es
```
     - name: path.repo
       value: /usr/share/elasticsearch/data/backup
```

# Exec inside old es and run commands for backup

```
k exec -it elastic-search-0  -- bash
```

```
curl -sS http://localhost:9200/_cat/indices?v
```


```
curl -sS -XPUT 'http://localhost:9200/_snapshot/my_backup' -H 'Content-Type: application/json' -d '{
  "type": "fs",
  "settings": {
    "location": "/usr/share/elasticsearch/data/backup/",
    "compress": true
  }
}'
```

```
curl -sS -XPUT 'http://localhost:9200/_snapshot/my_backup/snapshot-number-one?wait_for_completion=true'
```

```
cd /usr/share/elasticsearch/data/
tar -cvzf backup.tar.gz backup
mv backup.tar.gz /tmp/
exit
```

# Download the backup and upload to new es pod

```
k cp elastic-search-0:/tmp/backup.tar.gz /tmp/backup.tar.gz 
```

```
k cp /tmp/backup.tar.gz elasticsearch-new-0:/tmp/backup.tar.gz -n auth
```

# Exec to new es pod and run the commands for restore

```
k exec -it elasticsearch-new-0  -- bash
```

```
cd /tmp/

tar -xvzf backup.tar.gz

mv backup /usr/share/elasticsearch/data/

curl -sS -XPUT 'http://localhost:9200/_snapshot/my_backup' -H 'Content-Type: application/json' -d '{
  "type": "fs",
  "settings": {
    "location": "/usr/share/elasticsearch/data/backup/",
    "compress": true
  }
}'
```

```
curl -sS -XPOST 'http://localhost:9200/_snapshot/my_backup/snapshot-number-one/_restore'
```

# Check the data indexes

```
curl -sS http://localhost:9200/_cat/indices?v
```

# Check the data

```
curl -X GET "localhost:9200/index1/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "query": {
        "match_all": {}
    }
}
'
```

# If health showing yellow run below command

```
curl -H 'Content-Type: application/json' -XPUT 'localhost:9200/_settings' -d '
{
    "index" : {
        "number_of_replicas" : 0
    }
}'

```

```
exit
```
