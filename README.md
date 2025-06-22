# AutoPkg Runner

A GitHub Actions-based automation system for running AutoPkg recipes and managing Munki repositories. This project provides automated package building, virus scanning, and intelligent promotion of packages through testing catalogs.

## Features

- 🤖 **Automated Package Building**: Run AutoPkg recipes on schedule or manually
- 📦 **Munki Integration**: Direct integration with Munki repositories for package management
- 🔄 **Autopromote**: Automatically promote packages through testing catalogs based on age
- 🦠 **Virus Scanning**: Built-in VirusTotal integration for security scanning
- 💬 **Slack Notifications**: Real-time build status updates via Slack
- 📊 **Metadata Caching**: Smart caching system to avoid redundant downloads
- 🧹 **Repository Cleanup**: Automated removal of old packages
- 📝 **YAML Support**: Support for both .recipe and .yaml recipe formats

## Table of Contents

- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Configuration](#configuration)
- [Usage](#usage)
- [Workflows](#workflows)
- [Directory Structure](#directory-structure)
- [Custom Processors](#custom-processors)
- [Contributing](#contributing)
- [License](#license)

## Prerequisites

- GitHub repository with Actions enabled
- Munki repository (separate GitHub repo recommended)
- AutoPkg recipes
- Slack webhook (optional, for notifications)
- SSH deploy key for Munki repository access

## Setup

### 1. Fork or Clone This Repository

```bash
git clone https://github.com/yourusername/autopkg-runner.git
cd autopkg-runner
```

### 2. Configure GitHub Secrets

Add the following secrets to your GitHub repository settings:

- `CPE_MUNKI_LFS_DEPLOY_KEY`: SSH deploy key with read/write access to your Munki repo
- `SLACK_WEBHOOK_TOKEN`: Slack webhook URL for notifications
- `GITHUB_TOKEN`: Automatically provided by GitHub Actions
- `AUTOPROMOTE_SLACK_TOKEN`: Slack token for autopromote notifications (optional)

### 3. Update Configuration Files

1. **Update Munki Repository References**:
   - In `.github/workflows/autopkg.yml`: Update `MUNKI_REPO` path
   - In `.github/workflows/autopromote.yml`: Replace `replace_with_your_munki_repo`
   - In `.github/workflows/repoclean.yml`: Replace `replace_with_your_munki_repo`

2. **Configure Recipe List**:
   Edit `recipe_list.json` to include your desired recipes:
   ```json
   [
     "Firefox.munki.recipe",
     "GoogleChrome.munki.recipe",
     "YourApp.munki.recipe"
   ]
   ```

3. **Autopromote Configuration**:
   Edit `autopromote/autopromote.json` to customize catalog promotion rules and timing.

## Configuration

### Recipe List (`recipe_list.json`)

Specify which AutoPkg recipes to run:

```json
[
  "Ghostty.munki.recipe.yaml",
  "VisualStudioCode.munki.recipe"
]
```

### Autopromote Settings (`autopromote/autopromote.json`)

Key configuration options:

```json
{
  "channels": {
    "slow": 2.5,    // Multiplier for slow promotion
    "fast": 0.5     // Multiplier for fast promotion
  },
  "catalogs": {
    "import": {
      "days": 1,
      "next": "canary"
    },
    "canary": {
      "days": 5,
      "next": "prerelease",
      "force_install_days": 4
    }
  }
}
```

## Usage

### Manual Recipe Run

1. Go to Actions tab in your GitHub repository
2. Select "Autopkg run" workflow
3. Click "Run workflow"
4. Optionally specify:
   - Specific recipe to run (e.g., `Firefox.munki.recipe`)
   - Enable/disable debug mode

### Scheduled Runs

The autopkg workflow runs automatically:
- Monday-Friday at 14:00 UTC
- Can be customized in `.github/workflows/autopkg.yml`

### Monitoring

- **Slack Notifications**: Build results are posted to configured Slack channel
- **GitHub Actions Logs**: Detailed logs available in Actions tab
- **Metadata Cache**: View cached download information for debugging

## Workflows

### 1. AutoPkg Run (`autopkg.yml`)

Main workflow for running AutoPkg recipes:
- Installs AutoPkg and Munki tools
- Runs specified recipes
- Commits new packages to Munki repo
- Updates metadata cache
- Sends Slack notifications

**Triggers**:
- Schedule: Weekdays at 14:00 UTC
- Manual: Via workflow dispatch
- Watch: When repository is starred

### 2. Autopromote (`autopromote.yml`)

Promotes packages through testing catalogs:
- Runs daily at 16:30 UTC
- Moves packages based on age and configuration
- Creates pull requests for catalog changes

### 3. Repository Cleanup (`repoclean.yml`)

Maintains repository hygiene:
- Runs weekly (Wednesdays at 13:00 UTC)
- Removes old package versions
- Keeps latest 2 versions by default

## Directory Structure

```
.
├── .github/
│   └── workflows/
│       ├── autopkg.yml         # Main AutoPkg workflow
│       ├── autopromote.yml     # Package promotion workflow
│       └── repoclean.yml       # Repository cleanup workflow
├── autopromote/
│   ├── autopromote.json        # Promotion configuration
│   └── autopromote.py          # Promotion logic
├── codebase/
│   ├── autopkg_tools.py        # Main AutoPkg wrapper
│   └── helpers/
│       └── slack_notify.zsh    # Slack notification helper
├── recipes/                    # Custom AutoPkg recipes
│   ├── CacheRecipeMetadata/    # Metadata caching processor
│   └── VirusTotalAnalyzer/     # Virus scanning processor
├── repos/                      # AutoPkg recipe repos (managed)
├── overrides/                  # Recipe overrides
├── recipe_list.json           # List of recipes to run
└── requirements.txt           # Python dependencies
```

## Custom Processors

### CacheRecipeMetadata

Caches download metadata to avoid redundant downloads:
- Stores file hashes, ETags, and modification times
- Creates placeholder files for AutoPkg's download detection
- Automatically enabled with `--cache` flag

### VirusTotalAnalyzer

Scans downloaded files for malware:
- Integrated API key included
- Automatic submission for unknown files
- Results included in Slack notifications

## Environment Variables

Available for customization:

- `RECIPE`: Specific recipe to run
- `DEBUG`: Enable debug logging
- `SLACK_WEBHOOK_TOKEN`: Slack webhook URL
- `VIRUSTOTAL_API_KEY`: Custom VirusTotal API key
- `METADATA_CACHE_PATH`: Custom cache location

## Troubleshooting

### Common Issues

1. **Recipe Not Found**:
   - Ensure recipe name in `recipe_list.json` matches exactly
   - Check recipe exists in `recipes/` or `repos/` directories

2. **Munki Repo Access Denied**:
   - Verify SSH deploy key has write access
   - Check repository URL in workflow files

3. **Slack Notifications Not Working**:
   - Verify webhook URL is correctly set in secrets
   - Check Slack channel permissions

### Debug Mode

Enable debug mode for verbose logging:
1. Manual run: Set debug to 'True' in workflow dispatch
2. Command line: Add `--debug` flag to autopkg_tools.py

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Adding New Recipes

1. Add recipe file to `recipes/` directory
2. Update `recipe_list.json`
3. Test locally before committing
4. Submit PR with description of the recipe

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

### Acknowledgments

- Original autopkg_tools.py based on work by Facebook, Inc. and Gusto, Inc.
- VirusTotalAnalyzer processor by Hannes Juutilainen
- Autopromote system by Harry Seeber, Gusto ITCPE

## Support

For issues, questions, or contributions:
- Open an issue in this repository
- Check existing issues for solutions
- Review GitHub Actions logs for detailed error messages

