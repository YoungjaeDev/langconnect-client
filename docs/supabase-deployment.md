# Supabase Deployment Guide

This guide walks you through setting up a Supabase project for LangConnect deployment.

## Overview

LangConnect uses Supabase for:
- User authentication (email/password)
- PostgreSQL database with pgvector extension
- JWT token management
- Secure API access control

## Prerequisites

- Supabase account (sign up at https://supabase.com)
- Basic understanding of PostgreSQL
- OpenAI API key for embeddings

## Step 1: Create Supabase Project

1. Log in to Supabase Dashboard
2. Click "New Project"
3. Fill in project details:
   - Project name: e.g., "langconnect-production"
   - Database password: Generate a strong password
   - Region: Select closest to your users
4. Wait for project to be provisioned (1-2 minutes)

## Step 2: Enable pgvector Extension

pgvector is required for vector similarity search.

1. Navigate to Database → Extensions
2. Search for "vector"
3. Enable the "pgvector" extension
4. Verify status shows "Enabled"

## Step 3: Configure Authentication

1. Navigate to Authentication → Providers
2. Enable "Email" provider
3. Configure email settings:
   - Enable email confirmations (recommended for production)
   - Set custom email templates if needed
4. Save changes

Optional: Enable OAuth providers (Google, GitHub, etc.)

## Step 4: Collect Environment Variables

Navigate to Project Settings → API to find your credentials:

### Required Variables

Copy these values to your `.env.local` or `.env.production` file:

```bash
# Supabase project URL
SUPABASE_URL=https://xxxxx.supabase.co

# Supabase anon public key
SUPABASE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# API configuration
API_BASE_URL=http://localhost:8080

# OpenAI API key for embeddings
OPENAI_API_KEY=sk-proj-...
```

### PostgreSQL Direct Connection (Optional)

For direct database access, navigate to Project Settings → Database:

```bash
POSTGRES_HOST=db.xxxxx.supabase.co
POSTGRES_PORT=5432
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your-database-password
POSTGRES_DB=postgres
```

Note: LangConnect primarily uses Supabase's REST API, so direct PostgreSQL connection is optional.

### JWT Secret (For MCP Servers)

The JWT secret is obtained after user authentication:

1. Create a user account through your Next.js UI
2. Sign in successfully
3. Run `make mcp` to generate MCP configuration with valid JWT token
4. The token is automatically added to `.env.local` as `SUPABASE_JWT_SECRET`

Alternatively, you can extract it programmatically using `mcpserver/get_access_token.py`.

## Step 5: Create Environment File

Create `.env.local` in the project root:

```bash
# Copy from .env.example
cp .env.example .env.local

# Edit with your values
nano .env.local
```

Example `.env.local`:

```bash
# API key for the embeddings model
OPENAI_API_KEY=sk-proj-...

# Supabase project URL
SUPABASE_URL=https://qvjxgnecgsxaljqjjnpc.supabase.co

# Supabase anon public key
SUPABASE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# PostgreSQL configuration (optional, for direct access)
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_USER=yeongjae
POSTGRES_PASSWORD=yeongjae
POSTGRES_DB=yeongjae_db

# CORS configuration
ALLOW_ORIGINS=["*"]

# Testing mode (set to false for production)
IS_TESTING=false

# API configuration
API_BASE_URL=http://localhost:8080

# Frontend configuration
NEXT_PUBLIC_API_URL=http://localhost:8080
NEXTAUTH_SECRET=langconnect-next-server-secret
NEXTAUTH_URL=http://localhost:3000

# MCP SSE Server configuration
SSE_PORT=8765

# Supabase JWT access token (obtained after sign-in)
SUPABASE_JWT_SECRET=
```

## Step 6: Test Connection

### Local Development Test

1. Start the services:
```bash
make up
```

2. Check API health:
```bash
curl http://localhost:8080/health
```

Expected response:
```json
{
  "status": "healthy",
  "timestamp": "2025-01-16T12:00:00Z"
}
```

3. Test authentication by accessing Next.js UI:
```bash
open http://localhost:3000
```

4. Sign up with a test account
5. Verify you can sign in successfully

### Verify Supabase Connection

1. In Supabase Dashboard, navigate to Authentication → Users
2. You should see your test account listed
3. Check Database → Table Editor → auth.users for user data

## Step 7: Deploy to Production

When deploying to production environments (Vercel, Fly.io, etc.):

1. Create `.env.production` with production Supabase credentials
2. Never commit environment files to version control
3. Set environment variables through your hosting platform's dashboard
4. Use separate Supabase projects for staging and production
5. Enable RLS (Row Level Security) policies in Supabase for data protection

### Production Environment Variables

Minimum required for production:

```bash
OPENAI_API_KEY=sk-proj-...
SUPABASE_URL=https://prod-project.supabase.co
SUPABASE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
NEXTAUTH_SECRET=generate-strong-secret-for-production
NEXTAUTH_URL=https://your-production-domain.com
NEXT_PUBLIC_API_URL=https://api.your-production-domain.com
API_BASE_URL=https://api.your-production-domain.com
IS_TESTING=false
```

## Security Considerations

### Critical Security Practices

1. Never commit `.env` files to version control
2. Use different Supabase projects for development and production
3. Rotate JWT secrets regularly (access tokens expire in 1 hour by default)
4. Enable email confirmation for production sign-ups
5. Configure RLS policies in Supabase to restrict data access
6. Use strong database passwords (minimum 16 characters)
7. Enable MFA for Supabase dashboard access
8. Regularly audit Authentication logs for suspicious activity

### Token Management

- Access tokens (JWT) expire after 1 hour
- Refresh tokens are stored in httpOnly cookies
- LangConnect automatically refreshes tokens using NextAuth.js
- For MCP servers, re-run `make mcp` when tokens expire

### Environment Variable Security

- Use `.env.local` for local development (gitignored)
- Use platform environment variables for production
- Never expose `SUPABASE_KEY` in client-side code (use Next.js API routes)
- Keep `NEXTAUTH_SECRET` secret and unique per environment

## Troubleshooting

### Common Issues

#### Connection Refused

**Problem**: Cannot connect to Supabase
**Solution**:
- Verify `SUPABASE_URL` and `SUPABASE_KEY` are correct
- Check Supabase project is not paused (free tier auto-pauses after inactivity)
- Ensure network allows connections to Supabase

#### Authentication Fails

**Problem**: Sign-in returns 401 Unauthorized
**Solution**:
- Confirm Email provider is enabled in Supabase
- Check user exists in Authentication → Users
- Verify password is correct
- Clear browser cookies and try again

#### pgvector Extension Not Found

**Problem**: Error about missing pgvector extension
**Solution**:
- Enable pgvector in Database → Extensions
- Wait 1-2 minutes for extension to activate
- Restart API server: `make restart`

#### JWT Token Expired

**Problem**: MCP tools return authentication errors
**Solution**:
- Run `make mcp` to generate fresh token
- Update Claude Desktop configuration
- Restart Claude Desktop completely

### Debugging Tips

1. Check API logs:
```bash
docker-compose logs api
```

2. Verify Supabase connectivity:
```bash
curl -H "apikey: YOUR_SUPABASE_KEY" "YOUR_SUPABASE_URL/rest/v1/"
```

3. Test authentication endpoint:
```bash
curl -X POST http://localhost:8080/auth/signin \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password"}'
```

4. Check environment variables are loaded:
```bash
docker-compose exec api env | grep SUPABASE
```

## Next Steps

After completing Supabase setup:

1. Configure MCP servers: See [MCP WSL Setup Guide](./mcp-wsl-setup.md)
2. Set up authentication flows: See [Authentication Guide](../Auth.md)
3. Deploy to production: Configure your hosting platform
4. Set up monitoring: Enable Supabase logging and alerts

## Additional Resources

- [Supabase Documentation](https://supabase.com/docs)
- [pgvector Documentation](https://github.com/pgvector/pgvector)
- [NextAuth.js Documentation](https://next-auth.js.org/)
- [LangConnect README](../README.md)
- [API Documentation](http://localhost:8080/docs)

## Support

For issues specific to:
- Supabase: Check Supabase Support or Discord
- LangConnect: Open an issue on GitHub
- Authentication: Review Auth.md documentation
