
# Standard set of NFRs

## Instructions
**Treasury Management System (TMS) selection for Ministry of Finance, Kingdom of Saudi Arabia**

Please complete the functional requirements in the adjoining sheets. Vendors should insert their responses as indicated in the table below against each of the stated criteria. This should be done by typing in the response column labelled ‘4/3/2/1/0’ in the Excel spreadsheets provided with this document – additional comments can be inserted in the comments column. Vendors are requested to substantiate their response with additional comments.

### Response Scale
- **4 — Functionality Provided Out-Of-The-Box**: The Offeror provides the functionality from its existing code base and basic/minimal configuration may be required to deliver requirement.
- **3 — Functionality Provided, But Requires Extensive Configuration**: Extensive Configuration implies that solution requires configuration, in excess of two weeks and/or 80hrs, to deliver requirement.
- **2 — Functionality Provided, But Requires Customization**: Customization implies that specialized software and/or data programmatic coding is required to deliver on requirement.
- **1 — Functionality Provided, But Requires Integration With Third Party**: Solution requires third party hardware/software solution to meet requirement. Please name and identify third party solution required in the Description and/or Comments field.
- **0 — Functionality Not Provided**: Solution does not meet requirement, even with a third party solution.

---

## None Functional Requirements

| Category | Sub category | Execution / Non-Execution | None Functional Requirements description | Response | Comments |
|---|---|---|---|---|---|
| Availability_and_reliability | Resiliency / failback | Execution | Where data and/or files have been transmitted incorrectly by the System, it must be possible to re-send data. This will be identified and initiated and executed as per agreed process and/or design. |  |  |
| Availability_and_reliability | Disaster Recovery | Non-Execution | Web Service/API Availability: Service Level is to ensure that there are less than 10.5 hours (approx 95%) of non-availability for the full web service/API per calender month. |  |  |
| Availability_and_reliability | Disaster Recovery | Execution | Reasonable Throughput and performance requirements in DR situation |  |  |
| Availability_and_reliability | Availability | Execution | System should provide almost near real time transaction processing for all queries. |  |  |
| Availability_and_reliability | Resiliency / failback | Non-Execution | There is nothing inherent in the solution that prevents SLA being met. |  |  |
| Availability_and_reliability | Recovery | Execution | The time from identification of the need to revert to a backup and the time at which the backed up information is made available shall not exceed 3 hours. |  |  |
| Availability_and_reliability | Recovery | Execution | The time between failure of the service that does not require Disaster Recovery to be invoked (e.g. the failure of Solution hardware) and the recovery of the system (Recovery time objective) shall not exceed 5 hours. This shall be timed from the outage to the recommencement of the service. |  |  |
| Availability_and_reliability | Recovery | Execution | The system/infrastructure must be capable of being recovered after a failure in each hardware and software component. |  |  |
| Availability_and_reliability | Disaster Recovery | Execution | The solution will incorporate a manual disaster recovery (single site failure) capability (assuming it is not already operating on the DR site). |  |  |
| Availability_and_reliability | Resiliency / failback | Non-Execution | The Solution must be resilient such that any component or server failure does not cause a failure of the Solution (except for the documented SPOF). Design should be loosely coupled. |  |  |
| Availability_and_reliability | Disaster Recovery | Execution | The solution must be recoverable to a DR site within 5 hours after the decision to invoke DR (assuming it is not already operating on the DR site). |  |  |
| Availability_and_reliability | Resiliency / failback | Execution | The Solution must be capable to supporting a restoration to normal operations (i.e. using previously failed component/server) without requiring a Solution outage. |  |  |
| Availability_and_reliability | Recovery | Execution | The infrastructure must be capable of being recovered in a controlled manner after a failure in multiple hardware and software components. There should be documented process and steps to recovery the system upon failure. |  |  |
| Availability_and_reliability | Disaster Recovery | Execution | The existing Disaster Recovery capability must not be affected by the Solution. Periodic drill. |  |  |
| Availability_and_reliability | Disaster Recovery | Execution | The alternate site recovery (ASR) approach must be tested. - once in a quarter. |  |  |
| Availability_and_reliability | Disaster Recovery | Non-Execution | The alternate site recovery (ASR) approach must be documented and signed off. |  |  |
| Availability_and_reliability | Availability | Non-Execution | Solution availability will be measured from the user terminal: Standard Desktop PC; Mobile devices |  |  |
| Availability_and_reliability | Availability | Non-Execution | Solution availability will be 24 by 7. |  |  |
| Availability_and_reliability | Resiliency / failback | Non-Execution | Single points of failure within the solution must be identified, documented and agreed. |  |  |
| Availability_and_reliability | Availability | Non-Execution | Service maintaince windows will be allowed as 8 hours every month. |  |  |
| Availability_and_reliability | Disaster Recovery | Non-Execution | Other Service Levels for Service Availability are suspended whilst operating from the secondary site. |  |  |
| Availability_and_reliability | Availability | Non-Execution | All NFR's hold true assuming SLA for availability is 24X7 hrs per day . All NFR's without human interaction should hold true for 24X7. Whereas, NFR with human interaction should be applicable from 9:00 - 17:00 hours. This applies to all NFR's unless explicitly stated otherwise. |  |  |
| Availability_and_reliability | Availability | Non-Execution | Maintenance tasks that cannot be undertaken without impacting performance or availability will be agreed and documented.  |  |  |
| Availability_and_reliability | Availability | Execution | Maintenance tasks that can be undertaken without impacting performance or availability will be documented. |  |  |
| Availability_and_reliability | Resiliency / failback | Execution | It must be possible to reintroduce a failed component into the infrastructure, without service interruption, after appropriate diagnostic and fault repair actions have been taken. |  |  |
| Availability_and_reliability | Recovery | Non-Execution | In the event of reverting to a backup, all data across the system must remain integral. |  |  |
| Availability_and_reliability | Resiliency / failback | Execution | In the event of a timeout of an interface link the solution must have the ability to automatically re-send data with no loss or duplication. |  |  |
| Availability_and_reliability | Resiliency / failback | Execution | In the event of a failure of an interface link the solution must have the ability to automatically re-send data with no loss or duplication. |  |  |
| Availability_and_reliability | Resiliency / failback | Execution | In the event of a component or server failure, an automatic attempt to resume operations must be made. |  |  |
| Availability_and_reliability | Resiliency / failback | Execution | In the event of a component or server failure a log / system event must be created. |  |  |
| Availability_and_reliability | Resiliency / failback | Execution | In the event of a application and web server failure, throughput and performance must meet at least 50% of service targets. This assumes a load balanced active-active application and web server configuration. |  |  |
| Availability_and_reliability | Resiliency / failback | Non-Execution | In the event of a component/server failure, data loss must be limited to 6 hours. |  |  |
| Availability_and_reliability | Disaster Recovery | Non-Execution | DR process must ensure no more than 6 hours of data loss covering the End to End Solution. |  |  |
| Availability_and_reliability | Disaster Recovery | Execution | Benchmark service levels for DR Site Performance monitoring to be 50% greater than for Primary site operations when a DR is invoked. |  |  |
| Availability_and_reliability | Availability | Non-Execution | Availability target will be 99.99% (this excludes any scheduled downtime). |  |  |
| Availability_and_reliability | Disaster Recovery | Non-Execution | All other Service Levels and the Service Credit scheme remain in force whilst operating from the Secondary Site. |  |  |
| Performance | Txn Performance | Execution | Updated batch jobs must not extend execution times by more than 5%. Updated batch job mean that job has undergone some changes . |  |  |
| Performance | Capacity | Non-Execution | Unless otherwise specifically documented within the business requirements, there must be sufficient capacity within the developed solution received at handover to meet the levels of service specified within the service specification for a minimum of 12 months following implementation into the production environment. |  |  |
| Performance | Capacity | Non-Execution | Unless otherwise specifically documented within the business requirements, there must be sufficient capacity within the developed solution received at handover to meet the levels of service specified within the service specification for a minimum of 24 months following implementation into the production environment. |  |  |
| Performance | Txn Performance | Execution | Unaffected batch jobs must not extend execution times by more than 1% within in first 24 months. |  |  |
| Performance | Scalability | Non-Execution | There must be nothing inherenet in the solution that will prevent future vertical scaling. |  |  |
| Performance | Scalability | Non-Execution | There must be nothing inherenet in the solution that will prevent future horizontal scaling. In term of system functionality, system should be scalable to add new functions/features as well. |  |  |
| Performance | Throughput | Execution | The solution must be capable of supporting processing of the volumes as outlined within the technical solution design document for the project in question in the areas of:
 - Seasonal Variations
 - Peak volumes
 - Concurrent Users |  |  |
