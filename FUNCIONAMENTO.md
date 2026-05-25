# 🟦 Arquitetura Completa do SPECTRAWAVE

## Explicação Matemática, Física e Computacional do Sistema

O **SPECTRAWAVE** é um sistema de análise e criptografia de áudio que funciona inteiramente no navegador usando apenas:

* HTML
* CSS
* JavaScript puro

O sistema não depende de servidores externos.
Todo o processamento acontece localmente no computador do usuário.

---

# 🟦 O QUE O SISTEMA FAZ

Quando um áudio é enviado, o sistema:

1. lê o arquivo WAV;
2. extrai os dados binários;
3. converte o áudio em amostras numéricas;
4. aplica processamento matemático;
5. transforma o áudio em frequências;
6. analisa o espectro sonoro;
7. reconstrói ondas usando Fourier;
8. criptografa o áudio.

---

# 🟦 FLUXO COMPLETO DO SISTEMA

```text
Áudio WAV
   ↓
Leitura binária do arquivo
   ↓
Extração das amostras PCM
   ↓
Pré-processamento do sinal
   ↓
Janelamento de Hann
   ↓
FFT (Transformada de Fourier)
   ↓
Cálculo das magnitudes
   ↓
Convolução Gaussiana
   ↓
Visualização do espectro
   ↓
Análise harmônica
   ↓
Reconstrução por Fourier
   ↓
Criptografia SHA-256 + XOR
```

---

# 🟦 O QUE É UM ÁUDIO DIGITAL

## Física do Som

Na física, som é:

# Uma onda mecânica longitudinal

O ar vibra e gera regiões de compressão e rarefação.

Essas vibrações chegam aos nossos ouvidos e são interpretadas pelo cérebro.

---

# 🟦 COMO O COMPUTADOR ENXERGA O SOM

O computador não entende “música” ou “voz”.

Ele trabalha apenas com:

# números

Exemplo:

```text
[0.1, 0.4, 0.7, -0.2, -0.9]
```

Cada número representa:

# a amplitude da onda sonora em um instante específico do tempo

---

# 🟦 PCM — PULSE CODE MODULATION

Arquivos WAV utilizam:

# PCM (Pulse Code Modulation)

O áudio é armazenado como milhares de amostras por segundo.

Exemplo:

```text
44100 Hz
```

Isso significa:

# 44.100 amostras por segundo

Ou seja:

o computador “fotografa” a onda sonora 44.100 vezes por segundo.

---

# 🟦 LEITURA DO ARQUIVO WAV

## Código

```javascript
const file = fileInput.files[0];

const arrayBuffer =
    await file.arrayBuffer();

const bytes =
    new Uint8Array(arrayBuffer);
```

---

# 🟦 O QUE CADA LINHA FAZ

## 1. Captura do arquivo

```javascript
const file = fileInput.files[0];
```

Pega o arquivo enviado pelo usuário.

Exemplo:

```text
musica.wav
```

---

## 2. Conversão para memória binária

```javascript
await file.arrayBuffer();
```

Transforma o arquivo em dados binários armazenados na memória.

O navegador passa a enxergar algo parecido com:

```text
010101010101010...
```

---

## 3. Conversão para vetor de bytes

```javascript
new Uint8Array(arrayBuffer);
```

Transforma os dados em um vetor numérico.

Cada posição guarda um byte:

```text
0 → 255
```

Exemplo:

```text
[82, 73, 70, 70, 120, 35...]
```

---

# 🟦 ESTRUTURA INTERNA DO WAV

Um arquivo WAV possui duas partes principais:

```text
[ CABEÇALHO ][ DADOS DE ÁUDIO ]
```

---

# 🟦 CABEÇALHO

O cabeçalho possui informações como:

* sample rate;
* número de canais;
* profundidade de bits;
* formato do áudio.

Normalmente ocupa:

```text
44 bytes
```

---

# 🟦 DADOS PCM

Após o cabeçalho ficam:

