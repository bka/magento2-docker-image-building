FROM bleers/magento2-base:1.0

ADD Source /var/www/html
ADD Docker/release/files/vhost.conf /etc/apache2/sites-enabled/000-default.conf
ADD Docker/release/files/env.php /var/www/html/app/etc/env.php
RUN chown -R www-data:www-data /var/www/html
