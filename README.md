# THM_W1se
Challenge Xor TryHackMe

Analise de vulnerabilidade em Xor.

---
# XOR Cryptanalysis Challenge — Write-up

Este repositório contém a solução de um desafio de criptografia baseado em **Ataque de Texto em Claro Conhecido (Known Plaintext Attack)** aplicado a uma implementação insegura de **XOR**.

O objetivo do desafio é explorar uma falha criptográfica em um serviço de rede que utiliza uma chave curta e repetitiva para mascarar informações sensíveis.

---

## Descrição do Desafio

O servidor (`server.py`) opera na porta **1337** e executa o seguinte fluxo:

1. Gera uma chave aleatória de **5 caracteres alfanuméricos**
2. Aplica a operação **XOR** entre uma flag falsa (`THM{thisisafakeflag}`) e a chave
3. Envia o resultado codificado em **hexadecimal** ao cliente
4. Solicita a chave de criptografia  
   - Caso a chave passada esteja correta, o servidor retorna a **flag real** armazenada em `flag.txt`

---

## Tecnologias Utilizadas

- **Python 3** — Automação do bruteforce e análise do código
- **Pwntools** — Biblioteca para exploração de serviços de rede
- **Criptografia XOR** — Cifra simétrica reversível

---

## Lógica de Exploração

A vulnerabilidade explorada está na própria natureza da operação XOR.

Sabendo que: KPA -> known-plaintext attack (ataque de texto plano conhecido)

- Temos o **Texto Criptografado (C)** - 
- Conhecemos parte do **Texto Original (P)**  
  (todas as flags seguem o padrão `THM{`)

Podemos recuperar a **Chave (K)** usando:

\[
P \oplus K = C \Rightarrow K = P \oplus C
\]

Utilizando os **4 primeiros bytes** do texto hexadecimal recebido, é possível extrair os **4 primeiros caracteres da chave**.

Pois sabemos que o padrão da chave 'THM{'

Como a chave possui exatamente **5 caracteres**, o ataque se resume a um **brute force controlado** apenas sobre o último caractere.

---

## Script de Resolução (`xor.py`)

```python
import string
from pwn import *

# Hexadecimal recebido do servidor
enc_flag = bytes.fromhex(
    "0231271f276718060a2313011e2523224d090f3417171857363a35130c02240d135422240125162a"
)

part_flag = b'THM{'

# Recupera os 4 primeiros caracteres da chave
part_key = xor(enc_flag, part_flag)[:4]

# Brute force do 5º caractere
charset = string.ascii_letters + string.digits
for c in charset:
    key = part_key + c.encode()
    dec_flag = xor(enc_flag, key).decode()

    if dec_flag.endswith('}'):
        print(f"Chave Encontrada: {key.decode()}")
        print(f"Flag Decodificada: {dec_flag}")
