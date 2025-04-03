import json, os, sys, re
import asyncio
from telethon import TelegramClient, events, Button
from telethon.sessions import StringSession
from telethon.tl.functions.contacts import ImportContactsRequest, GetContactsRequest
from telethon.tl.types import InputPhoneContact
from telethon.tl.functions.messages import ImportChatInviteRequest
from telethon.tl.functions.channels import JoinChannelRequest
from telethon.tl.functions.messages import AddChatUserRequest
from telethon.errors import UserAlreadyParticipantError, FloodWaitError
from telethon.tl.types import InputPhoneContact


API_ID = 24347380 
API_HASH = "1ad5dea4dfdddfed44df611dcd0d1736"
BOT_TOKEN = "توكن البوت"
DB_FILE = "users.json" 

ADMIN_USERNAME = "هنا حط يوزرك"


def load_data(file_path):
    try:
        with open(file_path, "r") as f:
            return json.load(f)
    except FileNotFoundError:
        return {}

def save_data(file_path, data):
    with open(file_path, "w") as f:
        json.dump(data, f, indent=4)

users = load_data(DB_FILE)
bot = TelegramClient("bot", API_ID, API_HASH).start(bot_token=BOT_TOKEN)


@bot.on(events.NewMessage(pattern='تحديث'))
async def restart(event):
    if event.sender.username != ADMIN_USERNAME.strip('@'):
        return
    await event.reply("تم اعادة تشغيل البوت")
    os.execv(sys.executable, ['python3'] + sys.argv)

@bot.on(events.CallbackQuery(data=b'login'))
async def add_account(event):
    user_id = str(event.sender_id)
    if not users.get(user_id, {}).get('is_vip', False):
        return
    await event.edit("**⌔︙ قم بإرسال كود التيرمكس **")
    
    @bot.on(events.NewMessage(from_users=event.sender_id))
    async def receive_session(evt):
        if evt.sender_id != event.sender_id:
            return
        session_string = evt.text.strip()
        if not session_string:
            await evt.reply("**⌔︙ الرجاء ارسال كود تيرمكس صالح**")
            return
        try:
            temp_client = TelegramClient(StringSession(session_string), API_ID, API_HASH)
            await temp_client.connect()

            if not await temp_client.is_user_authorized():
                await evt.reply("**⌔︙ كود التيرمكس غير صالح الرجاء إرسال كود صالح**")
                await temp_client.disconnect()
                return
            buttons = [[Button.inline('اضف جلسة اخرى', b'login')]]
            if 'sessions' not in users.get(user_id, {}):
                users[user_id] = {'sessions': [], 'is_vip': True}
            if session_string not in users[user_id]['sessions']:
                users[user_id]['sessions'].append(session_string)
                save_data(DB_FILE, users)
                await evt.reply("**⌔︙ تم إضافة كود التيرمكس بنجاح أستمتع بالبوت 🤍**", buttons=buttons)
            else:
                await evt.reply("**⌔︙ هذا الكود مضاف مسبقًا**", buttons=buttons)

        except Exception as e:
            print(f"خطأ أثناء التحقق من الجلسة: {e}")
            await evt.reply("**⌔︙ كود التيرمكس غير صالح الرجاء إرسال كود صالح**")
        finally:
            bot.remove_event_handler(receive_session, events.NewMessage)

async def wait_for_response(client, user_id, timeout=30):
    future = asyncio.Future()

    async def response_handler(event):
        if event.sender_id == int(user_id):
            future.set_result(event)

    client.add_event_handler(response_handler, events.NewMessage)

    try:
        return await asyncio.wait_for(future, timeout=timeout)
    except asyncio.TimeoutError:
        return None
    finally:
        client.remove_event_handler(response_handler, events.NewMessage)

def parse_vcard(content):
    """Extract phone numbers from vCard content."""
    phone_numbers = []
    lines = content.splitlines()
    for line in lines:
        if line.startswith("TEL:"):
            number = line.split("TEL:")[1].strip()
            if number.startswith("+"):
                phone_numbers.append(number)
    return phone_numbers

