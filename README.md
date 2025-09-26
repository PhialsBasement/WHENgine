# WHENgine

An HTTP server written in WHEN that serves static files and handles multiple connections in parallel.

## What it does

WHENgine listens on localhost:3000 and serves files from an `htdocs` directory. It handles MIME types, generates directory listings when you browse folders, and processes multiple client connections simultaneously using WHEN's parallel execution model.

The server includes a stats page at `/stats` and an admin interface at `/admin` for shutting down the server.

## Installation

### Requirements

- WHEN programming language runtime (version 0.5.0 or higher)
```pip install when-lang```

### Setup

1. Clone this repository
2. Run the server:
```bash
when whengine.when
```

The server automatically creates an `htdocs` directory with a sample index.html file if one doesn't exist.

## Usage

Once running, the server serves content from:
- `http://localhost:3000/` - Files from htdocs directory
- `http://localhost:3000/stats` - Server statistics
- `http://localhost:3000/admin` - Admin panel (password: admin123)

### Adding content

Create files and folders in the `htdocs` directory:

```
htdocs/
├── index.html          → http://localhost:3000/
├── about.html          → http://localhost:3000/about.html
├── css/
│   └── style.css       → http://localhost:3000/css/style.css
└── projects/
    ├── index.html      → http://localhost:3000/projects/
    └── demo.html       → http://localhost:3000/projects/demo.html
```

Directories without index files show automatic file listings.

## How it works

### WHEN language features

The server uses WHEN's conditional execution model where `when` statements control program flow instead of traditional if-else logic.

File operations happen in `os` blocks that handle system-level tasks and recursive function calls. For example, finding file extensions works by walking backward through filenames character by character:

```when
def get_file_extension(filename):
    dot_index = -1
    char_index = len(filename) - 1

    when char_index >= 0:
        when filename[char_index] == '.':
            dot_index = char_index
        when filename[char_index] != '.' and char_index > 0:
            char_index = char_index - 1
            get_file_extension_recursive()
```

### Parallel processing

Two execution streams run concurrently:
- `accept_connections.start()` - Accepts new client connections
- `request_handler.start()` - Processes queued requests

This uses WHEN's `fo` blocks for parallel execution without traditional threading.

### Request handling

1. HTTP requests are parsed to extract method, path, and headers
2. Paths are checked against built-in routes (`/stats`, `/admin`)
3. If not found, the path maps to files in htdocs
4. Files are served with appropriate MIME types
5. Directories show listings or serve index files

### File serving

The server detects MIME types based on file extensions and serves both text and binary files correctly. It supports common web file types like HTML, CSS, JavaScript, images, and documents.

## Server configuration

Key settings are defined as global variables:

- Host: localhost
- Port: 3000
- Document root: htdocs/
- Admin password: admin123
- Default index files: index.html, index.htm

## Error handling

The server returns appropriate HTTP status codes:
- 400 for malformed requests
- 403 for invalid admin credentials
- 404 for missing files with styled error pages

## Development status

WHENgine is in development and demonstrates WHEN's capabilities for network programming and parallel processing.
