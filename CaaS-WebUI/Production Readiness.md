## Production Consideration
### Infrastructure (WebUI)
https://katana.int.gdcorp.tools/apps/caas/caas-open-webui

https://github.com/gdcorp-dna/caas-open-webui/tree/main/infrastructure

- [ ] Environment
  - [x] Katana
  - [x] Regions: us-west-2
  - [ ] Need to add additional environments.
- [ ] Environment Configuration
  - [x] WebUI secrets and connections initialized via ENV during [bootstrap](https://github.com/gdcorp-dna/caas-open-webui/blob/main/open-webui/gd-bootstrap.py).
  - [x] PostGRES, ESSP, ElastiCache, and S3 connections are stored in AWS Secrets Manager.
  - [ ] Need to add additional infrastructure in additional ENVs.
- [ ] CI/CD
  - [x] Docker containers stored in AWS ECR.
  - [x] Docker container uses latest image of WebUI.
    - [ ] UI Customization will require changes to docker build.
  - [ ] GitHub actions for deployment
    - [ ] Need to add additional actions for each environment.
- [ ] Load Balancing/Scaling
  - [x] Katana handles load balancing
  - [x] Katana horizontal auto-scaling
  - [x] Fail-over for containers handled by Katana auto-scaling.
- [ ] Rollback
  - Handled by katana?

### Infrastructure (Pipelines)
https://katana.int.gdcorp.tools/apps/caas/caas-open-webui-pipeli-0

https://github.com/gdcorp-dna/caas-open-webui-pipelines/tree/main/infrastructure

Same considerations as above for WebUI.

## Security
- [x] Authentication & Authorization
  - [x] OKTA integration
    - [x] AD Username/Password, AD Groups
  - [x] User session and expiration handled by Okta.
- [x] HTTPS
  - SSL configuration through Katana and AWS Load Balancer
- [x] Access Control
  - [x] All OKTA users have general access.
  - [ ] Need to identify if we want additional restrictions based on user/group (i.e. models, pipelines, functions, etc...)
  - [ ] Sharing is natively supported by WebUI with the following considerations:
    - [ ] Sharing of "companion's" or knowledge bases only support Public/Private/AD Group (no sharing to specific users).
    - [ ] Sharing of conversations is supported by creating share links and manually passing the link to whichever user you wish to share to.
- [x] Secrets
  - [x] Secrets are stored within AWS SecretsManager and loaded into ENVs during bootstrap.
  - [x] WebUI has some support for storing OpenAI connection, pipeline, tool keys (aka valves) within the UI.
- [ ] Security Review (???)
- [ ] Audit Logging (???)
  - [ ] Persistent data storage contains file audit.log. This needs to be investigated further to figure out how to get these logs somewhere they can be stored indefinitely and analyzed.

## Monitoring
- [ ] Application Monitoring
  - [x] Kibana environment for CaaS project already setup for non-prod/prod ENVs.
  - [ ] Need to create dashboards.
- [ ] Logging
  - [ ] Infrastructure appears to be setup and currently writing APP logs to AWS CloudWatch.
    - [ ] Need to stream logs to ESSP to power monitoring dashboards.
    - [ ] Need to enable WebUI python logger for additional logging.
      - [ ] Will need to identify what additional python logs are available, and what to enable.
- [ ] Alerting
  - [ ] Will need to identify an alert mechanism (i.e. CloudWatch, Spaq, ???).
  - [ ] Integrate alerting mechanism with Slack, ServiceNow, and On-Call.
- [ ] Health Checks
  - [ ] Katana/Load Balancer performs health checks and auto-scales/rotates bad instances.
  - [ ] Integrate with Spaq for NOC support.

## Performance
- [ ] Load Testing
  - [ ] Need to create/execute load testing plan to gain a measure of reliability and maximum utilization.
- [x] Caching
  - [x] Frontend/Backend caching is handled by WebUI APP internally.
- [x] Database Maintenance/Optimization
  - [x] Database (Aurora PostGRES SQL) management handled by WebUI APP internally.

## User Experience (UX)
- [ ] Onboarding
  - [ ] Need to create onboarding documentation and basic tutorials for usage.
- [ ] Feedback
  - [ ] We can continue to use #go-caas channel for feedback, or we can create a new channel specific to WebUI.
  - [ ] Gathering feedback within UI will require forking and local development for custom features.

## Data Governance
- [ ] Audit Logging
  - [ ] We need to identify if the file audit.log within persistent storage can be output to ESSP for audit trails, or if there exists another way to capture events/changes using APPs python logger and ESSP.
- [ ] Retention
  - [ ] Need to define retention policies for conversation histories and documents/files.
    - [ ] WebUI APP maintains the conversation history (within Aurora PostGRES SQL) and documents/libraries (within S3) internally. I see no options to set retention policies within the app. We may need a custom solution to scrub storage of old items if desired (if possible without interfering with APPs ability to operate).
    - [ ] Audit/Log retention is pretty straightforward using existing patterns of index rotation and deletion within ESSP.
- [ ] PII Handling
  - [ ] Sensitive Content/Moderation
    - [ ] APP does not seem to support moderation/sensitivity providers like we currently use for CaaS-Web.
      - [ ] We need to investigate a way to add moderation/sensitivity checks; possibly by using Filter Functions or custom Pipelines.
  - [ ] SDM
    - [ ] APP does not natively support SDM as we currently have in CaaS-Web. Possibly solutions include:
      - [ ] APP UI customization by forking the WebUI repo and making modifications to the Svelte code.
        - [ ] Add an option to enable/disable SDM mode for each conversation or model.
        - [ ] Modify where the conversation history is stored when SDM is enabled to ensure data isolation.
        - [ ] Restrict usage of SDM to certain AD security groups.
        - [ ] This may or may not even be possible as there appear to be no way to dynamically change the location of stored conversations. This would mean we'd have to figure out how to modify the way WebUI stores/retrieves conversation histories/documents and dynamically change it depending upon the selected model.
        - [ ] Level of effort to accomplish this task would be rather substantial, because it would require substantial changes to the storage patterns of WebUI.
      - [ ] Or, create duplicate WebUI site in each environment, one for non-SDM and one for SDM-only conversations.
        - [ ] This would solve the problem of data isolation as well as not requiring customized UI changes.
        - [ ] Each instance of WebUI would point to different Aurora PostGRES SQL, ESSP, S3, and ElastiCache instances, thereby maintaining complete isolation of conversation histories/documents between the two instances. 
        - [ ] Would need two different Katana APPs with different URLs (i.e. NON-SDM at caas.open-webui.dev-godaddy.com, SDM at caas.open-webui-sdm.dev-godaddy.com).
        - [ ] We could use a [banner](https://docs.openwebui.com/features/banners#banner-options) to indicate visually whether the user is currently on the Non-SDM or the SDM version of the site. Or possibly identify a different theme for the UI.
        - [ ] Level of effort would be relatively simple. We would need to define a Function/Pipeline to handle each request on the SDM version of the site to implement/enforce any additional handling of SDM besides data isolation. It would also require relatively small amount of UI changes.
  - [ ] Backfill
    - [ ] May not be possible due to the radical differences in storage between WebUI (Aurora PostGRES SQL, S3) and current CaaS storage (DDB, S3); and whatever differences between storage structures and organization which would require transformation and mapping to other WebUI entities (such as models, knowledge bases, etc.)
      - [ ] Conversations: Probably the most difficult as it would require mappings to other WebUI entities such as knowledge bases, users, models, etc...
      - [ ] Companions: May be possible; however, certain aspects like sharing (WebUI only has support for sharing with other AD Groups) and hypothesis which doesn't appear to be supported at all.
      - [ ] Libraries: May be possible; would need to figure out not only where to save the documents, but also any SQL database records which may need to be updated for searching/mapping.

## Operations
- [ ] Runbooks
  - [ ] Create SOPs for outages, deployments, or known issues.
- [ ] Support Channels
  - [ ] Set up Slack, Jira, or internal ticketing for help requests.
- [ ] On-Call
  - [ ] Define ownership and escalation paths.

## Compliance & Auditing
- [ ] Access Logging
  - [ ] Track who accessed what and when.
- [ ] Change Tracking
  - [ ] Use git or changelogs to document updates.
  - [ ] Use ServiceNow for COs.