@bot.on(events.CallbackQuery(data=b'add_contact'))
async def add_contact(event):
    user_id = str(event.sender_id)
    print(f"بدء دالة add_contact للمستخدم {user_id}")
    sys.stdout.flush()
    if not users.get(user_id, {}).get('is_vip', False):
        print("المستخدم ليس VIP")
        sys.stdout.flush()
        return
    if 'sessions' not in users[user_id] or not users[user_id]['sessions']:
        print("لا توجد جلسات للمستخدم")
        sys.stdout.flush()
        await event.edit("**⌔︙ ماعندك حسابات ضايفهن استخدم زر اضف حساب**")
        return

    print("طلب إدخال رقم أو ملف vCard")
    sys.stdout.flush()
    await event.edit("**⌔︙ ارسل رقم هاتف واحد (مثل +964xxxxxxxxxx) أو ملف vCard يحتوي على أرقام\n عندك ٣٠ ثانية**")
    response_event = await wait_for_response(bot, user_id, timeout=30)
    if not response_event:
        print("انتهى الوقت، لم يتم إدخال شيء")
        sys.stdout.flush()
        await event.respond("**⌔︙ انتهى الوقت، يرجى إعادة المحاولة**")
        return

    phone_numbers = []
    
    if response_event.document:
        print("تم إرسال ملف")
        sys.stdout.flush()
        if response_event.document.mime_type not in ['text/vcard', 'text/x-vcard', 'text/plain', 'application/octet-stream']:
            print("نوع الملف غير مدعوم")
            sys.stdout.flush()
            await event.respond("**⌔︙ يرجى إرسال ملف vCard (.vcf) أو نصي (.txt) فقط**")
            return
            
        file_path = await response_event.download_media()
        print(f"تم تحميل الملف: {file_path}")
        sys.stdout.flush()
        try:
            with open(file_path, 'r', encoding='utf-8') as f:
                content = f.read()
                print(f"محتوى الملف: {content[:100]}...")
                sys.stdout.flush()
                if "BEGIN:VCARD" in content:
                    phone_numbers = parse_vcard(content)
                    print(f"تم استخراج الأرقام من vCard: {phone_numbers}")
                    sys.stdout.flush()
                else:
                    phone_numbers = [line.strip() for line in content.splitlines() if re.match(r'^\+\d{9,15}$', line.strip())]
                    print(f"تم استخراج الأرقام من نص: {phone_numbers}")
                    sys.stdout.flush()
            os.remove(file_path)
            print(f"تم حذف الملف: {file_path}")
            sys.stdout.flush()
        except Exception as e:
            print(f"خطأ أثناء قراءة الملف: {str(e)}")
            sys.stdout.flush()
            await event.respond(f"**⌔︙ حدث خطأ أثناء قراءة الملف: {str(e)}**")
            os.remove(file_path)
            return
    else:
        phone_number = response_event.text.strip()
        print(f"تم إرسال رقم: {phone_number}")
        sys.stdout.flush()
        if not re.match(r'^\+\d{9,15}$', phone_number):
            print("الرقم غير صالح (يجب أن يحتوي على + تليها 9-15 رقمًا)")
            sys.stdout.flush()
            await event.respond("**⌔︙ يرجى إرسال الرقم بالصيغة الصحيحة مثل +964xxxxxxxxxx (9-15 رقمًا)**")
            return
        phone_numbers = [phone_number]
        print(f"الأرقام المستلمة: {phone_numbers}")
        sys.stdout.flush()

    if not phone_numbers:
        print("لم يتم العثور على أرقام صالحة")
        sys.stdout.flush()
        await event.respond("**⌔︙ لم يتم العثور على أرقام صالحة**")
        return
    phone_numbers = [num.strip() for num in phone_numbers if re.match(r'^\+\d{9,15}$', num.strip())]
    print(f"الأرقام بعد التنظيف: {phone_numbers}")
    sys.stdout.flush()

    print(f"جاري إضافة {len(phone_numbers)} جهة اتصال...")
    sys.stdout.flush()
    await event.respond(f"**⌔︙ جاري إضافة {len(phone_numbers)} جهة اتصال لجميع الحسابات...**")
    
    total_successful_adds = 0
    failed_numbers = []
    contact_info = ""

    for idx, session_string in enumerate(users[user_id]['sessions']):
        print(f"معالجة الجلسة رقم {idx + 1}")
        sys.stdout.flush()
        temp_client = TelegramClient(StringSession(session_string), API_ID, API_HASH)
        
        max_retries = 3
        for attempt in range(max_retries):
            try:
                await temp_client.connect()
                if not await temp_client.is_user_authorized():
                    print(f"الجلسة {idx + 1} غير صالحة")
                    sys.stdout.flush()
                    await event.respond(f"**⌔︙ الجلسة {idx + 1} غير صالحة أو منتهية الصلاحية، أضف جلسة جديدة**")
                    failed_numbers.extend(phone_numbers)
                    await temp_client.disconnect()
                    break

                me = await temp_client.get_me()
                print(f"تم الاتصال بالحساب: {me.id} ({me.phone or me.username or 'بدون اسم'})")
                sys.stdout.flush()

                contacts_result = await temp_client(GetContactsRequest(0))
                sys.stdout.flush()
                if len(contacts_result.users) >= 1000:
                    await event.respond(f"**⌔︙ الحساب {me.phone or me.username}: وصلت إلى الحد الأقصى لجهات الاتصال (1000)**")
                    failed_numbers.extend(phone_numbers)
                    await temp_client.disconnect()
                    break
                batch_size = 15
                existing_contacts = {contact.phone for contact in contacts_result.users if contact.phone}
                for i in range(0, len(phone_numbers), batch_size):
                    batch = phone_numbers[i:i + batch_size]
                    contacts = [InputPhoneContact(client_id=0, phone=num, first_name="joker", last_name="") for num in batch]
                    try:
                        result = await temp_client(ImportContactsRequest(contacts))
                        total_successful_adds += len(result.imported)

                        if result.users:
                            for user in result.users:
                                info = f"ID: {user.id}, Username: @{user.username or 'غير متوفر'}, Phone: {user.phone}"
                                contact_info += f"**⌔︙ تمت إضافة: {info}**\n"
                            imported_numbers = {user.phone for user in result.users}
                            failed_in_session = [num for num in batch if num not in imported_numbers]
                        else:
                            failed_in_session = batch

                        for num in failed_in_session:
                            if num in existing_contacts:
                                contact_info += f"**⌔︙ الرقم {num} موجود مسبقًا في جهات الاتصال**\n"
                                total_successful_adds += 1
                            else:
                                try:
                                    entity = await temp_client.get_entity(num)
                                    contact_info += f"**⌔︙ الرقم {num} مسجل في تيليجرام ولكن لم يتم إضافته (قد يكون مخفيًا)**\n"
                                    total_successful_adds += 1
                                except ValueError as ve:
                                    print(f"الرقم {num} غير مسجل في تيليجرام، يتم تجاهله ({str(ve)})")
                                    sys.stdout.flush()
                                except Exception as e:
                                    contact_info += f"**⌔︙ فشل في التحقق من {num}: {str(e)}**\n"
                                    failed_numbers.append(num)

                        await asyncio.sleep(2)

                    except FloodWaitError as e:
                        await event.respond(f"**⌔︙ تجاوزت الحد الأقصى للحساب {me.phone or me.username}، انتظر {e.seconds} ثانية**")
                        failed_numbers.extend(batch)
                    except Exception as e:
                        await event.respond(f"**⌔︙ فشل في إضافة دفعة للحساب {me.phone or me.username}: {str(e)}**")
                        failed_numbers.extend(batch)

                await temp_client.disconnect()
                break

            except Exception as e:
                print(f"المحاولة {attempt + 1} للاتصال فشلت: {str(e)}")
                sys.stdout.flush()
                if attempt + 1 == max_retries:
                    await event.respond(f"**⌔︙ فشل الاتصال بالجلسة {idx + 1} بعد {max_retries} محاولات: {str(e)}**")
                    failed_numbers.extend(phone_numbers)
                else:
                    await asyncio.sleep(5)
                continue

    failed_numbers = list(set(failed_numbers))
    success_message = f"**⌔︙ تمت إضافة {total_successful_adds} جهة اتصال بنجاح ✓**\n"
    if contact_info:
        success_message += contact_info
    if failed_numbers:
        success_message += f"**⌔︙ فشل في إضافة {len(failed_numbers)} رقم/أرقام**\n"
        success_message += f"**⌔︙ الأرقام الفاشلة: {', '.join(failed_numbers[:10])}{'...' if len(failed_numbers) > 10 else ''}**"
        if total_successful_adds == 0 and failed_numbers:
            success_message += "\n**⌔︙ تحقق من الجلسة، قد تكون غير صالحة أو تواجه مشكلة**"
    await event.respond(success_message)
