---
layout: page
title: Troubleshooting
permalink: /docs/troubleshooting/
nav_order: 8
---

# Troubleshooting

This section addresses common issues you might encounter when using the Local Mail package and provides solutions.

## Common Issues

### Emails Not Being Captured

**Problem**: Emails are being sent normally instead of being captured.

**Solutions**:
1. Verify that `MAIL_MAILER=localmail` is set in your `.env` file
2. Clear configuration cache: `php artisan config:clear`
3. Check that the Local Mail service provider is properly registered
4. Ensure the package is installed: `composer show indianic/local-mail`
5. Verify that the configuration file exists and is properly set up

**Debugging**:
```bash
# Check current mail driver
php artisan tinker
>>> echo config('mail.default');
>>> echo config('mail.mailers.localmail.transport');
```

### Web Interface Not Accessible

**Problem**: The email viewing interface is not accessible at the expected URL.

**Solutions**:
1. Check that the route prefix in `config/local-mail.php` matches the URL you're accessing
2. Verify that the 'web' middleware is available and working
3. Clear route cache: `php artisan route:clear`
4. Run `php artisan route:list | grep emails` to see the registered routes
5. Ensure the service provider is properly loaded

**Check routes**:
```bash
php artisan route:list | grep local-mail
```

### Database Issues

**Problem**: Migration fails or database operations don't work.

**Solutions**:
1. Verify your database connection: `php artisan tinker` and test a simple query
2. Check that your database supports JSON columns (MySQL 5.7+, PostgreSQL, SQLite)
3. Run migrations: `php artisan migrate`
4. Check if the table name in config matches what was created
5. Verify database user permissions

**Check table structure**:
```bash
php artisan tinker
>>> Schema::hasTable(config('local-mail.table_name', 'email_messages'));
>>> DB::select('DESCRIBE '.config('local-mail.table_name', 'email_messages'));
```

### Attachment Issues

**Problem**: Attachments are not being saved or downloaded properly.

**Solutions**:
1. Verify that the `public/local-emails` directory exists and is writable:
   ```bash
   mkdir -p public/local-emails
   chmod 755 public/local-emails
   ```
2. Check that the web server can serve files from this directory
3. Verify that original attachment files exist at their source paths
4. Check file permissions and disk space
5. Ensure the File facade is properly imported in the transport class

**Debug attachment storage**:
```php
// Check if directory is writable
if (!File::isDirectory(public_path('local-emails'))) {
    File::makeDirectory(public_path('local-emails'), 0755, true);
}

// Verify file permissions
echo substr(sprintf('%o', fileperms(public_path('local-emails'))), -4);
```

## Error Messages and Solutions

### "Class 'Indianic\LocalMail\LocalMailServiceProvider' not found"

**Cause**: Package not properly installed or autoloader not refreshed.

**Solution**:
```bash
composer dump-autoload
composer install
```

### "Table 'email_messages' doesn't exist"

**Cause**: Migrations not run or table name mismatch.

**Solution**:
```bash
php artisan vendor:publish --tag="local-mail-migrations"
php artisan migrate
```

### "Call to undefined method LocalMailTransport::send()"

**Cause**: Version compatibility issue or incorrect transport registration.

**Solution**: Check that you're using the correct version of the package and Laravel.

### "Maximum execution time exceeded" when viewing emails

**Cause**: Too many emails in the database causing slow queries.

**Solution**:
1. Implement cleanup: `php artisan local-mail:cleanup --type=limit --force`
2. Add database indexes
3. Use pagination when accessing emails programmatically

## Debugging Steps

### Enable Debug Mode

Add debugging to your application:

```php
// In your AppServiceProvider
public function boot()
{
    if (app()->environment('local')) {
        Event::listen(function (\Illuminate\Mail\Events\MessageSent $event) {
            \Log::info('Email sent', [
                'mailer' => $event->sent->getTransport()::class,
                'to' => $event->message->getTo(),
                'subject' => $event->message->getSubject(),
            ]);
        });
    }
}
```

### Check Transport Registration

Verify the transport is properly registered:

```bash
php artisan tinker
>>> $manager = app('mail.manager');
>>> $manager->get('localmail');
```

### Database Query Debugging

Enable query logging to see what's happening:

```php
// In a controller or test
DB::enableQueryLog();
// Perform email operations
Mail::to('test@example.com')->send(new TestMailable());
// Check queries
dd(DB::getQueryLog());
```

## Performance Issues

### Slow Interface

If the email viewing interface is slow:

1. **Clean up old emails**: `php artisan local-mail:cleanup --type=old --force`
2. **Add database indexes** as mentioned in the maintenance section
3. **Use pagination** when viewing large numbers of emails
4. **Limit the number of emails stored** with automatic cleanup

### High Memory Usage

If you encounter memory issues:

```bash
# Increase PHP memory limit temporarily for cleanup
php -d memory_limit=512M artisan local-mail:cleanup --type=all --force
```

Or in your code:
```php
ini_set('memory_limit', '512M');
// Process emails in smaller batches
$emails = EmailMessage::select('id', 'subject', 'first_email')->limit(100)->get();
```

## Environment-Specific Issues

### Docker/Container Issues

If using Docker:

1. Ensure the `public/local-emails` directory is properly mapped
2. Check file permissions inside containers
3. Verify that the web server in the container can write to the directory

### Different Environment Behavior

If the package works in one environment but not another:

1. Compare `.env` files between environments
2. Check that the same version of the package is installed
3. Verify that migrations are run in all environments
4. Compare PHP and Laravel versions

## Verification Steps

### Complete Verification Process

Run through these steps to verify everything is working:

```bash
# 1. Check package installation
composer show indianic/local-mail

# 2. Verify configuration
php artisan config:cache

# 3. Check migrations
php artisan migrate:status

# 4. Test email sending
php artisan tinker
>>> Mail::raw('Test', function($m) { $m->to('test@example.com')->subject('Test'); });
>>> \Indianic\LocalMail\Models\EmailMessage::count();

# 5. Check routes
php artisan route:list | grep emails
```

### Automated Verification Script

Create a verification script:

```php
<?php
// verify-local-mail.php
require_once 'vendor/autoload.php';

$app = require_once 'bootstrap/app.php';
$kernel = $app->make(Illuminate\Contracts\Console\Kernel::class);
$kernel->bootstrap();

use Indianic\LocalMail\Models\EmailMessage;
use Illuminate\Support\Facades\Mail;

// Test email capture
Mail::raw('Verification email', function($message) {
    $message->to('verify@example.com')->subject('Verification');
});

// Check if email was captured
$count = EmailMessage::count();
echo "Emails in database: $count\n";

if ($count > 0) {
    echo "✓ Local Mail is working correctly\n";
} else {
    echo "✗ Local Mail is not capturing emails\n";
}
```

## Support Information

When seeking help with issues:

1. Include your Laravel version: `php artisan --version`
2. Include your Local Mail version: `composer show indianic/local-mail`
3. Include your PHP version: `php -v`
4. Provide relevant error messages
5. Show your configuration files (with sensitive data removed)
6. Include steps to reproduce the issue

## Next Steps

- Learn about [advanced usage patterns](/docs/advanced/)
- Explore [contributing to the package](/docs/contributing/)
- Review the [FAQ section](/docs/faq/)