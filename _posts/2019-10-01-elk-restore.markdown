---
layout: post
title: elk restore 
date: 2019-10-01 23:32:24.000000000 +09:00
---

Load 5GB log into logstash, the config of logstash as below
```shell
input {
	tcp {
		port => 5000
	}

}

filter {
	json {
		source => "message"
	}
}

output {
	elasticsearch {
		hosts => "elasticsearch:9200"
		user => "elastic"
		password => "changeme"
	}
}

```
The time spent in loading logs as below:
```shell
$ curl -u elastic:changeme "http://localhost:9200/_cat/indices?v&health=yellow"
health status index                      uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   logstash-2019.10.01-000001 tdCPW4lpRKOiH9d0-oYPwA   1   1    1289692            0      4.4gb          4.4gb
```

The time for snapshot creation:
```shell
$ time curl -X PUT -u elastic:changeme "http://localhost:9200/_snapshot/my_backup/snapshot_1?wait_for_completion=true&pretty" -H 'Content-Type: application/json' -d'
> {
>   "indices": "logstash-2019.10.01-000001",
>   "ignore_unavailable": true,
>   "include_global_state": false,
>   "metadata": {
>     "taken_by": "kimchy",
>     "taken_because": "backup before upgrading"
>   }
> }
> '
{
  "snapshot" : {
    "snapshot" : "snapshot_1",
    "uuid" : "WR6cyuTPTnyrvuiSSbU5-A",
    "version_id" : 7030199,
    "version" : "7.3.1",
    "indices" : [
      "logstash-2019.10.01-000001"
    ],
    "include_global_state" : false,
    "metadata" : {
      "taken_by" : "kimchy",
      "taken_because" : "backup before upgrading"
    },
    "state" : "SUCCESS",
    "start_time" : "2019-10-01T11:59:25.668Z",
    "start_time_in_millis" : 1569931165668,
    "end_time" : "2019-10-01T12:01:39.776Z",
    "end_time_in_millis" : 1569931299776,
    "duration_in_millis" : 134108,
    "failures" : [ ],
    "shards" : {
      "total" : 1,
      "failed" : 0,
      "successful" : 1
    }
  }
}

real	2m14.226s
user	0m0.016s
sys	0m0.004s
```
The time for restoration:
```shell
$ curl -u elastic:changeme -X DELETE "localhost:9200/logstash-2019.10.01-000001?pretty"
{
  "acknowledged" : true
}

$ time curl -u elastic:changeme -X POST "localhost:9200/_snapshot/my_backup/snapshot_1/_restore?pretty" -H 'Content-Type: application/json' -d'
> {
>   "indices": "logstash-2019.10.01-000001",
>   "ignore_unavailable": true,
>   "include_global_state": true,
>   "rename_pattern": "logstash-(.+)",
>   "rename_replacement": "restored_index_$1"
> }
> '
{
  "accepted" : true
}

real	0m0.091s
user	0m0.004s
sys	0m0.000s


$ curl -u elastic:changeme "http://localhost:9200/_cat/indices?v&health=yellow"
health status index                            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   restored_index_2019.10.01-000001 nydSXiUbR5S2T5AxW1uWUA   1   1    1289692            0      4.4gb          4.4gb

```

The status of restore:
```shell
$ curl -u elastic:changeme -X GET "localhost:9200/_snapshot/my_backup/snapshot_1/_status?pretty"
{
  "snapshots" : [
    {
      "snapshot" : "snapshot_1",
      "repository" : "my_backup",
      "uuid" : "WR6cyuTPTnyrvuiSSbU5-A",
      "state" : "SUCCESS",
      "include_global_state" : false,
      "shards_stats" : {
        "initializing" : 0,
        "started" : 0,
        "finalizing" : 0,
        "done" : 1,
        "failed" : 0,
        "total" : 1
      },
      "stats" : {
        "incremental" : {
          "file_count" : 118,
          "size_in_bytes" : 4821513312
        },
        "total" : {
          "file_count" : 118,
          "size_in_bytes" : 4821513312
        },
        "start_time_in_millis" : 1569931165724,
        "time_in_millis" : 133993
      },
      "indices" : {
        "logstash-2019.10.01-000001" : {
          "shards_stats" : {
            "initializing" : 0,
            "started" : 0,
            "finalizing" : 0,
            "done" : 1,
            "failed" : 0,
            "total" : 1
          },
          "stats" : {
            "incremental" : {
              "file_count" : 118,
              "size_in_bytes" : 4821513312
            },
            "total" : {
              "file_count" : 118,
              "size_in_bytes" : 4821513312
            },
            "start_time_in_millis" : 1569931165724,
            "time_in_millis" : 133993
          },
          "shards" : {
            "0" : {
              "stage" : "DONE",
              "stats" : {
                "incremental" : {
                  "file_count" : 118,
                  "size_in_bytes" : 4821513312
                },
                "total" : {
                  "file_count" : 118,
                  "size_in_bytes" : 4821513312
                },
                "start_time_in_millis" : 1569931165724,
                "time_in_millis" : 133993
              }
            }
          }
        }
      }
    }
  ]
}
```