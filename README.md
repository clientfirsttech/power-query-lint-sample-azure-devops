# Example of using Power Query Lint API in Azure DevOps Pipeline

This project provides an automated Azure DevOps pipeline for linting TMDL (Tabular Model Definition Language) files using the PQLint API. The pipeline automatically triggers when .tmdl files are modified and performs comprehensive code quality analysis on table definitions and expressions.

## Features

- **Automated Triggering**: Runs automatically when .tmdl files are changed
- **Comprehensive Coverage**: Analyzes all table definition files and expressions.tmdl
- **Detailed Reporting**: Provides color-coded output with severity levels (Error, Warning, Info)
- **Artifact Publishing**: Exports results as JSON artifacts for further analysis
- **Pipeline Integration**: Fails the build when errors are detected to maintain code quality

# Getting Started

Follow these steps to set up the PQLint pipeline in your Azure DevOps environment.

## Prerequisites

1. **Azure DevOps Project**: You need access to an Azure DevOps project with pipeline creation permissions
2. **PQLint API Subscription**: A valid subscription key for the PQLint API
3. **Repository Access**: The ability to import or clone this repository into your Azure DevOps project
4. **PBIP Project Format**: Your Power BI project must be in PBIP (Power BI Project) format with TMDL (Tabular Model Definition Language) files. This requires:
   - **Power BI Desktop** with PBIP support enabled
   - **Project saved as .pbip format** (not .pbix)
   - **TMDL semantic model structure** with separate .tmdl files for tables and expressions
   - **File structure** similar to:
     ```
     YourProject.pbip
     YourProject.SemanticModel/
       definition/
         tables/
           Table1.tmdl
           Table2.tmdl
           ...
         expressions.tmdl
         database.tmdl
         model.tmdl
         relationships.tmdl
     ```

## Installation Process

### Step 1: Import the Repository

1. **Navigate to your Azure DevOps project**
2. **Go to Repos → Files**
3. **Click "Import repository"** (or clone if you prefer)
4. **Enter the repository URL** for this project
5. **Click "Import"** to bring the code into your project

### Step 2: Create the Variable Group

The pipeline uses a secure variable group to store sensitive configuration like API keys.

1. **Navigate to Pipelines → Library** in your Azure DevOps project
2. **Click "+ Variable group"**
3. **Name the variable group**: `PQLint_Variables`
4. **Add the following variables**:
   
   | Variable Name | Value | Secret |
   |---------------|-------|---------|
   | `PQLINT_SUBSCRIPTION_KEY` | Your PQLint API subscription key | ✅ Yes |
   
5. **Mark the subscription key as secret** by clicking the lock icon
6. **Save the variable group**

### Step 3: Set Variable Group Permissions

1. **In the variable group settings, click "Pipeline permissions"**
2. **Grant access to the pipeline** (you can do this after creating the pipeline in Step 4)
3. **Alternatively, set it to allow access to all pipelines** if preferred

### Step 4: Create the Pipeline

1. **Navigate to Pipelines → Pipelines**
2. **Click "New pipeline"**
3. **Select "Azure Repos Git"** (or your source control option)
4. **Choose your repository** (the one you imported in Step 1)
5. **Select "Existing Azure Pipelines YAML file"**
6. **Choose the path**: `/azure-pipelines.yml`
7. **Click "Continue"**
8. **Review the pipeline configuration** and click "Run"

### Step 5: Grant Variable Group Access (if needed)

If the pipeline fails due to variable group access issues:

1. **Go back to Pipelines → Library**
2. **Open the `PQLint_Variables` group**
3. **Click "Pipeline permissions"**
4. **Add your newly created pipeline** to the allowed list

## Pipeline Configuration

The pipeline is configured with the following default paths:

```yaml
variables:
  - group: PQLint_Variables
  - name: modulePathRoot
    value: '$(Build.SourcesDirectory)\ci_cd'
  - name: tablesPath
    value: '$(Build.SourcesDirectory)\SampleModel.SemanticModel\definition\tables'
  - name: definitionPath
    value: '$(Build.SourcesDirectory)\SampleModel.SemanticModel\definition'
```

### Customizing Paths

If your project structure differs, update these variables in the `azure-pipelines.yml` file:

- **modulePathRoot**: Path to the `PQLintAPI.psm1` module
- **tablesPath**: Path to your table definition .tmdl files
- **definitionPath**: Path to your main definition folder (contains expressions.tmdl)

## Supported File Types

The pipeline analyzes the following TMDL files:

- **Table Definitions**: All `.tmdl` files in the `tables` directory
- **Expressions**: The `expressions.tmdl` file in the definition directory

## Pipeline Triggers

The pipeline automatically triggers on:

- **Push to main branch** when .tmdl files are modified
- **Pull requests to main or develop branches** when .tmdl files are modified

## Understanding Results

### Severity Levels

- **Error (3)**: Critical issues that will fail the pipeline
- **Warning (2)**: Best practice violations that should be addressed
- **Info (1)**: Informational suggestions for improvement

### Output Format

For each issue found, the pipeline displays:
- **Rule Name and Severity**
- **Rule ID and Category**
- **Description of the issue**
- **Location information** (line numbers, positions)
- **File name** where the issue was found

### Artifacts

When issues are found, the pipeline publishes a JSON artifact containing:
- **Detailed results** for each file analyzed
- **Complete error information** for programmatic processing
- **Summary statistics** by severity level

# Build and Test

## Running the Pipeline

The pipeline runs automatically when triggered, but you can also:

1. **Manual Run**: Go to Pipelines → Your Pipeline → "Run pipeline"
2. **Test Changes**: Create a pull request with .tmdl file modifications
3. **View Results**: Check the pipeline logs for detailed analysis results

## Local Testing

To test the PQLint module locally:

```powershell
# Import the module
Import-Module .\ci_cd\PQLintAPI.psm1

# Test with a sample TMDL file
$content = Get-Content .\SampleModel.SemanticModel\definition\tables\YourTable.tmdl -Raw
Invoke-CodeLinting -Code $content -SubscriptionKey "your-key" -Format "tmdl"
```

## Troubleshooting

### Common Issues

1. **"Variable group not found"**: Ensure the `PQLint_Variables` group exists and has proper permissions
2. **"Subscription key invalid"**: Verify your PQLint API key is correct and not expired
3. **"Path not found"**: Check that the configured paths match your project structure
4. **"Module import failed"**: Ensure `PQLintAPI.psm1` exists in the specified location
