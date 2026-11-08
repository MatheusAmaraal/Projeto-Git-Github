# "Na minha máquina não roda"

Uma situação comum em times de dados: um analista termina um script que puxa dados de uma API, trata com pandas e grava no banco. Funciona perfeitamente no computador dele. Um colega clona o projeto para dar continuidade e, logo na primeira execução, aparece um `ModuleNotFoundError`. Depois de instalar as bibliotecas na mão, surge outro erro: a versão do pandas é diferente e uma função mudou de comportamento. Quando finalmente roda, o colega percebe que a senha do banco está escrita no código e já foi publicada no GitHub.

Nenhum desses problemas tem a ver com a lógica do script. Todos vêm da falta de um ambiente bem configurado.

Este projeto mostra como evitar isso: versão do Python controlada por projeto, dependências isoladas e declaradas, credenciais fora do código e versionamento com Git e GitHub. O resultado é um projeto que roda na minha máquina **e na de qualquer pessoa que clonar o repositório**.

Projeto desenvolvido no curso de Formação em Inteligência Artificial & Automações da **Xperiun**. O foco não é a análise de dados, e sim a base que todo projeto de dados precisa ter: um ambiente que qualquer pessoa consegue clonar e executar sem ajustes manuais.

## O que este projeto mostra

- **pyenv** para usar versões diferentes do Python em cada projeto
- **Ambientes virtuais** (`venv` e `.venv`) para isolar as bibliotecas
- **uv** para criar o projeto e gerenciar dependências, registradas no `pyproject.toml` e fixadas no `uv.lock`
- **`.env` com python-dotenv** para manter credenciais fora do código
- **`.gitignore`** para impedir que dados sensíveis cheguem ao repositório
- **`.env.example`** para indicar quais variáveis o projeto precisa
- **Git e GitHub** com commits seletivos e publicação do projeto

## Estrutura

```
projeto-final/
├── src/
│   └── projeto_final/
│       └── __init__.py
├── .env.example       # modelo das variáveis de ambiente, sem valores
├── .gitignore         # arquivos que ficam fora do versionamento
├── .python-version    # versão do Python usada no projeto
├── main.py            # lê as variáveis de ambiente
├── pyproject.toml     # informações e dependências do projeto
├── README.md
└── uv.lock            # versões exatas das dependências
```

O `.env`, com os valores reais, fica apenas na máquina local.

## Como executar

```powershell
git clone https://github.com/MatheusAmaraal/Projeto-Git-Github.git
cd Projeto-Git-Github
uv sync
copy .env.example .env
```

Depois de preencher o `.env` com as suas credenciais:

```powershell
uv run main.py
```

---

# Passo a passo

Executei tudo no Windows, pelo terminal do VS Code (PowerShell). A única exceção foi a instalação do pyenv, que precisa do PowerShell como administrador.

## 1. Instalação do pyenv

