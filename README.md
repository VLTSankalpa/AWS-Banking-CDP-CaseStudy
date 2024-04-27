# **Building a Secure Customer Data Platform for Banking on AWS: A Case Study Using Lambda, Kinesis, Glue, S3, Athena, and SageMaker**

# **Introduction**

In this case study, we delve into the creation of a secure, scalable, and robust Customer Data Platform (CDP) specifically designed for the banking sector, utilizing the comprehensive capabilities of AWS. This platform addresses critical aspects of data management in the financial services industry, focusing on security, efficiency, and scalability. The use of AWS services such as Lambda, Kinesis, Glue, S3, Athena, and SageMaker enables us to handle sensitive personal identifiable information (PII) with utmost security, streamline data workflows, and unlock valuable insights through advanced analytics and machine learning.

# **Objectives**

The objectives of this Customer Data Platform are to:

## **Secure Management of PII**:

- **Goal**: Ensure the security and privacy of personal identifiable information at every stage of data handling.
- **Method**: Utilize AWS KMS for encryption and IAM for access control, creating a secure environment where PII data is encrypted both in transit and at rest.
- **Impact**: Minimize the risk of data breaches and comply with stringent data protection regulations, such as GDPR and CCPA.

## **Real-Time Data Streaming and Analytics**:

- **Goal**: Facilitate the seamless flow and real-time processing of large volumes of transactional data.
- **Method**: Implement Amazon Kinesis for data ingestion and streaming, coupled with AWS Lambda for real-time data processing.
- **Impact**: Enable immediate data availability and responsiveness, supporting timely decision-making and operational agility.

## **ETL Pipelines and BI Dashboards**:

- **Goal**: Transform raw data into actionable insights through well-structured ETL pipelines and intuitive business intelligence dashboards.
- **Method**: Leverage AWS Glue for data integration and transformation, and Amazon QuickSight for creating interactive dashboards that visualize key metrics.
- **Impact**: Enhance understanding of data patterns, support strategic planning, and provide stakeholders with easy access to critical business insights.

## **Advanced Machine Learning Models**:

- **Goal**: Implement predictive models for customer segmentation, lifetime value prediction, and fraud detection.
- **Method**: Deploy continuous training and deployment pipelines using AWS SageMaker, monitor performance via AWS CloudWatch, and automate retraining with SageMaker Pipelines and EventBridge.
- **Impact**: By dynamically adapting to new data and trends, these models significantly enhance customer retention, optimize marketing expenditure, and reduce financial losses due to fraud, thereby directly boosting the bank's profitability and customer satisfaction.

## **Serverless API for Model Access**:

- **Goal**: Provide scalable access to machine learning models through serverless architecture.
- **Method**: Develop model endpoints as AWS Lambda functions, expose them via Amazon API Gateway for secure and scalable API endpoints, and document APIs using industry standards like Swagger or OpenAPI.
- **Impact**: This architecture ensures seamless, scalable, and secure access to predictive analytics, enabling real-time customer insights and operational agility that can drive competitive advantage and improve service delivery.

## **Generative AI-Powered Customer Engagements**:

- **Goal**: Automate the generation and dispatch of real-time alerts for credit card fraud risks and targeted marketing campaigns.
- **Method**: Employ AWS Lambda and Amazon SageMaker to harness generative AI techniques, crafting personalized email communications based on customer behavior and risk profiles.
- **Impact**: Enhance fraud detection mechanisms and marketing responsiveness, ensuring timely interventions and engaging customer interactions.

By achieving these objectives, the Customer Data Platform not only supports the core operational needs of banks but also plays a crucial role in strategic business development. Through this AWS-based solution, banks can harness the power of their data to make informed decisions, personalize customer interactions, and maintain a competitive edge in the fast-evolving financial landscape.

# **About the Dataset**

## **Context**

Accessing comprehensive and realistic banking customer data poses significant challenges due to privacy concerns and regulatory restrictions. Available public datasets are often heavily encoded or anonymized, with sensitive personal identifiable information (PII) obscured through methods like principal component analysis (PCA) to protect user privacy. These datasets typically feature a limited number of transactions over short durations and lack genuine customer information, limiting their usefulness for in-depth analysis and model training.

