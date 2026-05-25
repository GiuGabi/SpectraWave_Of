# 🌊 SPECTRAWAVE - DSP & Criptografia de Áudio

SPECTRAWAVE é uma aplicação web focada em **Processamento Digital de Sinais (DSP)** e **Segurança da Informação**, desenvolvida para atuar diretamente sobre arquivos de áudio digital (formato WAV PCM). 

O grande diferencial deste projeto é a **implementação matemática manual** (Vanilla JavaScript) de conceitos complexos de física e engenharia, sem a dependência de bibliotecas externas para o cálculo das transformadas ou filtros.

## 🚀 Funcionalidades Principais

### 1. Análise Espectral Matemática
Conversão de áudio do domínio temporal para o domínio da frequência, permitindo a visualização de oscilações e harmônicas.
* **Transformada Discreta de Fourier (FFT):** Implementação manual do algoritmo *Cooley-Tukey Radix-2*.
* **Convolução Gaussiana:** Filtro passa-baixa implementado via convolução discreta para suavização do espectro e eliminação de ruídos.
* **Série Trigonométrica de Fourier:** Cálculo numérico dos coeficientes harmônicos para reconstrução aproximada do sinal periódico.

### 2. Pipeline Criptográfico Simétrico (WAV)
Criptografia executada diretamente sobre os bytes do áudio (bloco DATA do RIFF/WAV), com processamento 100% local no navegador.
* **Parsing de Estrutura RIFF:** Leitura manual e extração dos dados PCM.
* **Derivação de Chave:** Geração de chave pseudoaleatória de 256 bits via **SHA-256** a partir de uma senha numérica de 4 dígitos.
* **Embaralhamento Físico:** Inversão temporal dos frames (Reverse Frames) como camada extra de ofuscação.
* **Cifra XOR Byte a Byte:** Operação lógica simétrica entre os dados PCM e a chave gerada.

## 🛠️ Tecnologias Utilizadas
* **HTML5 & CSS3:** Interface responsiva e design imersivo.
* **JavaScript (Vanilla):** Toda a lógica de processamento vetorial, algoritmos DSP e manipulação de arrays binários (ArrayBuffer/Uint8Array) construída do zero.
* **Web Crypto API:** Utilizada exclusivamente para a geração do hash seguro (SHA-256).
* **KaTeX:** Renderização de equações matemáticas avançadas e fórmulas de física na interface.

## ⚙️ Como Executar
O projeto não requer instalação de pacotes (Node.js, npm, etc.) ou servidores locais. Toda a arquitetura foi pensada para rodar *Client-Side*.

1. Clone o repositório:
   ```bash
   git clone [https://github.com/SeuUsuario/spectrawave-dsp.git](https://github.com/SeuUsuario/spectrawave-dsp.git)
