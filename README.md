# 🌊 SPECTRAWAVE - DSP & Criptografia de Áudio

![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-success?style=for-the-badge)

## 📌 Sobre o Projeto

Este projeto foi desenvolvido como uma aplicação web focada em **Processamento Digital de Sinais (DSP)** e **Segurança da Informação**, explorando na prática conceitos avançados de Engenharia da Computação:

* Criptografia Simétrica;
* Processamento e manipulação binária de áudio (WAV PCM);
* Análise Espectral e oscilações;
* Transformadas matemáticas aplicadas a sinais sonoros;
* Processamento 100% *Client-Side* (sem envio de dados para servidores).

A aplicação permite:
* 🔒 Criptografar arquivos de áudio WAV localmente;
* 🔓 Descriptografar arquivos de áudio utilizando a mesma senha;
* 📊 Analisar a frequência dominante e o espectro do áudio;
* 📉 Suavizar sinais via convolução gaussiana;
* 🔄 Reconstruir aproximações de sinais utilizando a Série de Fourier.

O grande diferencial deste sistema é a **implementação matemática manual** (Vanilla JavaScript). Não foram utilizadas bibliotecas prontas (como DSP.js ou Math.js) para o cálculo das transformadas, demonstrando o funcionamento interno dos algoritmos.

---

# ⚙️ Como o Projeto Funciona

## 🎵 Criptografia de Áudio

O sistema atua diretamente sobre a estrutura de bytes do arquivo. O fluxo de criptografia segue as etapas:

1. **Leitura WAV:** Lê o arquivo identificando a estrutura RIFF;
2. **Conversão:** Converte os bytes do bloco DATA do áudio em amostras numéricas;
3. **Reverse Frames:** Inverte os frames do áudio temporalmente como uma camada extra de embaralhamento físico;
4. **Derivação de Chave:** Gera uma chave criptográfica baseada em uma senha do usuário (4 dígitos) utilizando algoritmo **SHA-256**;
5. **Operação Lógica:** Aplica operação **XOR** byte a byte entre os dados do áudio e a chave gerada;
6. **Reconstrução:** Remonta o cabeçalho e reconstrói o arquivo WAV criptografado para download.

A descriptografia utiliza exatamente o mesmo processo e simetria matemática, exigindo a senha original criada pelo usuário.

---

## 📈 Análise Espectral Matemática

O módulo de processamento de sinais converte o áudio do domínio temporal para o domínio da frequência. Toda a análise é feita utilizando algoritmos próprios:

* **FFT (Transformada Discreta de Fourier):** Implementação manual do algoritmo *Cooley-Tukey Radix-2* para revelar as frequências invisíveis ao ouvido humano.
* **Convolução Gaussiana:** Aplicação de um filtro passa-baixa discreto, criando um *kernel* para suavizar o espectro e reduzir ruídos abruptos.
* **Série Trigonométrica de Fourier:** Cálculo numérico dos coeficientes harmônicos para reconstruir visualmente a onda periódica.

---

# 🛠️ Tecnologias Utilizadas

## Frontend & Interface
* **HTML5** e **CSS3** (Interface imersiva e responsiva)
* **KaTeX** (Renderização de equações matemáticas avançadas na interface)

## Processamento Lógico e Matemático
* **Vanilla JavaScript** (Lógica vetorial e DSP implementada do zero)
* **Web Crypto API** (Para geração do hash seguro SHA-256)
* **Canvas API** (Para plotagem em tempo real dos gráficos de onda e espectro)

---

# 📂 Estrutura do Projeto

```bash
📁 spectrawave
│
├── index.html       # Capa de apresentação e introdução física
├── app.html         # Sistema principal (DSP, Gráficos e Criptografia)
├── README.md        # Documentação do projeto
