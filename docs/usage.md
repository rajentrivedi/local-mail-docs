---
layout: page
title: Usage Guide
permalink: /docs/usage/
nav_order: 3
---

# Usage Guide

This guide explains how to use the Local Mail package in your Laravel application.

## Basic Usage

Once configured, all emails sent through Laravel's mail system will be captured automatically. No additional code changes are required in your application.

```php
use Illuminate\Support\Facades\Mail;
use App\Mail\WelcomeEmail;

// Your existing email code continues to work unchanged
Mail::to('user@example.com')->send(new WelcomeEmail($user));

// The email will be captured and stored in the database instead of being sent
```

## Web Interface

The package provides a user-friendly web interface to view captured emails.

### Accessing the Interface

By default, you can access the email viewing interface at:

```
http://your-app.test/emails
```

The route prefix can be customized in the configuration file.

### Main Dashboard

The main dashboard shows:

- List of all captured emails
- Subject lines
- Recipients
- Timestamps
- Quick access to individual emails

### Email Details Page

When viewing an individual email, you'll see:

- Full email content (HTML/text)
- Recipients (To, CC, BCC)
- Subject line
- Attachments with download links
- Metadata and timestamps

## Programmatic Access

You can also access captured emails programmatically through the `EmailMessage` model:

```php
use Indianic\LocalMail\Models\EmailMessage;

// Get all captured emails
$emails = EmailMessage::all();

// Get emails for a specific recipient
$emails = EmailMessage::whereJsonContains('to', 'test@example.com')->get();

// Get emails with specific subject
$emails = EmailMessage::where('subject', 'like', '%Welcome%')->get();

// Get emails sorted by date
$emails = EmailMessage::orderBy('created_at', 'desc')->get();
```

## Email Model Properties

The `EmailMessage` model has the following properties:

- `message_id`: Unique identifier for the email
- `subject`: Email subject line
- `body`: Email content (HTML or text)
- `to`: Array of recipient addresses (JSON)
- `from`: Array of sender addresses (JSON)
- `cc`: Array of CC addresses (JSON)
- `bcc`: Array of BCC addresses (JSON)
- `attachments`: Array of attachment information (JSON)
- `first_email`: First recipient email address
- `created_at`: Timestamp when email was captured

### Formatted Properties

The model also provides formatted accessors:

```php
$email = EmailMessage::first();

// Get formatted addresses
$from = $email->formatted_from;  // Returns string
$to = $email->formatted_to;      // Returns array
$cc = $email->formatted_cc;      // Returns array
$bcc = $email->formatted_bcc;    // Returns array
```

## Attachments

The package handles email attachments by:

1. Saving them to the `public/local-emails` directory
2. Storing metadata in the database
3. Providing download links in the web interface

### Attachment Information

Each attachment contains:

- `filename`: Name of the file
- `file_path`: URL to download the file
- `original_file_path`: Original path of the file
- `content_type`: MIME type of the file
- `disposition`: Attachment disposition (inline or attachment)

## Advanced Usage

### Filtering Emails

You can filter emails using various criteria:

```php
// Emails sent to specific domain
$emails = EmailMessage::whereJsonContains('to', 'gmail.com')->get();

// Emails with attachments
$emails = EmailMessage::whereNotNull('attachments')
                     ->where('attachments', '!=', '[]')
                     ->get();

// Emails within a date range
$emails = EmailMessage::whereBetween('created_at', [
    now()->subDays(7), 
    now()
])->get();
```

### Searching Emails

For more complex searches, you can combine multiple conditions:

```php
// Find emails with specific subject and recipient
$emails = EmailMessage::where('subject', 'like', '%Welcome%')
                     ->whereJsonContains('to', 'user@example.com')
                     ->orderBy('created_at', 'desc')
                     ->get();
```

## Troubleshooting

### Emails Not Appearing

If emails are not appearing in the interface:

1. Verify that `MAIL_MAILER=localmail` is set in your `.env` file
2. Check that the database migrations have been run
3. Confirm that the email sending code is actually being executed

### Attachments Not Working

If attachments aren't being saved:

1. Verify that the `public/local-emails` directory exists and is writable
2. Check that your web server can serve files from this directory
3. Ensure the original attachment files exist at their source paths

## Next Steps

- Learn about [configuration options](/docs/configuration/)
- Understand [cleanup and maintenance](/docs/maintenance/)
- Explore [command-line tools](/docs/commands/)