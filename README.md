# Azure Airlines Incremental CICD  Pipeline
Data Engineering Pipeline with DevOps Automation  
Tech Stack : ADLS, Azure Data Factory,GitHub, Azure DevOPS

## 📋 Project Overview

An end-to-end data engineering solution for processing airline flight data with incremental loading, automated data transformations, and CI/CD deployment using Azure cloud services

---

## 🏗️ Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SOURCE DATA LAYER                                  │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  Azure Data Lake Storage Gen2 (ADLS)                                 │   │
│  │  ┌────────────────────┐        ┌────────────────────┐               │   │
│  │  │  landing-zn/       │        │  processed-data/   │               │   │
│  │  │  - flights.csv     │───────▶│  - transformed     │               │   │
│  │  │  - airports.csv    │        │    output          │               │   │
│  │  └────────────────────┘        └────────────────────┘               │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      TRANSFORMATION LAYER (ADF)                              │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  Azure Data Factory Pipeline: "airlinepipeline"                      │   │
│  │                                                                       │   │
│  │  ┌─────────────────┐                                                 │   │
│  │  │ GetMetadata     │  Check if flights.csv exists                    │   │
│  │  │ Activity        │                                                 │   │
│  │  └────────┬────────┘                                                 │   │
│  │           │                                                           │   │
│  │           ▼                                                           │   │
│  │  ┌─────────────────┐                                                 │   │
│  │  │ If Condition    │                                                 │   │
│  │  │ Activity        │                                                 │   │
│  │  └────┬───────┬────┘                                                 │   │
│  │       │       │                                                       │   │
│  │   TRUE│       │FALSE                                                 │   │
│  │       │       │                                                       │   │
│  │       ▼       ▼                                                       │   │
│  │  ┌────────┐  ┌─────────────┐                                        │   │
│  │  │Execute │  │ Set Variable │                                        │   │
│  │  │DataFlow│  │ (No Action)  │                                        │   │
│  │  └────────┘  └─────────────┘                                        │   │
│  │       │                                                               │   │
│  │       ▼                                                               │   │
│  │  ┌──────────────────────────────────────────────┐                   │   │
│  │  │  Data Flow: "AirlineDataTransformation"      │                   │   │
│  │  │                                               │                   │   │
│  │  │  1. Read airports.csv (Airport Dimension)    │                   │   │
│  │  │  2. Read flights.csv (Daily Flights)         │                   │   │
│  │  │  3. JOIN on Origin Airport ID                │                   │   │
│  │  │     └─▶ Derive Departure Airport Details     │                   │   │
│  │  │  4. JOIN on Destination Airport ID           │                   │   │
│  │  │     └─▶ Derive Arrival Airport Details       │                   │   │
│  │  │  5. Write to processed-data/                 │                   │   │
│  │  └──────────────────────────────────────────────┘                   │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         TARGET/ANALYTICS LAYER                               │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  Azure Synapse Analytics                                             │   │
│  │  - Analytical queries                                                │   │
│  │  - Business intelligence                                             │   │
│  │  - Data visualization                                                │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                           CI/CD DEPLOYMENT LAYER                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  GitHub Repository                                                    │   │
│  │  ├── ARM Templates                                                    │   │
│  │  ├── Pipeline Definitions                                            │   │
│  │  └── Configuration Files                                             │   │
│  └──────────────────┬───────────────────────────────────────────────────┘   │
│                     │                                                        │
│                     ▼                                                        │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  Azure DevOps                                                         │   │
│  │  ┌────────────┐      ┌────────────┐      ┌────────────┐             │   │
│  │  │   Build    │─────▶│  Release   │─────▶│   Deploy   │             │   │
│  │  │  Pipeline  │      │  Pipeline  │      │  to PROD   │             │   │
│  │  └────────────┘      └────────────┘      └────────────┘             │   │
│  │                                                                       │   │
│  │  Self-Hosted Agent Pool (Local Machine)                              │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---








## 🎯 What Does This Pipeline Do?

