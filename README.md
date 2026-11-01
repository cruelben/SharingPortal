# SharingPortal
## Technical Documentation and Project Reset Prompt

**Current version:** 1.6.0  
**Previous version:** 1.5.0  
**Repository:** https://github.com/cruelben/SharingPortal  
**Live application:** https://cruelben.github.io/SharingPortal/  
**Cloudflare Worker:** https://sharingportal-upload.cruelben.workers.dev/

---

# 1. PROJECT OBJECTIVE

SharingPortal is a lightweight web application designed to transfer files between people using a short sharing code.

Its main purpose is to allow a sender to upload one or more files to Dropbox and share a single short code with a recipient.

The recipient can enter that code on the SharingPortal website and access the corresponding files without having a Dropbox account or logging into Dropbox.

The application uses:

- Dropbox for file storage and public shared links.
- Cloudflare Workers for backend API operations.
- Cloudflare KV for storing codes and file/group metadata.
- GitHub Pages for hosting the frontend.

The recipient does not need to authenticate.

A valid code provides access to the public Dropbox links associated with that code.

**Important security consideration:** A sharing code behaves like an access key. Anyone who knows the code can retrieve the associated files. Users must not upload confidential or sensitive material.

## Main features

- Upload one or multiple files.
- Generate a five-character sharing code.
- Share files without requiring the recipient to have a Dropbox account.
- Download files individually.
- View previously uploaded files and groups.
- Copy sharing codes.
- Delete uploaded files or entire groups.
- Support both dark and light themes.
- Support large files through Dropbox upload sessions.
- Preserve original filenames.
- Maintain compatibility with legacy records from version 1.5.0.
- Application icons for browsers, Apple devices, and PWA installation.
- PWA support via `site.webmanifest`.
- Donation links (Ko-fi, PayPal) in the footer.

---

# 2. EXECUTION ENVIRONMENT AND TECHNOLOGIES

## Frontend

The frontend is a single HTML file:

`index.html`

Technologies:

- HTML5
- CSS3
- Vanilla JavaScript
- Tailwind CSS (CDN)
- Lucide Icons (CDN)
- Dropbox API
- Browser Fetch API
- Web Crypto API

No frontend framework or build process is required.

The application runs directly in a modern browser.

## Hosting

GitHub Pages hosts the frontend.

Repository:

https://github.com/cruelben/SharingPortal

Published address:

https://cruelben.github.io/SharingPortal/

## Backend

The backend is a Cloudflare Worker.

Address:

https://sharingportal-upload.cruelben.workers.dev/

The Worker handles:

- Registration of uploaded files and groups.
- Generation and validation of sharing-code records.
- Retrieval of files using a sharing code.
- Listing of the sender's uploads.
- Deletion of registered files and groups.
- Dropbox account ownership verification.
- Basic request rate limiting.

## Database

Cloudflare KV is used for lightweight metadata storage.

KV binding:

`SP_KV`

Namespace:

`SharingPortal_KV`

No R2 bucket or paid storage service is required by the current architecture.

## Dropbox

Dropbox application:

`SharingPortal`

Dropbox App Key:

`pjr4dcuo0wcdfpb`

Redirect URI:

`https://cruelben.github.io/SharingPortal/`

Application permission scopes:

```text
files.content.write sharing.write account_info.read
```

Dropbox is configured to use its App Folder.

---

# 3. ARCHITECTURE

The application consists of two independent components.

## 3.1 Frontend: index.html

The frontend is responsible for:

1. User interface.
2. Dropbox authentication.
3. File selection.
4. File uploads.
5. Dropbox shared-link creation.
6. Communication with Cloudflare Worker.
7. Code redemption.
8. Queue visualization.
9. Deletion requests.

The frontend does not directly store sharing-code records in the browser.

All persistent metadata is managed by the Worker and Cloudflare KV.

## 3.2 Backend: Cloudflare Worker

The Worker exposes HTTP endpoints.

Its responsibilities include:

