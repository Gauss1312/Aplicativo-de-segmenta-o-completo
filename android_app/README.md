# Android App Java

Aplicativo Android em Java da atividade de segmentacao semantica.

Abra esta pasta no Android Studio:

```text
android_app
```

Antes de rodar, substitua `app/src/main/assets/model.tflite` pelo modelo FP32 exportado em `../training/export_tflite.py`.

Pre-requisito de build: use JDK 17 ou JDK 21 no Android Studio/Gradle. O JDK 26 pode falhar com a versao do Gradle configurada neste projeto.

Fluxo no app:

1. Tirar foto ou carregar imagem.
2. Tocar em `Segmentar`.
3. Visualizar a imagem original na secao `Original`.
4. Visualizar a predicao na secao `Segmentacao`, com a mascara colorida sobreposta a imagem.

Arquivos principais:

- `app/src/main/java/com/andre/tflite/segmentation/MainActivity.java`
- `app/src/main/java/com/andre/tflite/segmentation/Segmenter.java`
- `app/src/main/java/com/andre/tflite/segmentation/SegmentationResult.java`
- `app/src/main/res/layout/activity_main.xml`

Guia detalhado de teste:

```text
../docs/mobile_app_test.md
```
