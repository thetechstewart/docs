---
author:
    name: Linode Community
    email: docs@linode.com
description: 'Host multiple rails apps with Nginx, Puma, Postgres, and rbenv on Ubuntu 14.04'
keywords: 'rails,ruby on rails,ror,ubuntu,linux,14.04,hosting,apps,puma,postgres,rbenv,nginx'
license: '[CC BY-ND 3.0](http://creativecommons.org/licenses/by-nd/3.0/us/)'
published: 'Thursday, September 10th, 2015'
modified: Thursday, September 10th 2013
modified_by:
    name: Linode
title: 'Hosting Multiple Rails Apps on Ubuntu 14.04'
contributor:
    name: Rick Peyton
    link: https://github.com/rickpeyton
---

Like any skilled Rails developer, you probably have several play apps. But just because these aren't serious production apps doesn't mean that you should have to host them somewhere that forces you to shut them down for six hours a day. Instead, follow this guide to host several Rails apps on a single Linode and subsequently avert daily shutdowns.

This guide provides step-by-step instructions for hosting multiple Rails apps on Ubuntu 14.04 (Trusty Tahr) with Linode.

{: .note }
>This guide is written for a non-root user. Commands that require elevated privileges are prefixed with `sudo`. If you’re not familiar with the sudo command, you can check our [Users and Groups](/docs/tools-reference/linux-users-and-groups) guide.

## Before You Begin