- Authenticating the sender using the Dropbox access token.
- Identifying the Dropbox account.
- Registering metadata.
- Associating sharing codes with records.
- Returning public file links.
- Listing records belonging to the authenticated account.
- Deleting files or groups from Dropbox.
- Removing corresponding KV records.

The Worker is independent of the frontend and can be updated separately.

---

# 4. FRONTEND STRUCTURE

The HTML application is organized into three main views.

## 4.1 Send

The Send section provides:

- File selection through the browser.
- Drag-and-drop support.
- Multiple-file selection.
- Selected-file preview.
- Individual file removal.
- Complete selection clearing.
- Upload progress.
- Group registration.
- Sharing-code display.

In version 1.6.0, multiple selected files are registered under a single sharing code.

## 4.2 Receive

The Receive section allows the recipient to:

1. Enter a five-character code.
2. Submit the code to the Worker.
3. Retrieve the associated files.
4. Download each file individually.

The recipient does not need Dropbox authentication.

The application receives an array of file records and displays a separate download link for each file.

## 4.3 Queue

The Queue section displays records belonging to the connected Dropbox account.

It supports:

- Listing previous uploads.
- Copying a sharing code.
- Displaying file names and sizes.
- Refreshing the list.
- Deleting individual legacy records.
- Deleting complete groups.

The queue must support both legacy single-file records and new group records.

---

# 5. IMPORTANT JAVASCRIPT FUNCTIONS

The frontend JavaScript is organized into logical sections.

## 5.1 General utilities

```javascript
const $ = id => document.getElementById(id);
```

Other utilities:

- `fmtSize(n)` - Formats file sizes.
- `esc(s)` - Escapes HTML-sensitive characters.
- `toast(msg)` - Displays notifications.
- `setConnected(ok, sub)` - Updates Dropbox connection status.

## 5.2 Navigation

```javascript
showTab(tab)
```

Controls the three application views:

- send
- receive
- queue

Selecting the Queue tab automatically triggers a queue refresh.

## 5.3 Theme

The application supports dark and light themes.

The selected theme is saved in:

```javascript
localStorage.setItem('sp_theme', 'light');
```

The default theme is dark.

## 5.4 Dropbox OAuth PKCE

Authentication uses OAuth 2.0 Authorization Code with PKCE.

Main functions:

```javascript
randomString()
base64url()
sha256()
connectDropbox()
handleOAuth()
identifyAccount()
```

Authentication flow:

1. Generate a random PKCE verifier.
2. Generate an OAuth state value.
3. Calculate the SHA-256 challenge.
4. Redirect the browser to Dropbox authorization.
5. Receive the authorization code.
6. Verify the OAuth state.
7. Exchange the authorization code for an access token.
8. Retrieve the Dropbox account information.

The access token is kept in JavaScript memory only.

It is not permanently stored in localStorage.

After reloading the page or losing the session, the user must reconnect Dropbox.

## 5.5 File selection

Main functions:

```javascript
addFiles(files)
renderSelected()
```

The application supports:

- Browser file picker.
- Drag and drop.
- Multiple files.
- Removing individual selected files.
- Clearing the selection.

Version 1.6.0 limits a group to:

```javascript
const MAX_FILES_PER_GROUP = 100;
```

## 5.6 Dropbox API

Main functions:

```javascript
dropboxRequest(endpoint, args)
createDropboxFolder(path)
deleteDropboxFolder(path)
uploadSmall(file, path)
uploadLarge(file, path, onProgress)
createSharedLink(path)
```

### Small files

Files up to 150 MiB use the standard Dropbox upload endpoint.

### Large files

Files larger than 150 MiB use Dropbox upload sessions.

Chunk size:

```javascript
const CHUNK_SIZE = 8 * 1024 * 1024;
```

The upload session uses:

- `/files/upload_session/start`
- `/files/upload_session/append_v2`
- `/files/upload_session/finish`

Progress is calculated using the total size of the selected files.

