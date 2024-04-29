---
title: Integration testing AWS services with Localstack
date: 2024-04-30 10:42:00 +0100
tags: [aws, testing, refactoring]
description: Using Localstack to write integration tests that depend on AWS services.
---

I recently started working in a new project at work, an since this is a brownfield software, I started adding tests both as a way to getting to know the code, but also creating a safety net for possible refactoring work later on.

One of the components of this software, has quite a lot of external dependencies, the majority being AWS services such as SQS queues, S3 buckets, etc. The code for this particular component was was written in an extremely linear fashion and is quite readable, but unfortunately lacked tests. Adding unit tests to the main classes in this component would be quite tricky without major refactorings due to the depencies being implicitly used inside the class methods. And since the initial rationale to add tests in the first place was to gain confidence in being able to change the code without breaking things, this because a "chicken and egg" problem.

So I decided to take a stab at writting some integration tests and after some research I found [Localstack](https://www.localstack.cloud). Localstack works by simulating AWS services locally. All we need is a simple docker-compose file for convenience where we run a Localstack container exposing a public API, where we can configure our AWS SDK library to point to. Then we can create and manage AWS resources as we would normally. The only downside of Localstack is that some AWS services are not available in the free tier of this tool. Luckily for our team, we didn't depend on any paid services.

We can create a `docker-compose.yaml` file running `localstack`:

```yaml
version: "3.7"

services:
  localstack:
    container_name: localstack_poc
    image: localstack/localstack
    ports:
      - "127.0.0.1:4566:4566"            # LocalStack Gateway
      - "127.0.0.1:4510-4559:4510-4559"  # external services port range
    environment:
      - DEBUG=1
      - SERVICES=s3 # Only start up S3
      - DOCKER_HOST=unix:///var/run/docker.sock
    volumes:
      - "${LOCALSTACK_VOLUME_DIR:-./volume}:/var/lib/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
```

And then we configure our AWS client to use the local endpoint like this:

```js
const AWS = require('aws-sdk');

const s3 = new AWS.S3({
  accessKeyId: 'dummy',
  secretAccessKey: 'dummy',
  region: 'us-west-1',
  endpoint: new AWS.Endpoint('http://localhost:4566'),
  s3ForcePathStyle: true
});

async function createBucket(bucketName) {
  const params = {
    Bucket: bucketName,
    CreateBucketConfiguration: {
     LocationConstraint: 'us-west-1'
    }
  };

  try {
    const data = await s3.createBucket(params).promise();
    return data;
  } catch (error) {
    console.log(`Failed to create bucket "${bucketName}".`, error);
    throw error;
  }
}
```

We can now write integration tests that create AWS resources without changing the parent code under testing. You can run this docker compose file in your CI pipeline or alternatively use [testcontainers](https://node.testcontainers.org/modules/localstack/) to launch Localstack containers on the fly when running your test suite.

Take a look at a PoC repo I created to test drive Localstack: [protectyrjewels/localstack-poc](https://github.com/protectyrjewels/localstack-poc)
