import discord
from discord.ext import commands
import aiohttp
import json
import asyncio

TOKEN = '봇토큰'
OLLAMA_URL = 'http://localhost:11434/api/generate'
MODEL_NAME = 'gpt-oss:120b-cloud'
TARGET_CHANNEL_ID = 봇을 작동시킬 채널 ID

intents = discord.Intents.default()
intents.message_content = True
bot = commands.Bot(command_prefix='!', intents=intents)

@bot.event
async def on_ready():
    print(f'Logged in as {bot.user.name}')

@bot.event
async def on_message(message):
    
    if message.author.bot:
        return
    
    if message.channel.id != TARGET_CHANNEL_ID:
        return

    async with message.channel.typing():
        system_prompt = (
            "너는 전문 영양사야. 사용자가 입력한 메시지에서 음식 정보를 찾아 분석해. "
            "반드시 다음 JSON 형식으로만 답해: "
            '{"calories": 0, "carbs": 0, "protein": 0, "fat": 0, "tip": ""}'
        )
        
        payload = {
            "model": MODEL_NAME,
            "prompt": f"{system_prompt}\n사용자 입력: {message.content}",
            "stream": False,
            "format": "json"
        }

        try:
            async with aiohttp.ClientSession() as session:
                async with session.post(OLLAMA_URL, json=payload) as response:
                    if response.status == 200:
                        result_json = await response.json()
                        diet_data = json.loads(result_json['response'])
                        
                        if diet_data.get('calories', 0) == 0:
                            return

                        embed = discord.Embed(
                            title=f"🍴 식단 분석 결과",
                            description=f"**입력 내용:** {message.content}",
                            color=discord.Color.green()
                        )
                        embed.add_field(name="🔥 칼로리", value=f"{diet_data['calories']} kcal", inline=True)
                        embed.add_field(name="🍞 탄수화물", value=f"{diet_data['carbs']}g", inline=True)
                        embed.add_field(name="🍗 단백질", value=f"{diet_data['protein']}g", inline=True)
                        embed.add_field(name="🥑 지방", value=f"{diet_data['fat']}g", inline=True)
                        embed.add_field(name="💡 영양사 팁", value=diet_data['tip'], inline=False)
                        embed.set_footer(text="Local LLM (Ollama) 분석 결과")
                        
                        await message.reply(embed=embed)
        except Exception as e:
            print(f"Error: {e}")

    await bot.process_commands(message)

bot.run(TOKEN)
