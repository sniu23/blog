# 分区

- / 主分区 ext4 30G
- /boot 逻辑分区 ext4 200M
- /SWP 2G
- /home 逻辑分区 ext4 剩余空间

# 安装htop

```
sudo apt install htop
```

- PID — 进程id
- USER — 进程所有者
- PR — 进程优先级
- NI — nice值。负值表示高优先级，正值表示低优先级
- VIRT — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
- RES — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
- SHR — 共享内存大小，单位kb
- S — 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
- %CPU — 上次更新到现在的CPU时间占用百分比
- %MEM — 进程使用的物理内存百分比
- TIME+ — 进程使用的CPU时间总计，单位1/100秒
- COMMAND — 进程名称（命令名/命令行）

# 安装git

```
sudo apt install git gitk
```

# 使用nvm管理node版本

```
sudo apt-get install git
git clone https://github.com/creationix/nvm.git ~/.nvm && cd ~/.nvm && - git checkout `git describe --abbrev=0 --tags`
cd
vim .bashrc
将 source ~/.nvm/nvm.sh 添加进我们的.bashrc中，保存退出
执行source .bashrc

nvm -v #查看nvm版本
nvm ls-remote #查看远端可用node版本
nvm install x.x.x #安装node到本地
nvm alias default x.x.x #调整node版本
nvm use x #临时调整node版本
```

# npm设置代理

```
$ npm config set proxy http://server:port
$ npm config set https-proxy http://server:port

# 如果代理需要认证的话可以这样来设置。
$ npm config set proxy http://username:password@server:port
$ npm config set https-proxy http://username:pawword@server:port

# 如果代理不支持https的话需要修改npm存放package的网站地址。
$ npm config set registry "http://registry.npmjs.org/"
```

# node-oracledb驱动安装测试

参考：http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html#ic_x64_inst

参考：https://github.com/oracle/node-oracledb/blob/master/INSTALL.md#instzip

1. ORACLE网站下载Instant Client for Linux x86-64： Basic、SDK、sqlplus 的ZIPs

```
cd /opt/oracle
unzip instantclient-basic-linux.x64-12.2.0.1.0.zip
unzip instantclient-sdk-linux.x64-12.2.0.1.0.zip
mv instantclient_12_2 instantclient
cd instantclient
ln -s libclntsh.so.12.1 libclntsh.so
ln -s libocci.so.12.1 libocci.so
```

2. ~/.bashrc添加配置环境变量

```
export ORACLE_BASE=/opt/oracle
export ORACLE_HOME=$ORACLE_BASE/instantclient
export OCI_LIB_DIR=/opt/oracle/instantclient
export OCI_INC_DIR=/opt/oracle/instantclient/sdk/include
export PATH=$ORACLE_HOME:$PATH
export LD_LIBRARY_PATH=$ORACLE_HOME:${LD_LIBRARY_PATH}
export TNS_ADMIN=$ORACLE_HOME/network/admin/
export NLS_LANG="AMERICAN_AMERICA.UTF8"
```

3. 重进终端，测试sqlplus

```
$ sqlplus username/password@//serverIP:1521/serviceName
>sql select sysdate from dual;
```

4. 安装 node-oracledb并测试

```
cd test && npm install oracledb
```

```js
// select1.js
// node select1.js
// https://github.com/oracle/node-oracledb/blob/master/examples/select1.js
var oracledb = require('oracledb');

// Get a non-pooled connection
oracledb.getConnection(
  {
    user          : 'user',
    password      : 'password',
    connectString : 'serverIP:1521/serviceName'
  },
  function(err, connection)
  {
    if (err) {
      console.error(err.message);
      return;
    }
    connection.execute(
      "SELECT sysdate " +
        "FROM dual " +
        "WHERE 1=1 ",
      [],
      function(err, result)
      {
        if (err) {
          console.error(err.message);
          doRelease(connection);
          return;
        }
        console.log(result.metaData); 
        console.log(result.rows);     
        doRelease(connection);
      });
  });

// Note: connections should always be released when not needed
function doRelease(connection)
{
  connection.close(
    function(err) {
      if (err) {
        console.error(err.message);
      }
    });
}
```

