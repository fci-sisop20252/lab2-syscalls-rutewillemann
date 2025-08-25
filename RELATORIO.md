# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 9 syscalls
- ex1b_write: 7 syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**

```
Há diferença porque, apesar do método write() ser mais seguro, confiável e previsível, também é mais lento. Já o método printf() é mais rápido, evita tantas chamadas de sistema custosas, porém pode comportamento variado, tornando-o mais imprevisível, devido às regras de buffer. 
```

**3. Qual método é mais previsível? Por quê você acha isso?**

```
O método write() é mais previsível para a saída de dados, porque cada chamada corresponde a uma syscall única e imediata, garantindo que os dados sejam enviados ao sistema operacional no momento exato da execução.
```

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3
- Bytes lidos: 127

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

```
O file descriptor usado foi o 3. Os file descriptors 0, 1 e 2 estão reservados para entrada padrão (stdin), saída padrão (stdout) e saída de erro padrão (stderr), respectivamente. O kernel atribui o próximo número inteiro disponível para um novo arquivo aberto.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
A saída do strace mostra que a chamada read() retornou o valor 127, que é o número máximo de bytes que o buffer pode conter. Isso indica que a operação preencheu completamente o buffer.
```

**3. Por que verificar retorno de cada syscall?**

```
Essa verificação de cada syscall (open, read, close) é importante para fazer tratamento de erros.Um valor de retorno negativo indica um erro, permitindo ao programa reagir de forma segura e evitar falhas inesperadas, garantindo a estabilidade da aplicação.
```

---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: 25 (esperado: 25)
- Caracteres: 1300
- Chamadas read(): 21
- Tempo: 0.000125 segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |        82       | 0.001261  |
| 64          |        21       | 0.000474  |
| 256         |         6       | 0.000276  |
| 1024        |         2       | 0.000211  |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
O tamanho do buffer afeta diretamente na quantidade de chamadas de sistema read(). Quanto maior o buffer, menor a quantidade de syscalls. Isso acontece porque o programa pode ler mais dados de uma vez só.
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

```
Não. Apenas as leituras intermediárias retornaram o tamanho total do buffer. A leitura final sempre retorna um número de bytes menor que o BUFFER_SIZE, pois lê apenas o que resta no arquivo. Por exemplo, com um buffer de 1024 bytes, sua saída mostrou que a última leitura retornou apenas 276 bytes. Após todos os dados serem lidos, uma chamada read() retorna 0, indicando o fim do arquivo.
```

**3. Qual é a relação entre syscalls e performance?**

```
O número de syscalls tem uma relação inversa com a performance, como foi observado na tabela. Fazer muitas chamadas de sistemas, como com um buffer menor, diminui a eficiência e aumenta o tempo de execução. Reduzir o número de syscalls, com um buffer maior, melhora a performance, pois o processador gasta menos tempo alternando entre o modo de usuário e o modo kernel.
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0.000194 segundos
- Throughput: 6866.14 KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [ X ] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Devemos fazer essa verificação para garantir a integridade dos dados, pois a syscall write() pode não conseguir escrever todos os bytes que foram solicitados (quando o disco está cheio, por exemplo). Se o número de dados escritos for menor que o número de dados lidos, significa que a cópia não está completa, e a verificação permite detectar essa falha e tratar o erro imeditamente, interrompendo a operação.
```

**2. Que flags são essenciais no open() do destino?**

```
As flags essenciais são O_WRONLY, O_CREAT, O_TRUC. 
- O_WRONLY: Abrir apenas para escrita
- O_CREAT: Criar arquivo se não existir
- O_TRUNC: Truncar arquivo existente (limpa o arquivo)
```

**3. O número de reads e writes é igual? Por quê?**

```
Sim, o número é igual. Porque para cada bloco de dados lidos no arquivo de origem, uma operação de write correspondente é feita para gravar o mesmo bloco no arquivo de destino. Isso garante que a gravação seja feita bloco por bloco, com cada read() sendo seguido de um write().
```

**4. Como você saberia se o disco ficou cheio?**

```
É possível saber verificando se o valor retornado de write() é menor do que o número de bytes lidos.
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
Se esquecer de fechar os arquivos, o sistema operacional continua a "segurar" esses arquivos, gastando recursos desnecessariamente. Com o tempo, se isso ocorrer repetidamente, o sistema pode ficar sobrecarregado e até travar, já que não conseguirá abrir novos arquivos. Ademais, os dados escritos podem ficar presos na memória e não serem salvos no disco, resultando em arquivos incompletos.
```

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
As chamadas de sistemas são a forma segura de transitar entre o espaço de usuário e o kernel, permitindo que programas solicitem serviços do kernel sem comprometer a segurança do sistema. Ao chamar uma função, como read() ou write(), o sistema interrompe sua execução, o kernel executa a operação solicitada e, ao final, devolve o controle e o resultado da operação para o programa. Essa transferência de controle é a transição usuário → kernel.

```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
Os file descriptors são indicadores numéricos que o kernel atribui a arquivos abertos e outros recursos de entrada/saída. Eles são importantes pois oferecem uma forma simples, padronizada e eficiente para o programa interagir com esses recursos, usando apenas um número inteiro para ler, escrever ou fechar o arquivo.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Um buffer maior permite que mais dados sejam lidos a cada chamada de sistema, o que reduz o número final de syscalls. Como cada transição para o kernel tem um custo, menos syscalls signifia menos overhead,implicando na melhora da performance. Analogamente, ter uma carteira (buffer) grande reduz o custo de ir ao banco (kernel) para ter mais dinheiro. O exercício 3 demonstrou essa relação.
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** 

```
O comando cp do sistema.
```

**Por que você acha que foi mais rápido?**

```
Porque o cp é altamente otimizado para a tarefa de cópia de arquivos. Ele executa a operação de forma mais eficiente do que um programa que utiliza um loop de leitura e escrita. O overhead das chamadas de sistema desse programa, mesmo com um buffer grande, faz com que o cp tenha uma performance superior.
```

---

## 📤 Entrega
Certifique-se de ter:
- [X] Todos os códigos com TODOs completados
- [X] Traces salvos em `traces/`
- [X] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
