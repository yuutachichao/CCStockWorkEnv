# 發送 Email

Send an email via Mailgun API.

## Usage

```
/send_email <subject> | <body>
/send_email
```

## Examples

```
/send_email TSLA 分析報告 | 詳見附件中的完整分析報告
/send_email   (sends last Claude response as email body)
```

## Instructions

When this command is invoked:

1. **Determine the content:**
   - If the user provided arguments, parse `<subject> | <body>` (split on `|`)
   - If only subject is provided (no `|`), use the subject and your last response as the body
   - If NO arguments provided, use "CCStockWorkEnv Report" as subject and your last response as body

2. Run the send_mail script:

```bash
cd $(pwd)/tool_scripts/send_mail && uv run python send_mail.py --subject "<SUBJECT>" --body "<BODY>"
```

3. Report success or failure to the user

## Configuration

Mailgun credentials are stored in:
`tool_scripts/send_mail/config.json`

## Notes

- Uses Mailgun API for email delivery
- Supports HTML body with `--html` flag
- Can read body from file with `--file` flag
- Recipients configured in config.json, override with `--to`
