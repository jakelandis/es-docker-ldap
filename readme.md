### Usage

```bash
docker-compose up
curl -u lisa:simpson http://localhost:9200/_security/_authenticate | jq
```

### Testing failover

```
curl -u elastic:changeme -X POST http://localhost:9200/_security/realm/ldap1/_clear_cache | jq
docker stop es-docker-ldap_ldap1_1
curl -u elastic:changeme -X POST http://localhost:9200/_security/realm/ldap1/_clear_cache | jq
curl -u lisa:simpson http://localhost:9200/_security/_authenticate | jq 
docker stop es-docker-ldap_ldap2_1
curl -u elastic:changeme -X POST http://localhost:9200/_security/realm/ldap1/_clear_cache | jq
curl -u lisa:simpson http://localhost:9200/_security/_authenticate | jq # expected to fail since no running LDAP servers
docker start es-docker-ldap_ldap1_1
curl -u lisa:simpson http://localhost:9200/_security/_authenticate | jq
docker start es-docker-ldap_ldap2_1
curl -u elastic:changeme -X POST http://localhost:9200/_security/realm/ldap1/_clear_cache | jq
curl -u lisa:simpson http://localhost:9200/_security/_authenticate | jq 
```

Repeat the above but use docker pause and docker unpause to simulate an HTTP disconnect. 

```
curl -u elastic:changeme -X POST http://localhost:9200/_security/realm/ldap1/_clear_cache | jq
docker pause es-docker-ldap_ldap1_1
curl -u elastic:changeme -X POST http://localhost:9200/_security/realm/ldap1/_clear_cache | jq
curl -u lisa:simpson http://localhost:9200/_security/_authenticate | jq # This errors due to https://github.com/elastic/elasticsearch/issues/93353
```
