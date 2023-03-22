# SecretMsg
Python

from PIL import Image
import random
import hashlib

# Definir a chave de criptografia
chave = "chave de criptografia"

# Abrir a imagem que deseja esconder a mensagem
img = Image.open('imagem.png')

# Converter a imagem para o modo RGB
img = img.convert('RGB')

# Definir a mensagem que deseja esconder
mensagem = "Mensagem secreta"

# Gerar um salt aleatório para adicionar segurança à criptografia
salt = hashlib.sha256(str(random.randint(0, 100000)).encode('utf-8')).hexdigest()[:16]

# Criptografar a mensagem usando a chave e o salt
mensagem_criptografada = hashlib.sha256((chave + salt).encode('utf-8')).hexdigest()[:16] + mensagem

# Converter a mensagem criptografada em uma sequência de bits
bits = ''.join(format(ord(c), '08b') for c in mensagem_criptografada)

# Obter o tamanho da mensagem
tamanho_mensagem = len(bits)

# Obter as dimensões da imagem
largura, altura = img.size

# Verificar se a mensagem pode ser armazenada na imagem
if tamanho_mensagem > largura * altura:
    raise ValueError("A mensagem é muito grande para a imagem.")

# Escolher aleatoriamente um conjunto de pixels para esconder a mensagem
pixels_escondidos = set()
while len(pixels_escondidos) < tamanho_mensagem:
    linha = random.randint(0, altura-1)
    coluna = random.randint(0, largura-1)
    if (coluna, linha) not in pixels_escondidos:
        pixels_escondidos.add((coluna, linha))

# Esconder a mensagem nos pixels escolhidos
for index, (coluna, linha) in enumerate(pixels_escondidos):
    r, g, b = img.getpixel((coluna, linha))
    r = r & ~1 | int(bits[index * 3])
    g = g & ~1 | int(bits[index * 3 + 1])
    b = b & ~1 | int(bits[index * 3 + 2])
    img.putpixel((coluna, linha), (r, g, b))

# Salvar a imagem com a mensagem escondida
img.save('imagem_com_mensagem.png')

# Salvar o salt usado na criptografia em um arquivo
with open('salt.txt', 'w') as f:
    f.write(salt)
