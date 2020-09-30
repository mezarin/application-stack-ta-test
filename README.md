# Transformation Advisor And OL Application Stack Integration

## Inner Loop For Binary Applications

### Java 8

A custom devfile that uses an Open Liberty image using Java 8 is required. 

1. Create the custom component:

`odo create odota --devfile ./dev/java8/devfile.yaml`

2. Push the created component:

`odo push`

3. Access the resorts application: 

`odo url list`
```
NAME     STATE      URL                            PORT     SECURE     KIND
ep1      Pushed     http://ep1-odota-route-url     9080     false      route
```

Use this URL: http://ep1-odota-route-url/resorts


### Java 11:

1. Create a new java-openliberty component:

`odo create java-openliberty odota`

2. Push the created component:

`odo push`

3. Access the resorts application:

`odo url list`
```
NAME     STATE      URL                            PORT     SECURE     KIND
ep1      Pushed     http://ep1-odota-route-url     9080     false      route
```

Use this URL: http://ep1-odota-route-url/resorts


## Outter Loop For Binary Applications:

### Pre-reqs:

- Install the Open Liberty operator.
- Build/use the odo client version containing the odo deploy implementation. Currently you need to build your own
bt cloning the deploy_command branch from here: https://github.com/EnriqueL8/odo.git and building it using `make goget-tools`. Note: You may need to use this command also: `go mod vendor`

### Java 8

Given the current state of the odo deploy command a number of workarounds are needed right now:

1. Create the custom component:

`odo create odota --devfile ./dev/java8/devfile.yaml`

2. Create a URL and route for the open liberty application to be accessed. This step should not be needed:

a. Create the needed port entry in .odo/env/env.yaml for odo deploy to see and replace in the app-deploy.yaml temaplate we supplied in devfile.yaml. Currently this is only possible by running the `odo url create` command:

`odo url create odotaport --port 9080`

**Workaround 1:** Add the port entry manually to .odo/env/env.yaml
```
ComponentSettings:
  Name: odota
  Namespace: service-binding-demo
  Url:
  - Name: odotaport
    Kind: route
    Port: 9080
  AppName: app
```

**Workaround 2:** `odo url create` currently has a side effect where a parent definition is disambiguated/expanded in the devfile. However, the parent definition is still present. It should not be this way, but it is. So, we need to remove the parent section from the devfile.yaml file; otherwise, odo commands will fail because the devfile now has duplicated entries.

Remove this section:

```
parent:
  uri: https://raw.githubusercontent.com/OpenLiberty/application-stack/master/dev/devfile.yaml
  components:
  - name: devruntime
    container:
      endpoints:
      - exposure: public
        path: /
        name: ep1
        targetPort: 9080
        protocol: http
      image: uberskigeek/java-openliberty-java8-odo:0.1.1
```
**Workaround 3:** `odo url create` currently has another side effect where entries such as alpha.build-dockerfile and alpha.deployment-manifest are renamed. Change those back to what they were prior to issuing the `odo url create` command:

  alpha.build-dockerfile: "https://raw.githubusercontent.com/mezarin/application-stack-ta-test/master/dev/java8/Dockerfile"
  
  alpha.deployment-manifest: "https://raw.githubusercontent.com/OpenLiberty/application-stack/master/dev/app-deploy.yaml"

Note that value of `alpha.build-dockerfile` is now a URL that points to a custom Dockerfile located in this repository's `dev` directory.
Technically, we could avoid specifying `alpha.build-dockerfile` by simply placing the desired Dockerfile along side devfile.yaml (base directory); however, there appears to be an odo/other issue at odo docker build time that causes this error: 

`"error: build error: no FROM image in Dockerfile"`

To bypass the error, put the Dockerfile in an accessible location and point to it in the devfile through a url, or use a url pointing to the Dockerfile in this repo (dev directory).

This repo's `dev` directory contains custom devfiles and Dockerfiles for java8 and java11 Open Liberty installs.


b. Create a route and other artifacts needed for inner loop development. Issuing an odo push is not required for outter loop testing, but do it anyway since we also want to enable/test inner loop development.

`odo push`

3. Run odo deploy:

`odo deploy`

This is what the output should look like (roughly):
```
 ✓  Validating arguments [111247ns]
 ✓  Validating build information [250ms]
 ✓  Validating deployment information [401ms]
Building component odota
 ✓  Preparing files for building image [1ms]
 ✓  Started build odota-1 using BuildConfig
 ✓  Waiting for build to complete [3m]
 ✓  Successfully built container image: odota:latest
Deploying component odota
 ✓  Creating resource of kind OpenLibertyApplication [41ms]
 ✓  Determining the application URL [2s]
 ✓  Successfully deployed component: http://odota-deploy-route-url
```

4. Access the resorts application using the route generated on the previous step:
http://odota-deploy-route-url/resorts

### Java 11:

1. Create the custom component:

`odo create java-openliberty odota`

2. Follow the same instructions and workarounds described in the java8 section with the only difference that instead of using the java8 version of the Dockerfile, point devfile.yaml entry `alpha.build-dockerfile` to the java 11 version of the Dockerfile located in the dev directory.

`  alpha.build-dockerfile: https://raw.githubusercontent.com/mezarin/application-stack-ta-test/master/dev/java11/Dockerfile`

3. Run odo deploy:

`odo deploy`

This is what the output should look like (roughly):

```
 ✓  Validating arguments [111247ns]
 ✓  Validating build information [250ms]
 ✓  Validating deployment information [401ms]
Building component odota
 ✓  Preparing files for building image [1ms]
 ✓  Started build odota-1 using BuildConfig
 ✓  Waiting for build to complete [3m]
 ✓  Successfully built container image: odota:latest
Deploying component odota
 ✓  Creating resource of kind OpenLibertyApplication [41ms]
 ✓  Determining the application URL [2s]
 ✓  Successfully deployed component: http://odota-deploy-route-url
```

4. Access the resorts application using the route generated on the previous step:
http://odota-deploy-route-url/resorts