# Privilege Audit 

## Scenario
Management board wants to look at who can do what in the Azure tenant after a incident that happened. They want to have someone to look around and report to the CISO of any findings from auditing. 

## Environment
Live multi-user Azure training tenant, Reader access and one PIM eligible role

## Audit Methods
| Method               | Sees                                        | Misses                              |
|----------------------|---------------------------------------------|-------------------------------------|
| IAM blade / export   | Active assignments at scope, inherited      | Group members, orphaned principals  |
| Azure CLI            | Same, plus null principalName (orphans)     | One scope per run                   |
| Resource Graph (KQL) | Whole tenant in one query                   | Eligible assignments                |
| PIM export           | Eligible vs active, activation history      | Assignments outside PIM             |

## Method 1. The Access Control (IAM) BLADE

The Access Control (IAM) section and role assignment export provide an overview of all identities with access to specific resource scopes, including inherited permissions. However, one limitation is that group memberships are not displayed, making it difficult to identify the individual users within each group. It can also be challenging to identify orphaned role assignments associated with deleted or unrecognized identities labeled as 'Identity not found.
  <img width="1925" height="877" alt="Screenshot 2026-11-22 164715" src="https://github.com/user-attachments/assets/ede30bac-8990-4d66-b6f5-212e0de3feb4" />
CSV Export showing all the subscriptions with excessive owner permissions of many resources.
  <img width="2482" height="1177" alt="Screenshot 2026-11-22 165310" src="https://github.com/user-attachments/assets/15de4f0c-1c33-455e-bb99-d7dbe5a9347c" />
Findings- Excessive Permissions
User Carl Jenkins has Owner role many resources. Owner role too much permission for what the user would require.
Recommended- Removal of owner roles and review what permissions user would need with least privilege to do their job. I would implement PIM for Carl Jenkins so he can activate role for the duration need and require MFA OR approval.


## Method 2. Azure CLI 

I used Azure CLI which provides the same role assignment information as the Azure portal but allows access reviews to be performed through repeatable scripts and automated processes. It can also identify assignments where the principalName field is empty, which may indicate a deleted account with permissions that were never revoked. One limitation is that each resource scope must be reviewed separately.
  Using command line az role assignment list --resource-group rg-madhatlabs-prod-cus in the cloud shell we get 
  <img width="2479" height="1327" alt="Screenshot 2026-11-22 192356" src="https://github.com/user-attachments/assets/0dc00a81-3b8e-4a35-8990-7c27e2ac788b" />
 Findings- A Orphaned permissions 
The account never had its permissions revoked even after the account was deleted.  
 Recommended- To properly delete orphaned permissions of account with proper Azure CLI commands.


## Method 3. Azure Resource Graph with KQL

The next method Azure Resource Graph with KQL checks the whole tenant in a single query instead of one scope at a time but it only sees ACTIVE assignments.
  <img width="2482" height="1251" alt="Screenshot 2026-11-22 194909" src="https://github.com/user-attachments/assets/fbb87204-9d0e-4992-9871-a8af8bc25a09" />
 Findings- KQL query can help broadly across without going to individual resources. 
In this method i was able i was able to find the principal ID with the Resource Graph to look at assignments more broadly. It also shows how KQL can help throughout the scopes. The results would need to depend on permissions and scope of the query.


## Method 4. PIM 

Privileged Identity Management (PIM) provides visibility into both eligible and active role assignments, allowing administrators to distinguish between permissions that require activation and those that are permanently assigned. It also provides an activation history showing which users elevated their privileges, when the activation occurred, and the justification provided. One limitation is that PIM does not account for permanent role assignments that have not been configured or managed through PIM.
  I went into the Azure portal and searched for PIM and under manage i select Azure resources. I then select the subscription and RG that was found in the other methods. After selecting manage resource at the bottom we can see the manage blade and under it after clicking on it we see Assignments we see the eligible and active assignments even expired ones. I exported the audit report which then gave me more identity information and helped me identify the user/account from the previous method. 
  <img width="2503" height="1265" alt="Screenshot 2026-11-22 202903" src="https://github.com/user-attachments/assets/298d065f-99aa-45b5-b031-4e9f2bd1c264" />
 Findings- Nothing in the scope had a expiration which meant that it had a permanent active time on all those assignments.
 Recommended- Review permanently assigned privileged permissions and move appropriate administrative accounts to PIM eligible access. Set up security requirements such as MFA, access justifications, time limits, and approval processes to ensure elevated permissions are only granted when necessary


## 5. The Hunt
Using all these methods i combed through and looked for a over-provisioned account. Using the my privilege activation in my Role assignment to inspect more permissions i noticed a resource that didn't come up in the previous methods. I went into Cloud Shell and used az role assignment list --resource-group rg-madhatlabs-RG THAT APPEARED AFTER USING THE AUTHORIZED PRIVILEGE ACTIVATION.
A resource-group scope with a Owner assignment came up.
<img width="2512" height="435" alt="Screenshot 2026-11-22 232357" src="https://github.com/user-attachments/assets/5bcc2b84-dbfd-447c-bb9b-15dd3d9b7be6" /> 
  Findings- To properly evaluate a role assignment, you must consider who has access, what resources they can access, how the permissions are inherited, and whether the access is necessary for their job.
  Recommended- Confirm what the account needs to do, review its permissions and access requirements, and remove any elevated access it does not need.
## What broke / what surprised me
“One of the biggest challenges was understanding why each tool showed different information about the same access environment.

The Azure Portal provided an easy-to-read overview, while Azure CLI showed more detailed information that helped me investigate accounts the portal could not identify.
Resource Graph allowed me to review permissions across multiple resources using KQL.
PIM helped me understand the difference between permissions users are eligible to activate and permissions they already have.
I learned that finding something unusual is only the first step in an audit. I also need to understand what it means, verify the information, and determine whether it creates a real security risk.”

## What I learned

-Through this audit, I learned that Azure role assignments need to be reviewed based on who has access, what resources they can access, how permissions are inherited, and whether the access is necessary for their job.

-I also learned that deleting an account does not always remove its assigned permissions, which can leave behind access that needs to be reviewed.

-Using Azure CLI helped me examine detailed permission information and identify unusual entries. Resource Graph and KQL made it easier to review access across multiple Azure resources.

-PIM helped me understand the difference between permissions that users can activate when needed and permissions they already have active.

-Most importantly, I learned that using multiple tools together provides a more complete picture of access, helps identify potential security risks, and supports better recommendations.

"What I'd do differently"

I would make sort of playbook for a more organized way of collecting documentation for the affected findings and recommendations for efficiency.
