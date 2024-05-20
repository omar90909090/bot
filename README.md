import discord
from discord.ext import commands
import traceback
import asyncio

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix='!', intents=intents)

@bot.event
async def on_ready():
    print(f'Logged in as {bot.user}')
    await bot.change_presence(activity=discord.Game(name="Broadcasting!"))

@bot.command(name='broadcast')
@commands.has_permissions(administrator=True)
async def broadcast(ctx, *, message: str):
    def check(m):
        return m.author == ctx.author and m.channel == ctx.channel

    await ctx.send("Are you sure you want to broadcast this message to all servers? Reply with 'yes' or 'no'.")

    try:
        confirmation = await bot.wait_for('message', check=check, timeout=30.0)
    except asyncio.TimeoutError:
        await ctx.send('Broadcast cancelled: No response.')
        return

    if confirmation.content.lower() != 'yes':
        await ctx.send('Broadcast cancelled.')
        return

    successful_broadcasts = 0
    failed_broadcasts = 0

    embed = discord.Embed(title="Broadcast Message", description=message, color=0x00ff00)

    for guild in bot.guilds:
        for channel in guild.text_channels:
            try:
                if channel.permissions_for(guild.me).send_messages:
                    await channel.send(embed=embed)
                    successful_broadcasts += 1
                else:
                    print(f"Permission denied to send message in {channel.name} of {guild.name}")
                    failed_broadcasts += 1
            except Exception as e:
                print(f"Failed to send message in {channel.name} of {guild.name}: {e}")
                traceback.print_exc()
                failed_broadcasts += 1

    await ctx.send(f"Broadcast message sent to {successful_broadcasts} channels. Failed to send in {failed_broadcasts} channels.")

@broadcast.error
async def broadcast_error(ctx, error):
    if isinstance(error, commands.MissingPermissions):
        await ctx.send("You do not have permission to use this command.")
    else:
        await ctx.send("An error occurred while trying to broadcast the message.")
        traceback.print_exception(type(error), error, error.__traceback__)

bot.run(MTI0MjAzMTk0NTk2NjQyMDAxOQ.GIQkga.8deLbm9nltoK3VAnlYGKqbLJkoaJ9pzFvfzEUQ)
