# Filey - A Web-Based File Browser and Streamer

Filey is a lightweight, web-based file browser built with Python and Tornado. It provides a simple and secure interface for browsing local directories, uploading files (including nested folders via drag-and-drop), and streaming file content in real-time, similar to `tail -f`.

## Features

- **File & Directory Browsing:** Navigate your local filesystem through a clean, black-and-white themed web interface.
- **Drag-and-Drop Uploads:** Easily upload files and entire folder structures by dragging them into the browser window.
- **Real-Time File Streaming:** Open any file in streaming mode to see new lines appended in real-time, perfect for monitoring logs or other active files. The stream will also show the last 100 lines for immediate context.
- **Secure Access:** The entire application is protected by a token-based authentication system.
- **Admin Panel:** A separate admin panel to enable/disable features like file upload, delete, and rename.
- **Rename & Delete:** Quickly rename or remove files and folders directly from the directory listing.
- **Configurable:** Specify the root directory, port, and token via a JSON config file or command-line options.

## Setup and Installation

1.  **Install Dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

2.  **Run the Application:**
    ```bash
    python -m wb --root /path/to/serve --port 8000 --token YOUR_TOKEN --admin-token YOUR_ADMIN_TOKEN
    ```
    You can also create a JSON file with `root`, `port`, `token`, and `admin_token` keys and pass it using `--config config.json`.
    If no token is provided, a random one will be generated and printed to the console.

## Endpoints

All endpoints (except the login page itself) require a valid authentication token, which is set as a secure cookie upon login.

### `GET /`
- **Description:** The main endpoint for browsing files and directories. It displays the contents of the configured root directory.
- **Usage:** Navigate to the root URL to start browsing.

### `GET /<path:path>`
- **Description:** Renders a directory listing or a file view, depending on the path.
- **Usage:**
    - If `<path>` is a directory, it shows the contents of that directory.
    - If `<path>` is a file, it displays the file's content and provides "Download" and "Stream" options.

### `GET /login`
- **Description:** Displays the login page where you can enter your access token.

### `POST /login`
- **Description:** Handles the authentication process.
- **Body:** `token=<your_access_token>`
- **On Success:** Sets a secure cookie and redirects to `/`.
- **On Failure:** Reloads the login page with an error message.

### `GET /admin/login`
- **Description:** Displays the admin login page.

### `POST /admin/login`
- **Description:** Handles admin authentication.
- **Body:** `token=<your_admin_token>`
- **On Success:** Sets a secure admin cookie and redirects to `/admin`.
- **On Failure:** Reloads the admin login page with an error message.

### `GET /admin`
- **Description:** Displays the admin panel with feature flags.

### `POST /admin`
- **Description:** Updates the feature flags.
- **Body:** `file_upload=on/off&file_delete=on/off&file_rename=on/off`

### `POST /upload`
- **Description:** Handles file and folder uploads. This endpoint is used by the drag-and-drop interface.
- **Body:** `multipart/form-data` containing the files and the target directory.
- **Usage:** Drag files/folders into the drop zone on the directory listing page. The frontend handles the request automatically.

### `POST /delete`
- **Description:** Deletes a file or directory.
- **Body:** `path=<target_path>`
- **Usage:** Triggered via the "Delete" button next to each item in the directory listing.

### `POST /rename`
- **Description:** Renames a file or directory.
- **Body:** `path=<target_path>&new_name=<new_name>`
- **Usage:** Triggered via the "Rename" button next to each item in the directory listing.

### `WS /stream/<path:path>`
- **Description:** A WebSocket endpoint for real-time file streaming.
- **Authentication:** Requires a valid session cookie.
- **Usage:** When a user clicks the "Stream" button, a WebSocket connection is established to this endpoint. The server will first send the last 100 lines of the file and then continue to send new lines as they are appended.

## Security TODO

This section tracks known security vulnerabilities that should be addressed.

- [ ] **Path Traversal on Upload:** The single-file upload handler in `UploadHandler` does not properly sanitize the `filename`. An attacker could use a filename like `../../malicious.txt` to write files to arbitrary locations on the server. The `os.path.join` and `os.path.abspath` combination needs to be carefully validated to ensure the final path is within the intended directory.
- [ ] **Denial of Service (DoS) via Large Files:** The application reads entire files into memory for both viewing and uploading without any size limits. An attacker could upload or request to view a very large file, exhausting server memory and causing a crash. Implement file size limits for uploads and consider streaming or paginating large files for viewing.
- [ ] **Missing CSRF Protection:** The application does not use Cross-Site Request Forgery (CSRF) protection. This makes `POST` endpoints, like file uploads, vulnerable. An attacker could trick a logged-in user into visiting a malicious site that forges a request to upload a file to the server without the user's consent. Tornado's built-in `xsrf_cookies=True` setting should be enabled.
- [ ] **Access Token Exposed in Logs:** The master access token is printed to the console on startup, which is a security risk if logs are not properly secured. The token should not be logged.
- [ ] **Disabled WebSocket Origin Check:** The `FileStreamHandler` allows WebSocket connections from any origin (`check_origin` always returns `True`). This should be restricted to only allow connections from the application's own domain to prevent cross-site WebSocket hijacking attacks.

## Future Enhancements

### UI/UX Improvements
- [ ] **Modern UI:** Enhance the interface with a lightweight CSS framework like Bootstrap or Tailwind CSS.
- [ ] **File Previews:** Add support for in-browser previews for common file types (images, PDFs, Markdown).
- [ ] **Search and Sort:** Add a search bar and make the directory listing sortable by name, size, or modification date.

### Core Functionality
- [ ] **Multi-File Operations:** Allow users to select multiple files/folders for batch actions (e.g., delete, download as zip).
- [ ] **In-Browser File Editor:** Embed a code editor like CodeMirror or Monaco to allow for in-browser text file editing.
- [ ] **User Management:** Expand to a full user management system with different roles and permissions.
- [ ] **Advanced Permissions:** Implement a role-based permission system (e.g., read-only users, upload-only users).
- [ ] **Create Files and Folders:** Add UI elements to create new empty files and folders.

### Performance
- [ ] **Asynchronous File I/O:** Use `aiofiles` to perform file operations asynchronously.
- [ ] **Pagination for Large Directories:** Implement pagination for directories with a large number of files.
