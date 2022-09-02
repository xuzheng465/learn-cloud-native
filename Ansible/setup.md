刚装完vagrant和virtualbox，

```sh
ssh-keygen

ssh-copy-id -i .ssh/id_rsa.pub ansible-node1
ssh-copy-id -i .ssh/id_rsa.pub ansible-node2

password: vagrant

test

ansible -i inventory.ini -m ping web
```