# 安装 vscode编辑器 （ubuntu server就不要了）

``` 
sudo add-apt-repository ppa:ubuntu-desktop/ubuntu-make
sudo apt-get update
sudo apt-get install ubuntu-make
umake web visual-studio-code
```

# 安装vim及插件 （看自己需要）

```
# sudo apt install vim
# 使用vundle管理插件，以下为~/.vimrc配置

set rtp+=~/.vim/bundle/vundle/
call vundle#rc()
Bundle 'gmarik/vundle'

call vundle#begin()
Plugin 'vim-scripts/L9'
Plugin 'mattn/emmet-vim' "zen coding
Plugin 'kien/ctrlp.vim' "ctrl+p search file
Plugin 'MarcWeber/vim-addon-mw-utils'
Plugin 'garbas/vim-snipmate'
"Plugin 'Valloric/YouCompleteMe'
call vundle#end() 

Bundle 'scrooloose/nerdtree'
let NERDTreeWinPos='right'
let NERDTreeWinSize=30
map <F2> :NERDTreeToggle<CR>

Bundle 'fholgado/minibufexpl.vim'
let g:miniBufExplMapWindowNavVim = 1   
let g:miniBufExplMapWindowNavArrows = 1   
let g:miniBufExplMapCTabSwitchBufs = 1   
let g:miniBufExplModSelTarget = 1  
let g:miniBufExplMoreThanOne=0
map <F11> :MBEbp<CR>
map <F12> :MBEbn<CR>

filetype plugin indent on
```

# 使用ssh协议（透过http代理）操作github

参考：https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#adding-your-ssh-key-to-the-ssh-agent

1. 检查ssh-agent和ssh-server进程。

```
ps -e |grep ssh
//没有ps就要安装：
sudo apt install openssl-server
```
2. 产生公钥和私钥。

```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
//非对称加密，默认不加密，默认存放在~/.ssh下：id_rsa和id_rsa.pub。
```

3. 登录github, 将.ssh/id_rsa.pub内容复制，添加SSH key。

4. OS在内网并使用http全局代理上网，SSH协议需要下载专用软件，编写config 

```
//安装 SSH over HTTP 专用软件
sudo apt install corkscrew 
//编写配置：~/.ssh/config
Host github.com
User git
Hostname ssh.github.com
Port 443
ProxyCommand corkscrew your_http_proxy_ip your_http_proxy_port %h %p
IdentityFile ~/.ssh/id_rsa
//参考：http://www.tuicool.com/articles/z632a2z
//参考：http://blog.sina.com.cn/s/blog_538d55be01018ksk.html
//参考：http://www.cnblogs.com/lihaiping/p/4728993.html
```

5. 添加 SSH key 到 ssh-agent 进程，并验证。 

``` 
eval "$(ssh-agent -s)" //查进程
ssh-add ~/.ssh/id_rsa //添加
ssh-add -l -E md5 //验证，看和github上传的fingerPoint一样吗？
```

6. 测试SSH方式：

```
ssh -vT git@github.com
//参考：https://help.github.com/articles/error-permission-denied-publickey/
```

7. 终于可以免密使用：

```
git remote add origin git@github.com:yourName/yourRepo.git
git push origin master
//参考：http://blog.csdn.net/a214919447/article/details/54602829
//参考：http://www.runoob.com/w3cnote/git-guide.html
``` 

# 编写gitignore

https://github.com/github/gitignore

```
# Windows:
Thumbs.db
ehthumbs.db
Desktop.ini

# cordova:
/hooks/*
/platforms/*
/plugins/*
/public/plugins/*

# editor:
/.idea/*
/.vscode/*

# node:
/node_modules/*

# yml:
/*.yml

logs & config:
/log/*
/config/*

```
