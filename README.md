# Telegram to Discord Signals Forwarder

A TypeScript application that forwards trading signals from Telegram channels to Discord using MTProto API and webhooks.

## Features

- **MTProto Integration**: Direct connection to Telegram using official MTProto API (no bot limitations)
- **Message Processing**: Intelligent parsing of signals and results with formatting preservation
- **Content Filtering**: Automatic removal of promotional content (Deep scan by Z99Bot)
- **Link Replacement**: Converts mevx.io chart links to dexscreener.com
- **Reply Handling**: Extracts token information from original signals when processing results
- **Discord Formatting**: Preserves text formatting (bold, italic, code, links) when forwarding to Discord
- **Auto Session Management**: Handles Telegram authentication automatically

## Tech Stack

- **TypeScript** - Type-safe development
- **mtcute** - Modern MTProto library for Telegram
- **Node.js** - Runtime environment
- **Discord Webhooks** - For sending messages to Discord

## Prerequisites

- Node.js 16+ 
- Telegram API credentials (api_id, api_hash)
- Discord webhook URL
- Access to target Telegram channel

## Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd tg-ds-signals
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Build the project**
   ```bash
   npm run build
   ```

## Configuration

1. **Get Telegram API credentials**
   - Visit [my.telegram.org](https://my.telegram.org)
   - Create an application to get `api_id` and `api_hash`

2. **Set up Discord webhook**
   - In your Discord server's channel, go to Channel settings → Integrations → Webhooks
   - Create a new webhook and copy the URL

3. **Configure environment variables**
   ```bash
   cp .env.example .env
   ```
   
   Edit `.env` with your credentials:
   ```env
   TELEGRAM_API_ID=123456
   TELEGRAM_API_HASH=your_api_hash_here
   TELEGRAM_CHANNEL_USERNAME=your_channel_username
   DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/...
   ```

## Usage

### Development
```bash
npm run dev
```

### Production
```bash
npm start
```

### First Run Authentication
On first run, the application will prompt for Telegram authentication:

1. **Phone Number**: Enter your phone number (e.g., +74441112233)
2. **Confirmation Code**: Enter the code sent to your Telegram app
3. **2FA Password**: Enter your 2FA password if enabled

Example authentication flow:
```
phone > *phone number*
The confirmation code has been sent via app.
code > *confirmation code*
2fa password > *your 2fa password*
```

After successful authentication, session data is saved and future runs won't require re-authentication.

## How It Works

### Message Processing Pipeline

1. **Telegram Connection**: Connects to Telegram using MTProto API
2. **Message Filtering**: Monitors specified channel for new messages
3. **Content Cleaning**: 
   - Removes "Deep scan by Z99Bot" promotional lines
   - Replaces mevx.io links with dexscreener.com links
4. **Signal Parsing**: Identifies signals vs results using MessageParser
5. **Entity Processing**: Preserves formatting (bold, italic, code blocks, links)
6. **Reply Handling**: For results, extracts first two lines from original signal
7. **Discord Forwarding**: Sends formatted message to Discord via webhook

### Signal Types

**Signals**: New trading opportunities
```
💊 BPftdcA4pxhwuLrBBBp5s4dJws
┌ethereham (ethereham)  
├USD: $0.0000862
├MC: $86.2K
└Chart: https://dexscreener.com/solana/...
```

**Results**: Updates on existing signals
```
💊 BPftdcA4pxhwuLrBBBp5s4dJwsUiZuVAi3Z83HHTpump
ethereham (ethereham)

💹 2.9x 86.2K to 249.7K
```

## Project Structure

```
src/
├── config/           # Configuration management
├── services/
│   ├── TelegramService.ts    # MTProto client & message handling
│   ├── DiscordService.ts     # Discord webhook integration  
│   ├── MessageParser.ts      # Signal/result classification
│   └── EntityParser.ts       # Text formatting preservation
├── types/            # TypeScript type definitions
└── index.ts          # Application entry point
```

## Key Classes

- **TelegramService**: Manages MTProto connection and message processing
- **DiscordService**: Handles Discord webhook formatting and sending
- **MessageParser**: Classifies messages as signals or results
- **EntityParser**: Preserves Telegram text formatting for Discord

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `TELEGRAM_API_ID` | Telegram API ID | `123456` |
| `TELEGRAM_API_HASH` | Telegram API Hash | `abcdef123456` |
| `TELEGRAM_CHANNEL_USERNAME` | Target channel username (without @) | `your_signals_channel` |
| `DISCORD_WEBHOOK_URL` | Discord webhook URL | `https://discord.com/api/webhooks/...` |

## Session Management

- Session data is stored in `client.session*` files
- These files are automatically generated and managed by mtcute
- **Important**: Session files contain authentication data and are added to `.gitignore`
- Each deployment/developer needs their own session files

## Troubleshooting

### Common Issues

**"Failed to resolve channel"**
- Ensure the channel username is correct (without @)
- Verify you have access to the channel
- Check if the channel is public or you're a member

**"Authentication failed"**
- Verify API credentials in `.env`
- Delete session files and restart for fresh authentication

**"No entities found"**
- Check if message processing is correctly cleaning content
- Verify entity adjustment logic in logs

### Logging

The application provides detailed logging:
- 🚀 Startup and connection status  
- 📨 Message reception and processing
- 🔧 Entity adjustment operations
- ✅ Successful forwards to Discord
- ❌ Error conditions

## Security Notes

- **Never commit session files** - they contain authentication tokens
- **Protect API credentials** - use environment variables
- **Monitor webhook usage** - Discord has rate limits
- **Channel access** - ensure proper permissions


## Parsing
Parsing format is based on signal example:
```
💊 4bqwJeEKy9D7T5CLaz15kKXN6TowH2u6kBKddzW9fBLV
┌MARUTA (MARUTA) 
├USD:       $0.0001068
├MC:        $106.8K
├Vol:       $153.6K
├Seen:      29m ago
├Dex:       Meteora
├Dex Paid:  🔴
├Holder:    Top 10: 🟡 25%
└TH:        30.7 |7.8 |2.9 |2.9 |2.3 

🔎 Deep scan by Z99Bot
Enjoy Fastest Trading on Bitfoot • Over 50% Faster than other Bots

📈 Chart: https://mevx.io/solana/4bqwJeEKy9D7T5CLaz15kKXN6TowH2u6kBKddzW9fBLV?ref=pvggbyhMPGy9
```

or on result example:
```
*signal message reply*

💹 3.0x 130.0K to 383.9K
```