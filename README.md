# Module Issue - Lingering Resources After Destroy

## Root Cause Explanation

The CloudSQL lingering resources issue occurs when a CloudSQL instance is destroyed while ongoing operations (such as backups) are in progress. This untimely destruction can leave the instance in a FAILED state, causing subsequent Terraform runs to fail due to unresolved SQL users and resources.

## Strategy Plan for Resolution

### Steps to Address the Issue

1. **Graceful Handling of Ongoing Operations:**
   - Update Terraform scripts to check ongoing operations (e.g., backups) on the CloudSQL instance before initiating the destroy operation. Delay the destroy operation until ongoing operations are completed or canceled gracefully.

2. **Error Handling and Reporting:**
   - Enhance error handling to capture cases where the destroy operation fails due to ongoing operations. Report detailed information on the ongoing operations to aid in troubleshooting.
   - Keep an eye on any read replicas, or other resources that may have been manually added outside of your Terraform configuration. 
   - Use terraform's `state rm` to tell it to forget that the users and database exist so it won't actively try to delete them (and fail) at destroy time.
      ```
      terraform state list 
      terraform state rm module.your_server_name.google_sql_user.users
      terraform state rm module.your_server_name.google_sql_database.your_database_name
      terraform destroy 
      ```

3. **Manual Cleanup:**
   - After the destroy operation, it might be necessary to manually remove the CloudSQL instance using the Google Cloud Platform (GCP) console. Ensure all associated resources are appropriately cleaned up.

4. **Manage Terraform State:**
   - Update the Terraform state file (`terraform.tfstate`) to reflect the changes made during the destroy operation. Ensure that the state accurately represents the current infrastructure configuration.
 