Ensure that you have followed the [Getting Started](https://www.linode.com/docs/getting-started) and [Securing Your Server](https://www.linode.com/docs/security/securing-your-server/) guides.

##Configuring Your First Rails App

### Install Postgres

1.  Update the system and install essentials by entering: 

        sudo apt-get update
        sudo apt-get upgrade --show-upgraded
        sudo apt-get install -y build-essential libssl-dev libreadline-dev zlib1g-dev git postgresql postgresql-contrib libpq-dev nodejs nginx

2.  Setup a postgres user with the same name as the current user:

        sudo -u postgres createuser --superuser $USER

    Log into postgres and set the password for your user, if you wish:

        sudo -u postgres psql
        \password username

    This guide assumes you will be using a local Postgres database; thus, you may wish to leave the password blank.

3.  To exit Postgres, type: 

        \q


### Install rbenv

1.  Install rbenv as user:

        git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
        echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
        echo 'eval "$(rbenv init -)"' >> ~/.bashrc

2.  Now, restart the shell (log out and back in) and check that rbenv is installed: 

        type rbenv

3.  Install ruby-build as an rbenv plugin: 

        git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build

4.  Install rbenv-vars to manage environment variables:

        git clone https://github.com/sstephenson/rbenv-vars.git ~/.rbenv/plugins/rbenv-vars

5.  Install the latest version of Ruby: 
        
        rbenv install 2.2.3

6.  Set the global ruby version: 
        
        rbenv global 2.2.3

7.  Confirm that it worked by verifying the version of Ruby:

        ruby -v
        ruby 2.2.3p173 (2015-08-18 revision 51636) [x86_64-linux]

### Configure Rails

1.  Install the rails gem:

        gem install rails

    {:.note }
    >
    >You may wish to speed up your install by omitting the docs; do so by entering:
    >
    >       gem install rails --no-ri --no-rdoc

2.  Likely, you will instead be cloning in a repo git repository. But for demo purposes, lets spin up a default Rails app:

        mkdir ~/www
        cd ~/www
        rails new default_app -T --database=postgresql

3.  Edit the database username in `config/database.yml` for the production environment:

        cd ~/www/default_app
        vim config/database.yml

4.  Once inside `database.yml`, you should see the following on your screen:

        ...
        production:
          <<: *default
          database: default_app_production
          username: rick
          password: <%= ENV['DEFAULT_APP_DATABASE_PASSWORD'] %>

5.  Set up your database: 

        RAILS_ENV=production rake db:create
        RAILS_ENV=production rake db:migrate

6.  Add puma to the Gemfile: 

        vim Gemfile

    Once inside your Gemfile, add puma by entering:

        gem 'puma'

7.  Save and exit your Gemfile and bundle it:

        bundle

8.  Add a secret to the environment: 

        rake secret

9.  Now, copy the secret and add it to the `.rbenv-vars` file: 

    {: .file-excerpt}
    .rbenv-vars
    :   ~~~
        SECRET_KEY_BASE=7ecb33b12412...
        ~~~

    {:.note }
    >
    >You can view your rbenv vars by entering:
    >    
    >       rbenv vars

### Configure Puma

Optimal configuration of Puma depends upon your Linode choice. The following config is based on the Linode 1GB plan. When needed, you can scale up your workers as your CPU count increases.

1.  Check the number of CPUs on your server:

        grep -c processor /proc/cpuinfo

2.  Create a Puma config file at `config/puma.rb`:

    {: .file}
    config/puma.rb
    :   ~~~
        # Change to match your CPU count
        workers 1

        threads 1, 6

        app_dir = File.expand_path("../..", __FILE__)
        shared_dir = "#{app_dir}/shared"

        rails_env = ENV['RAILS_ENV'] || "production"
        environment rails_env

        bind "unix://#{shared_dir}/sockets/puma.sock"

        stdout_redirect "#{shared_dir}/log/puma.stdout.log", "#{shared_dir}/log/puma.stderr.log", true

        pidfile "#{shared_dir}/pids/puma.pid"
        state_path "#{shared_dir}/pids/puma.state"
        activate_control_app

        on_worker_boot do
          require "active_record"
          ActiveRecord::Base.connection.disconnect! rescue ActiveRecord::ConnectionNotEstablished
          ActiveRecord::Base.establish_connection(YAML.load_file("#{app_dir}/config/database.yml")[rails_env])
        end
        ~~~

3.  Create the directories listed in the Puma config: 

        mkdir -p shared/pids shared/sockets shared/log

4.  Install Jungle to manage your Puma upstart scripts:

        cd ~
        wget https://raw.githubusercontent.com/puma/puma/master/tools/jungle/upstart/puma-manager.conf
        wget https://raw.githubusercontent.com/puma/puma/master/tools/jungle/upstart/puma.conf

5.  Edit the `puma.conf` you just grabbed and set the user equal to your user. Your file should have setuid apps and setgid apps. Change them using the name you chose in place of `username`:

    {: .file-excerpt}
    puma.conf
    :   ~~~    
        setuid username
        setgid username
        ~~~

6.  Copy the scripts to the init directory:

        cd ~
        sudo cp puma.conf puma-manager.conf /etc/init

    The `puma-manager.conf` script references `/etc/puma.conf` for a list of applications that it should manage. You can create and edit that inventory file by entering:

        sudo vim /etc/puma.conf

    and then, add the following. (Be sure to update the path with your username in place of `username`):

        /home/username/www/default_app

    You have just configured your application to start at boot.

7.  Start your Puma managed Rails app:
    
        sudo start puma-manager

    You may also start a single Puma application by entering the Puma script:
    
        sudo start puma app=/home/rick/www/default_app

    You may also use stop and restart to control the application:

        sudo stop puma-manager
        sudo restart puma-manager

Your application is not yet accessible to the outside world. To allow that access, you next have to configure Nginx.

### Configure Nginx

We already installed nginx when we setup our server. Let's configure it now.

1.  Open the default Nginx virtual host, located at `/etc/nginx/sites-available/default`.

2.  Replace the contents of the file with the following. Be sure to update the app path with your user (two locations):

    {: .file}
    /etc/nginx/sites-available/default
    :   ~~~
        upstream app {
            # Path to Puma SOCK file, as defined previously
            server unix:/home/username/www/default_app/shared/sockets/puma.sock fail_timeout=0;
        }

        server {
            listen 80;
            server_name localhost;

            root /home/username/www/default_app/public;

            try_files $uri/index.html $uri @app;

            location @app {
                proxy_pass http://app;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_redirect off;
            }

            error_page 500 502 503 504 /500.html;
            client_max_body_size 4G;
            keepalive_timeout 10;
        }
        ~~~

3.  Restart Nginx to put the changes into effect:

        sudo service nginx restart

Congratulations! The production environment of your Rails app should now be available via your server's public IP address or domain name. Unless you went off script, you are likely staring at a Rails error message; this is because we did not install a default route. But your Rails app is running!

{: .note}
>
>To fix the error message situation quickly, open `~/www/default_app/config/routes.rb` and uncomment `root 'welcome#index'`. Then, open `~/www/default_app/app/controllers/welcome_controller.rb` and generate a welcome message:
>
>     class WelcomeController < ApplicationController
>       def index
>         render inline: "Welcome to the default app!"
>       end
>     end
>
>Then restart puma using `sudo restart puma-manager`.

## Adding a Second Rails App

The first Rails app you set up was configured as your default route in Nginx, so it is accessible via your IP address as well as it's domain name. For this second app you will want to point an FQDN (Fully Qualified Domain Name) at your Linode.

For the purposes of this example, let's name this second app *stocks_app*.

The following command sequence by which you will add a second (and subsequent) rails app gets repetitive for a bit.

1.  Create the app:

        cd ~/www
        rails new stocks_app -T --database=postgresql

2.  Edit the database username in `config/database.yml` for the production environment:

    {: .file-excerpt}
    ~/www/stocks_app/config/database.yml
    :   ~~~
        ...
        production:
          <<: *default
          database: stocks_app_production
          username: rick
          password: <%= ENV['STOCKS_APP_DATABASE_PASSWORD'] %>
        ~~~

3.  Set up your database:

        RAILS_ENV=production rake db:create
        RAILS_ENV=production rake db:migrate

4.  Add puma to the Gemfile:

        vim Gemfile

    Once inside your Gemfile, add:

        gem 'puma'

5.  Save and exit your Gemfile and bundle it:

        bundle

6.  Add a secret to the environment:

        rake secret

7.  Copy the secret and add it to the `.rbenv-vars` file:

    {: .file-excerpt}
    .rbenv-vars
    :   ~~~
        SECRET_KEY_BASE=7ecb33b12412...
        ~~~

8.  Create a puma config file inside the second Rails app:

    {: .file}
    :   ~~~
        config/puma.rb

        # Change to match your CPU count
        workers 1

        threads 1, 6

        app_dir = File.expand_path("../..", __FILE__)
        shared_dir = "#{app_dir}/shared"

        rails_env = ENV['RAILS_ENV'] || "production"
        environment rails_env

        bind "unix://#{shared_dir}/sockets/puma.sock"

        stdout_redirect "#{shared_dir}/log/puma.stdout.log", "#{shared_dir}/log/puma.stderr.log", true

        pidfile "#{shared_dir}/pids/puma.pid"
        state_path "#{shared_dir}/pids/puma.state"
        activate_control_app

        on_worker_boot do
          require "active_record"
          ActiveRecord::Base.connection.disconnect! rescue ActiveRecord::ConnectionNotEstablished
          ActiveRecord::Base.establish_connection(YAML.load_file("#{app_dir}/config/database.yml")[rails_env])
        end
        ~~~

9.  Create the directories listed in the Puma config:

        mkdir -p shared/pids shared/sockets shared/log

10. Update the `/etc/puma.conf` script, adding the following at the end:

    {: .file-excerpt}
    /etc/puma.conf
    :   ~~~    
        /home/username/www/stocks_app
        ~~~

11. Restart Puma:

        sudo restart puma-manager

Your application is not yet accessible to the outside world. To allow that access, you have to add another virtual host to nginx.

### Nginx Configuration for the Second App

1.  Add the second virtual host opening the `/etc/nginx/sites-available/stocks_app` file and adding, updating your variables as needed:

    {: .file}
    /etc/nginx/sites-available/stocks_app
    :   ~~~
        upstream stocks_app {
            # Path to Puma SOCK file, as defined previously
            server unix:/home/rick/www/stocks_app/shared/sockets/puma.sock fail_timeout=0;
        }

        server {
            listen 80;
            server_name stocks.example.com;

            root /home/rick/www/stocks_app/public;

            try_files $uri/index.html $uri @stocks_app;

            location @stocks_app {
                proxy_pass http://stocks_app;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_redirect off;
            }

            error_page 500 502 503 504 /500.html;
            client_max_body_size 4G;
            keepalive_timeout 10;
        }
        ~~~

2.  Symlink the stocks_app host file into the sites-enabled folder:

        sudo ln -s /etc/nginx/sites-available/stocks_app /etc/nginx/sites-enabled/stocks_app

3.  Now, restart Nginx: 

        sudo service nginx restart

    (Congratulations! You should be up and running.)

4.  Configure the routes for the second site by opening `config/routes.rb` and uncommenting the line `root 'welcome#index'`.

5.  Make a controller by opening `app/controller/welcome_controller.rb` and generating a welcome message:

    {: .file}
    app/controller/welcome_controller.rb
    :   ~~~
        class WelcomeController < ApplicationController
          def index
            render inline: "Welcome to the stocks app!"
          end
        end
        ~~~

    Last, restart Puma:

        sudo restart puma-manager

You can repeat these steps to add each additional app you want.
