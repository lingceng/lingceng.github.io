---
layout: post
title: "Understand Capistrano"
date: 2017-07-29 19:00 
comments: true
categories: ["rails", "deploy"]
---

[Capistrano](https://github.com/capistrano/capistrano) is based on **Rake** and **SSHKit**.

Capistrano adds some DSL to **Rake**, such as `on roles(:app)`, `within release_path` and `with rails_env: fetch(:rails_env)`.
`cap -T` can list all Capistrano tasks, which is similar with `rake -T`. Capistrano executes commands with **SSHKit**.

When we run `cap install STAGE=feature`. It generates three files:

    ./Capfile
    ./config/deploy.rb
    ./config/deploy/feature.rb

Capistrano first load _Capfile_, then _deploy.rb_ and last _feature.rb_.

### Lazy set the configuration

See code below.
Things goes wrong if you set **deploy_to** (shared_path depends on deploy_to) at _feature.rb_.
You should use a block to do a **lazy set**.

    # ./config/deploy/stage.rb
    set :deploy_to, "/home/develop/#{fetch(:application)}/production"


    # ./config/deploy.rb
    # WRONG
    set :puma_bind, "unix://#{shared_path}/tmp/sockets/#{fetch(:application)}-puma.sock"

    # OK
    set :puma_bind, -> { "unix://#{shared_path}/tmp/sockets/#{fetch(:application)}-puma.sock" }


### Custom tasks
Let's say, we want to start some program after deploy. How to mount the task?  
The _lib/capistrano/tasks/framework.rake_ in Capistrano source has defined the structure.

    deploy:starting
    deploy:started
    deploy:updating
    deploy:updated
    deploy:publishing
    deploy:published
    deploy:finishing
    deploy:finished

Here's an example.
The `ruby scripts/ctrl_process_accuse.rb restart` will be executed after deploy.

    namespace :deploy do
      desc "Restart process accuse"
      task :restart_process_accuse do
        on roles(:db) do
          within release_path do
            with rails_env: fetch(:rails_env) do
              execute :ruby, "scripts/ctrl_process_accuse.rb restart"
            end
          end
        end
      end

      after :finished, :restart_process_accuse
    end

Here the generated commands.
I used **capistrano-rvm** which will wrap commands like _ruby_, _gem_ or _rake_.

    DEBUG [6dfb3053] Command: cd /home/develop/cms/production/releases/20170
    729094810 && ( export RAILS_ENV="production" ; ~/.rvm/bin/rvm default do
     ruby scripts/ctrl_process_accuse.rb restart )

You can also run the task alone. Option '--trace' is to show task invoke details.

    rake feature deploy:restart_process_accuse --trace

### The roles
The most used roles are: **app, web and db**. Here's some default roles.

    plugin                      | default role
    ---                         - ---
    capistrano/puma             | app
    capistrano/rails/assets     | web
    capistrano/rails/migrations | db
    whenever/capistrano         | db

We often set only one server with role **db**.
So you can specify some tasks that can only run once with **db** role.
