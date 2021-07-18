Angles is a centralised test automation dashboard where you can store your automated test results and screenshots using a clearly defined [API](https://editor.swagger.io/?url=https://raw.githubusercontent.com/AnglesHQ/angles/master/swagger/swagger.json). By using this API, you are no longer limited to a single framework or programming language and can use the same dashboard for all your test automation frameworks.
Some features already provided by Angles are:
- Visual comparison (against a configurable baseline)
- Setting ignore boxes for the visual comparisons (areas to exclude in the compare)
- Storing additional details as part of the test runs such as: 
    - team that ran the test
    - environment it was run on
    - artefact versions for system under test.
    - platform details (e.g., what platform the test has been executed against)
- Automatic cleanup of builds and screenshots after 90 days (configurable) 
- Test Case history (look at a single test case over time)
- Test Matrix (multiple test run comparison)

## Overview
The Angles dashboard consists of 3 containers:
- a front-end ([angles-ui](https://hub.docker.com/u/angleshq/angles-ui))
- a back-end ([angles](https://hub.docker.com/u/angleshq/angles))
- a Mongo container (to store the results)

### The back-end
The back-end for Angles provides an API to create, retrieve, update and remove your automation test data. 
It has been [configured](https://github.com/AnglesHQ/angles/blob/master/config/database.config.js) to use a local instance of mongo which is set-up using docker-compose. However, you can modify that config file and point it to another mongo instance (as long as it's setup with the right credentials and collections using the [mongo-init.js](https://github.com/AnglesHQ/angles/blob/master/setup/mongo-init.js))

If you want to run the Angles back-end locally with debug logging turned on, just navigate to the Angles folder in the terminal and run the following command.
``` shellscript
# clone the back-end repo, modify the database.config.js file and run:
DEBUG=*:controller node server.js | ./node_modules/.bin/pino-pretty
```
**NOTE:** The code for the angles back-end can be found ([here](https://github.com/AnglesHQ/angles)).

### The front-end (UI)
The Angles front-end uses the Angles API (provided by the back-end) to retrieve and display the stored test data and screenshots. To do this, it makes use of the [angles-javascript-client](https://github.com/AnglesHQ/angles-javascript-client) (the same client used for javascript tests).

To run the Angles front-end locally you can run the following command in the terminal. Make sure you provide the entire url for the api (e.g. http://127.0.0.1:3000).
```shellscript
# clone the front-end repo and run:
PORT=3001 REACT_APP_ANGLES_API_URL=<angles_api_url> npm start
```
**NOTE:** The code for the angles the frontend can be found ([here](https://github.com/AnglesHQ/angles-ui))

### Setting up Angles (with Docker-Compose)
To set up your own instance of the Angles dashboard you can use the [docker compose](https://github.com/AnglesHQ/angles/blob/master/setup/docker-compose.yml) file and [Docker-compose](https://docs.docker.com/compose/).
You'll need to clone the angles project and then using a terminal navigate to the *"setup"* which contains the docker-compose.yml file and the mongo-init.js file. The mongo-int.js file will setup the necessary database collections and indexes.

If you're not running Angles locally (e.g., 127.0.0.1), you should also change the environment variable "REACT_APP_ANGLES_API_URL" in the docker-compose file to point to the url where the angles API is accessible (e.g. domain name or external ip address).

```shellscript
# run in same directory as docker-compose file
docker-compose pull && docker-compose -f docker-compose.yml up -d 
```

### Tearing down Angles
If you would like to tear down the containers, you can run the following command in the directory with the docker-compose.yml file.

```shellscript
# and to tear it down
docker-compose down
```
**NOTE**: Angles creates volumes to store persistent data (e.g., database config and records), and these will remain even after running the command above. If you wanted to remove this as well you would have to do manually.

### Angles API
Once Angles is running, you can access the documentation by navigating to the following url http://<angles-server-ip:3000/api-docs.

Or if you just want to have a look at the api documentation it can be found in the [swagger.json](https://github.com/AnglesHQ/angles/blob/master/swagger/swagger.json) which you can then load in the [swagger editor](https://editor.swagger.io/?url=https://raw.githubusercontent.com/AnglesHQ/angles/master/swagger/swagger.json).

To use the API to store and retrieve your automated tests results you can use either the [Java Client](https://github.com/AnglesHQ/angles-java-client) or the [JavaScript Client](https://github.com/AnglesHQ/angles-javascript-client).

### Using the API
Before you can store any test results in the Angles dashboard you will need to set up:
- a team (and it's component(s))
- an environment. 

**NOTE:** Your team, component and environment have to exist before you can start adding builds for the relevant team (otherwise angles will return an error when trying to store the results). 

If you would like to use the [TestNG-Example Project](https://github.com/AnglesHQ/testng-example) setup the example team, component and environment mentioned below using the angles-api which is accessible via the following url when you setup the containers locally: [http://127.0.0.1:3000/api-docs](http://127.0.0.1:3000/api-docs).

- Setup a team (using the API)

``` json
{
  "name": "angles",
  "components": [
    {
      "name": "example"
    }
  ]
}
```
- Setup an environment (using the API)

``` json
{
  "name": "qa"
}
```

Please refer to the Swagger endpoint once your instance is up and running and click the "Try it out" button on the create team endpoint. Once you fill in the details and click execute, the team will be added. Repeat these steps for the environment.

### Angles Cleanup (Cron)
As you can imagine, storing all the test data (and screenshots) without any form of cleanup can mean your instance of Angles can fill up quite quickly.
Therefore, Angles has a nightly cron that will use the build delete API to remove any builds and screenshots **older than 90 days**. If you would like to change this number please BUILD_CLEAN_UP_AGE_IN_DAYS value in the docker-compose file and run the docker-compose command again. 
If you want to keep specific builds there is a "keep" flag which can be set. With this flag set, those builds will not be removed as part of the nightly run (once they hit the configured age). 

**NOTE** The cleanup will not remove any builds that contain a "baseline" image (to ensure these can still be used for comparison).
