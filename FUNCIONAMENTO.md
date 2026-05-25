# 🟦 Explicação Completa da Arquitetura Matemática, Física e Lógica do Sistema SPECTRAWAVE

O arquivo principal do sistema (`app.html`) é o cérebro do projeto **SPECTRAWAVE**.
Ele controla:

* leitura do áudio;
* processamento matemático;
* análise espectral;
* transformadas de Fourier;
* filtragem;
* reconstrução de sinais;
* criptografia;
* visualização gráfica.

Tudo acontece diretamente no navegador do usuário usando apenas:

* HTML
* CSS
* JavaScript puro (Vanilla JS)

Sem servidores externos.

---

# 🟦 VISÃO GERAL DO FUNCIONAMENTO

Quando o usuário envia um áudio, o sistema executa várias etapas matemáticas e computacionais.

Fluxo completo:

```text
Áudio WAV
   ↓
Leitura binária do arquivo
   ↓
Extração das amostras PCM
   ↓
Pré-processamento
   ↓
Janelamento de Hann
   ↓
FFT (Transformada de Fourier)
   ↓
Cálculo de magnitudes
   ↓
Convolução Gaussiana
   ↓
Visualização do espectro
   ↓
Análise harmônica
   ↓
Reconstrução Fourier
   ↓
Criptografia XOR + SHA-256
```

---

# 🟦 O QUE É UM ÁUDIO DIGITAL?

Antes de entender o código, precisamos entender o que é um áudio digital.

---

# Som na Física

Fisicamente, som é:

# Uma onda mecânica longitudinal

O ar vibra e cria compressões.

Essas compressões chegam aos nossos ouvidos.

---

# O que o computador vê?

O computador NÃO entende “som”.

Ele vê apenas:

# Números

Exemplo:

```text
[0.1, 0.4, 0.7, -0.2, -0.9]
```

Cada número representa:

# A amplitude da onda sonora em um instante do tempo

---

# PCM (Pulse Code Modulation)

Arquivos WAV usam:

# PCM

O áudio é armazenado como milhares de amostras por segundo.

Exemplo:

```text
44100 Hz
```

Significa:

# 44.100 amostras por segundo

Ou seja:

o computador “fotografa” a onda sonora 44.100 vezes por segundo.

---

# 🟦 COMO O SISTEMA LÊ O ÁUDIO

```javascript
const file = fileInput.files[0];

const arrayBuffer =
    await file.arrayBuffer();

const bytes =
    new Uint8Array(arrayBuffer);
```

---

# O que acontece aqui?

---

## 1. fileInput.files[0]

Pega o arquivo enviado pelo usuário.

Exemplo:

```text
musica.wav
```

---

## 2. arrayBuffer()

Transforma o arquivo em memória binária.

O navegador lê:

```text
010101010101010
```

---

## 3. Uint8Array

Converte os dados em:

# Vetor de bytes

Cada posição guarda um número de:

```text
0 → 255
```

Exemplo:

```text
[82, 73, 70, 70, 120, 35...]
```

---

# 🟦 ESTRUTURA INTERNA DO WAV

Um arquivo WAV possui duas partes:

```text
[ CABEÇALHO ][ DADOS DE ÁUDIO ]
```

---

# Cabeçalho (44 bytes)

Contém:

* sample rate
* quantidade de canais
* profundidade de bits
* tipo de codificação

---

# PCM Data

Aqui ficam:

# As amostras reais do áudio

---

# Código

```javascript
const headerSize = 44;

const audioData =
    bytes.subarray(headerSize);
```

---

# O que isso faz?

Ignora os 44 primeiros bytes.

Pega apenas:

# Os dados sonoros reais

---

# Por que isso é importante?

Se o cabeçalho for alterado:

* o arquivo quebra;
* o player não entende o áudio;
* o WAV fica corrompido.

Por isso apenas os dados PCM são processados.

---

# 🟦 AJUSTE PARA POTÊNCIA DE 2

```javascript
function nextPowerOfTwo(n) {

    let p = 1;

    while (p < n) {
        p <<= 1;
    }

    return p;
}
```

---

# O que essa função faz?

A FFT precisa obrigatoriamente que o tamanho do vetor seja:

# Potência de 2

Exemplos válidos:

```text
256
512
1024
2048
4096
```

---

# Por que?

Porque o algoritmo FFT divide o sinal em metades continuamente.

Exemplo:

```text
1024
↓
512 + 512
↓
256 + 256
↓
128 + 128
```

Se o número não for potência de 2:

a divisão recursiva quebra.

---

# Operador Binário

```javascript
p <<= 1
```

Significa:

# Deslocar bits para esquerda

---

# Exemplo Binário

```text
0001 → 0010
```

Decimalmente:

```text
1 → 2
```

Outro:

```text
0010 → 0100
```

Decimal:

```text
2 → 4
```

---

# Matemática Envolvida

A função busca:

2^n \geq N

Onde:

* (N) = tamanho original do áudio
* (2^n) = próxima potência de 2

---

# Objetivo

Garantir:

