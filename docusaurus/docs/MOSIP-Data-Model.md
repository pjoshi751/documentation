---
id: 
title: 
---
This section contains details related to MOSIP data model design.The section also provides the data dictionary of the tables and columns defined by MOSIP databases.

## Data Model Considerations

* **Meaningful Naming:** DB objects that are being created will have a meaningful naming.
* **Flexible model:** No business rules are set at database level other than few mapping data. Most of the business logic is applied at application layer.
* **Database specific features:** Use of DB specific features like defaults, DB sequences, identify fields are not used
* **No business logic at DB:** No business logic implemented at database level other than PK, UK, FKs. 
* **Data Security:** Individual and security related information is encrypted
## Data Model

Databases inventory in MOSIP

|Sl No|Database Name|Description|Data Model|Data Dictionary|
|---------|---------|------------|----------|-----------|
|1|mosip_kernel|Kernel database store security key details, data related to kernel services like sync process, OTP, etc.|<div>[DBM File](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_sources/mosip_kernel.dbm)</div> <div>[Image File ](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_images/mosip_kernel.png)</div>|<div>[mosip_kernel_dd.xlsx ](https://github.com/mosip/mosip-platform/tree/master/design/data_model/mosip_kernel_dd.xlsx)</div>|
|2|mosip_master|All the master data defined by a country / organization is maintained in mosip_master database. |<div>[DBM File](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_sources/mosip_master.dbm)</div> <div>[Image File ](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_images/mosip_master.png)</div>|<div>[mosip_master_dd.xlsx ](https://github.com/mosip/mosip-platform/tree/master/design/data_model/mosip_master_dd.xlsx)</div>|
|3|mosip_idrepo|ID repository database stores all the data related to an individual for which an UIN is generated.|<div>[DBM File](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_sources/mosip_idrepo.dbm)</div> <div>[Image File ](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_images/mosip_idrepo.png)</div>|<div>[mosip_idrepo_dd.xlsx](https://github.com/mosip/mosip-platform/tree/master/design/data_model/mosip_idrepo_dd.xlsx)</div>|
|4|mosip_prereg|Pre-registration database to store the data that is captured as part of pre-registration process and appointments booking.|<div>[DBM File](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_sources/mosip_prereg.dbm)</div> <div>[Image File ](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_images/mosip_prereg.png)</div>|<div>[mosip_prereg_dd.xlsx](https://github.com/mosip/mosip-platform/tree/master/design/data_model/mosip_prereg_dd.xlsx)</div>|
|5|mosip_reg|Registration client database to capture registration related data. The needed data from MOSIP system will be synched with this database.|<div>[DBM File](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_sources/mosip_reg.dbm)</div> <div>[Image File ](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_images/mosip_reg.png)</div>|<div>[mosip_reg_dd.xlsx](https://github.com/mosip/mosip-platform/tree/master/design/data_model/mosip_reg_dd.xlsx)</div>|
|6|mosip_regprc|The data related to Registration process flows and transaction will be maintained in this database.|<div>[DBM File](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_sources/mosip_regprc.dbm)</div> <div>[Image File ](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_images/mosip_regprc.png)</div>|<div>[mosip_regprc_dd.xlsx](https://github.com/mosip/mosip-platform/tree/master/design/data_model/mosip_regprc_dd.xlsx)</div>|
|7|mosip_ida|ID Authentication related requests, transactions will be stored in this database|<div>[DBM File](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_sources/mosip_ida.dbm)</div> <div>[Image File ](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_images/mosip_ida.png)</div>|<div>[mosip_ida_dd.xlsx](https://github.com/mosip/mosip-platform/tree/master/design/data_model/mosip_ida_dd.xlsx)</div>|
|8|mosip_audit|Audit related logs collected from all modules are stored in this database|<div>[DBM File](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_sources/mosip_audit.dbm)</div> <div>[Image File ](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_images/mosip_audit.png)</div>|<div>[mosip_audit_dd.xlsx](https://github.com/mosip/mosip-platform/tree/master/design/data_model/mosip_audit_dd.xlsx)</div>|
|9|mosip_iam|The users and roles management related data are stored in this database|<div>[DBM File](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_sources/mosip_iam.dbm)</div> <div>[Image File ](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_images/mosip_iam.png)</div>|<div>[mosip_iam_dd.xlsx](https://github.com/mosip/mosip-platform/tree/master/design/data_model/mosip_iam_dd.xlsx)</div>|
|10|mosip_idmap|Database to store and manage all the data related to mapping between various IDs, like vid with UIN of an individual|<div>[DBM File](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_sources/mosip_idmap.dbm)</div> <div>[Image File ](https://github.com/mosip/mosip-platform/tree/master/design/data_model/_images/mosip_idmap.png)</div>|<div>[mosip_idmap_dd.xlsx](https://github.com/mosip/mosip-platform/tree/master/design/data_model/mosip_idmap_dd.xlsx)</div>|