# as amostras reais do áudio

---

# 🟦 EXTRAÇÃO DOS DADOS SONOROS

## Código

```javascript
const headerSize = 44;

const audioData =
    bytes.subarray(headerSize);
```

---

# 🟦 O QUE ISSO FAZ

O sistema ignora os primeiros 44 bytes do arquivo.

Esses bytes pertencem ao cabeçalho.

Depois disso, ele pega apenas:

# os dados reais do som

---

# 🟦 POR QUE ISSO É IMPORTANTE

Se o cabeçalho for alterado:

* o áudio corrompe;
* o player não reconhece o arquivo;
* o WAV deixa de funcionar.

Por isso o sistema processa apenas os dados PCM.

---

# 🟦 AJUSTE PARA POTÊNCIA DE 2

## Código

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

# 🟦 POR QUE ISSO EXISTE

A FFT exige que o tamanho do vetor seja:

# potência de 2

Exemplos válidos:

```text
256
512
1024
2048
4096
```

---

# 🟦 MOTIVO MATEMÁTICO

A FFT divide o sinal continuamente em metades.

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

Se o tamanho não for potência de 2:

o algoritmo quebra a divisão recursiva.

---

# 🟦 OPERAÇÃO BINÁRIA

## Código

```javascript
p <<= 1
```

Isso significa:

# deslocamento binário para esquerda

---

# 🟦 EXEMPLO

```text
0001 → 0010
```

Decimal:

```text
1 → 2
```

Outro exemplo:

```text
0010 → 0100
```

Decimal:

```text
2 → 4
```

---

# 🟦 FÓRMULA MATEMÁTICA

O algoritmo busca:

2^n \geq N

Onde:

* (N) = tamanho original do áudio
* (2^n) = próxima potência de 2

---

# 🟦 OBJETIVO

Garantir:

* compatibilidade com FFT;
* estabilidade matemática;
* maior velocidade computacional.

---

# 🟦 JANELAMENTO DE HANN

## Código

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

# 🟦 PROBLEMA DO CORTE DIGITAL

O som real é contínuo.

Mas o computador analisa apenas um trecho dele.

Exemplo:

```text
onda infinita
↓↓↓↓↓↓↓↓↓↓
[ trecho analisado ]
```

Esse corte gera:

# descontinuidade

---

# 🟦 CONSEQUÊNCIA

A FFT interpreta o corte como frequências falsas.

Esse fenômeno chama-se:

# Spectral Leakage (vazamento espectral)

---

# 🟦 FÓRMULA DA JANELA DE HANN

w[n]=0.5\left(1-\cos\left(\frac{2\pi n}{N-1}\right)\right)

---

# 🟦 O QUE A JANELA FAZ

Ela suaviza o início e o fim do sinal.

Transforma isso:

```text
██████████
```

em algo mais suave:

```text
▁▃▅▇█▇▅▃▁
```

---

# 🟦 RESULTADO

Reduz:

* frequências fantasmas;
* ruídos artificiais;
* distorções espectrais.

---

# 🟦 FFT — TRANSFORMADA RÁPIDA DE FOURIER

## Código

```javascript
function fft(real, imag) {

    const n = real.length;

    if (n <= 1) return;
}
```

---

# 🟦 O QUE A FFT FAZ

A FFT transforma o sinal do:

```text
Domínio do Tempo
```

para:

```text
Domínio da Frequência
```

---

# 🟦 DOMÍNIO DO TEMPO

Mostra:

```text
amplitude vs tempo
```

---

# 🟦 DOMÍNIO DA FREQUÊNCIA

Mostra:

```text
quais frequências existem no áudio
```

---

# 🟦 EXEMPLO MUSICAL

Uma música possui:

* graves;
* médios;
* agudos.

A FFT separa cada frequência individualmente.

---

# 🟦 FÓRMULA DA DFT

X[k]=\sum_{n=0}^{N-1}x[n]e^{-i\frac{2\pi kn}{N}}

