---
layout: page
title: Maintenance
permalink: /docs/maintenance/
nav_order: 6
---

# Maintenance

This section covers maintenance tasks and best practices for managing your Local Mail installation.

## Database Maintenance

### Table Structure

The package creates an `email_messages` table (or custom name) with the following structure:

- `id`: Primary key
- `message_id`: Unique message identifier
- `subject`: Email subject
- `body`: Email content
- `to`, `from`, `cc`, `bcc`: Recipient addresses (JSON)
- `attachments`: Attachment information (JSON)
- `first_email`: First recipient email
- `headers`: Email headers (JSON)
- `created_at`, `updated_at`: Timestamps

### Indexing for Performance

For better performance with large datasets, consider adding indexes:

```php
// In a custom migration
Schema::table(config('local-mail.table_name', 'email_messages'), function (Blueprint $table) {
    $table->index('created_at');
    $table->index(['first_email', 'created_at']);
    $table->index('subject');
    $table->index('message_id');
});
```

## Cleanup Strategies

### Automatic Cleanup

Configure automatic cleanup in your `config/local-mail.php`:

```php
'truncate' => [
    'enabled' => true,
    'max_records' => 1000,
    'cleanup_days' => 30,
    'on_new_email' => false,  // Set to true for aggressive cleanup
],
```

### Scheduled Cleanup

Use Laravel's scheduler for regular maintenance:

```php
// In app/Console/Kernel.php
protected function schedule(Schedule $schedule)
{
    // Daily cleanup of old emails
    $schedule->command('local-mail:cleanup --type=old --force')->daily();
    
    // Weekly limit enforcement
    $schedule->command('local-mail:cleanup --type=limit --force')->weekly();
    
    // Monthly full cleanup (optional)
    $schedule->command('local-mail:cleanup --type=all --force')->monthly();
}
```

### Manual Cleanup

Perform manual cleanup as needed:

```bash
# Check current email count
php artisan tinker
>>> echo \Indianic\LocalMail\Models\EmailMessage::count();

# Clean up old emails
php artisan local-mail:cleanup --type=old --force

# Enforce limits
php artisan local-mail:cleanup --type=limit --force

# Complete reset
php artisan local-mail:cleanup --type=all --force
```

## Storage Management

### Attachment Storage

Attachments are stored in `public/local-emails/`. Monitor this directory:

```bash
# Check attachment directory size
du -sh public/local-emails/

# Remove all attachments (emails will still show but no download links)
rm -rf public/local-emails/*
```

### Database Size

Monitor your database size regularly:

```bash
# MySQL
SELECT table_name AS "Table", 
       ROUND(((data_length + index_length) / 1024 / 1024), 2) AS "Size (MB)" 
FROM information_schema.tables 
WHERE table_schema = "your_database_name" 
AND table_name = "email_messages";

# PostgreSQL
SELECT pg_size_pretty(pg_total_relation_size('email_messages'));
```

## Performance Optimization

### Large Datasets

For applications with many emails:

1. **Enable automatic cleanup** to prevent table bloat
2. **Add database indexes** for common queries
3. **Use pagination** when viewing emails
4. **Consider partitioning** for very large tables

### Query Optimization

When accessing emails programmatically:

```php
// Use pagination for large result sets
$emails = EmailMessage::orderBy('created_at', 'desc')
                     ->paginate(50);

// Use specific queries instead of loading all emails
$emails = EmailMessage::where('first_email', 'user@example.com')
                     ->orderBy('created_at', 'desc')
                     ->limit(100)
                     ->get();

// Use select() to limit fields when appropriate
$emails = EmailMessage::select('id', 'subject', 'first_email', 'created_at')
                     ->orderBy('created_at', 'desc')
                     ->get();
```

## Backup and Recovery

### Database Backup

Include the email messages table in your backups:

```bash
# MySQL
mysqldump -u username -p database_name email_messages > email_backup.sql

# PostgreSQL
pg_dump -t email_messages database_name > email_backup.sql
```

### Selective Backup

For development environments, you might want to exclude email data:

```bash
# Exclude email messages from backup
mysqldump --ignore-table=database_name.email_messages -u username -p database_name > backup.sql
```

## Troubleshooting Common Issues

### Database Performance

If database performance degrades:

1. Check table size and consider cleanup
2. Verify indexes are in place
3. Check for long-running queries
4. Consider partitioning for very large tables

### Attachment Issues

If attachments aren't working:

1. Verify `public/local-emails` directory permissions
2. Check disk space availability
3. Ensure web server can serve files from the directory
4. Verify original attachment files exist

### Memory Issues

If you encounter memory issues:

1. Use pagination when loading emails
2. Process emails in chunks
3. Increase PHP memory limit temporarily
4. Consider using lazy loading

## Monitoring

### Log Monitoring

Monitor cleanup operations in your logs:

```bash
# Monitor cleanup logs
tail -f storage/logs/laravel.log | grep "Local Mail"

# Check for errors
grep -i error storage/logs/laravel.log | grep "Local Mail"
```

### Email Volume Monitoring

Track email capture volume:

```php
// In a monitoring script
$emailCount = \Indianic\LocalMail\Models\EmailMessage::count();
$recentEmails = \Indianic\LocalMail\Models\EmailMessage::where('created_at', '>', now()->subHour())->count();

// Log or alert if volume is unusual
if ($recentEmails > 100) { // Threshold as appropriate
    Log::warning("High email volume detected: {$recentEmails} emails in last hour");
}
```

## Development Workflow

### Testing Environments

In testing environments:

1. Clear emails before each test run
2. Monitor email capture to ensure tests are working
3. Use different cleanup strategies than development

### Local Development

For local development:

1. Use appropriate cleanup settings to prevent disk bloat
2. Monitor attachment storage
3. Consider using different settings than staging/production

## Security Considerations

### Data Retention

Implement appropriate retention policies:

```php
// In config/local-mail.php
'truncate' => [
    'enabled' => true,
    'max_records' => 500,      // Limit for development
    'cleanup_days' => 7,       // Short retention period
    'on_new_email' => true,    // Aggressive cleanup
],
```

### Access Control

Ensure proper access controls:

1. Don't enable in production
2. Use authentication for web interface if needed
3. Restrict access to database data

## Next Steps

- Learn about [security best practices](/docs/security/)
- Explore [troubleshooting tips](/docs/troubleshooting/)
- Understand [advanced usage patterns](/docs/advanced/)