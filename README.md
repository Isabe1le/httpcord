# httpcord
A Python Discord Interaction bot API wrapper.

## `pip install --update httpcord`

From `examples/*.py`
```py
import asyncio
from enum import StrEnum
import random

from httpcord import HTTPBot, CommandResponse, Interaction
from httpcord.embed import Embed
from httpcord.enums import InteractionResponseType
from httpcord.types import AutocompleteChoice


CLIENT_ID = 0000000000000000000000
CLIENT_PUBLIC_KEY = "..."
CLIENT_TOKEN = "..."


bot = HTTPBot(
    client_id=CLIENT_ID,
    client_public_key=CLIENT_PUBLIC_KEY,
    register_commands_on_startup=True,
)

@bot.command("hello-world")
async def hello_world(interaction: Interaction) -> CommandResponse:
    return CommandResponse(
        type=InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
        content=f"hello, {interaction.user.mention}! You joined this server at <t:{int(interaction.user.joined_at.timestamp())}:F>.",
    )

@bot.command("ephemeral")
async def ephemeral(interaction: Interaction) -> CommandResponse:
    return CommandResponse(
        type=InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
        content="Hello!",
        ephemeral=True,
    )

@bot.command("guess-number")
async def guess_number(interaction: Interaction, *, guess: int, max_value: int = 10) -> CommandResponse:
    winning_number = random.randint(0, max_value)
    if guess == winning_number:
        return CommandResponse(
            type=InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
            content="Yay! You guessed the number correctly :)",
        )
    return CommandResponse(
        type=InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
        content="Aww, you got the number wrong. Better luck next time :)",
    )

@bot.command("embed")
async def embed(interaction: Interaction) -> CommandResponse:
    embed = Embed(title="Embed title")
    embed.add_field(name="Embed field title 1", value="Embed field value 1", inline=False)
    embed.add_field(name="Embed field title 2", value="Embed field value 2", inline=False)
    embed.add_field(name="Embed field title 3", value="Embed field value 3", inline=True)
    embed.add_field(name="Embed field title 4", value="Embed field value 4", inline=True)
    return CommandResponse(
        type=InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
        embeds=[embed],
    )

ANIMALS: list[str] = [
    "dog",
    "cat",
    "giraffe",
    "wolf",
    "parrot",
    "axolotl",
]

async def string_autocomplete(interaction: Interaction, current: str) -> list[AutocompleteChoice]:
    return [
        AutocompleteChoice(name=animal, value=animal)
        for animal in ANIMALS if current.lower() in animal
    ]

@bot.command(
    name="autocomplete",
    description="command with autocomplete",
    autocompletes={
        "string": string_autocomplete,
    },
)
async def autocomplete_command(interaction: Interaction, *, string: str) -> CommandResponse:
    return CommandResponse(
        type=InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
        embeds=[Embed(title=string)],
    )


@bot.command("defer-me")
async def defer_me(interaction: Interaction) -> CommandResponse:
    await interaction.defer()
    await asyncio.sleep(3)
    await interaction.followup(CommandResponse(
        type=InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
        content=f"Deferred message.",
    ))
    await interaction.followup(CommandResponse(
        type=InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
        content=f"Second follow up message.",
    ))

    # You can return another followup message, or just a PONG if you want to do nothing else.
    return CommandResponse(InteractionResponseType.PONG)

@bot.command("hello-world-deferred", auto_defer=True)
async def hello_world_long(interaction: Interaction) -> CommandResponse:
    await asyncio.sleep(3)
    await interaction.followup(CommandResponse(
        type=InteractionResponseType.DEFERRED_UPDATE_MESSAGE,
        content=f"Hello, {interaction.user.mention}!",
    ))
    return CommandResponse(InteractionResponseType.PONG)

class Fruits(StrEnum):
    apples = "apples"
    cherries = "cherries"
    kiwis = "kiwis"
    oranges = "oranges"

@bot.command("pick")
async def pick(interaction: Interaction, *, fruit: Fruits) -> CommandResponse:
    return CommandResponse(
        InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
        content=f"You picked: {fruit.value}!",
    )

bot.start(CLIENT_TOKEN)
```