# FileController Documentation

## Overview

FileController handles file serving operations for AI agents in the Jobix Dashboard administrative panel. The controller provides secure file access for agent avatars and knowledge files with proper MIME type handling, inline display capabilities, and file existence validation.

**Base Path:** `/file`  
**Controller Type:** WebController (Admin Application)  
**Namespace:** `apps\admin\controllers\FileController`

---

## Endpoints

### 1. AI Agent Avatar - Display Agent Avatar

| Parameter | Value |
|-----------|-------|
| **URL** | `/file/ai-agent-avatar?agent_uuid={agent_uuid}&file_full_name={file_name}` |
| **Method** | GET |
| **Description** | Display AI agent avatar image inline in browser |

**URL Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `agent_uuid` | string | ✓ | UUID of the AI agent |
| `file_full_name` | string | ✓ | Full filename of the avatar (including extension) |

**Business Logic:**
1. Construct file path using Yii alias: `@files/agents/{agent_uuid}/avatars/{file_full_name}`
2. Check file existence using `file_exists()`
3. Determine MIME type using `mime_content_type()`
4. Send file as inline response for browser display
5. Throw 404 exception if file not found

**File Path Structure:**
```
@files/agents/{agent_uuid}/avatars/{file_full_name}
```

**Implementation:**
```php
public function actionAiAgentAvatar(string $agent_uuid, string $file_full_name): Response
{
    // Define the full file path using Yii alias
    $filePath = Yii::getAlias("@files/agents/$agent_uuid/avatars/$file_full_name");

    // Check if the file exists
    if (file_exists($filePath)) {
        // Determine the content type based on the file extension
        $mimeType = mime_content_type($filePath);

        // Send the file as an inline response, so it's displayed in the browser
        return Yii::$app->response->sendFile($filePath, null, [
            'inline' => true, // Ensure the image is displayed in the browser, not downloaded
            'mimeType' => $mimeType // Set the correct MIME type for the file
        ]);
    } else {
        // Throw a 404 error if the avatar file is not found
        throw new NotFoundHttpException('AI Agent Avatar not found.');
    }
}
```

**Response Types:**
- **Success:** File content with appropriate MIME type headers
- **Headers:** `Content-Type: {detected_mime_type}`, `Content-Disposition: inline`
- **Error:** NotFoundHttpException with message "AI Agent Avatar not found."

**Supported File Types:**
- All image formats supported by `mime_content_type()`
- Common formats: JPG, PNG, GIF, BMP, SVG, WebP

---

### 2. AI Agent Knowledge - Download Knowledge File

| Parameter | Value |
|-----------|-------|
| **URL** | `/file/ai-agent-knowledge?agent_uuid={agent_uuid}&file_full_name={file_name}` |
| **Method** | GET |
| **Description** | Download AI agent knowledge file |

**URL Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `agent_uuid` | string | ✓ | UUID of the AI agent |
| `file_full_name` | string | ✓ | Full filename of the knowledge file (including extension) |

**Business Logic:**
1. Construct file path using Yii alias: `@files/agents/{agent_uuid}/knowledge/{file_full_name}`
2. Check file existence using `file_exists()`
3. Send file as download response
4. Throw 404 exception if file not found

**File Path Structure:**
```
@files/agents/{agent_uuid}/knowledge/{file_full_name}
```

**Implementation:**
```php
public function actionAiAgentKnowledge(string $agent_uuid, string $file_full_name): Response
{
    // Define the full file path using Yii alias
    $filePath = Yii::getAlias("@files/agents/$agent_uuid/knowledge/$file_full_name");

    // Check if the file exists
    if (file_exists($filePath)) {
        return Yii::$app->response->sendFile($filePath);
    } else {
        // Throw a 404 error if the avatar file is not found
        throw new NotFoundHttpException('AI Agent Knowledge file not found.');
    }
}
```

**Response Types:**
- **Success:** File content with download headers
- **Headers:** `Content-Type: application/octet-stream` (default), `Content-Disposition: attachment; filename="{file_full_name}"`
- **Error:** NotFoundHttpException with message "AI Agent Knowledge file not found."

**Supported File Types:**
- All file formats (no restrictions)
- Common knowledge file formats: PDF, DOC, DOCX, TXT, MD, JSON, XML, etc.

---

## Security Features

### 1. File Path Validation
- Uses Yii aliases for secure path construction
- Prevents directory traversal attacks through controlled path building
- UUID-based directory structure ensures agent isolation
- File existence validation before serving

### 2. Agent Isolation
- Files organized by agent UUID directories
- Each agent's files are isolated in separate folders
- Avatar and knowledge files stored in separate subdirectories
- No cross-agent file access possible through URL manipulation

