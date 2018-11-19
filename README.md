### Padrino
---
http://padrinorb.com/

https://github.com/padrino/padrino-framework

```sh
gem install padrino
```

```ruby
use Rack::Cors do
  allow do
    origins
    resource '*', :headers => :any, :methods => [:get, :post, :options]
  end
end

require File.expand_path("../config/boot.rb", __FILE__)
require 'resque/server'
run Rack::URLMap.new \
  "/" => Padrino.application,
  "/resque" => Resque::Server.new

require 'resque/tasks'
```

```ruby
Padrino.configure_apps do
  begin
    logger.info "Opening: #{Padrino.root}/config/my_main_config.yml"
    set :mainConfigYml, YAML::load(File.open("#{Padrino.root/config/main.yml}"))["main"]
    logger.info "Opening: #{Padrino.root}/config/my_other_config.yml"
    set :webConfigYml, YAML::load(File.open("#{Padrino.root}/config/my_other_config.yml"))["web"]
  rescue
    logger.fatal "ERROR: Cannot open config files!"
    exit
  end
end

Myapp.helpers do
  def config_get(path)
    if !path_get(path)
      logger.fatal ""
      error 500, ""
    else
      if path.match /^()$/
        logger.info "" + path
        partial_config = nil
        for node in path.split().each do
          if node.length > 0
            if partial_config.nil?
              if node == "web"
                partial_config = @webConfig
              elsif node == "main"
                partial_config = @mainConfig
              end
            else
              if partial_config[node].nil?
                logger.fatal "Undefined node " + node + "' in path '" + path + "'"
              else
                partial_config = partial_config[node]
              end
            end
          end
        end
        logger.info "ConfigPath '" + path + "' --> '" + partial_config.to_s + "'"
        return partial_config.to_s
      else
        logger.fatal "Invalid config path: " + path
        error 500, "Invalid config path!"
      end
    end
  end
end

Stop.controllers :front do
  before do
    @webConfig = settings.webConfigYml
    @mainConfig = settings.mainConfigYml
  end
end
```

```yml
%h1 = config_get '/main/enterprise/name'
%h2 Call us at: #{config_get '/main/enterprise/phone'}
%h2 Or send mail to: #{config_get '/main/contact/email'}

!!! 5
%html
  %head
    %meta{:content => "text/html; charset=utf-8", "http-equiv" => "Content-Type"}
    %meta{:author => "#{config_get '/main/enterprise/name'}"}
    %meta{:keywords => "#{config_get '/web/keywords'}"}
    %meta{:description => "#{config_get '/web/keywords'}"}
    %title= config_get '/web/title'
```


