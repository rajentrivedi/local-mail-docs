---
layout: page
title: Security
permalink: /docs/security/
nav_order: 7
---

# Security

Security is a critical aspect when implementing email testing in your development environment. This section covers security considerations and best practices for the Local Mail package.

## Production Safety

### Never Use in Production

**Critical**: The Local Mail package is designed exclusively for local development and testing environments. It should never be used in production.

To ensure the package is not loaded in production:

```php
// In your AppServiceProvider's register method
public function register()
{
    if ($this->app->environment('production')) {
        return;
    }
    
    $this->app->register(\Indianic\LocalMail\LocalMailServiceProvider::class);
}
```

Or use environment checks in your service provider registration:

```php
// In config/app.php or in your AppServiceProvider
if (!app()->environment('production')) {
    $app->register(\Indianic\LocalMail\LocalMailServiceProvider::class);
}
```

## Access Control

### Web Interface Security

By default, the email viewing interface uses the 'web' middleware. For additional security:

```php
// In config/local-mail.php
'route_middlewares' => [
    'web',
    'auth',           // Require authentication
    'role:admin',     // Require specific role
],
```

### Environment-Based Access

Restrict access based on environment:

```php
// In a custom middleware
public function handle($request, Closure $next)
{
    if (!app()->environment('local', 'development', 'testing')) {
        abort(404); // Or redirect, or show error
    }
    
    return $next($request);
}
```

### IP Restrictions

For extra security, you can restrict access by IP:

```php
// In a custom middleware
public function handle($request, Closure $next)
{
    $allowedIps = ['127.0.1', '::1', '192.168.0.0/16']; // Adjust as needed
    
    if (!in_array($request->ip(), $allowedIps)) {
        abort(403);
    }
    
    return $next($request);
}
```

## Data Protection

### Email Content Storage

- Email content is stored in your local database
- No external services are used
- All data remains within your local environment
- Email bodies may contain sensitive information

### Database Security

Ensure your development database has appropriate security:

```php
// Use strong database credentials in .env
DB_CONNECTION=mysql
DB_HOST=127.0.1
DB_PORT=3306
DB_DATABASE=your_dev_database
DB_USERNAME=your_dev_user
DB_PASSWORD=your_strong_password
```

### Sensitive Information

Be aware that captured emails may contain:

- Personal information
- API keys or tokens
- Password reset links
- Verification codes
- Business-sensitive data

## Attachment Security

### File Storage

Attachments are stored in the `public/local-emails` directory:

```bash
# Ensure proper permissions
chmod 755 public/local-emails/
chown www-data:www-data public/local-emails/  # Adjust user as needed
```

### File Validation

The package currently stores all attachments without validation. For enhanced security:

```php
// You could implement additional validation in your application
// before sending emails with attachments
public function sendEmailWithAttachment($request)
{
    $file = $request->file('attachment');
    
    // Validate file type and size
    $this->validate($request, [
        'attachment' => 'file|mimes:pdf,doc,docx|max:2048'  // Example validation
    ]);
    
    // Send email as normal - Local Mail will capture it
}
```

## Configuration Security

### Environment Variables

Store sensitive configuration in environment variables:

```env
# .env file
LOCAL_MAIL_TABLE_NAME=email_messages
LOCAL_MAIL_ROUTE_PREFIX=emails
LOCAL_MAIL_TRUNCATE_ENABLED=true
LOCAL_MAIL_MAX_RECORDS=1000
LOCAL_MAIL_CLEANUP_DAYS=30
LOCAL_MAIL_TRUNCATE_ON_NEW_EMAIL=false

# Don't commit .env to version control
# Add to .gitignore
```

### Configuration Validation

Validate configuration values:

```php
// In your service provider
public function boot()
{
    // Validate configuration
    $maxRecords = config('local-mail.truncate.max_records', 1000);
    if ($maxRecords > 10000) {
        Log::warning('Local Mail: High max_records value detected');
    }
}
```

## Web Interface Protection

### Route Security

Protect the email viewing routes:

