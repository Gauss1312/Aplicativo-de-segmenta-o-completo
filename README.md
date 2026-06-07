# Aplicativo de Segmentacao Completo

Projeto desenvolvido por **Gabriel F. S. de Oliveira** para a atividade de segmentacao semantica da disciplina de Deep Learning 2025.1.

Este repositorio reune, em uma unica entrega, o fluxo completo da atividade: treinamento de um modelo de segmentacao, avaliacao visual dos resultados, exportacao para TensorFlow Lite, aplicativo Android em Java e uma interface local em navegador para testar o modelo sem depender de celular, emulador ou ADB.

## Visao Geral

A proposta do projeto e aplicar segmentacao semantica em imagens de animais domesticos, separando cada pixel em uma das classes esperadas pelo modelo. Em vez de apenas classificar a imagem inteira, a rede produz uma mascara espacial, indicando quais regioes pertencem ao animal, ao fundo e a borda.

O projeto utiliza o dataset **Oxford-IIIT Pet**, que contem imagens de caes e gatos com anotacoes de segmentacao. A partir dessas mascaras, foi treinada uma arquitetura baseada em U-Net simples, escolhida por ser direta, eficiente para uma atividade academica e adequada para demonstrar o ciclo completo entre treinamento e aplicacao pratica.

A entrega tambem inclui uma alternativa de teste em `localhost`. Essa parte e importante porque permite validar rapidamente o comportamento do modelo em qualquer computador com Python configurado, antes de executar o fluxo Android.

## Objetivos da Atividade

- Treinar um modelo de segmentacao semantica usando imagens e mascaras do Oxford-IIIT Pet.
- Avaliar o desempenho por meio de metricas basicas e visualizacao de predicoes.
- Gerar um modelo utilizavel em aplicacao mobile.
- Integrar o modelo a um aplicativo Android em Java com TensorFlow Lite.
- Disponibilizar uma interface local para teste rapido da segmentacao.
- Organizar a entrega em um repositorio publico, com documentacao suficiente para reproducao e apresentacao.

## Estrutura do Projeto

```text
app_segmentation/
  README.md
  docs/
    localhost_test.md
    mobile_app_test.md
  training/
    segmentation-practice-2025-1.ipynb
    export_tflite.py
    requirements.txt
    requirements-export.txt
    README-training.md
    SimpleUNet_best.pth
  android_app/
    build.gradle
    settings.gradle
    gradlew
    gradlew.bat
    app/
      src/main/assets/
      src/main/java/com/andre/tflite/segmentation/
  local_web_app/
    server.py
    requirements.txt
    README.md
```

A pasta `training` contem o notebook, os requisitos de execucao e o script de exportacao. A pasta `android_app` contem o projeto Android. A pasta `local_web_app` contem um servidor simples em Python para testar imagens pelo navegador. A pasta `docs` guarda instrucoes mais especificas para os testes local e mobile.

## Modelo de Segmentacao

O modelo principal da entrega e uma **Simple U-Net**, treinada para receber imagens RGB redimensionadas para `128x128` pixels e devolver uma predicao pixel a pixel com tres classes:

```text
0 = Pet
1 = Fundo
2 = Borda
```

As mascaras originais do Oxford-IIIT Pet sao 1-indexadas. Durante o treinamento, elas sao ajustadas para o formato 0-indexado:

```text
target = mask_original - 1
```

No momento da inferencia, a classe final de cada pixel e definida pelo maior valor de saida entre os canais:

```text
classe(y, x) = argmax_c logits(c, y, x)
```

O pre-processamento usado no aplicativo replica o padrao do treinamento:

```text
x_norm = (x_rgb / 255 - mean) / std
mean = [0.485, 0.456, 0.406]
std  = [0.229, 0.224, 0.225]
```

Esse cuidado e necessario para que a imagem recebida pelo app tenha distribuicao semelhante a usada durante o treinamento do modelo.

## Execucao Local no Navegador

A forma mais simples de testar a entrega e usar a interface local. Ela carrega o peso `training/SimpleUNet_best.pth`, recebe uma imagem enviada pelo usuario e exibe tres resultados: imagem original, segmentacao sobreposta e mascara isolada.

