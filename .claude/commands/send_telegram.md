# 發送 Telegram

Send a message to Telegram using the configured bot.

## Usage

```
/send_telegram <message>
/send_telegram
```

## Examples

```
/send_telegram TSLA 分析報告已完成！
/send_telegram   (sends last Claude response to Telegram)
```

## Instructions

When this command is invoked:

1. **Determine the message content:**
   - If the user provided a message argument, use that as the message
   - If NO message argument is provided (empty), use **your last response in the conversation** as the message

2. Run the send_telegram script:

```bash
cd $(pwd)/tool_scripts/send_telegram && uv run python send_message.py --message "<USER_MESSAGE>"
```

3. Report success or failure to the user

## Configuration

The bot credentials and chat_id are stored in:
`tool_scripts/send_telegram/config.json`

## Sending a File as Document

To send a file as a document attachment (not as text content), use `--send-file`:

```bash
cd $(pwd)/tool_scripts/send_telegram && uv run python send_message.py --send-file /path/to/file.txt --caption "Optional caption"
```

## Notes

- **Default behavior**: If no message is specified, sends Claude's last response in the conversation
- `--message` / `--file`: sends text content as a chat message (auto-splits if > 4096 chars)
- `--send-file`: sends the file itself as a document attachment
- The message supports HTML formatting (`<b>bold</b>`, `<i>italic</i>`, `<code>code</code>`)
- Messages are sent to the default chat_id configured in config.json
- To send to a different chat, use `--chat_id` flag directly via bash