---

# 🟦 SIGNIFICADO DOS SÍMBOLOS

| Símbolo        | Significado           |
| -------------- | --------------------- |
| (x[n])         | sinal original        |
| (X[k])         | frequência encontrada |
| (N)            | número de amostras    |
| (e^{-i\theta}) | rotação complexa      |

---

# 🟦 NÚMEROS COMPLEXOS

A FFT trabalha com:

```text
a + bi
```

Onde:

* (a) = parte real
* (b) = parte imaginária

---

# 🟦 POR QUE NÚMEROS COMPLEXOS

Porque ondas possuem:

* fase;
* direção;
* rotação matemática.

---

# 🟦 IDENTIDADE DE EULER

e^{i\theta}=\cos(\theta)+i\sin(\theta)

---

# 🟦 INTERPRETAÇÃO FÍSICA

Toda onda pode ser representada por:

* senos;
* cossenos;
* rotações circulares.

---

# 🟦 BIT REVERSAL

A FFT reorganiza índices binários.

Exemplo:

```text
011 → 110
```

---

# 🟦 OBJETIVO

Permitir que o algoritmo utilize:

# Divide and Conquer

(dividir para conquistar)

---

# 🟦 COMPLEXIDADE COMPUTACIONAL

DFT tradicional:

```text
O(N²)
```

FFT:

```text
O(N log N)
```

---

# 🟦 RESULTADO

A FFT é extremamente mais rápida.

---

# 🟦 BUTTERFLY OPERATION

## Código

```javascript
const angle =
    -2 * Math.PI / len;

const wrStep =
    Math.cos(angle);

const wiStep =
    Math.sin(angle);
```

---

# 🟦 O QUE ACONTECE AQUI

A FFT combina frequências usando rotações trigonométricas.

Esse processo chama-se:

# Butterfly Operation

---

# 🟦 TRIGONOMETRIA UTILIZADA

O algoritmo usa:

* seno;
* cosseno;
* Euler;
* números complexos.

Para decompor a onda original em frequências menores.

---

# 🟦 CÁLCULO DE MAGNITUDE

## Código

```javascript
out[i] = Math.sqrt(
    real[i] * real[i] +
    imag[i] * imag[i]
);
```

---

# 🟦 PROBLEMA

A FFT retorna números complexos.

Mas o gráfico precisa mostrar:

# intensidade real da frequência

---

# 🟦 FÓRMULA

|X[k]|=\sqrt{Re(X[k])^2+Im(X[k])^2}

---

# 🟦 BASE MATEMÁTICA

Essa fórmula vem do:

# Teorema de Pitágoras

---

# 🟦 INTERPRETAÇÃO FÍSICA

Magnitude representa:

* força;
* energia;
* presença da frequência.

---

# 🟦 EXEMPLO

Se:

```text
440 Hz
```

possui magnitude alta:

então o áudio possui forte presença dessa frequência.

(440 Hz = nota Lá)

---

# 🟦 CONVOLUÇÃO GAUSSIANA

## Kernel Gaussiano

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

# 🟦 PROBLEMA

O espectro FFT bruto é muito irregular.

Exemplo:

```text
| | || ||| |||||| || |
```

---

# 🟦 SOLUÇÃO

Aplicar:

# convolução gaussiana

---

# 🟦 FÓRMULA DA CONVOLUÇÃO

(f*g)[n]=\sum_{k=0}^{K-1}f[n-k]g[k]

---

# 🟦 O QUE É CONVOLUÇÃO

Misturar informações vizinhas.

Cada ponto passa a considerar:

* os pontos ao redor;
* médias ponderadas;
* suavização local.

---

# 🟦 RESULTADO

O gráfico fica:

* mais suave;
* mais limpo;
* mais estável.

---

# 🟦 FILTRO PASSA-BAIXA

O kernel gaussiano funciona como:

# Low Pass Filter

Ele reduz mudanças bruscas no espectro.

---

