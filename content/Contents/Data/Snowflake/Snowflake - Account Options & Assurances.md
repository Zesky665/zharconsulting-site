---
dg-publish: true
tags:
  - Snowflake
---

## Guidance Questions

1. What are some industry compliance standards Snowflake has been certified in or awarded?
	1. **Global**:
		1. ISO-27001 - International Organization for Standardization, certificate for establishing, implementing, maintaining, and continually improving an information security management systems.
			1. ISO-27017 - 2015 version
			2. ISO-27018 - 2019 version
		2. SOC 1 Type II - independent auditor assessment of security controls. 
		3. SOC 2 Type II - independent auditor assessment of security controls. 
	2. **U.S. Government**:
		1. **CJIS** (Criminal Justice Information Services) - compliance for Public Sector customers, making sure they are in compliance with the  [FBI’s Criminal Justice Information Services (CJIS) Security Policy](https://le.fbi.gov/cjis-division-resources/cjis-security-policy-resource-center)
		2. **FedRAMP** (Federal Risk and Authorization Management Program) - compliance for security standards for cloud technologies used by federal agencies. 
		3. **ITAR** (International Traffic in Arms Regulations) - compliance standard for companies in the defense sector, which controls and restricts access to and export of military and defense articles, services and related technologies. 
			1. Companies which require **ITAR** compliance must purchase Business Critical edition or higher.
			2. Only available with vetted providers, employees and contractors in [SnowGov regions](https://docs.snowflake.com/en/user-guide/intro-regions.html#label-us-gov-regions). 
		4. **StateRAMP** - is a 501(c)6 nonprofit that offers cloud security certifications for:
			1. State and local government.
			2. Public education institutions.
			3. Special districts. 
	3. **Healthcare and Life Science**:
		1. **HITRUST CSF** (Health Information trust Alliance Common Security Framework) - serves to unify security standards based on US federal law (HIPPA & HITECH), state specific laws and other industry standards into a single comprehensive standard for information security in the healthcare industry. 
	4. **Regulatory Compliance**:
		1. **PCI-DSS** (Payment Card Industry Data Security Standards) - security standard for merchants, service providers and financial institutions building and deploying payment solutions and products. 
	5. **Regional -- Australia**:
		1. **IRAP (Protected)** (Infosec Registered Assessors Program) -  is a programs governed by the Australian Signals Directorate (ASD) which aims to certify cyber security professionals to provide relevant services to Australian Government and industry.  
	6. [Source](https://docs.snowflake.com/en/user-guide/intro-compliance))
2. What choices are available when choosing a cloud platform provider for your Snowflake account?
	1. Amazon Web Services (AWS).
	2. Microsoft Azure (Azure).
	3. Google Cloud Services (GCP).
	4. [Source](https://docs.snowflake.com/en/user-guide/intro-cloud-platforms)
3. What are regions and availability zones and how are they affected by your choice of a cloud platform provider?
	1. Regions are geographical subdivisions where data centers are physically located, you can choose regions to fit your compliance, latency and cost needs.  
	2. The available regions are determined by the cloud platform provider.
	3. [Source](https://docs.snowflake.com/en/user-guide/intro-regions)
4. What are the benefits for moving from Snowflake's Standard Edition to Enterprise? What are the benefits for moving from Enterprise to Business Critical?
	1. Standard to Enterprise: 
		1. 90 Time Travel.
		2. Rekeying.
		3. Row and Column level security.
		4. Object tagging.
		5. Classifying sensitive data.
		6. Auditing user access history.
		7. Query acceleration - parallel processing portions of eligible queries.
		8. Search Optimization - point lookup queries with automatic maintenance.
		9. Materialized Views - with automatic maintenance of results.
	2. Enterprise to Business Critical: 
		1. Failover and failback between snowflake accounts for business continuity and disaster recovery. 
		2. Redirecting Client Connections between Snowflake accounts for business continuity and disaster recovery. 
		3. Tri-Secret Secure. 
		4. Support for Private Connections.
		5. Support for internal Private Link(AWS&Azure).
		6. PHI DSS compliance and support.
		7. FedRAMP compliance and support.
		8. ITAR compliance and support.
		9. IRAP compliance and support.
		10. HIPAA and HITRUST CSF compliance and support.
5. What factors should be considered when choosing the Geographic Region for your account?
	1. Infrastructure available in the region (AWS, Azure, GCP).
	2. Region of the point of service (to reduce latency).
	3. Number of availability zones within a region.
	4. Compliance, as in being required to host data in a certain jurisdiction. 
	5. [Source](https://docs.snowflake.com/en/user-guide/intro-cloud-platforms)

## Quiz:

1. Is Snowflake HIPPA compliant?
	1. Yes
2. What are the names of the three Snowflake Editions offered when signing up for a trail account?
	1. Standard, Enterprise, Business Critical.
3. Which Snowflake Editions automatically store data in an encrypted state?
	1. Standard, Enterprise, Business Critical.
4. Which of the following industry compliance standards has Snowflake been audited and certified for?
	1. SOC 1
	2. SOC Type 2
	3. PCI DSS
	4. FedRAMP
	5. HIPPA
5. What setting up a new Snowflake Account, what steps or choices must the enrollee complete?
	1. Choose a Cloud Infrastructure Provider
	2. Choose a Snowflake Edition
	3. Choose a Geographic Deployment Region
6. Which cloud infrastructure providers are available as the cloud platform for Snowflake Accounts?
	1. AWS
	2. Azure
	3. Google Cloud Platform (GCP)
7. When signing up for a new Snowflake Account, enrolles first choose a cloud infrastructure provider and then a region. When choosing Azure, an enrollee might then choose the "Australia East" region. When choosing AWS, the "Australia East" region is not listed. Instead, a region called "Asia Pacific (Sydney)" is listed. Why?
	1. each cloud provider maintains its own regions and names the regions as they see fit.
8. When choosing a geographic deployment region, what factors might an enrollee consider?
	1. Proximity to the point of service.
	2. Number of availability zones within a region.
9. The following questions have to do cloud platforms. Select all the statements that are true.
	1. A company can use more than one cloud infrastructure provider by setting up several Snowflake accounts.
	2. A company can have its data stored in more than one geographical region by setting up several Snowflake account.
	3. A company can use a combination of data sharing and replication to distribute data to various regions and cloud platforms. 

