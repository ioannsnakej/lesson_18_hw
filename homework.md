Задание 1:
1. Создать два сервера, установить на них PostgreSQL и подключить их к
одной сети.
2. Настроить репликацию между серверами, чтобы изменения, вносимые
на одном сервере, автоматически реплицировались на другой.
3. Настроить отказоустойчивость, используя репликацию и механизм
автоматического переключения между серверами.
4. Проверить работу отказоустойчивого решения, симулируя отказ одного
из серверов и убедившись, что второй сервер продолжает работу и все
данные сохранены.

Устанавливаем PostgreSQL на наши сервера:

  <img width="626" height="196" alt="image" src="https://github.com/user-attachments/assets/bcf6c223-5a70-4ff9-a925-91427b013a1e" />
  <img width="586" height="179" alt="image" src="https://github.com/user-attachments/assets/b0aa8549-f532-40e2-80d3-7239243d05b9" />

На первом сервере подключаемся к бд и создаем роль replicator с атрибутом replication:

  <img width="901" height="223" alt="image" src="https://github.com/user-attachments/assets/6d505635-ea8b-4a8d-870d-5fdd2a7d44e5" />

Добавляю роль в файл pg_hba.conf:

  <img width="752" height="47" alt="image" src="https://github.com/user-attachments/assets/ad155e90-5ca9-436a-9458-b768ac5701b3" />

Правлю /etc/postgresql/14/main/postgresql.conf:

  <img width="1284" height="257" alt="image" src="https://github.com/user-attachments/assets/6395b464-14f3-4235-a551-9d03a4db33a5" />



