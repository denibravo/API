# API

## How to run the API for testing

### Installing AWS and SAM
- Install the AWS CLI and configure your credentials. (We did this in class, but if needed, the process is detailed [here](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/prerequisites.html)).
- Install the SAM CLI; instructions are available [here](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html).


### <ins>**How to Set up the Local Development Enviroment**<ins>
1. Download relevant repos:
```
git clone https://github.com/mirrulations/mirrulations-website.git
git clone https://github.com/mirrulations/API.git
```
- Clone the query functions in the API repo by running `git submodule update --init`
    * If you already have an `endpoints/queries` directory, delete that directory before running `git submodule update --init`.
    * Make sure to occasionally rerun `git submodule update` whenever there are changes to the query repository!

2. Install Docker: [Docker](https://www.docker.com/get-started/)    
- allow the default Docker socket to be used (info [here](https://stackoverflow.com/a/77926411)).
- If you're having CORS issues when using local dev environment and the `docker` command in the terminal is not found: 
    - Go to Docker Desktop -> settings -> advanced. Change the docker CLI install location to the other location (i.e. System -> User), click `Apply & restart`, then switch it back and click `Apply & restart` again.
    - This will remake the symlink from docker to CLI
    - We think this was caused by the fix to the docker malware issue earlier in the semester.

3. Install the prerequisites npm libraries
```
npm install dotenv vite
npm install react react-dom react-router-dom
npm install bootstrap
npm install -D @vitejs/plugin-react
```
4. Add a env.json to add all credentials necessary
```
{
    "ApiFunction": {
      "ENVIRONMENT": "local", #specify local if working in local
      "OPENSEARCH_HOST": "opensearch-node1",
      "OPENSEARCH_PORT": "9200",
      "OPENSEARCH_INITIAL_ADMIN_PASSWORD": "<password>",
      "POSTGRES_HOST": "db",
      "POSTGRES_PORT": "5432",
      "POSTGRES_DB": "postgres",
      "POSTGRES_USER": "postgres",
      "POSTGRES_PASSWORD": "password",
      "DB_SECRET_NAME": "mirrulationsdb/postgres/master",
      "OS_SECRET_NAME": "mirrulationsdb/opensearch/master"
    }
  }
```
5. Install the [Data-Product-Kit Repo](https://github.com/mirrulations/Data-Product-Kit) to set up the docker. Follow the `ReadMe.md` up to setting up the installations necessary.`LocalTesting.md` instructions. Start with section **"Clean Reset"** and then move on to the top of the document. 


6. Launch the API, navigate to the API repo and run the following commands
```
sam build && sam local start-api --env-vars env.json --docker-network shared_network
```
**Note:** Make sure that you have the env.json in the root directory as sam would need to pick up the environment variables!

If you make changes to template.yaml, run sam validate to check for errors and sam build to implement the changes.

Take note of your api gateway link for later, you can see it in the output under “Mounting ApiFunction at {GATEWAY_URL_HERE} [GET, OPTIONS]”


7. Launch the Website
- Create a file named “.env”
    - Type “VITE_GATEWAY_API_URL={GATEWAY_URL}”
    - Your Gateway URL is the output from the last step, it might look something like “http://127.0.0.1:3000/dummy”
- Save the file and run this command
```
npm run dev
```

**NOTE:** As of currently you can only look up "National" as that is what is currently in `data-product-kit`'s `query.py`. If you want to change the searchTerm you can do so in the file:

`data-pruduct-kit/queries/query.py`:
```bash
if __name__ == "__main__":
    query_params = {
        "searchTerm": "National", <--- change here
        "pageNumber": 0,
        "refreshResults": True,
        "sessionID": "session1",
        "sortParams": {
            "sortType": "dateModified",
            "desc": True,
        },
        "filterParams": {
            "agencies": [],
            "dateRange": {
                "start": "1970-01-01T00:00:00Z",
                "end": "2025-03-21T00:00:00Z",
            },
            "docketType": "",
        },
    }
```

### How to Launch just the API
- If any changes have been made to `template.yaml`, run `sam validate` to check for errors.
- Make sure to install [Docker](https://www.docker.com/get-started/)    
  - allow the default Docker socket to be used (info [here](https://stackoverflow.com/a/77926411)).
  - If you're having CORS issues when using local dev environment and the `docker` command in the terminal is not found: 
    - Go to Docker Desktop -> settings -> advanced. Change the docker CLI install location to the other location (i.e. System -> User), click `Apply & restart`, then switch it back and click `Apply & restart` again.
  ![docker settings](./documentation/Images/docker-configuration-snapshot.png)
    - This will remake the symlink from docker to CLI
    - We think this was caused by the fix to the docker malware issue earlier in the semester.
- To run locally, use `sam build` and then `sam local start-api`.
- The API should be running at `http://127.0.0.1:3000`. 

### On AWS
#### Automated Deployment
- Ensure the `samconfig.toml` file is correctly configured for automated deployments. This file contains configuration parameters that SAM CLI uses to deploy your application without manual input.
- Run `sam deploy --config-file samconfig.toml --config-env default --resolve-s3` to deploy using the default environment settings.
  - `--config-file` specifies the configuration file to use.
  - `--config-env` specifies the environment within the configuration file to use for deployment parameters.
  - SAM needs a place to put the deployment artifacts. `--resolve-s3` instructs SAM CLI to automatically handle the creation or identification of an S3 bucket for storing the deployment artifacts if not already specified. This ensures that the deployment package has a place to be stored without manual setup.

#### Understanding `samconfig.toml`
The `samconfig.toml` file stores configuration parameters for SAM CLI deployments. Here’s what each field represents:
- `stack_name`: Name of the AWS CloudFormation stack.
- `region`: AWS region where the resources will be deployed.
- `confirm_changeset`: Whether to prompt for confirmation before applying changes (set to `false` for no prompts).
- `capabilities`: Permissions the SAM CLI needs to deploy resources, typically including IAM capabilities.
- `disable_rollback`: Whether AWS CloudFormation should rollback changes if a deployment fails.

#### Endpoints and Sync
- Run `sam list endpoints --output json` to get the link for the endpoint you're testing.
- To update the API on AWS in real-time, use `sam sync --watch`.
    - Note: This may take a couple of minutes to start up.
    - It is ready once you see `Infra sync completed.`

#### Clean Up
- Use `sam delete` when done to avoid leaving resources up, which might incur costs.

## Running Tests
- Make a python virtual environment.
  - `python -m venv .venv`
  - `source ./.venv/bin/activate`
  - `pip install -r requirements.txt`
- cd to endpoints specifically and run `python -m pytest ../tests/handler_test.py`
  - I do not know why the tests don't work in other directories, here's a [stackoverflow link](https://stackoverflow.com/questions/45154583/pytest-running-from-parent-directory) that might have the answers.

This README has been adapted to utilize the automated deployment features of AWS SAM, making it easier and faster to manage your deployments. 

For more detailed information, refer to this [AWS Tutorial](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-getting-started-hello-world.html#serverless-getting-started-hello-world-delete).
