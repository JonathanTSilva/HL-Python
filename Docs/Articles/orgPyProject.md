<!-- LOGO DIREITO -->
<a href="#"><img width="300px" src="https://miro.medium.com/max/1400/1*jRO4h3b3f8cmsxW81NCoig.png" align="right" /></a>

# Como organizar um projeto Python?

<p align="left">
  <a href="https://github.com/JonathanTSilva/HL-Linux">
    <img src="https://img.shields.io/static/v1?label=Home%20Lab&message=Python&color=yellow&logo=python&logoColor=white&labelColor=blue&style=flat" alt="HL-Linux">
  </a>
</p>

🔧 Este artigo apresenta minha forma de organizar os projetos em Python, baseando-se nos artigos do Eduardo Mendes e André Felipe Dias.

<!-- SUMÁRIO -->
- [Como organizar um projeto Python?](#como-organizar-um-projeto-python)
  - [Estrutura](#estrutura)
  - [Ferramentas de desenvolvimento](#ferramentas-de-desenvolvimento)
  - [Ferramentas de *checkagem*](#ferramentas-de-checkagem)
  - [Automações](#automações)
    - [GNU Make](#gnu-make)
    - [Pre-commit](#pre-commit)
    - [GitHub Actions / GitLab CI / BitBucket Bamboo](#github-actions--gitlab-ci--bitbucket-bamboo)

<!-- VOLTAR AO INÍCIO -->
<a href="#"><img width="40px" src="https://github.com/JonathanTSilva/JonathanTSilva/blob/main/Images/back-to-top.png" align="right" /></a>

## Estrutura

Uma boa estrutura se baseia nestes aspectos:

- Separação de responsabilidade
  - Cada coisa deve estar no seu devido lugar
    - `docs/`
    - `src/` (código-fonte - em Python, *nome do projeto*)
    - `tests/`
- 

<!-- VOLTAR AO INÍCIO -->
<a href="#"><img width="40px" src="https://github.com/JonathanTSilva/JonathanTSilva/blob/main/Images/back-to-top.png" align="right" /></a>

## Ferramentas de desenvolvimento

1. **Escolher a versão do Python**
   - Preferencialmente usar a última possível
   - Nem sempre as bibliotecas que dependemos, oferecem suporte a última versão do Python, cabe a nós descobrirmos qual a última versão suportada por uma biblioteca que dependemos
   - Dica de ferramenta: `asdf` (`pyenv` também)
2. **Escolher ferramenta para ambientes virtuais**
   - Escolha pessoal
   - Opções: `virtualenv`, `venv`, `poetry`, `pipenv`,...
   - Recomendação pessoal: `poetry`
3. **Versionamento de código**
   - Partir pro Git
   - Escolher o repositório de preferência (GitHub, GitLab, BitBucket, etc.)
   - Não esquecer do [`.gitignore`][1]
4. **Adicionar testes**
   - Testes são uma peça fundamental de qualquer projeto Python
   - Garantem horas de sanidade mental
   - Diversas ferramentas: `pytest`, `unittest`, `behave`, `ward`,...
5. **Formatadores automáticos de código**
   - Já sentiu que quando você programa com outra pessoa as coisas tem uma "estática" um pouco diferente? Como assim?
     - Ordem de imports diferentes
     - " ou '?
     - Quebra de linha de maneiras diferentes
     - Tamanho de linhas sem padrão
     - ...
   - Para buscar uma padronização na maneira de escrever código, temos ferramentas de formatação: `blue`, `black`, `isort`, `autopep8`, `YAPF`, `darker*`, ...
6. **Criadores de documentação**
   - Formatar documentação nem sempre é fácil. Para isso existem ferramentas como: `mkdocs`, `sphinx`,...

<!-- VOLTAR AO INÍCIO -->
<a href="#"><img width="40px" src="https://github.com/JonathanTSilva/JonathanTSilva/blob/main/Images/back-to-top.png" align="right" /></a>

## Ferramentas de *checkagem*

1. **Análise estática (*linters*)**
   - Diferente dos testes, as ferramentas de análise estática procuram:
     - Erros de sintaxe
     - Potenciais problemas
       - Nomes duplicados
       - Nomes ruins
       - Códigos inseguros
     - Análise de complexidade de código
     - Violações de convenções
       - PEP-8
       - PEP-257
   - Algumas recomendações de ferramentas:
     - `flake8` - checkador da PEP-8
     - `pylint` - padronização, convenções e erros
     - `pydocstyle` - checkador da PEP-257
     - `bandit` - problemas de segurança
     - `radon` - busca de complexidade de código
     - `prospector` - agregador de ferramentas
       - flake8, mccabe, pylint, pep8, pep257, ...
     - `mypy` - checkador de sugestões de tipo
     - ...
2. **Segurança de bibliotecas**
   - Com o passar do tempo, às vezes algumas bibliotecas ficam "congeladas" no nosso projeto. E isso pode trazer várias vulnerabilidades de segurança.
   - Algumas ferramentas: `pip-audit`, `safety`, `pyup`, ...

<!-- VOLTAR AO INÍCIO -->
<a href="#"><img width="40px" src="https://github.com/JonathanTSilva/JonathanTSilva/blob/main/Images/back-to-top.png" align="right" /></a>

## Automações

- Até agora vimos que muitos passos devem ser realizados para seu projeto ficar em dia com as normas e convenções: black, isort, poetry, prospector, pip-audit, pytest, mkdocs, ...

### GNU Make

```makefile
# Makefile
.PHONY: install format lint test sec

install:
    @poetry install
format:
    @blue .
    @isort .
lint:
    @blue --check .
    @isort --check .
    @prospector --with-tool pep257 --doc-warning
test:
    @pytest -v
sec:
    @pip-audit
```

```console
$ make format
```

### Pre-commit

- Comandos mais simples fazem com que nós os executemos mais, entretanto, há sempre o "fator humano" (simplesmente esquecemos)
- Podemos usar os **hooks** do **git** para rodas as coisas
  - Antes do commit
  - Antes do rebase
  - Antes do push
  - Antes do merge
- É possível escrever esses hooks dentro de `.git/hooks/...`

### GitHub Actions / GitLab CI / BitBucket Bamboo

- Só que, ainda assim, pode ser que haja um erro no GitHub, por exemplo, ao ser feito um merge de fork.
- Para isso temos o Actions do GitHub, ou o Runners do GitLab, ou o Bamboo do BitBucket
- Basicamente, configurar um arquivo no GitHub que, quando realizado um push, realizar algumas atividades

```yml
# .github/workflows/continuous_integration.yml

name: Continuous Integration
on: [push, merge]
jobs:
    lint_and_test:
        runs-on: ubuntu-latest
        steps:

          - name: Set up python
            uses: actions/setup-python@v2
            with:
                python-version: 3.10

          - name: Check out repository
            uses: actions/checkout@v2

          - name: Install Poetry
            uses: snok/install-poetry@v1
            with:
                virtualenvs-in-project: true

          - name: Load cached venv
            id: cached-poetry-dependencies
            uses: actions/cache@v2
            with:
                path: .vemv
                key: venv-${{ hashFiles('**/poetry.lock') }}
        
          - name: Install dependencies
            if: steps.cached-poetry-dependencies.outputs.cache-hit != 'true'
            run: poetry install --no-interaction

          - name: Lint
            run: poetry run make lint

          - name: Run tests
            run: poetry run make test
            
```

## Pythonic

<!-- MARKDOWN LINKS -->
<!-- SITES -->
[1]: https://www.toptal.com/developers/gitignore/

<!-- IMAGES -->
[A]: asdfsda
