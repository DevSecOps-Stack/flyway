Redgate Flyway
	Redgate Flyway is an open-source database migration tool that simplifies and automates the process of managing and versioning database changes. It allows developers and DBAs to apply version control practices to their databases, enabling easier collaboration, tracking, and management of database schema changes over time. With Flyway, users can define their database changes as code, apply those changes consistently across different environments, and roll back changes when needed. It supports various database platforms, providing a unified approach to database versioning and migration regardless of the underlying technology.
Prerequisites
	Have a GitHub account.  There is a free edition or a free trial for paid tiers.
	Install the following on your developer machine:
Flyway Enterprise - Desktop - There is a free 28-day trial.
Git, if you don’t already have it
	Permission to create some SQL Server databases.
Ideally, this is on a local SQL Server instance that is accessible at localhost:1433. If not, that is ok.  You will be prompted to change your project's connection strings, which we explain below. 
Getting started
Deploy the SQL server, in my case I have deployed postgres sql onto azure.

 

Make sure to open firewall rules from your build server and also well where ever you are using flyway from
 

Now clone the git repo to the local machine and below is the files related to the repo
 
Adjust the configurations in the flyway.toml file to suit your specific database needs. For instance, when using Azure PostgreSQL, I made the following modifications to the configurations
Flyway.toml file
id = "testid"  # Unique identifier for your Flyway project
name = "testproject"  # Name of your Flyway project
databaseType = "PostgreSql"  # Database type being used

[environments.postgres]
url = "jdbc:postgresql://redgate-db.postgres.database.azure.com:5432/postgres"
user = "pdadmin"
password = "Python@01"
schemas = ["public"]
displayName = "Azure PostgreSQL Database"
provisioner = "clean"

[flyway]
locations = ["filesystem:migrations"]

Once after making the changes to the above file just fire up flyway desktop
Click the Open project… drop down > Open from version control.  Paste the copied url, select an empty folder as the directory, and click Clone
 




From flyway desktop Select click on open project and select flyway.toml file

 

Select target database, If the connections looks good in flyway.toml you get the below page, select the database base which you have provided from flyway.toml as shown below then click on run migrate

 

After successful migration you get the below confirmation but you can also verify under databases

 

 

You can verify from psql command line
Connect to the database from the command line
Command: psql -U pgadmin -d postgres -h redgate.postgres.database.azure.com
 

Configure azure Ubuntu server with azure agent as well as flyway
Flyway installation
   sudo apt install snapd
   sudo snap install flyway
   flyway -v

rdadmin@redgate-agent:~$ sudo find / -name flyway
/root/snap/flyway
/opt/flyway
/opt/flyway-8.0.1/flyway
/var/snap/flyway
/snap/flyway
/snap/flyway/4/flyway
/snap/bin/flyway
If you want to use the Flyway installed via Snap, you can add the Snap binaries directory to your PATH. Open your .bashrc or .bash_profile file in a text editor:
nano ~/.bashrc
And add this line to the end of the file:
export PATH=$PATH:/snap/bin
Save and close the file. Then, reload your bash profile by closing and reopening your terminal, or by running this command:
source ~/.bashrc
Now, you should be able to run the flyway command without sudo. If you’re still having issues, please let me know!.
7of10

Deploy azure agent
Follow the steps from azure devops agent pool section and deploy the agent make sure to open firewall rule to allow agent to connect to the DB
PAT: twcptuuev7ehcrrtcn47sizywhyigjhtanxnwogqdbktg5efr7sa
Azure git repo 
https://branybots@dev.azure.com/branybots/redgate/_git/redgate
Azure pipeline
Create the azure build pipeline with the below yaml and make sure it is pulling the specified code repo
trigger:
- master

pool:
  name: redgate-agent
  demands:
    - agent.name -equals redgate-agent

stages:
- stage: Migration
  jobs:
  - job: RunFlywayMigration
    displayName: 'Execute Flyway Migration'

    steps:
    - script: |
        # Change directory to the working directory
        cd $(Build.SourcesDirectory)

        # Run Flyway migrate using the specified configuration files
        
        flyway migrate -configFiles="flyway.toml" -locations="filesystem:migrations" -url="jdbc:postgresql://redgate-db.postgres.database.azure.com:5432/postgres" -user=pgadmin -password="Python@0101" -schemas=public
      displayName: 'Execute Flyway Migration'



