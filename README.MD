# Preparation
* cp .env.example .env
* cp docker-compose.yml.example docker-compose.yml
* cp container/nginx/config/vhost.conf.dist container/nginx/config/vhost.conf
* APP_NAME=app_name
* sed -i "s/project_identifier/${APP_NAME}/g" docker-compose.yml container/nginx/config/vhost.conf
# Run
* docker-compose up -d
# setup phpstorm xdebug
* open Settings>Servers
* create new server:
  * name xdebug
  * host localhost
  * mapping should be updated according to your filesystem
