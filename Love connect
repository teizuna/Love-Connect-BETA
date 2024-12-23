from pythonosc.dispatcher import Dispatcher
from pythonosc.osc_server import AsyncIOOSCUDPServer
import asyncio
import logging
from buttplug import Client, WebsocketConnector, ProtocolSpec

# 🌐 Config réseau
IP = "0.0.0.0"  # Écoute sur toutes les interfaces
PORT = 9001     # Port pour OSC
BUTTPLUG_SERVER_URL = "ws://127.0.0.1:12345"  # URL serveur Buttplug

# 🚀 Limites des moteurs
LIMITS = {
    "Lovense Gemini": 0.1,  # Limite pour Gemini 💎
    "Lovense Lush": 0.5     # Limite pour Lush 🌀
}

# 🛠️ Setup global
buttplug_client = None
devices = {}  # 🗆 Tous les dispositifs détectés


async def initialize_buttplug():
    """🔧 Init connexion serveur Buttplug"""
    global buttplug_client, devices
    buttplug_client = Client("Love-Connect BETA By Molly", ProtocolSpec.v3)
    connector = WebsocketConnector(BUTTPLUG_SERVER_URL, logger=buttplug_client.logger)

    try:
        await buttplug_client.connect(connector)  # 🔗 Connexion
        await buttplug_client.start_scanning()  # 🔍 Scan des dispositifs
        await asyncio.sleep(5)  # ⏳ Attente
        await buttplug_client.stop_scanning()  # 🚫 Fin du scan

        # 🔧 Save dispositifs
        for device in buttplug_client.devices.values():
            devices[device.name] = device

        if devices:
            print(f"🟢 Détecté : {list(devices.keys())}")
        else:
            print("🔴 Aucun dispositif détecté.")
    except Exception as e:
        logging.error(f"⚠️ Erreur serveur Buttplug : {e}")


async def adjust_intensity(device_name, motor_index, intensity):
    """⚙️ Ajuste l'intensité d'un moteur

    Paramètres :
        device_name (str) : Nom du dispositif
        motor_index (int) : Index du moteur dans le dispositif
        intensity (float) : Intensité souhaitée (entre 0 et 1 inclus)
    """
    global devices
    if device_name not in devices:
        print(f"❌ '{device_name}' introuvable.")
        return

    device = devices[device_name]
    max_intensity = LIMITS.get(device_name, 1)  # 💡 Limite max
    limited_intensity = min(max(intensity, 0), max_intensity)  # ⛔ Cap limite (entre 0 et max)

    try:
        if motor_index < len(device.actuators):  # ✅ Vérif moteur
            normalized_intensity = limited_intensity  # 🔄 Normalisation directe (0 à 1)
            await device.actuators[motor_index].command(normalized_intensity)  # 📤 Envoi
            print(f"✨ [{device_name}] Moteur {motor_index}: {limited_intensity}/{max_intensity}")
        else:
            print(f"⚠️ Moteur {motor_index} introuvable ({device_name}).")
    except Exception as e:
        logging.error(f"⚠️ Erreur moteur {motor_index} ({device_name}) : {e}")


# 🔔 Handlers OSC
def handle_message(device_name, motor_index, unused_addr, args):
    """🎵 OSC → Commande d'intensité

    Paramètres :
        device_name (str) : Nom du dispositif
        motor_index (int) : Index du moteur ciblé
        unused_addr : Adresse OSC (inutile ici)
        args : Liste ou valeur unique (intensité)
    """
    try:
        intensity = float(args[0]) if isinstance(args, list) else float(args)
        print(f"🔔 {device_name} Moteur {motor_index} msg : {intensity}")
        asyncio.create_task(adjust_intensity(device_name, motor_index, intensity))
    except Exception as e:
        logging.error(f"⚠️ OSC {device_name} Moteur {motor_index} : {e}")


def handle_message_gemini_left(unused_addr, args):
    handle_message("Lovense Gemini", 0, unused_addr, args)


def handle_message_gemini_right(unused_addr, args):
    handle_message("Lovense Gemini", 1, unused_addr, args)


def handle_message_lush(unused_addr, args):
    handle_message("Lovense Lush", 0, unused_addr, args)


async def run_server():
    """🚦 Démarre serveur OSC
    """
    dispatcher = Dispatcher()
    dispatcher.map("/avatar/parameters/RearMassageL", handle_message_gemini_left)  # ⬅️ Gemini L
    dispatcher.map("/avatar/parameters/RearMassageR", handle_message_gemini_right)  # ➡️ Gemini R
    dispatcher.map("/avatar/parameters/TickleSpot01", handle_message_lush)  # 🌀 Lush

    server = AsyncIOOSCUDPServer((IP, PORT), dispatcher, asyncio.get_event_loop())
    transport, protocol = await server.create_serve_endpoint()
    print(f"🟢 OSC écoute : {IP}:{PORT}...")

    try:
        while True:
            await asyncio.sleep(1)  # ⏳ Boucle active
    except KeyboardInterrupt:
        print("🚫 Arrêt OSC.")
    finally:
        transport.close()


async def main():
    """🚀 Lancement
    """
    logging.basicConfig(level=logging.INFO)  # 📜 Logs
    await initialize_buttplug()  # 🔧 Init Buttplug
    await run_server()  # 🚦 Démarre OSC


if __name__ == "__main__":
    asyncio.run(main())  # 🏁 Start
