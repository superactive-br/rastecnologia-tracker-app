# Rastec Tracker

Aplicativo móvel oficial para rastreamento GPS com a plataforma Rastec Tecnologia.

## Sobre

Rastec Tracker é um aplicativo Flutter para gerenciamento e monitoramento de dispositivos de rastreamento GPS. Oferece rastreamento em tempo real, gerenciamento de dispositivos, notificações de eventos e funcionalidades de histórico/relatórios.

**Recursos principais:**
- Rastreamento em tempo real
- Gerenciamento de dispositivos
- Notificações push de eventos
- Histórico e relatórios (XLSX, KML, CSV, GPX)
- Autenticação segura com biometria
- Suporte a deep linking

## Plataformas

- Android 5.0+ (API 21+)
- iOS 15.0+

## Servidor

Este app se conecta ao servidor Traccar em:
**https://tracker.rastecnologia.com.br**

## Desenvolvimento

### Pré-requisitos

- Flutter SDK
- Android Studio / Xcode
- CocoaPods (iOS)
- FlutterFire CLI

### Configuração

```bash
# Instalar dependências
flutter pub get

# iOS: instalar pods
cd ios && pod install && cd ..

# Verificar ambiente
flutter doctor -v
```

### Executar

```bash
# Android
flutter run -d android

# iOS (requer FlutterFire CLI no PATH)
export PATH="$PATH:$HOME/.pub-cache/bin"
flutter run -d ios
```

### Build

```bash
# Android APK
flutter build apk --release

# Android App Bundle (para Google Play)
flutter build appbundle --release

# iOS
flutter build ios --release
```

## Atribuição

Este projeto é baseado no [Traccar Manager](https://github.com/traccar/traccar-manager-flutter) de Anton Tananaev, licenciado sob Apache License 2.0.

Modificações e rebranding por **Rastec Tecnologia**.

## Contato

**Rastec Tecnologia**
Website: https://rastecnologia.com.br
Suporte: tracker@rastecnologia.com.br

## License

    Apache License, Version 2.0

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
