# Vilecong Blog

A modern, responsive blog built with Hugo and PaperMod theme with a profile-style homepage.

## Project Structure

```
vilecong-blog/
├── archetypes/              # Template files for new content
│   └── default.md          # Default markdown template
├── assets/                 # Asset compilation (CSS, JS)
│   └── css/
├── content/                # Blog posts and pages
│   └── posts/
│       └── first-post.md   # Example blog post
├── data/                   # Data files (JSON, YAML)
├── i18n/                   # Internationalization files
├── layouts/                # Custom HTML templates (overrides theme)
│   ├── _default/           # Default templates
│   └── partials/
│       └── extend_head.html # Custom CSS injection
├── public/                 # Generated static site (deployed to GitHub Pages)
├── static/                 # Static files (copied as-is)
│   ├── CNAME              # Custom domain configuration
│   ├── css/
│   │   └── custom.css     # Custom styling (profile layout, colors)
│   └── images/
│       ├── avatar.jpg     # Profile avatar image
│       └── hero.jpg       # Hero section image
├── themes/                # Theme directory
│   └── PaperMod/          # PaperMod theme (git submodule)
├── .github/workflows/     # GitHub Actions CI/CD
│   └── deploy.yml        # Automated Hugo build & deploy
├── hugo.yaml              # Main Hugo configuration file
└── README.md             # This file
```

## Key Configuration Files

### 1. **hugo.yaml** - Main Configuration

Controls all site settings:

- `baseURL`: `https://lcv-back.github.io/vilecongblog.github.io/` (subfolder deployment)
- `relativeURLs: true` - Enable relative URLs for CSS/assets
- `params.profileMode` - Enable profile layout on homepage
- `params.defaultTheme: light` - Use light theme for cream background

**How to modify:**

- Change `title` for site name
- Update `profileMode.title` and `profileMode.subtitle` for homepage text
- Add/remove social icons in `socialIcons` section
- Modify menu links in `menu.main`

### 2. **static/css/custom.css** - Styling

Custom CSS for profile layout:

- Background color: `#E8E0D5` (cream/beige)
- Profile container styling (border, padding, flexbox)
- Responsive design

**How to modify:**

- Change background color in `body { background-color: ... }`
- Adjust profile image size: `imageWidth`, `imageHeight` in hugo.yaml
- Modify fonts in `.profile_inner h1 { font-family: ... }`

### 3. **layouts/partials/extend_head.html** - CSS Injection

Automatically includes `custom.css` in page head:

```html
<link rel="stylesheet" href="/css/custom.css" />
```

### 4. **.github/workflows/deploy.yml** - CI/CD Pipeline

Automatically builds and deploys site when pushing to `main` branch:

1. Checks out repository with submodules
2. Builds Hugo site with `hugo --minify`
3. Deploys to GitHub Pages

### 5. **static/CNAME** - Custom Domain

Contains: `vilecongblog.github.io`

- Tells GitHub Pages to serve site from custom domain
- Must be present in `static/` folder (copied to `public/` by Hugo)

## File Purposes

| File/Folder                    | Purpose                      | Edit When                      |
| ------------------------------ | ---------------------------- | ------------------------------ |
| `hugo.yaml`                    | Site configuration & content | Change site info, colors, menu |
| `static/css/custom.css`        | Custom styling               | Adjust layout, colors, fonts   |
| `static/images/avatar.jpg`     | Profile picture              | Update profile photo           |
| `content/posts/`               | Blog articles                | Write new blog posts           |
| `static/CNAME`                 | Custom domain                | Change custom domain           |
| `.github/workflows/deploy.yml` | Build & deploy               | Modify build process           |

## How to Edit Homepage

### Change Profile Title/Subtitle

**File:** `hugo.yaml`

```yaml
params:
  profileMode:
    title: "Your Name"
    subtitle: "Your tagline here"
```

### Change Avatar Image

1. Replace `static/images/avatar.jpg` with new image
2. Ensure filename is `avatar.jpg`
3. Image size: 300x300px recommended

### Modify Colors & Styling

**File:** `static/css/custom.css`

- Line 7: Background color `#E8E0D5`
- Line 21-23: Border and layout adjustments
- Line 37-40: Font sizes and styles

### Change Social Icons

**File:** `hugo.yaml`

```yaml
socialIcons:
  - name: "github"
    url: "https://github.com/your-username"
  - name: "linkedin"
    url: "https://linkedin.com/in/your-profile"
```

### Change Profile Buttons (BIO, PORTFOLIO, BLOG)

**File:** `hugo.yaml`

```yaml
profileMode:
  buttons:
    - name: "BIO"
      url: "/about"
    - name: "PORTFOLIO"
      url: "/portfolio"
    - name: "BLOG"
      url: "/posts"
```

## Deployment

### Local Preview

```bash
hugo server --buildDrafts
```

Visit: `http://localhost:1313/vilecongblog.github.io/`

### Build for Production

```bash
hugo --minify
```

Output goes to `public/` folder

### Deploy to GitHub Pages

```bash
git add -A
git commit -m "Your message"
git push
```

GitHub Actions automatically:

1. Builds Hugo site
2. Deploys to GitHub Pages
3. Site available at: `https://vilecongblog.github.io/`

## Important Notes

### Subfolder Deployment

- Site is deployed to subfolder: `/vilecongblog.github.io/`
- `baseURL` must include subfolder: `https://lcv-back.github.io/vilecongblog.github.io/`
- `relativeURLs: true` automatically handles relative paths

### Theme Customization

- PaperMod theme files in `themes/PaperMod/`
- Custom overrides go in `layouts/` folder
- Don't edit theme files directly (they update via git)

### Adding New Blog Posts

1. Create file: `content/posts/post-title.md`
2. Add front matter:

```yaml
---
title: "Post Title"
date: 2024-04-18
draft: false
---
```

3. Write content in Markdown
4. Push to trigger automatic build

## Troubleshooting

| Issue            | Solution                                   |
| ---------------- | ------------------------------------------ |
| CSS not loading  | Check `relativeURLs: true` in hugo.yaml    |
| Images broken    | Verify images in `static/images/` folder   |
| Menu links wrong | Update `menu.main` in hugo.yaml            |
| Deploy failed    | Check GitHub Actions log in Repo > Actions |
| Wrong domain     | Verify `static/CNAME` content              |

## Links

- **GitHub Repo**: https://github.com/lcv-back/vilecongblog.github.io
- **Live Site**: https://vilecongblog.github.io/
- **Hugo Docs**: https://gohugo.io/documentation/
- **PaperMod Theme**: https://github.com/adityatelange/hugo-PaperMod