@bot.on(events.CallbackQuery(data=b'add_to_group'))
async def add_to_group(event):
    user_id = str(event.sender_id)
    if not users.get(user_id, {}).get('is_vip', False):
        return
    if 'sessions' not in users[user_id] or not users[user_id]['sessions']:
        await event.edit("**⌔︙ ماعندك حسابات ضايفهن استخدم زر اضف حساب**")
        return

    await event.edit("**⌔︙ ارسل رابط المجموعة (عام أو خاص) لإضافة الحسابات وجهات الاتصال إليها\n عندك ٣٠ ثانية**")
    response_event = await wait_for_response(bot, user_id, timeout=30)
    if not response_event:
        await event.respond("**⌔︙ انتهى الوقت، يرجى إعادة المحاولة**")
        return

    group_link = response_event.text.strip()
    if not (group_link.startswith('https://t.me/') or group_link.startswith('t.me/')):
        await event.respond("**⌔︙ يرجى إرسال رابط مجموعة صالح (مثل https://t.me/GroupName أو https://t.me/+xxxx)**")
        return

    if '+' in group_link:
        invite_hash = group_link.split('+')[-1].strip()
        group_username = None
    else:
        group_username = group_link.split('t.me/')[-1].strip()
        invite_hash = None

    await event.respond("**⌔︙ جاري إضافة الحسابات إلى المجموعة وجهات الاتصال...**")
    joined_sessions = []
    group_entity = None
    
    # إضافة الحسابات إلى المجموعة
    for idx, session_string in enumerate(users[user_id]['sessions']):
        try:
            temp_client = TelegramClient(StringSession(session_string), API_ID, API_HASH)
            await temp_client.connect()
            me = await temp_client.get_me()
            
            if invite_hash:
                try:
                    updates = await temp_client(ImportChatInviteRequest(hash=invite_hash))
                    group_entity = updates.chats[0]
                except UserAlreadyParticipantError:
                    group_entity = await temp_client.get_entity(f"t.me/+{invite_hash}")
            else:
                try:
                    group_entity = await temp_client.get_entity(group_username)
                    await temp_client(JoinChannelRequest(group_entity))
                except UserAlreadyParticipantError:
                    group_entity = await temp_client.get_entity(group_username)
            
            joined_sessions.append(temp_client)
            await event.respond(f"**⌔︙ تم إضافة حساب بنجاح إلى المجموعة**")
            await asyncio.sleep(2)
            
        except Exception as e:
            await event.respond(f"**⌔︙ فشل في إضافة حساب: {str(e)}**")
            continue

    if not joined_sessions:
        await event.respond("**⌔︙ لم يتم إضافة أي حساب إلى المجموعة، توقف العملية**")
        return

    total_successful_adds = 0
    failed_contacts = 0
    batch_size = 10 
    for idx, temp_client in enumerate(joined_sessions):
        try:
            contacts_result = await temp_client(GetContactsRequest(0))
            contacts = contacts_result.users
            if not contacts:
                await event.respond("**⌔︙ لا توجد جهات اتصال صالحة في هذا الحساب**")
                continue

            await event.respond(f"**⌔︙ يتم محاولة إضافة {len(contacts)} جهة اتصال من هذا الحساب على شكل دفعات...**")
           
            for i in range(0, len(contacts), batch_size):
                batch = contacts[i:i + batch_size]
                batch_success = 0
                batch_failed = 0
                
                for contact in batch:
                    try:
                        await temp_client(AddChatUserRequest(
                            chat_id=group_entity.id,
                            user_id=contact.id,
                            fwd_limit=10
                        ))
                        batch_success += 1
                        total_successful_adds += 1
                    except FloodWaitError as e:
                        batch_failed += 1
                        failed_contacts += 1
                        await event.respond(f"**⌔︙ تجاوزت الحد الأقصى، انتظر {e.seconds} ثانية**")
                        break
                    except Exception as e:
                        batch_failed += 1
                        failed_contacts += 1
              
                await asyncio.sleep(5)

            await temp_client.disconnect()
            await asyncio.sleep(2)
            
        except Exception as e:
            failed_contacts += len(contacts) if 'contacts' in locals() else 0
            await event.respond(f"**⌔︙ فشل في معالجة الحساب: {str(e)}**")
            continue

    success_message = f"**⌔︙ تمت إضافة {total_successful_adds} جهة اتصال إلى المجموعة بنجاح ✓**\n"
    if failed_contacts:
        success_message += f"**⌔︙ فشل في إضافة {failed_contacts} جهة/جهات اتصال**"
    await event.respond(success_message)