## 5.7 Cloudflare Worker communication

Main function:

```javascript
workerCall(path, options)
```

It handles HTTP requests to the Worker.

Authenticated operations include the Dropbox access token in the Authorization header.

## 5.8 Code generation

Function:

```javascript
randomCode()
```

The application generates a five-character code using:

```javascript
ABCDEFGHJKLMNPQRSTUVWXYZ23456789
```

Ambiguous characters such as I, O, 0 and 1 are excluded.

---

# 6. VERSION 1.6.0: GROUP UPLOAD SYSTEM

This is the main architectural change introduced in version 1.6.0.

## Previous behavior: version 1.5.0

In version 1.5.0:

- Every selected file was uploaded independently.
- Every file received its own sharing code.
- Every file was registered separately with the Worker.

For example:

```text
File A -> Code A123B
File B -> Code C456D
File C -> Code E789F
```

This required the sender to share multiple codes.

## New behavior: version 1.6.0

In version 1.6.0:

- The sender selects multiple files.
- The application creates one Dropbox folder for the group.
- Each file is uploaded separately.
- Each file receives its own Dropbox shared link.
- All file metadata is registered together.
- The Worker associates one sharing code with the entire group.

Example:

```text
Sharing code: A7K9Q

Group:
  document.pdf
  photo.jpg
  archive.dat
  presentation.pptx
```

The recipient enters `A7K9Q` and receives access to all four files.

Files remain individual files.

No ZIP archive is generated.

## Dropbox folder structure

Each group has its own folder.

Example:

```text
/SharingPortal_1730000000_UUID/
    /001/
        document.pdf
    /002/
        photo.jpg
    /003/
        archive.dat
```

The numbered subfolders avoid filename conflicts when multiple selected files have the same original name.

The filename itself is preserved.

For example:

```text
Original filename: document.pdf
Dropbox filename:  document.pdf
```

The timestamp and UUID are used in the group folder name, not added to the file name.

## Group metadata

The frontend registers the group using:

```javascript
{
  code: "A7K9Q",
  folderPath: "/SharingPortal_...",
  files: [
    {
      name: "document.pdf",
      size: 123456,
      link: "https://www.dropbox.com/...",
      path: "/SharingPortal_.../001/document.pdf"
    },
    {
      name: "photo.jpg",
      size: 654321,
      link: "https://www.dropbox.com/...",
      path: "/SharingPortal_.../002/photo.jpg"
    }
  ],
  createdAt: "2026-..."
}
```

The Worker receives this payload through:

```text
POST /api/register-group
```

Only one registration request is required for the group.

---

# 7. CLOUDFLARE WORKER API

The Worker supports legacy single-file records and new group records.

## 7.1 Register a legacy file

```text
POST /api/register
```

Used by version 1.5.0.

Example payload:

```json
{
  "code": "A123B",
  "name": "document.pdf",
  "size": 123456,
  "link": "https://www.dropbox.com/...",
  "path": "/document.pdf",
  "createdAt": "2026-..."
}
```

This endpoint must remain available for backward compatibility.

## 7.2 Register a group

```text
POST /api/register-group
```

Used by version 1.6.0.

Example payload:

```json
{
  "code": "A7K9Q",
  "folderPath": "/SharingPortal_...",
  "files": [
    {
      "name": "document.pdf",
      "size": 123456,
      "link": "https://www.dropbox.com/...",
      "path": "/SharingPortal_.../001/document.pdf"
    }
  ],
  "createdAt": "2026-..."
}
```

The Worker stores the group and associates the code with the group record.

## 7.3 Redeem a code

```text
GET /api/redeem?code=A7K9Q
```

Returns the files associated with the code.

Expected response structure:

```json
{
  "items": [
    {
      "name": "document.pdf",
      "size": 123456,
      "link": "https://www.dropbox.com/...",
      "createdAt": "2026-..."
    }
  ]
}
```

For a group, `items` contains all the files in the group.

For a legacy record, `items` contains one file.

