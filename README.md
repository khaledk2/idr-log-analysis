Installation instructions
=========================

- The code should be pulled from GitHub using the following command:
  
  ```
  https://github.com/IDR/idr-log-analysis.git
  ```
- The following commands should be run to create working folders for both Elasticsearch and Fluentd
  
  ```
  cd idr-log-analysis
  mkdir -p volumes/{es,fluentd}
  chown 1000 volumes/es
  chown 100 volumes/fluentd
  ```
- ```Nginx``` is configured to use HTTP Basic Authentication [HTTP Basic Authentication](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/%E2%80%B8), so it is needed to create a username and password, then save both in a file named ```httppasswd```, e.g.
  
  ```
  echo `username:password` > nginx/httppasswd
  ```
- To ensure that Docker is installed on your machine, you can run the following command:

  ```
  sudo docker --version
  ```

-If Docker is not installed, it needs to be installed.

- The following command is used to run the applications

```
docker compose up -d
```

_ To follow the log output, the user can run this command:

  ```
  docker compose logs -f
  ```

Kibana setup
============

1. Connect to http://localhost:12381 and login with Nginx basic auth.
2. In the Kibana Management go to [Saved Objects](http://localhost:12381/app/management/kibana/objects) and import the following file:

  ```
  kibana-export.ndjson
  ```

This should import saved index patterns, visualisations and dashboards.
If the visualisations are disconnected from the indices associate them as follows:

- IDR visualisations should be associated with index pattern `fluentd.nginx.access.*`.
- IDR-analysis visualisations should be associated with index pattern `fluentd.haproxy.http.*`.

3. If you don't have a default index pattern Under `Index Patterns` make `fluentd.nginx.access.*` the default.


If you need to create the index patterns yourself:

1. Create index pattern with pattern `fluentd.haproxy.http.*`.
2. Select `@timestamp` as  the `Time Filter field name`.
3. Check that the field `host` has type `ip` and `geoip` has type `geo_point`.
   If these types are incorrect it means the Elasticsearch index mapping wasn't created.
4. Repeat for index pattern `fluentd.nginx.access.*`.

Updating logs
=============

1. Create a directory `/uod/idr/versions/nginx-logs-combined/prodNN` where `prodNN` is the release that is being archived
2. Untar  the `nginx` archive copied to `/uod/idr/versions/prodNN` under this directory
3. Delete  all `error.log*` as well as all `access.log*` files that are not in the required timerange, i.e. those from pre-release or post-release work.
4. For convenience if you have any files that do not follow the `access.log-YYYYMMDD.gz` rename and gzip them.
5. Aggregate all these logs into a single uncompressed file, `zcat access.log*.gz > access.log-prodNN`.
6. Move `access.log-prodNN` to `/uod/idr/versions/nginx-logs-combined/prod-merged-agg/`
7. Fluentd should automatically start to process this file

Notes
=====

Although the log ingest process can handle individual `access.log*` files it will continually tail each file, so aggregating them massively reduces the number of open file handles.

