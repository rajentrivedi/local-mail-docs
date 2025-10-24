---
layout: page
title: Configuration
permalink: /docs/configuration/
nav_order: 4
---

# Configuration

This section covers all configuration options available in the Local Mail package.

## Configuration File

The configuration file is located at `config/local-mail.php`. You can publish it using:

```bash
php artisan vendor:publish --tag="local-mail-config"
```

## Configuration Options

### Table Name

```php
'table_name' => env('LOCAL_MAIL_TABLE_NAME', 'email_messages'),
```

Customize the name of the database table used to store captured emails. You can set this via the `LOCAL_MAIL_TABLE_NAME` environment variable.

### Route Configuration

```php
'route_middlewares' => [
    'web',
],

'route_prefix' => env('LOCAL_MAIL_ROUTE_PREFIX', 'emails'),
```

- `route_middlewares`: Middleware(s) applied to the email viewing routes. Defaults to 'web' middleware.
- `route_prefix`: URL prefix for the web interface. Defaults to 'emails', making the interface accessible at `/emails`.

### Truncation Settings

```php
'truncate' => [
    'enabled' => env('LOCAL_MAIL_TRUNCATE_ENABLED', false),
    'max_records' => env('LOCAL_MAIL_MAX_RECORDS', 1000),
    'cleanup_days' => env('LOCAL_MAIL_CLEANUP_DAYS', 30),
    'on_new_email' => env('LOCAL_MAIL_TRUNCATE_ON_NEW_EMAIL', false),
],
```

These settings control automatic cleanup of old emails:

- `enabled`: Enable/disable automatic truncation features
- `max_records`: Maximum number of emails to keep before cleanup
- `cleanup_days`: Number of days after which emails are considered old
- `on_new_email`: Whether to perform cleanup each time a new email is captured

## Environment Variables

You can configure all settings using environment variables in your `.env` file:

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

## Advanced Configuration Examples

### Custom Route Configuration

```php
'route_middlewares' => [
    'web',
    'auth',  // Require authentication
],

'route_prefix' => 'admin/emails',  // Custom route prefix
```

### Aggressive Cleanup Configuration

```php
'truncate' => [
    'enabled' => true,
    'max_records' => 500,           // Keep only 500 emails
    'cleanup_days' => 7,            // Delete emails older than 7 days
    'on_new_email' => true,         // Cleanup on each new email
],
```

### Conservative Configuration

```php
'truncate' => [
    'enabled' => true,
    'max_records' => 5000,          // Keep up to 5000 emails
    'cleanup_days' => 90,           // Delete emails older than 90 days
    'on_new_email' => false,        // Only cleanup via commands
],
```

## Security Considerations

### Production Environment

For security reasons, it's recommended to disable the package in production:

```php
// In your AppServiceProvider
public function register()
{
    if ($this->app->environment('production')) {
        // Don't register the package in production
        return;
    }
    
    $this->app->register(\Indianic\LocalMail\LocalMailServiceProvider::class);
}
```

### Access Control

You can add additional middleware to control access to the email interface:

```php
'route_middlewares' => [
    'web',
    'auth',
    'role:admin',  // Only admin users can access
],
```

## Performance Considerations

### Large Volume Handling

For applications that send many emails:

1. Set appropriate truncation limits
2. Use the `on_new_email` option to prevent table bloat
3. Consider running cleanup commands on a schedule

### Database Indexing

The package creates basic indexes, but for high-volume usage, you might want to add additional indexes:

```php
// In a custom migration
Schema::table(config('local-mail.table_name', 'email_messages'), function (Blueprint $table) {
    $table->index('created_at');
    $table->index(['first_email', 'created_at']);
    $table->index('subject');
});
```

## Next Steps

- Learn about [cleanup and maintenance](../maintenance/)
- Explore [command-line tools](../commands/)
- Understand [security best practices](../security/)