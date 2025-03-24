References:

- https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-20-04#creating-the-postgresql-database-and-user


# 0. Before config

In this blog, I logged into an Ubuntu server with the username "Jake" and created a database as well as a database role called "tutoring". Please replace them with your own names.

### Install PostgreSQL on Ubuntu:

1. Refresh the server local package index:
     `$ sudo apt update`
2.  Install the Postgres package along with a `-contrib` package that adds some additional utilities and functionality:
     `$ sudo apt install postgresql postgresql-contrib`
3. Ensure that the service is started:
     `$ sudo systemctl start postgresql.service`



# 1. Log into an interactive Postgres session

By default, Postgres uses an authentication scheme called “peer authentication” for local connections. Basically, this means that if the user’s operating system username matches a valid Postgres username, that user can login with no further authentication.

```bash
jake@vm-ubuntu:~$ sudo -u postgres psql
[sudo] password for jake: 
psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1))
Type "help" for help.

```

Check current connection information:

```postgresql
postgres=# \conninfo
You are connected to database "postgres" as user "postgres" via socket in "/var/run/postgresql" at port "5432".
```



# 2. Create a database for Django project

```postgresql
postgres=# CREATE DATABASE tutoring;
CREATE DATABASE
postgres=# \l
                                     List of databases
     Name     |  Owner   | Encoding |   Collate   |    Ctype    |     Access privileges     
--------------+----------+----------+-------------+-------------+---------------------------
 postgres     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres              +
              |          |          |             |             | postgres=CTc/postgres
 template1    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres              +
              |          |          |             |             | postgres=CTc/postgres
 tutoring     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
(5 rows)

```



# 3. Create a database role (user) for Django project

```postgresql
postgres=# CREATE USER tutoring WITH PASSWORD 'yourpassword';
CREATE ROLE
```

```postgresql
postgres=# \du
                                     List of roles
  Role name   |                         Attributes                         | Member of 
--------------+------------------------------------------------------------+-----------
 postgres     | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 tutoring     |                                                            | {}

```

Set the new role as a `SUPERUSER`. (This is optional. Without this modification, you may get a warning   `permission denied to create database` when running some Django test code.)

```postgresql
postgres=# ALTER USER tutoring WITH SUPERUSER;
ALTER ROLE
```

```postgresql
postgres=# \du
                                     List of roles
  Role name   |                         Attributes                         | Member of 
--------------+------------------------------------------------------------+-----------
 postgres     | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 tutoring     | Superuser                                                  | {}

```



# 4. Modify connection parameters for the new role

Modify a few of the connection parameters for the user we just created. This will speed up database operations so that the correct values do not have to be queried and set each time a connection is established.

Reference: https://docs.djangoproject.com/en/4.1/ref/databases/#optimizing-postgresql-s-configuration

```postgresql
postgres=# ALTER ROLE tutoring SET client_encoding TO 'utf8';
ALTER ROLE
postgres=# ALTER ROLE tutoring SET default_transaction_isolation TO 'read committed';
ALTER ROLE
postgres=# ALTER ROLE tutoring SET timezone TO 'UTC';
ALTER ROLE
```



# 5. Give the new role access to administer the new database

```postgresql
postgres=# GRANT ALL PRIVILEGES ON DATABASE tutoring TO tutoring;
GRANT
```

```postgresql
postgres=# \l
                                     List of databases
     Name     |  Owner   | Encoding |   Collate   |    Ctype    |     Access privileges     
--------------+----------+----------+-------------+-------------+---------------------------
 postgres     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres              +
              |          |          |             |             | postgres=CTc/postgres
 template1    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres              +
              |          |          |             |             | postgres=CTc/postgres
 tutoring     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres             +
              |          |          |             |             | postgres=CTc/postgres    +
              |          |          |             |             | tutoring=CTc/postgres
(5 rows)

```

Change the Database owner from postgres (default) to the new role. (This is optional.)

```postgresql
postgres=# ALTER DATABASE tutoring OWNER to tutoring;
ALTER DATABASE
```

```postgresql
postgres=# \l
                                     List of databases
     Name     |  Owner   | Encoding |   Collate   |    Ctype    |     Access privileges     
--------------+----------+----------+-------------+-------------+---------------------------
 postgres     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres              +
              |          |          |             |             | postgres=CTc/postgres
 template1    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres              +
              |          |          |             |             | postgres=CTc/postgres
 tutoring     | tutoring | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/tutoring             +
              |          |          |             |             | tutoring=CTc/tutoring
(5 rows)

```

Exit out of the PostgreSQL prompt:

```postgresql
postgres=# \q
jake@ubuntu:~$ 
```


# 6. Config Django settings.py

Reference: https://docs.djangoproject.com/en/4.1/ref/settings/#databases

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'tutoring',
        'USER': 'tutoring',
        'PASSWORD': 'yourpassword',
        'HOST': 'localhost',
        'PORT': '5432',
    },
}
```
