# Создание резервной копии MediaWiki
## Резервное копирование базы данных
MediaWiki хранит основное содержимое (страницы, истории редактирования, настройки пользователей и т.д.) в базе данных MySQL. Для создания резервной копии базы данных:

Подключитесь к серверу через SSH и выполните команду, заменив ```wikidb```, ```wikiuser``` и ```password``` на ваши данные:

```mysqldump -u wikiuser -p wikidb > /path/to/backup/mediawiki_db_backup.sql```

```/path/to/backup/``` — путь, куда будет сохранен файл резервной копии. Можно использовать, например, ```/home/youruser/backups/.```
Введите пароль для базы данных при запросе.

Файл ```mediawiki_db_backup.sql``` теперь содержит полный дамп базы данных.

## Резервное копирование файлов MediaWiki
Помимо базы данных, важно сохранить все файлы MediaWiki, особенно папку images, содержащую загруженные пользователями файлы, а также файл конфигурации LocalSettings.php.

Перейдите в директорию, где установлена MediaWiki (в вашем случае: ```/var/www/html/mediawiki/mediawiki-1.42.3)```.

Используйте команду tar, чтобы создать архив всех файлов MediaWiki:
```tar -czvf /path/to/backup/mediawiki_files_backup.tar.gz /var/www/html/mediawiki/mediawiki-1.42.3```
Это создаст сжатый архив ```mediawiki_files_backup.tar.gz```, содержащий все файлы MediaWiki.

# Восстановление MediaWiki на новом сервере
## Подготовка нового сервера
Установите необходимые компоненты LAMP (Linux, Apache, MySQL, PHP) на новом сервере.

Создайте новую базу данных и пользователя в MySQL, которые будут использоваться для MediaWiki (замените ```wikidb```, ```wikiuser``` и ```password``` на ваши данные):
```CREATE DATABASE wikidb;
CREATE USER 'wikiuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wikidb.* TO 'wikiuser'@'localhost';
FLUSH PRIVILEGES;
```
Установите ту же версию MediaWiki, что была на старом сервере.
## Восстановление файлов MediaWiki
Скопируйте файл архива ```mediawiki_files_backup.tar.gz``` на новый сервер. Используйте scp или загрузите файл через SFTP.

Перейдите в директорию, куда была установлена новая MediaWiki (например, /var/www/html/mediawiki/), и распакуйте архив:
```sudo tar -xzvf /path/to/backup/mediawiki_files_backup.tar.gz -C /var/www/html/mediawiki/```
Убедитесь, что права на файлы установлены корректно:
```
sudo chown -R www-data:www-data /var/www/html/mediawiki
sudo chmod -R 755 /var/www/html/mediawiki
```
## Восстановление базы данных
Скопируйте резервную копию базы данных ```mediawiki_db_backup.sql``` на новый сервер.

Импортируйте базу данных из резервной копии, используя команду mysql:
```mysql -u wikiuser -p wikidb < /path/to/backup/mediawiki_db_backup.sql```
## Шаг 4: Настройка LocalSettings.php
Проверьте файл ```LocalSettings.php``` в корневой директории MediaWiki, так как некоторые настройки могут потребовать обновления.

Убедитесь, что строка ```$wgServer``` указана с IP-адресом нового сервера или доменом (если он у вас есть).
Проверьте параметры подключения к базе данных (имя базы данных, пользователя и пароль).
Пример:
```
$wgDBserver = "localhost";
$wgDBname = "wikidb";
$wgDBuser = "wikiuser";
$wgDBpassword = "password";
$wgServer = "http://your-server-ip";
```
Сохраните изменения в ```LocalSettings.php```.
## Проверка
Перейдите в браузер и введите IP-адрес нового сервера (например, http://your-server-ip).
Проверьте, что все страницы, файлы и настройки корректно восстановлены.
