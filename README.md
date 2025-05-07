# pythonth1






![image](https://github.com/user-attachments/assets/191e78f8-c637-4fcf-b8cf-72863434a495)








import smtplib
import imaplib
import email
from email.mime.text import MIMEText

# --- Bước 1: Đọc thông tin tài khoản ---
with open('email_credentials.txt', 'r') as file:
    account_info = file.readlines()
    email_account = account_info[0].strip()
    app_password = account_info[1].strip()

# --- Bước 2: Đọc nội dung email và cấu hình ---
with open('email_content.txt', 'r', encoding='utf-8') as file:
    email_content = file.read()

with open('recipient.txt', 'r') as file:
    recipient_email = file.read().strip()

with open('email_filter.txt', 'r') as file:
    filter_sender = file.read().strip()

# --- Bước 3: Gửi email ---
msg = MIMEText(email_content)
msg['Subject'] = 'Email Tự Động'
msg['From'] = email_account
msg['To'] = recipient_email

try:
    with smtplib.SMTP('smtp.gmail.com', 587) as server:
        server.starttls()
        server.login(email_account, app_password)
        server.send_message(msg)
        print(f'✅ Email đã được gửi đến {recipient_email}')
except Exception as e:
    print(f"❌ Không thể gửi email: {e}")

# --- Bước 4: Nhận email từ hộp thư đến ---
try:
    with imaplib.IMAP4_SSL('imap.gmail.com') as server:
        server.login(email_account, app_password)
        server.select('INBOX')
        _, data = server.search(None, f'FROM "{filter_sender}"')

        for num in data[0].split():
            _, msg_data = server.fetch(num, '(RFC822)')
            email_msg = email.message_from_bytes(msg_data[0][1])
            subject = email_msg['subject']
            print(f"\n📥 Tiêu đề email nhận được: {subject}")

            if email_msg.is_multipart():
                for part in email_msg.walk():
                    if part.get_content_type() == 'text/plain':
                        print("📄 Nội dung:", part.get_payload(decode=True).decode())
            else:
                print("📄 Nội dung:", email_msg.get_payload(decode=True).decode())
            break  # Chỉ in email đầu tiên
except Exception as e:
    print(f"❌ Không thể nhận email: {e}")

