## news-aggregator

> This is a Node.js news aggregator that polls multiple news sources every 30 minutes and stores unique items in an SQLite database.

# News Aggregator Project

This is a Node.js news aggregator that polls multiple news sources every 30 minutes and stores unique items in an SQLite database.

## Architecture

- **Backend**: Express.js server with SQLite database
- **Fetchers**: RSS parser, web scraper, and Twitter API client
- **Scheduling**: Node-cron for automated polling every 30 minutes
- **Deduplication**: Unique constraint on (source_name, external_id)
- **API**: REST endpoint at `/api/latest` returning latest 200 items
- **Frontend**: Simple HTML/CSS/JS web interface

## Key Features

- Multi-source support (RSS, scraping, Twitter)
- Automatic deduplication
- Raw JSON storage with timestamps
- Environment-based configuration
- Docker support
- Responsive web interface

## Development Notes

- Database is automatically initialized on first run
- Twitter API requires BEARER_TOKEN in .env file
- All fetchers handle errors gracefully to prevent polling failures
- Web interface auto-refreshes every 5 minutes
- Cron job runs every 30 minutes for new content

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DreadTheMystery)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DreadTheMystery)
<!-- tomevault:4.0:gemini_md:2026-04-08 -->