The frontend is designed to support both cases.

## 7.4 List registered items

```text
GET /api/list
```

Requires Dropbox authentication.

Returns the records belonging to the authenticated account.

The response must support:

- Legacy single-file records.
- New group records.

Group records contain a `files` array.

## 7.5 Delete an item or group

```text
DELETE /api/item
```

Request body:

```json
{
  "id": "record-id",
  "code": "A7K9Q"
}
```

Requires Dropbox authentication.

For legacy records, the Worker deletes the corresponding Dropbox file.

For group records, the Worker deletes the Dropbox group folder and removes the associated metadata from KV.

The Worker must verify record ownership before deletion.

---

# 8. CLOUDFLARE KV DATA MODEL

The Worker uses KV keys to associate records with codes and owners.

Legacy records use keys such as:

```text
item:<id>
code:<code>
owner:<accountId>:<id>
```

New group records use:

```text
group:<id>
code:<code>
owner:<accountId>:<id>
```

For a group, the code mapping contains a value identifying the group:

```text
group:<id>
```

The Worker must distinguish between:

- A legacy record ID.
- A group record ID.

Existing version 1.5.0 records must continue to work after the Worker upgrade.

Do not remove or invalidate legacy KV records during routine upgrades.

---

# 9. SECURITY AND TECHNICAL CONSTRAINTS

Important rules:

1. Dropbox access tokens must not be stored permanently in frontend storage.
2. Recipients must not need Dropbox authentication.
3. Only the sender's authenticated Dropbox account can list or delete their records.
4. Sharing codes must be validated by the Worker.
5. Dropbox shared links must be generated with public viewer access and download permission.
6. The frontend must escape user-controlled text before inserting it into HTML.
7. The Worker must validate Dropbox paths and shared-link hosts.
8. The Worker must enforce request-size and rate limits.
9. The Worker CORS configuration must allow the official GitHub Pages origin.
10. Group records must not exceed the configured Worker request limits.

The Worker configuration includes:

```text
SP_KV -> SharingPortal_KV
```

The Worker also uses an approximate IP-based rate limit.

The existing Worker implementation uses a limit of approximately 30 requests per IP within 10 minutes.

The group implementation supports a maximum of 100 files per group.

The Worker request-size limit was increased for group metadata.

---

# 10. PREVIOUSLY VERIFIED FUNCTIONALITY

Version 1.5.0 was tested successfully.

The following behaviors were verified:

- Dropbox account connection.
- Dropbox account identification.
- Uploading one file.
- Uploading multiple files.
- Generation of individual codes.
- Registration with Cloudflare Worker.
- Retrieval using a sharing code.
- Download from an incognito Chrome session.
- Download without Dropbox login.
- Queue listing.
- Dropbox account displayed as CRBGITHUB.

The recipient was able to retrieve files without a Dropbox account.

One issue identified in version 1.5.0 was that Dropbox filenames included a timestamp and UUID prefix.

Version 1.6.0 changes the folder structure so the original filename can be retained.

---

# 11. CURRENT VERSION 1.6.0 STATUS

The following changes have been implemented in the frontend code:

- Single code for an entire group.
- Dedicated Dropbox folder for every group.
- Numbered subfolders for individual files.
- Original filenames preserved in Dropbox paths.
- Individual shared links.
- Individual downloads for recipients.
- Group-aware queue display.
- Group-aware deletion requests.
- Legacy single-file compatibility.
- Maximum of 100 files per group.
- Version number displayed in the footer.
- PWA support and web manifest.
- Custom app icons (favicon, Apple touch, PWA icons).
- Donation links in the footer.

The Worker was updated separately to version 1.6.0.

The new frontend code has been prepared using the original version 1.5.0 HTML supplied by the user.

**Important:** The complete version 1.6.0 end-to-end workflow has not yet been confirmed by a successful user test.

Do not assume that every new feature has already been verified.

---

# 12. TEST PLAN FOR VERSION 1.6.0

