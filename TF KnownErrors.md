**Error**
```
- Unsupported state file format: The state file could not be parsed as JSON: syntax error at byte offset 288.
- Unsupported state file format: The state file does not have a "version" attribute, which is required to identify the format version.
```
**Solution:**
On Windows, `terraform state pull > terraform.tfstate` results in a file with Windows `\r\n` line endings. But `terraform state mv` requires Unix-style `\n` line endings. Converting `terraform.tfstate` to Unix-style line endings fixes the problem.

convert the line endings from windows to Linux
```
((Get-Content .\terraform.tfstate) -join "`n") + "`n" | Set-Content -NoNewline .\terraform.tfstate
```