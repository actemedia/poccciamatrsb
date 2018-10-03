FROM wordpress:4.9.8-php7.2-fpm-alpine

RUN { \
		mv $PHP_INI_DIR/php.ini-production $PHP_INI_DIR/php.ini; \
		sed -r -i 's/session.save_handler = files/session.save_handler = redis\nsession.save_path = "redis:6379"/g' $PHP_INI_DIR/php.ini; \
		sed -r -i 's/;date.timezone =/date.timezone = Europe\/Paris/g' $PHP_INI_DIR/php.ini; \
		sed -r -i 's/<div class="wrap">/\<div class="wrap"\> \<h5\>Noeud de traitement de la requ\&ecirc;te\<\/h5\>\<pre\>\<\?php print php_uname(); \?\>\<\/pre\>\<h5\>Noeud ayant ouvert votre session (requ\&ecirc;te initiale)\<\/h5\>\<pre\>\<\?php print \$_SESSION["uname"]; \?\>\<\/pre\>/g' /usr/src/wordpress/wp-content/themes/twentyseventeen/index.php; \
		echo 'function register_my_session(){if(!session_id()){session_start();if(!$_SESSION["uname"]){$_SESSION["uname"]=php_uname();}}}add_action("init","register_my_session");' >> /usr/src/wordpress/wp-content/themes/twentyseventeen/functions.php; \
		apk add --no-cache --update --virtual .build-deps $PHPIZE_DEPS \
		&& pecl install redis-4.1.1 \
		&& docker-php-ext-enable redis \
		&& apk del .build-deps \
		&& rm -rf /tmp/pear ~/.pearrc; \
}
