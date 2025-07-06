# Scheduled Text Extractor

An n8n workflow that automatically extracts text from images in a Google Drive folder using OpenAI's vision model and saves the structured data to Google Sheets.

## Overview

This workflow performs the following tasks:
1. Periodically checks a specified Google Drive folder for new images
2. Downloads new images that haven't been processed yet
3. Uses OpenAI's GPT-4 Vision model to extract text and categorize the content
4. Saves the extracted information to a Google Sheets document in a structured format

## Prerequisites

- n8n instance
- Google Drive API credentials
- Google Sheets API credentials
- OpenAI API key

## Required API Credentials

1. Google Drive OAuth2 API
2. Google Sheets OAuth2 API
3. OpenAI API

## Workflow Components

### 1. Schedule Trigger
- Runs at specified intervals
- Configured to trigger based on minute intervals

### 2. Google Drive
- Monitors a specific folder (ID: 18N7AJb3uMoA7Rj09f-O_WpMwNV_yboun)
- Lists all files in the folder
- Filter settings for image files

### 3. Google Sheets Integration
- Initial read operation to check for previously processed files
- Append operation to add new extracted data
- Target spreadsheet ID: 12vaVr0izoR_v23eEwRzqOCtoXuhOXixaHGfQPPbkrrc

### 4. File Processing
- Downloads new image files
- Filters out already processed files using JavaScript code
- Only processes files with 'IMG_' in the filename

### 5. OpenAI Vision Model
- Uses GPT-4-O-mini model for text extraction
- Analyzes images to extract:
  - Dates (YYYY-MM-DD format)
  - Categories (금, 금리, 주식, 부동산)
  - Full text content
- Returns data in structured JSON format

### 6. Data Structure
The workflow extracts and stores the following information:
```json
{
  "date": "YYYY-MM-DD",
  "filename": "original_filename",
  "category": "category_name",
  "desc": "extracted_text"
}
```

## Output Format

The data is saved to Google Sheets with the following columns:
- Category
- Date
- Desc (Description/Content)
- FileName

## Error Handling

- JavaScript code nodes include error handling and data validation
- Regular expressions are used to properly format JSON responses
- Text processing includes cleaning of special characters and line breaks

## Setup Instructions

1. Import the workflow into your n8n instance
2. Configure the following credentials:
   - Google Drive OAuth2
   - Google Sheets OAuth2
   - OpenAI API
3. Update the Google Drive folder ID and Google Sheets document ID
4. Adjust the schedule trigger interval as needed
5. Activate the workflow

## Notes

- The workflow is designed to avoid duplicate processing by tracking processed files
- Text extraction is optimized for Korean language content
- JSON responses are cleaned and formatted before being saved to the spreadsheet
- Line breaks are converted to spaces in the final output
- Regular expressions are used to ensure proper JSON formatting

## Testing

Before activating the workflow:
1. Verify API credentials are properly configured
2. Test with a single image file
3. Check the Google Sheets output
4. Monitor the execution logs for any errors

## Maintenance

- Regularly check the Google Sheets document for proper data formatting
- Monitor OpenAI API usage
- Review execution logs for any recurring errors
- Update API credentials before they expire