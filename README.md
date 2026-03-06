# Lab  - Chaos Engineering


### Summary: Fully integrated chaos experiments with the delivery process

**Learning Objective(s):**

- Auto generate chaos experiments on deployed services
- Build a chaos experiments using a base fault (out of 200 OOTB faults)
- Embed chaos engineering experiments into the deployment process
- Add continuous verification to the deployed service
- Automate release validation

**Steps**

1. From the module selection menu select **resilience testing**

<img width="733" height="402" alt="Screenshot 2026-03-06 at 08 14 35" src="https://github.com/user-attachments/assets/3d6bba05-4189-4c14-893b-cb75e270029d" />


4. From the left hand menu, go to **Project Settings**
5. From the available tiles select “Discovery”
6. After expanding the side menu of the “DA-K8s” agent click on “Discover Now”

  ![image](https://github.com/user-attachments/assets/be1a029d-9b4e-4971-9519-f284ba75c815)



**Create Application Map**

1. After discovery is complete double click on the agent “DA-K8s”

   ![image](https://github.com/user-attachments/assets/3094b76d-1d27-429d-ab67-0fdb8e978894)


1. Select the "Application Maps" tab
2. Click on **Create New Application Map** and enter the following values

| Input                        | Value|  
| ---------------------------- | ------ |
| Name                         |workshop-am|

4. Select the relevant services for your project name "use the search function to find the services"
5. Click Save


**Auto Generate Chaos Experiments**
1. From left handside menu select **Resilience Management**
2. Drill down to the previously created application map
3. Navigate to **Chaos Experiments**
4. Select **Only a few**

Observe the auto generated experiments and run the **web-backend experiment**




---------------

**Create Experiments manually**

This time we will create a network corruption experiment

1. From the left hand menu, go to **Chaos Testing**
2. Select **+New Experiment**

| Input                        | Value|  
| ---------------------------- | ------ |
| Name                         |network-corruption|

3. Select **Harness Infra**

  ![Screenshot 2024-11-28 at 14 24 21](https://github.com/user-attachments/assets/c47834a3-fe88-44ed-be7e-7cee97bcb303)

  - Click on **"Select a chaos Infrastructure"**

 
4. On the popup window select the available options

| Input                        | Value|  
| ---------------------------- | ------ |
| Select Environment|prod|
| Select Infrastructure|k8s|

5. Click on next to navigate to the experiment builder
6. Click on **Add Fault**
7. From the list of available faults select **Pod Network Corruption**
8. From the navigation bar select **Target Application**

| Input                        | Value | Notes |
| ---------------------------- | ------ | -------|
| Target Workload Kind|deployment||
| Target Workload Namespace ||**Select the namespace available from the dropdown**|
| Target Workload Names | Pick the backend deployment name|We will change that later |
|Target Workload Labels | leave empty||


9. From the navigation bar select **Tune Fault**

| Input       | Value |
| ----------- | ----- |
| Total Chaos Duration |150|
| Network Packet Corruption Percentage |100|
10. Click on **Apply Changes** and then **Save**
11. Run the experiment and observe the logs. 
12. While the experiment is running navigate to the application's endpoint (see image below)
13. Observe the network errors

| project                | domain        | suffix |
| ---------------------- | ------------- | ------ |
| http\://\<project\_id>|.cie-bootcamp|.co.uk|

<img width="1415" height="205" alt="image" src="https://github.com/user-attachments/assets/cb85c608-bfbb-4ec5-b179-c6ebd6ff394c" />


**Validate the health automatically**

1. From the left hand menu, go to **Project Settings**
2. Select **Chaos Probles**
3. Create a new probe by clicking **+ New Probe**
4. Select the HTTP probe
   
| Field                | Value        | Notes |
| ---------------------- | ------------- | ------ |
| http\://\<project\_id>.cie-bootcamp.co.uk |||
| Criteria | == | | 
| Response Code | 200 | | 

5. Move to the next tab **Configure Properties**


| Field                | Value        | Notes |
| ---------------------- | ------------- | ------ |
| Timeout | 20s ||
| Interval | 2s | | 
| Attempt | 10 | | 
| Initial Delay | 5s ||

Now that we have our probe navigate to **Chaos Testing**
Drill down to the **network-latency** experiment

Hovering over the network fault we can add our newly created probe 
Select **+ Add a parallel node -> Add a probe**

<img width="519" height="354" alt="image" src="https://github.com/user-attachments/assets/519fc841-bbf1-4480-a01c-4583eab0b208" />

Save and rerun the experiment 
Observe the **Resilience Score** generated in comparisson to the previous execution



---------------

**Create Experiments manually**

1. From the left hand menu, go to **Chaos Testing**
2. Select **+New Experiment**

| Input                        | Value|  
| ---------------------------- | ------ |
| Name                         |pod-memory|

3. Select **Harness Infra**

  ![Screenshot 2024-11-28 at 14 24 21](https://github.com/user-attachments/assets/c47834a3-fe88-44ed-be7e-7cee97bcb303)

  - Click on **"Select a chaos Infrastructure"**

 
4. On the popup window select the available options

| Input                        | Value|  
| ---------------------------- | ------ |
| Select Environment|prod|
| Select Infrastructure|k8s|

5. Click on next to navigate to the experiment builder
6. Click on **Add Fault**
7. From the list of available faults select **Pod Memory Hog**
8. From the navigation bar select **Target Application**

| Input                        | Value | Notes |
| ---------------------------- | ------ | -------|
| Target Workload Kind|deployment||
| Target Workload Namespace ||**Select the namespace available from the dropdown**|
| Target Workload Names | Pick the backend deployment name|We will change that later |
|Target Workload Labels | leave empty||


9. From the navigation bar select **Tune Fault**

| Input       | Value |
| ----------- | ----- |
| Total Chaos Duration |600|
| Memory Consumption |300|
| Number of workers |1|
| Pod affected percentage|100|

10. Click on **Apply Changes** and then **Save**


**Change target service to canary using YAML**

1. From the pipeline visual editor switch to yaml
2. Click the edit button to go into edit mode
3. Locate the service name (set on previous state) **TARGET_WORKLOAD_NAMES**
4. Replace it with **backend-<project_name>-deployment-canary** where project_name is the harness project. Summary: add the suffic **-canary** to the target workload
5. Save the experiment


**Embed chaos experiments into CD pipelines**

1. From the module selection menu select Continuous Delivery & GitOps


   ![Screenshot 2024-11-28 at 14 07 22](https://github.com/user-attachments/assets/898ee27b-7369-47c6-a145-e74b49bb4bed)

   
2. From the left hand side menu select pipelines and drill down to the existing pipeline

3. In the existing pipeline, within the Deploy backend stage **after** Canary Deployment and **before** the approval step click on the plus icon to add a new step

4. Add a **Verify** step with the following configuration

| Input                        | Value  | Notes                                                                                            |
| ---------------------------- | ------ | ------------------------------------------------------------------------------------------------ |
| Name                         |Verify|                                                                                                  |
| Continuous Verification Type |Canary|                                                                                                  |
| Sensitivity                  |Low| This is to define how sensitive the ML algorithms are going to be on deviation from the baseline |
| Duration                     |10mins|                                                                                                  |

5. Under the verify step click on the plus icon to add a new step in parallel


   ![Screenshot 2024-11-28 at 14 28 38](https://github.com/user-attachments/assets/368ba808-d303-43f8-8824-5d2e09367b01)

   
6. Add a **chaos** step with the following configuration

| Input                        | Value  |
| ---------------------------- | ------ |
| Name                         |Chaos|
| Select Chaos Experiment |pod-memory|
|  Expected Resilience Score|50| 

7. Click on Apply Changes

8. Click **Save** and then click **Run** to execute the pipeline with the following inputs. As a bonus, save your inputs as an Input Set before executing (see below)

| Input       | Value | Notes       |
| ----------- | ----- | ----------- |
| Branch Name |main| Leave as is |


# Lab 6 - Validate Release

**Learning Objective(s):**

- Identify the difference in traffic between normal and canary instances of the application

- Automate release validation

- Use complex deployment strategies to reduce the blast radius

**Outcomes**
- Force failure of continuous delivery validation using chaos engineering

**Steps**:

- While the canary deployment is ongoing navigate to the web page and see if you can spot the canary (use the check release button) 

| project                | domain        | suffix |
| ---------------------- | ------------- | ------ |
| http\://\<project\_id>|.cie-bootcamp|.co.uk|

- Drill down to the distribution test tab and run the traffic generation by clicking the **Start** button

- Observe the traffic distribution

- Validate the outcome of the verification on the pipeline execution details


\
![](https://lh7-us.googleusercontent.com/docsz/AD_4nXdbAmEJ5zQPsKlw_nEknWvYo97pm5eWCXr6vU8-GgIL0ulAOSH9N07PoEcVSknARVQo7Tgj1s31VHqR1I3hu2dMIO1rIX5HHcmTPXoQPoyo8CPv13OhnJN5WVcZqSwUXzdDHmm3PxUnhtpGVl0PAMJ_1wnuodvUbVPBOdnGKQ?key=cRG2cvp_PHVW0KG2Gq6Y_A)
![](https://lh7-us.googleusercontent.com/docsz/AD_4nXf-5oWX9OfvdmEb9MBm2_h2KKAa_QwmiJoM0fiKrTuxAr6GR4wxeulSlk48gyBK3dykrtIslDSkxpiGytrxH0JaxaQ4ZgTYxbmc8OenAH3nhGCvvOAxkWVjVBp1TRg_qQQi9z8OrNPK4udPtNL1LIyym6Ch5IMzrulFOcXhOQ?key=cRG2cvp_PHVW0KG2Gq6Y_A)

**Bonus**:
- If the verification fails harness defaults to a manual intervention, you can now decide what you want to happen next (rollback, ignore etc.) 

- Add a canary rollout from 10% to 50% traffic and see how this impacts the traffic distribution


# Lab 7 - Governance/Policy as Code

### Summary: Create and apply policies as code in order to enable governance and promote self-service. In Lab 2 we saw how a user is impacted by policies in place, now is the time to create such policies

**Learning Objective(s):**

- Create a policy that evaluates when editing pipelines

- Create a policy that evaluates during pipeline execution

- Test policy enforcement

**Steps\
****Create a Policy to require Approvals**

1. From the secondary menu, select **Project Settings** and select **Governance Policies**

2. Click **Build a Sample Policy**

3. From the suggested list select **Pipeline - Approval**  and click on next

4. Click Next: Enforce Policy

5. Set the values according to the table  below and confirm

| Input            | Value        | Notes |
| ---------------- | ------------ | ----- |
| Trigger Event    |On Run|       |
| Failure Strategy |Error & exit|       |

**Test the Policy to require Approvals**

1. Open your pipeline

2. Try to run the pipeline and note that the failure due to lack of an approval stage

3. Click **Save** and note that the failure due to lack of an approval stage

4. Open the pipeline in edit mode and navigate to the “**frontend**” stage

5. Before the rolling deployment step add **Harness Approval** step according to the table  below

| Input            | Value            | Notes |
| ---------------- | ---------------- | ----- |
| Step Name        |Approval|       |
| Type of Approval |Harness Approval|       |

6. Configure the Approval step as follows

| Input       | Value             | Notes |
| ----------- | ----------------- | ----- |
| Name        |Approval|       |
| User Groups |All Project Users|       |

7. In a similar way as before navigate to the “**backend**” stage
8. Before the canary deployment block add **Harness Approval**
10. Click **Save** and note that the save succeeds without any policy failure


# Lab 8 - Governance/Policy as Code (Advanced)

**Create a Policy to block critical CVEs**

1. From the secondary menu, select **Project Settings** and select **Policies**

2. Select the **Policies** tab 

3. click **+ New Policy**, set the name to **Runtime OWASP CVEs** and click **Apply**

4. Set the rego to the following and click **Save**

<!---->

    package pipeline_environment
    deny[sprintf("Node OSS Can't contain any critical vulnerability '%d'", [input.NODE_OSS_CRITICAL_COUNT])] {  
       input.NODE_OSS_CRITICAL_COUNT != 0
    }

5. Select the **Policy Sets** tab

6. Click **+ New Policy Set** and configure as follows

| Input                      | Value                     | Notes |
| -------------------------- | ------------------------- | ----- |
| Name                       |Criticals Not Allowed|       |
| Entity Type                |Custom|       |
| Event Evaluation           |On Step|       |
| Policy Evaluation Criteria |                           |       |
| Policy to Evaluate         |Runtime OWASP CVEs|       |

7. For the new policy set, toggle the **Enforced** button

**Add Policy to Pipeline**

1. Open your pipeline

2. Go to an execution that already ran, and copy the CRITICAL output variable from the OWASP step like so:\
   ![](https://lh7-us.googleusercontent.com/docsz/AD_4nXfYQ7ba5Q_cQ9xy2AFVZ5Mt0iZPYbyQDmBonp0pBQA13Z_IUeYdK8gRSbddtf_V3bSRfbhKWDbRSUVJTx3BTCc_VmwLIWyWLkdh89nLh0sEBA6fqQxTy0NADZ0YPZwCirNycRVGUQACdItaBotovPs5Hg6CmRpQHk5ysgV6RUlhSbIbkNxmHAo?key=cRG2cvp_PHVW0KG2Gq6Y_A)

3. Select the **frontend** stage

4. Before the **Rollout Deployment** Step Group, add a **Policy** type step and configure as follow

| Input       | Value                                          | Notes                                                                                                                                                   |
| ----------- | ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Name        |Policy - No Critical CVEs|                                                                                                                                                         |
| Entity Type |Custom|                                                                                                                                                         |
| Policy Set  |Criticals Now Allowed| Make sure to select the Project tab in order to see your Policy Set                                                                                     |
| Payload     |{"NODE\_OSS\_CRITICAL\_COUNT": _\<variable>_}| Set the field type to Expression, then replace _\<variable>_ with OWASP output variable CRITICAL. Go to a previous execution to copy the variable path. |

5. Save the pipeline and execute. Note that the pipeline fails at the policy evaluation step due to critical vulnerabilities being found by OWASP.
