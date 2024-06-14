# authenticated-hive-docker
I haven't found any available projects for simple deployment of hive services with authentication mechanisms, so I shared it.

Actually, Hive supports many types of authentication methods, and I used what I think is the simplest one, ldap.

1.install ldap
```shell
$ docker-compose up -d
$ docker cp base.ldif my-openldap-container:/base.ldif
$ docker exec -it my-openldap-container ldapadd -x -D "cn=admin,dc=example,dc=com" -w adminpassword -f /base.ldif
```
2.ensure that the normal account is set up properly

open `https://localhost:6443`, login DN is `cn=admin,dc=example,dc=com` and password is `adminpassword` which is from `docker-compose.yml`

3.edit `hive-site.xml` and replace `ldap://192.168.64.1:389` to `ldap://your_ldap_ip:389`(`localhost` or `127.0.0.1`
is not useful, use the ip address of docker bridge interface)

4.run the hive instance with `hive-site.xml`

```bash
$ docker run -d -p 10000:10000 -p 10002:10002 --env SERVICE_NAME=hiveserver2 --name hive4 -v ./hive-site.xml:/opt/hive/conf/hive-site.xml apache/hive:4.0.0
```
5.ensure status of hive with open `http(s)://hive_addr:10002`

6.connect hive with incorrect authorization for testing and it doesn't work

```shell
$ docker exec -it hive4 beeline -u 'jdbc:hive2://localhost:10000/default' -n test -p xxxxx
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/hive/lib/log4j-slf4j-impl-2.18.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/opt/hadoop/share/hadoop/common/lib/slf4j-reload4j-1.7.36.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/hive/lib/log4j-slf4j-impl-2.18.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/opt/hadoop/share/hadoop/common/lib/slf4j-reload4j-1.7.36.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Connecting to jdbc:hive2://localhost:10000/default
24/05/26 14:56:19 [main]: WARN jdbc.HiveConnection: Failed to connect to localhost:10000
Unknown HS2 problem when communicating with Thrift server. Enable verbose error messages (--verbose=true) for more information.
Error: Could not open client transport with JDBC Uri: jdbc:hive2://localhost:10000/default: Peer indicated failure: Error validating the login (state=08S01,code=0)
[WARN] Failed to create directory: /home/hive/.beeline
No such file or directory
```
