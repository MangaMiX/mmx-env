# mmx-env

### Initial setup

Clone this repository onto the Docker host that will run the stack, then start the Elasticsearch service locally using
Docker Compose:

```console
$ docker-compose up -d elasticsearch
```

We will start the rest of the Elastic components _after_ completing the initial setup described in this section. These
steps only need to be performed _once_.

**:warning: Starting with Elastic v8.0.0, it is no longer possible to run Kibana using the bootstraped privileged
`elastic` user. If you are starting the stack for the very first time, you MUST initialize a password for the [built-in
`kibana_system` user][builtin-users] to be able to start and access Kibana. Please read the section below attentively.**

#### Setting up user authentication

*:information_source: Refer to [Security settings in Elasticsearch][es-security] to disable authentication.*

The stack is pre-configured with the following **privileged** bootstrap user:

* user: *elastic*
* password: *changeme*

For increased security, we will reset this bootstrap password, and generate a set of passwords to be used by
unprivileged [built-in users][builtin-users] within components of the Elastic stack.

1. Initialize passwords for built-in users

    The commands below generate random passwords for the `elastic` and `kibana_system` users. Take note of them.

    ```console
    $ docker-compose exec -T elasticsearch bin/elasticsearch-reset-password --batch --user elastic
    ```

    ```console
    $ docker-compose exec -T elasticsearch bin/elasticsearch-reset-password --batch --user kibana_system
    ```

    If the need for it arises (e.g. if you want to [collect monitoring information][ls-monitoring] through Beats and
    other components), feel free to repeat this operation at any time for the rest of the [built-in
    users][builtin-users].

1. Replace usernames and passwords in configuration files

    Replace the password of the `kibana_system` user inside the Kibana configuration file (`kibana/config/kibana.yml`)
    with the password generated in the previous step.

    Replace the password of the `elastic` user inside the Logstash pipeline file (`logstash/pipeline/logstash.conf`)
    with the password generated in the previous step.

    *:information_source: Do not use the `logstash_system` user inside the Logstash **pipeline** file, it does not have
    sufficient permissions to create indices. Follow the instructions at [Configuring Security in Logstash][ls-security]
    to create a user with suitable roles.*

    See also the [Configuration](#configuration) section below.

1. Unset the bootstrap password (_optional_)

    Remove the `ELASTIC_PASSWORD` environment variable from the `elasticsearch` service inside the Compose file
    (`docker-compose.yml`). It is only used to initialize the keystore during the initial startup of Elasticsearch, and
    is ignored on subsequent runs.

1. Start Kibana and Logstash

    ```console
    $ docker-compose up -d
    ```

    The `-d` flag runs all services in the background (detached mode).

    On subsequent runs of the Elastic stack, it is sufficient to execute the above command in order to start all
    components.

    *:information_source: Learn more about the security of the Elastic stack at [Secure the Elastic
    Stack][sec-cluster].*

#### Injecting data

Give Kibana about a minute to initialize, then access the Kibana web UI by opening <http://localhost:5601> in a web
browser and use the following credentials to log in:

* user: *elastic*
* password: *\<your generated elastic password>*

Now that the stack is running, you can go ahead and inject some log entries. The shipped Logstash configuration allows
you to send content via TCP:

```console
# Using BSD netcat (Debian, Ubuntu, MacOS system, ...)
$ cat /path/to/logfile.log | nc -q0 localhost 5000
```

```console
# Using GNU netcat (CentOS, Fedora, MacOS Homebrew, ...)
$ cat /path/to/logfile.log | nc -c localhost 5000
```

You can also load the sample data provided by your Kibana installation.

### Cleanup

Elasticsearch data is persisted inside a volume by default.

In order to entirely shutdown the stack and remove all persisted data, use the following Docker Compose command:

```console
$ docker-compose down -v
```