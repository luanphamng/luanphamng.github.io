# Code Circuits

Personal blog built with [Hugo](https://gohugo.io/) and the [Clarity](https://github.com/chipzoller/hugo-clarity) theme.

## Environment Setup

### Prerequisites

- Go 1.21+
- Hugo Extended 0.110+ (recommended: latest version)

### Install Hugo

**Ubuntu/Debian (via Snap):**
```bash
sudo snap install hugo --channel=extended
```

**macOS (via Homebrew):**
```bash
brew install hugo
```

**Windows (via Chocolatey):**
```bash
choco install hugo-extended
```

### Clone and Setup

```bash
git clone <repository-url>
cd <repository-folder>
hugo mod get -u
```

### Run Development Server

```bash
hugo server -D
```

Site will be available at `http://localhost:1313/`

## Writing Posts

### Create a New Post

```bash
hugo new content/post/your-post-title.md
```

### Post Front Matter

Each post uses TOML front matter:

```toml
+++
author = "Your Name"
title = "Post Title"
date = "2024-01-15"
description = "Brief description for SEO."
tags = ["tag1", "tag2"]
thumbnail = "images/post_thump/image.jpg"
draft = true
+++
```

### Front Matter Options

| Field | Description |
|-------|-------------|
| `author` | Post author name |
| `title` | Post title |
| `date` | Publication date (YYYY-MM-DD) |
| `description` | SEO description |
| `tags` | List of tags |
| `thumbnail` | Thumbnail image path |
| `draft` | Set `false` to publish |

### Publishing

1. Set `draft = false` in front matter
2. Build the site:
   ```bash
   hugo
   ```
3. Output will be in the `public/` folder

## Project Structure

```
├── archetypes/     # Post templates
├── config/         # Hugo configuration
├── content/        # Markdown content
│   └── post/       # Blog posts
├── layouts/        # Custom layouts
├── static/         # Static assets (images, etc.)
└── public/         # Generated site (gitignored)
```
