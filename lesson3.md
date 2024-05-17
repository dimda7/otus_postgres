1. поставил postgresql
+ sudo apt update && sudo apt upgrade -y -q
+ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' 
+ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15

2. после следующей команды увидил, что бд запустилась
+ sudo -u postgres pg_lsclusters

3. Поменял пароль и Создал тестувую таблицу в бд
+ sudo -u postgres psql
+ alter user postgres password 'postgres';
+ create table test(c1 text);
+ insert into test values('1');

4. Остановил бд
+ sudo -u postgres pg_ctlcluster 15 main stop

5. Создал диск и проинициализировал по инструкции( https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux)
+ sudo parted /dev/vdb mklabel gpt
+ sudo parted -a opt /dev/vdb mkpart primary ext4 0% 100%
+ sudo mkfs.ext4 -L datapartition /dev/vdb
+ sudo mkdir -p /mnt/data
sudo mount -o defaults /dev/vdb /mnt/data

6. Добавил в файл /etc/fstab строку для автоматического монтирования файловой системы при загрузке
+ LABEL=datapartition /mnt/data ext4 defaults 0 2

7. Поменял пользователя postgres на вновь созданный раздел и перенес данные бд
+ chown -R postgres:postgres /mnt/data/
+ mv /var/lib/postgresql/15 /mnt/data

8. Попытался запустить бд, но т.к данных не было на старом месте бд не стартанула
+ sudo -u postgres pg_ctlcluster 15 main start

9. В файле /etc/postgresql/15/main/postgres.conf  поменял параметр data_directory на /mnt/data/15

10. Запустил бд
+ sudo -u postgres pg_ctlcluster 15 main start

11. через psql провеил, что в таблице тест лежит ранее созданная запись
+ sudo -u postgres psql
+ select * from test;