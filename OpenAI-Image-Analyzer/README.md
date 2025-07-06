# OpenAI Image Analyzer Workflow Documentation

## Overview
This n8n workflow is designed to automatically process images stored in Google Drive, extract text content using OpenAI's vision model, and store the results in a Google Spreadsheet. The workflow monitors a specific Google Drive folder for new or updated image files and processes them automatically.

## Core Components

### Triggers
- **File Created Trigger**: Monitors a specific Google Drive folder for new files
- **File Updated Trigger**: Monitors the same folder for file updates
- Both triggers run every minute

### File Processing
1. **Set File ID Node**: Extracts key file metadata:
   - File ID
   - File Type (MIME type)
   - File Name

2. **Download File Node**: Downloads the image file from Google Drive for processing

3. **Loop Over Items Node**: Manages batch processing of multiple files

### Image Analysis
- **OpenAI Node**: Uses GPT-4-Vision (gpt-4o-mini model) to analyze images
- Extracts based on the gien prompt

### Data Processing
- **Code Node**: Processes the OpenAI response:
  - Extracts information using regular expressions
  - Cleans up the text content
  - Formats data for spreadsheet storage
  - Handles text formatting and special characters

### Data Storage
- **Google Sheets Node**: Appends extracted data to a specified spreadsheet
- Columns include:
  - Category
  - Date
  - Description
  - Filename

## Limitations
1. **Processing Frequency**
   - File Created node only processes one item per minute
   - Loop Over Items has limited batch processing capability

2. **Multiple File Handling**
   - When multiple files are added simultaneously, only the first item typically gets processed

## Technical Notes
- The workflow includes error handling and file type validation
- Text extraction is optimized for Korean language content
- JSON responses are carefully parsed to maintain data integrity
- All line breaks are converted to spaces in the final output
- Multiple whitespace characters are consolidated

## Integration Points
- Google Drive API
- OpenAI API (Vision Model)
- Google Sheets API

## Authentication
The workflow requires authentication for:
- Google Drive access
- Google Sheets access
- OpenAI API access
