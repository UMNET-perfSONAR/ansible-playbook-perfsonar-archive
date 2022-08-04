# ansible-playbook-perfsonar-archive

This is a playbook to deploy [perfsonar/archive/5.0.0](https://github.com/perfsonar/archive/tree/5.0.0) to CentOS for use with [pSSID](https://github.com/UMNET-perfSONAR/pSSID).

`1.1.1.1` is used throughout this readme as the server that `perfsonar/archive/5.0.0` will be put on.

## Deploying

```
ansible-playbook playbook.yml -i 1.1.1.1,
```

## Verifying Success

Open `1.1.1.1/opensearchdash/app/discover` in a browser and login using the credentials listed in `/etc/perfsonar/opensearch/opensearch_login` on `1.1.1.1`. (On U-M networks, `1.1.1.1:80` might have to be opened by a network admin.)

To verify logstash is responsive, on `1.1.1.1`, run

```
curl localhost/logstash -H "Authorization: Basic asdf"
```

where `Basic asdf` is the value listed in `/etc/perfsonar/logstash/proxy_auth.json` on the target server after deployment. If everything is working you should get `ok` with status `200` as the response.

To store pScheduler test results into the archive server, run

```
pscheduler task --archive @./archive.json rtt --dest perfsonar.net
```

where `archive.json` contains

```json
{
	"archiver": "http",
	"data": {
		"schema": 2,
		"_url": "http://1.1.1.1/logstash",
		"op": "put",
		"_headers": {
			"Authorization": "Basic asdf",
			"Content-Type": "application/json"
		}
	}
}
```

where `Basic asdf` is the value found in `/etc/perfsonar/logstash/proxy_auth.json` on the archive server after deployment. If successful, you will see a new entry at `1.1.1.1/opensearchdash/app/discover`.

For this setup to be secure, TLS must be set up within the Apache reverse proxy that is on `1.1.1.1`.
