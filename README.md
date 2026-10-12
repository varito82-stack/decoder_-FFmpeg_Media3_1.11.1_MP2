# decoder_FFmpeg_Media3_1.11.1_MP2
Decoder FFmpeg para Media3 1.11.1 (con MP2)
# Decoder FFmpeg para Media3 1.11.1 (con MP2)

Este repositorio compila, con GitHub Actions, el módulo oficial `decoder_ffmpeg` de
[AndroidX Media3](https://github.com/androidx/media) y genera un `.aar` listo para
usar en una app Android. No contiene código propio de reproducción: solo el workflow
que construye la librería.

## Para qué sirve

Muchos canales de TV en directo (por ejemplo, algunos de futbol) emiten el audio
en **MPEG Layer 2 (MP2)**, un formato para el que muchos dispositivos Android no tienen
decoder. ExoPlayer reproduce entonces el vídeo sin sonido. El decoder FFmpeg por
software cubre ese hueco: ExoPlayer lo usa solo cuando el hardware no soporta el audio.

## Qué incluye la build

| Elemento | Valor |
|---|---|
| Media3 | 1.11.1 (misma versión que la app) |
| FFmpeg | rama `release/6.0` |
| NDK | r26b (`26.1.10909125`) |
| API nativa (`ANDROID_ABI`) | 24 (igual que el `minSdk` de la app) |
| Decoders | `mp2 mp3 aac ac3 eac3` |
| ABIs | los que genera el script oficial: `armeabi-v7a`, `arm64-v8a`, `x86`, `x86_64` |

El workflow también modifica `FfmpegLibrary.java` para que el MIME `audio/mpeg-L2` se
asocie al decoder `mp2` de FFmpeg.

## Cómo generar el AAR

1. Abre la pestaña **Actions** del repositorio.
2. Elige el workflow **Compilar decoder FFmpeg para Media3 1.11.1 (con MP2)**.
3. Pulsa **Run workflow**. Tarda entre 15 y 30 minutos.
4. Al terminar, descarga el artefacto `decoder-ffmpeg-aar` (un zip con
   `decoder-ffmpeg-release.aar`).

En el log del paso *Reunir y comprobar el AAR* se listan las librerías nativas del AAR
y los decoders que se detectan dentro de `libffmpegJNI.so`.

## Cómo usarlo en la app

1. Copia el `.aar` a `app/libs/`.
2. En `app/build.gradle.kts`:

   ```kotlin
   implementation(files("libs/decoder-ffmpeg-release.aar"))
   ```

   Si la app tenía otra librería FFmpeg para Media3 (por ejemplo, la de Jellyfin o
   NextLib), hay que **quitarla**: ambas contienen las mismas clases
   (`androidx.media3.decoder.ffmpeg`) y el proyecto no compila con las dos.
3. Mantén `media3-exoplayer` en la misma versión (1.11.1).
4. Crea el `DefaultRenderersFactory` con las extensiones activadas como respaldo:

   ```kotlin
   val renderersFactory = DefaultRenderersFactory(context)
       .setExtensionRendererMode(DefaultRenderersFactory.EXTENSION_RENDERER_MODE_ON)
   ```

   `ON` usa FFmpeg solo cuando MediaCodec no soporta el formato. `PREFER` lo usaría
   antes que el hardware para todos los formatos incluidos, con más carga de CPU.
   El módulo se carga solo, sin tocar más código.

## Cómo comprobar que funciona

Reproduce un canal con audio MP2. En el log de la app, la pista de audio debe aparecer
como soportada y seleccionada, y el canal debe sonar:

```
TRACK [AUDIO]: mime=audio/mpeg-L2 ... soportada=true seleccionada=true
```

## Si cambias de versión de Media3

El módulo debe coincidir con la versión de `media3-exoplayer` de la app, porque la API
de las extensiones es inestable. Para actualizar, cambia `MEDIA3_TAG` en
`.github/workflows/build-ffmpeg-decoder.yml` y vuelve a ejecutar el workflow.
Si falla, revisa las instrucciones de `libraries/decoder_ffmpeg/README.md` en esa
versión de Media3, por si han cambiado la versión de FFmpeg, el NDK o los argumentos
del script.

## Si el workflow falla

- Abre la ejecución, entra en el trabajo `build` y copia el log del paso en rojo.
- Fallos típicos: nombre del módulo de Gradle distinto, versión de CMake o NDK no
  compatible, o un cambio en los argumentos de `build_ffmpeg.sh`.

## Licencias

- El código de Media3 y de su módulo FFmpeg es **Apache 2.0**.
- **FFmpeg** es LGPL si se compila sin opciones GPL. Revisa en el log de compilación
  (la salida de `configure`) que la licencia indicada es LGPL antes de distribuir.
- Si distribuyes una app que incluye esta librería, normalmente tendrás que incluir
  los avisos de licencia de Media3 y FFmpeg y ofrecer el código fuente de FFmpeg que
  usaste (versión `release/6.0` más los parámetros de este workflow).
- Esto es información general, no asesoría legal. Consulta a alguien con conocimientos
  legales antes de publicar la app.