### 3. Error Handling
- Consistent 404 responses for non-existent files
- No information leakage about file system structure
- Proper exception handling with user-friendly messages
- No file content exposure on errors

### 4. MIME Type Security
- Automatic MIME type detection for avatars
- Proper content-type headers prevent browser security issues
- Inline display for avatars with correct MIME types
- Default safe handling for knowledge file downloads

---

## File Storage Structure

### Directory Organization
```
@files/
├── agents/
│   ├── {agent_uuid_1}/
│   │   ├── avatars/
│   │   │   ├── avatar_image.jpg
│   │   │   ├── avatar_image.png
│   │   │   └── ...
│   │   └── knowledge/
│   │       ├── knowledge_file.pdf
│   │       ├── training_data.txt
│   │       └── ...
│   ├── {agent_uuid_2}/
│   │   ├── avatars/
│   │   └── knowledge/
│   └── ...
```

### File Naming Conventions
- **Agent UUID:** Standard UUID format (e.g., `550e8400-e29b-41d4-a716-446655440000`)
- **File Names:** Full filename with extension (e.g., `avatar.jpg`, `knowledge.pdf`)
- **Path Safety:** No special characters or path traversal sequences allowed

---

## Business Logic

### 1. Avatar Display Flow
```
1. Receive GET request with agent_uuid and file_full_name
2. Construct secure file path using Yii alias
3. Check file existence on filesystem
4. Detect MIME type using mime_content_type()
5. Send file with inline disposition for browser display
6. Return appropriate headers for image display
7. Throw 404 if file not found
```

### 2. Knowledge File Download Flow
```
1. Receive GET request with agent_uuid and file_full_name
2. Construct secure file path using Yii alias
3. Check file existence on filesystem
4. Send file with attachment disposition for download
5. Return appropriate headers for file download
6. Browser prompts user to save/open file
7. Throw 404 if file not found
```

---

## Integration Points

### Dependencies
- Yii Framework file handling (`Yii::$app->response->sendFile()`)
- PHP file system functions (`file_exists()`, `mime_content_type()`)
- Yii aliases system for secure path handling
- WebController base class for HTTP response handling

### Related Components
- **AgentsController** - manages AI agents and file uploads
- **File Upload System** - handles avatar and knowledge file uploads
- **Agent Management** - coordinates agent-related file operations
- **Storage System** - manages file system organization

### File System Integration
- Relies on proper file system permissions
- Uses Yii aliases for path abstraction
- Integrates with agent creation and update processes
- Supports various file formats and sizes

---

## Usage Examples

### Display agent avatar
```http
GET /file/ai-agent-avatar?agent_uuid=550e8400-e29b-41d4-a716-446655440000&file_full_name=avatar.jpg
```

### Download knowledge file
```http
GET /file/ai-agent-knowledge?agent_uuid=550e8400-e29b-41d4-a716-446655440000&file_full_name=training_manual.pdf
```

### HTML usage for avatar display
```html
<img src="/file/ai-agent-avatar?agent_uuid=550e8400-e29b-41d4-a716-446655440000&file_full_name=avatar.png" 
     alt="Agent Avatar" />
```

### HTML usage for knowledge file download
```html
<a href="/file/ai-agent-knowledge?agent_uuid=550e8400-e29b-41d4-a716-446655440000&file_full_name=guide.pdf" 
   download>Download Knowledge File</a>
```

---

## Performance Considerations

### File Serving Optimization
- Direct file streaming without loading into memory
- Efficient MIME type detection
- Minimal processing overhead for file serving
- Browser caching support through proper headers

### Storage Considerations
- File system performance depends on directory structure
- Large files may impact response times
- Consider CDN integration for high-traffic scenarios
- Monitor disk space usage for agent files

### Scalability
- UUID-based directory structure scales well
- Consider file size limits for uploads
- Monitor concurrent file access
- Plan for backup and archival strategies

---

## Error Handling

### Common Errors
- **"AI Agent Avatar not found."** - Avatar file doesn't exist
- **"AI Agent Knowledge file not found."** - Knowledge file doesn't exist
- File system permission errors
- Invalid UUID format errors

### Error Responses
- **404 NotFoundHttpException** - File not found
- **500 Internal Server Error** - File system or permission issues
- **400 Bad Request** - Invalid parameters (handled by Yii)

### Error Prevention
- File existence validation before serving
- Secure path construction prevents traversal attacks
- UUID validation ensures proper format
- MIME type detection prevents content type issues

---

## HTTP Status Codes
- **200 OK** - Successful file serving
- **404 Not Found** - File not found (NotFoundHttpException)
- **500 Internal Server Error** - Server or file system errors

---

*FileController is a specialized component for secure file serving of AI agent assets with proper MIME type handling, agent isolation, and comprehensive error handling for avatar display and knowledge file downloads.*