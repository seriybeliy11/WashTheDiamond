# Token Trust Rating Bot

This repository contains a Telegram bot that allows users to check the trust rating of a token by providing its address. The bot uses the `aiogram` library to interact with the Telegram API and performs web scraping to retrieve trust ratings from external sources.

## Features

- **Token Trust Rating**: Users can input a token address to receive a trust rating.
- **User Interaction**: The bot guides users through the process of entering a token address and provides feedback based on the trust rating.
- **State Management**: The bot uses Finite State Machine (FSM) to manage user interactions.

## Requirements

- Python 3.8+
- `aiogram` library
- `requests` library (for web scraping)
- `beautifulsoup4` library (for HTML parsing)

## Installation

1. Clone the repository:
    ```sh
    git clone https://github.com/yourusername/token-trust-rating-bot.git
    cd token-trust-rating-bot
    ```

2. Install the required dependencies:
    ```sh
    pip install -r requirements.txt
    ```

3. Set up your Telegram bot token:
    - Create a new bot using BotFather on Telegram.
    - Replace the `TOKEN` variable in the script with your bot's token.

## Usage

1. Run the bot:
    ```sh
    python main.py
    ```

2. Start a conversation with your bot on Telegram and use the `/start` command to begin.

## Code Overview

### Imports

```python
import asyncio
import logging
import sys
from aiogram import Bot, Dispatcher, html
from aiogram.client.default import DefaultBotProperties
from aiogram.enums import ParseMode
from aiogram.filters import CommandStart
from aiogram.types import Message
from aiogram.fsm.context import FSMContext
from aiogram.fsm.state import State, StatesGroup
from aiogram.fsm.storage.memory import MemoryStorage

from get_dyor import *
```

### Bot Initialization

```python
TOKEN = ''

storage = MemoryStorage()
dp = Dispatcher(storage=storage)
```

### States Definition

```python
class Form(StatesGroup):
    waiting_for_input = State()
```

### Command Handlers

```python
@dp.message(CommandStart())
async def command_start_handler(message: Message, state: FSMContext) -> None:
    """
    This handler receives messages with `/start` command
    """
    await message.answer(f"Hello, {html.bold(message.from_user.full_name)}! Please enter token address (main contract):")
    await state.set_state(Form.waiting_for_input)
```

### Input Processing

```python
@dp.message(Form.waiting_for_input)
async def process_input(message: Message, state: FSMContext) -> None:
    user_input = message.text
    body_request_dyor = 'https://dyor.io/ru/token/' + user_input
    body_request_fakewash = 'https://www.fakewash.com/asset/' + user_input
    html_dyor = get_html_content(body_request_dyor)
    score_trust = parse_html_for_score(html)
    print(score_trust)

    result = process_user_input(score_trust)
    await message.answer(f"Trust Rating: {score_trust}%")
    if int(score_trust) > 80 and int(score_trust) <= 100:
        await message.answer("Goodluck!")

    elif int(score_trust) > 50 and int(score_trust) < 80:
        await message.answer("Not seem that's suspicious, but please, stay focused")

    else:
        await message.answer("Warning")

    await state.clear()

    return score_trust
```

### Utility Functions

```python
def process_user_input(input_text: str) -> str:
    """
    This function processes the user input and returns a result
    """
    return input_text[::-1]
```

### Main Function

```python
async def main() -> None:
    bot = Bot(token=TOKEN, default=DefaultBotProperties(parse_mode=ParseMode.HTML))
    await dp.start_polling(bot)

if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO, stream=sys.stdout)
    asyncio.run(main())
```

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Contact

For any questions or suggestions, feel free to contact [your-email@example.com](mailto:your-email@example.com).

---

Happy coding!
