# TicketMonster - a JBoss example

TicketMonster is an online ticketing demo application that gets you started with JBoss technologies, and helps you learn and evaluate them.

Here are a few instructions for building and running it. You can learn more about the example from the [tutorial](http://www.jboss.org/ticket-monster).

## Build and run locally

```
mvn clean install
```

This will build our application into a fat-jar that gets bootstrapped with [WildFly Swarm](http://wildfly-swarm.io). This can be run with a simple `java -jar` command line because all of its dependencies and classpath modules are included in the uber jar. For example, after running `mvn install` you can run the microservice like this:

```
java -jar target/ticket-monster-orders-swarm.jar 
```

If you're changing this often, these two steps may not be ideal. You can run it with one command:

```
mvn wildfly-swarm:run
```

## Ticket Monster with Docker

To run with docker:

```
mvn clean package -Pdefault,f8-build docker:build
docker run -it --rm -p 8080:8080 -e AB_OFF=true fabric8/ticket-monster-orders:1.0-SNAPSHOT
```

Can port forward to your local machine from vagrant like this:

> vagrant ssh -- -vnNTL *:8080:$DOCKER_HOST_IP:8080

## Running with a MySQL backend:

```
mvn clean package -Pmysql
```

## Ticket Monster on Kubernetes

```
mvn -Pf8,mysql fabric8:deploy
```



## Bootstrap the databases

We need to refer to the mysql database template from [https://github.com/christian-posta/ticket-monster-infra](https://github.com/christian-posta/ticket-monster-infra). We can install the template with:

```
oc create -f mysql-openshift-template.yml
```

Now let's create the mysql database for the admin microservice:

```
oc process ticket-monster-mysql -v DATABASE_SERVICE_NAME=mysqlorders | oc create -f -
oc deploy mysqlorders --latest
```


## Liquibase commands

As root, we can reset the database:

```
mysql> drop database ticketmonster; create database ticketmonster;
```

```
mvn -Pdb-migration-mysql liquibase:update
mvn -Pdb-migration-mysql liquibase:updateSQL
mvn -Pdb-migration-mysql liquibase:status
mvn -Pdb-migration-mysql liquibase:tag -Dliquibase.tag=v2.0
```

Automatically discover diffs between the existing schema and what Hibernate sees currently

```
mvn -Pdb-migration-mysql liquibase:diff -Dliquibase.referenceUrl=hibernate:ejb3:primary
```

Generate the changeLog to a file:
```
mvn -Pmysql,db-migration-mysql liquibase:diff -Dliquibase.referenceUrl=hibernate:ejb3:primary -Dliquibase.diffChangeLogFile=target/changes.yml
```

From here you can evaluate what changes should go into the next `update`

After an update, it's recommended to tag if things went successful, or rollback if not.

Also, you can see what tags exist in the DB:

```
select ID, DATEEXECUTED,TAG from DATABASECHANGELOG WHERE TAG IS NOT NULL ORDER BY DATEEXECUTED;
```



TODO: eliminate sending back data model elements directly; we should make more clearly the "view" model and the "write" model .. this will involve cleaning up the DTO objects if they're not needed in the write model