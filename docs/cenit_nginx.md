# Cenit with Unicorn and Nginx

You can deploy our plataform in an standard server too. To deploy this you need to configure Unicorn and NGINX servers.

## Unicorn configuration

You don't need to configure **_unicorn.rb_** file, because it's configured:

```ruby
# set path to the application
app_dir = File.expand_path("../..", __FILE__)
shared_dir = "#{app_dir}/shared"
working_directory app_dir

# Set unicorn options
worker_processes 2
preload_app true
timeout 30

# Path for the Unicorn socket
listen "#{shared_dir}/sockets/unicorn.sock", :backlog => 64

# Set path for logging
stderr_path "#{shared_dir}/log/unicorn.stderr.log"
stdout_path "#{shared_dir}/log/unicorn.stdout.log"

# Set proccess id path
pid "#{shared_dir}/pids/unicorn.pid"
```

## Install and Configure Nginx

1. Install nginx:

    ```bash
    sudo apt-get install nginx
    ```

2. We need to configure nginx to work as the reverse proxy. Edit the config file /etc/nginx/nginx.conf and paste the following configuration in the HTTP block:

    ```conf
    pstream rails {
    # Path to Unicorn socket file
    server unix:/home/username/example/shared/sockets/unicorn.sock fail_timeout=0;
    }
    ```

    >**NOTE:**
    Edit username and example with appropriate values.

3. Remove the default nginx site configuration:

    ```bash
    sudo rm /etc/nginx/sites-enabled/default
    ```

4. Create new nginx site configuration file for Cenit App:

    ```conf
    server {
    listen 80;
    server_name localhost;

    root /home/username/example;

    try_files $uri/index.html $uri @cenit;

    location @rails {
    proxy_pass http://cenit;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
    proxy_redirect off;
    }

    error_page 500 502 503 504 /500.html;
    client_max_body_size 4G;
    keepalive_timeout 10;
    }
    ```

    >**NOTE:**
    Make sure you change the username and example with the appropriate values.

5. Create a symlink to nginx’s sites-enabled directory to enable your site configuration file:

    ```bash
    sudo ln -s /etc/nginx/sites-available/example /etc/nginx/sites-enabled
    ```

6. Restart nginx:

    ```bash
    sudo service nginx restart
    ```
