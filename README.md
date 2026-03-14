# ZipTax Sales Tax Skill

A [Claude Skill](https://docs.claude.com/en/docs/claude-code/skills) that teaches Claude how to effectively use the [ZipTax MCP server](https://github.com/ZipTax/ziptax-mcp) for US and Canadian sales tax rate lookups.

## What This Skill Does

This skill enhances Claude's ability to work with the ZipTax MCP server by providing:

- **Workflow guidance** for common tax lookup scenarios (by address, ZIP code, coordinates, or historical period)
- **Parameter selection logic** so Claude chooses the right inputs for the most accurate results
- **Response interpretation** so Claude presents tax rate breakdowns clearly and completely
- **Error handling** with actionable resolution steps

## Prerequisites

- A ZipTax API key from [platform.zip.tax](https://platform.zip.tax)
- The ZipTax MCP server connected at `https://mcp.zip-tax.com`

## Installation

### Claude Code

```bash
claude plugin add ZipTax/ziptax-claude-skill
```

Restart Claude Code after installation. The skill activates automatically when you ask about sales tax rates.

### Manual Installation

Copy the `SKILL.md` file to your Claude skills directory:

```bash
# Personal skill (available across all projects)
mkdir -p ~/.claude/skills/ziptax
cp SKILL.md ~/.claude/skills/ziptax/SKILL.md

# Or project-level skill
mkdir -p .claude/skills/ziptax
cp SKILL.md .claude/skills/ziptax/SKILL.md
```

## MCP Server Configuration

Configure the ZipTax MCP server in your MCP client:

### Claude Code

Add to your MCP configuration:

```json
{
  "ziptax": {
    "type": "http",
    "url": "https://mcp.zip-tax.com/",
    "headers": {
      "X-API-KEY": "your_api_key"
    }
  }
}
```

### Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "ziptax": {
      "type": "http",
      "url": "https://mcp.zip-tax.com/",
      "headers": {
        "X-API-KEY": "your_api_key"
      }
    }
  }
}
```

Replace `your_api_key` with your ZipTax API key from [platform.zip.tax](https://platform.zip.tax).

## Example Prompts

Once the skill and MCP server are configured, try:

- "What's the sales tax rate for 200 Spectrum Center Drive, Irvine, CA 92618?"
- "Look up the tax rate for ZIP code 10001"
- "What was the sales tax rate in Austin, TX in December 2023?"
- "Is shipping taxable in California?"
- "Check my ZipTax API usage"

## Tools Provided by the MCP Server

| Tool | Description |
|---|---|
| `lookup_tax_rate` | Look up sales and use tax rates by postal code, address, or coordinates |
| `get_account_metrics` | Get account usage metrics and quota information |

## Resources

- [ZipTax API Documentation](https://developers.zip.tax)
- [ZipTax MCP Server](https://github.com/ZipTax/ziptax-mcp)
- [Get an API Key](https://platform.zip.tax)
- [Claude Skills Documentation](https://docs.claude.com/en/docs/claude-code/skills)
- [Agent Skills Standard](https://agentskills.io)

## License

Apache License 2.0 — see [LICENSE](LICENSE) for details.
