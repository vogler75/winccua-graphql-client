# GraphQL Client for SIEMENS WinCC Unified

It is a Custom Web Control to get access to GraphQL servers.

In the "GraphQL-Client-Demo" example [Hasura.io](https://hasura.io) is used to get access to a PostgreSQL database.

See the example TIA V18 project.

## PostgreSQL Table
``` 
CREATE TABLE ORDERS (
	ORDERNR VARCHAR(30),	
	CUSTOMER VARCHAR(100),	
	AMOUNT integer, 
	STATUS VARCHAR(30),
	PRIMARY KEY (ORDERNR))
```

## Microsoft IIS Configuration
You have to add a rule to your IIS configuration, to forward requests to your hasura instance.  
In that example the Hasura server is running at the host 192.168.1.3 and listens on port 8089.  
C:\Program Files\Siemens\Automation\WinCCUnified\SimaticUA\web.config
```
                <rule name="hasura">
                    <match url="(.*)" />
                    <conditions>
                        <add input="{URL}" pattern="(.*)\/hasura\/(.*)" />
                    </conditions>
                    <action type="Rewrite" url="http://192.168.1.3:8089/v1/graphql" />
                </rule>  
```

## Hasura Docker 
A docker-compose.yml example to start a Hasura instance.   
Adapt the HASURA_GRAPHQL_METADATA_DATABASE_URL!
```
version: '3.6'
services:
  graphql-engine:
    image: hasura/graphql-engine:v2.21.0
    ports:
    - "8089:8080"
    restart: always
    environment:
      ## postgres database to store Hasura metadata
      HASURA_GRAPHQL_METADATA_DATABASE_URL: postgres://system:manager@192.168.1.3:5432/postgres
      ## this env var can be used to add the above postgres database to Hasura as a data source. this can be removed/updated based on your needs
      ## enable the console served by server
      HASURA_GRAPHQL_ENABLE_CONSOLE: "true" # set to "false" to disable console
      ## enable debugging mode. It is recommended to disable this in production
      HASURA_GRAPHQL_DEV_MODE: "true"
      HASURA_GRAPHQL_ENABLED_LOG_TYPES: startup, http-log, webhook-log, websocket-log, query-log
      ## uncomment next line to run console offline (i.e load console assets from server instead of CDN)
      # HASURA_GRAPHQL_CONSOLE_ASSETS_DIR: /srv/console-assets
      ## uncomment next line to set an admin secret
      # HASURA_GRAPHQL_ADMIN_SECRET: myadminsecretkey

```