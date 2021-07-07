# Cloudformation Templates for Ranger Self Installing and Integrating with AWS EMR Cluster and AD/LDAP

---

Author：Laurence Geng　　｜　　Created Date：2020-11-21　　｜　　Updated Date：2020-11-21

---


---

Update@2021/07/07

1. If EMR is based on Glue Data Catalog, Ranger still CAN work with it!
2. By now, this solution works with all EMR 6.X (6.0.0 - 6.3.0)

---

This project is a series of aws cloudformation templates which are used to install ranger and integrate a AWS EMR cluster and a windows AD or Open LDAP server as authentication channel. There is another closely related project: **[ranger-emr-cli-installer](https://github.com/bluishglc/ranger-emr-cli-installer)** which does the same job via command line. The two projects are very close, but can work independently，you can pick anyone as you wish.

## 1. Ranger Introduction

Let’s check out Ranger's architecture:

![ranger-architecture](https://user-images.githubusercontent.com/5539582/99872048-f0c24480-2c19-11eb-8c0f-43df2552837c.png)

Ranger has 5 parts:

1. Ranger Admin Service
2. Ranger UserSync Service
3. A Backend RDB for Storing User's Authorization
4. A Solr Server for Storing Audit Log
5. A Series of Plugins for Big Data Components/Services

Besides above, there are 2 external dependencies For Ranger to integrate:

6. A Windows AD or Open LDAD Server as Authentication Channel
7. A Hadoop (AWS EMR) Cluster to Be Managed by Ranger

So, a fully Ranger installation will cover following jobs:

1. Install JDK (Required by Ranger Admin and Solr)
2. Install MySQL (As Ranger Backend RDB)
3. Install Solr (As Ranger Audit Store)
4. Install Ranger Admin (and Integrate with AD/LDAP Server)
5. Install Ranger UserSync (and Integrate with AD/LDAP Server)
6. Install Ranger Plugins (i.e. HDFS, Hive, HBase and so on)

## 2. Cloud Formation Templates Introduction

There are 4 templates files in projects, here is difference:

Template Files|Integrated Authentication Service|AWS Region
--------------|---------------------------------|----------
ranger-emr-ad.template|Windows AD|Global Regions
ranger-emr-ad.cn.template|Windows AD|China Regions
ranger-emr-ldap.template|Open LDAP|Global Regions
ranger-emr-ldap.cn.template|Open LDAP|China Regions

## 3. Prerequisites

Before installing, make sure following items are ready or done:

1. Make sure the EMR cluster is in waiting status, no any job is running
2. Upload your private SSH key (the pem file) to private S3 bucket, for example `/my-bucket/my-key.pem`
3. It's recommanded to explore users and groups on Windows AD or Open LDAP via GUI tool, for example LDAP Admin, so as to detemine AD/LDAP related parameters
4. Check network connectivities among Ranger server, Windows AD or Open LDAP server and EMR nodes

## 4. Usage Examples

To explain how to use this tool, assume we have following environment:

**A Windows AD Server:**

Key|Value
---------:|:-----
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; IP|10.0.0.194
Domain Name|corp.emr.local
Base DN|cn=users,dc=corp,dc=emr,dc=local
Bind DN|cn=ranger,ou=service accounts,dc=example,dc=com
Bind DN Password|Admin1234!
User Object Class|person

**An Open LDAP Server:**

Key|Value
---------:|:-----
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; IP|10.0.0.41
Base DN|dc=example,dc=com
Bind DN|cn=ranger,ou=service accounts,dc=example,dc=com
Bind DN Password|Admin1234!
User DN Pattern|uid={0},dc=example,dc=com
Bind Group Search Filter|(member=uid={0},dc=example,dc=com)
User Object Class|inetOrgPerson


**A Multi-Master EMR Cluster:**

Node|IP
---:|:---
&emsp;&emsp;&emsp;&emsp;&emsp;Master Nodes|10.0.0.177,10.0.0.199,10.0.0.21
Core Nodes|10.0.0.114,10.0.0.136


**A Normal EMR Cluster:**

Node|IP
---:|:---
&emsp;&emsp;&emsp;&emsp;&emsp;Master Nodes|10.0.0.177,10.0.0.199,10.0.0.21
Core Nodes|10.0.0.114,10.0.0.136

### 4.1. Install Ranger + Integrate a Window AD Server + Integrate A Multi-Master EMR Cluster

The following diagram illustrates what this example will do:

![example1](https://user-images.githubusercontent.com/5539582/99872053-fc157000-2c19-11eb-94c4-ee36ed30ce14.png)

The following cloudformation template configuration will finish this job:

![cfn-example-1](https://user-images.githubusercontent.com/5539582/99896184-1b1f0b00-2cc9-11eb-9def-ef7bf06ef14b.png)

### 4.2. Install Ranger + Integrate a Open LDAP Server + Integrate A Multi-Master EMR Cluster

The following diagram illustrates what this example will do:

![example3](https://user-images.githubusercontent.com/5539582/99872059-059ed800-2c1a-11eb-82e7-da5e21949d44.png)

The following cloudformation template configuration will finish this job:

![cfn-example-2](https://user-images.githubusercontent.com/5539582/99896185-1e19fb80-2cc9-11eb-8183-592ebf9f0a36.png)


## 5. Versions & Compatibility

The following is Ranger and EMR version compatibility form:

&nbsp;|Ranger 1.X|Ranger 2.x
---|---|---
EMR 5.X|Y|N
EMR 6.X|N|Y

For Ranger 1, it works with Hadoop 2, for Ranger 2, it works with Hadoop 3, **This project is developed against Ranger 2.1.0, so now, it can only integrate EMR 6.X.** For Ranger 1.2 + EMR 5.X, it is to be developed in the next according to demands.