```php
// In routes/web.php - add custom protection
Route::middleware(['web', 'local.development'])->group(function () {
    Route::resource('emails', \Indianic\LocalMail\Controllers\EmailMessageController::class);
});
```

### Custom Middleware

Create custom middleware for development-only access:

```php
<?php
// app/Http/Middleware/DevelopmentOnly.php
namespace App\Http\Middleware;

use Closure;

class DevelopmentOnly
{
    public function handle($request, Closure $next)
    {
        if (!app()->environment(['local', 'development', 'testing'])) {
            abort(404);
        }

        return $next($request);
    }
}
```

Register the middleware in `app/Http/Kernel.php`:

```php
protected $middlewareGroups = [
    'web' => [
        // ... other middleware
        \App\Http\Middleware\DevelopmentOnly::class, // Only in development
    ],
];
```

## Data Retention and Cleanup

### Automatic Cleanup

Configure automatic cleanup to limit data retention:

```php
// In config/local-mail.php
'truncate' => [
    'enabled' => true,
    'max_records' => 500,      // Limit stored emails
    'cleanup_days' => 7,       // Delete emails after 7 days
    'on_new_email' => true,    // Cleanup when new emails arrive
],
```

### Scheduled Cleanup

Use Laravel's scheduler for automatic cleanup:

```php
// In app/Console/Kernel.php
protected function schedule(Schedule $schedule)
{
    // Clean up old emails daily
    $schedule->command('local-mail:cleanup --type=old --force')
             ->daily()
             ->environments(['local', 'development']);
}
```

## Monitoring and Logging

### Security Logging

Log access to the email interface:

```php
// In your controller or middleware
public function index()
{
    if (config('app.env') !== 'local') {
        Log::warning('Local Mail access attempt from non-local environment', [
            'ip' => request()->ip(),
            'user' => auth()->id(),
            'timestamp' => now(),
        ]);
    }
    
    return view('local-emails::index');
}
```

### Audit Trail

Keep track of email access:

```php
// You could extend the EmailMessage model with access tracking
class EmailMessage extends Model
{
    // Add access logging when emails are viewed
    protected static function booted()
    {
        static::retrieved(function ($email) {
            if (app()->environment('local')) {
                Log::info('Email accessed', [
                    'email_id' => $email->id,
                    'subject' => $email->subject,
                    'accessed_at' => now(),
                    'ip' => request()->ip() ?? 'unknown',
                ]);
            }
        });
    }
}
```

## Common Security Pitfalls

### Accidental Production Deployment

Always verify the package isn't deployed to production:

```php
// In your deployment scripts or health checks
if (app()->environment('production')) {
    // Verify Local Mail service provider is not registered
    $providers = app()->getLoadedProviders();
    if (isset($providers[\Indianic\LocalMail\LocalMailServiceProvider::class])) {
        throw new \RuntimeException('Local Mail should not be loaded in production!');
    }
}
```

### Data Leakage

Be careful not to accidentally include captured emails in:

- Production database dumps
- Shared development databases
- Public repositories
- Screenshots or screen shares

### Misconfigured Routes

Ensure the email viewing routes are properly secured:

```php
// Verify your route configuration
Route::middleware(config('local-mail.route_middlewares', ['web']))
    ->group(function () {
        // Routes only accessible in development
        if (app()->environment(['local', 'development', 'testing'])) {
            Route::resource('emails', \Indianic\LocalMail\Controllers\EmailMessageController::class);
        }
    });
```

## Security Best Practices Checklist

- [ ] Verify the package is not loaded in production
- [ ] Restrict access to the email viewing interface
- [ ] Implement automatic cleanup of old emails
- [ ] Monitor email capture volume
- [ ] Secure the database containing email data
- [ ] Validate file attachments before sending
- [ ] Use strong passwords for development database
- [ ] Regularly audit access to email interface
- [ ] Don't commit sensitive configuration to version control
- [ ] Implement proper error handling without information disclosure

## Next Steps

- Review [troubleshooting tips](/docs/troubleshooting/)
- Explore [advanced usage patterns](/docs/advanced/)
- Learn about [contributing to the package](/docs/contributing/)