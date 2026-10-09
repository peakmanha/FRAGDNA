import requests
import re
import os

API_KEY = os.environ.get("API_KEY")

def get_steam_id(steam_url):
    match = re.search(r"steamcommunity\.com/profiles/(\d+)", steam_url)
    if match:
        return match.group(1)
    return None

steam_url = input("Вставь ссылку на Steam-профиль: ")

steam_id = get_steam_id(steam_url)

if steam_id is None:
    print("Не удалось найти Steam ID в ссылке.")
    exit()

print("Steam ID найден:", steam_id)

url = "https://open.faceit.com/data/v4/players"

headers = {
    "Authorization": f"Bearer {API_KEY}"
}
params = {
    "game_player_id": steam_id,
    "game": "cs2"
}
response = requests.get(
    url, 
    headers=headers,
    params=params
)

if response.status_code == 200:
    
    player = response.json()

    print()
    print("========== FACEIT ==========")

    print("Ник:", player.get("nickname"))
    print("FACEIT ID:", player.get("player_id"))
    print("Steam ID:", player.get("steam_id_64"))

    games = player.get("games", {})
    cs2 = games.get("cs2")

    if cs2:
        print("FACEIT Level:", cs2.get("skill_level"))
        print("ELO:", cs2.get("faceit_elo"))

    print("============================")


    stats_url = f"https://open.faceit.com/data/v4/players/{player.get('player_id')}/stats/cs2"

    stats_response = requests.get(
    stats_url,
    headers=headers
    )
    
    if stats_response.status_code == 200:
        stats = stats_response.json()

        print()
        print("======= СТАТИСТИКА CS2 =======")

        lifetime = stats.get("lifetime", {})

        print("K/D:", lifetime.get("Average K/D Ratio"))
        print("Headshots:", lifetime.get("Average Headshots %"), "%")
        print("Winrate:", lifetime.get("Win Rate %"), "%")
        print("Матчи:", lifetime.get("Matches"))
        print("ADR:", lifetime.get("ADR"))
        print("Победы:", lifetime.get("Wins"))
        print("Лучший винстрик:", lifetime.get("Longest Win Streak"))
    else:
        print()
        print("Ошибка статистики:", stats_response.status_code)
        print(stats_response.text)
else:
    print()
    print("Ошибка FACEIT:", response.status_code)
    print(response.text)