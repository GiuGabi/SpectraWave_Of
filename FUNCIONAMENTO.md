
# 🟦 Explicação da Arquitetura Matemática e Lógica (app.html)

O arquivo `app.html` (ou a página principal onde o sistema roda) é o núcleo do **SPECTRAWAVE**. Diferente de sistemas tradicionais que utilizam backend (como Python/Flask), este projeto executa todo o processamento digital de sinais (**DSP - Digital Signal Processing**) e a criptografia localmente no navegador do usuário utilizando **JavaScript puro (Vanilla JS)**.

Como o projeto não utiliza bibliotecas matemáticas prontas (como DSP.js, NumPy ou Math.js), toda a matemática de ondas, transformadas, convoluções e Fourier foi implementada manualmente.

Abaixo está a explicação detalhada de cada bloco do sistema.

---

# 1. Ajuste de Tamanho de Amostra (Base 2)

```javascript
function nextPowerOfTwo(n) {
    let p = 1;

    while (p < n) {
        p <<= 1;
    }

    return p;
}
````

O algoritmo da **Transformada Rápida de Fourier (FFT)** utilizado no projeto segue o modelo **Cooley-Tukey Radix-2**, que exige obrigatoriamente que o número de amostras seja uma potência de 2:

* 256
* 512
* 1024
* 2048
* 4096
* etc.

Caso o áudio não possua esse tamanho exato, a FFT não consegue dividir corretamente os blocos internos do sinal.

---

## Operação Matemática

A função usa o operador binário:

```javascript
p <<= 1
```

Esse operador desloca os bits uma posição para a esquerda.

Exemplo:

```text
0001 → 0010 → 0100 → 1000
```

Na prática:

```text
1 → 2 → 4 → 8 → 16
```

Ou seja, multiplicação sucessiva por 2 em nível binário.

---

## Objetivo Matemático

Encontrar:

[
2^n \geq N
]

Onde:

* (N) = quantidade original de amostras
* (2^n) = próxima potência de 2 válida

Isso garante estabilidade matemática e máxima eficiência computacional da FFT.

---

# 2. Janelamento de Hann (Hann Window)

```javascript
function applyHannWindow(signal) {
    const n = signal.length;

    for (let i = 0; i < n; i++) {
        signal[i] *= 0.5 * (
            1 - Math.cos((2 * Math.PI * i) / (n - 1))
        );
    }
}
```

Quando cortamos um trecho do áudio para análise, criamos descontinuidades bruscas no início e no fim do sinal.

Essas quebras produzem um fenômeno chamado:

# Spectral Leakage (Vazamento Espectral)

O espectro de frequências fica artificialmente espalhado.

---

## Fórmula Matemática

A Janela de Hann é definida por:

[
w[n] =
0.5 \times
\left(
1 - \cos
\left(
\frac{2\pi n}{N-1}
\right)
\right)
]

Onde:

* (N) = tamanho da janela
* (n) = posição atual da amostra

---

## O que o código faz

Cada amostra do áudio é multiplicada pela curva de Hann:

```javascript
signal[i] *= janela
```

Isso suaviza as bordas do sinal:

* início → tende a 0
* meio → permanece forte
* fim → volta para 0

---

## Resultado Matemático

A FFT passa a enxergar um sinal mais contínuo.

Isso reduz:

* ruídos artificiais
* distorções
* frequências fantasmas

Melhorando drasticamente a precisão espectral.

---

# 3. Transformada Discreta de Fourier (FFT Manual)

```javascript
function fft(real, imag) {
    const n = real.length;

    if (n <= 1) return;

    // bit reversal
}
```

A FFT converte um sinal do:

* domínio do tempo
  → para →
* domínio da frequência

---

# Fórmula da DFT

[
X[k] =
\sum_{n=0}^{N-1}
x[n]
e^{-i\frac{2\pi kn}{N}}
]

Onde:

* (x[n]) = sinal original
* (X[k]) = frequência analisada
* (e^{-i\theta}) = rotação complexa

---

# Interpretação Física

A FFT descobre:

* quais frequências existem no áudio
* intensidade de cada frequência
* harmônicos
* padrões periódicos

---

# Bit-Reversal Sorting

A FFT Cooley-Tukey reorganiza os índices em ordem binária invertida.

Exemplo:

```text
011 → 110
```

Isso permite dividir recursivamente o sinal em blocos menores.

---

# Operação Butterfly

```javascript
for (let len = 2; len <= n; len <<= 1) {

    const half = len >> 1;

    const angle = -2 * Math.PI / len;

    const wrStep = Math.cos(angle);

    const wiStep = Math.sin(angle);

}
```

Aqui ocorre a famosa operação:

# Butterfly Operation

Ela combina pares de frequências usando:

[
e^{i\theta} =
\cos(\theta) +
i\sin(\theta)
]

(Identidade de Euler)

---

## Papel Matemático

A FFT transforma uma operação:

[
O(N^2)
]

em:

[
O(N \log N)
]

Reduzindo drasticamente o custo computacional.

---

# 4. Cálculo de Magnitude

```javascript
function magnitudes(real, imag) {

    const out = new Float64Array(real.length);

    for (let i = 0; i < real.length; i++) {

        out[i] = Math.sqrt(
            real[i] * real[i] +
            imag[i] * imag[i]
        );

    }

    return out;
}
```

A FFT retorna números complexos:

[
a + bi
]

Onde:

* parte real → cossenos
* parte imaginária → senos

Mas o gráfico precisa da intensidade real da frequência.

---

# Fórmula da Magnitude

[
|X[k]| =
\sqrt{
Re(X[k])^2 +
Im(X[k])^2
}
]

Baseado diretamente no:

# Teorema de Pitágoras

---

# Interpretação Física

A magnitude representa:

* volume
* energia
* força da frequência

Quanto maior a magnitude:

→ mais presente está aquela frequência no áudio.

---

# 5. Suavização Espectral (Convolução Gaussiana)

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

O espectro bruto geralmente é muito ruidoso e pontiagudo.

Para suavizar o gráfico usamos:

# Convolução Discreta

com um:

# Kernel Gaussiano

---

# Fórmula da Convolução

[
(f * g)[n] =
\sum_{k=0}^{K-1}
f[n-k]g[k]
]

Onde:

* (f) = sinal original
* (g) = kernel gaussiano

---

## Código

```javascript
for (let k = 0; k < K; k++) {

    const idx = n - k + half;

    if (idx >= 0 && idx < N) {

        acc += signal[idx] * kernel[k];

    }
}
```

---

# O que acontece matematicamente

Cada ponto do espectro passa a ser:

* uma média ponderada
* das frequências vizinhas

Isso remove ruídos abruptos.

---

# Interpretação Física

O filtro atua como um:

# Filtro Passa-Baixa

Reduzindo altas variações instantâneas.

---

# 6. Reconstrução com Série de Fourier

```javascript
function calculateFourierSeries(
    samples,
    sampleRate,
    fundamentalHz,
    numHarmonics
)
```

A Série de Fourier demonstra matematicamente que qualquer onda periódica pode ser construída usando:

* senos
* cossenos

---

# Fórmula Geral

[
f(t) =
a_0 +
\sum_{n=1}^{\infty}
\left(
a_n\cos(n\omega t)
+
b_n\sin(n\omega t)
\right)
]

---

# Cálculo do componente DC

```javascript
let a0 = 0;