### **Business Problem**
Airlines generate massive amounts of flight data daily. We need to:
- Process daily flight records incrementally
- Enrich flight data with airport information (city, state, airport names)
- Track departure and arrival delays
- Make data available for analytics and reporting

### **Pipeline Workflow**

1. **File Existence Check**
   - Pipeline checks if new daily flights file (`flights.csv`) exists in the landing zone
   - Uses GetMetadata activity to validate file availability

2. **Conditional Processing**
   - **If file exists (TRUE)**: Proceeds with data transformation
   - **If file doesn't exist (FALSE)**: Sets a variable and skips processing

3. **Data Transformation** (Mapping Data Flow)
   
   **Step 1: Load Source Data**
   - `airports.csv` - Dimension table with airport details (airport_id, city, state, name)
   - `flights.csv` - Daily flight facts (Carrier, OriginAirportID, DestAirportID, delays)

   **Step 2: Join on Departure Airport**
   - Joins flights with airports on `OriginAirportID = airport_id`
   - Derives departure airport details: city, state, airport name
   - Renames columns: `Depcity`, `Depstate`, `Depname`

   **Step 3: Join on Arrival Airport**
   - Joins result with airports again on `DestAirportID = airport_id`
   - Derives arrival airport details: city, state, airport name
   - Renames columns: `Arrcity`, `Arrstate`, `Arrname`

   **Step 4: Write Transformed Data**
   - Outputs enriched data to `processed-data/` folder
   - Truncates existing data before writing (full refresh within incremental load)

4. **Final Output Schema**
```
Carrier, DepDelay, ArrDelay, 
Depcity, Depstate, Depname,
Arrcity, Arrstate, Arrname
```

### **Example Transformation**

**Input (flights.csv):**
```
Carrier | OriginAirportID | DestAirportID | DepDelay | ArrDelay
UA      | 12478           | 14107         | 10       | 5
```

**Reference (airports.csv):**
```
airport_id | city          | state | name
12478      | New York      | NY    | JFK International
14107      | Los Angeles   | CA    | LAX International
```

**Output (processed):**
```
Carrier | DepDelay | ArrDelay | Depcity  | Depstate | Depname            | Arrcity     | Arrstate | Arrname
UA      | 10       | 5        | New York | NY       | JFK International  | Los Angeles | CA       | LAX International
```

---

## 🛠️ Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Source** | Azure Data Lake Storage Gen2 | Raw data storage (landing zone & processed data) |
| **Orchestration** | Azure Data Factory | Pipeline orchestration, data movement, transformations |
| **Transformation** | ADF Mapping Data Flows | ETL logic, joins, derived columns |
| **Analytics** | Azure Synapse Analytics | Data warehousing, querying, BI |
| **Monitoring** | Logic Apps | Alerts and notifications (optional) |
| **Version Control** | GitHub | Source code repository |
| **CI/CD** | Azure DevOps | Automated build and deployment |
| **Agent** | Self-Hosted Agent | Local machine agent for deployment |

---

## 📂 Repository Structure

```
azure-airlines-incremental-pipeline/
│
├── arm-templates/
│   ├── ARMTemplateForFactory.json              # Main ARM template
│   ├── ARMTemplateParametersForFactory.json    # Parameters for DEV
│   ├── ArmTemplate_master.json                 # Linked template master
│   ├── ArmTemplateParameters_master.json       # Master parameters
│   └── ArmTemplate_0.json                      # Component template
│
├── pipelines/
│   └── airlinepipeline.json                    # Pipeline definition
│
├── dataflows/
│   └── AirlineDataTransformation.json          # Data flow transformation logic
│
├── datasets/
│   ├── AirlineDimSource.json                   # Airport dimension dataset
│   ├── DailyFlightsSource.json                 # Daily flights dataset
│   └── Processeddata.json                      # Output dataset
│
├── linkedservices/
│   └── AzureDataLakeStorageForDevAirline.json  # ADLS connection
│
├── .github/
│   └── workflows/
│       └── ci-cd-pipeline.yml                  # GitHub Actions workflow
│
├── docs/
│   ├── setup-guide.md                          # Setup instructions
│   └── deployment-guide.md                     # Deployment steps
│
└── README.md                                    # This file
```

