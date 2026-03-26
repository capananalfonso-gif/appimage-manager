# Security

## Trust model

This tool executes AppImage files (via `--appimage-extract`) to extract icons and metadata. **Only place AppImages from trusted sources in the managed directory.**

The extraction runs with the following mitigations:
- 15-second timeout on all subprocess calls
- Path traversal guards on all extracted file operations
- Temporary directories with automatic cleanup
- No shell execution (`shell=True` is never used)

## Reporting vulnerabilities

If you discover a security vulnerability, please report it privately via email to **vojnisek.peter@qualidat.hu** rather than opening a public issue.

I will respond within 48 hours and work with you on a fix before public disclosure.
