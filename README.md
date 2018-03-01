# Minimal Rails Deployment

Deploy a Ruby on Rails application to server from zero 101

PS: only deployment, skip server configuration and libraries installation÷, Rails development (may add latter)

- ruby on rails
- capistrano
- nginx
- rbenv

New a rails project (Rails 5.1.5)

Add a page controller instead of using default welcome page

```
rbenv local 2.5.0
```

Add gems to `Gemfile`

```
gem 'capistrano', require: false
gem 'capistrano-rails', require: false
gem 'capistrano3-puma'
gem 'capistrano-rbenv'
gem 'capistrano-bundler'
```

Exec `bundle install`

Exec `bundle exec cap install`

Edit `Capfile`

Uncomment

```
require "capistrano/rbenv"
require "capistrano/bundler"
require "capistrano/rails/assets"
require "capistrano/rails/migrations"
```

Add

```
require "capistrano/puma"
install_plugin Capistrano::Puma
install_plugin Capistrano::Puma::Nginx
```

Edit `config/deploy.rb`

```
set :application, "<project_name>"
set :repo_url, "git@xxx"
set :rbenv_type, :user
set :rbenv_ruby, File.read('.ruby-version').strip
```

Uncomment

```
append :linked_files, "config/database.yml", "config/secrets.yml"
append :linked_dirs, "log", "tmp/pids", "tmp/cache", "tmp/sockets", "public/system"
```

Edit `config/deploy/production.rb`

```
set :branch, "master"
set :deploy_to, "<project_path>"
server "<server_ip>", user: "<user>", roles: %w{web app db}
```

Upload nginx config (default 80 port)

```
bundle exec cap production puma:nginx_config
```

SSH to server, create files

```
mkdir -p <project_path>/shared/config
touch <project_path>/shared/config/database.yml
touch <project_path>/shared/config/secrets.yml
```

Edit `<project_path>/shared/config/database.yml` (copy from `<local_project_path>/config/database.yml`)

```
default: &default
  adapter: sqlite3
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  timeout: 5000

production:
  <<: *default
  database: db/production.sqlite3
```

Exec `rails secret` in local project path to gen a secret key

Edit `<project_path>/shared/config/secrets.yml`

```
production:
  secret_key_base: <generated_key>
```

deploy, remember to push project to git server

```
bundle exec cap production deploy
```

restart nginx in server
```
sudo service nginx restart
```
