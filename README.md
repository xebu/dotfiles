# dotfiles

Managed with chezmoi.

## Bootstrap (clean Mac)

/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
eval "$(/opt/homebrew/bin/brew shellenv)"
brew install chezmoi
chezmoi init --apply xebu
exec zsh
