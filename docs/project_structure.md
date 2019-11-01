# Project Structure

To simplify moving our code, configuration, and data between the container and local filesystem, it is best to designate few key directories.

Sample structure:

```
.
├── config
│   └── nginx
│   │   ├── certs
│   │   │   ├── nginx-selfsigned.crt
│   │   │   └── nginx-selfsigned.key
│   │   └── d2compare.conf
│   └── mysql
│       └── my.conf
├── data
│   └── <empty dir>
├── src
│   ├── app_code
│   └── app.py
├── Dockerfile
├── docker-compose.prod.yml
└── docker-compose.yml
```

## Config

Application configuration files, that will be mapped into the container.

These configurations do not have to be tied to your source code, and can be mapped into any destination within the container.

E.g. mapping `config/nginx/default.conf` to `/etc/nginx/sites-available`


## Data Directories

While it is recommended to keep data in persistent Docker volumes, there may be need to retain data outside of Docker.

To store data externally, you can map a local directory to a data directory in the container, and have Docker mirror the container directory/files into your local filesystem.

e.g. retaining a MySQL database, we would map `data` to `/var/lib/mysql`


## Src

Application source code for the application.

If applicable, all source code that runs in the application should live here. If there are multiple applications being brought up, they should be mapped into their own source directories.


## Docker Files

Individual files that are used to configure our Docker stack, which should live at the root of the project directory.

- Dockerfile
- docker-compose.yml
- docker-compose.prod.yml
- .dockerignore
