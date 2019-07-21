# Django Docker

## 1. Create executable build_local.sh

Copy `build_local_example.sh` to `build_local.sh`.

Edit the `build_local.sh` file and add sensible values there.

Add execution permissions:

```bash
$ chmod +x build_local.sh
```

## 2. Build the Docker containers

Run `build_local.sh`:

```bash
$ ./build_local.sh
```

## 3. Check if the build was successful

If you now go to `http://0.0.0.0/` you should see a "Hello, World!" page there.

If you now go to `http://0.0.0.0/admin/`, you should see 

```
OperationalError at /admin/
FATAL:  role "myproject" does not exist
```

This means that you have to create the user and the database in the Docker container.

## 4. Create database user and project database

SSH into the database container and create user and database there with the same values as in the `.build_local.sh` script:

```bash
$ docker exec -it django_docker_db_1 bash
/# su - postgres
/$ createuser --createdb --password myproject
/$ createdb --username myproject myproject
```

Press [Ctrl + D] twice to logout of the postgres user and Docker container.

If you now go to `http://0.0.0.0/admin/`, you should see 

```
ProgrammingError at /admin/
relation "django_session" does not exist
LINE 1: ...ession_data", "django_session"."expire_date" FROM "django_se...
```

This means that you have to run migrations to create database schema.

## 5. Run migration and collectstatic commands

SSH into the gunicorn container and run the necessary Django management commands:

```bash
$ docker exec -it django_docker_gunicorn_1 bash
/usr/src/myproject# python manage.py migrate
/usr/src/myproject# python manage.py collectstatic
/usr/src/myproject# python manage.py createsuperuser
```

Press [Ctrl + D] twice to logout of the Docker container.

If you now go to `http://0.0.0.0/admin/`, you should see the Django administration where you can login with the super user's credentials that you have just created.

## 7. Overview of useful commands

### Rebuild docker image

```bash
$ docker-compose down
$ ./build_local.sh
```

### SSH to the Docker images

```bash
$ docker exec -it django_docker_gunicorn_1 bash
$ docker exec -it django_docker_nginx_1 bash
$ docker exec -it django_docker_db_1 bash
```

### View logs

```bash
docker-compose logs nginx
docker-compose logs gunicorn
docker-compose logs db
```

## 8. Create analogous scripts for staging, production, and test environments

Copy `build_local.sh` to `build_staging.sh`, `build_production.sh`, and `build_test.sh` and change the environment variables analogously.