* estabilidade matemática;
* compatibilidade com FFT;
* máxima velocidade computacional.

---

# 🟦 JANELAMENTO DE HANN

```javascript
function applyHannWindow(signal) {

    const n = signal.length;

    for (let i = 0; i < n; i++) {

        signal[i] *=
            0.5 * (
                1 -
                Math.cos(
                    (2 * Math.PI * i) /
                    (n - 1)
                )
            );
    }
}
```

---

# Problema Físico

O áudio real é contínuo.

Mas o computador corta apenas um pedaço dele.

Exemplo:

```text
onda infinita
↓↓↓↓↓↓↓↓↓↓
[ trecho analisado ]
```

Esse corte cria:

# Descontinuidade

---

# Consequência

A FFT interpreta isso como frequências falsas.

Fenômeno:

# Spectral Leakage

(Vazamento espectral)

---

# Fórmula da Janela de Hann

w[n]=0.5\left(1-\cos\left(\frac{2\pi n}{N-1}\right)\right)

---

# O que ela faz?

Ela suaviza:

* começo do sinal;
* final do sinal.

Transformando:

```text
██████████
```

em:

```text
▁▃▅▇█▇▅▃▁
```

---

# Efeito Matemático

Reduz:

* ruídos artificiais;
* frequências fantasmas;
* distorções espectrais.

---

# 🟦 FFT — TRANSFORMADA RÁPIDA DE FOURIER

```javascript
function fft(real, imag) {

    const n = real.length;

    if (n <= 1) return;
}
```

---

# O que a FFT faz?

Transforma o áudio do:

```text
Domínio do Tempo
```

para:

```text
Domínio da Frequência
```

---

# Tempo

Mostra:

```text
amplitude vs tempo
```

---

# Frequência

Mostra:

```text
quais frequências existem
```

---

# Exemplo

Uma música possui:

* graves;
* médios;
* agudos.

A FFT separa tudo isso.

---

# Fórmula da DFT

X[k]=\sum_{n=0}^{N-1}x[n]e^{-i\frac{2\pi kn}{N}}

---

# Explicação de cada parte

| Símbolo        | Significado           |
| -------------- | --------------------- |
| (x[n])         | sinal original        |
| (X[k])         | frequência encontrada |
| (e^{-i\theta}) | rotação complexa      |
| (N)            | número de amostras    |

---

# Números Complexos

A FFT trabalha com:

```text
a + bi
```

Onde:

* (a) → parte real
* (b) → parte imaginária

---

# Por que números complexos?

Porque ondas possuem:

* fase;
* direção;
* rotação matemática.

---

# Identidade de Euler

e^{i\theta}=\cos(\theta)+i\sin(\theta)

---

# Significado Físico

Toda onda pode ser representada por:

* senos;
* cossenos;
* rotações circulares.

---

# 🟦 BIT REVERSAL

A FFT reorganiza índices.

Exemplo:

```text
011 → 110
```

---

# Por que?

Para dividir o problema em partes menores.

A FFT usa:

# Divide and Conquer

(dividir para conquistar)

---

# Complexidade Computacional

DFT tradicional:

O(N^2)

FFT:

O(N\log N)

---

# Resultado

Processamento extremamente mais rápido.

---

# 🟦 BUTTERFLY OPERATION

```javascript
const angle =
    -2 * Math.PI / len;

const wrStep =
    Math.cos(angle);

const wiStep =
    Math.sin(angle);
```

---

# O que acontece aqui?

A FFT combina frequências usando rotações trigonométricas.

Esse processo chama-se:

# Butterfly Operation

---

# Papel da trigonometria

O sistema usa:

* seno;
* cosseno;
* Euler;
* números complexos.

Para decompor a onda original.

---

# 🟦 CÁLCULO DE MAGNITUDE

```javascript
out[i] = Math.sqrt(
    real[i] * real[i] +
    imag[i] * imag[i]
);
```

---

# Problema

A FFT retorna números complexos.

Mas o gráfico precisa mostrar:

# Intensidade real da frequência

---

# Fórmula

|X[k]|=\sqrt{Re(X[k])^2+Im(X[k])^2}

---

# Matemática Utilizada

Baseado no:

# Teorema de Pitágoras

---

# Interpretação Física

Magnitude significa:

* força;
* energia;
* presença da frequência.

---

# Exemplo

Se:

```text
440 Hz
```

possui magnitude alta:

então o som possui forte presença dessa frequência.

(440 Hz = nota Lá)

---

# 🟦 CONVOLUÇÃO GAUSSIANA

```javascript
const GAUSSIAN_KERNEL = [
    0.0625,
    0.125,
    0.1875,
    0.25,
    0.1875,
    0.125,
    0.0625
];
```

---

# Problema

O espectro FFT bruto é muito pontiagudo.

Exemplo:

```text
| | || ||| |||||| || |
```

Muito ruído visual.

---

# Solução

Aplicar:

# Convolução

com:

# Kernel Gaussiano

---

# Fórmula

(f*g)[n]=\sum_{k=0}^{K-1}f[n-k]g[k]

---

# O que é convolução?

