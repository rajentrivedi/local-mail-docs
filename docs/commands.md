---
layout: page
title: Commands
permalink: /docs/commands/
nav_order: 5
---

# Commands

The Local Mail package includes several Artisan commands for maintenance and management of captured emails.

## Cleanup Command

The primary command for managing captured emails is `local-mail:cleanup`. This command provides different cleanup strategies.

### Command Syntax

```bash
php artisan local-mail:cleanup {--type= : Type of cleanup (all, old, limit)} {--force : Force deletion without confirmation}
```

### Cleanup Types

#### All Emails

Remove all captured emails:

```bash
php artisan local-mail:cleanup --type=all --force
```

**Without `--force` flag, you'll be prompted for confirmation.**

#### Old Emails

Remove emails older than the configured number of days:

```bash
php artisan local-mail:cleanup --type=old --force
```

This uses the `LOCAL_MAIL_CLEANUP_DAYS` configuration value (default: 30 days).

#### Limit-Based Cleanup

Remove excess emails to maintain the configured maximum:

```bash
php artisan local-mail:cleanup --type=limit --force
```

This uses the `LOCAL_MAIL_MAX_RECORDS` configuration value (default: 1000 records).

### Interactive Mode

Run the command without `--force` to be prompted for confirmation:

```bash
php artisan local-mail:cleanup --type=all
# This will ask: "This will delete ALL email messages. Continue?"
```

## Examples

### Manual Cleanup Operations

```bash
# Remove all emails (with confirmation)
php artisan local-mail:cleanup --type=all

# Remove old emails (with confirmation)
php artisan local-mail:cleanup --type=old

# Remove excess emails (with confirmation)
php artisan local-mail:cleanup --type=limit

# Force remove all emails (no confirmation)
php artisan local-mail:cleanup --type=all --force
```

### Scheduled Cleanup

You can automate cleanup using Laravel's task scheduling. Add this to your `app/Console/Kernel.php`:

```php
protected function schedule(Schedule $schedule)
{
    // Clean up old emails daily at 2 AM
    $schedule->command('local-mail:cleanup --type=old --force')->dailyAt('02:00');
    
    // Enforce limits weekly on Sundays at midnight
    $schedule->command('local-mail:cleanup --type=limit --force')->weeklyOn(0, '00:00');
    
    // Monthly deep cleanup (remove all emails)
    $schedule->command('local-mail:cleanup --type=all --force')->monthlyOn(1, '01:00');
}
```

### Cleanup with Logging

The cleanup command logs its operations. You can check your application logs to see cleanup results:

```bash
# Check logs after running cleanup
tail -f storage/logs/laravel.log | grep "Local Mail"
```

Expected log entries:
- `"Local Mail: Deleted X email records to maintain limit of Y"`
- `"Local Mail: Deleted X email records older than Y days"`
- `"Local Mail: Truncated all X email records"`

## Other Commands

The package also provides these additional commands:

### Install Command

```bash
php artisan local-mail:install
```

This command performs installation tasks if needed.

### General Package Command

```bash
php artisan local-mail
```

Provides general package information and options.

## Best Practices

### Regular Maintenance

Set up regular cleanup to prevent database bloat:

```php
// In app/Console/Kernel.php
protected function schedule(Schedule $schedule)
{
    // Daily cleanup of old emails
    $schedule->command('local-mail:cleanup --type=old --force')->daily();
    
    // Weekly enforcement of limits
    $schedule->command('local-mail:cleanup --type=limit --force')->weekly();
}
```

### Development Workflow

During development, you might want to clear all emails at the start of a testing session:

```bash
# Clear all emails before running tests
php artisan local-mail:cleanup --type=all --force
```

### Monitoring Cleanup

Monitor your cleanup operations by checking logs:

```bash
# Monitor cleanup operations
php artisan local-mail:cleanup --type=old --force
# Check logs to confirm operation
```

## Troubleshooting Commands

### Command Not Found

If you get "command not found" errors:

1. Verify the package is properly installed
2. Clear Artisan cache: `php artisan cache:clear`
3. Re-register commands: `composer dump-autoload`

### Permission Issues

If cleanup fails:

1. Check database write permissions
2. Verify the `email_messages` table exists
3. Ensure your database user has DELETE permissions on the table

## Next Steps

- Learn about [security considerations](../security/)
- Explore [troubleshooting tips](../troubleshooting/)
- Understand [advanced usage patterns](../advanced/)