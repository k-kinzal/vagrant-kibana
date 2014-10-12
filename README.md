# vagrant-kibana

===

Fluentd + Elasticsearch + Kibana3 and 4 beta on Vagrant.

## Get started

```
vagrant plugin install vagrant-berkshelf
vagrant up
```

### Kibana3

    http://localhost:9200/_plugin/kibana3

### Kibana4

    http://localhost:5601

## Use Bigquery

create bigquery token file ```config/.bigqueryrc``` and ```config/.bigquery.v2.token```.
https://pypi.python.org/pypi/bigquery/2.0.17


## Use Cloudwatch log

your write to ```config/.env```.

```
export AWS_REGION="[region]"
export AWS_ACCESS_KEY_ID="[access key]"
export AWS_SECRET_ACCESS_KEY="[secret key]"
```
