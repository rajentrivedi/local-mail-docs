---
layout: page
title: FAQ
permalink: /docs/faq/
nav_order: 9
---

# Frequently Asked Questions

This section answers common questions about the Local Mail package.

## General Questions

### Q: Can I use this in production?
**A**: No, this package is designed specifically for local development and testing. It should not be used in production environments as it captures all outgoing emails instead of sending them, which would result in no emails being delivered to actual recipients.

### Q: How are attachments handled?
**A**: Attachments are saved to the `public/local-emails` directory and their metadata is stored in the database. The web interface provides download links for each attachment.

### Q: Can I customize the database table name?
**A**: Yes, you can customize the table name using the `LOCAL_MAIL_TABLE_NAME` environment variable or the `table_name` configuration option in `config/local-mail.php`.

### Q: Is there a limit to how many emails can be stored?
**A**: By default, there's no hard limit, but you can configure automatic cleanup to maintain a maximum number of records using the truncation settings.

### Q: What happens to the original emails?
**A**: Original emails are not sent when using the localmail transport. They are captured and stored in the database instead of being delivered to recipients.

## Installation and Configuration

### Q: I'm getting "Class not found" errors after installation.
**A**: Run `composer dump-autoload` to refresh the autoloader. This is commonly needed after installing new packages.

### Q: The email viewing interface is not accessible.
**A**: Verify that:
1. You've published the migrations and run them
2. The route prefix matches what you're accessing
3. The 'web' middleware is available
4. You've cleared route cache with `php artisan route:clear`

### Q: How do I change the URL for the email viewing interface?
**A**: Set the `route_prefix` option in `config/local-mail.php` or use the `LOCAL_MAIL_ROUTE_PREFIX` environment variable.

## Usage Questions

### Q: Can I search through captured emails?
**A**: Yes, the web interface provides search functionality, and you can also query the EmailMessage model directly using Laravel's query builder.

### Q: How do I access emails programmatically?
**A**: Use the `Indianic\LocalMail\Models\EmailMessage` model to query captured emails directly:

```php
use Indianic\LocalMail\Models\EmailMessage;

$emails = EmailMessage::whereJsonContains('to', 'test@example.com')->get();
```

### Q: Can I export emails?
**A**: The package doesn't include built-in export functionality, but you can access the EmailMessage model to build custom export features.

### Q: Do emails get deleted automatically?
**A**: By default, no. However, you can configure automatic cleanup through the truncation settings in the configuration file.

## Performance and Maintenance

### Q: The database is getting too large.
**A**: Configure automatic cleanup in your configuration:

```php
'truncate' => [
    'enabled' => true,
    'max_records' => 100,  // Keep maximum 1000 emails
    'cleanup_days' => 30,   // Delete emails older than 30 days
],
```

Or use the cleanup command:
```bash
php artisan local-mail:cleanup --type=limit --force
```

### Q: How can I schedule automatic cleanup?
**A**: Add to your `app/Console/Kernel.php`:

```php
protected function schedule(Schedule $schedule)
{
    $schedule->command('local-mail:cleanup --type=old --force')->daily();
}
```

## Security Questions

### Q: Is it safe to use this package?
**A**: Yes, when used appropriately in development environments. However, never install or enable this package in production as it will prevent emails from being sent.

### Q: How can I restrict access to the email viewing interface?
**A**: Add additional middleware in the configuration:

```php
'route_middlewares' => [
    'web',
    'auth',        // Require authentication
    'role:admin',  // Require admin role
],
```

### Q: Are sensitive data safe in the captured emails?
**A**: Email content is stored locally in your development database. Make sure to:
1. Not commit database dumps containing email data
2. Use appropriate cleanup settings to limit data retention
3. Secure your development database with strong credentials

## Technical Questions

### Q: What database systems are supported?
**A**: The package requires a database that supports JSON columns, such as:
- MySQL 5.7 or higher
- PostgreSQL
- SQLite 3.38.0 or higher

### Q: Can I use this with Laravel Octane?
**A**: Yes, the package should work with Laravel Octane, but make sure to test thoroughly as Octane's process isolation might affect file-based operations like attachment storage.

### Q: How does the localmail transport work?
**A**: The package registers a custom mail transport that intercepts all outgoing emails and stores them in the database instead of sending them through traditional mail transports.

### Q: Can I customize the email storage format?
**A**: The package uses a predefined schema for storing email data. You can extend the EmailMessage model if you need additional fields, but the core storage format is fixed.

## Troubleshooting

### Q: Emails are still being sent instead of captured.
**A**: Verify that:
1. `MAIL_MAILER=localmail` is set in your `.env` file
2. The configuration is loaded: run `php artisan config:clear`
3. The service provider is registered properly

### Q: Attachments are not showing up.
**A**: Check that:
1. The `public/local-emails` directory exists and is writable
2. Your web server can serve files from this directory
3. The original attachment files exist at their source paths

### Q: I'm getting memory errors with many emails.
**A**: Use pagination when querying emails:

```php
$emails = EmailMessage::orderBy('created_at', 'desc')->paginate(50);
```

## Advanced Questions

### Q: Can I customize the web interface?
**A**: Yes, you can publish the views and customize them:

```bash
php artisan vendor:publish --tag="local-mail-views"
```

### Q: How can I integrate with my existing test suite?
**A**: You can query the EmailMessage model in your tests to verify emails were sent:

```php
public function testEmailIsSent()
{
    // Perform action that should send email
    $user = User::factory()->create();
    $user->notify(new WelcomeNotification());
    
    // Verify email was captured
    $this->assertDatabaseHas('email_messages', [
        'first_email' => $user->email,
        'subject' => 'Welcome!'
    ]);
}
```

### Q: Can I use this for API testing?
**A**: Yes, you can query the EmailMessage model directly in your API tests to verify email sending behavior without relying on external email services.

## Support Questions

### Q: Where can I get support?
**A**: Check the GitHub repository for the package. You can create an issue if you encounter problems. Make sure to include:
- Laravel version
- Package version
- Steps to reproduce the issue
- Error messages or logs

### Q: How can I contribute?
**A**: Contributions are welcome! Please see the CONTRIBUTING.md file in the repository for details on how to contribute to this package.

## Next Steps

- Review [all documentation](/)
- Try the [getting started guide](/docs/getting-started/)
- Explore [advanced usage](/docs/advanced/)