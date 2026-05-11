# TODO

## Bugs

### Gmail header values are not decoded properly (UTF-8 / RFC 2047)

**Where**: `src/tools/gmail.ts` — every code path that extracts header values from `payload.headers` (e.g. `queryEmails`, `getEmailById`, `bulkGetEmails`, `listDrafts`, `extractEmailHeaders`).

**Symptom**: subjects (and other header values) containing non-ASCII characters come back mangled in tool output. Example observed in `gmail_list_drafts`:

```
"subject": "Test gmail_list_drafts â€\" Ã  supprimer"
```

Expected: `Test gmail_list_drafts — à supprimer`.

The `â€"` / `Ã ` pattern is classic UTF-8 bytes interpreted as Windows-1252 (mojibake). It's not RFC 2047 encoded-words (those would look like `=?UTF-8?Q?...?=`).

**Investigation pending**:
- Is Gmail API returning raw RFC 2047 encoded-words that we'd need to decode with a lib like `libmime`?
- Or are the bytes UTF-8 already but decoded as Latin-1 somewhere in the chain (`googleapis` client / `JSON.stringify` / MCP transport)?
- Reproduce by creating a draft with `—` and `à` in the subject, then inspecting both the raw `users.messages.get` response and what `gmail_list_drafts` returns.

**Fix direction**: once located, either decode encoded-words explicitly (`libmime.decodeWords(value)`) or fix the charset handling at the byte boundary. Should be upstreamable.
