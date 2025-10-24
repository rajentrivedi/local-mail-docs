---
layout: home
title: Home
nav_order: 1
---

# Local Mail - Laravel Email Testing Package

<div class="text-large margin-top-2">
  <p>Local Mail is a Laravel package that provides a local email testing solution for development environments.</p>
 <p>Instead of sending emails to actual recipients, this package captures all outgoing emails and stores them in your local database, allowing you to inspect, debug, and test email functionality without affecting real users.</p>
</div>

<div class="features-grid">
  <div class="feature-card">
    <h3><a href="{{ '/docs/getting-started/' | relative_url }}">Getting Started</a></h3>
    <p>Install and configure the Local Mail package in your Laravel application.</p>
  </div>
  <div class="feature-card">
    <h3><a href="{{ '/docs/usage/' | relative_url }}">Usage Guide</a></h3>
    <p>Learn how to use the package and view captured emails.</p>
  </div>
</div>

## Features

- **Email Capture**: Intercepts all outgoing emails in your Laravel application
- **Database Storage**: Stores email content, recipients, attachments, and metadata
- **Web Interface**: Provides a clean web interface to view captured emails
- **Attachment Support**: Handles and stores email attachments
- **Flexible Configuration**: Configurable cleanup and storage limits
- **Command Line Tools**: Artisan commands for maintenance and cleanup
- **JSON Data Storage**: Efficient storage of email data using JSON columns

## Quick Installation

```bash
composer require indianic/local-mail

php artisan vendor:publish --tag="local-mail-migrations"
php artisan migrate

# Set in your .env file
MAIL_MAILER=localmail
```

## Why Use Local Mail?

Testing email functionality in development can be challenging. Instead of sending emails to real addresses or relying on temporary email services, Local Mail provides:

- **Privacy**: No risk of sending test emails to real users
- **Convenience**: All emails captured locally for easy inspection
- **Efficiency**: No need to create temporary email accounts
- **Integration**: Works seamlessly with Laravel's mail system
- **Control**: Complete control over email storage and cleanup