for (let i = 0; i < usedLen; i++) {
    a0 += samples[i];
}

a0 = (2 / usedLen) * a0;
```

---

## Significado Físico

(a_0) representa:

* deslocamento médio
* offset do sinal
* nível DC

---

# Harmônicos

```javascript
for (let n = 1; n <= numHarmonics; n++) {

    const omega =
        2 *
        Math.PI *
        n *
        fundamentalHz *
        t;

    an += samples[i] * Math.cos(omega);

    bn += samples[i] * Math.sin(omega);

}
```

---

# Fórmulas dos Coeficientes

[
a_n =
\frac{2}{N}
\sum x[t]\cos(n\omega_0 t)
]

[
b_n =
\frac{2}{N}
\sum x[t]\sin(n\omega_0 t)
]

---

# Objetivo Matemático

O sistema descobre:

* quais senos existem
* quais cossenos existem
* intensidade de cada harmônico

Depois reconstrói a onda original usando apenas trigonometria.

---

# 7. Pipeline de Criptografia Simétrica (WAV)

```javascript
async function runCrypto(mode) {

    const file = fileInput.files[0];

    const arrayBuffer =
        await file.arrayBuffer();

    const bytes =
        new Uint8Array(arrayBuffer);
}
```

Nesta etapa o sistema deixa de trabalhar com ondas e passa a manipular:

# Bytes Binários

diretamente na memória.

---

# Estrutura WAV

Um arquivo WAV possui:

* cabeçalho
* dados PCM

---

# Isolamento do Cabeçalho

```javascript
const headerSize = 44;

