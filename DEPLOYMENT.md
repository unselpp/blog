# Deploying to Vercel

There are two easy ways to deploy your Astro blog to Vercel:

## Option 1: GitHub Integration (Recommended)

This is the easiest and keeps your code backed up:

1. **Initialize Git and push to GitHub:**
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   ```
   
   Then create a new repository on GitHub and push:
   ```bash
   git remote add origin https://github.com/yourusername/your-repo-name.git
   git branch -M main
   git push -u origin main
   ```

2. **Deploy on Vercel:**
   - Go to [vercel.com](https://vercel.com) and sign up/login
   - Click "Add New Project"
   - Import your GitHub repository
   - Vercel will auto-detect Astro - no configuration needed!
   - Click "Deploy"
   - Done! Your site will be live in ~2 minutes

3. **Custom Domain (Optional):**
   - In your Vercel project settings, go to "Domains"
   - Add your custom domain
   - Follow the DNS instructions

## Option 2: Vercel CLI (Quick Deploy)

If you want to deploy without GitHub:

1. **Install Vercel CLI:**
   ```bash
   npm install -g vercel
   ```

2. **Login:**
   ```bash
   vercel login
   ```

3. **Deploy:**
   ```bash
   vercel
   ```
   
   Follow the prompts - it will ask:
   - Link to existing project? (No for first time)
   - Project name? (Press enter for default)
   - Directory? (Press enter for current directory)
   
4. **Deploy to production:**
   ```bash
   vercel --prod
   ```

That's it! Your blog will be live at `your-project.vercel.app`

## Build Settings

Vercel automatically detects Astro projects, but if needed:
- **Build Command:** `npm run build`
- **Output Directory:** `dist`
- **Install Command:** `npm install`

## Environment Variables

If you add any environment variables later, add them in:
Vercel Dashboard → Your Project → Settings → Environment Variables

## Continuous Deployment

With GitHub integration, every push to `main` automatically deploys!

