FROM postgres:16.0
LABEL SI="Sergei Ionin"
ENV POSTGRES_PASSWORD=postgres
ENV POSTGRES_USER=postgres
ENV POSTGRES_DB=testdb
COPY init/init.sql /docker-entrypoint-initdb.d/init.sql
WORKDIR /db
COPY . . /db/
VOLUME [ "/datavolume" ]