const audioData =
    bytes.subarray(headerSize);
```

Os primeiros 44 bytes contêm:

* sample rate
* canais
* formato
* metadata

Se criptografarmos isso:

→ o áudio quebra completamente.

Por isso apenas os dados PCM são alterados.

---

# Derivação da Chave SHA-256

```javascript
const encoder = new TextEncoder();

const passData =
    encoder.encode(password);

const hashBuffer =
    await crypto.subtle.digest(
        'SHA-256',
        passData
    );

const hashArray =
    new Uint8Array(hashBuffer);
```

A senha nunca é usada diretamente.

Ela é transformada em:

# Hash Criptográfico

de 256 bits.

---

# Objetivo Matemático

SHA-256 produz uma assinatura irreversível:

[
H(x)
]

onde:

* entrada pequena
  → gera →
* saída enorme pseudoaleatória

---

# Embaralhamento Temporal

```javascript
audioData.reverse();
```

O áudio inteiro é invertido temporalmente:

* começo vira fim
* fim vira começo

Isso adiciona uma camada extra de ofuscação.

---

# Criptografia XOR

```javascript
for (let i = 0; i < audioData.length; i++) {

    audioData[i] ^=
        hashArray[i % hashArray.length];

}
```

---

# Fórmula Matemática

[
C_i =
P_i
\oplus
K_{(i \mod 32)}
]

Onde:

* (P_i) = byte original
* (K_i) = byte da chave
* (C_i) = byte criptografado

---

# Propriedade Matemática do XOR

[
(A \oplus B) \oplus B = A
]

Essa propriedade faz a cifra ser:

# Simétrica

A mesma operação usada para criptografar também descriptografa.

---

# Processo de Descriptografia

O sistema:

1. aplica XOR novamente
2. desfaz a inversão temporal

Resultado:

* áudio original restaurado
* sem perdas
* sem compressão

---

# 🟦 Resumo Geral 🟦

O `app.html` executa processamento matemático avançado diretamente no navegador do usuário.

O sistema:

* converte ondas em frequências
* aplica FFT manual
* utiliza Euler e Fourier
* suaviza espectros via convolução
* reconstrói sinais usando trigonometria
* manipula bytes binários diretamente
* executa criptografia simétrica local

O projeto demonstra que áudio digital, em sua forma mais pura, é apenas uma estrutura matemática composta por:

* vetores
* funções trigonométricas
* números complexos
* álgebra booleana
* operações binárias

Esses dados podem ser:

* analisados
* filtrados
* reconstruídos
* criptografados

utilizando exclusivamente matemática aplicada e processamento digital de sinais.

```
```
