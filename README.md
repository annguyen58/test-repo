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
# The Google credentials file should have the extension .json.
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

4. Login into Airflow UI

   http://localhost:8080/home\
   Login: airflow \
   Password: airflow

5. Setting up your access key and configure gateway

- Step 1 : Login to the Nimbus2 Data Pipeline\
  Go to https://nimbus2-data-pipeline.appspot.com and if it's your first time, sign up using your leadplus Google account. After that administrators will be able to give you permissions.
- Step 2 : Access Site Administration and explore menu items\
  Navigate to the Site Administration section and focus on the `Access tokens` and `Resource access` menu items.
- Step 3 : Create your own access key\
  In `Access tokens`, generate your own access key. Remember to set the limitation by selecting the `Inactive` radio button for your access key/token under `All accounts accessible`.
- Step 4 : Associate your access key with Publisher's Resources\
  In `Resource access`, you will need to associate your access key/token with the Publisher's resources. `Resource access` is about determining specific resources (endpoints or data) within the production API that your application or system needs to access to perform its tasks. Associating your access key/token with these resources helps limit the number of API requests from the development environment. Keeping the API requests low in the development environment is crucial to adhere to the request limits of the external API and avoid any disruptions to the provider's system or services.
- Step 5 : Configure access key for Gateway\
  The Gateway serves as a local proxy for all external APIs, enhancing security and controlling API requests from the development environment.\
  Copy the access key/token you generated in Step 3. Then configure it for the local Airflow connection setup at the record where the `Connd Id` is `http_gateway`. This allows your Gateway to authenticate and interact with external APIs on behalf of your application while staying within defined access limits.

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
