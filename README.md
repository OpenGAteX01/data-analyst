Portfolio project 1: Calcualtion of students required to withdraw
the raw dataset i used as a sample had the following columns:

    StudentID: Unique identifier for each student.
    AcademicLevel: The academic level of the student (e.g., Graduate).
    Term: The term in which the data is collected (e.g., 2024Q1).
    CGPA: The cumulative GPA of the student.
    NumberOfFails: The number of courses the student has failed.
    AcademicStanding: The current academic standing (e.g., Good Academic Standing, Academic Probation).
    ActionsTaken: Actions suggested or taken for the student (e.g., Withdraw for 1 year, Continue as normal).
This dataset was stored in S3 and cleeaned and structured through AWS Glue-Databrew
 the following transformations were applied to the dataset in the Graduate students project-recipe:

    Change type of CGPA to Float: This ensures that the CGPA values are stored as floating-point numbers for more accurate numerical computations.

    Delete column ActionsTaken: The column related to actions suggested or taken (e.g., Withdraw for 1 year) was removed.

    Delete column AcademicLevel: The academic level (e.g., Graduate) column was also removed.

a similar process was used on all 3 datasets
the data was then checked for sensitive information and for data quality in AWS visual ETL
   ![image](https://github.com/user-attachments/assets/9a4ddf0c-a58d-42b7-8318-8363e3afde9a)
here’s a breakdown of the operations that have been applied:

    Data Sources:
        Three data sources from S3 buckets (Q1, Q2, and Q3) are used as inputs.

    Transformations:
        Change Schema: The schema for the Q1 and Q3 data is modified.
        Rename Keys for Join: Keys from Q2 and Q1 datasets are renamed to prepare for the join operation.
        Join Transformations: The Q1, Q2, and Q3 datasets are joined sequentially to create a consolidated dataset.

    Sensitive Data Detection:
        The combined dataset is passed through a transformation to Detect Sensitive Data, which likely checks for personally identifiable information (PII) or other sensitive content.

    Data Quality Evaluation:
        The next step evaluates the quality of the data through a Data Quality Evaluation transformation.

    Row-Level Outcomes:
        A transformation checks specific row-level outcomes, possibly flagging any anomalies or quality issues at the row level.

    Conditional Routing:
        Based on the data quality and outcomes, the data is conditionally routed. Two branches exist:
            Default Group: If certain conditions are not met, data may be processed here.
            Passed: If the data passes all checks, it continues to the next step.

    Final Transformations:
        Change Schema: Schema transformations are applied again before the data is sent to an Amazon S3 bucket as the final output.
     


This flow demonstrates how the system is handling the data, detecting sensitive information, checking data quality, and routing the data based on evaluation results before storing it back in S3.
        ![image](https://github.com/user-attachments/assets/9456723d-7dc8-4a08-896b-0f4da412c721)
steps taken to determine students who are "Required to Withdraw" based on their GPA across three terms. Here's a breakdown:

    Data Sources:
        Q1, Q2, and Q3 data are pulled from S3 buckets that have been checked for sensitive informationa and data quality issues 

    Transformations:
        Rename Keys for Join and Change Schema: These are preparatory steps to ensure the datasets are ready for joining.
        Join Operations: Q1, Q2, and Q3 data are joined to consolidate the information for each student across three terms.

    Filter Operation:
        After the datasets are joined, a Filter transformation is applied, selecting students who are "Required to Withdraw". This filter is likely based on students who consistently had a GPA below 2.0 for all three terms.

    Schema Change and Final Target:
        The filtered data then goes through another schema transformation before being written back to an S3 bucket as the final output.



This process identifies the students who consistently underperformed (below a 2.0 GPA for all terms) and flags them as required to withdraw.
IAM Usage:

For secure access and role management, IAM (Identity and Access Management) was set up to ensure that each service interacted securely. Below are the main IAM roles and policies that were applied:

    IAM Roles for EC2 Instances:
        EC2 instances used in the project were assigned IAM roles that allowed them to access S3 for reading/writing datasets and to interact with KMS for encryption purposes.
        The roles were configured with the least privilege principle, ensuring that each EC2 instance only had the permissions it needed for the task at hand.

    IAM Policies:
        A policy was created to allow access to specific S3 buckets.
        Another policy granted access to CloudWatch Logs for monitoring the health of instances and tracking actions performed by the ETL pipeline.


KMS Encryption:

Data security was a crucial aspect of this project, and KMS (Key Management Service) was used to encrypt both the data at rest and in transit.

    S3 Bucket Encryption:
        Server-Side Encryption (SSE-KMS) was enabled on the S3 buckets where the datasets were stored. This ensured that all data written to S3 was encrypted using KMS keys.
        KMS keys were also used to encrypt the backup data that was stored in secondary S3 buckets for disaster recovery purposes.

    EBS Volume Encryption:
        The EC2 instances used for processing the data were configured with EBS volumes encrypted using KMS. This ensured that any intermediate data stored on the instance was secure.

    RDS (if applicable):
        If databases were involved in the process, RDS snapshots were also encrypted using the KMS-managed keys.
![image](https://github.com/user-attachments/assets/0dc9a630-2bf5-4e5d-89ad-c2e2cc43ca73)

 /
Replication and Backup Rules:

To ensure redundancy and data availability in case of disaster, S3 Replication and Backup Rules were applied:

    S3 Cross-Region Replication:
        Cross-Region Replication (CRR) was enabled for the S3 buckets storing the data. Data from the primary S3 bucket in the us-east-1 region was replicated to a secondary bucket in eu-west-1.
        This ensures that in the event of a regional outage, the data remains available in a different geographical location.

    Backup Rules via AWS Backup:
        AWS Backup was used to automate backups for the EBS volumes attached to the EC2 instances.
        Backups were scheduled to run daily, and encrypted snapshots were stored in a backup vault.
        Backup retention policies were defined to keep snapshots for 60 days, after which they were automatically deleted to save storage costs.

    Data Lifecycle Management:
        S3 Lifecycle Rules were also implemented to automatically transition older data to lower-cost storage classes (such as S3 Glacier) after a set retention period.
        This process helped manage storage costs effectively without compromising on the availability of historical data.
 /
 VPC Setup:

    VPC Name: Reg-RTW-VPC-Gurfateh-vpc
    VPC ID: vpc-0c72cd8f8b22ff2bd
    IPv4 CIDR: 10.0.0.0/27
    State: Available (indicating that the VPC is active and ready for use).
    DHCP Option Set: Linked to the VPC (for configuring domain name servers and other network settings).
    Main Route Table: rtb-01c6edaa1bf2a510e (determines how traffic is routed in the VPC).

Subnet Setup:

There are multiple subnets in this environment, both public and private:

    Public Subnet:
        Name: Reg-RTW-VPC-Gurfateh-subnet-public1c
        Subnet ID: subnet-01e0d593fff2887e1
        IPv4 CIDR: 10.0.0.0/28
    Private Subnet:
        Name: Reg-RTW-VPC-Gurfateh-subnet-private1a
        Subnet ID: subnet-0b9384605a98eec9b
        IPv4 CIDR: 10.0.0.16/28
![image](https://github.com/user-attachments/assets/d9f30105-ad21-420e-a9b1-11781adc3ae1)

There are additional subnets listed with no specific names, each associated with different CIDR blocks within the range 172.31.x.x.

This configuration indicates that you are using multiple subnets within the VPC for different purposes, such as public-facing instances (web servers) and private instances (databases or internal servers).
 /
 Security Groups Configuration:

In this project, Security Groups were used to control access to the EC2 instances and other resources within the VPC (Virtual Private Cloud). Security Groups act as virtual firewalls that regulate inbound and outbound traffic based on predefined rules.
Key Security Group Configurations:

    Security Group for Web Servers:
        Purpose: This group was assigned to EC2 instances acting as web servers, which required public access.
        Inbound Rules:
            Allowed HTTP traffic (port 80) from anywhere (0.0.0.0/0).
            Allowed HTTPS traffic (port 443) from anywhere (0.0.0.0/0).
            Allowed SSH access (port 22) from a specific trusted IP range to enable secure administration of the instances.
        Outbound Rules:
            All outbound traffic was allowed, enabling the web servers to send requests as needed (for example, sending logs to CloudWatch or fetching updates from external repositories).

    Security Group for Database and Backend Services:
        Purpose: This group was applied to instances hosting internal databases or backend services that did not require public access.
        Inbound Rules:
            Allowed database-specific traffic (e.g., MySQL on port 3306 or PostgreSQL on port 5432) from the internal subnets only (VPC CIDR range).
            Blocked all public access to ensure that the databases were only accessible from within the private subnet.
        Outbound Rules:
            Outbound traffic was restricted to allow communication only with the web server instances or external services like S3 for backups.

    Security Group for NFS (Network File System) Traffic:
        Purpose: This group managed access to a shared file system used by multiple EC2 instances.
        Inbound Rules:
            Allowed NFS traffic on ports 2049 and 111 from other instances in the same VPC.
        Outbound Rules:
            Allowed outbound traffic to all instances in the VPC for shared access to the file system.

General Security Group Best Practices:

    Least Privilege: Only the necessary ports and IP ranges were allowed to minimize exposure to external threats.
    VPC Segmentation: Public-facing and private resources were isolated using separate security groups and subnets to ensure that internal services were protected from external access.

Routing and Subnet Configuration:

Routing within the VPC ensured that traffic was appropriately directed between public and private resources. The routing configuration was critical to maintaining network security and optimizing resource accessibility.
Public Subnet Routing:

    Purpose: Public subnets hosted web servers and any other resources that needed to be accessible over the internet.
    Routing Table Configuration:
        A default route (0.0.0.0/0) was directed to the Internet Gateway (IGW), allowing instances within the public subnet to send and receive traffic from the internet.
        Instances in these subnets had public IPs assigned to them to facilitate direct communication with users.

Private Subnet Routing:

    Purpose: Private subnets were used for backend services, databases, and other internal resources that should not be exposed to the internet.
    Routing Table Configuration:
        No direct access to the Internet Gateway.
        Instances in the private subnet used a NAT Gateway for outbound traffic. This allowed them to make requests to the internet (e.g., for updates or backups) without being directly accessible from the outside.

Security through Network Segmentation:

    Web Servers (Public Subnet): Traffic destined for the web servers was routed through the Internet Gateway. Incoming traffic was filtered by the security group to ensure that only HTTP/HTTPS and SSH connections were allowed.
    Backend Services (Private Subnet): Backend services had no direct access to the internet. They could only communicate with instances within the VPC or make outbound connections using the NAT Gateway, ensuring that sensitive data remained secure.

Key Routing Best Practices:

    Private Subnets for Sensitive Data: Databases and backend services were placed in private subnets to ensure that they were not publicly accessible.
    NAT Gateway: Instances in the private subnet used a NAT Gateway for outbound traffic. This allowed them to download updates or backups without exposing them to inbound internet traffic.
    Internet Gateway: Public-facing instances had a route to the Internet Gateway to allow users to access services hosted on the EC2 instances.

By configuring Security Groups and Routing carefully, the project ensured that resources were securely segmented, public-facing services were protected, and internal services were shielded from unauthorized access. All EC2 instances and S3 buckets were encrypted with KMS to provide an additional layer of security for data at rest.
/.
In this project, the EC2 instances are integral to processing and publishing the list of students required to withdraw based on their academic performance across three terms. The roles of the general server and the web server are particularly important in managing this process efficiently.
General Server: Data Processing

The general server is responsible for handling the backend processing tasks associated with the project. Here's how it operates:

    Instance ID: i-05d80240d368569a0
    Type: t2.micro
    Availability Zone: us-east-1a
    Purpose: This instance is tasked with executing the ETL (Extract, Transform, Load) pipeline, which includes joining datasets from multiple terms, applying transformations, detecting sensitive data, and evaluating data quality.

Once the data is processed and filtered for students with a GPA below 2.0 consistently across three terms, the list of students who are "Required to Withdraw" is generated. This filtered dataset is securely stored in an S3 bucket, encrypted using KMS.

The general server performs these tasks in the private subnet, as it does not need public access. This enhances the security of the data and ensures that the processing is isolated from external threats.
Web Server: Publishing the Withdrawal List

The web server is responsible for publishing the final withdrawal list so that it can be accessed by the necessary administrative teams. Here's how it functions:

    Instance ID: i-004a42015d065ccf9
    Type: t2.micro
    Availability Zone: us-east-1c
    Purpose: This server hosts the front-end application that presents the withdrawal list. After the list is processed by the general server and uploaded to S3, the web server retrieves the data and displays it on a secure web interface for authorized personnel to access.

The web server resides in the public subnet, allowing it to be accessible via the internet. However, access is tightly controlled via security groups that only permit specific IPs to connect via HTTPS, ensuring that the data remains secure while being publicly accessible for the intended administrative staff.
Process Overview:

    Data Processing:
        The general server processes the datasets for Q1, Q2, and Q3, applying the necessary filters and transformations to identify students who are required to withdraw based on their academic standing.

    Data Storage:
        Once the list is generated, it is stored in an encrypted S3 bucket, ensuring that sensitive information is protected.

    Data Publishing:
        The web server retrieves the withdrawal list from the S3 bucket and publishes it on a secure web interface, allowing authorized users to view the list.
        The interface ensures that the appropriate administrative teams can easily access the data, without compromising security.

By separating the responsibilities between the general and web servers, the project ensures that the data processing and publishing phases are efficiently managed. This separation also enhances security by isolating the back-end data processing from public access while maintaining a secure and easily accessible interface for publishing the results.
![image](https://github.com/user-attachments/assets/d088d4da-9ba0-470b-8506-dec2209d4131)

 /
**Project: Property Tax Report Analysis and Implementation Using AWS**
 /
 In this project, I focused on creating a secure, scalable, and automated data analytics solution for the City of Vancouver’s Property Tax Report dataset. My role involved designing and implementing a data pipeline that utilized a variety of AWS services to ensure the data was processed efficiently, securely, and in a cost-effective manner. Below is an overview of the steps I took to achieve this:
Data Overview

The dataset I worked with contained property tax reports for the year 2024, specifically focusing on the Limited Agriculture zone classification. The key columns included tax levy, legal type (LAND and STRATA), and property value data. My objective was to cleanse, transform, and analyze the data to derive insights related to tax distribution and ensure the dataset adhered to strict governance standards due to its sensitive nature.
The data used for the analysis is the taxes levied by legal type for the zone classification of limited agriculture as available on the city of Vancouver portal
 
Source: https://opendata.vancouver.ca/explore/embed/dataset/property-tax-report/analyze/?sort=-tax_assessment_year&refine.report_year=2024&refine.zoning_classification=Limited%20Agriculture&dataChart=eyJxdWVyaWVzIjpbeyJjb25maWciOnsiZGF0YXNldCI6InByb3BlcnR5LXRheC1yZXBvcnQiLCJvcHRpb25zIjp7InNvcnQiOiItdGF4X2Fzc2Vzc21lbnRfeWVhciIsInJlZmluZS5yZXBvcnRfeWVhciI6IjIwMjQiLCJyZWZpbmUuem9uaW5nX2NsYXNzaWZpY2F0aW9uIjoiTGltaXRlZCBBZ3JpY3VsdHVyZSJ9fSwiY2hhcnRzIjpbeyJhbGlnbk1vbnRoIjp0cnVlLCJ0eXBlIjoicGllIiwiZnVuYyI6IlNVTSIsInlBeGlzIjoidGF4X2xldnkiLCJzY2llbnRpZmljRGlzcGxheSI6dHJ1ZSwiY29sb3IiOiJyYW5nZS1jdXN0b20iLCJwb3NpdGlvbiI6ImNlbnRlciJ9XSwieEF4aXMiOiJsZWdhbF90eXBlIiwibWF4cG9pbnRzIjo1MCwic29ydCI6IiIsInNlcmllc0JyZWFrZG93biI6IiIsInNlcmllc0JyZWFrZG93blRpbWVzY2FsZSI6IiJ9XSwidGltZXNjYWxlIjoiIiwiZGlzcGxheUxlZ2VuZCI6dHJ1ZSwiYWxpZ25Nb250aCI6dHJ1ZX0%3D 
Legal type	Tax 
LAND	3717126.62
STRATA	556507.85
![image](https://github.com/user-attachments/assets/4bf76d1f-8ddc-4745-9cfc-8bb052db4505)

Step 1: Data Ingestion and Cleansing
![image](https://github.com/user-attachments/assets/661b51d1-e45e-4f2c-9677-4f70f73efd4f)
![image](https://github.com/user-attachments/assets/d7a4932b-bd3c-48e3-a32d-db856d979635)


Using AWS Glue DataBrew, I performed the following tasks:

    Cleaning the dataset: Removed unnecessary columns like personal identifiers and report years that posed privacy risks.
    Data type standardization: I ensured that fields like tax levy, land value, and legal type were correctly formatted to avoid any discrepancies during data analysis.

Once cleaned, the data was stored in an S3 bucket, which served as the source for further processing and analysis.
Step 2: Data Security and Protection

Due to the sensitivity of the property tax data, I implemented the following security measures:

    KMS Encryption: A KMS key was generated and applied to encrypt the S3 bucket containing both raw and processed data. This ensured that the data remained secure both at rest and in transit.
    IAM Roles: I adhered to the principle of least privilege, ensuring that only authorized users could access or manipulate the data, further reducing the risk of breaches.
    Backup and Replication: I established S3 replication rules to create an automatic backup of the S3 bucket in a different region. This ensured data redundancy in case of disasters or regional outages.

Step 3: ETL Pipeline Development
![image](https://github.com/user-attachments/assets/af6c712f-a862-463c-9d2e-55304e14c44a)


I used AWS Glue to automate the extraction, transformation, and loading (ETL) processes:

    Data transformation: Applied filtering to focus on 2024 data for the Limited Agriculture zone and aggregated tax levies based on legal type (LAND, STRATA).
    Data quality checks: Implemented automatic checks for data accuracy and completeness to ensure that only valid records were included in the final analysis.
    Scheduled automation: The ETL pipeline was scheduled to run automatically, allowing new data to be processed and analyzed without manual intervention.

Step 4: Data Governance and Privacy
![image](https://github.com/user-attachments/assets/a1c61e22-9027-484b-bc89-7cac4eecbb1d)
![image](https://github.com/user-attachments/assets/32d84578-3ae9-42b4-b06c-bba3718e127c)


Data governance was a top priority for this project:

    I set up a trusted zone in the S3 bucket where only verified and validated data was stored after the ETL process.
    AWS Glue was used to continuously monitor data quality and ensure privacy standards were met, especially by detecting and eliminating sensitive information from the dataset.
    A visual ETL workflow was designed to provide real-time monitoring of jobs and transformations.

Step 5: Data Analysis Using AWS Athena
![image](https://github.com/user-attachments/assets/c9aec79e-b053-4d22-a04f-dfa0fda6a5e1)

![image](https://github.com/user-attachments/assets/ba003650-ba22-4a11-9e1b-22028d9c34d4)
![image](https://github.com/user-attachments/assets/99df89b6-f775-44a1-bbb8-17032e4291d1)


Once the data was prepared and loaded into the curated folder, I used AWS Athena for querying and analysis:

    The key analysis involved calculating the total tax levy for each legal type (LAND and STRATA).

    Example query:

    sql

    SELECT legal_type, SUM(tax_levy) AS total_tax
    FROM property_tax_report
    WHERE zoning_classification = 'Limited Agriculture'
    GROUP BY legal_type;

    This allowed me to derive insights regarding tax distribution, which helped inform decisions about tax contributions across property types.

Step 6: Data Visualization Using Amazon QuickSight
![image](https://github.com/user-attachments/assets/dfce4c10-76df-4ca6-a39d-2b0959cd004d)


To make the data more accessible to stakeholders, I used Amazon QuickSight to create interactive visualizations:

    Pie charts visualized the percentage contribution of LAND and STRATA legal types to the overall tax levy.
    Bar charts compared land and improvement values across different properties, providing a comprehensive view of the tax structure.

Step 7: Cost Optimization and Sustainability

Cost and sustainability were critical considerations for this project:

    S3 Intelligent-Tiering was used to automatically move infrequently accessed data to lower-cost storage tiers, reducing overall storage costs.
    CloudWatch alarms were configured to monitor costs related to S3, Athena, and EC2, with notifications sent if costs exceeded a set threshold (e.g., $45 USD).
    By leveraging AWS-managed infrastructure, the solution helped reduce the carbon footprint by optimizing energy consumption and relying on AWS's sustainable practices.

Step 8: Data Monitoring and CloudWatch
![image](https://github.com/user-attachments/assets/753cdd4e-71e5-48a0-9db4-3a872f41bcf9)
![image](https://github.com/user-attachments/assets/fe9da594-0538-4555-9563-8a7394ca3774)



To ensure the system remained efficient and secure over time:

    AWS CloudWatch was used to monitor key metrics like storage usage, service costs, and overall system performance.
    CloudTrail tracked user activities and API calls, providing a detailed audit trail that enhanced data governance and security compliance.

Step 9: Data Protection in the Long Term

To protect data even after processing:

    Data stored in the S3 bucket remained encrypted using KMS, ensuring long-term security.
    The replication rules ensured that a backup of the data existed in a secondary location, preserving data integrity and availability in case of regional outages.

Key Takeaways and Results

Through this project, I successfully demonstrated the use of AWS services for securely managing sensitive data while optimizing performance and cost. Key outcomes included:

    Efficient data processing: The automated ETL pipeline ensured data was processed and transformed without manual intervention, saving time and reducing errors.
    Data security: The use of KMS encryption and IAM roles ensured data privacy and protection at every stage.
    Scalability and sustainability: By leveraging AWS's managed services, the system was highly scalable and minimized energy consumption, contributing to the city’s sustainability goals.
