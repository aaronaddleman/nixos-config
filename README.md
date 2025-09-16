# NixOS Configuration

This repository contains NixOS and nix-darwin configurations using Nix flakes.

## Common Nix Commands

### Flake Apps (Custom Scripts)

This repository includes custom flake apps for common operations:

```bash
# Apply configuration changes
nix run .#apply

# Build and switch in one command
nix run .#build-switch

# Check available apps
nix flake show | grep apps

# Run other available apps (check apps/ directory for available options)
nix run .#build
nix run .#check-keys
nix run .#copy-keys
nix run .#create-keys
nix run .#rollback
```

### Flake Management

```bash
# Update flake inputs
nix flake update

# Update specific input
nix flake update nixpkgs

# Show flake metadata
nix flake show

# Check flake for issues
nix flake check
```

### Building and Switching

#### macOS (nix-darwin)
```bash
# Build configuration
nix build .#darwinConfigurations.<hostname>.system

# Build for specific architecture (useful for M1/M2 Macs)
nix build .#darwinConfigurations.aarch64-darwin.system --show-trace

# Build and switch using flake apps (recommended)
nix run .#build-switch

# Apply configuration using flake app
nix run .#apply

# Traditional darwin-rebuild commands
darwin-rebuild switch --flake .
darwin-rebuild build --flake .
darwin-rebuild rollback
```

#### NixOS (Linux)
```bash
# Build configuration
nix build .#nixosConfigurations.<hostname>.config.system.build.toplevel

# Build and switch to new configuration
sudo nixos-rebuild switch --flake .

# Build without switching
sudo nixos-rebuild build --flake .

# Test configuration (temporary, reverts on reboot)
sudo nixos-rebuild test --flake .

# Boot into new configuration on next reboot
sudo nixos-rebuild boot --flake .

# Rollback to previous generation
sudo nixos-rebuild switch --rollback
```

### Generations Management

```bash
# List system generations
nix-env --list-generations --profile /nix/var/nix/profiles/system

# Delete old generations (keep last 3)
sudo nix-collect-garbage -d --delete-older-than 3d

# Delete specific generation
sudo nix-env --delete-generations <generation-number> --profile /nix/var/nix/profiles/system
```

### Development and Testing

```bash
# Enter development shell
nix develop

# Run a specific app from flake
nix run .#<app-name>

# Build specific output with detailed trace
nix build .#<output> --show-trace

# Build specific output
nix build .#<output>

# Evaluate Nix expression
nix eval .#<attribute>

# Show build logs
nix log <derivation-path>

# Check flake for issues
nix flake check
```

### Package Management

```bash
# Search for packages
nix search nixpkgs <package-name>

# Install package temporarily
nix shell nixpkgs#<package-name>

# Run package without installing
nix run nixpkgs#<package-name>

# Show package information
nix eval nixpkgs#<package-name>.meta
```

### Secrets Management (agenix)

```bash
# Edit secret
agenix -e <secret-name>.age

# Rekey all secrets
agenix -r

# List secrets
agenix -l
```

### Homebrew Configuration (macOS)

For macOS systems, Homebrew packages and casks are managed through Nix:

```bash
# Install/manage Homebrew packages
# Add packages to modules/darwin/packages.nix
# Add casks to modules/darwin/casks.nix

# View current Homebrew status
brew doctor

# List installed packages
brew list

# Update Homebrew (managed by Nix)
nix run .#build-switch  # This will update Homebrew as part of the build
```

#### Adding Custom Homebrew Taps

To add custom Homebrew taps, modify the `flake.nix` file:

1. **Add the tap as a flake input** (in the `inputs` section):
```nix
custom-tap = {
  url = "github:owner/homebrew-tapname";
  flake = false;
};
```

2. **Add the input to the outputs function**:
```nix
outputs = { self, darwin, nix-homebrew, homebrew-bundle, homebrew-core, homebrew-cask, custom-tap, home-manager, nixpkgs, disko, agenix } @inputs:
```

3. **Configure the tap in the nix-homebrew section** (around line 92):
```nix
taps = {
  "homebrew/homebrew-core" = homebrew-core;
  "homebrew/homebrew-cask" = homebrew-cask;
  "homebrew/homebrew-bundle" = homebrew-bundle;
  "owner/homebrew-tapname" = custom-tap;
};
```

4. **Rebuild your system**:
```bash
nix run .#build-switch
```

**Configuration Files:**
- `modules/darwin/packages.nix` - Homebrew CLI packages
- `modules/darwin/casks.nix` - Homebrew GUI applications (casks)
- `flake.nix` - Tap definitions and nix-homebrew configuration

### System Information

```bash
# Show current system configuration
nix-store --query --references /run/current-system

# Show system profile
nix profile history --profile /nix/var/nix/profiles/system

# Check store integrity
nix store verify --all

# Show store statistics
nix store info
```

### Cleanup

```bash
# Garbage collect unused packages
nix-collect-garbage

# Aggressive garbage collection
nix-collect-garbage -d

# Optimize nix store
nix store optimise
```

### Remote Building

```bash
# Build for different architecture
nix build .#nixosConfigurations.<hostname>.config.system.build.toplevel --system <arch>

# Build on remote machine
nixos-rebuild switch --flake . --target-host <hostname>
```

## Troubleshooting

### Common Issues

1. **Flake lock conflicts**: `nix flake update` or edit `flake.lock` manually
2. **Build failures**: Check logs with `nix log` or use `--show-trace` for detailed errors
3. **Generation rollback**: Use `nixos-rebuild switch --rollback` or `darwin-rebuild rollback`
4. **Storage cleanup**: Run `nix-collect-garbage -d` to free up space

### Useful Debugging Options

```bash
# Show detailed build trace
nixos-rebuild switch --flake . --show-trace

# Keep build directories for inspection  
nixos-rebuild switch --flake . --keep-going

# Verbose output
nixos-rebuild switch --flake . --verbose

# Dry run (show what would be done)
nixos-rebuild switch --flake . --dry-run
```

## Repository Structure

- `flake.nix` - Main flake configuration
- `flake.lock` - Locked input versions
- `hosts/` - Host-specific configurations
  - `darwin/` - macOS configurations
  - `nixos/` - NixOS configurations
- `modules/` - Reusable configuration modules
  - `darwin/` - macOS-specific modules
  - `nixos/` - NixOS-specific modules  
  - `shared/` - Cross-platform modules
- `overlays/` - Package overlays
- `apps/` - Flake apps for different architectures

## Getting Started

1. Clone this repository
2. Update hardware configuration in appropriate host file
3. Run initial build: `nixos-rebuild switch --flake .` (Linux) or `darwin-rebuild switch --flake .` (macOS)
4. Reboot and enjoy your new system configuration!