---

## 🚀 Project Setup & Deployment

### **Prerequisites**

- Azure Subscription
- Azure Data Factory (DEV & PROD instances)
- Azure Data Lake Storage Gen2 account
- Azure Synapse Analytics workspace
- GitHub account
- Azure DevOps organization
- Self-hosted agent (local machine)

### **Phase 1: Development Environment Setup**

#### 1. **Create Azure Resources**

```bash
# Resource Group
az group create --name rg-airline-dev --location eastus

# Storage Account (ADLS Gen2)
az storage account create \
  --name airlinedev \
  --resource-group rg-airline-dev \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --enable-hierarchical-namespace true

# Create containers
az storage container create --name united-airlines --account-name airlinedev

# Data Factory (DEV)
az datafactory create \
  --resource-group rg-airline-dev \
  --name airline-adf-develop
```

#### 2. **Upload Sample Data**

Upload to ADLS `united-airlines/landing-zn/`:
- `airports.csv` - Airport dimension data
- `flights.csv` - Daily flight data

#### 3. **Configure ADF Pipeline**

- Create linked service to ADLS
- Create datasets (airports, flights, processed)
- Create data flow (transformation logic)
- Create pipeline (orchestration)
- Test and validate

### **Phase 2: Azure DevOps Setup**

#### 1. **Create Organization & Project**

```
1. Go to https://dev.azure.com
2. Create Organization: "YourOrgName"
3. Create Project: "Airlines-Data-Pipeline"
4. Create Repository: Import from GitHub or create new
```

#### 2. **Setup Self-Hosted Agent**

```bash
# Download agent
mkdir myagent && cd myagent
wget https://vstsagentpackage.azureedge.net/agent/4.268.0/vsts-agent-osx-x64-4.268.0.tar.gz
tar zxvf vsts-agent-osx-x64-4.268.0.tar.gz

# Configure agent
./config.sh
# Enter: https://dev.azure.com/{your-org}
# Enter: PAT token
# Enter: Agent pool name (Default)
# Enter: Agent name

# Run agent
./run.sh
```

#### 3. **Create Build Pipeline**

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - develop

pool:
  name: 'Default'  # Your self-hosted agent pool

steps:
  - task: CopyFiles@2
    inputs:
      SourceFolder: '$(Build.SourcesDirectory)'
      Contents: |
        **/ARMTemplateForFactory.json
        **/ARMTemplateParametersForFactory.json
      TargetFolder: '$(Build.ArtifactStagingDirectory)'
  
  - task: PublishBuildArtifacts@1
    inputs:
      PathtoPublish: '$(Build.ArtifactStagingDirectory)'
      ArtifactName: 'arm-templates'
```

### **Phase 3: CI/CD Release Pipeline**

#### 1. **Create Release Pipeline**

```
1. Go to Pipelines → Releases → New Pipeline
2. Add Artifact: Select your build pipeline
3. Add Stage: "Deploy to PROD"
4. Add Tasks:
   - ARM Template Deployment
   - Configure parameters for PROD environment
```

#### 2. **Configure ARM Deployment**

```json
{
  "factoryName": "airline-adf-prod",
  "AzureDataLakeStorageForDevAirline_accountKey": "$(StorageAccountKey)",
  "AzureDataLakeStorageForDevAirline_properties_typeProperties_url": "https://airlineprod.dfs.core.windows.net/"
}
```

#### 3. **Enable Continuous Deployment**

- Enable CD trigger on artifact
- Set branch filter: `main`
- Every merge to main → auto-deploys to PROD

---

## 🔄 CI/CD Workflow

```
Developer makes changes
        ↓
Commit & Push to GitHub (develop branch)
        ↓
Create Pull Request to main
        ↓
Code Review & Approval
        ↓
Merge to main
        ↓
Azure DevOps Build Pipeline Triggered
        ↓