@bot.on(events.CallbackQuery(data=b'view_accounts'))
async def view_accounts(event):
    user_id = str(event.sender_id)
    if not users.get(user_id, {}).get('is_vip', False):
        return
    if 'sessions' not in users[user_id] or not users[user_id]['sessions']:
        await event.edit("**⌔︙ ليس لديك جلسات مضافة بعد، استخدم زر 'اضف حساب'**")
        return

    await show_account_page(event, user_id, 0)

async def show_account_page(event, user_id, page):
    sessions = users[user_id]['sessions']
    total_pages = len(sessions)
    
    if page < 0 or page >= total_pages:
        return

    session_string = sessions[page]
    temp_client = TelegramClient(StringSession(session_string), API_ID, API_HASH)
    await temp_client.connect()
    me = await temp_client.get_me()
    await temp_client.disconnect()
    
    account_name = me.phone or me.username or "حساب بدون اسم"
    
    buttons = [
        [Button.inline(f"الحساب: {account_name}", b'noop'), Button.inline("🗑️ حذف", f'delete_{page}'.encode())]
    ]
    
    nav_buttons = []
    if page > 0:
        nav_buttons.append(Button.inline("⬅️ السابق", f'page_{page-1}'.encode()))
    if page < total_pages - 1:
        nav_buttons.append(Button.inline("التالي ➡️", f'page_{page+1}'.encode()))
    if nav_buttons:
        buttons.append(nav_buttons)
    
    await event.edit(
        f"**⌔︙ جلساتك ({page + 1}/{total_pages}):**\n"
        f"**الحساب الحالي:** {account_name}",
        buttons=buttons
    )

