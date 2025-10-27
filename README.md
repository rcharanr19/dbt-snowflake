# dbt-snowflake
Project to setup Native DBT Project Object in Snowflake

## Steps to setup Native DBT Project Object on Snowflake

1. **Create dbt_network_rule**  
   List URLs Snowflake can access for DBT packages.

2. **Create EXTERNAL ACCESS INTEGRATION**  
   For dbt access to external dbt package locations:
   ```
   CREATE OR REPLACE EXTERNAL ACCESS INTEGRATION dbt_ext_access
     ALLOWED_NETWORK_RULES = (dbt_network_rule)
     ENABLED = TRUE;
   ```

3. **Create a new schema `DBT_PROJECT`**  
   In your Snowflake database to host Native DBT Project Object.

4. **Add `profiles.yml`**  
   Ensure `profiles.yml` is present in the DBT Project.

5. **Create `dbt-snowflake.yml` to deploy from Github to Snowflake**
   - i. Install latest version of Python (>3.10)
   - ii. Install Snowflake CLI and dbt-snowflake (`snowflake-cli==3.9.* dbt-snowflake==1.10.*`)
   - iii. Run dbt deps:
     ```
     python -m pip install --upgrade pip
     pip install snowflake-cli==3.9.0 dbt-snowflake==1.10.*
     dbt deps
     snow --version
     ```
   - iv. Zip the DBT package.
   - v. Deploy to Snowflake using:
     ```
     snow dbt deploy $(DBT_PROJECT_NAME) \
       --source . \
       --profiles-dir . \
       --database "$(SF_ANALYTICS_CLOUDLINK_DATABASE)" \
       --schema "$(SF_ANALYTICS_CLOUDLINK_DBT_PROJECT_SCHEMA)" \
       --role "$(APP_DATA_TRANSFORM_USER_ROLE)" \
       --warehouse "$(APP_DATA_TRANSFORM_USER_WAREHOUSE)" \
       --account "$(SF_ACCOUNT)" \
       --user "$(APP_DATA_TRANSFORM_USER)" \
       --authenticator SNOWFLAKE_JWT \
       --private-key-path "decrypted_rsa_key.p8" \
       -x
     ```
   - vi. Grant users access to Native DBT Project Object in Snowflake:
     ```
     snow sql -q "GRANT USAGE ON DBT PROJECT $(DBT_PROJECT_NAME) TO ROLE AZ_GRP_SF_DEVELOPER_ROLE;" \
       --database "$(SF_ANALYTICS_CLOUDLINK_DATABASE)" \
       --schema "$(SF_ANALYTICS_CLOUDLINK_DBT_PROJECT_SCHEMA)" \
       --role "$(APP_DATA_TRANSFORM_USER_ROLE)" \
       --warehouse "$(APP_DATA_TRANSFORM_USER_WAREHOUSE)" \
       --account "$(SF_ACCOUNT)" \
       --user "$(APP_DATA_TRANSFORM_USER)" \
       --authenticator SNOWFLAKE_JWT \
       --private-key-path "decrypted_rsa_key.p8" \
       -x
     ```

## Steps to Create DBT workspace in Snowflake

Creating a dbt workspace in Snowflake, especially utilizing the native dbt Projects on Snowflake feature, involves several steps:

1. **Launch Workspaces in Snowflake**  
   - Navigate to the "Workspaces" section within your Snowflake environment (typically found on the left-hand menu).
   - Click on "+ New" to create a new workspace.
   - Provide a name for your workspace and confirm its creation.

2. **Add a dbt Project to the Workspace**  
   - Within your newly created workspace, click "+ Add New" and select "dbt Project."
   - Enter the Project Name, and specify the Snowflake Role, Warehouse, Database, and Schema that dbt will use for development and materialization. This database and schema will be the target for your dbt models and tests.

3. **Configure dbt Project Details**  
   - Snowflake will automatically create the necessary dbt project structure, including folders and files like `dbt_project.yml` and `profiles.yml`.
   - The `profiles.yml` file will contain the connection details (role, warehouse, database, schema) you provided in the previous step.

4. **Connect to a Git Repository (Optional but Recommended)**  
   For version control and collaborative development, link your workspace to a Git repository (e.g., GitHub, GitLab). This allows you to clone your dbt project into the workspace and push changes.
   - i. Create a Secret in Snowflake to use in API INTEGRATION to connect to Azure ADO:
     ```
     CREATE OR REPLACE SECRET my_api_key_secret
     TYPE = GENERIC_STRING
     SECRET_STRING = 'replace-with-ADO-PAT'
     COMMENT = 'Secret for authenticating with External API';
     ```
   - ii. Create API INTEGRATION:
     ```
     CREATE OR REPLACE API INTEGRATION cloudlink_dbt_ado_api_integration
     API_PROVIDER = git_https_api
     API_ALLOWED_PREFIXES = ('https://TexasFirstRentals@dev.azure.com/TexasFirstRentals/Data%20Platform')
     -- Comment out the following line if your forked repository is public
     ALLOWED_AUTHENTICATION_SECRETS = (my_api_key_secret)
     ENABLED = TRUE;
     ```

5. **Run dbt Commands**  
   Within the workspace's terminal or UI, you can now execute standard dbt commands such as:
   - `dbt deps`: To install project dependencies.
   - `dbt run`: To execute your dbt models and materialize them in Snowflake.
   - `dbt test`: To run tests defined in your dbt project.

6. **Review Results**  
   Monitor the logs and review the results of your dbt runs and tests directly within the workspace interface.
