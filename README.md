# Local Mail Documentation

This directory contains the documentation for the Local Mail Laravel package, designed to be hosted on GitHub Pages.

## Overview

Local Mail is a Laravel package that provides a local email testing solution for development environments. Instead of sending emails to actual recipients, this package captures all outgoing emails and stores them in your local database, allowing you to inspect, debug, and test email functionality without affecting real users.

## Features

- **Email Capture**: Intercepts all outgoing emails in your Laravel application
- **Database Storage**: Stores email content, recipients, attachments, and metadata
- **Web Interface**: Provides a clean web interface to view captured emails
- **Attachment Support**: Handles and stores email attachments
- **Flexible Configuration**: Configurable cleanup and storage limits
- **Command Line Tools**: Artisan commands for maintenance and cleanup
- **JSON Data Storage**: Efficient storage of email data using JSON columns

## Documentation Sections

- [Getting Started](docs/getting-started/) - Installation and basic setup
- [Usage Guide](docs/usage/) - How to use the package in your Laravel application
- [Configuration](docs/configuration/) - Detailed configuration options
- [Commands](docs/commands/) - Available Artisan commands
- [Maintenance](docs/maintenance/) - Cleanup and maintenance tasks
- [Security](docs/security/) - Security considerations and best practices
- [Troubleshooting](docs/troubleshooting/) - Common issues and solutions
- [API Documentation](docs/api/) - Technical API documentation
- [FAQ](docs/faq/) - Frequently asked questions

## GitHub Pages Deployment

This documentation is designed to be deployed to GitHub Pages. To deploy:

1. Ensure your repository has GitHub Pages enabled in the settings
2. Set the source to deploy from the `/docs` folder on the `main` branch
3. The documentation will be available at `https://[username].github.io/[repository]`

## Contributing

Contributions to the documentation are welcome! Please feel free to submit a Pull Request with improvements or corrections.

## Support

For support with the Local Mail package, please check the main repository or create an issue if you encounter problems.