After publishing the new `index.html` on GitHub Pages, perform the following tests.

## Test 1: Dropbox connection

1. Open the application.
2. Click Connect Dropbox.
3. Authorize the application.
4. Confirm that the connected account is displayed.

## Test 2: Group upload

Select three files with different names.

Expected result:

- All three files are uploaded.
- One Dropbox group folder is created.
- Three numbered subfolders are created.
- Three individual shared links are generated.
- Only one sharing code is displayed.

## Test 3: Duplicate filenames

Select two files with the same filename from different local folders.

Expected result:

- Both files are uploaded.
- Their original names are preserved.
- They do not overwrite one another.
- They are stored in separate numbered subfolders.

## Test 4: Group redemption

1. Open an incognito Chrome window.
2. Open SharingPortal.
3. Do not connect Dropbox.
4. Enter the group code.

Expected result:

- All files in the group are displayed.
- Every file has its own download button.
- Each file can be downloaded independently.

## Test 5: Queue

Connect Dropbox and open the Queue tab.

Expected result:

- The new group appears as one record.
- The group code is displayed once.
- The file names and sizes are visible.
- The code can be copied.

## Test 6: Group deletion

Delete the group from the Queue.

Expected result:

- The Worker verifies ownership.
- The Dropbox group folder is deleted.
- The group record is removed from KV.
- The sharing code stops working.

## Test 7: Legacy compatibility

If old version 1.5.0 records are still present:

- They must remain visible in the queue.
- Their codes must still work.
- Their files must still be downloadable.
- They must still be deletable.

---

# 13. KNOWN POINTS TO CHECK

The following points should be checked during testing and must not be silently assumed to work.

## 13.1 Dropbox folder deletion

Verify that the Worker can delete a non-empty Dropbox group folder using the Dropbox `files/delete_v2` endpoint.

If Dropbox rejects deletion of a non-empty folder, implement deletion of the individual files before deleting the folder.

Do not leave orphaned files in Dropbox.

## 13.2 Filename preservation

Verify the actual downloaded filename in Chrome.

The frontend preserves the original filename in the Dropbox upload path and metadata.

However, the final download filename depends on Dropbox shared-link behavior and the browser.

## 13.3 Sharing-code collision

The code contains five characters.

The Worker must reject an already-used code.

The frontend currently generates a code and submits the group registration.

A future improvement could retry with a different code if the Worker reports a collision.

## 13.4 Partial upload failure

If a file upload or shared-link creation fails before group registration:

- The frontend attempts to delete the incomplete group folder.
- The user receives an error message.
- The selected files remain available for another attempt.

Verify that this cleanup works correctly.

If group registration succeeds, the frontend must not delete the group folder because of a later display error.

## 13.5 Worker/frontend compatibility

The frontend depends on the Worker version 1.6.0 endpoint:

```text
POST /api/register-group
```

The old endpoint:

```text
POST /api/register
```

must remain available for legacy records and backward compatibility.

Do not revert the Worker to the old version.

---

# 14. DEVELOPMENT RULES FOR FUTURE CHANGES

When continuing development, follow these rules.

1. Preserve the existing visual design unless a redesign is explicitly requested.
2. Keep the application as a single `index.html` file unless there is a compelling technical reason to split it.
3. Do not introduce unnecessary frameworks or build tools.
4. Keep the Dropbox OAuth PKCE implementation.
5. Do not store access tokens permanently in localStorage.
6. Preserve the dark and light themes.
7. Preserve the three existing sections: Send, Receive and Queue.
8. Maintain backward compatibility with version 1.5.0 records.
9. Do not generate ZIP archives.
10. Keep files individually downloadable.
11. Preserve original filenames.
12. Keep the single-code-per-group behavior.
13. Keep the version number visible in the footer.
14. Do not change Dropbox App Key, redirect URI or Worker URL without explicit authorization.
15. Do not remove working functionality when introducing new features.
16. Test JavaScript syntax before delivering a complete replacement file.
17. Prefer providing complete files rather than isolated fragments because the user wants to replace the old file with a ready-to-use version.
18. Clearly distinguish tested functionality from proposed or unverified changes.

