# CloudInit with Hetzner

*based on [this tutorial](https://community.hetzner.com/tutorials/basic-cloud-config)*

### Create A Server

- ![create server](/screens/server.png) A simple instance is enough for this tutorial (2gb RAM, ARM)
- ![debian as image](/screens/image-1.png)
choose a debian based distro as image
-![Alt text](/screens/config.png) paste your cloud config and finish the setup
 

### Create SSH Key Pair

- create ssh key pair 
    `ssh-keygen -b 4096`
- copy pub key to file config see [Config File](/hetzner_cloudinit_example.yml)  
    **MacOS:** `cat ~/.ssh/my_pub_key | pbcopy`   
    **Linux:** `cat ~/.ssh/my_pub_key | xclip -selection clipboard`

### Config from URL

- public file

```yaml
#include
https://URLtoCode/config.yaml
```

- *example usage for this repo*

```yaml
#include
https://raw.githubusercontent.com/EricKrg/cloudinit-breakout-un-deRSE23/main/hetzner_cloudinit_example.yml
```

- private file with JWT, will also work with basic HTTP Auth

```sh
curl \
	-X POST \
	-H "Authorization: Bearer $API_TOKEN" \
	-H "Content-Type: application/json" \
	-d '{"image":"ubuntu-22.04","location":"nbg1","name":"my-server","server_type":"cx11","user_data":"#include\nhttps://URLtoCode/config.yaml"}' \
  	'https://api.hetzner.cloud/v1/servers'
```



### Test the setup

- test the setup by trying to connect to the created instance via ssh `ssh <ssh-user>@<server-ip>`   

**example usage:** `ssh ci@65.108.253.152 ` 