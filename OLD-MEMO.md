# Learn CoreOS and Container things

## 1. create etcd cluster

#### 1-1. digital ocean

- get DO api token
- register ssh key and get ssh key id
- create config yaml for etcd clustering
- traspile this config yaml into `Ignition Config` by using `ct`

```bash
ct --platform=digitalocean < test_config_01.yml > config01.ign

```

- config01.ign
```json
{
  "ignition": {
    "config": {},
    "security": {
      "tls": {}
    },
    "timeouts": {},
    "version": "2.2.0"
  },
  "networkd": {},
  "passwd": {},
  "storage": {},
  "systemd": {
    "units": [
      {
        "dropins": [
          {
            "contents": "[Unit]\nRequires=coreos-metadata.service\nAfter=coreos-metadata.service\n\n[Service]\nEnvironmentFile=/run/metadata/coreos\nExecStart=\nExecStart=/usr/lib/coreos/etcd-wrapper $ETCD_OPTS \\\n  --listen-peer-urls=\"http://${COREOS_DIGITALOCEAN_IPV4_PRIVATE_0}:2380\" \\\n  --listen-client-urls=\"http://0.0.0.0:2379\" \\\n  --initial-advertise-peer-urls=\"http://${COREOS_DIGITALOCEAN_IPV4_PRIVATE_0}:2380\" \\\n  --advertise-client-urls=\"http://${COREOS_DIGITALOCEAN_IPV4_PRIVATE_0}:2379\" \\\n  --discovery=\"https://discovery.etcd.io/092f387591e79bd9557eea0298972fd1\"",
            "name": "20-clct-etcd-member.conf"
          }
        ],
        "enable": true,
        "name": "etcd-member.service"
      }
    ]
  }
}
```

- create coreos vm via digital ocean API
```bash
curl --request POST "https://api.digitalocean.com/v2/droplets" \
     --header "Content-Type: application/json" \
     --header "Authorization: Bearer $DO_TOKEN" \
     --data '{
      "region":"sgp1",
      "image":"coreos-stable",
      "size":"512mb",
      "name":"core-1",
      "private_networking":true,
      "ssh_keys":['$DO_SSHKEY_ID_1'],
      "user_data": "'"$(cat config01.ign | sed 's/"/\\"/g')"'"
}'
```