@bot.on(events.CallbackQuery(pattern=r'page_(\d+)'))
async def handle_page(event):
    page = int(re.search(r'page_(\d+)', event.data.decode()).group(1))
    user_id = str(event.sender_id)
    await show_account_page(event, user_id, page)

@bot.on(events.CallbackQuery(pattern=r'delete_(\d+)'))
async def delete_session(event):
    match = re.search(r'delete_(\d+)', event.data.decode())
    if not match:
        await event.edit("**⌔︙ خطأ في معالجة الطلب**")
        return
    page = int(match.group(1))
    user_id = str(event.sender_id)
    
    if 'sessions' not in users[user_id] or page >= len(users[user_id]['sessions']):
        await event.edit("**⌔︙ الجلسة غير موجودة**")
        return
    
    deleted_session = users[user_id]['sessions'].pop(page)
    save_data(DB_FILE, users)
    
    sessions_left = len(users[user_id]['sessions'])
    if sessions_left == 0:
        await event.edit("**⌔︙ تم حذف الجلسة بنجاح، لم يعد لديك جلسات**")
        return
    
    new_page = min(page, sessions_left - 1)
    await show_account_page(event, user_id, new_page)
    await event.respond("**⌔︙ تم حذف الجلسة بنجاح ✓**")

@bot.on(events.CallbackQuery(data=b'noop'))
async def no_op(event):
    pass

