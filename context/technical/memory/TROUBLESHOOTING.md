# Troubleshooting Guide

## Metadata
- **Category**: Memory
- **Last Update**: 2025-10-20
- **Owner**: CTO (Lucas Gaillard)
- **Status**: Active
- **Source Files**: `/README.md`, operational experience

## Common Issues

### Database Connection Issues

**Issue**: Cannot connect to database
```
Error: connect ECONNREFUSED 127.0.0.1:5432
```

**Causes**:
- PostgreSQL not running
- Incorrect connection string
- Network issues

**Solutions**:
```bash
# Check if PostgreSQL is running
docker ps

# Restart PostgreSQL
cd apps/api
docker-compose restart db

# Verify connection string in .env
cat apps/api/.env | grep POSTGRES
```

---

### Module Resolution Errors

**Issue**: Module not found
```
Error: Cannot find module '@boilerplate-turbo/types'
```

**Causes**:
- Workspace dependencies not installed
- Types package not built
- Node modules out of sync

**Solutions**:
```bash
# Clear and reinstall
rm -rf node_modules apps/*/node_modules packages/*/node_modules
npm install

# Build types package
npm run build:packages

# Restart TypeScript server in IDE
```

---

### Port Already in Use

**Issue**: Port conflict
```
Error: listen EADDRINUSE: address already in use :::3000
```

**Solutions**:
```bash
# Find process using port
lsof -ti:3000

# Kill the process
kill -9 <PID>

# Or use different port
PORT=3002 npm run dev
```

---

### Transaction Not Rolling Back

**Issue**: Transaction commits despite error

**Causes**:
- Error status code < 300
- Transaction middleware not applied
- Error thrown after response sent

**Solutions**:
```typescript
// Ensure proper status codes
res.status(500).json({ error: 'ERROR_CODE' });  // ≥ 300 triggers rollback

// Verify middleware order
app.use(withTransaction())
   .use('/api', routes);

// Don't send response before operations complete
```

---

### Validation Errors Not Caught

**Issue**: Invalid data reaches route handler

**Causes**:
- Validation middleware not applied
- Middleware in wrong order

**Solutions**:
```typescript
// Apply validation before handler
router.post('/',
  validateRequestBody(schema),    // Must come before handler
  handler
);
```

---

### Clerk Authentication Issues

**Issue**: User authenticated but not found in database

**Causes**:
- Webhook not received
- Webhook signature verification failed
- Database sync issue

**Solutions**:
```bash
# Check webhook logs
grep "clerk-webhook" logs/api.log

# Verify webhook secret
echo $CLERK_WEBHOOK_SIGNING_SECRET

# Manually sync user if needed (check Clerk dashboard)
```

---

### Docker Compose Communication Issues

**Issue**: Services cannot reach each other

**Causes**:
- Using `localhost` instead of service names
- Services in wrong network
- Port mapping issues

**Solutions**:
```yaml
# Use service names as hostnames
environment:
  - DATABASE_HOST=db  # NOT localhost
  - API_HOST=api      # NOT localhost
```

---

### Turbo Cache Issues

**Issue**: Stale build artifacts

**Causes**:
- Turbo cache not invalidated
- Dependencies changed but not detected

**Solutions**:
```bash
# Clear Turbo cache
rm -rf .turbo

# Force rebuild
turbo run build --force

# Or disable cache temporarily
turbo run build --no-cache
```

---

### Source Map Issues

**Issue**: Sentry shows minified stack traces

**Causes**:
- Source maps not uploaded
- Release version mismatch
- Source map configuration incorrect

**Solutions**:
```bash
# Upload source maps for API
cd apps/api
npm run sentry:sourcemaps

# Verify COMMIT_SHA matches
echo $COMMIT_SHA
```

---

### E2E Test Failures

**Issue**: Playwright tests fail inconsistently

**Causes**:
- Timing issues
- State not reset between tests
- Clerk test environment issues

**Solutions**:
```typescript
// Add proper waits
await page.waitForSelector('[data-testid="user-menu"]');

// Reset state in beforeEach
beforeEach(async () => {
  await setupClerkTestingToken({ page });
});

// Increase timeouts if needed
test.setTimeout(60000);
```

---

## **[To be enriched with operational experience]**

As issues arise in production or development, document:
- Problem description
- Root cause
- Solution steps
- Prevention measures

### Template for New Issues

```markdown
### Issue Title

**Issue**: Brief description

**Symptoms**:
- Observable behavior
- Error messages

**Causes**:
- Root cause analysis

**Solutions**:
# Step-by-step fix

**Prevention**:
- How to avoid in future
```

---

## References

- **README**: `/README.md` (Troubleshooting section)
- **Logs**: Check application logs for detailed errors
- **Monitoring**: Sentry dashboard for production issues
- **Owner**: CTO (Lucas Gaillard) for escalation
