---
layout: page
title: Getting Started
permalink: /docs/getting-started/
nav_order: 2
---

# Getting Started

This guide will help you install and configure the Local Mail package in your Laravel application.

## Prerequisites

- Laravel 8.0 or higher
- PHP 8.0 or higher
- Database that supports JSON columns (MySQL 5.7+, PostgreSQL, SQLite)

## Installation

### Step 1: Install the Package

Install the package via Composer:

```bash
composer require indianic/local-mail
```

### Step 2: Publish and Run Migrations

Publish the migration files and run them to create the necessary database table:

```bash
php artisan vendor:publish --tag="local-mail-migrations"
php artisan migrate
```

This will create the `email_messages` table to store captured emails.

### Step 3: Configure Email Transport

To use the Local Mail transport, update your `.env` file:

```env
MAIL_MAILER=localmail
```

Alternatively, you can configure it in your `config/mail.php` file:

```php
'mailers' => [
    // ... other mailers
    'localmail' => [
        'transport' => 'localmail',
    ],
],
```

### Step 4: Publish Configuration (Optional)

If you want to customize the package configuration, publish the config file:

```bash
php artisan vendor:publish --tag="local-mail-config"
```

## Basic Configuration

After publishing the configuration, you can customize the following options in `config/local-mail.php`:

```php
return [
    // Name of the database table to store emails
    'table_name' => env('LOCAL_MAIL_TABLE_NAME', 'email_messages'),

    // Middleware to apply to the email viewing routes
    'route_middlewares' => [
        'web',
    ],

    // Route prefix for the web interface
    'route_prefix' => env('LOCAL_MAIL_ROUTE_PREFIX', 'emails'),
    
    // Truncation settings for automatic cleanup
    'truncate' => [
        'enabled' => env('LOCAL_MAIL_TRUNCATE_ENABLED', false),
        'max_records' => env('LOCAL_MAIL_MAX_RECORDS', 1000),
        'cleanup_days' => env('LOCAL_MAIL_CLEANUP_DAYS', 30),
        'on_new_email' => env('LOCAL_MAIL_TRUNCATE_ON_NEW_EMAIL', false),
    ],
];
```

## Environment Variables

You can also configure the package using environment variables in your `.env` file:

```env
# Table name for storing emails
LOCAL_MAIL_TABLE_NAME=email_messages

# Route prefix for the web interface
LOCAL_MAIL_ROUTE_PREFIX=emails

# Enable automatic truncation
LOCAL_MAIL_TRUNCATE_ENABLED=true

# Maximum number of emails to keep
LOCAL_MAIL_MAX_RECORDS=1000

# Days after which emails are considered old
LOCAL_MAIL_CLEANUP_DAYS=30

# Whether to cleanup on each new email
LOCAL_MAIL_TRUNCATE_ON_NEW_EMAIL=false
```

## Testing the Installation

After installation, you can test that the package is working by sending a simple email:

```php
use Illuminate\Support\Facades\Mail;

// Send a test email
Mail::raw('This is a test email', function ($message) {
    $message->to('test@example.com')
            ->subject('Test Email');
});
```

The email will be captured and stored in your database instead of being sent. You can view it through the web interface.

## Next Steps

- Learn how to [use the web interface](/docs/usage/)
- Explore [advanced configuration options](/docs/configuration/)
- Understand [cleanup and maintenance](/docs/maintenance/)