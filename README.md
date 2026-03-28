# kyungw00k-skills

AI agent usage guides for kyungw00k CLI tools.

## Skills

| Skill | Description |
|-------|-------------|
| [juso](skills/juso/SKILL.md) | 한국 주소/우편번호 검색 |
| [upbit](skills/upbit/SKILL.md) | Upbit 암호화폐 거래소 시세/거래 |
| [stealth-wright](skills/stealth-wright/SKILL.md) | 스텔스 브라우저 자동화 |
| [apkpure](skills/apkpure/SKILL.md) | APK 다운로드 |

## Install

Add the marketplace and enable the plugin:

```bash
claude /install kyungw00k-skills
```

Or manually add to your Claude Code settings:

```json
{
  "permissions": {
    "additionalMarketplaces": ["kyungw00k/skills"]
  }
}
```

## License

MIT
