# Creating a Netlify Deployment

This guide documents the process of creating a Netlify project, deploying a site, and connecting it to a GitHub repository for continuous deployment.

## Prerequisites

1. **Netlify CLI installed and authenticated**
   ```bash
   # Check if Netlify CLI is installed
   which netlify
   
   # If not installed, install it:
   npm install -g netlify-cli
   
   # Login to Netlify (if not already logged in)
   netlify login
   ```

2. **GitHub repository** - The repository should already exist in the Pink-Marlin-Digital organization

3. **Project ready** - The project should have:
   - A build command configured (e.g., `pnpm build` or `npm run build`)
   - A build output directory (e.g., `dist/` for Astro projects)
   - All dependencies installed

## Step 1: Create Netlify Configuration File

Create a `netlify.toml` file in the root of your project:

```toml
[build]
  command = "pnpm build"
  publish = "dist"

[[plugins]]
  package = "@netlify/plugin-lighthouse"
```

**Note:** Adjust the `command` and `publish` values based on your project:
- For npm projects: `command = "npm run build"`
- For yarn projects: `command = "yarn build"`
- For Astro projects: `publish = "dist"`
- For React/Vite projects: `publish = "dist"`
- For Next.js: `publish = ".next"`

## Step 2: Create Netlify Site

Create a new Netlify site using the CLI:

```bash
# Create a new site (will prompt for team selection)
netlify sites:create --name your-site-name
```

**Important:** 
- The site name must be unique across all Netlify sites
- Select the appropriate team (e.g., "Pink Marlin Digital") when prompted
- The command will output the site ID, admin URL, and live URL

**Example output:**
```
Site Created

Admin URL: https://app.netlify.com/projects/your-site-name
URL:       https://your-site-name.netlify.app
Site ID:   xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

Linked to your-site-name
```

## Step 3: Link Local Directory to Netlify Site

If the site was just created, it should already be linked. Verify with:

```bash
netlify status
```

If not linked, link it manually:

```bash
netlify link
```

This will prompt you to either:
- Connect to an existing site (select this if the site already exists)
- Create a new site

## Step 4: Initial Deployment

Deploy the site to verify everything works:

```bash
# Build and deploy to production
netlify deploy --build --prod
```

This command will:
1. Run the build command specified in `netlify.toml`
2. Upload the build output to Netlify
3. Deploy to the production URL

**Expected output:**
```
Build logs:         https://app.netlify.com/projects/your-site-name/deploys/xxxxx
Website URL:        https://your-site-name.netlify.app
```

## Step 5: Connect GitHub Repository for Continuous Deployment

### Option A: Via Netlify Web UI (Recommended)

The GitHub connection requires OAuth authorization, which is best done through the web interface:

1. **Open the Netlify Admin Panel:**
   ```bash
   netlify open:admin
   ```
   Or navigate to: `https://app.netlify.com/projects/your-site-name`

2. **Navigate to Site Settings:**
   - Click on "Site settings" in the top navigation
   - Go to "Build & deploy" → "Continuous Deployment"

3. **Link Repository:**
   - Click "Link repository" or "Configure provider"
   - Select "GitHub" as your Git provider
   - If prompted, authorize Netlify to access your GitHub account
   - Select the organization: `Pink-Marlin-Digital`
   - Select the repository: `your-repository-name`

4. **Configure Build Settings:**
   - **Branch to deploy:** `main` (or your default branch)
   - **Build command:** `pnpm build` (or your build command)
   - **Publish directory:** `dist` (or your build output directory)
   - Click "Save"

5. **Verify Connection:**
   - After saving, Netlify will trigger an initial deployment from GitHub
   - You should see a new deploy in the "Deploys" tab

### Option B: Via Netlify API (Advanced)

If you have API access and the repository is already authorized, you can try to update via API:

```bash
# Get your site ID first
cat .netlify/state.json | grep siteId

# Update site configuration (may require OAuth to be set up first)
netlify api updateSite --data '{
  "site_id": "your-site-id",
  "repo": {
    "provider": "github",
    "repo": "Pink-Marlin-Digital/your-repo-name",
    "branch": "main",
    "cmd": "pnpm build",
    "dir": "dist"
  }
}'
```

**Note:** This method may not work if GitHub OAuth hasn't been authorized yet. Use Option A if this fails.

## Step 6: Commit and Push Configuration

Commit the `netlify.toml` file to your repository:

```bash
git add netlify.toml
git commit -m "Add Netlify configuration"
git push origin main
```

## Step 7: Verify Continuous Deployment

After connecting GitHub:

1. **Make a test change** to your repository
2. **Push to the main branch:**
   ```bash
   git add .
   git commit -m "Test: Verify Netlify auto-deploy"
   git push origin main
   ```
3. **Check Netlify Dashboard:**
   - Go to the "Deploys" tab in your Netlify site
   - You should see a new deploy triggered automatically
   - The deploy status will show "Building" → "Published"

## Troubleshooting

### Site Creation Fails

- **Issue:** Site name already exists
- **Solution:** Choose a different, unique site name

### Build Fails

- **Issue:** Build command fails
- **Solution:** 
  - Test the build locally: `pnpm build` (or your build command)
  - Check build logs in Netlify dashboard
  - Verify all dependencies are in `package.json`
  - Ensure Node.js version is compatible (check `package.json` engines or `.nvmrc`)

### GitHub Connection Fails

- **Issue:** Cannot connect to GitHub repository
- **Solution:**
  - Ensure you're logged into the correct Netlify account
  - Verify the repository exists in the Pink-Marlin-Digital organization
  - Check that you have access to the repository
  - Try disconnecting and reconnecting the repository

### Deployment Not Triggering

- **Issue:** Pushes to GitHub don't trigger deployments
- **Solution:**
  - Verify the branch name matches (usually `main`)
  - Check that the repository is properly linked in Netlify
  - Look for webhook errors in GitHub repository settings
  - Check Netlify build logs for errors

## Useful Commands

```bash
# Check Netlify status
netlify status

# View site information
netlify api getSite --data '{"site_id": "your-site-id"}'

# Open site in browser
netlify open:site

# Open admin panel
netlify open:admin

# View deploy logs
netlify deploy:list

# Deploy without building (if already built)
netlify deploy --prod --dir=dist

# Deploy a draft (not production)
netlify deploy
```

## Project-Specific Notes

### For Astro Projects

- Build command: `pnpm build` or `npm run build`
- Publish directory: `dist`
- Ensure `astro.config.mjs` is properly configured

### For React/Vite Projects

- Build command: `pnpm build` or `npm run build`
- Publish directory: `dist`
- May need to configure base path if using subdirectories

### For Next.js Projects

- Build command: `next build`
- Publish directory: `.next`
- May require additional Netlify plugins for Next.js

## Additional Resources

- [Netlify Documentation](https://docs.netlify.com/)
- [Netlify CLI Documentation](https://cli.netlify.com/)
- [Continuous Deployment Guide](https://docs.netlify.com/site-deploys/create-deploys/)
- [Build Configuration](https://docs.netlify.com/configure-builds/overview/)

## Summary Checklist

- [ ] Netlify CLI installed and authenticated
- [ ] `netlify.toml` created with correct build settings
- [ ] Netlify site created
- [ ] Local directory linked to Netlify site
- [ ] Initial deployment successful
- [ ] GitHub repository connected via web UI
- [ ] Build settings configured (branch, command, directory)
- [ ] `netlify.toml` committed to repository
- [ ] Continuous deployment verified with test push
- [ ] Site accessible at `https://your-site-name.netlify.app`