The user prefers complete source files ready to copy and paste.

---

# 15. PROJECT CONFIGURATION REFERENCE

| Component | Configuration |
|---|---|
| Project | SharingPortal |
| Current version | 1.6.0 |
| Frontend | Single HTML file |
| Frontend hosting | GitHub Pages |
| Repository | cruelben/SharingPortal |
| Dropbox application | SharingPortal |
| Dropbox App Key | pjr4dcuo0wcdfpb |
| Redirect URI | https://cruelben.github.io/SharingPortal/ |
| Worker | sharingportal-upload.cruelben.workers.dev |
| KV binding | SP_KV |
| KV namespace | SharingPortal_KV |
| Upload chunk size | 8 MiB |
| Large-file threshold | 150 MiB |
| Maximum group size | 100 files |
| Sharing-code length | 5 characters |
| Recipient authentication | Not required |
| File packaging | Individual files, no ZIP |
| App Icons | `favicon.ico`, `icon-192.png`, `icon-512.png`, `apple-touch-icon.png`, `icon.svg` |
| PWA Manifest | `site.webmanifest` |

---

# 16. APPLICATION ICONS AND PWA

The application uses a set of custom icons based on the header logo:

- Blue background (`#2563eb`).
- White filled paper plane (not outline).
- Rounded corners (~22%).
- Medium padding (~15%).

The following files are placed in the repository root:

- `favicon.ico` (multi-size: 16, 32, 48)
- `icon-192.png`
- `icon-512.png`
- `apple-touch-icon.png` (180×180)
- `icon.svg` (vector)
- `site.webmanifest` (PWA manifest)

A local tool `icon-generator.html` was used to generate these assets from the SVG source. It does not need to be uploaded to the repository.

The `<head>` of `index.html` includes the correct links for browsers, Apple devices, and PWA installation, using the `/SharingPortal/` base path.

---

# 17. SUPPORT THE PROJECT

If you find SharingPortal useful, you can support its development via Ko-fi or PayPal.

See [pay.md](pay.md) for details.

---

# 18. PROMPT FOR THE NEXT ASSISTANT

You are continuing development of an existing project called SharingPortal.

The project is a browser-based file-sharing application built with HTML, CSS and Vanilla JavaScript.

It uses:

- GitHub Pages for frontend hosting.
- Dropbox API for file storage and shared links.
- Cloudflare Workers for backend operations.
- Cloudflare KV for metadata and sharing-code storage.

The current target version is 1.6.0.

The main change from version 1.5.0 is that multiple files are now registered under one five-character sharing code.

Each file remains separate in Dropbox and can be downloaded individually.

The application must preserve original filenames and support both new group records and legacy single-file records.

The frontend also includes custom app icons, a `site.webmanifest` for PWA support, and a donation section in the footer.

The Cloudflare Worker has already been updated to support:

- `/api/register`
- `/api/register-group`
- `/api/redeem`
- `/api/list`
- `/api/item`

The frontend uses `/api/register-group` to register a group.

The user has supplied the complete original version 1.5.0 `index.html` and received a complete version 1.6.0 replacement.

The new version has not yet been fully tested end-to-end.

## Your first task

Continue from the current project state.

Do not redesign the application.

Do not invent missing code or assume that untested functionality works.

First, establish which version of `index.html` is currently published and whether the version 1.6.0 tests have been completed.

If a problem is reported, investigate the actual code and the relevant Worker endpoint before proposing changes.

When modifications are necessary, provide the complete corrected file rather than small patches.

The user prefers instructions in Italian, even though this README and the source code documentation are in English.

Keep explanations practical and proceed one step at a time.

The primary objective is to finish testing and stabilizing SharingPortal 1.6.0 without breaking the functionality already verified in version 1.5.0.

---

**End of README.md**