### 1. Preparar o Ambiente

Na raiz do projeto:

```powershell
python -m venv .venv
.\.venv\Scripts\activate
```

Instale as dependencias:

```powershell
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu118
pip install -r training\requirements.txt
pip install -r local_web_app\requirements.txt
```

Em ambientes sem CUDA, e possivel instalar a versao CPU do PyTorch seguindo as instrucoes oficiais do pacote.

### 2. Rodar o Servidor

```powershell
python local_web_app\server.py
```

Quando o servidor estiver ativo, acesse:

```text
http://localhost:8000
```

### 3. Testar uma Imagem

Na pagina aberta no navegador:

1. Escolha uma imagem `.jpg`, `.jpeg`, `.png` ou `.webp`.
2. Prefira uma foto de cachorro ou gato, pois o modelo foi treinado com esse dominio.
3. Clique em `Segmentar`.
4. Confira a imagem original, a mascara colorida e os percentuais de cada classe.

O guia detalhado esta em:

```text
docs/localhost_test.md
```

## Execucao no Android

O aplicativo Android foi implementado em Java e utiliza TensorFlow Lite para executar a segmentacao no dispositivo. Ele permite selecionar uma imagem da galeria ou capturar uma foto pela camera, executar a inferencia e visualizar o overlay da mascara sobre a imagem.

Para abrir o projeto no Android Studio, use a pasta:

```text
android_app
```

Recomendacao de ambiente:

```text
JDK 17 ou JDK 21
```

Versoes muito recentes do JDK podem causar erro no Gradle antes mesmo da compilacao. Por isso, a recomendacao e manter uma versao estavel e compativel com o Android Studio.

O app espera encontrar o modelo TensorFlow Lite em:

```text
android_app/app/src/main/assets/model.tflite
```

Caso o arquivo ainda nao exista, ele deve ser gerado pelo script de exportacao descrito na proxima secao.

O guia de teste mobile esta em:

```text
docs/mobile_app_test.md
```

## Exportacao Para TensorFlow Lite

Depois do treinamento no notebook, o peso `SimpleUNet_best.pth` pode ser convertido para um modelo FP32 em TensorFlow Lite.

Entre na pasta de treinamento:

```powershell
cd training
```

Instale a dependencia de exportacao:

```powershell
pip install -r requirements-export.txt
```

Execute:

```powershell
python export_tflite.py
```

Por padrao, o arquivo gerado sera salvo em:

```text
../android_app/app/src/main/assets/model.tflite
```

O codigo Android foi preparado para reconhecer diferentes formatos comuns de saida de segmentacao, como:

```text
[1, H, W, C]
[1, C, H, W]
[1, H, W]
```

Isso torna a integracao um pouco mais flexivel caso o formato final exportado varie de acordo com a ferramenta de conversao.

## Observacoes Sobre Arquivos Grandes

O dataset bruto e alguns pesos auxiliares podem ocupar centenas de megabytes. Para manter o repositorio publico leve e compativel com os limites do GitHub, esses arquivos grandes nao sao versionados.

Arquivos como `training/data/`, caches do Gradle e pesos maiores, por exemplo `FCN_best.pth`, devem ser mantidos localmente ou regenerados quando necessario. O peso principal leve da U-Net, `SimpleUNet_best.pth`, foi mantido no projeto para facilitar o teste da aplicacao local.

Essa decisao evita problemas de upload, reduz o tempo de clonagem do repositorio e deixa a entrega focada no codigo-fonte, na documentacao, no notebook e no modelo principal usado pelos testes.

## Referencias

- Oxford-IIIT Pet Dataset
- TensorFlow Lite
- PyTorch
- Android Studio
- Projeto base Android: `Aandre99/Tflite-Android-Classification`
- Referencia de segmentacao: `hglps/semantic-segmentation-dl251`

## Autor

**Gabriel F. S. de Oliveira**

Projeto organizado como entrega academica de segmentacao semantica, reunindo treinamento, teste local, exportacao de modelo e aplicacao Android em um unico fluxo.
