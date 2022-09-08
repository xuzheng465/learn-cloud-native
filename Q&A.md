### vagrant 配置代理

```sh
$ vagrant plugin install vagrant-proxyconf

# check plugins
$ vagrant plugin list
```

在Vagrantfile加上

```ruby
if Vagrant.has_plugin?("vagrant-proxyconf")
  config.proxy.http     = "http://192.168.0.1:3128/"
  config.proxy.https    = "http://192.168.0.1:3128/"
  config.proxy.no_proxy = "localhost,127.0.0.1,.example.org"
 end
```



修改完再执行`vagrant up`

