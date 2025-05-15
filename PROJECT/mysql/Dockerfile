FROM mysql:8.0

ENV MYSQL_ROOT_PASSWORD=admin@123 \
    MYSQL_DATABASE=aja_lib \
    MYSQL_USER=admin \
    MYSQL_PASSWORD=Admin@1234!

COPY scripts/*.sql /docker-entrypoint-initdb.d/

