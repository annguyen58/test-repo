### LOCAL DEVELOPMENT (Docker)

1. Install Docker and Docker Compose in your system

If you prefer a visual Docker management tool, you can also install Docker Desktop.

2. Set up your local environment. Required to work correctly inside the IDE

```bash
# Check the Python version in your local environment. The Python version in your local environment should match the version specified in the Dockerfile (We need Python version 3.8 for this project)
python3 --version

# Create python virtual environment in any folder convenient for you. Replace "env" with the name you want to give your virtual environment
python3 -m venv env

# Activate the virtual environment. You can use the following command or use a support tool like Visual Studio Code or PyCharm
source env/bin/activate

# Install the required libraries inside the virtual environment
python3 -m pip install -r requirements.txt

# Install pre-commit hook with
pre-commit install
```

3. Run Airflow inside Docker containers

This project needs to be run with Docker. Please avoid attempting to run the project with a local Python virtual environment, as the Dockerfile already contains the necessary environment and variable configuration commands for the project.

```bash
# Set environment variables for running Airflow inside Docker containers
export AIRFLOW_UID=501  # You can find out your user ID by running: id -u
export AIRFLOW_PROJ_DIR=<your_path_to_local_project>
export GOOGLE_APPLICATION_CREDENTIALS=<path_to_your_google_credentials_file>
# Note: The Google credentials file should have the extension .json.
# To ensure the program runs smoothly, you should run the 'export' command again every time you create a new terminal or reopen the IDE, even when running tests.

# Move to the 'devops' folder
cd devops

# Run once for setup
make setup_dev

# Or run the project regularly (no Docker container settings)
make start

# Check if the program is running in Docker Desktop or use the following command to view running containers
docker ps

# .. do some work with source code

# Stop this project's Docker containers. You can also stop or delete them using Docker Desktop.
make stop
```

4. Login into Airflow UI \
   http://localhost:8080/home

Login: airflow \
Password: airflow

5. Create your own access key

- Login here https://nimbus2-data-pipeline.appspot.com \
  If this is your first time, you need to sign up with your leadplus google account. After that administrators will be able to give you permissions
- Access Site administration and pay attention to these two menu items: "Access tokens" and "Resource access".
- In "Access tokens", create your own access key\
  Make sure to set the limitation (select the "Inactive" radio button) for your access key/token under "All accounts accessible".
- In "Resource access", you will need to associate your access key with the Publisher's resources.
- Copy the access key/token to your local Airflow connection setup at the record where the Connd Id is "http_gateway"

Now you can run pipelines in Airflow. If you see that the pipelines are running successfully, the setup is complete.

##### Docker tips

- If you need to connect to local gateway or any other service
  use `host.docker.internal` as host in Airflow connection settings
- If you restart the program, all your settings on airflow home will be lost. Of course the access key/token will also be lost. So you have to settings it again

### TESTING

- To run tests, once again make sure you have installed the appropriate Python version as noted in the [local development setup](#local-development-docker)
- Also, make sure that your account has been granted access rights by administrator

There are several types of autotests:

| Path                               | Description                                                               |
| ---------------------------------- | ------------------------------------------------------------------------- |
| `unit`                             | unit tests of simple classes, models or DAGs                              |
| `integration/validate_sql_queries` | SQL query validation tests on the BigQuery side                           |
| `integration/aggregation`          | Complex integration tests. They are destructive. Building a test pipeline |

```bash
PYTHONPATH=./dags pytest tests/unit
PYTHONPATH=./dags pytest tests/integration/validate_sql_queries
```

**WARNING!** These are destructive tests. It creates and deletes datastores.
Please use a restricted account for the test project only

For example
this [Service Account](https://console.cloud.google.com/iam-admin/serviceaccounts/details/115850219784663299931?authuser=1&project=airflow-test-101)

```bash
export GOOGLE_APPLICATION_CREDENTIALS=<path_to_google_cloud_access_key>
PYTHONPATH=./dags pytest tests/integration/aggregation
```
