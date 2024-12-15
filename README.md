# Transcrição de Texto dos vídeos do YouTube

Utilizar no Google Colab.

1 - Instalar as bibliotecas abaixo:

```python
!pip install yt-dlp openai-whisper==20231106 openai
!apt install -y ffmpeg
```

2 - Insira abaixo o link completo do youtube e também a língua: Português é "pt", espanhol é "es" e inglês é "en".

```python
link='https://youtu.be/bbGaxAZ6dnY'

if 'youtu.be' in link:
  arquivo=link.split('/')[-1]+".mp3"
  arquivo_edit=link.split('/')[-1]+"_edit.mp3"
else:
  arquivo=link.split('=')[-1]+".mp3"
  arquivo_edit=link.split('=')[-1]+"_edit.mp3"

lingua='pt'
```

3 - Quando acabar de baixar o áudio, verifique se o arquivo baixado do YouTube no passo anterior se encontra dentro da pasta “content”. Exemplo de caminho: /"content/bbGaxAZ6dnY.mp3"
Para transcrever um áudio no seu computador sem relação com o YouTube, há 02 formas diferentes:
- Clique no ícone de pasta Gerenciador de arquivos no canto esquerdo, depois no ícone de seta para cima Upload de arquivos e escolha o arquivo; ou
- Clique no ícone de pasta no canto esquerdo Gerenciador de arquivos e arraste o arquivo para a área abaixo do ícone da pasta (onde a pasta sample_data está);

Rode os dois seguintes códigos:

```python
!yt-dlp -x -f bestaudio --audio-format mp3 -o "%(id)s.%(ext)s" $link
```
```python
!ffmpeg -i {arquivo} -vn -ab 320k -ar 44100 -y {arquivo_edit}
```

4 - Após a criação do arquivo .mp3, instalar as bibliotecas para convertê-lo em formato WAV.
```python
!pip install pydub
!pip install SpeechRecognition
```

5 - Transformação em arquivo de áudio formato WAV.
Em src e dst dê o caminho completo da pasta do Collab, exemplo:

src = "/content/sample_data/iHjpboBrRLw.mp3"

dst = "/content/sample_data/iHjpboBrRLw.wav"

```python
from pydub import AudioSegment
import speech_recognition as sr

# Verificar se o arquivo existe no caminho correto
src = "/content/bbGaxAZ6dnY.mp3"
dst = "/content/bbGaxAZ6dnY.wav"

# Verifique se o arquivo está presente
import os
if not os.path.exists(src):
    print(f"Erro: O arquivo {src} não foi encontrado. Verifique o caminho!")
else:
    # Converter MP3 para WAV
    sound = AudioSegment.from_mp3(src)
    sound.export(dst, format="wav")
    print("Arquivo convertido com sucesso!")

    # Carregar o arquivo WAV para reconhecimento
    file_audio = sr.AudioFile(dst)
    print("Arquivo carregado com sucesso!")
```

6 - Agora o arquivo WAV será transformado em pequenos pedaços de 30 segundos e enviados para API do Google Speech Recognition - não dá para fazer um arquivo de áudio só, pois "estoura" pelo limite de tempo do arquivo. 
O código salva a transcrição de cada segmento em arquivos .txt.

Em audio_path, cole caminho de onde o arquivo WAV foi salvo: linha "dst" acima:

dst = "/content/bbGaxAZ6dnY.wav"

```python
from pydub import AudioSegment
import speech_recognition as sr

# Inicializar o reconhecedor
r = sr.Recognizer()

# Caminho do arquivo de áudio
audio_path = "/content/bbGaxAZ6dnY.wav"
sound = AudioSegment.from_wav(audio_path)

# Dividir o áudio em segmentos de 30 segundos
segment_duration = 30 * 1000  # 30 segundos em milissegundos
segments = [sound[i:i + segment_duration] for i in range(0, len(sound), segment_duration)]

# Processar cada segmento
for idx, segment in enumerate(segments):
    segment_path = f"segmento_{idx}.wav"
    segment.export(segment_path, format="wav")  # Exportar o segmento

    with sr.AudioFile(segment_path) as source:
        audio_text = r.record(source)
        try:
            # Reconhecimento de fala
            text = r.recognize_google(audio_text, language='pt-BR')
            print(f"Segmento {idx}: {text}")

            # Salvar cada transcrição
            with open(f"transcricao_segmento_{idx}.txt", "w") as arq:
                arq.write(text)
        except sr.RequestError as e:
            print(f"Erro no segmento {idx}: {e}")
        except sr.UnknownValueError:
            print(f"Não foi possível reconhecer o segmento {idx}")
```

7 - O conteúdo de todos os arquivos transcricao_segmento_X.txt será consolidado em um único arquivo chamado transcricao_completa.txt.

```python
import os

# Lista para armazenar os nomes dos arquivos
arquivos_txt = [f"transcricao_segmento_{i}.txt" for i in range(len(segments))]

# Nome do arquivo consolidado
arquivo_final = "transcricao_completa.txt"

# Abrir o arquivo final no modo de escrita
with open(arquivo_final, "w", encoding="utf-8") as arquivo_saida:
    for arquivo in arquivos_txt:
        try:
            # Verificar se o arquivo existe
            if os.path.exists(arquivo):
                # Abrir o arquivo atual e ler o conteúdo
                with open(arquivo, "r", encoding="utf-8") as f:
                    conteudo = f.read()
                    # Escrever o conteúdo no arquivo final
                    arquivo_saida.write(conteudo + "\n")
                    print(f"Adicionado: {arquivo}")
            else:
                print(f"Arquivo não encontrado: {arquivo}")
        except Exception as e:
            print(f"Erro ao processar {arquivo}: {e}")

print(f"Todos os arquivos foram concatenados em: {arquivo_final}")
```

8 - Baixar para sua máquina o arquivo transcricao_completa.txt que foi disponibilizado ao lado, acima de todos os pedaços menores das transcrições.

9 - Delete todos os arquivos mp3, WAV e txt gerados a partir do código acima (para não consumir espação no Colab):

```python
for file in os.listdir():
    if file.endswith(".txt") or file.endswith(".wav") or file.endswith(".mp3"):
        os.remove(file)
```

Enjoy it!
