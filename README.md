import time
import sqlite3
from playsound import playsound
from datetime import datetime
import tkinter as tk
from tkinter import messagebox, ttk

# Função para inicializar o banco de dados
def init_db():
    conn = sqlite3.connect('alarmes.db')
    cursor = conn.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS alarmes (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            horario TEXT NOT NULL,
            remedio TEXT NOT NULL,
            audio_path TEXT NOT NULL
        )
    ''')
    conn.commit()
    conn.close()

def salvar_alarme(horario, remedio, audio_path):
    conn = sqlite3.connect('alarmes.db')
    cursor = conn.cursor()
    cursor.execute('''
        INSERT INTO alarmes (horario, remedio, audio_path)
        VALUES (?, ?, ?)
    ''', (horario, remedio, audio_path))
    conn.commit()
    conn.close()

def iniciar_alarme():
    horario = f"{spin_horas.get()}:{spin_minutos.get()}"
    audio_path = audio_path_var.get()
    remedio = entry_remedio.get()

    if not horario or not audio_path or not remedio:
        messagebox.showwarning("Atenção", "Por favor, preencha todos os campos e selecione um áudio.")
        return

    # Salvar alarme no banco de dados
    salvar_alarme(horario, remedio, audio_path)

    def verificar_alarme():
        while True:
            agora = datetime.now().strftime("%H:%M")
            if agora == horario:
                print(f"É hora de tomar o remédio: {remedio} ({horario})")
                playsound(audio_path)
                break
            time.sleep(30)

    # Inicia a função verificar_alarme em uma thread separada
    import threading
    thread_alarme = threading.Thread(target=verificar_alarme)
    thread_alarme.start()
    messagebox.showinfo("Alarme Configurado", f"Alarme configurado para {horario}. Remédio: {remedio}")

def selecionar_audio(opcao):
    audio_path_var.set(opcao)

# Inicializa o banco de dados
init_db()

# Configuração da janela principal
janela = tk.Tk()
janela.title("Despertador Medicinal")
janela.geometry("400x400")
janela.configure(bg="#f0f8ff")  # Cor de fundo

# Título do aplicativo
titulo_label = tk.Label(
    janela, 
    text="Despertador Medicinal", 
    font=("Arial", 16, "bold"), 
    bg="#f0f8ff", 
    fg="#2e4a62"
)
titulo_label.pack(pady=10)

# Seletor de horário
tk.Label(
    janela, 
    text="Escolha o Horário:", 
    font=("Arial", 12), 
    bg="#f0f8ff"
).pack(pady=5)

frame_horario = tk.Frame(janela, bg="#f0f8ff")
frame_horario.pack(pady=5)

spin_horas = ttk.Spinbox(frame_horario, from_=0, to=23, wrap=True, font=("Arial", 12), width=5, format="%02.0f")
spin_horas.set("00")
spin_horas.pack(side=tk.LEFT, padx=5)

tk.Label(frame_horario, text=":", font=("Arial", 12), bg="#f0f8ff").pack(side=tk.LEFT)

spin_minutos = ttk.Spinbox(frame_horario, from_=0, to=59, wrap=True, font=("Arial", 12), width=5, format="%02.0f")
spin_minutos.set("00")
spin_minutos.pack(side=tk.LEFT, padx=5)

# Entrada para o nome do remédio
tk.Label(
    janela, 
    text="Nome do Remédio:", 
    font=("Arial", 12), 
    bg="#f0f8ff"
).pack(pady=5)
entry_remedio = tk.Entry(janela, font=("Arial", 12), width=20)
entry_remedio.pack(pady=5)

# Menu suspenso para escolher o arquivo de áudio
tk.Label(
    janela, 
    text="Escolha o Som:", 
    font=("Arial", 12), 
    bg="#f0f8ff"
