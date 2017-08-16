# Build Docker Image for Magento2

This is an example how to build a docker image for a Magento2 project e.g. to deploy it into a Kubernetes Cluster.

# Prepare project

This setup assumes all files of projects will be placed in folder `Source`. For demonstration purpose I use Magento2 CE repository from Github here.

    git clone https://github.com/magento/magento2.git Source
    cd Source && git checkout 2.1.7
    cd ..

For deployment, a lot of files need to be pre-generated. Unfortunately there is still a database required to do this. So we use a little help of docker for building these files inside containers. First start an app and a database container with docker-compose:

    docker-compose up -d

Execute all generation commands:

    docker-compose run --rm --no-deps php bash -c 'composer install --optimize-autoloader --no-dev'
    docker-compose run --rm --no-deps php bash -c 'bin/magento setup:install --db-host=mysql --db-name=magento2 --db-user=magento2 --db-password=magento2 --admin-user=admin --admin-password=admin123 --admin-email=admin@localhost.de --admin-firstname=admin --admin-lastname=admin --backend-frontname=admin --base-url=http://localhost:8123/'
    docker-compose run --rm --no-deps php bash -c "bin/magento dev:source-theme:deploy"
    docker-compose run --rm --no-deps php bash -c "php -d memory_limit=1024M bin/magento setup:di:compile"
    docker-compose run --rm --no-deps php bash -c "php -d memory_limit=1024M bin/magento setup:static-content:deploy"

# Optional: Install Sampledata

Usually, you would not want to do this, because all images would remain in pub/media and be included into the image.

    docker-compose run --rm --no-deps php bash -c "composer config repositories.magento composer https://repo.magento.com/"
    docker-compose run --rm --no-deps php bash -c "bin/magento sampledata:deploy"
    docker-compose run --rm --no-deps php bash -c "php -d memory_limit=1024M bin/magento setup:upgrade"

# Finish preparation

You may dump the database, if required:

    mysqldump -u magento2 -pmagento2 -h 0.0.0.0 --port 3307 magento2 > dump.sql

Stop intermediate containers:

    docker-compose stop

Restore your permissions, they may got destroyed:

    sudo chown -R $(whoami):$(whoami) Source

Build your image from prepared `Source` folder:

    docker build -f Docker/release/Dockerfile -t magento2-demo:1.0 .

Check your image to see if everything is in place:

    docker run -it magento2-demo:1.0 bash

# Push Image to Registry

    docker login
    docker tag magento2-demo:1.0 $USER/magento2-demo:1.0
    docker push $USER/magento2-demo:1.0
