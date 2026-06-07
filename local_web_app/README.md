# Local Web App

Versao local em navegador para testar a segmentacao sem Android Studio, ADB, celular ou emulador.

Esta aplicacao nao substitui o APK Android. Ela usa o mesmo modelo treinado (`training/SimpleUNet_best.pth`) e permite validar visualmente:

- upload de imagem;
- predicao da mascara;
- overlay da mascara na imagem original;
- percentuais das classes `Pet`, `Fundo` e `Borda`.

## Como Rodar

Use o mesmo ambiente Python em que o notebook foi executado.

Instale as dependencias:

```powershell
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu118
pip install -r training\requirements.txt
pip install -r local_web_app\requirements.txt
```

Confirme que existe:

```text
training/SimpleUNet_best.pth
```

Inicie o servidor:

```powershell
cd app_segmentation
python local_web_app\server.py
```

Abra no navegador:

```text
http://localhost:8000
```

Depois:

1. Clique em escolher arquivo.
2. Selecione uma imagem `.jpg`, `.jpeg`, `.png` ou `.webp`.
3. Clique em `Segmentar`.
4. Confira a imagem original, a imagem com overlay e a mascara.

## Dependencias

O ambiente precisa ter:

```text
torch
torchvision
pillow
```

Se precisar instalar:

```bash
pip install -r local_web_app/requirements.txt
```

## Peso Usado

Por padrao, o servidor procura:

```text
training/SimpleUNet_best.pth
```

Para usar outro arquivo:

```powershell
python local_web_app\server.py --weights caminho\para\SimpleUNet_best.pth
```
