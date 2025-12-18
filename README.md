# TinyML – Classificação do Dataset Wine no Raspberry Pi Pico W  
### Prática com Rede Neural Artificial (RNA) para Microcontroladores

Este projeto implementa uma **Rede Neural Artificial (RNA)**, Perceptron Multicamadas (MLP), embarcada no **Raspberry Pi Pico W**, utilizando a biblioteca **TensorFlow Lite Micro (TFLM)** para executar inferência diretamente no microcontrolador — abordagem típica de **TinyML**.

Este código faz parte de um projeto que demonstra como treinar, converter e executar um modelo inteligente real em um dispositivo de recursos extremamente limitados. Como conteúdo complementar, o modelo foi treinado usando o google colab, o link do código está disponível em: https://colab.research.google.com/drive/1MnmXluBn_oCctJ-MPaiS2RxqsRbwg4Fk?usp=sharing 

---

## 📌 Objetivos

- Demonstrar o fluxo completo de TinyML:  
  **Criação do modelo → Treinamento → Conversão → Deploy → Inferência embarcada**
- Normalizar dados embarcados de forma idêntica ao treinamento.
- Executar inferências usando TFLM. Biblioteca disponível em: https://github.com/raspberrypi/pico-tflmicro.git
- Construir e imprimir a **matriz de confusão** 3×3.
- Calcular a acurácia final diretamente no microcontrolador.
- Integrar código C/C++ ao TensorFlow Lite Micro via wrapper.

---

## 🧠 Visão geral

A aplicação embarcada no Pico W:

1. Carrega um modelo **MLP (rede neural multicamadas)** treinado com o dataset Wine.
2. Aplica normalização padrão (média e desvio) nas primeiras 4 características.
3. Executa inferência nas primeiras 150 amostras do dataset.
4. Constrói a **matriz de confusão 3×3** (real × predito).
5. Calcula a acurácia final da rede.
6. Exibe tudo via USB/serial.

Essa prática permite que estudantes compreendam como modelos inteligentes podem ser executados em **microcontroladores**, base fundamental para aplicações TinyML e Edge AI.

---

## 📁 Organização dos arquivos

### `tiny_ml.c`
Aplicação principal em C.  
Responsável por:

- Inicializar o Pico W e o ambiente TFLM.  
- Normalizar as primeiras 4 características de cada amostra com `wine_means` e `wine_stds`.  
- Realizar inferências nas primeiras 150 amostras via `tflm_infer()`.  
- Construir a matriz de confusão.  
- Calcular a acurácia e imprimir os resultados.

---

### `tflm_wrapper.h` / `tflm_wrapper.cpp`
Wrapper em C/C++ para o TensorFlow Lite Micro. Forma uma camada de abstração que encapsula o TensorFlow Lite Micro, oferecendo funções simples para inicializar o modelo, passar entradas e pegar saídas, sem que você precise lidar diretamente com todos os detalhes internos da biblioteca.

- Configura a arena de tensores.  
- Carrega o modelo embarcado (`wine_mlp_float_tflite`).  
- Registra operações necessárias (FullyConnected, ReLU, Softmax, Reshape).  
- Expõe:
  - `tflm_init_model()`  
  - `tflm_infer(float input[4], float output[3])` - Utiliza as primeiras 4 características do Wine

---

### `wine_mlp_float.h`
Modelo TFLite convertido para array C (`unsigned char[]`), contendo a rede neural MLP treinada previamente em Python.

---

### `wine_dataset.h`
Dataset Wine embarcado no firmware:

- `wine_features[178][13]` - 178 amostras com 13 características cada
- `wine_labels[178]` - Classes correspondentes (0, 1 ou 2)

O dataset Wine contém 13 atributos químicos de vinhos italianos:
- Alcohol, Malic acid, Ash, Alcalinity of ash, Magnesium
- Total phenols, Flavanoids, Nonflavanoid phenols, Proanthocyanins
- Color intensity, Hue, OD280/OD315 of diluted wines, Proline

**Nota**: A implementação atual utiliza apenas as primeiras 4 características do dataset para a inferência.

---

### `wine_normalization.h`
Estatísticas de normalização utilizadas:

- `wine_means[13]` - Média de cada atributo
- `wine_stds[13]` - Desvio padrão de cada atributo

Esses valores replicam exatamente o StandardScaler do treinamento, garantindo consistência na inferência.

**Nota**: A implementação atual utiliza apenas as primeiras 4 médias e desvios padrão.

---

