---
layout: page
title: API Documentation
permalink: /docs/api/
nav_order: 10
---

# API Documentation

This section provides detailed information about the Local Mail package's API and available classes.

## Main Classes

### LocalMailServiceProvider

The service provider registers the Local Mail transport and handles package bootstrapping.

#### Methods

- `configurePackage(Package $package)`: Configures the package with its assets, migrations, and commands
- `packageBooted()`: Registers the 'localmail' transport with Laravel's mail manager

### LocalMailTransport

The core transport class that captures emails instead of sending them.

#### Methods

- `send(RawMessage $message, ?Envelope $envelope = null)`: Captures email data and stores it in the database
- `__toString()`: Returns 'localMail' as the transport name

#### Email Processing

The transport processes different types of email messages:

1. **Email objects**: Extracts subject, body, recipients, and attachments directly
2. **Message objects**: Extracts data from headers
3. **Raw messages**: Parses message string to extract information

### EmailMessage Model

The Eloquent model that represents captured emails in the database.

#### Properties

- `message_id`: Unique identifier for the email
- `subject`: Email subject line
- `body`: Email content
- `to`: Array of recipient addresses (JSON)
- `from`: Array of sender addresses (JSON)
- `cc`: Array of CC addresses (JSON)
- `bcc`: Array of BCC addresses (JSON)
- `attachments`: Array of attachment information (JSON)
- `first_email`: First recipient email address
- `headers`: Email headers (JSON)
- `created_at`: Timestamp when email was captured
- `updated_at`: Timestamp when record was last updated

#### Methods

- `getTable()`: Returns the configured table name
- `truncateIfNecessary()`: Truncates emails if they exceed the configured limit
- `cleanupOldEmails()`: Removes emails older than the configured days
- `truncateAll()`: Removes all emails from the table
- `getFormattedFromAttribute()`: Returns formatted from address
- `getFormattedToAttribute()`: Returns formatted to addresses
- `getFormattedCcAttribute()`: Returns formatted CC addresses
- `getFormattedBccAttribute()`: Returns formatted BCC addresses

## Configuration Options

### Configuration File

The configuration file is located at `config/local-mail.php`:

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

### Environment Variables

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

## Available Commands

### LocalMailCleanUpCommand

Command for cleaning up captured emails.

#### Signature

```
local-mail:cleanup {--type=all : Type of cleanup (all, old, limit)} {--force : Force deletion without confirmation}
```

#### Options

- `--type`: Specifies the cleanup type
  - `all`: Delete all emails
  - `old`: Delete emails older than configured days
  - `limit`: Delete excess emails to maintain the configured limit
- `--force`: Skip confirmation prompts

## Facades

### LocalMail Facade

Provides access to package functionality (though not extensively used in current implementation).

## Database Schema

The package creates the following database table:

```php
Schema::create(config('local-mail.table_name') ?? 'email_messages', function (Blueprint $table) {
    $table->id();
    $table->string('message_id')->unique()->nullable();
    $table->string('subject')->nullable();
    $table->text('body')->nullable();
    $table->json('to')->nullable();
    $table->json('from')->nullable();
    $table->json('cc')->nullable();
    $table->json('bcc')->nullable();
    $table->json('attachments')->nullable();
    $table->string('first_email')->nullable();
    $table->json('headers')->nullable();
    $table->timestamps();
});
```

## Controllers

### EmailMessageController

Handles web interface requests for viewing emails.

#### Methods

- `index()`: Shows the main email listing page
- `showByEmail(Request $request)`: Shows emails for a specific recipient
- `downloadAttachment($path)`: Handles attachment downloads
- `show($id)`: Shows details for a specific email

## Views

The package provides the following views:

- `index.blade.php`: Main email listing page
- `show.blade.php`: Individual email detail page
- `main.blade.php`: Alternative email listing page
- Component views for:
  - `content.blade.php`
  - `header.blade.php`
  - `sidebar.blade.php`

## Events

The package includes an event system for handling new emails:

```php
event(new NewEmail($firstEmail));
```

Though currently commented out, this provides a hook for custom event handling.

## Advanced Usage

### Custom Email Processing

You can extend the EmailMessage model to add custom processing:

```php
use Indianic\LocalMail\Models\EmailMessage;

class CustomEmailMessage extends EmailMessage
{
    protected static function booted()
    {
        static::created(function ($emailMessage) {
            // Custom processing when email is captured
            // For example, send notification, log to external service, etc.
        });
    }
}
```

### Programmatic Email Access

Access captured emails programmatically:

```php
use Indianic\LocalMail\Models\EmailMessage;

// Get all emails
$emails = EmailMessage::all();

// Get emails with specific criteria
$emails = EmailMessage::where('subject', 'like', '%Welcome%')
                     ->whereJsonContains('to', 'user@example.com')
                     ->orderBy('created_at', 'desc')
                     ->get();

// Get paginated results
$emails = EmailMessage::orderBy('created_at', 'desc')->paginate(50);
```

## Next Steps

- Review [security best practices](../security/)
- Explore [troubleshooting tips](../troubleshooting/)
- Learn about [advanced usage patterns](../advanced/)