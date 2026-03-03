# BioBB Apache Airflow implementation

<div align="center"><a href="https://airflow.apache.org/"><img src="airflow.png" height="80"></a><a href="https://mmb.irbbarcelona.org/biobb/"><img src="biobb.png"></a></div>

This repo hosts a Docker Compose implementation for Apache Airflow.

## Prepare configuration files

### docker-compose.yml

The [**docker-compose.yaml**](docker-compose.yaml) is the file that specifies what **images** are required, what **ports** they need to expose, whether they have access to the host **filesystem**, what **commands** should be run when they start up, and so on.

### airflow.cfg

The [**airflow.cfg**](./config/airflow.cfg) file is the main Airflow configuration file. It controls every aspect of the Airflow installation, organized in sections like `[core]`, `[database]`, `[celery]`, `[scheduler]` and so on.

### .env file

⚠️ No sensible default value is provided for any of these fields, they **need to be defined** ⚠️

An `.env` file must be created in the project folder. The file [`.env.git`](./.env.git) can be taken as an example. The file must contain the following environment variables:

| key              | value   | description                                     |
| ---------------- | ------- | ----------------------------------------------- |
|AIRFLOW_IMAGE_NAME        | string  | Image version used for Apache Airflow                               |
|AIRFLOW_UID        | number  | Airflow user identifier                              |
|AIRFLOW_PROJ_DIR        | string  | Absolute path to this Apache Airflow implementation                               |

## Utils

### airflow_cwl_utils.py

The [**airflow_cwl_utils.py**](./dags/airflow_cwl_utils.py) file is a utility module shared across all DAGs in the Apache Airflow setup. It acts as the **bridge between Airflow and CWL (Common Workflow Language)**. It has two responsibilities:

* **resolve_inputs():** Reads a step's input YAML file and resolves references before execution.
* **create_bash_command():** Builds the shell command string that Airflow's BashOperator will execute for each step.

## Plugins

### cwl_run.sh

The [**cwl_run.sh**](./plugins/cwl_run.sh) is the shell script that **actually executes a single CWL workflow step**. It's called by every BashOperator task via _create_bash_command()_ in [**airflow_cwl_utils.py**](./dags/airflow_cwl_utils.py).

### docker_wrapper.sh

Why [**docker_wrapper.sh**](./plugins/docker_wrapper.sh)? Because **cwltool** normally calls `docker run` directly, but the apache worker is itself a container. The wrapper remaps paths from the container's `/opt/airflow/...` namespace to the host's real paths, so **Docker-in-Docker** mounts work correctly.

## CWL Airflow

The [**x-airflow-common**](./docker-compose.yaml#L47) is a **YAML anchor** — a reusable configuration block that avoids repeating the same settings across every **Airflow service**. In this implementation it has been **customized** for installing **cwltool**, needed for executing the **BioExcel Building Blocks** on Apache Airflow.

The [**CWL Airflow Dockerfile**](./cwl-airflow/Dockerfile) installs **cwltool** in each of the **Apache Airflow Services** automatically.

## Build services

First off, go to the project [**root**](./) folder.

For building the services via **Docker Compose**, please execute the following instruction:

```sh
docker compose build
```

Deploy services:

```sh
docker compose up -d
```

Lists the containers:

```sh
$ docker ps -a
CONTAINER ID   IMAGE                          COMMAND                  CREATED        STATUS                     PORTS                    NAMES
<ID>           docker-airflow-worker          "/usr/bin/dumb-init …"   16 hours ago   Up 16 hours (healthy)      8080/tcp                 <NAME>
<ID>           docker-airflow-apiserver       "/usr/bin/dumb-init …"   16 hours ago   Up 16 hours (healthy)      0.0.0.0:8080->8080/tcp   <NAME>
<ID>           docker-airflow-triggerer       "/usr/bin/dumb-init …"   16 hours ago   Up 16 hours (healthy)      8080/tcp                 <NAME>
<ID>           docker-airflow-scheduler       "/usr/bin/dumb-init …"   16 hours ago   Up 16 hours (healthy)      8080/tcp                 <NAME>
<ID>           docker-airflow-dag-processor   "/usr/bin/dumb-init …"   16 hours ago   Up 16 hours (healthy)      8080/tcp                 <NAME>
<ID>           docker-airflow-init            "/bin/bash -c 'if [[…"   16 hours ago   Exited (0) 16 hours ago                             <NAME>
<ID>           postgres:16                    "docker-entrypoint.s…"   16 hours ago   Up 16 hours (healthy)      5432/tcp                 <NAME>
<ID>           nginx:alpine                   "/docker-entrypoint.…"   16 hours ago   Up 16 hours                0.0.0.0:8888->80/tcp     <NAME>
<ID>           redis:7.2-bookworm             "docker-entrypoint.s…"   16 hours ago   Up 16 hours (healthy)      6379/tcp                 <NAME>
```

## Execute services

### Apache Airflow

Open a browser and type:

```
http://localhost:8080/
```

### Outputs server

Apache Airflow **doesn't serve output files** because it was designed as a **workflow orchestrator**, not a data platform. Its core philosophy is:

_"Airflow schedules and monitors tasks. What those tasks do with data is not Airflow's concern."_

So, in this implementation, an **Nginx server** for accessing the **outputs via web** is provided.

Once a workflow has run, open a workflow and type:

```
http://localhost:8888/<WF NAME>/outputs/<STEP>/<FILE NAME>
```

## Copyright & Licensing

This software has been developed in the [MMB group](http://mmb.irbbarcelona.org) at the [IRB](https://www.irbbarcelona.org/) for the [European BioExcel](http://bioexcel.eu/), funded by the European Commission (EU Horizon Europe [101093290](https://cordis.europa.eu/project/id/101093290), EU H2020 [823830](http://cordis.europa.eu/projects/823830), EU H2020 [675728](http://cordis.europa.eu/projects/675728)).

* (c) 2015-2026 [Institute for Research in Biomedicine](https://www.irbbarcelona.org/)

Licensed under the
[Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0), see the file LICENSE for details.

![](https://bioexcel.eu/wp-content/uploads/2019/04/Bioexcell_logo_1080px_transp.png "Bioexcel")
