# Testcontainers

> Testcontainers is a NodeJS library that supports tests, providing lightweight, throwaway instances of common databases, Selenium web browsers, or anything else that can run in a Docker container.

[![Build Status](https://travis-ci.org/testcontainers/testcontainers-node.svg?branch=master)](https://travis-ci.org/testcontainers/testcontainers-node)
[![npm version](https://badge.fury.io/js/testcontainers.svg)](https://www.npmjs.com/package/testcontainers)
[![npm version](https://img.shields.io/npm/dt/testcontainers.svg)](https://www.npmjs.com/package/testcontainers)

## Install

```bash
npm i -D testcontainers
```

## Usage

Run your app with the `DEBUG=testcontainers` environment variable set to see debug output.

The following environment variables are supported:

| Key | Example value | Behaviour |
| --- | --- | --- |
| `DOCKER_HOST` | `tcp://docker:2375` | Override the Docker host, useful for DIND in CI environments |


## Examples

Using a pre-built Docker image:

```javascript
const redis = require("async-redis");
const { GenericContainer } = require("testcontainers");

(async () => {
  const container = await new GenericContainer("redis")
    .withExposedPorts(6379)
    .start();
  
  const redisClient = redis.createClient(
    container.getMappedPort(6379),
    container.getContainerIpAddress(),
  );
  await redisClient.quit();

  await container.stop();
})();
```

Building and using your own Docker image:

```javascript
const path = require('path');
const { GenericContainer } = require("testcontainers");

(async () => {
  const buildContext = path.resolve(__dirname, "my-dir");
  
  const container = await GenericContainer.fromDockerfile(buildContext);
  
  const startedContainer = await container
    .withExposedPorts(8080)
    .start();

  await startedContainer.stop();
})();
```

Execute commands inside a running container:

```javascript
const { output, exitCode } = await container.exec(["echo", "hello", "world"]);
```

Creating a container with a `tmpfs` mount:

 ```javascript
const container = await new GenericContainer("postgres")
  .withExposedPorts(5432)
  .withTmpFs({ '/temp_pgdata': 'rw,noexec,nosuid,size=65536k' })
  .start();
 ```

Specifying a timeout to wait for a container to stop. Note that the
default is 10 seconds:

```javascript
const { GenericContainer } = require("testcontainers");
const { Duration, TemporalUnit } = require("node-duration");

const container = await new GenericContainer("postgres")
  .withExposedPorts(5432)
  .start();

await container.stop({ 
  timeout: new Duration(10, TemporalUnit.SECONDS) 
})
 ```

By default, Testcontainers will remove any associated volumes created
by the container when it is stopped. You can override this behaviour:

 ```javascript
const container = await new GenericContainer("postgres")
  .withExposedPorts(5432)
  .start();

await container.stop({ 
  removeVolumes: false
})
 ```

## Wait Strategies

Ordinarily Testcontainers will wait for up to 60 seconds for the container's mapped network ports to start listening. 
If the default 60s timeout is not sufficient, it can be altered with the `withStartupTimeout()` method:

```javascript
const { GenericContainer } = require("testcontainers");
const { Duration, TemporalUnit } = require("node-duration");

const container = await new GenericContainer("redis")
  .withExposedPorts(6379)
  .withStartupTimeout(new Duration(100, TemporalUnit.SECONDS))
  .start();
```

### Log output Wait Strategy

In some situations a container's log output is a simple way to determine if it is ready or not. For example, we can 
wait for a `Ready` message in the container's logs as follows:

```javascript
const container = await new GenericContainer("redis")
  .withExposedPorts(6379)
  .withWaitStrategy(Wait.forLogMessage("Ready to accept connections"))
  .start();
```
