containerspec
==============

Containerspec lets you write and run specifications about docker [1] images and docker containers, using
cucumber [2] and serverspec [3]. It is available as a docker image itself, so you don't have to install ruby or
other dependencies to make it work.

**Example**

Pull our cucumber-enable spec image , pull a sample image and write a spec for that image:
```bash
$ docker pull dockerfile/rethinkdb
$ docker pull dewiring/spec_cucumber

$ mkdir spec
$ cd spec

~/spec$ cat >f1.feature <<EOF
Feature: I need my images

Scenario:  RethinkDB spec images
 Given i pull 'dockerfile/rethinkdb'
 When there are images tagged 'dockerfile/rethinkdb:latest'
 Then 'Checksum' should be set
 Then within Config, 'ExposedPorts' should be like '8080/tcp'
EOF
```

Start a `dewiring/spec_cucumber` container, allowing read-only access to docker socket of the host, and inject current directory as /spec volume:

```bash
~/spec$ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock:ro -v `pwd`:/spec dewiring/spec_cucumber --color

Feature: I need my images

Scenario: RethinkDB spec images                              # features/f1.feature:3
Given i pull 'dockerfile/rethinkdb'                          # /project_step_definitions/image_definitions.rb:43
When there are images tagged 'dockerfile/rethinkdb:latest'   # /project_step_definitions/image_definitions.rb:78
Then 'Checksum' should be set                                # /project_step_definitions/image_definitions.rb:122
Then within Config, 'ExposedPorts' should be like '8080/tcp' # /project_step_definitions/image_definitions.rb:138

1 scenario (1 passed)
4 steps (4 passed)
0m6.020s
```

Now given that image, instantiate a container with some parameters, write a spec for it:

```bash
~/spec$ docker run -d -P --name "db" dockerfile/rethinkdb

~/spec$ cat >f2.feature <<EOF
Feature: I need a running db

Scenario: RethinkDB container spec
 When there is a running container named 'db'
 Then it should run on image 'dockerfile/rethinkdb'
 Then container should expose port '8080'
 And container volume '/data' should be mounted
 And within container Config, 'Cmd' should be like 'rethinkdb'

Scenario: Not privileged, no ssh port
 When there is a running container named 'db'
 Then container should not expose port '22'
 And it should not run privileged
EOF
```

and run it again:

```bash
~/spec$ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock:ro -v `pwd`:/spec dewiring/spec_cucumber --color
Feature: I need my images

(...)

Feature: I need a running db

Scenario: RethinkDB container spec                              # features/f2.feature:3
When there is a running container named 'db'                  # /project_step_definitions/container_definitions.rb:29
Then it should run on image 'dockerfile/rethinkdb'            # /project_step_definitions/container_definitions.rb:240
Then container should expose port '8080'                      # /project_step_definitions/container_definitions.rb:203
And container volume '/data' should be mounted                # /project_step_definitions/container_definitions.rb:167
And within container Config, 'Cmd' should be like 'rethinkdb' # /project_step_definitions/container_definitions.rb:93

Scenario: Not privileged, no ssh port        # features/f2.feature:10
When there is a running container named 'db' # /project_step_definitions/container_definitions.rb:29
Then container should not expose port '22'   # /project_step_definitions/container_definitions.rb:230
And it should not run privileged             # /project_step_definitions/container_definitions.rb:117

3 scenarios (3 passed)
12 steps (12 passed)
0m4.074s
```

Find a **more detailed example** in [tests/examples-tests](https://github.com/de-wiring/spec-dockerbox/tree/master/tests/example-tests).

`Vagrantfile` will set up a vm where an automated build test can be run.

References
----------
- [1] [www.docker.com](www.docker.com)
- [2] [Cucumber - cukes.info](cukes.info)
- [3] [www.serverspec.org](www.serverspec.org)


License
-------

The MIT License (MIT) Copyright (c) 2014 Andreas Schmidt
