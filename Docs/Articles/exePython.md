<!-- LOGO DIREITO -->
<a href="#"><img width="300px" src="https://miro.medium.com/max/1400/1*Dd0-ftvJxAcSdSZHgNrz0w.png" align="right" /></a>

# Como gerar executáveis de um projeto Python?

<p align="left">
  <a href="https://github.com/JonathanTSilva/HL-Linux">
    <img src="https://img.shields.io/static/v1?label=Home%20Lab&message=Python&logo=python&color=yellow&logoColor=white&labelColor=blue&style=flat" alt="HL-Linux">
  </a>
</p>

🔋 Um overview das noções básicas de Python.

<!-- SUMÁRIO -->
- [Como gerar executáveis de um projeto Python?](#como-gerar-executáveis-de-um-projeto-python)
  - [Overview](#overview)
    - [O tabu dos executáveis](#o-tabu-dos-executáveis)
    - [Executável VS Interpretador](#executável-vs-interpretador)
    - [Como distribuir o meu código?](#como-distribuir-o-meu-código)
  - [PyInstaller](#pyinstaller)
    - [Como o PyInstaller empacota as coisas?](#como-o-pyinstaller-empacota-as-coisas)
    - [Runtime vs Runtime: entendendo o bundle](#runtime-vs-runtime-entendendo-o-bundle)
    - [Importes ocultos, *hooks* e *bootloader*](#importes-ocultos-hooks-e-bootloader)

> **Nota:** este documento foi desenvolvido por base nos vídeos publicados pelo Eduardo Mendes, na série [O tabu dos executáveis (TDE)][1].

<!-- VOLTAR AO INÍCIO -->
<a href="#"><img width="40px" src="https://github.com/JonathanTSilva/JonathanTSilva/blob/main/Images/back-to-top.png" align="right" /></a>

## Overview

Para gerar um executável a partir de um projeto Python, você pode usar ferramentas como `PyInstaller` `cx_Freeze` ou `Py2exe`. 
Essas ferramentas empacotam seu código Python juntamente com o interpretador Python e quaisquer bibliotecas externas necessárias em um único arquivo executável, permitindo que você distribua seu programa como um aplicativo independente.

### O tabu dos executáveis

Alguns dos motivos pelos quais você pode querer gerar um executável?

- Ofuscação de código (`pyArmor`)
  - "Ninguém pode saber as regras de negócio"
  - Não dá pra ler
  - Ainda não existe engenharia reversa
  - O pyArmor não precisa estar instalado no cliente
  - O código pode ter data de expiração
  - Bloqueio de máquinas para execução (MAC, Serial do HD, ...)
- Velocidade
  - Se gerar o executável do código ele não rodará mais rápido
  - O jeito certo pra isso, é usando JIT (Just-In-Time): a ideia jo JIT é compilar códigos "quentes". Imagine um bloco de código "caro" em tempo de execução. A partir do momento em que ele precisa ser executado diversas vezes, essa parte "cara" do código passa a ser compilada.
  - `LLVM`, `Numba`, `pypy`
- Compilados (`cython` - compila python pra C, `nuitka`)
- Distribuição
  - Plataformas
  - PyInstaller

### Executável VS Interpretador

![arquivosExe][A]

Um arquivo executável contém as instruções necessárias para dizer a máquina o que fazer. Embora não possa ser lido, ele contém todas as instruções de máquina, que fazem o fluxo do arquivo Python.

O fluxo de um código Python segue:

1. Arquivo fonte (código)
2. Interpretador (python.exe)
   1. Token (`python -m tokenize exemplo.py`)
   2. Parse
   3. AST (`python -m ast exemplo.py` - árvore sintática)
   4. Byte Code (`python -m py_compile exemplo.py`)
3. Máquina (00011011)

### Como distribuir o meu código?

Avaliando o quadro geral das ferramentas:

- PyInstaller:
  - o estado da arte
  - multiplataforma
  - build spec
  - troca de bootloaders
  - reconhecimento de bibliotecas complexas (QT, GTK, ...)
  - ** Muitos falsos positivos
- cx_Freeze
  - Usa o setup.py
  - Multiplataforma
  - Não desempacote em memória
  - ** Empacota a `python.dll`, o que trá muitos problemas no linux
- PyOxidizer
  - Ferramenta nova - Rust
  - multiplataforma
  - bezel scripts
  - Não desempacote em memória
  - ** Só suporta python 3.8+; bezel scripts
- PyInsist
  - Gera instaladores (SNIS)
  - Somente para Windows
  - ** Só funciona no windows, problemas com empacotamento

## PyInstaller

PyInstaller é uma ferramenta popular e amplamente usada para empacotar programas Python em executáveis independentes. Ele converte seu código em um único arquivo executável que contém o interpretador Python, suas dependências e qualquer recurso necessário para a execução do programa. Isso sem a necessidade de ter o Python instalado no sistema do usuário final, facilitando a distribuição e o compartilhamento de seus programas.

O PyInstaller suporta várias plataformas, incluindo **Windows**, **macOS** e **Linux** (assim como **AIX**, **Solaris**, **FreeBSD** e **OpenBSD**), permitindo que você gere executáveis para diferentes sistemas operacionais a partir do mesmo código-fonte. Também possui recursos para lidar com dependências, como bibliotecas externas e recursos estáticos, permitindo que você inclua todos os componentes necessários no executável final.

Além disso, o PyInstaller fornece opções para personalizar o processo de empacotamento, como definir ícones para o executável, especificar arquivos adicionais a serem incluídos e ajustar opções de otimização e compactação.

O primeiro commit do PyInstaller foi feito em 2005, e suas versões 1.0 e 5.7.0 foram lançadas, respectivamente, em 2015 e 2022. A licença atual é DUAL, ou seja, está licenciado sobre a GPL2+ e Apache 2.0.

### Como o PyInstaller empacota as coisas?

Antes de entender seu funcionamento, deve-se levantar a questão do que o PyInstaller NÃO É:

1. PyInstaller é um criador de "bundles" [agrupador] de pacotes. Isso quer dizer que ele **não é cum compilador de código** Python como `Cython`, `nuitka` ou `mypyc`.
2. PyInstaller **não é um otimizador de código**. Um bundle criado com ele não é mais rápido que um código interpretado.
3. PyInstaller **não é um ofuscador de código**. É possível fazer engenharia reversa no bundle gerado.
4. PyInstaller **não é um criador de instaladores**. Você deve conduzir isso a parte com ferramentas como `inno setup`, `NSIS`, etc. para instalar seu bundle.

O PyInstaller junta tudo em um lugar só.

O PyInstaller não MULTIPLATAFORMA. Por varrer as dependências da sua instalação e usar o seu python como base, o executável gerada não é cross-system. Isso quer dizer que um executável gerado no Linux, só funcionará no Linux. No Windows, somente no Windows.

Tanto as bibliotecas e o interpretador são compilados para arquiteturas diferentes. Como 32-bits e 64-bits. Então um bundle gerado em um Linux 64-bits não funcionará em um 32-bits. O mesmo vale para outros sistemas.

Como bibliotecas (.so e DLLs) são coletadas do sistema, em teoria, versões mais antigas de sistemas são mais propensas a maior compatibilidade entre sistemas mais novos. Ex.: Bundle de Win8 funciona no Win11, porém o inverso não acontece.

### Runtime vs Runtime: entendendo o bundle

### Importes ocultos, *hooks* e *bootloader*

<!-- MARKDOWN LINKS -->
<!-- SITES -->
[1]: https://www.youtube.com/playlist?list=PLOQgLBuj2-3Kpx-Qaf00mVUUmSgv4Vinx

<!-- IMAGES -->
[A]: https://i.imgur.com/e0KHGEj.png