Instalei o **pyenv-win** com o script oficial do repositório [pyenv-win](https://github.com/pyenv-win/pyenv-win). Em seguida, abri um novo terminal no VS Code para que o pyenv fosse reconhecido.

## 2. Duas versões do Python na mesma máquina

```powershell
pyenv install 3.10.11
pyenv install 3.12.0
pyenv versions          # lista as versões instaladas
pyenv global 3.12.0     # define a versão padrão da máquina
python --version
```

As duas versões convivem sem conflito: instalar uma não substitui a outra.

## 3. Uma versão para cada projeto

```powershell
mkdir projeto-a
mkdir projeto-b

cd projeto-a
pyenv local 3.10.11     # projeto-a usa Python 3.10
cd ..\projeto-b
pyenv local 3.12.0      # projeto-b usa Python 3.12
```

O `pyenv local` cria o arquivo `.python-version` na pasta. A partir daí, a versão certa é usada automaticamente sempre que entro nela.

## 4. Ambiente virtual com venv

```powershell
cd projeto-a
python -m venv venv         # cria o ambiente virtual na pasta "venv"
venv\Scripts\activate       # ativa o ambiente
pip install pandas
pip freeze                  # lista os pacotes instalados
deactivate                  # desativa o ambiente
```

Aqui aparece uma limitação do `pip`: instalei apenas o pandas, mas o `pip freeze` listou vários pacotes, porque ele traz suas próprias dependências. Com o tempo, fica difícil separar o que foi escolhido do que veio junto. Por isso usei o **uv** no projeto principal.

## 5. Instalação do pipx e do uv

```powershell
pip install pipx
python -m pipx ensurepath   # depois deste comando, reabri o terminal
pip install uv
uv --version
```

- **pipx**: instala ferramentas de linha de comando isoladas dos projetos
- **uv**: cuida do ambiente virtual, das dependências e da execução em um só lugar

## 6. Criação do projeto com uv

```powershell
uv init projeto-final
cd projeto-final
uv add pandas
uv add requests
uv add python-dotenv
Get-Content pyproject.toml
```

A vantagem do `uv add` sobre o `pip install` é que ele registra a dependência no `pyproject.toml`, que mostra só os pacotes que eu escolhi. O `uv.lock` guarda as versões exatas de tudo, o que garante que o ambiente seja o mesmo em qualquer máquina.

Para ativar o ambiente criado pelo uv:

```powershell
.venv\Scripts\Activate.ps1
```

## 7. Variáveis de ambiente no .env

```powershell
New-Item .env
```

Conteúdo, com valores fictícios para teste:

```
DB_HOST=servidor.banco.com
DB_USER=admin
DB_PASSWORD=senha-super-secreta-123
API_KEY=ghp_chave-fake-para-teste
```

Assim o código só indica quais variáveis precisa, e os valores vêm de fora:

```python
senha = "minha-senha-real"        # evitar: a senha fica exposta no código
senha = os.getenv("DB_PASSWORD")  # recomendado: o valor vem do .env
```

## 8. Leitura das variáveis no Python

`main.py`:

```python
from dotenv import load_dotenv
import os

load_dotenv()  # carrega as variáveis do .env

db_host = os.getenv("DB_HOST")
db_user = os.getenv("DB_USER")
db_password = os.getenv("DB_PASSWORD")
api_key = os.getenv("API_KEY")

print(f"Conectando em {db_host} com usuário {db_user}")
print("Credenciais carregadas com sucesso!")
```

```powershell
uv run main.py
```

Saída:

```
Conectando em servidor.banco.com com usuário admin
Credenciais carregadas com sucesso!
```

## 9. Protegendo o .env

Adicionei ao `.gitignore`:

```
# Credenciais
.env
```

Com isso, o Git ignora o arquivo e as credenciais não são publicadas.

## 10. Criação do .env.example

```powershell
New-Item .env.example
```

Conteúdo, com as mesmas chaves do `.env`, mas sem valores:

```
DB_HOST=
DB_USER=
DB_PASSWORD=
API_KEY=
```

Diferente do `.env`, este arquivo vai para o repositório. Ele mostra quais variáveis o projeto exige e serve de modelo para quem for configurar o ambiente.

## 11. Versionamento com Git

```powershell
git init
git status
git add .gitignore README.md main.py pyproject.toml uv.lock
git commit -m "Salvando meu projeto final e subindo para o GitHub"
```

Adicionei os arquivos um a um, em vez de usar `git add .`, para controlar exatamente o que entraria no primeiro commit. O `.env` não aparece no `git status`, o que confirma que o `.gitignore` está funcionando.

## 12. Publicação no GitHub

Criei um repositório vazio no GitHub e conectei o projeto local a ele:

```powershell
git branch -M main
git remote add origin https://github.com/MatheusAmaraal/Projeto-Git-Github.git
git push -u origin main
```

- `git branch -M main`: renomeia a branch principal de `master` para `main`
- `git remote add origin`: conecta o repositório local ao GitHub
- `git push -u origin main`: envia os commits e vincula a branch local à remota

Os demais arquivos (`src/`, `.python-version`, `.env.example` e este README) foram enviados depois, com o fluxo do dia a dia:

```powershell
git add .
git commit -m "Adiciona pasta src, .env.example e documentação no README"
git push
```

## Resultado

| | Arquivos |
|---|---|
| No repositório | `src/`, `.env.example`, `.gitignore`, `.python-version`, `main.py`, `pyproject.toml`, `uv.lock`, `README.md` |
| Apenas local | `.env` |

---

**Matheus Amaral** · Dados e Business Intelligence · [GitHub](https://github.com/MatheusAmaraal)