# CloudInit with Hetzner

*based on [this tutorial](https://community.hetzner.com/tutorials/basic-cloud-config)*
- create ssh key pair 
- copy pub key to file config


### Config from URL

- public file

```yaml
#include
https://URLtoCode/config.yaml
```

- example usage for this repo

```yaml
#include
https://raw.githubusercontent.com/EricKrg/cloudinit-breakout-un-deRSE23/main/hetzner_cloudinit_example.yml
```

- private file

```sh
curl \
	-X POST \
	-H "Authorization: Bearer $API_TOKEN" \
	-H "Content-Type: application/json" \
	-d '{"image":"ubuntu-22.04","location":"nbg1","name":"my-server","server_type":"cx11","user_data":"#include\nhttps://URLtoCode/config.yaml"}' \
  	'https://api.hetzner.cloud/v1/servers'
```

