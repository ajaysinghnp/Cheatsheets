# Cheatsheets

Commands, short procedures and running notes: Linux, Git, security tooling, smart-home flashing and project setup.

The sheets are read straight from this repository by the portfolio site through the GitHub API, so pushing to the configured branch updates the site without a rebuild.

## Sheets

- Linux: [Users and groups](./sheets/linux/managing-users-and-groups.md), [Network configuration](./sheets/linux/managing-network-and-configurations.md)
- Git: [Git workflows](./sheets/git/@git-workflows-cheatsheet/article.md)
- Search engines: [Devices and internet](./sheets/search-engines/device-and-internet.md), [Domains and DNS](./sheets/search-engines/domain-subdomain-dns.md), [Files and metadata](./sheets/search-engines/file-and-metadata.md), [Smart devices and IoT](./sheets/search-engines/smart-devices-iot.md), [Vulnerabilities and exploits](./sheets/search-engines/vulnerability-and-exploit.md)
- Smart home: [Aubess mini smart switch](./sheets/smart-home/@aubess-mini-smart-switch/article.md), [DIY smart switch](./sheets/smart-home/@diy-smart-switch/article.md), [Generic BK7231N board](./sheets/smart-home/boards/generic-bk7231n-qfn32-tuya.md)
- Starters: [Next.js starter guide](./sheets/starters/nextjs-starter-guide.md)

## Layout

Everything lives under `sheets/`. Names are lowercase kebab-case.

- A folder is a category. Categories can nest: `sheets/smart-home/boards/`.
- A Markdown file directly inside a category is a sheet, and its filename is the slug.
- A sheet that needs images or other files gets its own folder starting with `@` (for example `@diy-smart-switch/`). It holds exactly one Markdown file, preferably `article.md`, with the resources next to it, referenced relatively (`./photo.jpg`).
- A category folder starting with `_` is hidden from listings.
- Write GitHub-flavored Markdown and tag fenced code blocks with a language.

## Frontmatter

| Field | Required | Notes |
| --- | --- | --- |
| `title` | yes | |
| `excerpt` | yes | one-line summary shown on cards |
| `author` | yes | author id from the blog repo's `authors/` folder |
| `tags` | yes | list |
| `created_at` | yes | ISO date and time |
| `updated_at` | no | set when you meaningfully revise a sheet |
| `published` | no | `false` hides a draft |
| `slug` | no | legacy; must match the filename if present |
| `type`, `standard`, `board` | no | device details for smart-home sheets |

## Commands

    pnpm install
    pnpm new <category> "Sheet title" [--resource]
    pnpm check        # validate frontmatter, structure and images, then check formatting
    pnpm format
