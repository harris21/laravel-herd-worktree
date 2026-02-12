# Laravel Herd Worktree

<p align="center">
  <img src="art/logo.png" alt="Laravel Herd Worktree" width="400">
</p>

A Claude Code skill that automates setting up git worktrees for Laravel projects served by Laravel Herd.

## What It Does

When you need to work on a feature branch in isolation, this skill:

1. **Creates a git worktree** in `.worktrees/<project-name>-<branch-name>`
2. **Links with Laravel Herd** to serve the worktree at `http://<project-name>-<branch-name>.test`
3. **Configures environment** - copies and updates `.env` with correct URLs, session domains, and Sanctum settings
4. **Installs dependencies** - runs `composer install` and `npm install`
5. **Starts development** - kills stale Vite processes and starts fresh

It also handles cleanup and PR creation when you're done.

**Why project name + branch name?** This prevents conflicts when multiple projects have the same branch (e.g., both `project-a` and `project-b` have a `feature/login` branch).

## Prerequisites

- **Laravel Herd** installed and running on macOS
- **Git** for version control
- **Vite** as the frontend build tool (skill assumes Vite, not Webpack/Mix)
- **npm** as package manager (adjust commands if using yarn/pnpm)
- **Laravel Sanctum** if using API authentication (optional - skill handles this if present)

## Installation

### Via Claude Marketplace (Recommended)

```bash
/plugin install laravel-herd-worktree@harrisrafto-laravel-tools
```

### Manual Installation

1. Clone this repository:
   ```bash
   git clone https://github.com/harris21/laravel-herd-worktree.git ~/.claude/plugins/laravel-herd-worktree
   ```

2. Register the marketplace and install the plugin:
   ```bash
   claude plugin marketplace add ~/.claude/plugins/laravel-herd-worktree && claude plugin install laravel-herd-worktree@harrisrafto-laravel-tools
   ```
## Usage

The skill is automatically invoked when you mention worktrees with Laravel Herd projects. Example prompts:

- "Set up a worktree for feature-login"
- "Create an isolated workspace for this branch"
- "I need to work on a feature branch in isolation"

### Manual Invocation

```
/laravel-herd-worktree
```

## Configuration

### Default Branch

The skill defaults to `develop` as the base branch for PRs. If your project uses a different default branch (e.g., `main` or `master`), you can:

1. Let Claude detect it automatically (it will check `git symbolic-ref refs/remotes/origin/HEAD`)
2. Specify it when creating the PR

### Package Manager

The skill assumes npm. If you use yarn or pnpm, the skill will ask before running install commands.

### Composer Flags

If your project requires specific composer flags (like `--ignore-platform-reqs`), the skill will ask during setup.

## What Gets Created

```
your-project/
├── .worktrees/
│   └── your-project-feature-branch/  # Your isolated worktree
│       ├── .env             # Configured for http://your-project-feature-branch.test
│       ├── vendor/          # Fresh composer install
│       └── node_modules/    # Fresh npm install
```

Plus a Herd site at `http://your-project-feature-branch.test`

## Common Issues

### 401 Unauthorized on API routes
- Add worktree domain to `SANCTUM_STATEFUL_DOMAINS` in `.env`
- Run `php artisan config:clear`

### Cookie rejected for invalid domain
- Update `SESSION_DOMAIN` in `.env` to match worktree domain
- Add `SESSION_SECURE_COOKIE=false` for HTTP sites

### CORS Errors / White page
- Ensure `vite.config.js` has `host: 'localhost'` and `cors: true`
- Restart Vite from the worktree directory

### Mixed Content Error
- Don't secure the Herd site (use HTTP, not HTTPS)
- Keep `APP_URL` as `http://` not `https://`

### Assets not loading
- Kill all Vite processes: `pkill -f "node.*vite"`
- Remove hot file: `rm -f public/hot`
- Restart Vite from worktree

See the skill's "Common Issues" section for complete troubleshooting.

## Publishing (For Maintainers)

To add this skill to the Claude marketplace:

```bash
/plugin marketplace add harris21/laravel-herd-worktree
```

To update an already installed plugin:

```bash
/plugin update laravel-herd-worktree@harrisrafto-laravel-tools
```

## License

MIT - see [LICENSE](LICENSE)
