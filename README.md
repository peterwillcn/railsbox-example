# Railsbox example

## 说明

[Railsbox](https://github.com/ouyangzhiping/railsbox)是一个整合了多个cookbook与Ruby社区部署最佳实践的项目，它已测试实现：

* 默认基于ruby2.0+rben+nginx+unicorn
* 实现在linode vps、阿里云与vagrant上的一键部署
* 无缝重启的nginx作为前端，unicorn作为后端的Rails最佳部署方案
* 自动上传github 密钥
* 无密码deploy等登陆
* postgresql作为数据库

以下以vagrant为例，示范如何使用：

## 必备

* 下载并安装[Vagrant](http://www.vagrantup.com/)
* 已具备Ruby开发环境，如rvm或rbenv+shell+git等。

假设本机是mac，分发主机通过vagrant来分发，是33.33.33.10。

## git clone

在mac上的shell执行。

	git clone https://github.com/ouyangzhiping/railsbox-example.git
	cd railsbox-example


## 初始化设置

在mac上的shell执行。

### 安装gem与cookbook

	bundle install                  # 更新gemfile文件依赖

更新bersk的cook依赖：

	bundle exec berks install -p cookbooks/       # 更新bersk的cook依赖

### 设置vagrantfile

注意其中的：

* `config.vm.network :private_network, ip: "33.33.33.10"`

如果修改了ip，请修改对应`33.33.33.10.json`的文件名。

如果你的网速较慢，将我共享在[Dropbox的box文件](https://www.dropbox.com/s/jgpdtomcgpshd4s/precise-server-cloudimg-amd64-vagrant-disk1.box)下载到本地.

然后设置为本地路径，这样未来销毁与开新主机速度快：

	config.vm.box_url = "/users/ouyang/dev/vagrant/box/precise-server-cloudimg-amd64-vagrant-disk1.box"


### 设置host.json文件

打开 nodes下面的`host.json.example`文件，根据自己分发的服务器，修改为对应主机名或者ip地址名，例如：`psyapp.org.json`或者`33.33.33.10.json`。在本示例中，我们修改为：`33.33.33.10.json`。

	cp nodes/host.json.example nodes/33.33.33.10.json

根据提示，依次修改相应配置信息。

* appbox部分填写本机的~/.ssh/id_rsa.pub的值，用于自动免密码登陆主机。在shell中执行`cat ~/.ssh/id_rsa.pub`获得。
* railsbox填写数据库、app等值
* github_deploys填写github等值，用于将主机的ssh键值自动上传给github。

更多配置信息，参考：[ouyangzhiping/railsbox](https://github.com/ouyangzhiping/railsbox)


## 启动vagrant

在mac上的shell执行。

	vagrant up


## 执行cookbook

在mac上的shell执行。

初始化主机，给`33.33.33.10`这台主机安装chef。

	bundle exec knife solo prepare vagrant@33.33.33.10

提示输入服务器密码。咱们使用的是vagrant，默认密码是：`vagant`，输入即可。根据网速，需要花费二三分钟不等。你可以去喝杯茶，然后慢慢来。

安装成功，提示：

	ouyang at ouyangtekiMacBook-Air in ~/dev/chef/railsbox-example on master [d]
	± knife solo prepare vagrant@33.33.33.10
	Bootstrapping Chef...
	Enter the password for vagrant@33.33.33.10:
	  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
	                                 Dload  Upload   Total   Spent    Left  Speed
	100  6516  100  6516    0     0   5047      0  0:00:01  0:00:01 --:--:--  8165
	Downloading Chef 11.4.4 for ubuntu...
	Installing Chef 11.4.4
	Selecting previously unselected package chef.
	(Reading database ... 61109 files and directories currently installed.)
	Unpacking chef (from .../chef_11.4.4_amd64.deb) ...
	Setting up chef (11.4.4-2.ubuntu.11.04) ...
	Thank you for installing Chef!


如果安装死掉，请开vpn爬墙。以下操作，请仍然保持爬墙姿态。

	knife solo cook vagrant@33.33.33.10

继续喝茶，根据网速不等，需要花费十分钟到半个小时不等。让机器干活才是爽。我们可以看到，机器帮我们完成了111次重启、注册新用户、安装软件等操作。

	Recipe: nginx::source
	  * service[nginx] action reload (up to date)
	  * service[nginx] action restart
	    - restart service service[nginx]

	Recipe: railsbox::unicorn
	  * runit_service[railsbox-example-app1] action restart
	    - restart service runit_service[railsbox-example-app1]

	Chef Client finished, 111 resources updated


## 测试是否成功

在mac上的shell执行。

	ssh deploy@33.33.33.10

此时，应能免密码，登陆成功主机33.33.33.10。查看各个组建的版本，例如：

* ruby -v
* rbenv -v
* git --version
* curl --version
* psql --version
* ...


## 分发Rails App

在mac上的shell执行。

在`host.json`示范中，我们默认的appname为：`railsbox-example-app1`。该示范app地址是：

将它git clone 下来。

	git clone https://github.com/ouyangzhiping/railsbox-example-app1.git
	cd railsbox-example-app1

安装Gem集合：

	bundle install

初始化分发：

	bundle exec cap deploy:setup

	提示成功：
	
	* executing "chmod g+w /home/apps/railsbox-example-app1 /home/apps/railsbox-example-app1/releases /home/apps/railsbox-example-app1/shared /home/apps/railsbox-example-app1/shared/system /home/apps/railsbox-example-app1/shared/log /home/apps/railsbox-example-app1/shared/pids"
	  servers: ["33.33.33.10"]
	  [33.33.33.10] executing command
	  command finished in 60ms

## 服务器上的一些小准备

在`33.33.33.10`上的shell执行。

这些小准备，一般为了安全操作起见，不建议写入cookbook，也不建议纳入版本控制中。

### 准备数据库密码文件

开发与分发时，密码文件一般不能置入版本管理系统，因此，我们需要首先创建一个数据库密码文件。

仍然是： `ssh deploy@33.33.33.10`，登陆系统: 

	ssh deploy@33.33.33.10
	cd /home/apps/railsbox-example-app1/shared #注意与你之前的host.json文件中的appname对应
	mkdir config
	vi config/database.yml.production

内容填入：

```
common: &common
    adapter: postgresql
    username: psyapp
    password: psqlpassword
    #host: 33.33.33.10
 
development:
    <<: *common
    database: appname_production
 
test:
    <<: *common
    database: appname_production
 
production:
    <<: *common
    database: appname_production
```
`:wq 
`保存文件退出。此处请特别检查，复制内容是否齐全！尤其开头与结尾！

### 修改数据库

为了避免出现`FATAL: Peer authentication failed for user`的postgresql数据库报错信息。

	sudo nano /etc/postgresql/9.1/main/pg_hba.conf  #注意你安装的postgresql版本号！是9.1还是9.2

将其中的`peer` 改为 `md5`。

	# "local" is for Unix domain socket connections only
	local   all             all                                     peer

改为：

	# "local" is for Unix domain socket connections only
	local   all             all                                     md5


然后运行`sudo /etc/init.d/postgresql reload`重启postgresql。

	deploy@vagrant-ubuntu-precise-64:~$ sudo /etc/init.d/postgresql reload
	 * Reloading PostgreSQL 9.1 database server                              [ OK ]
	 
也可以找到:`railsbox-example\cookbooks\postgresql\templates\default\`目录下面的`pg_hba.conf.erb`文件，cook前修改好。

### 添加Github的knowhosts

使用：ssh -T git@github.com

提示，是否添加，选择`yes`。提示：

	Warning: Permanently added 'github.com,204.232.175.90' (RSA) to the list of known hosts.

以上操作均在`33.33.33.10`上的shell执行。

## 正式分发

在mac上的shell执行。回到`railsbox-example-app1`目录下面。部署数据库与种子文件

	bundle exec cap deploy:migrations

嗯，我们发现，机器又在吭哧吭哧帮我们干活了！心情很爽，离开几分钟，喝今天的第三杯茶去！

	** [out :: 33.33.33.10]
	** [out :: 33.33.33.10] Installing sprockets (2.10.0)
	** [out :: 33.33.33.10]
	** [out :: 33.33.33.10] Installing sprockets-rails (2.0.0)
	** [out :: 33.33.33.10]
	** [out :: 33.33.33.10] Installing rails (4.0.0)
	** [out :: 33.33.33.10]
	** [out :: 33.33.33.10] Installing rdoc (3.12.2)
	** [out :: 33.33.33.10]
	** [out :: 33.33.33.10] Installing sass (3.2.9)
	** [out :: 33.33.33.10]	

一切完成！

打开浏览器：`http://33.33.33.10`，我们即看到一个漂亮的rails应用，它还是完全基于rbenv+nginx+unicorn的！


## 常见问题


### knife solo prepare vagrant@33.33.33.10时授权报错

手动清除~/.ssh/known_hosts中的内容，或者使用vi快速清除：

	vi ~/.ssh/known_hosts
	按 `esc`退出，再按`dG`，清除掉known_hosts里的内容
	按`:wq`保存

### your Gemfile.lock into version control before deploying

记得在`.gitignore`中去掉对`gemfile.lock`文件的屏蔽。将其注释掉。


### FATAL:  Peer authentication failed for user "psyapp"

postgresql的问题，请记得修改相应配置，从peer改为md5。

## Author

Author:: zhiping (<http://yangzhiping.com>)