However, the dataset available at [this Kaggle link](https://www.kaggle.com/datasets/ealtman2019/credit-card-transactions) offers a unique exception. It includes more than 20 million transactions generated from a sophisticated multi-agent virtual world simulation conducted by IBM. This rich dataset represents 2,000 synthetic consumers based in the United States, who exhibit global transactional behavior. Spanning several decades, the dataset captures diverse purchasing activities and includes multiple cards per consumer, providing a broad spectrum of consumer spending patterns.

Additional details about the dataset's creation can be found in this [research paper](https://arxiv.org/abs/1910.03033). Analyses of the dataset have demonstrated its ability to closely replicate real-world data across various dimensions, including fraud rates, transaction volumes, Merchant Category Codes (MCCs), and other relevant metrics. Crucially, all columns retain their natural values, with the sole exception of the merchant name. These natural values are instrumental in simulating real-life scenarios, making the dataset an invaluable resource for feature engineering and generating insights in big data environments.

The dataset consists of three CSV files, detailing user profiles, card details, and credit card transactions. Below is a brief explanation of the fields within these files:

### **`sd254_users.csv`**

- **Person**: Name of the person.
- **Current Age, Retirement Age, Birth Year, Birth Month, Gender**: Demographic information.
- **Address, Apartment, City, State, Zipcode, Latitude, Longitude**: Location details.
- **Per Capita Income - Zipcode, Yearly Income - Person, Total Debt**: Financial information.
- **FICO Score, Num Credit Cards**: Credit score and number of credit cards owned.

### **`sd254_cards.csv`**

- **User, CARD INDEX**: Identifiers for the user and a specific card.
- **Card Brand, Card Type**: Information about the card brand (e.g., Visa, MasterCard) and type (e.g., Debit, Credit).
- **Card Number, Expires, CVV**: Card details including the card number, expiration date, and CVV.
- **Has Chip**: Indicates whether the card is equipped with a chip.
- **Cards Issued**: Number of cards issued to the user.
- **Credit Limit**: The credit limit on the card.
- **Acct Open Date, Year PIN last Changed**: Account opening date and the last year the PIN was changed.
- **Card on Dark Web**: Indicates if the card's details have been found on the dark web.

### **`User0_credit_card_transactions.csv`**

- **User, Card**: Identifiers for the user and the card used.
- **Year, Month, Day, Time**: Timestamp details of the transaction.
- **Amount**: The amount of the transaction.
- **Use Chip**: Indicates whether the transaction was made using a chip.
- **Merchant Name, City, State, Zip**: Information about the merchant.
- **MCC (Merchant Category Code)**: A four-digit number listing the merchant's type of business.
- **Errors?**: Indicates if there were any errors during the transaction.
- **Is Fraud?**: Indicates if the transaction was fraudulent.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2f7830c3-173b-41cd-aa45-25fca7558acb/6198d4a1-885f-49c3-b2ea-00fbee7a2c4e/Untitled.png)

# **Architecture Overview**

This section provides a high level overview of the design and architecture of a secure, scalable Customer Data Platform (CDP) tailored specifically for the banking sector utilizing a range of AWS services. The architecture is split into two main parts: the Core Banking System VPC and the Customer Data Platform VPC.

## **Core Banking System VPC**

- **Online Transaction Processing Database:** DynamoDB acts as the core database for all sensitive customer data, encompassing user profiles, card details, and transaction logs. Positioned within a private Virtual Private Cloud (VPC), it provides robust data isolation and security, effectively replicating an Online Transaction Processing (OLTP) database environment typical of core banking systems on AWS. This setup ensures that DynamoDB handles high-transaction loads efficiently, maintaining data integrity and swift access, crucial for day-to-day banking operations.
- **AWS Lambda for Data Extraction and PII Encryption**: A dedicated Lambda function is configured to retrieve data securely from DynamoDB. It also performs PII masking and encryption using AWS Key Management Service (KMS) to ensure data privacy.
- **AWS Key Management Service (KMS) for Encryption Key Management**: AWS KMS is utilized to manage and safeguard encryption keys that secure PII during the data extraction process. This service provides centralized control over cryptographic keys, ensuring high standards of data privacy and security as mandated by industry regulations.
- **Amazon Kinesis for Secure Data Transport**: Once the AWS Lambda function processes and encrypts the data, it securely transmits this data to the Customer Data Platform VPC using Amazon Kinesis. This approach ensures that all PII remains encrypted and is managed securely throughout the transmission process, maintaining data integrity and privacy across different network boundaries.

## **Customer Data Platform VPC**

### **Data Reception and Processing**

- **Lambda Functions for Data Transformations**: These functions, triggered in real-time by the Amazon Kinesis stream, execute essential data transformations and integrate machine learning predictions by calling AWS SageMaker model endpoints. The enriched data is then loaded into Amazon S3 for durable storage and subsequent analysis.

### **Analytics and Reporting**

- **AWS Glue**: Utilized for schema management and data cataloging, AWS Glue Crawlers automatically extract metadata from the data stored in Amazon S3, organizing it for efficient querying and comprehensive analysis.
- **Amazon Athena**: Executes sophisticated SQL queries on the structured data residing in Amazon S3, enabling the creation of dynamic and scalable reports.
- **Amazon QuickSight**: Generates interactive business intelligence (BI) dashboards that provide visual insights, enhancing data-driven decision-making processes.

### **Advanced Machine Learning Models**