Build ARM Templates
        ↓
Create Build Artifacts
        ↓
Release Pipeline Triggered (CD)
        ↓
Deploy ARM Templates to PROD ADF
        ↓
Update Pipelines, Datasets, Data Flows
        ↓
Production Deployment Complete ✅
```

---

## 📊 Data Flow Details

### **Transformation Steps**

1. **Source: AirportDimdata**
   - Schema: `airport_id, city, state, name`
   - Source: `united-airlines/landing-zn/airports.csv`

2. **Source: DailyFlightsData**
   - Schema: `Carrier, OriginAirportID, DestAirportID, DepDelay, ArrDelay`
   - Source: `united-airlines/landing-zn/flights.csv`

3. **Join: joinOnDeptAirport**
   - Type: Inner Join
   - Condition: `OriginAirportID == airport_id`
   - Result: Flight + Departure Airport details

4. **Select: DeriveDeptAirportDetails**
   - Rename: `city → Depcity`, `state → Depstate`, `name → Depname`
   - Keep: `Carrier, DestAirportID, DepDelay, ArrDelay`

5. **Join: JoinOnArrivalAirport**
   - Type: Inner Join
   - Condition: `DestAirportID == airport_id`
   - Result: Flight + Departure + Arrival Airport details

6. **Select: DeriveArrivalAirport**
   - Rename: `city → Arrcity`, `state → Arrstate`, `name → Arrname`
   - Final Schema: `Carrier, DepDelay, ArrDelay, Depcity, Depstate, Depname, Arrcity, Arrstate, Arrname`

7. **Sink: writeProcessedData**
   - Destination: `united-airlines/processed-data/`
   - Mode: Truncate (full refresh)

---

## 🔧 Configuration

### **Environment Variables**

Create parameter files for each environment:

**DEV:**
```json
{
  "factoryName": "airline-adf-develop",
  "storageAccountUrl": "https://airlinedev.dfs.core.windows.net/",
  "storageAccountKey": "***"
}
```

**PROD:**
```json
{
  "factoryName": "airline-adf-prod",
  "storageAccountUrl": "https://airlineprod.dfs.core.windows.net/",
  "storageAccountKey": "***"
}
```

### **Linked Service Configuration**

```json
{
  "name": "AzureDataLakeStorageForDevAirline",
  "type": "AzureBlobFS",
  "typeProperties": {
    "url": "https://airlinedev.dfs.core.windows.net/",
    "accountKey": {
      "type": "SecureString",
      "value": "***"
    }
  }
}
```

---

## 📈 Monitoring & Alerts

### **ADF Monitoring**

1. **Pipeline Runs**: Monitor execution history
2. **Activity Runs**: Track individual activity status
3. **Data Flow Debug**: Test transformations
4. **Metrics**: Success rate, duration, data volume

### **Logic Apps Integration** (Optional)

```
Pipeline Failure/Success
        ↓
Trigger Logic App
        ↓
Send Email/Teams Notification
        ↓
Log to Monitoring Dashboard
```

---

## 🧪 Testing

### **Unit Testing**

```bash
# Test pipeline with sample data
az datafactory pipeline create-run \
  --resource-group rg-airline-dev \
  --factory-name airline-adf-develop \
  --name airlinepipeline
```

### **Data Validation**

1. Check row counts: Source vs. Processed
2. Validate joins: No data loss
3. Verify schema: All columns present
4. Test edge cases: Empty files, missing airports

---

## 🤝 Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open Pull Request

---

## 📝 License

This project is licensed under the MIT License.

---

## 👥 Authors

**Your Name**  
- GitHub: [@yourusername](https://github.com/yourusername)
- LinkedIn: [Your Profile](https://linkedin.com/in/yourprofile)

---

## 🙏 Acknowledgments

- Microsoft Azure Documentation
- Azure Data Factory Best Practices
- Azure DevOps Community

---

## 📞 Support

For issues and questions:
- Open an issue in this repository
- Contact: your.email@example.com

---

**Last Updated:** February 2026