@bot.on(events.NewMessage(pattern="/start"))
async def start(event):
    user_id = str(event.sender_id)
    if user_id not in users:
        users[user_id] = {'sessions': [], 'is_vip': False}
        save_data(DB_FILE, users)
    if not users[user_id]['is_vip']:
        await event.reply(
            "**⌔︙ للأسف، أنت بحاجة للاشتراك من المطور للحصول على عضوية VIP**\n"
            "**⌔︙ اضغط على الزر أدناه للتواصل مع المطور**",
            buttons=[
                [Button.url("المطور", 'https://t.me/F_O_1')]
            ]
        )
        return
    await event.reply(
        "**⌔︙ هلا بيك، اختر من الأزرار التالية:**",
        buttons=[
            [Button.inline("اضف حساب", b'login'), Button.inline("عرض الحسابات", b'view_accounts')],
            [Button.inline("إضافة جهة اتصال", b'add_contact')],
            [Button.inline("اضافة جهات الى المجموعة", b'add_to_group')]
        ]
    )

@bot.on(events.CallbackQuery(data=b'enable_vip'))
async def enable_vip(event):
    if event.sender.username != ADMIN_USERNAME.strip('@'):
        return
    await event.edit("**⌔︙ ارسل يوزر المستخدم او الايدي حتى تفعل اله عضوية VIP**")
    user_id = str(event.sender_id)
    response = await wait_for_response(bot, user_id, timeout=60)

    if not response:
        await event.respond("**⌔︙ انتهى الوقت، لم يتم استلام أي استجابة**")
        return

    target = response.text.strip()
    if target.isdigit(): 
        user_id = target
    else: 
        try:
            user = await bot.get_entity(target)
            user_id = str(user.id)
        except Exception:
            await event.respond("**⌔︙ مالگيت هذا المستخدم تأكد من اليوزر او الايدي**")
            return

    if user_id not in users:
        await event.respond("**⌔︙ هذا المستخدم ليس مسجلاً في قاعدة البيانات**")
        return

    users[user_id]['is_vip'] = True
    save_data(DB_FILE, users)
    await event.respond(f"**⌔︙ تم تفعيل العضوية للمستخدم {target} بنجاح**")

@bot.on(events.CallbackQuery(data=b'disable_vip'))
async def disable_vip(event):
    if event.sender.username != ADMIN_USERNAME.strip('@'):
        return
    await event.edit("**⌔︙ارسل يوزر المستخدم او الايدي حتى تعطل اله عضوية VIP**")
    user_id = str(event.sender_id)
    response = await wait_for_response(bot, user_id, timeout=60)

    if not response:
        await event.respond("**⌔︙ انتهى الوقت، لم يتم استلام أي استجابة**")
        return

    target = response.text.strip()
    if target.isdigit(): 
        user_id = target
    else:
        try:
            user = await bot.get_entity(target)
            user_id = str(user.id)
        except Exception:
            await event.respond("**⌔︙ مالگيت هذا المستخدم تأكد من اليوزر او الايدي**")
            return

    if user_id not in users:
        await event.respond("**⌔︙ هذا المستخدم ليس مسجلاً في قاعدة البيانات**")
        return

    users[user_id]['is_vip'] = False
    save_data(DB_FILE, users)
    await event.respond(f"**⌔︙ تم تعطيل العضوية للمستخدم {target} بنجاح**")

@bot.on(events.NewMessage(pattern='/admin'))
async def admin(event):
    if event.sender.username != ADMIN_USERNAME.strip('@'):
        return
    await event.reply(
        "**⌔︙ حياك الله يالادمن هاي لوحة لتفعيل وتعطيل العضوية من المستخدمين**",
        buttons=[
            [Button.inline("تفعيل VIP", b'enable_vip')],
            [Button.inline("تعطيل VIP", b'disable_vip')]
        ]
    )

print("⌔︙ البوت يعمل الآن...")

bot.run_until_disconnected()