| Performance | Txn Performance | Execution | The batch system must not exceed the batch-processing window available when executed with production data volumes. The schedule must contain an agreed-on level of contingency for unplanned activities (job failures). |  |  |
| Performance | Soak | Execution | The architecture must demonstrate that anticipated average and peak transmission levels can be delivered successfully and sustained. |  |  |
| Performance | Txn Performance | Execution |  Operation manual should document batch job wise for e.g the <name> batch component must process <number> <items> within <time> hours. |  |  |
| Performance | Txn Performance | Execution | Performance Testing will be measured and conducted against the agreed SLA and TVM. |  |  |
| Performance | Txn Performance | Execution | Performance Testing must be carried out on all types of transaction types to conform to response time SLA's. |  |  |
| Performance | Soak | Execution | Performance must not degrade below NFR Performance Targets over an 24 hour window when system is executing against typical daily load. |  |  |
| Performance | Capacity | Execution | Performance and capacity planning should account for generation of business audit data and security audit data with minimum impact on system performance.  |  |  |
| Performance | Txn Performance | Execution | Overnight housekeeping batch execution processing must be optimized to minimize execution time and maximize use of available system resources. |  |  |
| Performance | Concurrency | Execution | Online solution must support <NUMBER> concurrent users in a peak hour/day as agreed with the Authority
Please put the expected no. of users |  |  |
| Performance | Txn Performance | Execution | Online performance of replaced <txn> must not be degraded or even perform better(where this transaction is on a system that is being replaced by this implementation). This degraded applies to both response time and throughput. |  |  |
| Performance | Txn Performance | Execution | Online performance of new txn must complete within 3 seconds in 95 % of cases. |  |  |
| Performance | Txn Performance | Execution | Online performance of existing (updated) <txn> must not be degraded and even perform better (where this transactions is on the system that is being upgraded). This degraded applies to both response time and throughput. |  |  |
| Performance | Txn Performance | Execution | Online performance levels are to be achieved against a fully loaded System. |  |  |
| Performance | Txn Performance | Execution | Online perf to be measured end to end (from user performing operation to txn complete (e.g. displayed) |  |  |
| Performance | Capacity | Execution | Concurrent execution - servers not to exceed an average of 75% CPU utilisation under peak load. |  |  |
| Performance | Capacity | Execution | Concurrent execution - servers not to exceed an average 75% memory utilisation under peak load. |  |  |
| Performance | Txn Performance | Execution | batch jobs that replace others should not extend execution time by <10%> |  |  |
| Performance | Txn Performance | Execution | All batch components that constitue the overnight batch window must complete within 6 hours whilst adhering to batch schedule dependencies. |  |  |
| Performance | Capacity | Non-Execution | A capacity plan will be delivered for the release. |  |  |
| Security |  | Non-Execution | Unnecessary system utilities should be removed from the application and database servers. |  |  |
| Security | Application and access control | Execution | The Solution must provide a protection mechanism if the authentication process is repeated unsuccessfully more than a specified number of times. |  |  |
| Security | Application and access control | Non-Execution | The Solution must not provide a mechanism to establish access to the Solution other than the approved and documented login process. |  |  |
| Security | Application and access control | Execution | The Solution must ensure that only authorised Users with the required access rights are given access to functionality or data and shall manage access control. |  |  |
| Security | Application and access control | Execution | The removal of any component for maintenance purposes (if applicable) must not compromise the security of the infrastructure. |  |  |
| Security | Application and access control | Execution | The removal of any component for maintenance purposes (if applicable) must not compromise the security of the infrastructure. |  |  |
| Security | Application and access control | Execution | The recovery mechanisms must ensure that the security of the Solution is not degraded. |  |  |
| Security | Application and access control | Execution | The new/enhanced system must enable Service Delivery to undertake user access management. |  |  |
| Security | Compliance | Non-Execution | The new/enhanced system must conform to existing Internet access policies. |  |  |
| Security | Network | Execution | The new/enhanced system must be installed in a manner whereby all network connections must be secure. |  |  |
| Security | Application and access control | Execution | The new/enhanced system must administer passwords and user IDs. |  |  |
| Security | Network | Execution | The methods used to access the Solution must not transmit or store security-related data that could be used for un-authorised access. |  |  |
| Security | Physical | Non-Execution | The IT Solution Operator must report in a timely manner any loss or corruption of data to the Business Information Owner. |  |  |
| Security | Application and access control | Execution | The existing security arrangements must not be compromised by the new Solution. |  |  |
| Security | Application and access control | Non-Execution | System User identifications must be unique and shall not change during the 'lifetime' of the User. |  |  |
| Security | Application and access control | Non-Execution | Security of the Solution must not impact the security of connected systems. |  |  |
| Security | Application and access control | Execution | Security of the Solution must be controlled by MOF security department. |  |  |
| Security | Network | Execution | Remote diagnostic ports must be securiely managed and controlled. |  |  |
| Security | Network | Execution | Procedures to re-introduce failed components must not compromise the security features of the Solution. |  |  |
| Security | Operating system | Non-Execution | Patches and upgrades must be applied in a timely manner if the Solution would otherwise suffer a security degradation. |  |  |
| Security | Application and access control | Non-Execution | Passwords must be one-way encrypted and must not be embedded in application code. |  |  |
| Security | Network | Execution | Network security must be implemented to separate the Solution Components into separate domains. |  |  |
| Security | Network | Execution | Multiple component failures, where the Solution continues to offer a service, must not compromise security. |  |  |
| Security | Resiliency / failback | Execution | Multiple component failures, where the infrastructure continues to offer service, must not compromise system security. |  |  |
| Security | Application and access control | Execution | It must not be possible to disable the security features of the Solution other than where documented and agreed. |  |  |
| Security | Application and access control | Execution | It must not be possible to disable the audit and alarm functions of systems other than in the circumstances defined in the relevant Detailed Design Documentation. |  |  |
| Security | Physical | Execution | Default vendor passwords should be changed following installation of software |  |  |
| Security | Application and access control | Non-Execution | Connectivity with External systems must be documented, agreed and controlled. |  |  |
| Security | Application and access control | Execution | Component failure must not compromise the security features of the architecture and solution. |  |  |
| Security | Application and access control | Non-Execution | Backups of the solution must maintain the same security as the Solution. |  |  |
| Security | Application and access control | Execution | Authentication should be managed through the MOF's Active Directoy through the Active Identity corporate solution. |  |  |
| Security | Compliance | Non-Execution | Any divergence from the client's security policy and high-level standards, and associated operating system/platform specific standards, must be documented in a gap analysis and approved by the client and Service Delivery security managers before acceptance. |  |  |
| Security | Compliance | Non-Execution | Any divergence from the client's security policy and high-level standards, and associated operating system/platform specific standards, must be documented in a gap analysis and approved by <team / people> before acceptance. |  |  |
| Security | Compliance | Non-Execution | Anti-virus software must be installed, with procedures for updating, on the Solution. |  |  |
| Security | Application and access control | Execution | All logon attempts must be recorded within a secure log and converted into a readable report format. |  |  |
| Security | Application and access control | Execution | All services that are exposed by the solution (including front-end, middle-tier or back-end services) must only allow authorised access. User provisioning process to be deifned for approval.  |  |  |
| Security | Application and access control | Execution | Access to the System, and User profiles within the System, for Users shall be login and not desktop specific. |  |  |
| Security | Application and access control | Execution | Access to the Solution shall be restricted so that any one log-in identity may have only 1 open session at any one time. |  |  |
| Security | Sensitive data proctection | Execution | Sensitive stored information (example: authentication verification data) should be encrypted on the server side including database encryption. Uploaded file should also be encrypted from the customers. All files should be uploaded with digital signatures. |  |  |
| Security | Sensitive data proctection | Execution | Auto complete features on forms expected to contain sensitive information, including authentication should be disabled.Application validates and restricts input, allowing only those data types that are known to be correct |  |  |
| Security | Sensitive data proctection | Execution | Disable client-side caching on pages containing sensitive information. |  |  |
| Security | Sensitive data proctection | Execution | If the application provides a capability to users to upload files, files should be scanned by antivirus scanners to prevent upload of known malicious content.. |  |  |
| Security | Application and access control | Execution | All communications with internal and external systems should be done through secure https protocol. |  |  |
| Security | Application and access control | Execution | Implement two factor authentications by introducing Abhser OTP or at least rebuild custom OTP service.  |  |  |
| Security | Application and access control | Execution | System should able to detect multiple login attempts are made with wrong credentials and should disable the user account. User account should be inactive after 30 mins of user inactiveness. |  |  |
| Security | Application and access control | Execution | All login attempts must be recorded within a secure log and converted into a readable report format. |  |  |
| Security | Sensitive data proctection | Execution | There should be End User data privacy agreement for users to understand the data security policy. |  |  |
| Security | Application and access control | Execution | Access to Data and/or Information Systems should be reauthenticated after a period of inactivity |  |  |
| Security | Application and access control | Execution | Where username and password authentication is employed, passwords should be managed according to the Guidelines for Password Management Policy |  |  |
| Security | Application and access control | Execution | Access to Data and/or Information Systems should be authorized by a Data Steward or a delegate prior to provisioning |  |  |
| Security | Application and access control | Execution | Access to Data and/or Information Systems should be authorized based on a business need  |  |  |
| Security | Application and access control | Execution | Access to Data and/or Information Systems should be based on the principle of least privilege |  |  |
| Security | Application and access control | Execution | Access to Data should be reviewed and reauthorized by a Data Steward or a delegate on a periodic basis |  |  |
| Security | Application and access control | Execution | Access is promptly revoked when it should be no longer necessary to perform authorized job responsibilities |  |  |
| Security | Application and access control | Execution | Access logs should be reviewed on a periodic basis for security events |  |  |
| Security | Application and access control | Execution | Access logs should be protected against tampering |  |  |
| Security | Session Management | Execution | Application sessions are uniquely associated with an individual or system |  |  |
| Security | Session Management | Execution | Session identifiers should be generated in a manner that makes them difficult to guess |  |  |
| Security | Session Management | Execution | Session identifiers should be regenerated following a change in the access profile of a user or system |  |  |
| Security | Session Management | Execution | Active sessions timeout should be configured after a period of inactivity |  |  |
| Security | HTTP/S Traffice Inspection/WAF | Execution | HTTP/S traffice should be filtered and monitored to and from web application to protect against malicious attempts to compromise the system or exfiltration of data. |  |  |
| Security | DDOS | Execution | A denial of services protection should be in place. |  |  |

---

## EA Requirements

| SR# | Execution / Non-Execution | EA Requirements | Vendor Response |
|---:|---|---|---|
| 1 |  | Proposed solution should be based on "Commercial Off the shelf package" (COTS) and should be globally recognized product by Gartner or IDC. |  |
| 2 |  | The product should have strong partner network in KSA and support services |  |
| 3 |  | Product should be installed as On-Prem solution only and as option it should offer both perpetual and subscription based license model.  |  |
| 4 |  | The tool should able to support latest versions of MS Windows and MS SQL server as database. |  |
| 5 |  | The tool should also able to expose its results as API to be consumed by other platforms such MoF DWH platform. |  |
| 6 |  | Integration with other systems in MOF through Enterprise Service Bus (ESB). |  |
| 7 |  | Product should support SSO through integration with MOF IDM and AD platforms. |  |

---

## Categories

| Categories | Sub cat | Description | Count |
|---|---|---|---|
| Security | Physical | The ability of the Solution to protect itself from physical security attacks. | 6 |
| Security | Network | The ability of the Solution to protect itself from security threats from a machine on the network. | 9 |
| Security | Operating system | TBC | 4 |
| Availability_and_reliability |  |  | 16 |
| Security | Application and access control | The ability of the Solution to control authorised access and prevent unauthorised access. | 13 |
| Security | Non-execution | TBC | 3 |
| Technical_Integration_Proving |  |  | 3 |
| Security | Compliance | The ability of the Solution to adhere to industry standards as well as internal requirements | 1 |
| Other |  |  | 1 |
| Performance | Txn Performance | The ability of the Solution to provide feedback to user requests within a specified time period. | 0 |
| Performance | Concurrency | The ability of the Solution to support a number of concurrent users or processes. | 0 |
| Performance | Soak | The ability of the Solution to support user and/or transaction volumes of an extended time period. | 0 |
| Performance | Stress | The ability of the Solution to support user and/or transaction volumes higher than expected. | 0 |
| Performance | Storm | The ability of the Solution to continue functioning following the outage of an internal or external component that causes a backlog of transactions. | 0 |
| Performance | Throughput | The ability of the Solution to support a given number of transactions over a given unit of time. | 0 |
| Performance | Scalability | The ability of the Solution to be expanded beyond its existing capacity. | 0 |
| Performance | Capacity | The ability of the Solution to support a given number of transactions over a cumulative period of time (disk space over years). | 0 |
| Performance | Non-execution | TBC | 0 |
| Availability_and_reliability | Availability | Defining the time scales within which the Solution will be able to support user transactions. | 0 |
| Availability_and_reliability | Resiliency / failback | The ability of the Solution to automatically maintain operating in the event of a failure to a single component or server. | 0 |
| Availability_and_reliability | Recovery | The ability of the Solution to support being manually restored to a previously known state in time. | 0 |
| Availability_and_reliability | Disaster Recovery | The ability of the Solution to handle the a failure of the primary operating site. | 0 |
| Operability | Re-runability | The ability of a process (typically batch job) to be re-run. The definition of re-running is that a process is started again in the same batch window after it has completed successfully. | 0 |
| Operability | Restartability | The ability of a process (typically batch job) to be re-start. The definition of re-starting is that a process is started again in the same batch window when it failed. | 0 |
| Operability | System clock | The ability of the Solution to maintain a consistent and accurate system clock. | 0 |
| Operability | Scheduling | The ability to control the execution of processes and/or jobs within the Solution. | 0 |
| Operability | Application alerting and error handling | The ability of the Solution to raise alerts in the event of errors. | 0 |
| Operability | Event Management | The ability of the Solution to raise, transport and process incidents and events raised within the Solution to a resolution / manual input required. | 0 |
| Operability | System Startup / shutdown | The ability to consistently and repeatably start and stop the Solution. | 0 |
| Operability | Archiving and record retention | The ability of the Solution to store historical data for a pre-determined amount of time (typically years). | 0 |
| Operability | Operation Acceptance | TBC | 0 |
| Operability | Monitoring and Logging | The ability of the Solution to capture and store the events and alerts raised within the Solution. | 0 |
| Operability | Backup and restore | The ability to create another copy of the Solution's data files, as well as the ability to return to that point in time in the future. | 0 |
| Operability | Deployment | The ability to consistently and repeatably deploy code and configuration to the target Solution environment. | 0 |
| Operability | Non-execution | TBC | 0 |
| Technical_Integration_Proving | Internal Compatibility | The ability of the Solution to operate on the suggested versions of all elements in the hardware and software stack. | 0 |
| Technical_Integration_Proving | External Compatibility | The ability of the Solution to integrate with external components and systems at a technical level. | 0 |
| Technical_Integration_Proving | Non-execution | TBC | 0 |
| Other | Static Test | The ability of the Solution's development to meet checks applied against it. | 0 |