- **AWS SageMaker**: Employs advanced machine learning techniques to build, train, and deploy models that predict customer lifetime value, segment customers for targeted marketing, and detect fraudulent transactions.
- **SageMaker Pipelines and EventBridge**: Automate the workflow of continuous training and deployment, responding dynamically to new data and insights with automated updates and performance tuning.

### **CI/CD for Machine Learning Pipelines**

- **AWS CodePipeline and CodeBuild**: Implement continuous integration and deployment processes for machine learning models, ensuring they adapt and perform optimally as new data emerges.
- **Infrastructure as Code (IaC) with AWS Cloud Development Kit (CDK) in Python**: Automates the setup and management of AWS resources, providing a consistent and error-free deployment environment.

### **Serverless Model Deployment**

- **AWS Lambda and API Gateway**: Models are deployed as serverless endpoints using AWS Lambda, ensuring efficient scaling and reduced management overhead. API Gateway secures and manages access to these endpoints.
- **API Documentation**: Developed using Swagger or OpenAPI standards, this documentation provides comprehensive guidelines on how external systems can interact with the deployed model APIs.

### **Generative AI-Powered Email Alerts for Fraud Detection**

- **Anthropic's Claude in Amazon Bedrock**: We will integrate Anthropic’s Claude, hosted on Amazon Bedrock, to generate coherent and contextually relevant email alerts. This setup leverages the cutting-edge generative AI to compose email content that is both informative and easy to understand for end-users.
- **Data Decryption with AWS KMS in Core Banking System VPC**: For each alert, necessary PII data will be securely retrieved and decrypted using AWS Key Management Service (KMS). All operations involving sensitive decrypted data will be handled within the confines of the Core Banking System VPC to ensure maximum security and compliance with data protection regulations.
- **AWS Lambda and Simple Notification Service (Amazon SNS) for Email Composition and Sending**: AWS Lambda will be used to trigger the generative AI in Amazon Bedrock, compose the email content based on the AI’s output, and then send the email alerts to the customers. This setup ensures that the email generation and dispatch processes are scalable and efficient. Once the emails are composed, Amazon SNS will be used for sending out the emails. This service provides a robust, scalable, and secure platform for outgoing email communications.

### **Security and Compliance Framework**

- **AWS KMS and IAM**: Key Management Service (KMS) manages encryption keys used to secure PII, while IAM policies ensure rigorous access control, upholding the highest security standards.
- **Monitoring with Amazon CloudWatch and AWS CloudTrail**: CloudWatch oversees the operational health of services, whereas CloudTrail logs all actions, aiding in compliance and security audits.

By utilizing these AWS services and configurations, the architecture ensures that the Customer Data Platform is secure, compliant, and capable of handling the complex needs of the banking sector, facilitating real-time processing, advanced analytics, and dynamic responses to customer behaviors and market conditions.

# **Approach for Real-Time Data Simulation and Batch Processing**

## **Real-Time Data Streaming**

- **Data Selection**: Approximately 5% of the total dataset is designated for real-time data streaming to simulate real-world transactional flows.
- **AWS Services**:
    - **Lambda Functions**: Select and preprocess the designated real-time streaming data, ensuring it is securely streamed.
    - **Kinesis**: Manages the ingestion and processing of streamed data.
    - **AWS KMS**: Encrypts sensitive data fields before they are streamed, safeguarding data in transit.

## **Batch Data Processing**

- **Data Selection**: The remaining 95% of the data is processed in batches.
- **AWS Services**:
    - **AWS Batch**: Manages large-scale batch processing jobs.
    - **AWS KMS**: Encrypts batch-processed data to ensure security.
    - **Amazon S3**: Acts as the primary data storage post-processing.
    - **AWS Glue and Amazon Athena**: Handle ETL processes and data querying, respectively.
    - **Amazon QuickSight**: Used for visualizing and interpreting processed data

# **AWS Services Utilized**

Here is a list of the AWS services employed in the architecture of the secure Customer Data Platform (CDP) for the banking sector:

- **Amazon DynamoDB**
- **AWS Lambda**
- **AWS Key Management Service (KMS)**
- **Amazon Kinesis**
- **AWS Glue**
- **Amazon S3**
- **Amazon Athena**
- **AWS Batch**
- **Amazon QuickSight**
- **AWS SageMaker**
- **Amazon EventBridge**
- **AWS CodePipeline**
- **AWS CodeBuild**
- **AWS Cloud Development Kit (CDK)**
- **Amazon API Gateway**
- **Swagger or OpenAPI** (for API documentation)
- **AWS Identity and Access Management (IAM)**
- **Amazon CloudWatch**
- **AWS CloudTrail**
- **Simple Notification Service (Amazon SNS)**
- **Amazon Bedrock**

This comprehensive use of various AWS services ensures a robust, scalable, and secure platform, catering to the extensive needs of data management, processing, and analytics within the banking industry.