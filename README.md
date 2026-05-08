# Software Theory - Blog

This is the source code for my personal blog, built with [Jekyll](https://jekyllrb.com/) and hosted on GitHub Pages.

## Local Development

To run this blog locally and test your changes, follow these steps:

### Prerequisites

You need to have **Ruby** and **Bundler** installed on your machine.

- **Ruby**: [Installation Guide](https://www.ruby-lang.org/en/documentation/installation/)
- **Bundler**: Run `gem install bundler`

### Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/carlosblanco/carlosblanco.github.io.git
   cd carlosblanco.github.io
   ```

2. **Install dependencies:**
   ```bash
   bundle install
   ```

### Running Locally

To start the Jekyll server and preview the site:

```bash
bundle exec jekyll serve
```

Once the server is running, you can view the blog at:
[http://localhost:4000](http://localhost:4000)

The server will automatically watch for changes in your files and rebuild the site.

## Project Structure

- `_posts/`: Contains the blog posts in Markdown format.
- `assets/main.scss`: Main stylesheet containing custom styles and overrides.
- `_includes/`: HTML fragments like the header and footer.
- `_layouts/`: Page templates (home, post, article).
- `external-publications.md`: The page listing articles published on other platforms.
