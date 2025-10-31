# Azure Content Safety - Custom Categories

This repository demonstrates how to use Azure AI Content Safety's custom text categorization capabilities through both **Standard** and **Rapid** modes.

## Overview

Azure Content Safety allows you to create custom categories to detect specific types of content relevant to your use case. This repo includes examples for:

- **Standard Mode**: Train a custom category with a larger dataset for more robust classification
- **Rapid Mode**: Quickly create an "incident" with a few samples for fast prototyping

Both modes are demonstrated using a "survival advice" theme to detect wilderness/camping-related content.

## Prerequisites

- Python 3.13+
- [uv](https://github.com/astral-sh/uv) - Python package installer and resolver
- Azure AI Content Safety resource in a [supported region](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/overview#region-availability) (Custom categories are available only in limied number of regions)
- Azure Storage account (for Standard mode) - **Note**: Must be a regular storage account, not hierarchical namespace
- [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) VS Code extension (for .http files)

> **Important**: Custom categories (standard) feature is only available in certain Azure regions. Ensure you create your Content Safety resource in one of the supported regions listed above.

## Setup

1. **Clone the repository**

```bash
git clone <repository-url>
cd azure-content-safety-custom-categories
```

2. **Install dependencies using uv**

```bash
uv sync
```

Alternatively, you can use:

```bash
uv pip install -r requirements.txt
```

3. **Configure environment variables**

Create a `.env` file in the root directory with the following variables:

```env
# Azure Content Safety
API_KEY=your_content_safety_api_key
ENDPOINT=https://your-resource.cognitiveservices.azure.com

# Standard Mode Configuration
CATEGORY_NAME=survival-advice
CATEGORY_VERSION=1
CATEGORY_DEFINITION=text prompts about survival advice in camping/wilderness situations
AZURE_STORAGE_URL=https://your-storage-account.blob.core.windows.net
AZURE_STORAGE_CONTAINER=your-container-name
TRAINING_DATA_SET=survival-advice.jsonl

# Rapid Mode Configuration
RAPID_INCIDENT_NAME=survival-advice-incident
RAPID_INCIDENT_DEFINITION=Content related to wilderness survival techniques and camping advice
```

## Project Structure

```
├── .env                          # Environment variables (create this)
├── .gitignore
├── .python-version              # Python 3.13
├── pyproject.toml               # Project dependencies
├── requirements.txt             # Pip dependencies
├── README.md
├── Rapid/
│   ├── rapid-python.ipynb       # Rapid mode Jupyter notebook
│   └── rapid-rest-api.http      # Rapid mode REST API examples
└── Standard/
    ├── standard-python.ipynb    # Standard mode Jupyter notebook
    ├── standard-rest-api.http   # Standard mode REST API examples
    └── survival-advice.jsonl    # Training dataset (50 examples)
```

## Usage

### Standard Mode

Standard mode requires training data uploaded to Azure Blob Storage and involves a training process.

> **Important**: 
> - Training can take 5-10 hours to complete. Plan your moderation pipeline accordingly.
> - You must enable **Managed Identity** for your Content Safety resource and grant it **Storage Blob Data Contributor** or **Storage Blob Data Owner** role on your storage account.
> - Storage account must be a regular blob storage account, not a hierarchical namespace (Data Lake) account.

#### Using Python (Jupyter Notebook)

Open and run `Standard/standard-python.ipynb`:

1. **Create a category version** - Define your custom category with training data
2. **Start the build process** - Trigger model training
3. **Check build status** - Monitor training progress
4. **Analyze text** - Test your trained category
5. **Manage categories** - List, get details, or delete categories

Key functions:
- `create_new_category_version()` - Initialize a new category
- `trigger_category_build_process()` - Start training
- `get_build_status()` - Check training status
- `analyze_text_with_customized_category()` - Classify text

#### Using REST API

Open `Standard/standard-rest-api.http` in VS Code with the REST Client extension and execute the requests sequentially.

**Workflow:**
1. Create new category version (PUT)
2. Start the category build process (POST)
3. Get the category build status (GET) - Wait for completion
4. Analyze text with your customized category (POST)
5. List/Get/Delete categories as needed

### Rapid Mode

Rapid mode allows quick prototyping with just a few samples (5-10) without extensive training.

#### Using Python (Jupyter Notebook)

Open and run `Rapid/rapid-python.ipynb`:

1. **Create an incident** - Define what type of content to detect
2. **Add samples** - Provide 5-10 example texts
3. **Deploy the incident** - Activate the detector (takes ~10 seconds)
4. **Detect text incidents** - Test with new text
5. **Clean up** - Delete the incident when done

#### Using REST API

Open `Rapid/rapid-rest-api.http` in VS Code with the REST Client extension.

**Workflow:**
1. Create an incident object (PATCH)
2. Add samples to the incident (POST)
3. Deploy the incident (POST)
4. Detect text incidents (POST) - Wait ~10 seconds after deployment
5. List/Get/Delete incidents as needed

## Training Data Format

The training data for Standard mode is in JSONL format (see `survival-advice.jsonl`):

```json
{"text": "How to build a shelter in the wilderness"}
{"text": "Identifying edible plants in your surroundings"}
{"text": "Finding and purifying water in the wild"}
```

Requirements:
- Minimum 50 examples recommended
- Each line is a valid JSON object with a "text" field
- Upload to Azure Blob Storage for Standard mode

## Example Results

Testing with: *"Creating fire without matches or a lighter in a rainy environment."*

**Rapid Mode Response:**
```json
{
  "incidentMatches": [
    {
      "incidentName": "survival-advice-incident"
    }
  ]
}
```

**Standard Mode Response:**
```json
{
  "categoryAnalysis": {
    "categoryName": "survival-advice",
    "detected": true,
    "score": 0.95
  }
}
```

## API Versions

- **Standard Mode**: `2024-09-15-preview`
- **Rapid Mode**: `2024-02-15-preview`

## Key Differences: Standard vs Rapid

| Feature | Standard Mode | Rapid Mode |
|---------|--------------|------------|
| Training Data | 50+ samples required | 5-10 samples sufficient |
| Training Time | Minutes to hours | ~10 seconds deployment |
| Storage Required | Azure Blob Storage | No external storage |
| Use Case | Production, high accuracy | Quick prototyping, testing |
| API Endpoint | `/text/categories` | `/text/incidents` |

## Dependencies

- `requests` - HTTP library for API calls
- `python-dotenv` - Environment variable management
- `ipykernel` - Jupyter notebook support
- `pandas` - Data manipulation (optional)

## Troubleshooting

### "Category not found or not ready to use"

- Check that the build process completed successfully using the status endpoint
- Verify the category name and version match your configuration
- Wait a few minutes after training completes

### "Incident not deployed"

- Wait at least 10 seconds after deploying before attempting detection
- Check deployment status using the GET incident details endpoint

### REST Client not working

- Install the [REST Client extension](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)
- Ensure `.env` file is properly configured
- Check that variables are loading correctly (`{{$dotenv VARIABLE_NAME}}`)

### Storage access issues (Standard mode)

- Verify your Content Safety resource has **Managed Identity** enabled
- Ensure the Managed Identity has **Storage Blob Data Contributor** or **Storage Blob Data Owner** role assigned on your storage account
- Confirm your storage account is NOT a hierarchical namespace (Data Lake) account
- Check the blob URL is accessible and formatted correctly

### Region not supported

- Custom categories are only available in specific regions. See [Region availability](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/overview#region-availability)
- Supported regions for custom categories (standard): **East US**, **East US 2**, **France Central**, **Sweden Central**, **UK South**, **West US**

## Resources

- [Azure AI Content Safety Documentation](https://learn.microsoft.com/azure/ai-services/content-safety/)
- [Custom Categories Overview](https://learn.microsoft.com/azure/ai-services/content-safety/how-to/custom-categories)
- [REST API Reference](https://learn.microsoft.com/rest/api/cognitiveservices/contentsafety/)

## License

This project is licensed under the MIT License - see below for details.

**MIT License**

Copyright (c) 2025 sjuratov

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---

**Note**: This is a demonstration project. When using Azure AI Content Safety services, you must comply with [Azure Content Safety terms of service](https://azure.microsoft.com/support/legal/) and [Responsible AI principles](https://www.microsoft.com/ai/responsible-ai).
