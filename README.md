import telebot
import time
import threading
from telebot.types import ReplyKeyboardMarkup, KeyboardButton

# Token do Bot Administrativo
BOT_TOKEN = "7826678584:AAFqBMDRw7SP1BJ3H63sIoVVkk1sJ16mfpU"
BOT_TOKEN_COMPRAS = "7858924435:AAE0H8X-DhE_LkOQbk2rj4M2Em-JQVyZ47U"  # Token do bot de compras

# Criando os bots
bot = telebot.TeleBot(BOT_TOKEN)
bot_compras = telebot.TeleBot(BOT_TOKEN_COMPRAS)

# Lista de setores que receberão notificações
setores_chat_ids_admin = ["CHAT_ID_DO_ADMIN"]  # Substitua pelo chat ID correto
setores_chat_ids_compras = ["CHAT_ID_DO_COMPRAS"]  # Substitua pelo chat ID correto

# Dicionários para armazenar dados
solicitacoes_pecas = {}
veiculos = {}
servicos = {}
municipios = ["Ielmo Marinho", "Ceará-Mirim", "João Câmara", "Parnamirim", "Jardins de Angicos", "Pedra Grande"]

# Criando menu de opções
menu_principal = ReplyKeyboardMarkup(resize_keyboard=True)
menu_principal.add(KeyboardButton("📋 Atualizar Status"), KeyboardButton("🔍 Consultar Veículos"), KeyboardButton("➕ Cadastrar Veículo"), KeyboardButton("🔧 Abrir Serviço"), KeyboardButton("📊 Gerar Relatório"))

# Mensagem de boas-vindas com nome do usuário
@bot.message_handler(commands=['start'])
def send_welcome(message):
    nome_usuario = message.from_user.first_name  # Obtém o primeiro nome do usuário
    bot.send_message(message.chat.id, f"👋 Olá, {nome_usuario}! Eu sou o Bot Administrativo. Escolha uma opção abaixo:", reply_markup=menu_principal)

# Relatórios
@bot.message_handler(func=lambda message: message.text == "📊 Gerar Relatório")
def gerar_relatorio(message):
    menu_relatorio = ReplyKeyboardMarkup(resize_keyboard=True)
    menu_relatorio.add(KeyboardButton("📌 Serviços em Andamento"), KeyboardButton("📋 Relatório Geral"))
    bot.send_message(message.chat.id, "📊 Escolha o tipo de relatório:", reply_markup=menu_relatorio)
    bot.register_next_step_handler(message, processar_relatorio)

def processar_relatorio(message):
    if message.text == "📌 Serviços em Andamento":
        gerar_relatorio_servicos(message)
    elif message.text == "📋 Relatório Geral":
        gerar_relatorio_geral(message)

def gerar_relatorio_servicos(message):
    relatorio = "📌 **Serviços em Andamento**:\n\n"
    encontrou = False
    for placa, dados in servicos.items():
        if dados["status"] != "Finalizado":
            encontrou = True
            historico = "\n".join(dados.get("historico", ["Nenhum avanço registrado"]))
            relatorio += f"🚗 **Veículo**: {placa}\n🔧 **Serviço**: {dados['servico']}\n👨‍🔧 **Mecânico**: {dados['mecanico']}\n📅 **Histórico**:\n{historico}\n\n"
    if not encontrou:
        relatorio = "✅ Não há serviços em andamento."
    bot.send_message(message.chat.id, relatorio)

def gerar_relatorio_geral(message):
    relatorio = "📋 **Relatório Geral**:\n\n"
    encontrou = False
    for placa, dados in servicos.items():
        encontrou = True
        inicio = dados.get("inicio", "Data não registrada")
        fim = dados.get("fim", "Em andamento")
        relatorio += f"🚗 **Veículo**: {placa}\n🔧 **Serviço**: {dados['servico']}\n👨‍🔧 **Mecânico**: {dados['mecanico']}\n📅 **Início**: {inicio}\n📅 **Término**: {fim}\n\n"
    if not encontrou:
        relatorio = "✅ Nenhum serviço foi registrado ainda."
    bot.send_message(message.chat.id, relatorio)

# Iniciar o bot
if __name__ == "__main__":
    print("🤖 Bot Administrativo rodando...")
    bot.infinity_polling(timeout=10, long_polling_timeout=5)
