---
layout: post
title: 使用capistrano3 进行rails自动化部署
tags: rails capistrano
category: deploy
---

# Rails Capistrano 3自动化部署

## 环境及软件
	ubuntu 14.04
	nginx
	Capistrano 3
	unicorn

## 步骤

>https://github.com/cheenwe/capistrano-3-rails-template.git

### 1.添加Gemfile

```ruby
gem 'capistrano', '~> 3.1.0'
# rails specific capistrano funcitons
gem 'capistrano-rails', '~> 1.1.0'
# integrate bundler with capistrano
gem 'capistrano-bundler'
# if you are using RBENV
gem 'capistrano-rbenv', "~> 2.0"
# Use the Unicorn app server
gem 'unicorn'
```
### 2.添加配置

```ruby
bundle exec cap install
```
产生以下文件和目录结构:

```
├── Capfile
├── config
│   ├── deploy
│   │   ├── production.rb
│   │   └── staging.rb
│   └── deploy.rb
└── lib
    └── capistrano
            └── tasks
```

### 3 在 Capfile 添加以下内容
```ruby
Dir.glob('lib/capistrano/**/*.rb').each { |r| import r }
```
加载rb文件 lib / capistrano
及其子文件夹。 这是后来用于定义简单的辅助函数用于任务。

取消注释:

```ruby
require 'capistrano/bundler'
require 'capistrano/rbenv'
require 'capistrano/rails/migrations'
```

### 4 配置
尽可能多的常见的配置 config/ deploy.rb
特殊的配置 放在 config/deploy/production.rb中

```ruby
# config/ deploy.rb

set :application, '--这里是应用名称--'
set :deploy_user, '--这里是部署用户账号--'


# setup repo details
set :scm, :git
set :repo_url, '--这里是git地址--'

# setup rbenv.

set :rbenv_path, "/home/--这里是部署用户账号--/.rbenv"
set :rbenv_type, :system
set :rbenv_ruby, '2.2.0' #ruby 版本
set :rbenv_prefix, "RBENV_ROOT=#{fetch(:rbenv_path)} RBENV_VERSION=#{fetch(:rbenv_ruby)} #{fetch(:rbenv_path)}/bin/rbenv exec"
set :rbenv_map_bins, %w{rake gem bundle ruby rails}

# how many old releases do we want to keep, not much
set :keep_releases, 5

# files we want symlinking to specific entries in shared
set :linked_files, %w{config/database.yml}

# dirs we want symlinking to shared
set :linked_dirs, %w{bin log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system}

# what specs should be run before deployment is allowed to
# continue, see lib/capistrano/tasks/run_tests.cap
set :tests, []

# which config files should be copied by deploy:setup_config
# see documentation in lib/capistrano/tasks/setup_config.cap
# for details of operations
set(:config_files, %w(
  nginx.conf
  database.example.yml
  log_rotation
  monit
  unicorn.rb
  unicorn_init.sh
))

# which config files should be made executable after copying
# by deploy:setup_config
set(:executable_config_files, %w(
  unicorn_init.sh
))


# files which need to be symlinked to other parts of the
# filesystem. For example nginx virtualhosts, log rotation
# init scripts etc. The full_app_name variable isn't
# available at this point so we use a custom template {{}}
# tag and then add it at run time.
set(:symlinks, [
  {
    source: "nginx.conf",
    link: "/etc/nginx/sites-enabled/{{full_app_name}}"
  },
  {
    source: "unicorn_init.sh",
    link: "/etc/init.d/unicorn_{{full_app_name}}"
  },
  {
    source: "log_rotation",
   link: "/etc/logrotate.d/{{full_app_name}}"
  },
  {
    source: "monit",
    link: "/etc/monit/conf.d/{{full_app_name}}.conf"
  }
])

# this:
# http://www.capistranorb.com/documentation/getting-started/flow/
# is worth reading for a quick overview of what tasks are called
# and when for `cap stage deploy`

namespace :deploy do
  # make sure we're deploying what we think we're deploying
  before :deploy, "deploy:check_revision"
  # only allow a deploy with passing tests to deployed
  before :deploy, "deploy:run_tests"
  # compile assets locally then rsync
  # after 'deploy:symlink:shared'
  # after 'deploy:symlink:shared', 'deploy:compile_assets_locally' #由于我只提供api接口取消使用asset文件
  after :finishing, 'deploy:cleanup'

  # remove the default nginx configuration as it will tend
  # to conflict with our configs.
  before 'deploy:setup_config', 'nginx:remove_default_vhost'

  # reload nginx to it will pick up any modified vhosts from
  # setup_config
  after 'deploy:setup_config', 'nginx:reload'

  # Restart monit so it will pick up any monit configurations
  # we've added
  after 'deploy:setup_config', 'monit:restart'

  # As of Capistrano 3.1, the `deploy:restart` task is not called
  # automatically.
  after 'deploy:publishing', 'deploy:restart'
end

```

```ruby

set :stage, :production
set :branch, "unicorn" #部署的分支

# This is used in the Nginx VirtualHost to specify which domains
# the app should appear on. If you don't yet have DNS setup, you'll
# need to create entries in your local Hosts file for testing.
set :server_name, "--你部署的服务器ip,或者域名--"

# used in case we're deploying multiple versions of the same
# app side by side. Also provides quick sanity checks when looking
# at filepaths
set :full_app_name, "#{fetch(:application)}"

server '--你部署的服务器ip,或者域名--', user: '--部署的账号名--', roles: %w{web app db}, primary: true

set :deploy_to, "/home/#{fetch(:deploy_user)}/#{fetch(:full_app_name)}"

# dont try and infer something as important as environment from
# stage name.
set :rails_env, :production

# number of unicorn workers, this will be reflected in
# the unicorn.rb and the monit configs
set :unicorn_worker_count, 1 #开启unicorn进程数

# whether we're using ssl or not, used for building nginx
# config file
set :enable_ssl, false


```

其他文件夹
config/deploy
/lib/capistrano
复制到项目中


### 5 生成nginx/unicorn等配置文件

cap production deploy:setup_config

执行后文件位置在 部署文件夹/shared/config 内
nginx配置文件
unicorn配置文件

### 6编辑 config.yml 文件并拷贝到 shared/config下

### 7 部署

cap production deploy

部署完成

### 8 其他命令
rake logs:tail[production]
显示生产环境下日志 rails_app_path/shared/log/production.log

rake logs:tail[unicorn]
显示unicorn运行日志 rails_app_path/shared/log/production.log

