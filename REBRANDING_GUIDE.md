# Guia de Rebranding: Traccar Manager → Rastec Tracker

Este documento fornece o passo a passo completo para transformar o app Traccar Manager no white label "Rastec Tracker" para a Rastec Tecnologia.

---

## Índice

1. [Pré-requisitos](#1-pré-requisitos)
2. [Preparação](#2-preparação)
3. [Alterações de Código](#3-alterações-de-código)
4. [Ícones e Assets](#4-ícones-e-assets)
5. [Integração Firebase](#5-integração-firebase)
6. [Configuração de Build](#6-configuração-de-build)
7. [Testes](#7-testes)
8. [Deploy nas Lojas](#8-deploy-nas-lojas)
9. [Checklist Final](#9-checklist-final)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. Pré-requisitos

### Ferramentas Necessárias
- ✅ Flutter SDK instalado e atualizado
- ✅ Xcode (para builds iOS)
- ✅ Android Studio / Android SDK
- ✅ CocoaPods (para dependências iOS)
- ✅ FlutterFire CLI instalado: `dart pub global activate flutterfire_cli`
- ✅ Conta Firebase/Google Cloud
- ✅ Conta Apple Developer (US$ 99/ano)
- ✅ Conta Google Play Console (US$ 25 taxa única)

### Verificar Ambiente
```bash
flutter doctor -v
pod --version
dart pub global list | grep flutterfire
```

### Adicionar FlutterFire ao PATH
```bash
# Adicione ao seu ~/.zshrc ou ~/.bash_profile
export PATH="$PATH:$HOME/.pub-cache/bin"
```

---

## 2. Preparação

### 2.1. Criar Projeto Firebase

1. Acesse [Firebase Console](https://console.firebase.google.com/)
2. Clique em "Adicionar projeto"
3. Nome do projeto: **Rastec Tracker**
4. Habilite Google Analytics (opcional)
5. Criar projeto

### 2.2. Adicionar Apps ao Firebase

#### Android:
1. No painel do projeto, clique em "Adicionar app" → Android
2. **Package name:** `rastec.Android`
3. **App nickname:** Rastec Tracker Android
4. Baixar `google-services.json`
5. **NÃO copie ainda** - faremos isso na Fase 5

#### iOS:
1. Clique em "Adicionar app" → iOS
2. **Bundle ID:** `br.com.rastecnologia`
3. **App nickname:** Rastec Tracker iOS
4. Baixar `GoogleService-Info.plist`
5. **NÃO copie ainda** - faremos isso na Fase 5

### 2.3. Habilitar Firebase Cloud Messaging (Notificações)

1. No Firebase Console, vá em **Build** → **Cloud Messaging**
2. Clique em "Começar"
3. Para iOS, você precisará:
   - Upload do **APNs Authentication Key** (.p8 file)
   - Ou **APNs Certificate** (.p12 file)
   - Obtenha em: [Apple Developer → Certificates, Identifiers & Profiles](https://developer.apple.com/account/resources/authkeys/list)

### 2.4. Verificar Keystore Android Existente

Como você usará keystore existente com package `rastec.Android`, certifique-se de que:

```bash
# Verificar informações do keystore
keytool -list -v -keystore /caminho/para/seu/keystore.jks -alias sua_alias
```

Confirme que o keystore foi criado para o package `rastec.Android`.

### 2.5. Verificar Apple Developer

1. Acesse [Apple Developer Portal](https://developer.apple.com/account/)
2. Vá em **Certificates, Identifiers & Profiles**
3. Verifique se existe App ID para `br.com.rastecnologia`
   - Se não existir, crie um novo **App ID** com esse Bundle Identifier
4. Verifique/crie **Provisioning Profiles**:
   - Development Profile (para testes)
   - Distribution Profile (para App Store)

### 2.6. Preparar Ícones

Você precisará gerar os seguintes ícones a partir do logo da Rastec:

#### Android (7 arquivos):

**Ícones Launcher (PNG, fundo transparente ou colorido):**
- `ic_launcher.png` - 48x48 px (mdpi)
- `ic_launcher.png` - 72x72 px (hdpi)
- `ic_launcher.png` - 96x96 px (xhdpi)
- `ic_launcher.png` - 144x144 px (xxhdpi)
- `ic_launcher.png` - 192x192 px (xxxhdpi)

**Adaptive Icon (Android 8.0+):**
- `ic_launcher_foreground.xml` - Vector Drawable (SVG convertido para XML)
  - Tamanho: 108x108 dp (área segura: 66x66 dp centralizado)
  - Formato: Vector XML ou PNG 432x432px

**Ícone de Notificação:**
- `ic_stat_notify.xml` - Vector Drawable monocromático branco
  - Tamanho: 24x24 dp
  - Estilo: Apenas silhueta/contorno branco em transparente

#### iOS (19 arquivos PNG):

Todos com fundo **opaco** (não transparente) e formato **PNG**:

| Arquivo | Tamanho | Uso |
|---------|---------|-----|
| 20.png | 20x20 | iPad Notifications |
| 29.png | 29x29 | Settings 1x |
| 40.png | 40x40 | Spotlight/Notifications 2x |
| 50.png | 50x50 | iPad Spotlight 1x |
| 57.png | 57x57 | iPhone 1x (legacy) |
| 58.png | 58x58 | Settings 2x |
| 60.png | 60x60 | iPhone App 3x |
| 72.png | 72x72 | iPad App 1x (legacy) |
| 76.png | 76x76 | iPad App 1x |
| 80.png | 80x80 | Spotlight 2x |
| 87.png | 87x87 | Settings 3x |
| 100.png | 100x100 | iPad Spotlight 2x |
| 114.png | 114x114 | iPhone App 2x (legacy) |
| 120.png | 120x120 | iPhone App 2x/3x |
| 144.png | 144x144 | iPad App 2x (legacy) |
| 152.png | 152x152 | iPad App 2x |
| 167.png | 167x167 | iPad Pro |
| 180.png | 180x180 | iPhone App 3x |
| **1024.png** | **1024x1024** | **App Store** |

**Ferramentas Recomendadas para Gerar Ícones:**
- [AppIcon.co](https://appicon.co/) - Gera todos os tamanhos automaticamente
- [MakeAppIcon](https://makeappicon.com/)
- Adobe Illustrator/Photoshop (manual)
- Figma (design + export)

---

## 3. Alterações de Código

### Resumo das Mudanças:

| Item | De | Para |
|------|------|------|
| **Android Package** | `org.traccar.manager` | `rastec.Android` |
| **iOS Bundle ID** | `org.traccar.TraccarManager` | `br.com.rastecnologia` |
| **Flutter Package** | `traccar_manager` | `rastec_tracker` |
| **App Name** | Traccar Manager | Rastec Tracker |
| **Deep Link Android** | `org.traccar.manager://` | `rastec.android://` |
| **Deep Link iOS** | `org.traccar.manager://` | `br.com.rastecnologia://` |
| **URL Padrão** | https://demo.traccar.org | https://tracker.rastecnologia.com.br |

---

### 3.1. Atualizar `pubspec.yaml`

**Arquivo:** `/pubspec.yaml`

```yaml
# ANTES:
name: traccar_manager
description: "Traccar Manager"
publish_to: 'none'
version: 5.2.0+63

# DEPOIS:
name: rastec_tracker
description: "Rastec Tracker - Rastreamento GPS"
publish_to: 'none'
version: 1.0.0+1  # Resetando versão para novo app
```

---

### 3.2. Atualizar Imports em Arquivos Dart

#### Arquivo: `lib/main.dart`

```dart
// ANTES:
import 'package:traccar_manager/main_screen.dart';

// DEPOIS:
import 'package:rastec_tracker/main_screen.dart';
```

#### Arquivo: `lib/main_screen.dart`

```dart
// ANTES:
import 'package:traccar_manager/error_screen.dart';
import 'package:traccar_manager/main.dart';
import 'package:traccar_manager/token_store.dart';

// DEPOIS:
import 'package:rastec_tracker/error_screen.dart';
import 'package:rastec_tracker/main.dart';
import 'package:rastec_tracker/token_store.dart';
```

#### Arquivo: `lib/error_screen.dart`

```dart
// ANTES:
import 'package:traccar_manager/main.dart';

// DEPOIS:
import 'package:rastec_tracker/main.dart';
```

---

### 3.3. Android - Mover e Atualizar MainActivity.kt

#### Passo 1: Criar nova estrutura de diretórios

```bash
cd android/app/src/main/kotlin
mkdir -p rastec/Android
```

#### Passo 2: Mover MainActivity.kt

```bash
mv org/traccar/manager/MainActivity.kt rastec/Android/
```

#### Passo 3: Deletar estrutura antiga

```bash
rm -rf org/
```

#### Passo 4: Atualizar package em MainActivity.kt

**Arquivo:** `android/app/src/main/kotlin/rastec/Android/MainActivity.kt`

```kotlin
// ANTES:
package org.traccar.manager

// DEPOIS:
package rastec.Android

import io.flutter.embedding.android.FlutterActivity

class MainActivity: FlutterActivity() {
}
```

---

### 3.4. Android - Atualizar `build.gradle.kts`

**Arquivo:** `android/app/build.gradle.kts`

```kotlin
// ANTES (linha ~22):
namespace = "org.traccar.manager"

// DEPOIS:
namespace = "rastec.Android"

// ---

// ANTES (linha ~36):
applicationId = "org.traccar.manager"

// DEPOIS:
applicationId = "rastec.Android"
```

**Verificar configuração de signing (linha ~16):**

```kotlin
val keystorePropertiesFile = rootProject.file("../../environment/key.properties")
// Certifique-se de que este arquivo aponta para o keystore correto da Rastec
```

---

### 3.5. Android - Atualizar `AndroidManifest.xml`

**Arquivo:** `android/app/src/main/AndroidManifest.xml`

```xml
<!-- ANTES (linha ~8): -->
<application
    android:label="Traccar Manager"
    ...>

<!-- DEPOIS: -->
<application
    android:label="Rastec Tracker"
    ...>

<!-- ========== -->

<!-- ANTES (linha ~37 - deep linking): -->
<data android:scheme="org.traccar.manager" android:pathPrefix="/"/>

<!-- DEPOIS: -->
<data android:scheme="rastec.android" android:pathPrefix="/"/>
```

---

### 3.6. iOS - Atualizar `Info.plist`

**Arquivo:** `ios/Runner/Info.plist`

```xml
<!-- ANTES (linha ~10): -->
<key>CFBundleDisplayName</key>
<string>Traccar Manager</string>

<!-- DEPOIS: -->
<key>CFBundleDisplayName</key>
<string>Rastec Tracker</string>

<!-- ========== -->

<!-- ANTES (linha ~18): -->
<key>CFBundleName</key>
<string>traccar_manager</string>

<!-- DEPOIS: -->
<key>CFBundleName</key>
<string>rastec_tracker</string>

<!-- ========== -->

<!-- ANTES (linha ~64 - deep linking): -->
<key>CFBundleURLSchemes</key>
<array>
    <string>org.traccar.manager</string>
</array>

<!-- DEPOIS: -->
<key>CFBundleURLSchemes</key>
<array>
    <string>br.com.rastecnologia</string>
</array>
```

---

### 3.7. iOS - Atualizar `project.pbxproj`

**Arquivo:** `ios/Runner.xcodeproj/project.pbxproj`

**Este arquivo é extenso. Você precisará substituir em 6 locais:**

#### Localizar e substituir:

```bash
# Use um editor de texto ou faça find-replace:
# ANTES:
PRODUCT_BUNDLE_IDENTIFIER = org.traccar.TraccarManager;

# DEPOIS:
PRODUCT_BUNDLE_IDENTIFIER = br.com.rastecnologia;
```

**Localizações exatas (linhas aproximadas):**
- Linha ~525 (Release configuration)
- Linha ~710 (Debug configuration)
- Linha ~735 (Profile configuration)

**Para os Test Targets:**

```bash
# ANTES:
PRODUCT_BUNDLE_IDENTIFIER = com.example.traccarManager.RunnerTests;

# DEPOIS:
PRODUCT_BUNDLE_IDENTIFIER = br.com.rastecnologia.RunnerTests;
```

**Localizações (linhas aproximadas):**
- Linha ~542
- Linha ~560
- Linha ~576

**Dica:** Use o comando sed ou editor de texto com find/replace global.

```bash
# Backup antes de modificar
cp ios/Runner.xcodeproj/project.pbxproj ios/Runner.xcodeproj/project.pbxproj.backup

# Substituir automaticamente (macOS/Linux)
sed -i '' 's/org\.traccar\.TraccarManager/br.com.rastecnologia/g' ios/Runner.xcodeproj/project.pbxproj
sed -i '' 's/com\.example\.traccarManager/br.com.rastecnologia/g' ios/Runner.xcodeproj/project.pbxproj
```

---

### 3.8. Atualizar `main_screen.dart`

**Arquivo:** `lib/main_screen.dart`

#### Deep Link Scheme (linha ~57):

```dart
// ANTES:
if (uri.scheme == 'org.traccar.manager') {

// DEPOIS:
if (uri.scheme == 'rastec.android' || uri.scheme == 'br.com.rastecnologia') {
```

#### OAuth Redirect URI Rewrite (linha ~78):

```dart
// ANTES:
scheme: 'org.traccar.manager',

// DEPOIS:
scheme: Platform.isAndroid ? 'rastec.android' : 'br.com.rastecnologia',
```

**Adicione import no topo do arquivo:**

```dart
import 'dart:io'; // Se ainda não estiver importado
```

#### URL Padrão do Servidor (linha ~97):

```dart
// ANTES:
return _preferences.getString(_urlKey) ?? 'https://demo.traccar.org';

// DEPOIS:
return _preferences.getString(_urlKey) ?? 'https://tracker.rastecnologia.com.br';
```

---

### 3.9. Atualizar Documentação

#### Arquivo: `README.md`

Substitua o conteúdo com:

```markdown
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

## Licença

Este projeto é baseado no [Traccar Manager](https://github.com/traccar/traccar-manager-flutter) de Anton Tananaev, licenciado sob Apache License 2.0.

Modificações e rebranding por **Rastec Tecnologia**.

```
Copyright 2024 Rastec Tecnologia

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

## Atribuição

Baseado em [Traccar Manager Flutter](https://github.com/traccar/traccar-manager-flutter)
Copyright (c) Anton Tananaev
Licensed under Apache License 2.0

## Contato

**Rastec Tecnologia**
Website: https://rastecnologia.com.br
Suporte: tracker@rastecnologia.com.br
```

#### Arquivo: `CLAUDE.md`

Atualize a seção **Project Overview**:

```markdown
## Project Overview

Rastec Tracker é o aplicativo móvel oficial Flutter da Rastec Tecnologia para gerenciamento e monitoramento de dispositivos de rastreamento GPS com a plataforma Traccar. O app oferece rastreamento ao vivo, gerenciamento de dispositivos, notificações de eventos e funcionalidades de histórico/relatórios.

**Servidor:** https://tracker.rastecnologia.com.br

**Identificadores:**
- Android Package: `rastec.Android`
- iOS Bundle ID: `br.com.rastecnologia`
- Deep Link Schemes: `rastec.android://` (Android), `br.com.rastecnologia://` (iOS)

**Principais Funcionalidades:**
- Arquitetura baseada em WebView envolvendo interface web Traccar
- Integração Firebase (Analytics, Crashlytics, Messaging)
- Armazenamento seguro de tokens com autenticação biométrica
- Suporte a deep linking para fluxos OAuth e notificações
- Download/compartilhamento de arquivos (XLSX, KML, CSV, GPX)
- Notificações push com tratamento de eventos

[... restante do conteúdo permanece igual ...]
```

---

## 4. Ícones e Assets

### 4.1. Android Icons

#### Substituir Launcher Icons (PNG)

Copie seus ícones preparados para:

```bash
android/app/src/main/res/mipmap-mdpi/ic_launcher.png       # 48x48
android/app/src/main/res/mipmap-hdpi/ic_launcher.png       # 72x72
android/app/src/main/res/mipmap-xhdpi/ic_launcher.png      # 96x96
android/app/src/main/res/mipmap-xxhdpi/ic_launcher.png     # 144x144
android/app/src/main/res/mipmap-xxxhdpi/ic_launcher.png    # 192x192
```

#### Substituir Adaptive Icon Foreground

**Arquivo:** `android/app/src/main/res/drawable/ic_launcher_foreground.xml`

Substitua o conteúdo com seu logo em formato Vector Drawable (XML).

**Exemplo de estrutura:**

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="108"
    android:viewportHeight="108">
    <!-- Seu logo SVG convertido para paths aqui -->
    <path
        android:fillColor="#SUA_COR"
        android:pathData="M... (dados do path)"/>
</vector>
```

**Ferramentas para converter SVG → Vector Drawable:**
- [svg2vector](https://svg2vector.com/)
- Android Studio: New → Vector Asset → Local file

#### Verificar Background do Adaptive Icon

**Arquivo:** `android/app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@color/ic_launcher_background"/>
    <foreground android:drawable="@drawable/ic_launcher_foreground"/>
</adaptive-icon>
```

Se quiser mudar a cor de fundo, edite:

**Arquivo:** `android/app/src/main/res/values/colors.xml`

```xml
<resources>
    <color name="ic_launcher_background">#FFFFFF</color> <!-- Sua cor -->
</resources>
```

#### Substituir Ícone de Notificação

**Arquivo:** `android/app/src/main/res/drawable/ic_stat_notify.xml`

Deve ser um ícone **monocromático branco** (silhueta).

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M... (silhueta do logo)"/>
</vector>
```

---

### 4.2. iOS Icons

#### Substituir AppIcon

Copie todos os 19 PNGs preparados para:

```
ios/Runner/Assets.xcassets/AppIcon.appiconset/
```

**Estrutura de arquivos:**

```
AppIcon.appiconset/
├── 20.png          (20x20)
├── 29.png          (29x29)
├── 40.png          (40x40)
├── 50.png          (50x50)
├── 57.png          (57x57)
├── 58.png          (58x58)
├── 60.png          (60x60)
├── 72.png          (72x72)
├── 76.png          (76x76)
├── 80.png          (80x80)
├── 87.png          (87x87)
├── 100.png         (100x100)
├── 114.png         (114x114)
├── 120.png         (120x120)
├── 144.png         (144x144)
├── 152.png         (152x152)
├── 167.png         (167x167)
├── 180.png         (180x180)
├── 1024.png        (1024x1024)
└── Contents.json
```

**Verifique se `Contents.json` está correto:**

```json
{
  "images" : [
    {
      "filename" : "20.png",
      "idiom" : "ipad",
      "scale" : "1x",
      "size" : "20x20"
    },
    // ... (restante das entradas)
    {
      "filename" : "1024.png",
      "idiom" : "ios-marketing",
      "scale" : "1x",
      "size" : "1024x1024"
    }
  ],
  "info" : {
    "author" : "xcode",
    "version" : 1
  }
}
```

Se você usou uma ferramenta automática de geração, o `Contents.json` já virá pronto.

---

## 5. Integração Firebase

### 5.1. Copiar Arquivos de Configuração

#### Android:

```bash
# Copie o google-services.json baixado do Firebase Console para:
cp ~/Downloads/google-services.json android/app/
```

Verifique se o arquivo contém o package correto:

```json
{
  "client": [
    {
      "client_info": {
        "android_client_info": {
          "package_name": "rastec.Android"  // ← Deve ser exatamente isso
        }
      }
    }
  ]
}
```

#### iOS:

```bash
# Copie o GoogleService-Info.plist baixado do Firebase Console para:
cp ~/Downloads/GoogleService-Info.plist ios/Runner/
```

Verifique se o Bundle ID está correto:

```xml
<key>BUNDLE_ID</key>
<string>br.com.rastecnologia</string>
```

---

### 5.2. Executar FlutterFire Configure

Este comando regenera `lib/firebase_options.dart` com as novas configurações:

```bash
# Certifique-se de que o FlutterFire CLI está no PATH
export PATH="$PATH:$HOME/.pub-cache/bin"

# Execute o comando
flutterfire configure
```

**Interação esperada:**

1. Selecione o projeto Firebase: **Rastec Tracker**
2. Selecione plataformas: **android, ios**
3. Para Android, confirme package: `rastec.Android`
4. Para iOS, confirme bundle ID: `br.com.rastecnologia`
5. Confirme sobrescrever `firebase_options.dart`

**Verificar arquivo gerado:**

`lib/firebase_options.dart` deve conter:

```dart
static const FirebaseOptions android = FirebaseOptions(
  apiKey: '...',
  appId: '...',
  messagingSenderId: '...',
  projectId: 'rastec-tracker', // ou similar
  storageBucket: '...',
);

static const FirebaseOptions ios = FirebaseOptions(
  apiKey: '...',
  appId: '...',
  messagingSenderId: '...',
  projectId: 'rastec-tracker',
  storageBucket: '...',
  iosBundleId: 'br.com.rastecnologia',
);
```

---

## 6. Configuração de Build

### 6.1. Android - Configurar Signing

**Arquivo:** `../../environment/key.properties`

Crie ou verifique este arquivo (fora do repositório Git):

```properties
storePassword=SUA_SENHA_DO_KEYSTORE
keyPassword=SUA_SENHA_DA_CHAVE
keyAlias=SUA_ALIAS
storeFile=/caminho/absoluto/para/seu/keystore.jks
```

**Exemplo:**

```properties
storePassword=Rastec@2024
keyPassword=Rastec@2024
keyAlias=rastec_key
storeFile=/Users/rafael/keystores/rastec_android.jks
```

**IMPORTANTE:**
- Nunca versione `key.properties` no Git (já está no `.gitignore`)
- Faça backup seguro do keystore e senhas
- Perder o keystore = não pode atualizar o app na Google Play

---

### 6.2. iOS - Configurar Code Signing no Xcode

1. Abra o projeto no Xcode:
   ```bash
   open ios/Runner.xcworkspace
   ```

2. No Xcode, selecione **Runner** no Project Navigator

3. Vá para a aba **Signing & Capabilities**

4. **Para Debug:**
   - Team: Selecione sua conta Apple Developer
   - Bundle Identifier: `br.com.rastecnologia` (já atualizado)
   - Signing Certificate: Apple Development
   - Provisioning Profile: Automatic ou selecione manualmente

5. **Para Release:**
   - Team: Sua conta Apple Developer
   - Bundle Identifier: `br.com.rastecnologia`
   - Signing Certificate: Apple Distribution
   - Provisioning Profile: App Store (ou Ad Hoc para testes)

6. Certifique-se de que não há erros de signing

---

### 6.3. Atualizar Versão

Se quiser resetar a versão para 1.0.0 (novo app):

**Arquivo:** `pubspec.yaml`

```yaml
version: 1.0.0+1
```

Isso define:
- **Version name:** 1.0.0 (visível para usuários)
- **Build number:** 1 (incrementa a cada build)

---

## 7. Testes

### 7.1. Limpar Build Cache

```bash
flutter clean
rm -rf build/
rm -rf ios/Pods/
rm -rf ios/Podfile.lock
```

### 7.2. Instalar Dependências

```bash
flutter pub get
cd ios && pod install && cd ..
```

### 7.3. Verificar Configuração

```bash
flutter doctor -v
```

Certifique-se de que não há issues críticos.

---

### 7.4. Testar Build Android

```bash
# Build debug APK
flutter build apk --debug

# Build release APK
flutter build apk --release

# Ou build App Bundle para Google Play
flutter build appbundle --release
```

**Verificar saída:**

```
Built build/app/outputs/flutter-apk/app-release.apk (XX MB)
```

**Instalar no dispositivo para testar:**

```bash
flutter install -d <device-id>
```

---

### 7.5. Testar Build iOS

**IMPORTANTE:** Requer Xcode e configuração de signing.

```bash
# Listar dispositivos disponíveis
flutter devices

# Build iOS (sem assinar)
flutter build ios --debug --no-codesign

# Build iOS release (requer signing)
flutter build ios --release
```

**Ou abra no Xcode e rode:**

```bash
open ios/Runner.xcworkspace
```

1. Selecione um simulador ou dispositivo
2. Product → Run (⌘R)

---

### 7.6. Testes Funcionais

Teste os seguintes cenários:

#### Conexão ao Servidor
- [ ] App abre e carrega https://tracker.rastecnologia.com.br
- [ ] Login funciona corretamente
- [ ] Dados são exibidos após login

#### Autenticação
- [ ] Token é salvo após login
- [ ] App pede biometria (se configurado)
- [ ] Logout funciona

#### Deep Linking
- [ ] OAuth redirect funciona (se aplicável)
- [ ] Testar URL: `rastec.android://event/123` (Android)
- [ ] Testar URL: `br.com.rastecnologia://event/123` (iOS)

Comando para testar deep link:

**Android:**
```bash
adb shell am start -W -a android.intent.action.VIEW -d "rastec.android://event/123" rastec.Android
```

**iOS:**
```bash
xcrun simctl openurl booted "br.com.rastecnologia://event/123"
```

#### Notificações Push
- [ ] Registrar token no Firebase
- [ ] Enviar notificação de teste do Firebase Console
- [ ] Tocar na notificação abre o app
- [ ] Deep link da notificação funciona

#### Downloads
- [ ] Exportar relatório XLSX funciona
- [ ] Download abre share sheet
- [ ] Arquivo é salvo corretamente

---

## 8. Deploy nas Lojas

### 8.1. Google Play Store

#### Passo 1: Criar App no Play Console

1. Acesse [Google Play Console](https://play.google.com/console/)
2. Selecione "Todos os apps" → "Criar app"
3. Detalhes:
   - **Nome:** Rastec Tracker
   - **Idioma padrão:** Português (Brasil)
   - **App ou jogo:** App
   - **Gratuito ou pago:** Gratuito
   - **Declarações:** Aceite os termos
4. Criar app

#### Passo 2: Preencher Ficha da Loja

1. **Ficha da loja principal:**
   - **Título curto:** Rastec Tracker (até 30 caracteres)
   - **Descrição completa:**
     ```
     Rastec Tracker é o app oficial de rastreamento GPS da Rastec Tecnologia.

     Recursos:
     • Rastreamento em tempo real de veículos e ativos
     • Visualização de histórico e rotas
     • Notificações de eventos (velocidade, cerca geográfica, etc.)
     • Relatórios detalhados em XLSX, KML, CSV
     • Interface intuitiva e responsiva
     • Autenticação segura com biometria

     Conecte-se ao servidor Rastec e monitore sua frota com facilidade.
     ```
   - **Descrição curta:**
     ```
     Rastreamento GPS em tempo real com Rastec Tecnologia
     ```

2. **Recursos gráficos:**
   - **Ícone do app:** 512x512 PNG (32-bit com transparência)
   - **Imagem de destaque:** 1024x500 JPEG/PNG
   - **Capturas de tela:**
     - **Telefone:** Mínimo 2 (máximo 8) - 16:9 ou 9:16
       - Tamanho: 320px - 3840px (lado mais longo)
     - **Tablet 7":** Opcional
     - **Tablet 10":** Opcional

   **Dica:** Use o app no simulador e tire screenshots:
   - Tela de login
   - Mapa com rastreamento
   - Lista de dispositivos
   - Relatórios

3. **Classificação do conteúdo:**
   - Preencha o questionário (geralmente classificação "Livre")

4. **Público-alvo:**
   - Selecione "18+" ou "13+" conforme aplicável

5. **Categoria:**
   - Categoria: **Negócios** ou **Produtividade**

6. **Informações de contato:**
   - Email: suporte@rastecnologia.com.br
   - Website: https://rastecnologia.com.br
   - Política de privacidade: [URL da sua política]

#### Passo 3: Build e Upload do App Bundle

```bash
# Gerar App Bundle assinado
flutter build appbundle --release
```

**Arquivo gerado:**
```
build/app/outputs/bundle/release/app-release.aab
```

**Upload:**

1. No Play Console, vá em **Produção** → **Criar nova versão**
2. Clique em "Upload" e selecione `app-release.aab`
3. **Nome da versão:** 1.0.0 (ou conforme seu `pubspec.yaml`)
4. **Notas da versão:**
   ```
   Versão inicial do Rastec Tracker
   - Rastreamento GPS em tempo real
   - Notificações de eventos
   - Relatórios e histórico
   ```
5. Clique em "Salvar"

#### Passo 4: Testar com Internal Testing (Recomendado)

Antes de enviar para produção, teste com um grupo fechado:

1. Vá em **Testes** → **Internal testing**
2. Crie uma nova versão
3. Upload do AAB
4. Adicione email de testadores
5. Compartilhe o link de teste

Após validar, promova para **Produção**.

#### Passo 5: Enviar para Revisão

1. Vá em **Produção**
2. Revise todas as seções (devem estar com ✓ verde)
3. Clique em **Enviar para revisão**

**Tempo de revisão:** Normalmente 1-3 dias.

---

### 8.2. Apple App Store

#### Passo 1: Criar App no App Store Connect

1. Acesse [App Store Connect](https://appstoreconnect.apple.com/)
2. Vá em **Meus apps** → **+** (botão de adicionar)
3. Selecione "Novo app"
4. Detalhes:
   - **Plataformas:** iOS
   - **Nome:** Rastec Tracker
   - **Idioma principal:** Português (Brasil)
   - **Bundle ID:** Selecione `br.com.rastecnologia`
   - **SKU:** rastec-tracker (identificador único interno)
   - **Acesso de usuário:** Acesso total
5. Criar

#### Passo 2: Preencher Informações do App

1. **Informações do app:**
   - **Nome:** Rastec Tracker
   - **Subtítulo (iOS 11+):** Rastreamento GPS em Tempo Real
   - **Categoria primária:** Negócios
   - **Categoria secundária:** Produtividade (opcional)

2. **Classificação etária:**
   - Preencha o questionário (normalmente "4+")

3. **Informações de preço e disponibilidade:**
   - **Preço:** Gratuito
   - **Disponibilidade:** Todos os países ou selecione Brasil

#### Passo 3: Preparar Build

**IMPORTANTE:** Build iOS para App Store requer:
- Xcode instalado
- Conta Apple Developer ativa
- Certificado de distribuição (Apple Distribution)
- Provisioning Profile de produção

##### Opção A: Build via Xcode (Recomendado para primeira vez)

1. Abra o workspace:
   ```bash
   open ios/Runner.xcworkspace
   ```

2. No Xcode:
   - Selecione **Any iOS Device (arm64)** como destino
   - Vá em **Product** → **Archive**
   - Aguarde build completar

3. Na janela **Organizer** que abre:
   - Selecione o archive criado
   - Clique em **Distribute App**
   - Escolha **App Store Connect**
   - Escolha **Upload**
   - Selecione as opções padrão
   - Clique em **Upload**

##### Opção B: Build via Flutter + Upload Manual

```bash
# Build iOS release
flutter build ios --release

# Build IPA via Xcode (requer configuração manual)
# Ou use: flutter build ipa --release (Flutter 3.13+)
flutter build ipa --release
```

**Arquivo gerado:**
```
build/ios/ipa/rastec_tracker.ipa
```

**Upload via Transporter:**

1. Baixe app **Transporter** na Mac App Store
2. Abra Transporter
3. Arraste o arquivo `.ipa`
4. Clique em "Deliver"

#### Passo 4: Configurar Versão na App Store Connect

1. Vá em **App Store** → **Preparação de envio do iOS**
2. Selecione a build que você enviou (pode demorar alguns minutos para processar)
3. Preencha:

   **Capturas de tela:** (pelo menos 2 para cada tamanho obrigatório)
   - **iPhone 6.9" (iPhone 16 Pro Max):** 1320x2868 ou 2868x1320
   - **iPhone 6.7" (iPhone 15 Pro Max):** 1290x2796 ou 2796x1290
   - **iPad Pro (6ª geração) 12.9":** 2048x2732 ou 2732x2048

   **Dica:** Use simuladores do Xcode e tire screenshots (⌘S).

   **Texto promocional (opcional):**
   ```
   Monitore sua frota em tempo real com o Rastec Tracker!
   ```

   **Descrição:**
   ```
   Rastec Tracker é o aplicativo oficial de rastreamento GPS da Rastec Tecnologia.

   Recursos principais:
   • Rastreamento em tempo real de veículos e ativos
   • Visualização de histórico e rotas percorridas
   • Notificações push de eventos importantes
   • Cercas geográficas (geofencing)
   • Relatórios detalhados exportáveis
   • Autenticação segura com Face ID/Touch ID
   • Interface moderna e intuitiva

   Conecte-se ao servidor Rastec e tenha controle total da sua frota na palma da mão.
   ```

   **Palavras-chave (100 caracteres):**
   ```
   rastreamento,gps,frota,veículos,traccar,localização,tempo real
   ```

   **URL de suporte:** https://rastecnologia.com.br/suporte

   **URL de marketing:** https://rastecnologia.com.br

4. **Informações de contato:**
   - Nome: Rastec Tecnologia
   - Email: suporte@rastecnologia.com.br
   - Telefone: +55 XX XXXX-XXXX

5. **Classificação etária:** Conforme questionário preenchido

6. **Informações de copyright:**
   ```
   2024 Rastec Tecnologia
   ```

#### Passo 5: Testar com TestFlight (Recomendado)

Antes de enviar para revisão, teste com TestFlight:

1. Vá em **TestFlight** (aba superior)
2. A build aparecerá automaticamente após processamento
3. Adicione testadores internos (até 100)
4. Ou crie grupo de testadores externos
5. Compartilhe link de teste

Valide funcionalidade, performance, e estabilidade.

#### Passo 6: Enviar para Revisão

1. Volte para **App Store** → **Preparação de envio**
2. Certifique-se de que todos os campos obrigatórios estão preenchidos
3. **Export Compliance:** Responda "Não" se o app não usa criptografia além do HTTPS padrão
4. **Advertising Identifier (IDFA):** Responda "Não" se não usa ads ou tracking de terceiros
5. Clique em **Adicionar para revisão**
6. Clique em **Enviar para revisão**

**Tempo de revisão:** Normalmente 1-2 dias (primeira submissão pode demorar mais).

---

## 9. Checklist Final

### Antes de Submeter

- [ ] **Código:**
  - [ ] Todos os imports atualizados (`rastec_tracker`)
  - [ ] MainActivity.kt movido para pacote correto
  - [ ] Namespace e applicationId corretos (Android)
  - [ ] Bundle IDs corretos (iOS)
  - [ ] Deep link schemes atualizados
  - [ ] URL padrão aponta para `tracker.rastecnologia.com.br`

- [ ] **Assets:**
  - [ ] Android: 5 launcher icons substituídos
  - [ ] Android: Adaptive icon foreground atualizado
  - [ ] Android: Ícone de notificação atualizado
  - [ ] iOS: 19 ícones AppIcon substituídos

- [ ] **Firebase:**
  - [ ] Projeto Firebase criado
  - [ ] Apps Android e iOS adicionados com packages corretos
  - [ ] `google-services.json` copiado para `android/app/`
  - [ ] `GoogleService-Info.plist` copiado para `ios/Runner/`
  - [ ] `flutterfire configure` executado com sucesso
  - [ ] APNs key/certificate uploadado (iOS)

- [ ] **Build:**
  - [ ] Keystore configurado corretamente (Android)
  - [ ] Code signing configurado no Xcode (iOS)
  - [ ] Versão atualizada em `pubspec.yaml`
  - [ ] `flutter clean` e `pub get` executados

- [ ] **Testes:**
  - [ ] Build Android funciona
  - [ ] Build iOS funciona
  - [ ] Login no servidor funciona
  - [ ] Deep links funcionam
  - [ ] Notificações push funcionam
  - [ ] Downloads funcionam

- [ ] **Documentação:**
  - [ ] README.md atualizado com atribuição Apache
  - [ ] CLAUDE.md atualizado
  - [ ] Comentários de modificação adicionados nos arquivos alterados

- [ ] **Lojas:**
  - [ ] App criado no Google Play Console
  - [ ] Ficha da loja preenchida (descrição, screenshots)
  - [ ] App Bundle (.aab) gerado e testado
  - [ ] App criado no App Store Connect
  - [ ] Informações do app preenchidas
  - [ ] Build (.ipa) enviado via Xcode/Transporter
  - [ ] TestFlight testado (iOS)

---

## 10. Troubleshooting

### Problemas Comuns

#### 1. Erro: "The plugins `firebase_*` use a deprecated version of the Android embedding"

**Solução:** Este é apenas um warning. Pode ignorar por enquanto. Firebase plugins serão atualizados.

---

#### 2. Erro: "Could not find google-services.json"

**Solução:**
```bash
# Verifique se o arquivo está no local correto:
ls -la android/app/google-services.json

# Se não estiver, copie novamente do Firebase Console
```

---

#### 3. Erro: "No Firebase App '[DEFAULT]' has been created"

**Solução:**
- Verifique se `Firebase.initializeApp()` está sendo chamado em `main.dart`
- Verifique se `google-services.json` e `GoogleService-Info.plist` têm os packages corretos
- Execute `flutterfire configure` novamente

---

#### 4. Erro de Build iOS: "Signing for 'Runner' requires a development team"

**Solução:**
1. Abra `ios/Runner.xcworkspace` no Xcode
2. Selecione Runner → Signing & Capabilities
3. Selecione seu Team (Apple Developer Account)
4. Certifique-se de que Bundle ID está correto: `br.com.rastecnologia`

---

#### 5. Deep Link não funciona no Android

**Solução:**
- Verifique `AndroidManifest.xml` tem o scheme correto: `rastec.android`
- Teste com:
  ```bash
  adb shell am start -W -a android.intent.action.VIEW -d "rastec.android://test" rastec.Android
  ```
- Verifique logs com: `adb logcat | grep -i flutter`

---

#### 6. Notificações Push não chegam

**Solução:**

**Android:**
- Verifique `google-services.json` tem o package correto
- Certifique-se de que Firebase Messaging está habilitado no Console
- Teste envio de notificação de teste no Firebase Console

**iOS:**
- Verifique se APNs key foi uploadado no Firebase Console
- Verifique se as capabilities de Push Notifications estão habilitadas no Xcode
- Teste no dispositivo real (push não funciona no simulador)

---

#### 7. Erro: "CocoaPods could not find compatible versions for pod"

**Solução:**
```bash
cd ios
rm Podfile.lock
rm -rf Pods/
pod install --repo-update
cd ..
```

---

#### 8. Erro: "Execution failed for task ':app:processReleaseGoogleServices'"

**Solução:**
- O `google-services.json` não corresponde ao `applicationId` no `build.gradle.kts`
- Verifique se ambos têm `rastec.Android`
- Baixe novamente o arquivo do Firebase Console com o package correto

---

#### 9. Build iOS falha com "FlutterFire CLI not found"

**Solução:**
```bash
# Instalar FlutterFire CLI
dart pub global activate flutterfire_cli

# Adicionar ao PATH permanentemente
echo 'export PATH="$PATH:$HOME/.pub-cache/bin"' >> ~/.zshrc
source ~/.zshrc

# Verificar instalação
flutterfire --version
```

---

#### 10. App rejeitado na revisão da App Store

**Motivos comuns:**

1. **Falta política de privacidade:** Adicione URL válida
2. **Descrição inadequada:** Seja claro sobre o propósito do app
3. **Screenshots desatualizadas:** Certifique-se de que refletem o app atual
4. **Funcionalidade quebrada:** Teste rigorosamente antes de submeter
5. **Informações de login necessárias:** Se o app requer login, forneça credenciais de teste para os revisores

**Solução:** Leia atentamente o feedback da Apple, corrija os problemas e resubmeta.

---

## Comandos Úteis de Referência

### Flutter

```bash
# Limpar build cache
flutter clean

# Instalar dependências
flutter pub get

# Verificar environment
flutter doctor -v

# Listar dispositivos
flutter devices

# Executar em dispositivo específico
flutter run -d <device-id>

# Build APK debug
flutter build apk --debug

# Build APK release
flutter build apk --release

# Build App Bundle (Google Play)
flutter build appbundle --release

# Build iOS
flutter build ios --release

# Build IPA (Flutter 3.13+)
flutter build ipa --release
```

### iOS (CocoaPods)

```bash
cd ios

# Instalar pods
pod install

# Atualizar pods
pod update

# Limpar cache
pod cache clean --all
rm Podfile.lock
rm -rf Pods/
pod install

cd ..
```

### Firebase

```bash
# Instalar FlutterFire CLI
dart pub global activate flutterfire_cli

# Configurar Firebase
flutterfire configure

# Listar projetos Firebase
firebase projects:list
```

### Git

```bash
# Adicionar todas as mudanças
git add .

# Commit
git commit -m "Rebrand to Rastec Tracker"

# Push
git push origin main
```

### ADB (Android Debug Bridge)

```bash
# Listar dispositivos conectados
adb devices

# Instalar APK
adb install build/app/outputs/flutter-apk/app-release.apk

# Ver logs
adb logcat

# Filtrar logs Flutter
adb logcat | grep -i flutter

# Testar deep link
adb shell am start -W -a android.intent.action.VIEW -d "rastec.android://test" rastec.Android
```

### Xcode (iOS)

```bash
# Abrir workspace
open ios/Runner.xcworkspace

# Listar simuladores
xcrun simctl list devices

# Boot simulador
xcrun simctl boot "iPhone 15 Pro"

# Testar deep link no simulador
xcrun simctl openurl booted "br.com.rastecnologia://test"
```

---

## Próximos Passos Após Deploy

1. **Monitorar Crashlytics:** Acompanhe crashes no Firebase Console
2. **Analytics:** Monitore uso e comportamento dos usuários
3. **Feedback:** Configure sistema de feedback in-app ou email
4. **Atualizações:** Planeje ciclo de atualizações (mensal, trimestral, etc.)
5. **Suporte:** Configure canal de suporte (email, chat, FAQ)
6. **Marketing:** Promova o app no site da Rastec e redes sociais

---

## Recursos Adicionais

- [Flutter Documentation](https://docs.flutter.dev/)
- [Firebase for Flutter](https://firebase.google.com/docs/flutter/setup)
- [Google Play Console Help](https://support.google.com/googleplay/android-developer/)
- [App Store Connect Help](https://developer.apple.com/help/app-store-connect/)
- [Traccar Documentation](https://www.traccar.org/documentation/)
- [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0)

---

## Contato e Suporte

Para dúvidas sobre este guia ou o processo de rebranding:

**Rastec Tecnologia**
Email: dev@rastecnologia.com.br
Website: https://rastecnologia.com.br

---

**Boa sorte com o lançamento do Rastec Tracker! 🚀**
