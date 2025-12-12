# BotOS - Android OS powered by General Bots

**BotOS** transforma qualquer Android em um sistema dedicado ao General Bots, removendo todo bloatware de fabricantes (Samsung, Huawei, Xiaomi, etc) e substituindo pela interface GB.

## Níveis de Instalação

| Nível | Requisitos | O que faz |
|-------|-----------|-----------|
| **1** | Apenas ADB | Remove bloatware, instala BotOS como app |
| **2** | Root + Magisk | Boot animation GB, BotOS como system app |
| **3** | Bootloader desbloqueado | Substitui Android inteiro por BotOS |

## Quick Start

```bash
cd botos/rom
./install.sh
```

O instalador detecta automaticamente o dispositivo e mostra as opções disponíveis.

## Arquitetura

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              NÍVEL 3: GSI                                    │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │ Android AOSP customizado - Zero apps de fabricante                      ││
│  │ Boot animation GB desde inicialização                                   ││
│  │ BotOS integrado como launcher único                                     ││
│  └─────────────────────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────────────────────┤
│                           NÍVEL 2: MAGISK MODULE                            │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │ Android original + Magisk                                               ││
│  │ Bloatware removido via overlay                                          ││
│  │ Boot animation GB                                                       ││
│  │ BotOS como system app privilegiado                                      ││
│  └─────────────────────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────────────────────┤
│                          NÍVEL 1: DEBLOAT + APP                             │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │ Android original (Samsung/Huawei/Xiaomi/etc)                            ││
│  │ Bloatware removido via ADB (sem root)                                   ││
│  │ BotOS instalado como app normal                                         ││
│  │ Pode ser definido como launcher padrão                                  ││
│  └─────────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                             BotOS App (Tauri)                               │
├─────────────────────────────────────────────────────────────────────────────┤
│  botui/ui/suite        │  Tauri Android     │  src/lib.rs (Rust)           │
│  (Interface Web)       │  (WebView + NDK)   │  (Backend + Hardware)        │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Estrutura do Projeto

```
botos/
├── Cargo.toml                    # Dependências Rust/Tauri
├── tauri.conf.json               # Config Tauri → botui/ui/suite
├── build.rs                      # Build script
├── src/lib.rs                    # Entry point Android
│
├── icons/
│   ├── gb-bot.svg                # Ícone fonte
│   ├── icon.png (512x512)        # Ícone principal
│   └── */ic_launcher.png         # Ícones por densidade
│
├── scripts/
│   ├── generate-icons.sh         # Gera PNGs do SVG
│   └── create-bootanimation.sh   # Gera animação de boot
│
├── capabilities/
│   └── default.json              # Permissões Tauri
│
├── gen/android/                  # Projeto Android gerado
│   └── app/src/main/
│       ├── AndroidManifest.xml   # HOME intent (launcher)
│       └── res/values/themes.xml # Tema dark GB
│
└── rom/                          # Tools de instalação
    ├── install.sh                # Instalador interativo
    ├── scripts/
    │   ├── debloat.sh            # Remove bloatware (sem root)
    │   └── build-magisk-module.sh # Gera módulo Magisk
    └── gsi/
        ├── README.md             # Instruções GSI/AOSP
        └── device/pragmatismo/botos/  # Device tree AOSP
```

## Pré-requisitos

### Para compilar BotOS App

```bash
# Rust e Android targets
rustup target add aarch64-linux-android armv7-linux-androideabi

# Android SDK e NDK
export ANDROID_HOME=$HOME/Android/Sdk
export NDK_HOME=$ANDROID_HOME/ndk/25.2.9519653

# Tauri CLI
cargo install tauri-cli
```

### Para instalar em dispositivos

```bash
# ADB
sudo apt install adb

# Para gerar ícones/boot animation
sudo apt install librsvg2-bin imagemagick
```

## Compilação

```bash
cd botos

# Gerar ícones
./scripts/generate-icons.sh

# Inicializar projeto Android
cargo tauri android init

# Build APK
cargo tauri android build --release
```

## Instalação

### Método Rápido (Interativo)

```bash
cd botos/rom
chmod +x install.sh
./install.sh
```

### Método Manual

#### Nível 1: Debloat + App (Sem Root)

```bash
# Conectar dispositivo via USB (debug ativo)
cd botos/rom/scripts
./debloat.sh

# Instalar APK
adb install ../gen/android/app/build/outputs/apk/release/app-release.apk

# Definir como launcher: Home → BotOS → Sempre
```

#### Nível 2: Magisk Module (Com Root)

```bash
# Gerar módulo
cd botos/rom/scripts
./build-magisk-module.sh

# Copiar para dispositivo
adb push botos-magisk-v1.0.zip /sdcard/

# No celular: Magisk → Modules → + → Selecionar ZIP → Reboot
```

#### Nível 3: GSI (Bootloader Desbloqueado)

Veja instruções detalhadas em `rom/gsi/README.md`.

## Bloatware Removido

O debloat remove automaticamente:

**Samsung One UI:**
- Bixby, Samsung Pay, Samsung Pass
- Apps duplicados (Email, Calendar, Browser)
- AR Zone, Game Launcher

**Huawei EMUI:**
- AppGallery, HiCloud, HiCar
- Huawei Browser, Music, Video

**Xiaomi MIUI:**
- MSA (analytics), Mi Apps
- GetApps, Mi Cloud

**Universal (todos):**
- Facebook, Instagram pré-instalados
- Netflix, Spotify pré-instalados
- Jogos como Candy Crush

## Boot Animation

Para customizar a animação de boot (requer root):

```bash
# Gerar animação
./scripts/create-bootanimation.sh

# Instalar (root)
adb root
adb remount
adb push bootanimation.zip /system/media/
adb reboot
```

## Desenvolvimento

```bash
# Dev mode (conecta ao dispositivo)
cargo tauri android dev

# Logs
adb logcat -s BotOS:*
```

## Parceria Chuna

BotOS foi criado para vendas/parcerias na Chuna, oferecendo:
- Celulares com sistema "limpo" - sem bloatware
- Interface única conectada ao General Bots
- Experiência simplificada para usuários finais
- Controle total do dispositivo

## Recursos

- 🏠 **Launcher Mode**: Substitui home screen
- 🤖 **Interface Chat**: botui/ui/suite
- 🦀 **Backend Rust**: Via Tauri
- 📍 **GPS**: Acesso a localização
- 📷 **Câmera**: Via plugins Tauri
- 🔔 **Notificações**: Push notifications
- 🌐 **Internet**: Comunicação com botserver
- 🎨 **Boot Animation**: Customizável com gb-bot.svg

## Licença

AGPL-3.0