Misturar informações vizinhas.

---

# Interpretação simples

Cada ponto do gráfico passa a considerar:

* seus vizinhos;
* médias ponderadas;
* suavização local.

---

# Resultado

O gráfico fica:

* mais limpo;
* mais suave;
* mais estável.

---

# Filtro Passa-Baixa

O kernel gaussiano atua como:

# Low Pass Filter

Ele reduz mudanças bruscas.

---

# 🟦 SÉRIE DE FOURIER

```javascript
function calculateFourierSeries(
    samples,
    sampleRate,
    fundamentalHz,
    numHarmonics
)
```

---

# Ideia Central

Fourier descobriu algo revolucionário:

# Qualquer onda periódica pode ser construída usando senos e cossenos

---

# Fórmula Geral

f(t)=a_0+\sum_{n=1}^{\infty}(a_n\cos(n\omega t)+b_n\sin(n\omega t))

---

# Significado

Uma onda complexa:

```text
████▓▒▒
```

pode ser reconstruída usando ondas simples.

---

# Harmônicos

Cada seno adicional é chamado:

# Harmônico

---

# Exemplo Musical

Se a frequência fundamental é:

```text
440 Hz
```

os harmônicos são:

```text
880 Hz
1320 Hz
1760 Hz
```

---

# Coeficientes

```javascript
an += samples[i] * Math.cos(omega);

bn += samples[i] * Math.sin(omega);
```

---

# Fórmulas

a_n=\frac{2}{N}\sum x[t]\cos(n\omega_0 t)

b_n=\frac{2}{N}\sum x[t]\sin(n\omega_0 t)

---

# Objetivo Matemático

Descobrir:

* quais senos existem;
* quais cossenos existem;
* intensidade de cada um.

---

# 🟦 COMPONENTE DC

```javascript
a0 += samples[i];
```

---

# O que é DC?

É o valor médio da onda.

---

# Interpretação Física

Representa:

* offset;
* deslocamento vertical;
* média do sinal.

---

# 🟦 CRIPTOGRAFIA SHA-256

```javascript
const hashBuffer =
    await crypto.subtle.digest(
        'SHA-256',
        passData
    );
```

---

# O que é SHA-256?

É uma função matemática criptográfica.

Ela transforma:

```text
senha pequena
```

em:

```text
hash gigante pseudoaleatório
```

---

# Exemplo

```text
"senha123"
↓
A94F239A...
```

---

# Propriedades

SHA-256 é:

* irreversível;
* determinístico;
* extremamente complexo matematicamente.

---

# 🟦 XOR

```javascript
audioData[i] ^=
    hashArray[
        i % hashArray.length
    ];
```

---

# O que é XOR?

Operação booleana binária.

---

# Tabela XOR

| A | B | Resultado |
| - | - | --------- |
| 0 | 0 | 0         |
| 0 | 1 | 1         |
| 1 | 0 | 1         |
| 1 | 1 | 0         |

---

# Fórmula

C_i=P_i\oplus K_{(i\bmod 32)}

---

# Significado

| Símbolo | Significado        |
| ------- | ------------------ |
| (P_i)   | byte original      |
| (K_i)   | byte da chave      |
| (C_i)   | byte criptografado |

---

# Propriedade Matemática

(A\oplus B)\oplus B=A

---

# Consequência

A mesma operação:

* criptografa;
* descriptografa.

---

# 🟦 INVERSÃO TEMPORAL

```javascript
audioData.reverse();
```

---

# O que isso faz?

Inverte completamente o áudio.

---

# Exemplo

Antes:

```text
[1,2,3,4]
```

Depois:

```text
[4,3,2,1]
```

---

# Efeito Físico

O áudio toca:

* de trás para frente;
* completamente embaralhado.

---

# 🟦 PIPELINE COMPLETO DE CRIPTOGRAFIA

## Criptografia

```text
Áudio original
   ↓
SHA-256 da senha
   ↓
Inversão temporal
   ↓
XOR binário
   ↓
Áudio criptografado
```

---

## Descriptografia

```text
Áudio criptografado
   ↓
XOR novamente
   ↓
Reverse novamente
   ↓
Áudio original restaurado
```

---

# 🟦 O QUE O PROJETO DEMONSTRA

O SPECTRAWAVE demonstra que:

# Áudio é matemática pura

O sistema utiliza:

* álgebra linear;
* trigonometria;
* cálculo numérico;
* DSP;
* números complexos;
* Fourier;
* convolução;
* lógica binária;
* criptografia;
* processamento espectral.

---

# 🟦 RESUMO FINAL

O `app.html` funciona como:

# Um laboratório matemático e físico de áudio digital

Ele:

* lê sinais sonoros;
* transforma ondas em frequências;
* aplica FFT manual;
* usa Euler e Fourier;
* reconstrói ondas;
* suaviza espectros;
* calcula magnitudes;
* manipula bytes binários;
* criptografa dados localmente.

Tudo isso utilizando exclusivamente:

* matemática aplicada;
* física ondulatória;
* processamento digital de sinais;
* álgebra booleana;
* computação binária.
