+++
title = "Existing A2HA to Automate HA"

draft = false

gh_repo = "automate"
[menu]
  [menu.automate]
    title = "Existing A2HA to Automate HA"
    parent = "automate/deploy_high_availability/migration"
    identifier = "automate/deploy_high_availability/migration/ha_existing_a2ha_to_automate_ha.md Existing A2HA to Automate HA"
    weight = 200
+++

{{< warning >}}

- A2HA user can be migrated to Automate HA with minimum Chef Automate version [20201230192246](https://docs.chef.io/release_notes_automate/#20201230192246)

{{< /warning >}}


This page explains migrating the existing A2HA data to the newly deployed Chef Automate HA. This migration involves the following steps:


## Prerequisites

- Ability to mount the file system, which was mounted to A2HA Cluster for backup purpose, to Automate HA. 

- A2HA is configured to take backup on mounted network drive (location example : `/mnt/automate_backup`).
## Migration

1. Run the following commands from any automate instance in A2HA Cluster.

    ```cmd
    sudo chef-automate backup create
    sudo chef-automate bootstrap bundle create bootstrap.abb
    ```

    - The first command will take the backup at the mount file system. You can get the mount path from the file `/hab/a2_deploy_workspace/a2ha.rb` on bastion node.
    - The second command will create the bootstrap bundle, which we need to copy all the frontend nodes of Automate HA cluster.
    - Once the backup is completed successfully, please save the backup Id. For example: `20210622065515`.
    - If you want to use backup created previously run the command on Automate node, to get the backup id
      `chef-automate backup list`

    ```
    Backup             State       Age
    20180508201548    completed  8 minutes old
    20180508201643    completed  8 minutes old
    20180508201952    completed  4 minutes old
    ```
2. Detach the File system from the old A2HA cluster.

3. Configure the backup at Automate HA cluster, in case if you have not configured, please refer this [Doc: Pre Backup Configuration for File System Backup](/automate/ha_backup_restore_prerequisites/#pre-backup-configuration-for-file-system-backup)

4. From the Step 3, you will get the backup mount path.

5. Stop all the services at frontend nodes in Automate HA Cluster.

    - Run the below command to all the Automate and Chef Infra Server nodes
      
    ``` bash
    sudo chef-automate stop
    ```

6. To run the restore command we need the airgap bundle. Get the Automate HA airgap bundle from the location `/var/tmp/` in Automate instance. Example : `frontend-4.x.y.aib`.
    - In case of airgap bundle is not present at `/var/tmp`, in that case we can copy the bundle from the bastion node to the Automate node. 

7. Run the command at the Chef-Automate node of Automate HA cluster to get the applied config

    ```bash
    sudo chef-automate config show > current_config.toml 
    ``` 

8. Add the OpenSearch credentials to the applied config.

    - If using Chef Managed Opensearch, then add the below config into `current_config.toml` (without any changes).  
        ```bash
        [global.v1.external.opensearch.auth.basic_auth]
            username = "admin"
            password = "admin"
        ```
    - If using AWS Managed services, then add the below config into `current_config.toml` (change this with your actual credentials)
        ```
        [global.v1.external.opensearch.auth]
            scheme = "aws_os"
        [global.v1.external.opensearch.auth.aws_os]
            username = "THIS YOU GET IT FROM AWS Console"
            password = "THIS YOU GET IT FROM AWS Console"
            access_key = "<YOUR AWS ACCESS KEY>"
            secret_key = "<YOUR AWS SECRET KEY>"
        ```



9. To restore the A2HA backup on Chef Automate HA, run the following command from any Chef Automate instance of the Chef Automate HA cluster:

    ```cmd
    sudo chef-automate backup restore /mnt/automate_backups/backups/20210622065515/ --patch-config current_config.toml --airgap-bundle /var/tmp/frontend-4.x.y.aib --skip-preflight
    ```

10. After the restore is successfully executed, you will see the below message:
  
    ```bash
    Success: Restored backup 20210622065515
    ```

11. Copy the `bootstrap.abb` bundle to all the Frontend nodes of the Chef Automate HA cluster. Unpack the bundle using the below command on all the Frontend nodes.

    ```cmd
    sudo chef-automate bootstrap bundle unpack bootstrap.abb
    ```

12. Start the Service in all the frontend nodes with the below command.

    ``` bash
    sudo chef-automate start
    ```
{{< warning >}}

- After the restore command is successfully executed. If we run the `chef-automate config show`, we can see that both ElasticSearch and OpenSearch config are part of Automate Config. After restoring Automate HA talk to OpenSearch.

-  We should remove the elaticsearch config from all Frontend nodes, to do that, redirect the applied config to the file and set the config again.
   For example:
 
    ```bash
    chef-automate config show > applied_config.toml
    ```
    Remove the below field from the `applied_config.toml`.
    ```
   [global.v1.external]
     [global.v1.external.elasticsearch]
       enable = true
       nodes = [""]
       [global.v1.external.elasticsearch.auth]
       scheme = ""
       [global.v1.external.elasticsearch.auth.basic_auth]
           username = ""
           password = ""
       [global.v1.external.elasticsearch.ssl]
       root_cert = ""
       server_name = "" 
    ```

    Apply this modified config by running below command. 

    ```bash
    chef-automate config set applied_config.toml
    ```

    These steps should be executed on all the Frontend nodes.  

{{< /warning >}}


## Troubleshooting

**In case of Restore failure from ElasticSearch to OpenSearch**

> **Error: Failed to restore a snapshot**

Get the basepath location from the A2HA Cluster using the curl request below.

> **REQUEST**

```bash
curl -XGET http://localhost:10144/_snapshot/_all?pretty -k 
```

> **RESPONSE**

Look for the `location` value in the response.

```json
"settings" : {
    "location" : "/mnt/automate_backups/automate-elasticsearch-data/chef-automate-es6-compliance-service",
}
```

`location` value should be matched with the OpenSearch cluster. In case of `location` value is different, use the below script to create the snapshot repo.

```bash
indices=(
chef-automate-es5-automate-cs-oc-erchef
chef-automate-es5-compliance-service
chef-automate-es5-event-feed-service
chef-automate-es5-ingest-service
chef-automate-es6-automate-cs-oc-erchef
chef-automate-es6-compliance-service
chef-automate-es6-event-feed-service
chef-automate-es6-ingest-service
)

for index in ${indices[@]}; do

curl -XPUT -k -H 'Content-Type: application/json' http://localhost:10144/_snapshot/$index --data-binary @- << EOF
{
  "type": "fs",
  "settings": {
    "location" : "/mnt/automate_backups/automate-elasticsearch-data/$index"
  }
}
EOF
done
```