### `CMakeLists.txt`
Arquivo de build usando pico-sdk + TFLM:

- Configuração do projeto
- Inclusão do TensorFlow Lite Micro
- Compilação dos arquivos `.c` e `.cpp`
- Links com bibliotecas padrão do Pico

---

## 🔧 Como compilar o projeto

### 1. Instale o Pico SDK
Disponível em:  
https://github.com/raspberrypi/pico-sdk

---

### 2. Configure e compile
```bash
mkdir build
cd build
cmake ..
make -j4
```

O processo de build irá:
- Baixar e compilar a biblioteca pico-tflmicro (se não estiver na pasta `lib/`)
- Compilar os arquivos do projeto
- Gerar os binários `.uf2`, `.elf` e `.bin`

---

### 3. Faça o upload para o Pico W

1. Segure o botão **BOOTSEL** no Pico W
2. Conecte-o ao computador via USB
3. Copie o arquivo `tiny_ml.uf2` (da pasta `build/`) para o drive que aparece
4. O Pico W reiniciará automaticamente

---

### 4. Monitore a saída

Use um terminal serial para ver os resultados:

```bash
# Linux/macOS
screen /dev/ttyACM0 115200

# Ou use minicom
minicom -D /dev/ttyACM0 -b 115200
```

No Windows, use PuTTY ou outro terminal serial na porta COM correspondente.

---

## 📊 Saída esperada

A aplicação exibe:

- Status de inicialização do modelo
- Dimensões dos tensores de entrada e saída
- Primeiras 10 predições com scores de probabilidade
- Matriz de confusão 3×3 completa
- Acurácia final do modelo

Exemplo:
```
=== TinyML wine - Matriz de Confusao ===
Modelo inicializado com sucesso!
Iniciando inferencia nas 150 amostras do dataset wine...

Amostra   0  Real: 0  Pred: 0  [0.987 0.012 0.001]
Amostra   1  Real: 0  Pred: 0  [0.991 0.008 0.001]
...

Matriz de Confusao (real vs predito)
          Pred0   Pred1   Pred2
Real 0       50        0        0
Real 1        0       48        2
Real 2        0        1       49

Acuracia final: 0.9800  ( 147 / 150 )
```

---

## 🔍 Detalhes técnicos

### Estrutura do modelo
- **Entrada**: 4 features (primeiras características do Wine dataset)
- **Arquitetura**: MLP com camadas densas + ativações ReLU
- **Saída**: 3 classes (Softmax)
- **Formato**: TensorFlow Lite (.tflite convertido para array C)

### Recursos utilizados
- **Memória arena**: 8KB para tensores intermediários
- **Modelo**: ~5KB (incluído no firmware)
- **Dataset**: ~3KB embarcado
- **Total estimado**: ~15-20KB de RAM

### Limitações da implementação atual
- Utiliza apenas as **primeiras 4 características** das 13 disponíveis no Wine dataset
- Processa apenas as **primeiras 150 amostras** das 178 disponíveis
- Modelo treinado com 4 features (versão simplificada)

---

## 🎓 Conceitos aprendidos

Este projeto demonstra:

1. **TinyML**: Machine Learning em microcontroladores
2. **TFLite Micro**: Conversão e deploy de modelos TensorFlow
3. **Normalização embarcada**: Pré-processamento de dados
4. **Inferência on-device**: Processamento local sem nuvem
5. **Avaliação de modelo**: Matriz de confusão e acurácia
6. **Integração C/C++**: Wrapper para biblioteca C++ em código C

---

## 📚 Recursos adicionais

- [TensorFlow Lite Micro](https://www.tensorflow.org/lite/microcontrollers)
- [Raspberry Pi Pico SDK](https://github.com/raspberrypi/pico-sdk)
- [Pico TFLite Micro](https://github.com/raspberrypi/pico-tflmicro)
- [Wine Dataset (UCI)](https://archive.ics.uci.edu/ml/datasets/wine)
- [Google Colab - Treinamento](https://colab.research.google.com/drive/1MnmXluBn_oCctJ-MPaiS2RxqsRbwg4Fk?usp=sharing)

---

## 🤝 Contribuindo

Contribuições são bem-vindas! Sinta-se à vontade para:

- Reportar bugs
- Sugerir melhorias
- Enviar pull requests
- Melhorar a documentação

---

## 📄 Licença

Este projeto é distribuído para fins educacionais como parte do programa EmbarcaTech.

---

## ✨ Autores

Desenvolvido como material didático para demonstração de TinyML em microcontroladores.
