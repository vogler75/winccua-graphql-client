# GraphQL Client for SIEMENS WinCC Unified

It is a Custom Web Control to get access to GraphQL servers.

You have to download the webcc.min.js from SIEMENS and save it to the control/js directory ! You can get it out of a Custom Web Control example from [here](https://support.industry.siemens.com/cs/document/109779176/benutzerdefinierte-controls-in-wincc-unified-einbinden-(custom-web-controls)?dti=0&lc=de-WW) 

Then you can create a Custom-Web.Control zip file from this directory:   
> cd winccua-mqtt-client\Custom-Web-Control\{D52DB1F6-58DC-449E-85A8-317C1A025951}

> "C:\Program Files\7-Zip\7z.exe" a -tzip "..\{D52DB1F6-58DC-449E-85A8-317C1A025951}.zip" *

Copy the {D52DB1F6-58DC-449E-85A8-317C1A025951}.zip file to your project into the UserFiles\CustomControls directory.

In the "GraphQL-Client-Demo" example [Hasura.io](https://hasura.io) is used to get access to a PostgreSQL database.  

In the second screen we send questions to OpenAI ChatGPT and display the result. The Custom Web Control can also be used for REST API's with Content-Type: "application/json". Instead of setting the "Query" property you have to set your request body at the "Content" property. 

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

## ChatGPT Demo
You have to add a rule to your IIS configuration, to forward requests to OpenAI and you have to set your OpenAI API-Key into the input field in the screen. 
```
                <rule name="chatgpt">
                    <match url="(.*)" />
                    <conditions>
                        <add input="{URL}" pattern="(.*)\/chatgpt\/(.*)" />
                    </conditions>
                    <action type="Rewrite" url="https://api.openai.com/v1/chat/completions" />
                </rule>       
```