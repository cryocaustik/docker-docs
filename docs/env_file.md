# Environment FIle

While docker compose does allow us to define variables within the `docker-compose.yml` file, this is not a secure or effective way to store sensitive information, as the docker-compose file should be committed and tracked in your version control, without concern of sharing passwords, usernames, etc.

To simplify management of sensitive information between docker-compose configuration files, and allow for re-usability of variables, docker-compose recognizes `.env` files that live alongside the compose file.

In the below example, we will define how to use `.env` files, version control them, and share the variables in docker-compose:

## Environment File Example

While our actual `.env` file should never be committed or left in unsecured areas, it is very helpful to include an example of the actual file, showing what variables are required in an actual `.env`.

Create a `.env.example` file and fill in your variable names, but not the values:

```env
MYSQL_ROOT_PASSWORD=
MYSQL_DATABASE=
MYSQL_USER=
MYSQL_PASSWORD=
```

## Live Environment File

```env
MYSQL_ROOT_PASSWORD=root_pwd
MYSQL_DATABASE=app_db
MYSQL_USER=db_admin
MYSQL_PASSWORD=dn_admin_pwd
```

## Use Env Variables in docker-compose

Once we've defined our actual environment file, we can now call on them from our docker-compose files by using `${variable_name}`.

Example:

```yaml
services:
  node_db:
    container_name: node_db
    image: mysql:latest
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
```