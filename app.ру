from flask import Flask, request, render_template
import requests
import re
import os

app = Flask(__name__)

API_KEY = os.environ.get("API_KEY")
STEAM_API_KEY = os.environ.get("STEAM_API_KEY")


def get_steam_id(steam_url):

    # Обычная ссылка Steam /profiles/7656119...
    match = re.search(
        r"steamcommunity\.com/profiles/(\d+)",
        steam_url
    )

    if match:
        return match.group(1)

    # Красивая ссылка Steam /id/username
    match = re.search(
        r"steamcommunity\.com/id/([^/?#]+)",
        steam_url
    )

    if match:
        vanity_name = match.group(1)

        url = "https://api.steampowered.com/ISteamUser/ResolveVanityURL/v1/"

        params = {
            "key": STEAM_API_KEY,
            "vanityurl": vanity_name
        }

        response = requests.get(url, params=params)

        if response.status_code == 200:

            data = response.json()

            steam_id = data.get("response", {}).get("steamid")

            if steam_id:
                return steam_id

    return None


def calculate_dna(kd, headshots, winrate, adr):

    try:
        kd = float(kd)
        headshots = float(headshots)
        winrate = float(winrate)
        adr = float(adr)
    except (TypeError, ValueError):
        return {
            "name": "ЭНТРИ-ФРАГГЕР",
            "description": "Агрессивный стиль игры с упором на перестрелки.",
            "scores": {
                "entry": 50,
                "lurker": 50,
                "sniper": 50,
                "support": 50
            }
        }

    # -------------------------
    # ENTRY
    # -------------------------

    entry = 50

    if kd >= 1.10:
        entry += 15

    if kd >= 1.20:
        entry += 10

    if adr >= 80:
        entry += 15

    if adr >= 90:
        entry += 10

    entry = min(entry, 99)

    # -------------------------
    # SNIPER
    # -------------------------

    sniper = 50

    if headshots >= 45:
        sniper += 15

    if headshots >= 55:
        sniper += 15

    if kd >= 1.10:
        sniper += 10

    if adr >= 80:
        sniper += 10

    sniper = min(sniper, 99)

    # -------------------------
    # LURKER
    # -------------------------

    lurker = 50

    if kd >= 1.05:
        lurker += 10

    if winrate >= 50:
        lurker += 15

    if winrate >= 55:
        lurker += 10

    if adr >= 75:
        lurker += 10

    lurker = min(lurker, 99)

    # -------------------------
    # SUPPORT
    # -------------------------

    support = 50

    if winrate >= 50:
        support += 15

    if winrate >= 55:
        support += 10

    if kd >= 1.00:
        support += 10

    if adr >= 70:
        support += 10

    support = min(support, 99)

    scores = {
        "entry": entry,
        "lurker": lurker,
        "sniper": sniper,
        "support": support
    }

    # Определяем главный стиль
    
    priority = {
        "lurker": 4,
        "entry": 3,    
        "sniper": 2,
        "support": 1,
    }

    main_style = max(scores, key=lambda style: (scores[style], priority[style]))

    dna_data = {

        "entry": {
            "name": "ЭНТРИ-ФРАГГЕР",
            "description": "Агрессивный игрок, который первым вступает в перестрелки и создаёт пространство для команды."
        },

        "lurker": {
            "name": "ЛЮРКЕР",
            "description": "Игрок, который умеет играть отдельно от команды и использовать ошибки соперника."
        },

        "sniper": {
            "name": "СНАЙПЕР",
            "description": "Игрок с сильной стрельбой и хорошей эффективностью в перестрелках."
        },

        "support": {
            "name": "САППОРТ",
            "description": "Командный игрок, который помогает команде и стабильно приносит пользу."
        }

    }

    return {
        "name": dna_data[main_style]["name"],
        "description": dna_data[main_style]["description"],
        "scores": scores
    }


@app.route("/", methods=["GET", "POST"])
def home():

    result = None

    if request.method == "POST":

        steam_url = request.form.get("steam_url", "").strip()

        steam_id = get_steam_id(steam_url)
        
        if steam_id is None:

            result = {
                "error": "Не удалось найти Steam ID в ссылке."
            }

        else:

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

                # Получаем статистику CS2
                stats_url = (
                    f"https://open.faceit.com/data/v4/players/"
                    f"{player.get('player_id')}/stats/cs2"
                )

                stats_response = requests.get(
                    stats_url,
                    headers=headers
                )

                stats = {}

                if stats_response.status_code == 200:

                    stats = stats_response.json().get(
                        "lifetime",
                        {}
                    )

                games = player.get("games", {})
                cs2 = games.get("cs2")

                kd = stats.get("Average K/D Ratio")
                headshots = stats.get("Average Headshots %")
                winrate = stats.get("Win Rate %")
                adr = stats.get("ADR")

                # Рассчитываем Player DNA
                dna = calculate_dna(
                    kd,
                    headshots,
                    winrate,
                    adr
                )

                result = {

                    "nickname": player.get("nickname"),

                    "avatar": player.get("avatar"),

                    "level": (
                        cs2.get("skill_level")
                        if cs2 else None
                    ),

                    "elo": (
                        cs2.get("faceit_elo")
                        if cs2 else None
                    ),

                    "kd": kd,

                    "headshots": headshots,

                    "winrate": winrate,

                    "adr": adr,

                    "matches": stats.get("Matches"),

                    "wins": stats.get("Wins"),

                    "win_streak": stats.get(
                        "Longest Win Streak"
                    ),

                    # PLAYER DNA
                    "dna_name": dna["name"],

                    "dna_description": dna["description"],

                    "dna_scores": dna["scores"]
                }

            else:

                result = {
                    "error": f"Ошибка FACEIT: {response.status_code}"
                }

    return render_template(
        "index.html",
        result=result
    )

if __name__ == "__main__":
    app.run(debug=True)