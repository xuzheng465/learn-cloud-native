### 环境

虚拟软件: Parallel

OS: Ubuntu 22.04

### 必要软件

#### Editor

在Ubuntu 22.04下安装neovim 0.7

```sh
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:neovim-ppa/stable
sudo apt-get update
sudo apt-get install neovim
```



#### Docker

https://docs.docker.com/engine/install/ubuntu/



#### Zsh

```sh
sudo apt update
sudo apt -y install zsh
```



#### Golang

https://go.dev/doc/install

#### Python

##### pyenv

https://github.com/pyenv/pyenv#getting-pyenv

```sh
 git clone https://github.com/pyenv/pyenv.git ~/.pyenv
 
 # for zsh
 echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

##### pyenv virtualenv

```bash
$ git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv

$ echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc

$ pyenv install 3.8.13
$ pyenv virtualenv 3.8.13 env-3.8.13
$ pyenv activate env-3.8.13

# 退出虚拟环境
$ pyenv deactivate

# 移除虚拟环境
$ pyenv uninstall env-3.8.13
```







```sh

```





```sh

```









```sh

```







```sh

```