# 🟦 SÉRIE DE FOURIER

## Código

```javascript
function calculateFourierSeries(
    samples,
    sampleRate,
    fundamentalHz,
    numHarmonics
)
```

---

# 🟦 IDEIA CENTRAL

Fourier descobriu que:

# qualquer onda periódica pode ser construída usando senos e cossenos

---

# 🟦 FÓRMULA GERAL

f(t)=a_0+\sum_{n=1}^{\infty}(a_n\cos(n\omega t)+b_n\sin(n\omega t))

---

# 🟦 HARMÔNICOS

Cada seno adicional é chamado de:

# harmônico

---

# 🟦 EXEMPLO MUSICAL

Se a frequência fundamental é:

```text
440 Hz
```

os harmônicos podem ser:

```text
880 Hz
1320 Hz
1760 Hz
```

---

# 🟦 CÁLCULO DOS COEFICIENTES

## Código

```javascript
an += samples[i] * Math.cos(omega);

bn += samples[i] * Math.sin(omega);
```

---

# 🟦 FÓRMULAS

a_n=\frac{2}{N}\sum x[t]\cos(n\omega_0 t)

b_n=\frac{2}{N}\sum x[t]\sin(n\omega_0 t)

---

# 🟦 OBJETIVO

Descobrir:

* quais senos existem;
* quais cossenos existem;
* intensidade de cada frequência.

---

# 🟦 COMPONENTE DC

## Código

```javascript
a0 += samples[i];
```

---

# 🟦 O QUE É DC

Representa:

# o valor médio do sinal

---

# 🟦 INTERPRETAÇÃO FÍSICA

Corresponde ao:

* deslocamento vertical;
* offset do sinal;
* média da onda.

---

# 🟦 CRIPTOGRAFIA SHA-256

## Código

```javascript
const hashBuffer =
    await crypto.subtle.digest(
        'SHA-256',
        passData
    );
```

---

# 🟦 O QUE É SHA-256

É uma função matemática criptográfica.

Ela transforma uma senha em um hash gigantesco.

---

# 🟦 EXEMPLO

```text
"senha123"
↓
A94F239A...
```

---

# 🟦 PROPRIEDADES

SHA-256 é:

* determinístico;
* irreversível;
* extremamente complexo matematicamente.

---

# 🟦 XOR

## Código

```javascript
audioData[i] ^=
    hashArray[
        i % hashArray.length
    ];
```

---

# 🟦 O QUE É XOR

Operação lógica binária.

---

# 🟦 TABELA XOR

| A | B | Resultado |
| - | - | --------- |
| 0 | 0 | 0         |
| 0 | 1 | 1         |
| 1 | 0 | 1         |
| 1 | 1 | 0         |

---

# 🟦 FÓRMULA

C_i=P_i\oplus K_{(i\bmod 32)}

---

# 🟦 SIGNIFICADO

| Símbolo | Significado        |
| ------- | ------------------ |
| (P_i)   | byte original      |
| (K_i)   | byte da chave      |
| (C_i)   | byte criptografado |

---

# 🟦 PROPRIEDADE MATEMÁTICA

(A\oplus B)\oplus B=A

---

# 🟦 CONSEQUÊNCIA

A mesma operação serve para:

* criptografar;
* descriptografar.

---

# 🟦 INVERSÃO TEMPORAL

## Código

```javascript
audioData.reverse();
```

---

# 🟦 O QUE ISSO FAZ

Inverte completamente o áudio.

---

# 🟦 EXEMPLO

Antes:

```text
[1,2,3,4]
```

Depois:

```text
[4,3,2,1]
```

---

# 🟦 EFEITO FÍSICO

O som toca:

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

# áudio é matemática aplicada

O sistema utiliza:

* álgebra linear;
* trigonometria;
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

# um laboratório matemático e físico de áudio digital

Ele:

* lê sinais sonoros;
* transforma ondas em frequências;
* aplica FFT manual;
* utiliza Fourier e Euler;
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
