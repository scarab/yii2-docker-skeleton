# yii2-docker-skeleton
Skeleton for work with Yii2 in docker environment

## Installation

1. Clone repository.
2. Build base image:
```
make build
```
3. If you don't have project and want to create new one, do:
```
make first-install
```
to install Yii2 basic version

OR 
```
make first-install YII_INSTALL_VERSION=advanced
```
to install Yii2 advanced version.

4. Start your stack to work:
```
make start
```

## Some other useful commands:
- `make start` - runs your stack
- `make stop` - stops it
- `make ssh` - enter into your web application ssh console
- `make help` or just "make" to see other stuff

## Directory structure

* **application** - your application directory. It mounts locally for development and will be copied into docker image for production.
* **docker** - all infrastructure and docker data:
  * **build** - build configuration
  * **config** - configuration files for different environments: dev, staging, production
  * **runtime** - here you can find all your runtime data - logs, database files etc.